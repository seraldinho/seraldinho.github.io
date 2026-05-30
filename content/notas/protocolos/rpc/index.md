+++
title = 'RPC'
draft = false
description = "Explicación del funcionamiento del protocolo RPC"
tags = ["RPC", "Protocolo"]
showToc = true
showRelatedContent = false
+++

# Remote Procedure Call

## Resumen

RPC es un protocolo que permite que un programa ejecute código o funciones en otro dispositivo remoto y reciba su output (return) como si ese código se hubiese ejecutado localmente.

> Al hablar de los puertos hay que distinguir entre el **Endpoint Mapper** y los servicios RPC (Funciones) en sí, que pueden estar a la escucha en muchos puertos diferentes (TCP, UDP, HTTP, SMB...)

* Puerto **TCP/135** -> RCP Endpoint Mapper.
  * Indica a los clientes qué servicios RPC están disponibles y cómo acceder a ellos.

RPC puede usarse mediante diferentes métodos de comunicación entre cliente-servidor:

* `ncacn_ip_tcp`: **RPC sobre TCP**
  * La función RPC estará en escucha en un puerto alto aleatorio que puede consultarse al mapper.
* `ncacn_np`: **RPC sobre SMB**
  * Se accede al servicio RPC sobre SMB (TCP/445) en el share oculto `IPC$`.
* `ncacn_http`: **RPC sobre HTTP(S)**
  * Se accede al servicio RPC sobre HTTP(S), (TCP/80 & TCP/443)
* `ncacn_nb_tcp`: **RPC sobre NetBIOS**
  * Se accede al servicio RPC sobre NetBIOS (TCP/139).
* `ncacn_ip_udp`: **RPC sobre UDP**
  * Similar al caso de TCP pero sobre UDP.

## Enumeración del servidor

### `ncacn_np`: RPC sobre SMB

> En este caso, aunque el mapper no esté en escucha (TCP/135 cerrado), sigue siendo posible enumerar mientras SMB esté en escucha porque las funciones RPC de SMB están hardcodeadas y son conocidas, así que no hace falta preguntárselas.

```bash
# Conexión sin credenciales
rpcclient -U '' -N //10.10.11.87

# Conexión con credenciales
rpcclient -U 'DOMAIN\username%password' //10.10.11.87 
```

### `ncacn_ip_tcp`: RPC sobre TCP

> En máquinas Windows con el entorno de AD (o sus servicios comunes como SMB), suele poder verse en la enumeración una gran cantidad de puertos altos abiertos (alrededor del puerto 40000). Estos son los casos de RPC sobre TCP.

> Aunque puede ser útil enumerarlos en casos específicos, los servicios de estos puertos suelen no dar demasiada info y aquellos relevantes suelen ser también accesibles mediante SMB (`IPC$`).

```bash
#Enumeración de endpoints de funciones
#El puerto 135 es el puerto por defecto para el mapper
rpcdump.py 10.10.11.87 -p 135

# O su alternativa
impacket-rpcdump 10.10.11.87 -p 135
```
