+++
title = 'IKE'
draft = false
description = "Explicación del funcionamiento del protocolo IKE"
tags = ["IKE", "Protocolo"]
showToc = true
showRelatedContent = false
+++

# Internet Key Exchange

## Resumen

Protocolo encargado de establecer asociaciones de seguridad (SA) entre dos puntos, normalmente para el posterior uso de [IPSec](notas/protocolos/ipsec).

* Puertos: **UDP\500**, **UDP\4500**.

## IKEv1

> [!TIP]+ Nota: ISAKMP vs IKE
> Aunque se usan como sinónimos, técnicamente no son lo mismo. ISAKMP es el estándar teórico de cómo deben negociarse las cosas (sin especificar algoritmos) mientras que IKE es la implementación real de las reglas de ISAKMP, y, a efectos prácticos, la única que triunfó.

### Funcionamiento: Fases
La creación de una SA tiene lugar en varias fases:
* **Fase 1 (Autenticación de máquinas)**: La idea es crear un túnel seguro *solo para negociar lo demás*, no para datos. Este primer contrato de túnel se llama **IKE SA** o **ISAKMP SA**.

  * Para crearlo hay **2 Modos**: Principal (6 mensajes) o Agresivo (3 mensajes, desvela identidades, menos seguro).
  * Se negocian varias políticas (*HAGLE*): Hash, Autenticación, Grupo DH, Lifetime, Encryption. (P.ej: SHA256,PSK,19,8 horas,AES)
  * Se autentica cada máquina por medio de un certificado digital o un PSK

* **Fase 1.5 (XAuth)**: No obligatoria, pero se usa para, a través del túnel creado en el paso 1, verificar la identidad del usuario que intenta conectarse, solicitando usuario y contraseña, PIN, token u otro. Esta autenticación suele ir conectada por detrás a un server AD, RADIUS o similar.

* **Fase 2**: Ahora que tanto máquina como (opcionalmente) humano están verificados, se procede a crear el contrato final que usará IPSec para comunicarse.
  * Se negocian algoritmos de cifrado para ESP, que pueden ser diferentes a los de la fase 1.
  * Se generan (si [PFS](https://es.wikipedia.org/wiki/Perfect_forward_secrecy) está activado) nuevas claves DH para el cifrado que solo usará IPSec
  * Se crean **dos túneles unidireccionales**: `Firewall -> Usuario` y `Usuario -> Firewall`

### Transformaciones
Son sets de reglas que el cliente propone al servidor. Dado que el servidor suele tener una lista estricta de reglas permitidas, el cliente debe adivinar exactamente qué combinaciones (al menos una de ellas) acepta el servidor.

Una transformación consta de:
1. **Algoritmo de cifrado**: Para los datos enviados (DES, 3DES, AES256...)
2. **Algoritmo de hash**: Para comprobar la integridad de los datos (MD5,SHA1...)
3. **Método de Auth**: PSK, Certificados...
4. **Grupo de Diffie-Hellman**: Complejidad usada para generar las claves secretas compartidas. (1,2,14...)
5. **Tiempo de vida**: Cuánto dura la clave antes de tener que generar una nueva.

## IKEv2
> [!TIP]+ Nota: Vulnerabilidades
> La mayoría de vulnerabilidades y vectores de ataque aquí se dan sólamente para la versión 1 de IKE. El mundo se mueve hacia IKEv2, que corrige muchos de los errores de la versión anterior.

### Funcionamiento: Fases
Las fases de IKEv2 son más homogéneas que las de IKEv1, ya no hay modo agresivo o modo normal, todo pasa en 4 paquetes.

### Transformaciones
El concepto de transformaciones sigue siendo el mismo, lo único que cambia es la forma de encontrar una buena. 
- **IKEv1**: El cliente enviaba una transformación específica y el servidor decía si la aceptaba o no. Así hasta que aceptaba una.
- **IKEv2**: El cliente manda qué opciones acepta en general, y el servidor elige una transformación que él quiera que cumpla esas opciones.

## Conseguir y crackear PSK (IKEv1)
Si el servidor usa IKEv1 en modo agresivo, es posible conseguir el hash del PSK para luego crackearlo.
Para ello necesitamos el FQDN del usuario del que queremos conseguir el PSK (p.ej `ike@expressway.htb`):
```bash
ike-scan -M --aggressive -n <FQDN_usuario> --pskcrack=hash.txt 10.10.11.87
```
Esto guardará el hash en `hash.txt`. En teoría podemos usar `john` o `hashcat` para descifrarlo, pero generalmente suele funcionar mejor `psk-crack`:
```bash
psk-crack -d /usr/share/wordlists/rockyou.txt hash.txt
```
