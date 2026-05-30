+++
title = 'MySQL'
draft = false
description = "Explicación del funcionamiento del protocolo MySQL"
tags = ["MySQL", "Protocolo"]
showToc = true
showRelatedContent = false
+++

# MySQL

## Resumen

Base de datos y protocolo de Oracle, closed source. (Su alternativa equivalente es MariaDB, que sí es de código abierto)

* Puerto **TCP 3306**.

## Conexión
```shell
mysql 10.10.11.87 -u 'user' -p'password'
# No hay espacio entre -p y la contraseña
```

## Lectura & Escritura
Si ya tenemos acceso a la consola de MySQL y queremos manipular algún archivo, primero debemos comprobar que tenemos permisos para ello, que se controlan con la variable `secure_file_priv`:
```sql
SHOW variables LIKE 'secure_file_priv`;
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| secure_file_priv |       |
+------------------+-------+
```
El valor de la variable puede significar varias cosas:
- `"(vacío)"`: Se permite R/W (está sin configurar).
- `Nombre de directorio`: Se permite R/W solo en ese directorio (y sus archivos)
- `NULL`: No se permite R/W

Para escribir en un archivo:
```sql
SELECT "contenido del archivo" INTO OUTFILE 'Archivo_destino.txt';
```

Para leer un archivo:
```sql
select LOAD_FILE("/etc/passwd");
```
