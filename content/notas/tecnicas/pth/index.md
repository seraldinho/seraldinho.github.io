+++
title = 'Pass-The-Hash'
draft = false
description = "Explicación del concepto de Pass-The-Hash."
tags = ["Windows", "PtH"]
showToc = true
math = true
+++

## Introducción
Tanto en entornos AD y sistemas Windows independientes es posible realizar un ataque PtH, que consiste en usar el hash NTLM de un usuario ya autenticado y usarlo para autenticarnos en otros servicios, sin necesidad de una contraseña conocida.

Esto es posible debido al funcionamiento de la autenticación NTLM:
> En sistemas Windows sin AD, las credenciales del usuario se almacenan en el SAM como hashes NT, mientras que en entornos AD se almacenan (también como hashes) en los controladores de dominio (DC).

Cuando un usuario solicita iniciar sesión en una cuenta:
1. El servidor envía un challenge (número aleatorio de 16 bytes) al cliente.
2. El cliente calcula el hash NT de la contraseña introducida por el usuario y encripta el challenge usando el hash como clave de cifrado.
3. El servidor hace lo mismo con la credencial almacenada del usuario.
4. Si las respuestas coinciden, se concede acceso.

Esto se hizo con la idea de ni enviar contraseñas por la red ni almacenarlas en texto plano, pero si un atacante ya dispone del hash de las credenciales (p.ej, al extraerlas de memoria), parte del paso 2, con el hash de la contraseña real ya calculado, lo que implica que *es suficiente con solicitar al servidor un challenge y cifrarlo con el hash para iniciar sesión sin tener la contraseña.*

## Técnica
La aplicación de la técnica puede hacerse con herramientas como `netexec`, si ya tenemos el hash:
```bash
netexec smb 10.10.11.87 -u Administrator -H <hash>
```
