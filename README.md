# Proyecto de Recomendación basado en Clasificación

## Introduccion:


La empresa se encuentra en la fase de planificación para lanzar una estrategia dirigida a su base de clientes al final del mes. El propósito fundamental de esta iniciativa es personalizar las interacciones con los clientes al enviarles el producto que más se ajuste a sus necesidades, al mismo tiempo que se busca maximizar las ganancias.

En términos de ganancias, la empresa ha establecido que obtiene 10€ por cada cuenta vendida. Además, se han identificado dos categorías clave de productos con sus respectivas ganancias: 40€ por productos relacionados con ahorro e inversión, y 60€ por productos de financiación, que incluyen préstamos y tarjetas.

El alcance de esta acción se ve delineado por un presupuesto específico, con la capacidad de enviar alrededor de 10,000 correos electrónicos. Con el objetivo claro de optimizar los resultados, se busca no solo maximizar la tasa de respuesta de los clientes, sino también calcular el Retorno de la Inversión (ROI) asociado con esta campaña.

En cuanto a las acciones solicitadas, se ha expresado la necesidad de prever la tasa de respuesta antes de enviar los correos electrónicos. Este enfoque proactivo permitirá afinar la estrategia de manera anticipada, aumentando la probabilidad de éxito y optimizando el impacto de la acción planificada.


## Planteamiento de resolución:

Debido a la necesidad de preveer la tasa de respuesta, vamos a desarrollar modelos de clasificación para cada producto y obtener la probabilidad de pertenecer a la clase 1 (contrata el producto). Esta probabilidad la recalibraremos para poder usarla como *tasa de respuesta*. 

Una vez tengamos las tasas de respuesta, las ponderaremos por el retorno que se espera de cada producto para hacer una clasificación para cada usuario - producto.

Con todos estos datos obtendremos:
- los 10.000 usuarios con mayor tasa de retorno/beneficio a los que recomendaremos productos.
- la cantidad de cada uno de los productos que recomendaremos, con us tasa de respuesta esperado e ingreso por tipo de producto.
- el ingreso total esperado de la campaña.

  
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


## Selección de modelos

Al tener en su mayoría que realizar clasificaciones con datos desequilibrados, seleccionaremos dos modelos de clasificación binaría los cuales tienen un parametro para tratar con este tipo de datos (Árbol de decisión y Bosques Aleatorios). 

## Entrenamiento:

Centraremos el entrenamiento en 10 de los 15 productos debido a las pocas observaciones de productos contratados que hay para 5 de los 15 productos.

Realizaremos una busqueda del mejor modelo con sus mejores hiperparametros usando Ramdom Search evaluado por la precisión de los modelos. El motivo de usar esta metrica se debe a la importancia de detectar aquellos clientes que realmente estén interesados en contratar el producto.

**Recalibrado:**

Una vez tenemos los distintos modelos con los distintos hiperparametros, realizaremos un recalibrado para obtener las probabilidades reales.

Para aquellos casos en los que los datos están más desbalanceados, el recalibrado está funcionando peor por norma general, no mejorando mucho los modelos originales.

<table>

 <caption style="text-align: center;">Recalibrado con datos muy desbalanceados</caption>
  <tr>
    <td><img src="/images/modelo_b.png" alt="Analisis AVProductInstalled" ></td>
    <td><img src="/images/felcha.png" alt="Analisis AVProductInstalled" width="200px"></td>
    <td><img src="/images/modelos_b_recalibrado.png" alt="Analisis AVProductInstalled"></td>    
  </tr>
</table>

<table>
  <caption style="text-align: center;">Recalibrado con datos muy desbalanceados</caption>
  <tr>
    <td><img src="/images/modelo_a.png" alt="Analisis AVProductInstalled"></td>
    <td><img src="/images/felcha.png" alt="Analisis AVProductInstalled" width="200px"></td>
    <td><img src="/images/modelos_a_recalibrado.png" alt="Analisis AVProductInstalled"></td>
  </tr>
</table>

## Resultado:

A nivel de usuario, nos quedaremos solo con aquellos usuarios que en la última imagen no habían contratado el producto, debido a que la probabilidad de que un usuario siga contratando un producto es cercana al 100%.

Multiplicaremos la probabilidad de cada uno de los productos/usuario por la precisión de cada uno de los modelos/producto para a continuación asignar el beneficio por producto. Obtendremos así una puntuación que tenga en cuenta la probabilidad de que se compre un producto y el ingreso de ese producto.

Seleccionaremos para cada usuario el producto con mayor puntuación y ordenaremos los usuarios por la puntuacion de los productos seleccionados, de modo que aquellos usuarios con mayor puntuación serán los 10.000 seleccionados para ser contactados.

Obtendremos los ingresos esperados para cada uno de los productos, multiplicando la probabilidad de compra de los distintos productos por el ingreso por porducto.







probabilidad por su su beneficio esperado para obtener la puntuación de cada uno de los productos para cada usuario. Seleccionaremos el producto para cada usuario con mayor puntuación y los ordenaremos por puntuación para quedarnos con las top 10000 puntuaciones.
