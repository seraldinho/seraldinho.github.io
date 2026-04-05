+++
title = 'IMAP'
draft = false
description = "Explicación del funcionamiento del protocolo IMAP"
tags = ["IMAP", "Protocolo"]
showToc = true
showRelatedContent = false
+++

# Internet Message Access Protocol

## Resumen

IMAP es un _protocolo de entrada_ (_pull_) que permite que los clientes accedan al correo electrónico disponible en sus cuentas una vez este ha sido entregado por el servidor SMTP.

Las acciones realizadas sobre el correo (Archivar, borrar, etc.) en IMAP se realizan en el servidor y se sincronizan en todos los clientes.

* Puertos **TCP 143, 993**.
  * 143 -> Puerto por defecto para IMAP
  * 993 -> Puerto para IMAP cifrado (IMAPS)

## Conexión CLI

Podemos conectarnos con curl o netcat:

```shell
$ curl -k imap(s)://10.10.11.87 --user usuario@contraseña
$ ncat 10.10.11.87 143
```

Tras conectarnos:

* `LOGIN usuario contraseña` ⇒ Inicio sesión
* `SELECT carpeta` ⇒ Seleccionar carpeta
* `FETCH` ⇒ Sacar email específico
* `SEARCH` ⇒ Buscar
* `EXAMINE` ⇒ Ver carpetas
* `LOGOUT` ⇒ Salir

Si la conexión es cifrada (IMAPS, p993), podemos usar openssl:

```shell
$ openssl s_client -connect 10.10.11.87:993
```

## Conexión GUI

Para conectarnos a un servidor IMAPS desde un entorno gráfico, podemos usar **Thunderbird** o **Gnome Evolution** si conocemos unas credenciales válidas.
