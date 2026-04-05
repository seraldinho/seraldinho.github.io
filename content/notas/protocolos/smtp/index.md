+++
title = 'SMTP'
draft = false
description = "Explicación del funcionamiento del protocolo SMTP"
tags = ["SMTP", "Protocolo"]
showToc = true
showRelatedContent = false
+++

# Simple Mail Transfer Protocol

## Resumen

SMTP es un *protocolo de salida* (*push*) de la capa de aplicación que se encarga de entregar los correos electrónicos a su destino. Al enviar un email, el cliente de correo del usuario se conecta con un servidor SMTP que lo transmite hasta el servidor SMTP destinatario, que lo transmite al servidor IMAP/POP3 correspondiente para que el cliente lo recoja cuando quiera.

El diagrama es algo así (para un email desde una cuenta de Gmail a una de Protonmail):
```
1. Cliente (p.ej Thunderbird o Gmail)
2. Servidor SMTP 1 (p.ej servidor de Gmail)
3. Servidores SMTP intermedios (SMTP Relays)
4. Servidor SMTP destino (P.ej SMTP de Protonmail)
5. Servidor IMAP/POP3 (Email almacenado).
```
Cuando el receptor quiere mirar su correo, accede al servidor IMAP/POP3 (O app web de correo) y recoge su correo entrante.

* Puertos **TCP 25, TCP 587**.
  * TCP/25 -> Puerto SMTP por defecto, sin cifrar.
  * TCP/587 -> Puerto SMTP cifrado (SMTPS)

## Enumeración
Un servicio SMTP expuesto nos permite enumerar principalmente su versión y nombres de usuario válidos del SO.

Hay varios métodos para enumerar usuarios. Podemos hacerlo manualmente mediante netcat:
```shell
netcat -vn 10.10.11.87 25
Ncat: Connected to 10.10.11.87:25.
220 #vulnmachine ESMTP Postfix (Ubuntu)

vrfy admin@vulnserver.htb
250 2.0.0 admin@vulnserver.htb
# Recibir códigos 250,251 y 252 significa que existe el usuario.
# Recibir un código 550 significa que no existe.
```

Esto puede automatizarse con varias herramientas:
- Metasploit: `auxiliary/scanner/smtp/smtp_enum`
- Nmap: `sudo nmap -p 25 --script=smtp-enum-users 10.10.11.87`
