+++
title = "HackTheBox - Simple Encryptor"
draft = false
description = "Resolución del desafío Simple Encryptor"
tags = ["HTB", "Reversing", "Easy", "Ghidra", "Encryption"]
summary = "Dificultad: Easy | Conceptos: Reversing, Ghidra, Cifrado, XOR, Rotaciones"
categories = ["Writeups"]
showToc = true
showRelated = true
date = "2025-10-19T00:00:00"
+++

**CHALLENGE DESCRIPTION**
>*On our regular checkups of our secret flag storage server we found out that we were hit by ransomware! The original flag data is nowhere to be found, but luckily we not only have the encrypted file but also the encryption program itself.*

Archivos iniciales:
- `encrypt`: ELF 64-bit. El programa de cifrado
- `flag.enc`: El Flag cifrado

## Análisis inicial

Usando el program `strings` podemos ver si hay datos detectados como strings dentro de cualquiera de los dos archivos:
```bash
$ strings encrypt
/lib64/ld-linux-x86-64.so.2
libc.so.6
srand #Generación del seed de nums aleatorios
fopen #Abrir archivo en memoria
ftell #Ver dirección del puntero del archivo
time #Ver tiempo del sistema
__stack_chk_fail #Por el nombre, se intuye que comprueba la integridad del stack
fseek #Mover puntero del archivo a un punto específico
fclose #Cerrar archivo
malloc #Asignar memoria
fwrite #Escribir en archivo
fread #Leer archivo
__cxa_finalize
[...]
$
```

Del archivo `encrypt` podemos ver varias funciones que pueden dar una primera imagen del funcionamiento del binario. Por otro lado, para `flag.enc`:

```bash
$ strings flag.enc
$ #No se encuentra ningún string, lo normal para un archivo cifrado.
```

---

Al ejecutar el programa con `strace` (*syscall trace*) podemos ver las syscalls que realiza el programa en runtime:
```bash
strace ./encrypt         
execve("./encrypt", ["./encrypt"], 0x7fff2fea2170 /* 68 vars */) = 0
brk(NULL)                               = 0x55f5d61ee000

...[SNIP]...

openat(AT_FDCWD, "flag", O_RDONLY)      = -1 ENOENT (No existe el fichero o el directorio)
--- SIGSEGV {si_signo=SIGSEGV, si_code=SEGV_MAPERR, si_addr=0x1} ---
+++ killed by SIGSEGV (core dumped) +++
[1]    11282 segmentation fault (core dumped)  strace ./encrypt

```
Aquí podemos ver que, al ejecutar el programa, este busca un archivo `flag` en el mismo directorio, y al no encontrarlo, genera un segfault. De esto podemos imaginar que el funcionamiento del programa es tomar un archivo `flag`, cifrarlo, y generar un archivo `flag.enc`, como el que tenemos.

## Decompilando el binario
Abrimos el binario con `ghidra` y lo decompilamos, obteniendo el siguiente código:
```C
undefined8 main(void)

{
  int iVar1;
  time_t tVar2;
  long in_FS_OFFSET;
  uint local_40;
  uint local_3c;
  long local_38;
  FILE *local_30;
  size_t local_28;
  void *local_20;
  FILE *local_18;
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  local_30 = fopen("flag","rb");
  fseek(local_30,0,2);
  local_28 = ftell(local_30);
  fseek(local_30,0,0);
  local_20 = malloc(local_28);
  fread(local_20,local_28,1,local_30);
  fclose(local_30);
  tVar2 = time((time_t *)0x0);
  local_40 = (uint)tVar2;
  srand(local_40);
  for (local_38 = 0; local_38 < (long)local_28; local_38 = local_38 + 1) {
    iVar1 = rand();
    *(byte *)((long)local_20 + local_38) = *(byte *)((long)local_20 + local_38) ^ (byte)iVar1;
    local_3c = rand();
    local_3c = local_3c & 7;
    *(byte *)((long)local_20 + local_38) =
         *(byte *)((long)local_20 + local_38) << (sbyte)local_3c |
         *(byte *)((long)local_20 + local_38) >> 8 - (sbyte)local_3c;
  }
  local_18 = fopen("flag.enc","wb");
  fwrite(&local_40,1,4,local_18);
  fwrite(local_20,1,local_28,local_18);
  fclose(local_18);
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return 0;
}
```
Los nombres de funciones y variables se pierden al compilar el binario, así que habrá que ir viendo qué función cumple cada variable y asignarle un nombre identificativo.

### Stack Canary
En primer lugar, vemos que a la variable `local_10` se le asigna el valor  que haya en la dirección en memoria `in_FS_OFFSET + 0x28`:
```c
local_10 = *(long *)(in_FS_OFFSET + 0x28);
```
Y luego, al final del código, se comprueba si su valor sigue igual:
```c
if (local_10 != *(long *)(in_FS_OFFSET + 0x28))
{ /* WARNING: Subroutine does not return */
__stack_chk_fail();
  }
```
Si por un stack overflow, el valor de `local_10` se ha sobreescrito, este ya no coincidirá con el valor almacenado en `(in_FS_OFFSET + 0x28)`, la comprobación dará `true` y se llamará a `__stack_chk_fail()`.

Con esto, podremos cambiarle el nombre a `local_10` a algo como `stackcanary`.
```c
stackcanary = *(long *)(in_FS_OFFSET + 0x28);
...
```

### Primeros archivos abiertos y asignación de memoria.
Tras asignar el valor al canary, se abre un archivo `flag` y se guarda su puntero en `local_30`, a la que llamaremos `flagSOURCE`. 
Después, se mueve el puntero al final del archivo en memoria `flag` y se comprueba la ubicación del puntero. Esto es una técnica usada para conocer el tamaño del archivo, cuyo valor luego se guarda en `local_28`, al que llamaremos `tamañoFLAG`.

```c
local_30 = fopen("flag","rb"); //local_30 = flagSOURCE
fseek(local_30,0,2); //Se mueve el puntero al final (2 = SEEK_END)
local_28 = ftell(local_30); //Se guarda la ubicación del puntero en local_28 o tamañoFLAG
fseek(local_30,0,0); //Se vuelve al inicio del archivo
```

Después vemos que se asigna una cantidad en memoria equivalente al tamaño del flag, se copia todo el archivo `flag` a memoria y se cierra el file descriptor de `flagSOURCE`.
```c
local_20 = malloc(tamañoFLAG); //Asignación de memoria de tamaño de tamañoFLAG.
fread(local_20,tamañoFLAG,1,flagSOURCE); //Se copia todo el flag a local_20
fclose(flagSOURCE);//Se cierra flagSOURCE
```

Podemos intuir que la copia de `flagSOURCE` a `local_20` se ha hecho con la idea de cifrar este archivo en memoria y luego guardarlo en el ya conocido `flag.enc`.

Dado que el programa va a trabajar con el archivo en `local_20`, podemos llamar a esta variable, p.ej, `Workspace`.

### Aleatorización
Tras copiar el flag original, vemos que se realiza lo siguiente:
Se guarda en `tVar2` el tiempo del sistema en el momento de ejecución, como entero con signo (`int`), el formato default que devuelve la función  `time()`. Luego, se cambia a entero sin signo (`uint`) y se guarda en `local_40`.
```c
tVar2 = time((time_t *)0x0); //le llamaremos tiempoDelSistemaINT
local_40 = (uint)tVar2; //le llamaremos tiempoDelSistemaUINT
```

Después se inicializa una semilla basada en `tiempoDelSistemaUINT` para la generación de números aleatorios:

```c
srand(tiempoDelSistemaUINT);
```

### Modificación del archivo
Aquí empieza un bucle, con la variable a iterar siendo `local_38`:
```c
for (local_38 = 0; local_38 < (long)tamañoFLAG; local_38 = local_38 + 1){...}
```
Para más claridad, le cambiamos el nombre a uno común, `i`:
```c
for (i = 0; i < (long)tamañoFLAG; i = i + 1){...}
```
Y aquí podemos entender que lo que va a hacerse es iterar sobre cada byte individual del archivo (recordar que `tamañoFLAG` es el tamaño del archivo en bytes.) y modificarlo.

El bucle, simplificado para más claridad (quitando castings como `(long)`):
```c
  for (i = 0; i < tamañoFLAG; i = i + 1) {
    iVar1 = rand();
    *(byte *)(Workspace + i) = *(byte *)(Workspace + i) ^ (byte)iVar1;
    local_3c = rand();
    local_3c = local_3c & 7;
    *(byte *)(Workspace + i) =
         *(byte *)(Workspace + i) << local_3c |
         *(byte *)(Workspace + i) >> 8 - local_3c;
  }
```
Aquí vemos que:
1. `iVar1` es un número aleatorio dado por `rand()`, por lo que le llamaremos `random`
2. `local_3c`  es un número aleatorio, del que luego se hace Bitwise AND con 7 (binario 111), por lo que su valor estará entre 0 y 7, así que le llamaremos `entre0y7`.
Así queda que:
```c
  for (i = 0; i < tamañoFLAG; i = i + 1) {
    random = rand();
    *(byte *)(Workspace + i) = *(byte *)(Workspace + i) ^ (byte)random;
    entre0y7 = rand();
    entre0y7 = local_3c & 7;
    *(byte *)(Workspace + i) =
         *(byte *)(Workspace + i) << entre0y7 |
         *(byte *)(Workspace + i) >> 8 - entre0y7;
}
```
`*(byte *)` significa que se está trabajando con bytes individuales, sin importar el tipo de cada dato individual dentro del propio archivo (`char`, `int`, `uint`, etc.), todo se trata como bytes sin más. Tras dejar esto claro y para centrarnos solo en la estructura, omitimos los `*(byte *)`:

```c
  for (i = 0; i < tamañoFLAG; i++) {
    random = rand();
    (Workspace + i) = (Workspace + i) ^ random; //XOR de random y el byte
    entre0y7 = rand(); 
    entre0y7 = local_3c & 7; //Ahora entre0y7 está entre 0b000 y 0b111
    (Workspace + i) = ((Workspace + i) << entre0y7) | ((Workspace + i) >> (8 - entre0y7));
}
```
Por cada byte vemos que:
1. Se hace un Bitwise XOR (`^`) del byte específico y `random`.
2. El byte se convierte en el resultado de una operación OR entre:
	- El byte desplazado a la izquierda `entre0y7` bits (los bits que salen fuera del byte se pierden)
	- El byte desplazado a la derecha (8-`entre0y7`) bits, es decir, el byte que contiene a los bits antes perdidos al desplazar a la izquierda `entre0y7` bits

### Escritura en archivo cifrado
Tras iterar sobre todo el archivo, se abre `local_18`, al que llamaremos `flagENCRYPTED`, que corresponde al archivo `flag.enc` en modo write, y se escribe sobre él:
- Los primeros 4 bytes de `tiempoDelSistemaUINT`, que, al tener `UINT` 4 bytes, es toda la variable.
- Todo el `Workspace`, es decir, el archivo cifrado
Y se cierra `flagENCRYPTED`
```c
  flagENCRYPTED = fopen("flag.enc","wb");
  fwrite(&tiempoDelSistemaUINT,1,4,flagENCRYPTED);
  fwrite(Workspace,1,tamañoFLAG,flagENCRYPTED);
  fclose(flagENCRYPTED);
```
Y así, finalmente, se tiene el archivo cifrado. 

Ahora, conociendo el algoritmo de cifrado, hay que hacer un programa encargado de descifrarlo.

## Descifrado
El objetivo de nuestro programa de descifrado es tomar los primeros 4 bytes del archivo cifrado, que corresponden a `tiempoDelSistemaUINT`, e inicializar el seed con esa variable. Luego, por cada byte de flag cifrado (ignorando los 4 primeros del seed):
- Llamar a `rand()` y guardar su resultado en un byte para el XOR (Pues el valor de `random` para el XOR es resultado de la primera llamada a `rand()` al cifrar, y si los cambiamos de orden se descifrará mal.)
- Llamar a `rand()` para sacar `entre0y7`, con la segunda llamada y el Bitwise AND 7.
Después, tendremos que modificar el byte cifrado siguiendo el camino inverso:
- Rotar a la **derecha** `entre0y7` bytes, sin perderlos (como al cifrar).
- Hacer el XOR con `random`
Y finalmente copiar ese byte específico al byte del resultado final.

En C, el código sería algo así (convendría hacer comprobaciones tras abrir archivos con fopen):
```c
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>

//Rota "byte" a la derecha el nº de bits "bits", pasando los bits que se perderían al otro lado
//Aquí rotamos a la derecha para invertir la rotación a la izquierda hecha por el algoritmo de cifrado
uint8_t rotarDCHAsinperder(uint8_t byte, unsigned bits){
    bits &= 7;
    if (bits == 0) return byte;
    return (uint8_t)((byte >> bits) | (byte << (8 - bits)));
}

int main(void){
    printf("Descifrando flag.enc...\n");
    unsigned seed; //Declarar seed
    FILE *encrypted = fopen("flag.enc", "rb"); //Abrir flag.enc a encrypted
    FILE *output = fopen("flag", "wb"); //Abrir lo que será el flag descifrado

    fread(&seed, 4, 1, encrypted); //Copiar a seed el seed guardado en el archivo
    srand(seed);

    int i; 
    while((i = fgetc(encrypted)) != EOF){ //fgetc(encrypted) lee el siguiente byte y se lo asigna a "i", luego se comparan "i" y "EOF"
        uint8_t byteCifrado = (uint8_t)i;

        uint8_t byteRandomXOR = (uint8_t)rand(); //primera llamada a rand() para el XOR
        uint8_t entre0y7 = ((uint8_t)rand()) & 7; //Segunda llamada a rand() para entre0y7

        uint8_t afterRot = rotarDCHAsinperder(byteCifrado, entre0y7);
        uint8_t byteOriginal = afterRot ^ byteRandomXOR; //El inverso del XOR es él mismo, así que repetimos el XOR Para obtener el original

        fputc(byteOriginal, output);
    }

    fclose(encrypted);
    fclose(output);
    return 0;
}
```

Y al ejecutarlo en el mismo directorio que `flag.enc`:
```bash
$ ./decrypt 
Descifrando flag.enc...
$ cat flag 
HTB{vRy_s1MplE_F1LE3nCryp0r}
```
