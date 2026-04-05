+++
title = 'DNS'
draft = false
description = "Explicación del funcionamiento del protocolo DNS"
tags = ["DNS", "Protocolo"]
showToc = true
showRelatedContent = false
+++

# Domain Name System

## Resumen

El protocolo DNS es un sistema distribuido que traduce nombres de dominio (p.ej `google.com`) a direcciones IP (`142.250.201.14`).

Por defecto, el protocolo DNS **no tiene cifrado**, por lo que suelen usarse **DoT** (DNS over TLS) y **DoH** (DNS over HTTPS)

* Puertos **UDP/53, TCP/53**.
  * UDP/53 -> Usado por defecto por el protocolo para resoluciones DNS, más rápido que TCP.
  * TCP/53 -> Si el tamaño de los mensajes excede los 512 bytes (el máximo de un datagrama UDP), se usará TCP en lugar de UDP.


## Tipos de registros
Los registros DNS son archivos de texto que dan info sobre un host o dominio (como IPs actuales y otros datos), hay varios tipos:
- **A**: El más común, mapea un nombre de dominio a una IPv4
- **AAAA**: Mapea un nombre de dominio a una IPv6
- **CNAME**: Sirve como alias, para cuando un dominio o subdominio es un alias de otro dominio. (Siempre apunta a otro dominio, no a IPs)
- **MX**: Apunta a nombres de servidores de correo de un dominio
- **TXT**: Info en texto relacionada con dominios y subdominios.
- **NS**: Apunta a los servidores DNS que proporcionan los registros DNS para el dominio específico.
- **SOA**: Almacena info sobre la zona DNS, como detalles sobre el servidor primario y la administración.

## Enumeración
### Enumeración Activa
Con `dig` podemos realizar consultas DNS a nuestros objetivos para conseguir una imagen general de la infraestructura:
```shell
dig ns google.com

; <<>> DiG 9.18.41 <<>> ns google.com
...[SNIP]...

;; ANSWER SECTION:
google.com.		7191	IN	NS	ns4.google.com.
google.com.		7191	IN	NS	ns2.google.com.
google.com.		7191	IN	NS	ns1.google.com.
google.com.		7191	IN	NS	ns3.google.com.

;; Query time: 0 msec
;; SERVER: 127.0.0.53#53(127.0.0.53) (UDP)
;; WHEN: Sat Nov 00 00:00:00 CET 2025
;; MSG SIZE  rcvd: 111
```
Esto da bastante información, podemos usar `+short` para filtrar solo lo relevante:
```shell
dig ns google.com +short
ns4.google.com.
ns3.google.com.
ns1.google.com.
ns2.google.com.
```
Y cambiando `ns` por cualquier otro tipo de registro (`SOA`, `A`, `CNAME`, `MX`...) podemos solicitar cualquier otro tipo de información:
```shell
$ dig mx google.com +short
10 smtp.google.com
```
### AXFR
AXFR (Authoritative Transfer) es un mecanismo de transferencia de zona DNS usado para replicar bases de datos DNS entre servidores. 

Normalmente sirve para sincronizar info DNS entre servidores, pero si está mal configurado podemos solicitar un AXFR nosotros mismos, obteniendo un dump de la info dns de un servidor.

```shell
dig axfr google.com +short
; Transfer failed.
# No es raro que falle en este caso, google está bien protegido.
```

### Fuerza Bruta
Podemos usar herramientas como `fierce` o `gobuster`:
```shell
$ ./fierce.py --domain google.com
NS: ns3.google.com. ns2.google.com. ns4.google.com. ns1.google.com.
SOA: ns1.google.com. (216.239.32.10)
Zone: failure
Wildcard: failure
Found: 1.google.com. (172.217.19.46)
Nearby:
{'172.217.19.41': 'mrs08s03-in-f9.1e100.net.',
 '172.217.19.42': 'mrs08s03-in-f10.1e100.net.',
 '172.217.19.43': 'ham02s11-in-f43.1e100.net.',
[SNIP]...
```

```shell
$ gobuster dns -d google.com -w /usr/share/wordlists/n0kovo_subdomains.txt 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Domain:     google.com
[+] Threads:    10
[+] Timeout:    1s
[+] Wordlist:   /usr/share/wordlists/n0kovo_subdomains.txt
===============================================================
Starting gobuster in DNS enumeration mode
===============================================================
Found: mail.google.com
Found: www.google.com
Found: m.google.com
Found: image.google.com
Found: api.google.com
Found: images.google.com
...
```

### Hostname Reverse Lookup
Para entornos AD, es muy probable que necesitemos un hostname y no podamos autenticarnos (p.ej por Kerberos) simplemente contra una IP. Para ello, si estamos ante un DC con el puerto DNS (53) abierto, podemos probar a hacer un DNS reverse lookup:
```bash
nslookup
> server 10.10.11.87
> 127.0.0.1
> 127.0.0.2
> 10.10.11.87
# Probamos a ver "cómo se hace llamar" el servidor DNS a sí mismo para conseguir un hostname.
```
