+++
title = 'RDP'
draft = false
description = "Comandos y funcionamiento del protocolo RDP"
tags = ["RDP", "Protocolo"]
showToc = true
showRelatedContent = false
+++

# Remote Desktop Protocol

## Resumen

Protocolo propietario de Microsoft que permite una GUI para conectarse a otro dispositivo por la red.

* Puerto **TCP 3389**.

## Conexión
```shell
xfreerdp /u:'user' /p:'password' /v:10.10.11.87
```
