+++
title = 'Python Library Hijacking'
draft = false
description = "Explicación del método de escalada de privilegios Python Library Hijacking."
tags = ["Python", "Privesc", "Python Library Hijacking"]
showToc = true
math = true
+++

El Python Library Hijacking es una técnica de escalada de privilegios que consiste en aprovechar la forma en la que Python busca las librerías para cargar código malicioso en lugar de las librerías originales.

## Funcionamiento
Python busca módulos en un órden específico definido por `sys.path`, que puede verse de la siguiente manera:
```bash
$ python3 -c 'import sys; print(sys.path)'
['', '/usr/lib/python312.zip', '/usr/lib/python3.12', '/usr/lib/python3.12/lib-dynload', '/usr/local/lib/python3.12/dist-packages', '/usr/lib/python3/dist-packages']
```
Normalmente, el orden es el siguiente:
1. El directorio que contiene el script a ejecutar (definido por `''`)
2. Directorios especificados en la variable env `PYTHONPATH` (No aparece)
3. Directorios estándar del sistema

Para explotarlo, un atacante necesitaría uno de estos:
- Permiso de escritura en el directorio del script
- Poder pasar la variable `PYTHONPATH` al script al ejecutarlo con sudo (normalmente se limpian las variables de entorno para evitar esto precisamente)
- Permiso de escritura para una de las librerías estándar (sería raro)

Si un usuario tiene alguno de estos y **conoce alguna librería que carga el script**, p.ej `random`, puede crear un archivo `random.py` en el directorio al que tiene acceso y, cuando se ejecute el script, primero cargará el `random.py` malicioso que, p.ej, dará un shell como un usuario privilegiado.
