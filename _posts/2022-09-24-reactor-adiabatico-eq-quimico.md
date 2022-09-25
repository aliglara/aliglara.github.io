---
layout: post
title:  "Resolución de reactor adiabático en equilibrio químico."
author: ali
categories: [ termodinamica, tutorial ]
tags: [scipy, sympy, python]
image: "https://ik.imagekit.io/x860v02j7/working-white-star-military-food-balance-806073-pxhere.com_y4BxGZ7GA.jpg?ik-sdk-version=javascript-1.4.3&updatedAt=1664023662576"
# beforetoc: "Markdown editor is a very powerful thing. In this article I'm going to show you what you can actually do with it, some tricks and tips while editing your post."
# toc: true
---

Mientras estuve dictando clases de Termodinámica del Equilibrio en Venezuela, uno de los aspectos que resultaba laborioso para los estudiantes era combinar el desarrollo de modelo matemáticos simbólicos para luego debían ser resueltos numéricamente.

Cuando se necesita determinar la máxima conversión (o grado de avance) para reacción reversible se suele tener dicha situación. Python entre una de sus ventajas es que posee una libreria llamada [__Sympy__](https://www.sympy.org/en/index.html) para matemática simbólica que facilita el proceso.

En este post, se ejemplifica el uso de esta librería mediante la resolución de un problema de equilibrio químico termodinámico.

## Enunciado

La reacción de síntesis catalítica de metanol puede ser representada mediante la siguiente ecuación

\begin{equation*}
\text{CO(g)} + 2\text{H}_2\text{(g}) = \text{CH}_3\text{OH(g)}
\end{equation*}

Si se cuenta con una alimentación equimolar de los reactantes e igual a 5 moles a 200 $^\circ$C. ¿Cuál será las fracciones molares si el sistema opera adiabáticamente a 10 bar y el catalizador permite que el sistema alcance el equilibrio sin que exista alguna otra reacción no deseada?

_Datos_ del problema se encuentran en el siguiente [enlace](https://github.com/aliglara/posts/blob/main/problems/5311-quimico-01.md)

## Solucion

### Balance de moles

El primer paso en este tipo de problemas, es establecer el balance de moles

$$ n_i = n_{i0} + \nu_i\:\xi $$

Que indica como cambia los moles iniciales de un componente _i_ ($n_{i0}$) debido al grado de avance de la reaccion $\xi$. La variación molar del componente_i_ esta ponderada por su coeficiente estequiométrico ($\nu_i$), el cual sera negativo del compuesto es un reactante o positivo en caso de ser un producto.

Para los cálculos de equilibrio químico, lo que se necesita es conocer la composición molar de cada componente en la fase de reacción. En este caso, la fase de gaseosa, así que emplearemos la letra _y_ para describirlas.

Para conocer las expresiones simbólicas de las fracciones molares, se define la variable _xi_ empleando Sympy

```python
xi = sp.symbols("xi", real=True)

ni = molesAlime + CoefMatrix * xi
yi = ni / np.sum(ni)
```

Ahora se puede conocer las fracciones molares de cada componente cuando el sistema reactivo se encuentre en equilibrio químico:

$$
\text{CO \quad\quad H2 \quad\quad CH3OH} \\
\left[ \frac{5 - \xi}{10 - 2 \xi}, \  \frac{5 - 2 \xi}{10 - 2 \xi}, \  \frac{\xi}{10 - 2 \xi}\right]
$$

Ahora las fracciones molares en equilibrio, solo tendran significado físico cuando los moles finales de todos los componentes sean mayores o iguales a cero. Nuevamente, Sympy permite encontrar la solucion a este sistema de tres inecuaciones considerando la restricción física.

```python
sol = sp.solve([ni[n] >= 0 for n in np.arange(3)])
display(sol)
```

$$
\xi \geq 0 \wedge \xi \leq \frac{5}{2}
$$

## Constante de equilibrio químico

### En función de la actividad

Cuando un sistema reactivo alcanza el equilibrio químico termodinámico tendrá una restricción adicional que esta descrita matemáticamente con la constante de equilibrio químico ($K$). En este caso, el sistema es una solución gaseosa que se comporta como gas ideal, por lo cual la expresión de $K$ será:

$$ K(y_i) = \prod {a_i^{\nu_i}} = \prod {\frac{y_i\:P^{\nu_i}}{P^\circ}} $$

donde $a_i$ es la actividad del componente _i_, _P_ es la presión del sistema y $P^\circ$ es la presión estándar (1 bar)

```python
Ky = np.prod((yi * PresionOp)**CoefMatrix) 
```

$$
K(\xi) = \frac{\xi \left(10 - 2 \xi\right)^{2}}{100 \left(5 - 2 \xi\right)^{2} \cdot \left(5 - \xi\right)}
$$

### En función de la energía libre de Gibbs

Se conoce la expresión de la constante de equilibrio químico, pero es una ecuación con dos incognitas : _K_ y $\xi$

La segunda restricción para _K_ viene expresada por la energía requerida para que la sistema alcance el equilibrio y esa viene dado por la ecuación de Gibbs-Duhem, la cual a presión y temperatura constante permite definir la constante de equilibrio químico como:

$$ K(T) = \exp{\frac{-\Delta G^\circ(T)}{R\,T}} $$

Para lo cual es necesario resolver empleando el método de van't Hoff:

$$
\begin{align*}
\ln K(T) &= \ln K(T^\circ) + \frac{1}{R}\int_{T^\circ}^{T} \frac{\Delta H^\circ(t)}{t^2} \, dt\\
\Delta H^\circ(T) &= \Delta H^\circ(T^\circ) + \int_{T^\circ}^{T} {\Delta C_p^\circ(t) \, dt} \\
\Delta C_p^\circ(T) &= \sum{\nu_i\,C_{p,i}^\circ(T)}  \\
& \equiv \sum{\nu_i\,A_i} + \sum{\nu_i\,B_i}\,T \\ &  = \Delta A + \Delta B\,T
\end{align*}
$$

Los detalles de como Sympy permite poner todo junto se encuentra en [GitHub](https://github.com/aliglara/posts/blob/main/codes/5311-t04-reactor-adiabatico.ipynb) pero en esencia permite obtener la siguiente expresión:

$$
K(T) = \frac{8.494\cdot 10^{9}}{T^{7.69}} \exp\left({3.92\cdot 10^{-2}T + \frac{8928.032}{T}}\right)
$$

Ahora se puede combinar ambas expresiones para la constante de equilibrio químico

$$
\frac{\xi \left(10 - 2 \xi\right)^{2}}{100 \left(5 - 2 \xi\right)^{2} \cdot \left(5 - \xi\right)} = \frac{8.494\cdot 10^{9}}{T^{7.69}} \exp\left({3.92\cdot 10^{-2}T + \frac{8928.032}{T}}\right)
$$

## Balance de energía

Se sigue teniendo una ecuación con dos incognitas (_T_) y $\xi$) por lo cual se necesita otra expresión para resolver el problema. Esta expresión viene dada por el balance de energía del sistema.

Si se toma como referencia la temperatura de alimentación al reactor y opera de forma adiabática, entonces el balance de energía será:

$$
Q(T) = \sum{n_{i0} \, \int_{T_0}^{T} C_{p_i}(t)\: dt} + \xi \: \Delta H^\circ (T) = 0
$$

El cual se puede escribir empleando Sympy como:

```python
F2 = np.sum([molesAlime[n] * sp.integrate(data[n, 2] +  data[n, 3] * 1e-2 * t, (t, TempInicial, Tout)) 
             for n in np.arange(3)]) + xi * DHt
```

Resultando en:

$$
Q(T) = T^{2} \cdot \left(0.032615 \xi - 0.00895\right) + T \left(290.05 - 64.0 \xi\right) - 74227.65 \xi - 135233.51
$$

Para resolver el sistema de ecuaciones, se necesita convertir ambas expresiones en ecuaciones numéricas y _Sympy_ tiene el módulo _lambdify_ que hace lo propio.

```python
F1 = sp.log(Ky) - lnKe 

f1 = lambdify((T, xi), F1, "numpy")
f2 = lambdify((T, xi), F2, "numpy")
```

Ahora f1 y f2 son dos objetos que representan las funciones anónimas de las expresiones simbólicas, con lo cual se puede emplear cualquier de los resolvedores de Python para obtener la resolución del sistema.

En este caso, emplearemos el modulo de _least_squares_ de _Scipy_ ya que facilita la especificaciones de los rangos de valores para cada una de las incógnitas.

```python
def problema(z, *args):
    """ Resuelve el grado avance y temperatura de operacion
    de un reactor que opera de forma adiabatica.
    
    Input: z: es un vector donde se encuentran las variables
           independientes: temperatura y grado de avance
           args: son las funciones que seran resueltas
           En este caso, args[0] es la funcion de igualdad de 
           constante de equilibrio mientras que args[1] es
           el balance de energia del reactor
    Output: funciones a ser resueltas por el resolvedor
    """
    temp, grado_avance = z
    ecuacion_1, ecuacion_2 = args
    
    fun1 = ecuacion_1(temp, grado_avance)
    fun2 = ecuacion_2(temp, grado_avance)
    return [fun1, fun2]
```

Implementación de least_squares

```python
x0 = (400, 0.01)
bnds = ((298.15, 1e-3), (800, 2.5-1e-3))
solucion = least_squares(problema, [400, 0.01], 
                   args=(f1, f2), bounds=bnds)
```

Y la solución es:

    La temperatura de equilibrio es 530.83 K, mientras que el grado de avance es igual a 0.1637

Como se muestra en este post, la libreria de Sympy facilita el desarrollo de modelos matemáticos simbólicos.

Mas detalles sobre el codigo disponible aca [enlace](https://github.com/aliglara/posts/blob/main/codes/5311-t04-reactor-adiabatico.ipynb)
