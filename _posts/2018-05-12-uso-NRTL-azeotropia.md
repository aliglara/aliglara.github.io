---
layout: post
title:  "Ejemplo de determinacion de punto azeotropico"
author: ali
categories: [ termodinamica, tutorial, NRTL ]
tags: [scipy, python]
image: https://ik.imagekit.io/x860v02j7/split-ntrl__uLlIciCF.jpg?ik-sdk-version=javascript-1.4.3&updatedAt=1663423244335
---
En Ingeniería de procesos es común encontrar disoluciones de compuestos no ideales. Por lo cual para la determinación de las propiedades en equilibrio de fases es necesario emplear alguna modificación de la condición de equilibrio que tome en cuenta el comportamiento no ideal.

En este post, se presenta un ejemplo de calculo empleando la relacion de Raoult modificada y el modelo de coeficiente de actividad NRTL para determinar si un sistema binario:

- Presenta un punto azeotropico
- Si es positivo, que tipo de punto azeotropico es y donde se encuentra.
- Cuales serian las condiciones de separacion necesarias?

## Problema

Para un mezcla de Acetonitrilo/n-heptano que se encuentra a 1 bar. Sabiendo que el modelo de NRTL representa adecuadamente las propiedades de la solución líquida con los siguientes parámetros: $\alpha = 0.333, \: b_{12}/R = 768.57 \text{ y } \: b_{21}/R = 465.43$.

La presión de saturación puede ser estimada mediante:

$$
P^\text{sat} = \exp\left(A + \frac{B}{T} + C \, \ln(T) + D \, T^E\right)
$$

Donde la presión en Pa usando T en grados Kelvin

Las constantes de la ecuación de Antoine son

Componentes       | A | B | C | D | E | Tmin [K] | Tmax [K]
------------------|---|---|---|---|---|----------|---------
Acetonitrilo (1)  | 58.302| -5385.6| -5.4954| 5.3634e-6| 2| 229.32| 545.5
n-heptano (2)     | 154.62| -8793.1| -21.684| 2.3916e-2| 1| 182.56| 540.26

## Presentacion de resultados

Para obtener una composicion de acetonitrilo igual al 85% de la composicion del punto azeotropico, el flash debe operar a una temperatura 349.59 K y una presion de 1 Pa.

Mas detalles sobre el codigo disponible aca [enlace](https://github.com/aliglara/posts/blob/main/codes/5311-t03-flash-nrtl.ipynb)