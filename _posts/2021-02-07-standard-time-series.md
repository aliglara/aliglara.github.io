---
layout: post
title:  "¿Es realmente necesario normalizar o estandarizar datos de series de tiempo antes de analizarlos?"
author: ali
categories: [ Machine learning, Preprocessing, tutorial ]
tags: [happycoding, python]
image: "https://ik.imagekit.io/x860v02j7/standarization-img_1F6oYefra.jpeg?ik-sdk-version=javascript-1.4.3&updatedAt=1662109049334"
description: "Brief comments about standarization of a time-serie dataset."
---

En mi último proyecto fui consultado con esta duda varias veces, acá publico mi opinión al respecto y cómo se ejecuta en Python.

Basado en mi ámbito de trabajo cuando las series de tiempo posee features (variables independientes) y target (variable dependiente) con órdenes de magnitudes muy diferentes suelo escalar los datos.

## ¿Cuál es la diferencia entre normalizar y estandarizar?

__Normalizar__ es escalar los datos desde su valores originales a un rango entre 0 y 1, por lo tanto se requiere conocer cual es el valor mínimo y máximo del universo para realizar la operación. Una lista de la manera de hacerlo esta en [wikipedia](https://en.wikipedia.org/wiki/Normalization_(statistics)).

Normalmente, trabajo con serie temporales proveniente de registros de procesos industriales y son no estacionarias desde el punto de vista estadístico, asi que normalizar no suele ser beneficioso. Sin embargo, existen algoritmos de machine learning que requieren que la data sea normalizada.

Por otra parte, __estandarizar__ se refiere a escalar la distribución de los datos de forma tal que la media de los valores observados sea igual a 0 y su desviación estándar igual a 1. ([wikipedia](https://en.wikipedia.org/wiki/Normalization_(statistics)))

En general, los algoritmos de machine learning que se basen sobre la hipótesis que los datos poseen una distribución gaussiana requieren que los mismos sean estandarizados. Sin embargo, la selección de la manera de escalar la data no es una tarea de una sola vía, depende mucho de las circunstancias y de los resultados del análisis descriptivo inicial del problema.

## ¿Cómo se puede hacer?

Las maneras de lograr este tipo de transformación son variadas pero Python brinda un ecosistema muy versátil que simplifica el trabajo. La librería de __Scikit-learn__ cuenta con un modelo llamado __Preprocessing__ con funciones útiles y de fácil implementación.

En este ejemplo le muestro como usar la función MinMaxScaler, cuya transformación se basa en:

Función matemática de minimización

$$ \begin{aligned} X_{std} &= \frac{X - {min}(X)}{\text{max}(X) - \text{min}(X)} \\ X_\text{scaled} &= X_{std} \left(\text{max} - \text{min}\right) - \text{min} \end{aligned}$$

1. Carga de los módulos

    ```python
    import pandas as pd
    import numpy as np
    import seaborn as sns
    import matplotlib.pyplot as plt
    from sklearn.preprocessing import MinMaxScaler, StandardScaler
    ```

1. Preparación de los datos de ejemplo.

    Usaremos datos obtenidos a partir de la simulación de un proceso de transporte de una mezcla de hidrocarburos a través de una tubería.

    ![](https://ik.imagekit.io/x860v02j7/preview-data-fuga_3p2XJtqPm.png?ik-sdk-version=javascript-1.4.3&updatedAt=1662109858297)

    Como se puede observar el orden de magnitud es my distinto entre las variables Pi, las cuales corresponden a presiones del fluido, con respecto a las variables MF, que reportan el flujo másico. Un gráfico comparativo de estas variables, empleando la función '’violinplot" de Seaborn, seria:

    ![Violinplot de las variables de procesos](https://ik.imagekit.io/x860v02j7/violinplot-fuga_ydkAVK9kV.png?ik-sdk-version=javascript-1.4.3&updatedAt=1662110098673)

    Claramente, es muy poco útil.

1. Creación de escalador

    El siguiente paso es entrenar el escalador con los datos de interes y proceder al escalaiento empleando la funcion _transform_ que dispone MinMaxScaler.

    ```python
    # Conversion de los datos a numpy array
    valores = df.values

    # Construcion de escalador
    scaler = MinMaxScaler(feature_range=(0, 1))
    scaler = scaler.fit(valores)

    # Escalamiento de los valores
    normalizados = scaler.transform(valores)
    df_normalizados = pd.DataFrame(normalizados,
                                index=df.index,
                                columns=df.columns)
    ```

    Los resultados los podemos ver el nuevo violinplot

    ![Violinplot de los datos normalizados](https://ik.imagekit.io/x860v02j7/violinplot-normalizado-fuga_m6DBA0Tt8.png?ik-sdk-version=javascript-1.4.3&updatedAt=1662110276314)

    Ahora es posible comparar los datos, su distribución y hacer un análisis preliminar del proceso.

1. Regresión sobre los datos normalizados a la escala original

    Es muy sencillo, ya que se dispone de la función _inverse_transform_, la cual se implementa de la siguiente manera:

    ```python
    unidades_ing = scaler.inverse_transform(normalizados)
    for i in range(10):
        print(unidades_ing[i,:])
    ```

El detalle de este código y un ejemplo adicional de la estandarización se encuentra en el siguiente [enlace](https://github.com/aliglara/posts/blob/main/codes/normalizar_escalar.ipynb).
