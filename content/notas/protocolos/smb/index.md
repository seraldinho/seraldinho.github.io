+++
title = 'SMB'
draft = false
description = "Explicación del funcionamiento del protocolo SMB"
tags = ["SMB", "Protocolo"]
showToc = true
showRelatedContent = false
+++

# Server Message Block

## Resumen

SMB es un protocolo que permite el acceso compartido a recursos como archivos, directorios o impresoras (entre otros). También sirve como túnel para comunicación entre procesos (IPC).

Los puertos sobre los que se expone dependen del protocolo sobre el que se ejecuta, pueden aparecer ambos activos a la vez:

* **TCP/139** -> SMB ejecutado sobre NetBIOS sobre TCP/IP. Obsoleto pero se suele mantener abierto para tener retrocompatibilidad con sistemas legacy.
* **TCP/445** -> SMB ejecutado sobre TCP/IP directamente. El método "moderno".

## Enumeración

### Linux

Para listar los shares disponibles en una determinada IP:

```bash
smbclient -N -L //10.10.11.87
# -N indica login sin proporcionar credenciales. Puede (y es probable) que se dé el caso en que esto no esté permitido.
# -L sirve para listar los shares.

#Output:
    Sharename       Type      Comment
    ---------       ----      -------
    print$          Disk      
    Documentos      Disk      Documentos Secretos
    IPC$            IPC       
```

Otros programas para enumerar que pueden ser útiles:

* `SMBMap`
* `enum4linux`
  * Usa varias herramientas manuales (rpcclient, smbclient, etc.) para automatizar la enumeración del servidor SMB.

Para enumerar SMB mediante MSRPC, podemos acceder a RPC a través de SMB usando `rpcclient`:

```shell
rpcclient -U "" 10.10.11.87
```

> RPCClient usa MSRPC sobre SMB, la comunicación se establece mediante los named pipes (`IPC$`) de SMB, no sobre el puerto 135 del endpoint mapper. Para más info mirar `rpc.md`.

Si se permite la conexión, podremos consultar más información.

```shell
rpcclient $> help
srvinfo
enumdomains
querydominfo
enumdomusers
...
```

Para determinados queries necesitaremos los RID del usuario específico, podemos sacarlos por fuerza bruta con `samrdump`:

```bash
samrdump.py 10.10.11.87
```

### Windows

Para listar archivos en un share desde cmd/powershell:

```powershell
dir \\10.10.11.87\Documentos
Get-ChildItem \\10.10.11.87\Documentos #Solo Powershell
```

## Conexión y montaje

### Linux

Para conectarse a determinado share (desde Linux):

```bash
smbclient //10.10.11.87/Documentos
```

Para montar un share como partición:

```bash
sudo mount -t cifs -o username="admin",password="iLoveYou" //10.10.11.87/Documentos /mnt/mountpoint
```

### Windows

Para conectarse a determinado share (desde Windows, GUI):

> _Pulsar \[WIN]+\[R], en la ventana "ejecutar" que se abra:_

```
\\10.10.11.81\Documentos
```

## Bruteforcing

Si no podemos enumerar archivos sin autenticarnos y necesitamos unas credenciales de las que no disponemos, podemos usar `netexec` para hacer bruteforce o password spraying:

```bash
netexec smb 10.10.11.87 -u /usr/share/wordlists/users.txt -p 'iLoveYou' --local-auth
```

> _Es **necesario** poner `--local-auth` si quieremos autenticarnos con el SAM del servidor si no hay AD DC disponible, netexec no recurre a la autenticación contra el SAM automáticamente si no se detecta un Domain Controller._

## Pass-the-Hash (PtH)

> Mirar [Pass-the-Hash](/notas/tecnicas/pth)

## Movimiento lateral
SMB se usa como canal de comunicación para la subida de archivos y la recepción y envío de comandos y respuestas al usar PsExec. 
Más info en [movimiento lateral](notas/tecnicas/psexec)
