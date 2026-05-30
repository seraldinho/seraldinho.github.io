+++
title = "Espacios vectoriales"
description = "Notas que tomaba mientras aprendía qué es un tensor."
summary = "Definición de espacio vectorial y otros conceptos básicos."
series = ["Tensores"]
series_order = 2
tags = ["mates"]
date = "2026-03-26T00:00:00"
showToc = true
showRelated = true
math = true
layoutBackgroundBlur = true
cardView = true
featureimage = "featured.jpg"
showSummary = true
showHero = true
heroStyle = "background"
+++
{{< katex >}}

Un **espacio vectorial** sobre un cuerpo (\\(\mathbb{K},+,\*\\)), con \\(\mathbb{K}=\mathbb{R}\\) o \\(\mathbb{C}\\), es toda terna (\\(\space V,+,\*_k\space\\)) donde:
- \\(V\\) es un conjunto no vacío de elementos llamados **vectores**
- \\(+\\) es la operación interna: Suma de vectores en \\(V\\).

$$
+: V \times V \to V
$$

- \\(\*_k\\) es la operación externa: Producto de un vector y un escalar

$$
\cdot : V \times \mathbb{K} \to V
$$

A partir de ahora consideraremos que \\(\mathbb{K} = \mathbb{C}\\) y \\(V = \mathbb{C}^n\\), es decir, los escalares serán números complejos de la forma \\(a+bi\\), con \\(a,b\in \mathbb{R}\\) y los vectores se definirán sobre espacios n-dimensionales de complejos.

---
## Notación de Dirac (Bra-Ket)
En general, a la hora de representar vectores usaremos la siguiente notación, denominada notación de Dirac o Bra-Ket. Esta forma es prácticamente equivalente a la usada comúnmente, \\(\vec{v_1}, \vec{v_2}, \vec{u}=2\vec{v_1}+5\vec{v_4}...\\), pero es más cómoda de usar cuando los tensores entran en juego.

Los vectores de \\(V=\mathbb{C}^n\\) se representarán de la siguiente forma, denominada ket:

$$
\ket{v_1}, \ket{v_2}, \ket{v_3}...\space\space\space\text{P.ej: }\ket{u}=2\ket{v_1}+5\ket{v_4}
$$

Un ket \\(\ket{v}\\) en esencia representa al vector abstracto original, sin embargo, cuando le fijemos una base, un ket **siempre** se "traducirá" a un **vector columna (vertical)**.

Por otro lado, existen otro tipo de entes denominados covectores, que habitan en un espacio dual que definiremos más adelante, y se representarán de la siguiente forma, denominada bra:

$$
\bra{v_1}, \bra{v_2}, \bra{v_3}...
$$

Aunque no hemos definido nada de los covectores todavía, lo importante de momento es que un covector se representa como un bra, y un bra **siempre** se "traducirá" a un **vector fila (horizontal)** cuando se mire desde una base.

> [!tip]+ Nota: Vectores abstractos
> Cuando se habla de vectores abstractos originales aquí se hace referencia a que un bra o un ket son los vectores en sí, es decir, un polinomio, un estado cuántico o, más comúnmente, una flecha que apunta hacia un sitio independientemente de respecto a qué base se mida. Para entender más, mira **Bases y coordenadas** y más específicamente **Aclaración respecto a las coordenadas**.

---
## Subespacios vectoriales
Son subconjuntos de V con estructura y propiedades de espacio vectorial, p.ej, en \\(\mathbb{R}^3\\), que es un espacio tridimensional, un plano o una recta serían subespacios.

Un subespacio \\(S\\) cumple lo siguiente:
1. Para todo par \\(\ket{v_1},\ket{v_2} \in S\\), se cumple que \\(\ket{v_1}+\ket{v_2} \in S\\)
2. Para todo escalar \\(\lambda \in \mathbb{K}\\) y \\(\ket{v} \in S\\), se cumple que \\(\lambda\ket{v} \in S\\)
3. \\(S\\) contiene el vector nulo, \\(\mathbb{0}_v\\).

- La intersección de subespacios **siempre** es un subespacio, esto no necesariamente pasa con su unión.

---
## Bases y coordenadas
Definimos una **familia de vectores** \\(\lbrace V_i \rbrace_{i \in I}\space\\) como un conjunto no vacío de vectores, no necesariamente únicos, y en el que a cada vector le corresponde un índice. P.ej:

$$
\lbrace V_i \rbrace_{i \in I} = \lbrace(1,0,0),(0,1,0),(0,0,1)\rbrace
$$


Si consideramos una familia de vectores, \\(F\\), como:

$$
F=\lbrace w_1,w_2,w_3,...,w_n\rbrace
$$

Diremos que la familia es un **sistema generador de V** si cualquier elemento \\(v\in V\\) puede generarse a partir de una combinación lineal de vectores de la familia. En otras palabras, \\(F\\) puede llegar a generar todo \\(V\\).

Diremos que la familia es **libre** si la única forma de obtener el vector nulo \\(\mathbb{0}_v\\) como combinación lineal de los vectores de la familia es si todos los escalares que los multiplican son 0, si esto no se cumple, diremos que es **ligada**. Dicho de otra forma, lo es si cada vector aporta información única a la familia y ninguno es redundante o combinación lineal de otros.

Si una familia de vectores es a la vez **libre** y **sistema generador de V**, diremos que es una **base de V**

- Trabajar en una base garantiza que solo hay una única forma de generar cada vector de V como combinación lineal de los vectores de la base.
- Una base de V tendrá tantos vectores como dimensiones tenga V (\\(\mathbb{R}^3\to\text{3 vectores.}\\))

### Coordenadas respecto a una base
Una base actúa como un sistema de cordenadas: Sus vectores definen las direcciones principales y las coordenadas del vector son los coeficientes para reconstruirlo *respecto de esa base*, como usar ejes en un plano. Una base puede verse como un punto de referencia desde el cual expresamos o vemos los vectores.

Si tenemos una familia como la siguiente:

$$
B = \lbrace(2,0,0),(0,1,0),(2,0,1)\rbrace
$$

Podemos calcular las coordenadas de un vector, como \\(\ket{v_1}=(5,9,1)\\), haciendo lo siguiente:

$$
(5,9,1) = \alpha_1(2,0,0)+\alpha_2(0,1,0)+\alpha_3(2,0,1) \\ \space \\ 5 = 2\alpha_1+2\alpha_3 \\ 9 = 1\alpha_2 \\ 1 = 1\alpha_3
$$

Y despejando:
$$
\alpha_1 = 3/2 \\ \alpha_2 = 9 \\ \alpha_3 = 1
$$

Por lo que las coordenadas de \\(\ket{v_1}\\) *respecto de la base \\(B\\) dada* serán
$$
\begin{pmatrix}
3/2 \\
9 \\
1
\end{pmatrix}_B
$$

### Aclaración respecto a las coordenadas
Cuando nosotros vemos un vector como el anterior, \\(\ket{v_1}=(5,9,1)\\), podemos pensar que, aunque todavía no hemos definido una base y estamos tratando con el "vector puro", técnicamente ya lo estamos definiendo respecto a una base, la canónica, pues podríamos pensar que:

$$
(5,9,1)=5(1,0,0)+9(0,1,0)+1(0,0,1)
$$

Lo que ocurre aquí es un "accidente": Estamos trabajando (en este caso) en \\(\mathbb{R}^3\\), y, por definición, en \\(\mathbb{R}^3\\) los propios vectores son listas de números. Como los vectores de \\(\mathbb{R}^3\\) ya son tuplas de números, **los propios vectores en sí son indistinguibles de sus bases**

Algo que podríamos hacer sería, p.ej, trabajar con polinomios, como \\(P_2(x)\\) (polinomios de grado 2 o menor), donde esa ilusión se rompe:
- Un vector \\(\ket{p} \in P_2(x)\\) se representa como, p.ej: \\(5x^2+2x+1\\)
- Sus coordenadas respecto de la base \\(\lbrace x^2,x,1 \rbrace\\) son \\((5,2,1)\\)
- Si elegimos otra base \\(\lbrace 2x^2+1,x+2,4 \rbrace\\), serán \\((\frac{5}{2},2,\frac{-11}{8})\\)

Aquí se distingue bien que el vector en sí no depende de las bases y no necesariamente (aunque sí normalmente) está formado por números, mientras que sus coordenadas respecto de una base siempre son números.

### Traducción a columnas
Sabemos que las coordenadas de un Ket \\(\ket{v}\\) respecto a una base siempre se escriben como un vector columna. Si tenemos un espacio vectorial como \\(\mathbb{C}^n\\), podemos ver esa asignación de coordenadas de la siguiente forma:

$$
\ket{v} \xrightarrow{\text{Base } B} \begin{pmatrix} c_1 \\ c_2 \\ \vdots \\ c_n \end{pmatrix}
$$

Donde \\(c_1,c_2,\dots,c_n\\) son los n números complejos que definen \\(\ket{v}\\) respecto de \\(B\\).

Ahora que sabemos que las coordenadas de los vectores son números, surge la duda de por qué existe la necesidad de representarlos en columnas.

Aunque todavía no lo hemos visto, la respuesta es una necesidad mecánica, y es que a continuación veremos que las transformaciones que modifican a estos vectores (Aplicaciones lineales) se representan mediante matrices.

Para que las reglas de multiplicación de matrices funcionen y podamos aplicar la transformación \\(A\\) sobre un vector \\(\ket{x}\\), la operación \\(A\cdot\ket{x}\\) exige obligatoriamente que \\(\ket{x}\\) sea una columna.

Por otro lado, los covectores o bras (\\(\bra{v}\\)), mencionados levemente, se traducirán siempre a vectores fila porque cumplen un papel complementario al ket. Precisamente la simetría entre bra (\\(\bra{v_1}, \text{fila}\\)) y ket (\\(\ket{v_2}, \text{columna}\\)) es lo que fundamenta los tensores.

---
## Aplicaciones lineales
Sean \\(V\\) y \\(W\\) dos espacios vectoriales sobre un mismo cuerpo \\(\mathbb{K}\\), como \\(\mathbb{C}\\), una aplicación lineal u homomorfismo es una función:
$$
f:V\to W
$$
Que cumple las siguientes propiedades fundamentales:
1. \\(f(\ket{v_1}+\ket{v_2})=f(\ket{v_1})+f(\ket{v_2})\\)
2. \\(f(\lambda \ket{v}) = \lambda f(\ket{v}),\space \forall \lambda \in \mathbb{K}\\)

### Matriz coordenada
Es la representación matricial de una aplicación lineal *respecto de unas bases específicas*. Lo bueno o útil de esta representación es que aplicar una función abstracta sobre el vector, como:
$$
f(\ket{v}) = \ket{w}
$$

Se convierte en una multiplicación por matrices:
$$
A \cdot \begin{pmatrix} v_1 \\ v_2 \\ \vdots \\ v_n \end{pmatrix} = \begin{pmatrix} w_1 \\ w_2 \\ \vdots \\ w_m \end{pmatrix}
$$

Para conseguirla (ejemplo) con una aplicación lineal \\(f:V\to W\\):
Suponemos una base \\(B_V=\lbrace \ket{v_1},\ket{v_2},\dots,\ket{v_n} \rbrace\\) para el espacio de origen y otra \\(B_W=\lbrace \ket{w_1},\ket{w_2},\dots,\ket{w_m} \rbrace\\) para el de destino.

1. Aplicamos la función \\(f\\) a cada uno de los vectores de la base de \\(V\\).
2. Como esos vectores viven en \\(W\\), podemos calcular sus coordenadas respecto de la base \\(B_W\\), es decir, los expresamos como combinación lineal de los vectores de la base de \\(W\\)

$$
\begin{aligned}
f(\ket{v_1}) &= a_{11}\ket{w_1} + a_{21}\ket{w_2} + \dots + a_{m1}\ket{w_m} \\
f(\ket{v_2}) &= a_{12}\ket{w_1} + a_{22}\ket{w_2} + \dots + a_{m2}\ket{w_m} \\
&\ \ \vdots \\
f(\ket{v_n}) &= a_{1n}\ket{w_1} + a_{2n}\ket{w_2} + \dots + a_{mn}\ket{w_m}
\end{aligned}
$$

3. Ponemos los coeficientes de cada transformación (las coordenadas de cada vector) y las colocamos por columnas en la matriz \\(A\\).

$$
A = \begin{pmatrix}
a_{11} & a_{12} & \dots & a_{1n} \\
a_{21} & a_{22} & \dots & a_{2n} \\
\vdots & \vdots & \ddots & \vdots \\
a_{m1} & a_{m2} & \dots & a_{mn}
\end{pmatrix}
$$

### Características
Una aplicación lineal \\(f:V\to W\\) se dice:
- **Inyectiva (Monomorfismo)**: si \\(\forall \ket{u},\ket{v} \in V\\), \\(\ket{u}\neq \ket{v}\\), \\(f(\ket{u})\neq f(\ket{v})\space\\)
  - En otras palabras, no hay dos vectores de \\(V\\) que vayan al mismo vector de \\(W\\). Esto implica que \\(W\\) debe tener las dimensiones suficientes para guardar todas las de \\(V\\) de forma que quepa todo individualmente. Esta idea resultará más intuitiva al ver las condiciones para la inyectividad.
- **Suprayectiva (Epimorfismo)**: si \\(\forall \ket{w} \in W, \exists \ket{v} \in V\\) tal que \\(f(\ket{v})=\ket{w}\\)
  - Hay una cantidad de vectores (o más bien dimensiones) suficientes en \\(V\\) como para llenar todo \\(W\\), y además la aplicación lineal los transforma de tal forma que verdaderamente llenen todo el espacio.
- **Biyectiva (Isomorfismo)**: Si \\(f:V\to W\\) es tanto inyectiva como suprayectiva.

### Kernel (\\(\text{Ker} f\\))
El núcleo o kernel de una aplicación lineal es el conjunto de todos los vectores de entrada (\\(V\\)) que al pasar por la función \\(f:V\to W\\) se convierten en el vector nulo, \\(0_w\\), en \\(W\\)

$$
\text{Ker} f = \lbrace \ket{v} \in V\space |\space f(\ket{v}) = 0_w \in W \rbrace
$$

- \\(\text{Ker} f\\) siempre es un subespacio vectorial de \\(V\\).

Para hallarlo, se iguala la definición de la aplicación lineal al vector nulo de \\(W\\), p.ej, en el caso de una función \\(f:\mathbb{R}^2\to M_2(\mathbb{R)}\\):

$$
f(x,y)= \begin{pmatrix} 
x & y-x \\
x+y & y \end{pmatrix}
$$

Se resuelve el siguiente sistema:
$$
\begin{pmatrix} 
x & y-x \\
x+y & y \end{pmatrix}
= \begin{pmatrix} 
0 & 0 \\
0 & 0 \end{pmatrix}
$$
Que en este caso saldrá \\(x=0,y=0\\), por lo tanto \\(\text{Ker} f = \lbrace(0,0)\rbrace = 0_v\\)

> [!TIp] Nota: Inyectividad
> Dada una aplicación lineal \\(f:V\to W\\), \\(f\\) es **inyectiva** si y solo si \\(\text{Ker} f = 0_v\\)

### Imagen (\\(\text{Im} f\\))
El conjunto imagen de una aplicación lineal \\(f:V\to W\\) es el conjunto de elementos de \\(W\\) a los que se puede llegar al realizar la aplicación lineal sobre elementos de \\(V\\), los "resultados posibles" que la función puede dar.

$$
\text{Im} f = \lbrace \ket{w} \in W \space | \space \exists \ket{v} \in V \space\text{t.q.}\space f(\ket{v})=\ket{w}
$$

> [!TIp] Nota: Suprayectividad
> Dada una aplicación lineal \\(f:V\to W\\), \\(f\\) es **suprayectiva** si y solo si \\(\text{dim}(\text{Im} f) = \text{dim}(W)\\). (\\(\text{dim = dimensión}\\))

## Endomorfismos
Existe un caso particular de aplicaciones lineales que resulta de importancia para el caso de los tensores. Se trata de la situación en la que el espacio de origen y de destino son el mismo, es decir:
$$
f:V\to V
$$
Esta aplicación lineal recibe el nombre de endomorfismo, y será de gran utilidad en el siguiente elemento a definir. Además, como el espacio de origen, \\(V\\), y el de destino, \\(W\\), tienen la misma dimensión, p.ej \\(\mathbb{C}^n\\), la matriz coordenada que representa la aplicación siempre será cuadrada (tamaño \\(n \times n\\)), lo que también es relevante porque solo a las matrices cuadradas se les pueden calcular cosas como los valores propios.
