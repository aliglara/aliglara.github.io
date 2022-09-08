---
layout: post
title:  "Una manera sencilla de resolver una ecuacion de estados cúbica."
author: ali
categories: [ termodinamica, tutorial ]
tags: [scipy, python]
image: assets/images/cubic-math.jpeg
# beforetoc: "Markdown editor is a very powerful thing. In this article I'm going to show you what you can actually do with it, some tricks and tips while editing your post."
# toc: true
---
Es muy frecuente tener que calcular el factor de compresibilidad (Z) de un gas, empleando ecuaciones de estado cúbicas. En este post, les muestro una manera sencilla para obtener  los valores de Z que tienen significado físico.

Como ejemplo, suponga que desea determinar las raíces del factor de compresibilidad empleando la ecuación generalizada de Peng-Robinson para una sustancia que se encuentra a una temperatura reducida (Tr) de 0.7, Presion reducida (Pr) igual a 0.1 y un factor acéntrico de 0.09.

## Solución

Para encontrar las raices de la ecuación usaremos aproximacion numérica. Existen diversas maneras para lograr el objetivo. En este caso, usaremos la funcion *polynomial* implementado en el modulo **numpy** de Python, la cual permite obtner las raices de una funcion lineal o no lineal del tipo *f(x) = 0*

```python
import numpy as np
import matplotlib.pyplot as plt
```

La ecuacion de estado de Peng-Robinson tiene la siguiente forma:

$$
\begin{align*}
&\alpha = \left[1 + \left(0.37464 + 1.5422* \omega - 0.26992*\omega^2\right) * \left(1 - \sqrt(Tr)\right)\right]^2 \\
&A = 0.45724\left(\frac{P_{r}}{T_{r}^{2}}\right)\:\alpha \\
&B = 0.07780\left(\frac{P_{r}}{T_{r}}\right) \\
&Z^3 + (-1+B)\:Z^2 + (A-2B-3B^2)\:Z + (-AB + B^2+B^3)
\end{align*}
$$

**Paso 1**. Definir una funcion en Python para estimar 

```python

def eos(z, tr, pr, omega):
    '''
    Calculo de las raices de la ecuaion cubica de Peng Robinson
    input: Tr - Temperatura reducida
           Pr: Presion reducida
           omega: factor acentrico de sustancia
    output:
           zsol: objeto polinomico de numpy
           zeval: valores de funcion para grafico
    '''
    alpha = (1 + (0.37464 + 1.5422*omega - 0.26992*omega**2) * 
             (1 - np.sqrt(tr)))**2
    A = 0.45724 * (pr/tr**2) * alpha
    B = 0.07780 * (pr/tr)
    p = -1 + B
    q = A - 2*B - 3*B**2
    r = -A*B + B**2 + B**3
    
    zsol = np.polynomial.Polynomial((r, q, p, 1))
    zeval = z**3 + p * z**2 + q * z + r
    return [zeval, zsol]
```

Esta función tiene como salida el objeto polinómico de [numpy](https://numpy.org/doc/stable/reference/generated/numpy.polynomial.polynomial.Polynomial.html) y una numpy array con la evaluacion de la función solo para construir un grafico de la ecuacion cúbica.

**Paso 2**. Visualización del comportamiento

El gráfico del comportamiento de la ecuacion de estado de Peng-Robinson

![EOS Peng-Robinson](https://ik.imagekit.io/x860v02j7/figura-zvalues_WAJJwfHUC.png?ik-sdk-version=javascript-1.4.3&updatedAt=1662416424323
)

**Paso 3**. Presentacion de resultados

Los valores del factor de compresibilidad (Z) para la sustancion en estado líquido saturado es 0.893, mientras que para el vapor es igual 0.015. Estos resultados son consistente con lo observado en el gráfico anterior.

Mas detalles sobre el codigo disponible aca [enlace](https://github.com/aliglara/posts/blob/main/codes/cubica-solucion-numpy.ipynb)