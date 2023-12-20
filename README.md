# Proyecto de Recomendación basado en Clasificación

## Datos:

Partimos con 3 datasets.
- datos sociodemograficos
- datos de actividad_comercial
- datos de interacciones de los distintos usuarios con los 15 productos de la compañía a lo largo de 17 meses

## Transformación de los datos:

**Tratamiento de *nan*:**

Los *nan* se encuentran mayormente en las dos primeras tablas. Al contar con 17 particiones se opta por completar los valores faltantes con los datos para los usuarios presentes en el resto de particiones. En los casos en los que no tuviera registro en otras particiones se genera una nueva categoría de *“OTROS”* para no perder información eliminando registros o imputando valores como la media o moda.

En el salary se opta por hacer binning en distintas categorías, añadiendo una categoría de *“Salario no informado”*.

Se eliminan dos registros que no indican el *género*.

En los productos hay *nan* en *pension_plan y payroll*. Se trata de los mismos usuarios. Se eliminan 63 registros.

**Ingeniería de variables:**

El principal reto es aplanar los datos de las interacciones de los usuarios con los productos, para pasar de una serie temporal (17 meses) a distintos hitos para cada uno de los usuarios.

De los datos se extraen las siguientes nuevas variables:
- total de meses que cada cliente ha contratado cada producto
- total productos contratados en la última imagen
- total de productos contratados a lo largo de los 17 meses
- total de altas para el conjunto de productos
- se crean 3 categorías de productos (*ahorro e inversión, financiación y servicios*) y para cada una de ellas se computa el total de productos por cliente, así como el total de productos por cliente en la última imagen
- por último, se extrae la situación final de cada cliente para los distintos productos, que a la hora de realizar el entrenamiento de cada algoritmo, se extrae la última imagen del producto que se está entrenando, ya que hay un gran % de casos en los que si se tiene contratado el producto no se da de baja el mes siguiente y sea la variable de mayor importancia para el modelo.

Con la fecha de entrada se calcula la variable antigüedad.

**Codificación de variables:**

Se hace OHE en la variable *entry_channel, segment y salary*.

**Preparación X e y para entrenar:**

Para entrenar el modelo usaremos las primeras 16 particiones, fijando la imagen de productos en la partición 17 como la etiqueta.

**Preparación DF para predicciones:**

Para hacer el X debemos de tomar las 17 particiones y realizar todas las transformaciones anteriores de modo que incorpore los cambios realizados en la última de las particiones.

## Entrenamiento:

Generamos modelos para 10 de los productos debido a las pocas observaciones de productos contratados que hay para 5 de los productos.

Para entrenar y evaluar los modelos nos centraremos en la métrica de *precisión* ya que el recomendar un producto tiene un coste directo asociado y queremos primar el retorno de la inversión.

**Recalibrado:**

Cada uno de los 15 modelos tiene una métrica distinta de *precisión* con lo que debemos de recalibrar los resultados. Para ello usaremos un modelo de calibración sobre el modelo base. `CalibratedClassifier`

## Resultado:

Para cada uno de los productos nos quedaremos con aquellos que tienen unas predicciones superiores al 50%. A nivel de usuario, nos quedaremos solo con aquellos usuarios que en la última imagen no habían contratado el producto.

Por último, multiplicaremos cada probabilidad por su beneficio esperado para obtener la puntuación de cada uno de los productos para cada usuario. Seleccionaremos el producto para cada usuario con mayor puntuación y los ordenaremos por puntuación para quedarnos con las top 10000 puntuaciones.
