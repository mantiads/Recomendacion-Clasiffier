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

Los modelos identifican que las variables más importantes para predecir son el comportamiento con el grupo asignado por tipo de producto ya sea por historico como por ultima imagen así como productos con los que tengan una gran aparición de contrataciones conjuntas.

<img src="/images/importancia.png" alt="Analisis AVProductInstalled" width="600px">

**Recalibrado:**

Una vez tenemos los distintos modelos con los distintos hiperparametros, realizaremos un recalibrado para obtener las probabilidades reales.

Este proceso ha mejorado la preción de 8 de los 10 modelos, para los dos restantes se ha mantenido el mismo resultado.

<table>
  <tr>
    <td><img src="/images/sin calibrar.png" alt="Analisis AVProductInstalled"></td>
    <td><img src="/images/felcha.png" alt="Analisis AVProductInstalled" width="150px"></td>
    <td><img src="/images/calibrado.png" alt="Analisis AVProductInstalled"></td>    
  </tr>
  <tr>
    <td><img src="/images/sin_calibrar_metricas.PNG" alt="Analisis AVProductInstalled"></td>
    <td><img src="/images/felcha.png" alt="Analisis AVProductInstalled" width="150px"></td>
    <td><img src="/images/calibrado_metricas.PNG" alt="Analisis AVProductInstalled"></td>
  </tr>
</table>

**Desempeño modelos tras el recalibrado:**

Una vez realizado el recalibrado, las metricas de evaluación de los distintos modelos son las siguientes.

<img src="/images/Desempeno modelos.PNG" alt="Analisis AVProductInstalled">

Se observa un desempeño caso perfecto en el modelo de em_acount. Esto se debe a que es el producto mayormente contratado. Es el producto estrella. Los clientes que lo contratan son clientes que lo hacen desde un inicio¡. El 60 % de los clientes lo tienen contratado desde un inicio. Esto hace que tenga un gran impacto en las variables de total ultima imagen, servicios, número de servicios y total cuentas. 

## Resultado:

A nivel de usuario, nos quedaremos solo con aquellos usuarios que en la última imagen no habían contratado el producto, debido a que la probabilidad de que un usuario siga contratando un producto es cercana al 100%.

Multiplicaremos la probabilidad de cada uno de los productos/usuario por la precisión de cada uno de los modelos/producto para a continuación asignar el beneficio por producto. Obtendremos así una puntuación que tenga en cuenta la probabilidad de que se compre un producto y el ingreso de ese producto.

Seleccionaremos para cada usuario el producto con mayor puntuación y ordenaremos los usuarios por la puntuacion de los productos seleccionados, de modo que aquellos usuarios con mayor puntuación serán los 10.000 seleccionados para ser contactados.

<img src="/images/usuarios_prod recomendados.PNG" alt="Analisis AVProductInstalled" width="200px">

Obtendremos los ingresos esperados para cada uno de los productos, multiplicando la probabilidad de compra de los distintos productos por el ingreso por producto.

A continuación se muestra un cuadro resumen final con datos por producto de total de veces recomendado, ingreso unitario, Tasa de Respuesta e Ingrsos totales esperados por producto.

<img src="/images/resultado.PNG" alt="Analisis AVProductInstalled">

El ingreso total esperado de la campaña será la suma de los ingresos totales esperados por producto. 

## TOTAL DE INGRESOS ESPERADOS CAMPAÑA = 147819 €

