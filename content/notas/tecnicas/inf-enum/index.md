+++
title = 'Enumeración de Infraestructuras'
draft = false
description = "Explicación del proceso de enumeración de infraestructuras de red."
tags = ["Infrastructure"]
showToc = true
math = true
+++

## Introducción
La enumeración de la infraestructura de una empresa u organización empieza desde el punto de vista más global, desde los sistemas autónomos (AS), y se va volviendo más específico hasta llegar a IPs y servidores específicos. 

## 1. Sistemas Autónomos (AS)
Un AS es una colección de redes IP ([rangos CIDR](https://aws.amazon.com/es/what-is/cidr/)) que están bajo el control de una misma entidad administrativa, como pueden ser ISPs o empresas.

Los rangos que pertenecen a cada AS son necesariamente públicos para que los AS puedan comunicarse entre sí y sepan hacia dónde enrutar cada paquete. Por ello podemos usar *[bgp.tools](https://bgp.tools)*, *[bgp.he.net](https://bgp.he.net)* o herramientas automatizadas como [*Metabigor*](https://github.com/j3ssie/Metabigor) para encontrar cuál es el AS (si tiene uno propio) de una organización y, dentro de ese AS, cuál es el rango de IPs que le pertenece.

### Ejemplo: Mercadona
P.ej, podemos considerar la empresa **Mercadona**, que es lo suficientemente grande como para tener su propio AS, pero no es una multinacional dispersa que complique el análisis.

Si miramos [bgp.tools](https://bgp.tools), veremos que "*Mercadona S.A*" tiene el **ASN 201976**. Nos lo apuntamos.

Si entramos al apartado de prefijos de ese AS específico, veremos 12 rangos de IPs. En un análisis real también los apuntaríamos.

## 2. IPs, dominios
Ahora que tenemos todos los rangos de IPs, tenemos que conseguir encontrar todos los dominios, subdominios e IPs posibles dentro de nuestro scope. Dado que una IP puede resolver a varios dominios y un dominio puede resolver a varias IPs, haremos lo siguiente de forma cíclica:

1. **Reverse DNS**: Para las IPs descubiertas, buscamos registros PTR para ver a qué dominios están asociadas. Esto también puede hacerse en las páginas anteriores.
2. **DNS Lookup**: Para los dominios descubiertos en el paso 1, hacemos DNS lookups para resolverlos a direcciones IP.

> Mientras hacemos esto (podemos usar herramientas que lo automatizan), podemos usar [Shodan](https://www.shodan.io/) o [Censys](https://search.censys.io/) para encontrar servicios expuestos que correspondan a dominios encontrados o que contengan el nombre de nuestra organización objetivo.

3. **Certificados**: Cuando ya tengamos unos cuantos dominios, podemos usar [crt.sh](https://crt.sh/) para encontrar más dominios y subdominios asociados a certificados SSL/TLS que pueden no estar listados en DNS (P.ej entornos de pruebas o desarrollo).
4. **Iterar**: Cada dominio/subdominio/IP nuevo entra al paso 1, repitiéndose el bucle.
5. **Fin del bucle**: Mientras se encuentre una cantidad relevante de información en cada iteración, el bucle sigue. Cuando ya se considere que la información conseguida no es suficiente para que merezca la pena seguir o si se tiene lo que se buscaba, se puede concluir la enumeración.

## 3. Infraestructura en la nube
Cada vez es más frecuente que parte de la infraestructura de una organización esté alojada en la nube y no en direcciones de su ASN. Esto significa que buscando sólamente en ese ASN potencialmente nos perderíamos gran parte de la información.

Además, en este caso deja de ser posible el bucle de DNS Lookups, dado que el AS es del proveedor y no de nuestro objetivo. Por ello aquí la idea es buscar huellas de identidad de nuestro objetivo, independientemente de dónde esté hosteado. Esto significa que podemos buscar:
### 3.1 Certificados SSL/TLS 
Rara vez se emiten certificados SSL/TLS para subdominios individuales. Así como hemos buscado antes en `crt.sh`, podemos buscar en otras páginas (Shodan/Censys) certificados con el nombre de la empresa (u otros campos en común). P.ej, en Censys:
```Censys
services.tls.certificates.leaf_data.subject.organization: "NombreEmpresa"
```
Si ejecutamos esto para Mercadona, encontraremos IPs de ASNs de Oracle o Google que no hubiésemos visto solo con la metodología anterior.

### 3.2 Permutaciones de nombres
Si tenemos un subdominio específico confirmado, podemos resolverlo por DNS. Si el subdominio corresponde a un recurso en la nube, rara vez apuntará a una IP (A), sino que lo normal es que sea un alias (CNAME) a un subdominio del proveedor.

Si conseguimos un subdominio del proveedor (normalmente con un nombre puesto por alguien de la organización), podremos ver si hay algún patrón que podamos mantener para ir probando otros subdominios.

### 3.3 Favicon Hashing
Muchos servicios de la empresa, tanto en la nube como en su infraestructura propia, tendrán el mismo favicon. Por lo que podremos hacer lo siguiente:
1. Calcular el hash de `favicon.ico` de una página de la empresa (p.ej la web principal)
2. Buscar en Shodan el hash (p.ej `http.favicon.hash:<HASH_AQUI>`)
3. Shodan mostrará cualquier servidor con el mismo favicon.
