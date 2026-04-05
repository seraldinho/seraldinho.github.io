+++
title = 'Port Forwarding'
draft = false
description = "Explicación del concepto de Port Forwarding y sus métodos."
tags = ["Infrastructure"]
showToc = true
math = true
+++

## Local Port Forwarding con SSH

> Esto **no** requiere privilegios administrativos en la cuenta del servidor SSH. Para hacer port forwarding local:

```bash
ssh -L 4321:localhost:3306 usuario@10.10.11.87
```

> Con esto estamos mapeando el socket `localhost:3306` abierto en la máquina con dirección (accesible) `10.10.11.87` en _nuestro_ socket `localhost:4321` (El `localhost` en nuestro dispositivo va implícito al usar `-L`) mediante SSH, conectándonos como `usuario`.

Si quisiésemos especificar explícitamente la IP de nuestra máquina a la que se mapea la remota, la sintaxis completa sería:

```bash
ssh -L [IP_Local]:[Puerto_Local]:[IP_Destino]:[Puerto_Destino] usuario@10.10.11.87
```

Podemos hacer port forwarding de varios servicios a la vez:

```bash
ssh -L 4321:localhost:3306 8000:localhost:80 usuario@10.10.11.87
```

> En la mayoría de implementaciones de OpenSSH, la capacidad de tunelizar dinámicamente con `-L` está activada por defecto.

## Dynamic Port Forwarding con SSH & SOCKS

> Esto **no** requiere privilegios administrativos en la cuenta del servidor SSH. El `-L` en LPF es una relación 1 a 1, un puerto nuestro = un puerto e IP de destino específicos. Si queremos escanear una subred entera accesible desde la víctima, tendríamos que abrir muchos túneles independientes manualmente.

Para solucionar esto está el Dynamic Port Forwarding, que abre un puerto local como proxy y convierte a la máquina víctima (por SSH) en un router por el que pasa el proxy.

```bash
ssh -D 9050 usuario@10.10.11.87
```

Desde ahí, todo lo que hagas a través de `localhost:9050` se hará en realidad desde la máquina con IP `10.10.11.87`.

> En la mayoría de implementaciones de OpenSSH, la capacidad de tunelizar dinámicamente con `-D` está activada por defecto.

### Herramientas a través de proxies SSH & SOCKS

> Aquí hay que tener cuidado si nos preocupa además ocultar nuestra IP, pues según que herramienta usemos, puede que se filtre. `ping` usa ICMP (capa 3), que ignora por completo los proxies (y o no funcionará o filtrará nuestra IP directamente); nmap con `-sT` funciona, pero los escaneos con ping o `-sS` no funcionan y algunos scripts tampoco lo harán.

Podemos usar proxychains para ejecutar herramientas a través de proxies, como pueden ser `tor` (127.0.0.1:9050) o incluso un servidor SSH que hayamos comprometido (con `-D`). Proxychains debe estar configurado con el proxy que queramos (Normalmente en `/etc/proxychains.conf`).

#### Nmap

Para escanear una IP específica a través del proxy (debe ser escaneo -sT con ICMP echo desactivado):

```bash
proxychains nmap -Pn -sT -v 10.10.12.88
```

Para escanear una subred entera en busca de algunos dispositivos vivos podemos usar:

```bash
proxychains nmap -sT -Pn -n -v --top-ports=10 10.10.12.0/24
```

Esto mira los 10 puertos más comunes (se pueden aumentar) en todas las direcciones IP de la subred `10.10.11.0/24` usando el TCP Full scan. `-Pn` desactiva ICMP pings, `-n` desactiva resoluciones DNS.

#### RDP

Si tenemos un proxy por SSH configurado en proxychains, también podemos conectarnos a un servidor de una red interna si tiene en escucha, p.ej, RDP.

```bash
xfreerdp /v:10.10.12.88 /u:user52 /p:p@sSw0rd
```

## Reverse/Remote Port Forwarding con SSH

Si queremos conseguir una reverse shell desde una máquina en una subred interna, no podremos ejecutar el payload sin más, dado que la máquina interna no tiene una ruta definida para llegar hasta nuestra IP. Es por esto que necesitamos configurar un port forwarding _remoto_ (_`-R`_).

```bash
ssh -R <IP_Interna_Pivote>:<Puerto_Pivote>:<IP_Atacante_Local>:<Puerto_Local>
```

P.ej, si tenemos una máquina pivote con ip pública `10.10.11.87` e ip en una subred `10.10.12.87` en `10.10.12.0/24`, y una máquina objetivo con ip `10.10.12.88`:

```bash
# Si nuestra IP (atacante) es 80.81.50.27
ssh -R 10.10.12.87:8080:127.0.0.1:4321 usuario@10.10.11.87

# También valdrían 80.81.50.27 o 0.0.0.0 (localhost) en lugar de 127.0.0.1, 
# solo habría que cambiar la interfaz en la que escuchamos con el listener.
```

Este comando creará un túnel bidireccional que hará que los datos vayan en dos direcciones:

* `NuestraMáquina:4321` <-> `10.10.12.87:8080`

Todo lo que llegue a 10.10.12.87 irá a parar a nuestra máquina (127.0.0.1 como hemos especificado), al puerto 4321, y viceversa.

&#x20;

![](remoteportforward.png)

## Socat (Alternativa a SSH, sin credenciales)
Socat es una herramienta que sirve para crear vías de comunicación bidireccionales entre 2 canales de red independientes **sin necesitar usar túneles SSH**. Necesitamos acceso a la víctima, pero no necesariamente sus credenciales (que sí necesitamos para hacerlo por ssh).

P.ej, usar el siguiente comando pondrá en escucha el puerto `8080` de todas las interfaces (`0.0.0.0`) de la máquina víctima y redirigirá todo el tráfico que llegue al puerto `80` de la IP `10.10.12.90`:
```bash
victima@10.10.11.87:~$ socat TCP4-LISTEN:8080,fork TCP4:10.10.12.90:80
```
- `TCP4-LISTEN:8080` pone en escucha `0.0.0.0:8080`
  - `fork` hace que con cada conexión se cree un proceso hijo que la gestione (lo que permite manejar varias a la vez)
- `TCP4:10.10.12.90:80` es el socket al que se redirigen los datos

## Plink (PuTTY, Windows LotL)
Plink (PuTTY Link) es una herramienta de Windows que permite conectarse por SSH a otros dispositivos. Hasta 2018, Windows no tenía un cliente ssh nativo, así que los administradores se tenían que descargar uno, y, en su momento, el de preferencia era PuTTY.

Podemos crear un túnel similar al de SSH (`-D`) de la siguiente manera:
```bash
plink -ssh -D 9050 user@10.10.11.87
```
Esto creará un túnel en `127.0.0.1:9050` hasta `10.10.11.87`, que actuará como proxy.

## Sshuttle (Auto-routing a través de pivote por SSH)
Sshuttle es una herramienta que automatiza la configuración de iptables y de rutas para que la máquina atacante pueda acceder a otra máquina en una subred interna a través de un pivote de forma "directa" (o al menos transparente).

P.ej, para poder acceder a la subred `172.16.15.0/23`, a la que solo es posible acceder desde `10.10.11.87`, directamente, necesitaríamos usar ssh o similares, pero con `sshuttle` podemos hacer lo siguiente:
```bash
sudo sshuttle -r user@10.10.11.87 172.16.15.0/24 -v 
```
E inmediatamente intentar acceder con cualquier otra herramienta a una IP en la subred de forma directa, sin necesitar usar `proxychains`.
