+++
title = 'TFTP'
draft = false
description = "Explicación del funcionamiento del protocolo TFTP"
tags = ["TFTP", "Protocolo"]
showToc = true
showRelatedContent = false
+++

# Trivial File Transfer Protocol

## Resumen

TFTP es un protocolo que permite transferir archivos entre dispositivos. Es una versión simple de FTP.

No tiene **ni autenticación ni cifrado**, por lo que es potencialmente inseguro. Tener acceso a un puerto TFTP abierto indica un potencial vector de entrada, pues está hecho para ser accesible sólamente dentro de LANs.

* Puerto **UDP 69**.
  * Sólo se usa como punto inicial de conexión. La transferencia real de datos se realiza en un puerto aleatorio alto (1024)

## Enumeración

```bash
nmap -sU -Pn -n -sVC -p69 10.10.11.87
```

## Bruteforcing

Dado que TFTP no permite listar los archivos del directorio compartido, es necesario saber qué archivos existen para poder descargarlos. Para esto podemos usar el módulo `tftpbrute` de Metasploit:

```msfconsole
use auxiliary/scanner/tftp/tftpbrute
```
