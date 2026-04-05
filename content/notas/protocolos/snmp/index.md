+++
title = 'SNMP'
draft = false
description = "Explicación del funcionamiento del protocolo SNMP"
tags = ["SNMP", "Protocolo"]
showToc = true
showRelatedContent = false
+++

# Simple Network Management Protocol

## Resumen

SNMP es un protocolo que permite la gestión remota de dispositivos de red, generalmente IoT, routers, switches y demás.

Para garantizar una compatibilidad entre todos los fabricantes, se usa un formato llamado MIB para guardar los datos (Management information Base). Esta MIB representa la "estructura" de los datos que pueden solicitarse al servidor. En ella, todo son OIDs (Object Identifiers), que a su vez pueden ser nodos o referencias a otros nodos (de forma análoga a varios directorios y los archivos en ellos).

En la MIB **no hay datos guardados**, solo está la estructura de esos datos, las indicaciones de qué OID corresponde a cada dato. Los datos reales se guardan en cada servidor SNMP, pero conocer el OID de un dato no garantiza poder acceder a él, su acceso está protegido por:
- **Community Strings**: Determinan permisos read-write (SNMPv1 y v2)
  - Las comm. strings no están cifradas en tránsito.
- **Credenciales de usuario cifradas** (SNMPv3)


* Puertos **UDP 161, 162**.
  * UDP/161 -> Envío de comandos al servidor.
  * UDP/162 -> Envío de traps ("eventos") desde el servidor de forma automática.

## Enumeración y acceso
Para acceder y leer datos con OID específicos: `snmpwalk`
```shell
snmpwalk -v2c -c public 10.10.11.87
```

Para tratar de sacar una community string por fuerza bruta: `onesixtyone`
```shell
onesixtyone -c wordlist.txt 10.10.11.87
```

Una vez tengamos una community string, para hacer fuerza bruta en los OID individuales: `braa`
```shell
#Para sacar el dato guardado en cada OID que empiece por ".x.y.z."
braa public@10.10.11.87:x.y.z.*
```
