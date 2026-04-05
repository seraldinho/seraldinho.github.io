+++
title = 'IPSec'
draft = false
description = "Explicación del funcionamiento del protocolo IPSec"
tags = ["IPSec", "Protocolo"]
showToc = true
showRelatedContent = false
+++

# Internet Protocol security

## Resumen

IPSec es un conjunto de protocolos encargados de proteger comunicaciones sobre IP, autenticando y cifrando cada paquete IP de un flujo de datos. Trabaja en la capa de red. A diferencia de SSL/TLS, que cifran el contenido pero no el destinatario, IPSec cifra el paquete de red entero.

Dado que IPSec no es un protocolo específico sino un conjunto, conviene detallar algunos de lo que lo forman.

> [!tip]+ Nota: Comparación con HTTP/QUIC 
> Otros protocolos como HTTP/QUIC tienen integrado directamente el cifrado con TLS (actualmente TLS1.3), que se encarga de negociar las claves DH y cifrar los paquetes con AES, pero esto es porque tales protocolos **funcionan en las capas 4-7 (OSI)** y están hechos para proteger **procesos específicos**. Por otro lado, IP e IPSec (Capa 3) son stateless, no tienen el concepto de conexiones/handshakes, IPSec sólamente puede coger un paquete IP entero (o casi) y cifrarlo/autenticarlo, pero para llegar a un acuerdo acerca de cómo, se requiere otro protocolo, [IKE](notas/protocolos/ike).

## Authentication Header (AH) -> Integridad
Firma criptográficamente todo el paquete IP, incluyendo cabeceras, por lo que se encarga de asegurar que el paquete proviene del destinatario y que no ha sido manipulado, pero **NO cifra el payload**.
- Añade un encabezado con datos de autenticación. Si el hash del payload no coincide con el encabezado, se sabe que el paquete ha sido manipulado.

> [!warning] Nota: Incompatibilidad con NAT
> *Dado que AH firma **todo el paquete**, incluyendo cabeceras IP originales, si un dispositivo está tras un NAT, se manipularán los headers de los paquetes que vayan hacia él, por lo que se detectará una manipulación, aunque no maliciosa, del paquete y el router lo descartará. Por esto, AH no es compatible con NAT (y no suele usarse, prefiriéndose ESP).*

## Encapsulated Security Payload (ESP) -> Cifrado
IPSec tiene dos modos de funcionamiento:
- **Modo túnel**: Normalmente de `PC <-> Firewall` o de `Router <-> Router`, se cifra todo el paquete, incluyendo cabecera IP, y se le pone otra nueva apuntando al Firewall/Router.
- **Modo Transporte**: De `PC <-> PC`, sin pasarelas VPN intermedias. IPSec coge el paquete, deja la cabecera IP intacta y cifra solo el payload.

## Internet Key Exchange (IKE) -> Negociación
Negocia y establece Asociaciones de Seguridad (SA) para IPSec. Las SA son contratos matemáticos que representan el conjunto de reglas y parámetros que ambos extremos han acordado para poder entenderse y proteger el tráfico. Para más info mirar [IKE](notas/protocolos/ike)
