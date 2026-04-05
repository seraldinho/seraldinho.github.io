+++
title = 'FTP'
draft = false
description = "Explicación del funcionamiento del protocolo FTP"
tags = ["FTP", "Protocolo"]
showToc = true
showRelatedContent = false
+++

# File Transfer Protocol

## Resumen

FTP es un protocolo de la capa de aplicación que permite transferir archivos entre dispositivos. Opera bajo una arquitectura cliente-servidor, usando dos canales distintos, uno de "control" (comandos) y otro para transferir los datos. Los datos se transmiten en **texto plano**, para cifrado se usa SFTP.

* Puertos **TCP 20, 21**.
  * 21 -> Control y autenticación. Aquí se listan los directorios, se solicitan archivos...
  * 20 -> Transferencia de datos

Tiene 2 modos de conexión:

* **Activo**: Cliente inicia conexión e indica a qué puerto de éste debe conectarse el servidor para la transferencia de datos. Si hay firewall en el lado del cliente puede rechazarse la solicitud del servidor.
* **Pasivo**: El cliente inicia la conexión, pero es el servidor el que le indica a qué puerto de éste debe intentar conectarse el cliente. Ahora, como es el cliente el que se conecta al servidor (para transmitir los datos), el firewall no bloqueará ninguna conexión del servidor.

## Enumeración

```bash
nmap -sT -Pn -n -sVC -p21 10.10.11.87
```

## Conexión

Para conectarse a FTP (Conociendo las credenciales de `user`):

```bash
ftp user@10.10.11.87
Password for user: ******
```

Para conectarse a FTP como usuario `anonymous` (si está permitido):

```bash
ftp anonymous@10.10.11.87
Password for anonymous: [Enter, dejando el campo vacío]
```

## Bruteforcing

Si se conoce un usuario (o una contraseña), es posible hacer bruteforcing o password spraying:

```bash
hydra -l [User] -P [Wordlist] ftp://[IP]
```

## FTP Bounce Attack
En el modo activo de FTP, el cliente puede usar el comando `PORT` para especificar a qué dirección y puerto debe conectarse el servidor para la transmisión de datos. Esto, si está mal configurado, puede permitir que el cliente (o atacante) especifique una dirección IP diferente a la suya propia y un puerto arbitrario al que el servidor tratará de conectarse, sirviendo como proxy para la solicitud y avisando de si el puerto está o no abierto.

Esto puede usarse para escanear puertos internos del servidor o de una subred, permitiendo además evadir firewalls e IDS ya que las solicitudes provienen del servidor FTP y no de nuestra IP.

Análisis bounce básico con nmap:
```bash
nmap -b admin:password@[FTP Server IP] [Bounced IP]
```
