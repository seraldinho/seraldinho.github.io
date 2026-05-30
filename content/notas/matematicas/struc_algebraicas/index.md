+++
title = "Estructuras algebraicas"
description = "Notas que tomaba mientras aprendía qué es un tensor."
summary = "Definición de grupo, anillo y cuerpo."
series = ["Tensores"]
series_order = 1
date = "2026-03-25T00:00:00"
tags = ["mates"]
showToc = true
showRelated = true
math = true
layoutBackgroundBlur = true
cardView = true
featureimage = "featured.jpg"
showSummary = true
showHero = true
heroStyle = "background"
hideFeatureImage = false
+++
{{< katex >}}

Una **estructura algebraica** puede definirse como una tupla de elementos \\(a_1, a_2, ..., a_n\\) que consta de un conjunto no vacío \\(a_1\\) (o \\(G\\)) y otro conjunto \\(\lbrace a_2, ..., a_n\rbrace\\) de operaciones que pueden aplicarse a los elementos del primer conjunto, \\(a_1\\).

#### Propósito
Normalmente, las matemáticas se centran en qué son los objetos (qué es un número, un vector, una matriz). Las estructuras algebraicas, por otro lado, abstraen ese "qué" para enfocarse en cómo se comportan esos objetos y en cuáles son sus propiedades, recogiéndolos en una serie de estructuras.

Esto permite generalizar y simplificar sistemas, reduciéndolos a sus axiomas para estudiar sus comportamientos y patrones.

En nuestro caso, buscando llegar a definir y entender los tensores, primero tenemos que definir la base para luego poder definir la estructura sobre la que se fundamentan.

### Definiciones

##### **Producto cartesiano**
Dados dos conjuntos \\(A\\) y \\(B\\), el producto cartesiano de ambos (\\(A \times B\\)) es el conjunto de todos los pares ordenados \\((a,b)\\) en los que \\(a \in A\\) y \\(b \in B\\). No se exige que \\(A\\), \\(B\\) sean disjuntos. Es decir:
  - \\(A \times B = \lbrace (a,b)\space |\space a\in A \land b \in B \rbrace\\)
  - \\(A=\lbrace 1,2\rbrace ,\space B=\lbrace 2,3\rbrace ,\space A \times B = \lbrace(1,2),(1,3),(2,2),(2,3) \rbrace\\)

##### **Aplicación entre \\(A\\) y \\(B\\)**
Es una función que asocia elementos de \\(A\\) con elementos de \\(B\\) de forma unívoca, un elemento de \\(A\\) se asocia con un solo elemento de \\(B\\).
  - Sintaxis: \\(f:A \to B\\)

##### **Operación binaria interna sobre \\(G\\)**
Aplicación que asigna a cada par ordenado de elementos \\((a_1,a_2) \in A\times A\\) un elemento \\(a_3=a_1*a_2\\) tal que \\(a_3 \in A\\).
  - Sintaxis: \\(*:A \times A \to A\\)

##### **Operación binaria externa en \\(G\\)**
Dados dos conjuntos \\(A\\), \\(\Omega\\), una operación binaria externa es una aplicación que asigna a cada par ordenado de elementos \\((\lambda,x) \in \Omega \times A\\) un elemento \\(c=\lambda\bullet x\\) tal que \\(c \in A\\).
  - Sintaxis: \\(\bullet :\Omega \times A \to A\\)
  
>[!tip]+ Nota: \\(\Omega\\)
> Posteriormente, al hablar de espacios vectoriales, ese \\(\Omega\\) será nuestro conjunto de escalares y \\(A\\) será el conjunto de vectores.

---

### (\\(\mathbb{G},*\\)) - Grupos
Se llama **grupo** al par (\\(G,\*\\)), en el que \\(G\\) es un conjunto no vacío y \\(*\\) es una operación binaria interna. Esta estructura cumple las siguientes propiedades:
- Se cumple la propiedad asociativa respecto de \\(*\\).
- Existe elemento neutro tal que \\(a\*e=a\space \land e\*a=a\\), y es único.
- Para todo elemento \\(a \in G\\) existe un simétrico \\(a'\\) tal que \\(a\*a'=e \land a'\*a=e\\)
- Si \\(\forall a,b \in G, \space a\*b=b\*a\\), se le llama grupo abeliano o conmutativo.

Las operaciones de todos los elementos de \\(G\\) entre sí puede representarse en una [tabla de Cayley](https://es.wikipedia.org/wiki/Tabla_de_Cayley).

> [!Tip]+ Nota: Utilidad
> Los grupos son la base que necesitamos definir y a partir de la cual construiremos las siguientes estructuras. De momento tenemos un conjunto y podemos sumar elementos, pero necesitamos más, necesitamos poder multiplicar.

---

### (\\(\mathbb{A},+,*\\)) - Anillos
Los anillos son el paso intermedio hacia la estructura algebraica que buscamos definir, los espacios vectoriales.

Un **anillo** es un conjunto \\(A\\) no vacío en el que se definen dos operaciones binarias internas, \\(+\\) y \\(*\\), que cumplen lo siguiente:
- (\\(A,+\\)) forma un grupo conmutativo.
- La operación \\(*\\) es asociativa.
- Se cumple la propiedad distributiva de \\(*\\) respecto de \\(+\\)
- Si el producto también es conmutativo, entonces se llama a la estructura anillo conmutativo.
- Si el producto también tiene elemento neutro, se llama a la estructura anillo con unidad.

Un elemento \\(a \in A\\) se dice invertible o unidad si \\(\exists a' \in A, \space a*a'=a'*a=1\\).

Un elemento \\(a \in A\\) se dice divisor de cero si \\(\exists b \in A\\) tal que \\(a,b \neq 0\\) y  \\(\space ab=0\space\lor ba=0\\).

> [!Tip]+ Nota: Problema
> Un anillo es en esencia un grupo conmutativo al que le añadimos una segunda operación, la multiplicación, pero tiene un problema: En un anillo estándar, como (\\(\mathbb{Z},+,*\\)), no siempre podemos dividir y obtener un elemento de \\(\mathbb{Z}\\), esto es, no todos los elementos tienen inverso para la multiplicación. Necesitamos algo más.

### (\\(\mathbb{K},+,*\\)) - Cuerpos
Un **cuerpo** (*field* en inglés) es un conjunto \\(\mathbb{K}\\) no vacío en el que se definen dos operaciones binarias internas, \\(+\\) y \\(*\\), que cumplen lo siguiente:
- (\\(\mathbb{K},+\\)) forma un grupo conmutativo.
- (\\(\mathbb{K} \setminus \lbrace 0 \rbrace,*\\)) forma un grupo conmutativo.
- Se cumple la propiedad distributiva de \\(*\\) respecto de \\(+\\)

Es decir, un cuerpo se define como *un anillo conmutativo con unidad en el que todo elemento no nulo tiene inverso multiplicativo*. En definitiva, es un conjunto con dos operaciones que satisfacen:
- Las dos operaciones son asociativas
  - \\(a_1+(a_2+a_3)=(a_1+a_2)+a_3\\), \\(\space\space a_1\*(a_2\*a_3)=(a_1\*a_2)\*a_3\\)
- Existe elemento neutro para cada operación
  - \\(0+a=a+0=a,\space\space 1\*a=a\*1=a \space \space \forall a \in \mathbb{K}\\)
- Las operaciones son conmutativas
  - \\(a_1+a_2=a_2+a_1, \space \space a_1\*a_2=a_2\*a_1\\)
- Respecto a la suma, **todos los elementos**  tienen inverso.
  - \\(a+(-a)=(-a)+a=0\\)
- Respecto al producto, **todos los elementos menos el cero** tienen inverso.
  - \\(a\*a^{-1}=a^{-1}\*a=1\\)
- Se cumple la propiedad distributiva de \\(\*\\) respecto de \\(+\\).

> [!Tip]+ Nota: Anillo completo
> Ahora ya podemos sumar, restar, multiplicar y *dividir* sin salirnos del conjunto definido en \\(\mathbb{K}\\). Ejemplos de esto son \\(\mathbb{R}\\) o \\(\mathbb{C}\\) y van a permitir definir la siguiente estructura, el espacio vectorial.
