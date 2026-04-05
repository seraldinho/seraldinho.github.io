+++
title = 'PSExec'
draft = false
description = "Explicación acerca de PSExec."
tags = ["PSExec", "Windows"]
showToc = true
math = true
+++

## Explicación e implementación básica
PsExec es una herramienta que permite ejecutar procesos en otros sistemas de la red sin tener que instalar nada manualmente. Hay varias implementaciones de la herramienta:

Por defecto, al ejecutar `psexec.exe`:
1. El programa copia el ejecutable `PSEXESVC.exe`, ubicado dentro del propio `psexec.exe`, al share `ADMIN$` del objetivo (A través de SMB, y que apunta a `C:\Windows`)
2. Mediante el API de servicios de Windows (SVCCTL RPC), contactando con él a través de MSRPC (p135) o de named pipes (Share `$IPC`, p139,445), crea un nuevo servicio con el ejecutable subido antes, con privilegios `SYSTEM`. 
3. Ese nuevo servicio crea varios named pipes nuevos en el share a través de los cuales tendrá lugar la comunicación. 
4. Al terminar, PsExec detiene el servicio y lo borra del sistema, pero *el binario de `C:\Windows` puede quedar ahí, lo que deja rastro de la conexión*.

Por esto último hay otras implementaciones, algunas que siguen el modelo original, pero reescritas en otros lenguajes, y otras que se ejecutan únicamente en memoria.

## Otras implementaciones
- **Impacket PsExec**: Versión en python, usa `RemComSvc` en lugar de `PSEXESVC.exe`
- **Impacket SMBExec**: Monta un servidor SMB en la máquina local en el que la víctima escribe el output de los comandos.
- **Impacket Atexec**: Crea una tarea inmediata en el task scheduler de Windows por cada comando, escribe su output al share, lo lee y lo borra. (Solo permite comandos individuales, no sesiones interactivas)
- **Netexec**: Herramienta de post-explotación que integra muchas de éstas.
- **Metasploit psexec_psh**: Implementación en memoria, sin tocar el disco, usando IEX de Powershell. Más info [aquí](https://www.mindpointgroup.com/blog/lateral-movement-with-psexec)

## Uso
Para usar cualquier implementación de PSExec generalmente necesitaremos **privilegios administrativos** (y en este caso también nos sirven hashes, así que podremos usar [PtH](../tecnicas/pass-the-hash.md))

Para `Impacket-PsExec`/`-AtExec`/`-SMBExec`:
```bash
impacket-psexec administrator:'PassWordABC123'@10.10.11.87
```

Para `netexec`/`crackmapexec`:
```bash
netexec smb 10.10.11.87 --local-auth -u Administrator -p 'PassWordABC123' --exec-method [smbexec/psexec/atexec] 
```
Por defecto `netexec` usa `atexec`, tenemos que especificar manualmente si queremos otra implementación.
