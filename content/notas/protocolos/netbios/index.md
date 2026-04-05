+++
title = 'NetBIOS'
draft = false
description = "Explicación del funcionamiento del protocolo NetBIOS"
tags = ["NetBIOS", "Protocolo"]
showToc = true
showRelatedContent = false
+++

# Network Basic Input/Output System

## Resumen

NetBIOS es una interfaz que permite que los dispositivos se comuniquen entre sí en una LAN. Se encarga de mantener conexiones entre ellos.

> [!tip] Nota: Obsolescencia
> Gran parte de esto no es relevante porque, si NetBIOS está activo alguna vez en algún servidor, es probablemente porque SMB (sobre TCP/IP, puerto 445) también lo está y se busca mantener compatibilidad con equipos legacy de la red, pero en sí NetBIOS ya prácticamente no se usa.

Dicho esto, hay que destacar que NetBIOS es **completamente independiente de SMB**.

Usa principalmente los siguientes puertos:

* UDP/137 -> Servicio de nombres: Resuelve nombres NetBIOS en IPs
* UDP/138 -> Servicio de datagramas: Permite comms. sin conexión pero con más funcionalidades que UDP sin más.
* **TCP/139** -> Servicio de sesión: Conexiones punto a punto, SMBv1 se ejecuta sobre este.
