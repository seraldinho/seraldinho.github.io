+++
title = 'TNS'
draft = false
description = "Explicación del funcionamiento del protocolo TNS"
tags = ["TNS", "Protocolo"]
showToc = true
showRelatedContent = false
+++

# Oracle Transparent Network Substrate 

## Resumen

TNS es un protocolo de la capa de aplicación que funciona como canal de comunicación a través del cual se accede a las bases de datos oracle (Oracle Database) desde las aplicaciones cliente. 

> Oracle TNS es análogo al protocolo MySQL, usado como vía de comunicación entre los clientes y el servidor MySQL.

TNS ofrece varios servicios como cifrado o resolución de nombres.

* Puerto **TCP 1521  (y a veces 1522-1529)**.
    - El puerto  1521 es el que se usa por defecto
    - En entornos empresariales pueden usarse más puertos con el fin de dar un balanceo de carga y mayor disponibilidad.

## Enumeración
Si tenemos credenciales, podemos conectarnos a la DB con [dbeaver](https://github.com/dbeaver/dbeaver) (GUI).

Tanto como herramienta de enumeración como de elevación de privilegios existe [**ODAT**](https://github.com/quentinhardy/odat) (Oracle Database Attacking Tool), con usos varios, según el autor en Github:

**Usage examples of ODAT**:
  1. You have an Oracle database listening remotely and want to find valid SIDs and credentials in order to connect to the database
  2. You have a valid Oracle account on a database and want to escalate your privileges to become DBA or SYSDBA
  3. You have a Oracle account and you want to execute system commands (e.g. reverse shell) in order to move forward on the operating system hosting the database*
    ...
