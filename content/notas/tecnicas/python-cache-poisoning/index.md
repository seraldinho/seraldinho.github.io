+++
title = 'Python Cache Poisoning'
draft = false
description = "Explicación del método de escalada de privilegios Python Cache Poisoning."
tags = ["Python", "Privesc", "Python Cache Poisoning"]
showToc = true
math = true
+++

El **python cache poisoning** es una vulnerabilidad (o técnica, según se mire) que permite escalar privilegios aprovechando la confianza de Python en archivos bytecode (`.pyc`) precompilados de scripts que ejecuta (normalmente con privilegios elevados).

Cuando Python ejecuta un script o importa una librería, hace lo siguiente:
- Si **no existe** una versión precompilada, compila el código fuente a bytecode, que normalmente se almacena en `__pycache__/NOMBRELIB.cpython-VERSION.pyc` o (menos común) bajo un prefijo `PYTHONPYCACHEPREFIX`.
- Si **existe** una versión precompilada, comprueba la cabecera (Magic Number que depende de la versión de Python, flags, metadatos como `timestamp`, `size` o `hash`), y, si al compararla con el `.py` original está bien, la ejecuta confiando en el `.pyc`.

> [!danger] Nota: Escalada de privilegios
> *Si un proceso con más privilegios ejecuta un .py o importa un módulo cuyo .pyc ha sido manipulado por un usuario menos privilegiado (y pasa la comprobación de cabecera), el payload corre con esos privilegios elevados.*

> [!tip] Nota: Windows vs Linux
> El mecanismo de `.pyc` y `__pycache__` es igual en Linux, Windows y macOS; solo cambian rutas y permisos. Lo más normal es ver esta vulnerabilidad en Linux, pero en Windows podría darse el caso.

## Caso Común: Permiso de escritura en `__pycache__` junto a script privilegiado
Aparece en [máquinas de HTB](writeups/browsed), consiste en lo siguiente:
- Script de python ejecutado como usuario privilegiado
- Permisos de escritura para todos en carpeta `__pycache__` 
- Script importa módulos locales

El exploit es el siguiente:
1. Se compila un exploit a `.pyc` con **la misma versión de Python** (si no fallará las comprobaciones)
```bash
$ python3.12 -m py_compile exploit.py
$ ls __pycache__
exploit.cpython-312.pyc
# Esto habrá que fusionar con el pyc original
```
2. Se lee el `.py` legítimo (del módulo) para robarle `mtime` y `size`, se escriben a la cabecera del `.pyc`
3. Se sobrescribe el `.pyc` bueno con el malicioso. Si no existe aún puede compilarse el `.py` legítimo con el mismo comando que con el exploit.
4. Cuando el script llame a `import modulo`, python verá un `.pyc` válido que pasará las comprobaciones, lo importará y ejecutará.

> Hay [un script](https://hardsoftsecurity.es/index.php/2026/01/19/python-cache-poisoning-privesc-linux/) que automatiza los pasos 2 y 3.
