+++
title = 'POP3'
draft = false
description = "Explicación del funcionamiento del protocolo POP3"
tags = ["POP3", "Protocolo"]
showToc = true
showRelatedContent = false
+++

# Post Office Protocol

## Resumen

POP3 es un *protocolo de entrada* (*pull*) que permite que los clientes accedan al correo electrónico disponible en sus cuentas una vez este ha sido entregado por el servidor SMTP. 

Las acciones realizadas sobre el correo (Archivar, borrar, etc.) en POP3 se realizan en el cliente. Éste descarga el correo localmente, normalmente borrándolo del servidor, y todos los cambios se hacen localmente, no se sincronizan.

* Puertos **TCP 110, 995**.
  * 110 -> Puerto por defecto para POP3
  * 995 -> Puerto para POP3 cifrado (POP3S)

## Conexión
La enumeración es prácticamente igual que para IMAP(S), solo cambian los comandos para inicio de sesión y para ver el correo.

Podemos conectarnos con curl o netcat:
```shell
$ curl -k pop3(s)://10.10.11.87 --user usuario@contraseña
$ ncat 10.10.11.87 110
```
Tras conectarnos:
- `USER nombreusuario` ⇒ Usuario
- `PASS contraseña` ⇒ Contraseña
- `LIST` ⇒ Listar Correo
- `RETR` ⇒ Ver X Mensaje
- `QUIT` ⇒ Salir

Si la conexión es cifrada (POP3, p995), podemos usar openssl:
```shell
$ openssl s_client -connect 10.10.11.87:995
```
