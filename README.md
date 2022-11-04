### Modelo de predicción de precio de propiedades mediante análsis de Lenguaje Natural (NPL)

### NPL 
___
El Procesamiento del Lenguaje Natural (NLP por sus siglas en inglés) es el campo de estudio que se enfoca en la comprensión mediante ordenador del lenguaje humano. 

### Definición del proyecto
___
Una importante empresa inversora dentro del rubro de la inmobiliaria en Colombia, nos pide implementar un modelo de clasificación que permita clasificar el precio de las propiedades en venta, utilizando los datos que se han puesto a su disposición correspondientes al año 2020. ​Para esto, específicamente, debemos predecir la categorización de las propiedades entre baratas o caras, considerando como criterio el valor promedio de los precios (la media).​

### Recolección de datos 
___
​Se proveen los archivos dentro del archivo comprimido 'properties_colombia.zip':

'properties_colombia_train.csv': Contiene 197549 registros y 26 dimensiones, el cual incluye la información numérica del precio.
'propiedades_colombia_test.csv': Contiene 65850 registros y 25 dimensiones, el cual no incluye la información del precio.

### Descripción de las dimesiones
___
- id - Identificador del aviso. No es único: si el aviso es actualizado por la inmobiliaria (nueva versión del aviso) se crea un nuevo registro con la misma id pero distintas fechas: de alta y de baja.
- ad_type - Tipo de aviso (Propiedad, Desarrollo/Proyecto).
- start_date - Fecha de alta del aviso.
- end_date - Fecha de baja del aviso.
- created_on - Fecha de alta de la primera versión del aviso.
- lat - Latitud.
- lon - Longitud.
- l1 - Nivel administrativo 1: país.
- l2 - Nivel administrativo 2: usualmente provincia.
- l3 - Nivel administrativo 3: usualmente ciudad.
- l4 - Nivel administrativo 4: usualmente barrio.
- l5 - Nivel administrativo 5.
- l6 - Nivel administrativo 6.
- rooms - Cantidad de ambientes.
- bedrooms - Cantidad de dormitorios (útil en el resto de los países).
- bathrooms - Cantidad de baños.
- surface_total - Superficie total en m².
- surface_covered - Superficie cubierta en m².
- price - Precio publicado en el anuncio.
- currency - Moneda del precio publicado.
- price_period - Periodo del precio (Diario, Semanal, Mensual)
- title - Título del anuncio.
- description - Descripción del anuncio.
- property_type - Tipo de propiedad (Casa, Departamento, PH).
- operation_type - Tipo de operación (Venta).
- geometry - Puntos geométricos formados por las coordenadas latitud y longitud.

### Metodología
____
Luego de explorar los datos proporcionados, notamos que las columnas con menos campos vacíos en ambos dataset proporcionados son 'title' y 'description', por tanto concluí aplicar un modelo de NPL que aproveche toda la información contenida en el texto de los anuncios para identificar patrones en el lenguaje natural que nos permita predecir y clasificar si una propiedad está cara o barata, tomando como referencia el precio promedio de las propiedades. 

Existen dos corrientes de aplicación de NPL, la clásica y la conocida como Deep Learning que emplea redes neuronales. Este proyecto lo abordaremos desde la postura clásica. 

[![NPL](https://www.xenonstack.com/hs-fs/hubfs/deep-learning-nlp-applications.png?width=1280&name=deep-learning-nlp-applications.png "NPL")](https://www.xenonstack.com/hs-fs/hubfs/deep-learning-nlp-applications.png?width=1280&name=deep-learning-nlp-applications.png "NPL")

# Importación y Limpieza de datos

En esta etapa importamos los dataset y analizamos su contenido. Había una gran proporción de datos nulos en los datos, salvo en las columnas 'title' y 'description' por esta razón, el proceso de limpieza se concentró en estas columnas. 

![enter image description here](https://github.com/LilaAlvesDC/P2-Prediccion-Precio-Casas-Machine-Learning/blob/main/_str/datos_nulos_train.JPG)

El proceso consistió en 
1. Convertir los precios expresados en USD a COP
2. Concatenar las columnas 'title' y 'description' en una sola llamada 'text'
3. Eliminar las filas que tuvieran datos nulos en la columna 'price'

## Eliminar outliers 

Para eliminar los outliers o valores atípicos extrajimos los datos dentro del rango intercuartílico de percentil 10. Como sigue: 

    Q1, Q2  =  percentile(dataset['price'], 10), percentile(dataset['price'], 90)

Determinamos el límite inferior: 

$$ q \lt  Q_1  -  1.5 * IQR $$

y el límite superior

$$ q \gt  Q_1  -  1.5 * IQR $$

Los valores de la columnas 'price' después de remover de los outliers 

![enter image description here](https://github.com/LilaAlvesDC/P2-Prediccion-Precio-Casas-Machine-Learning/blob/main/_str/Sin_outliers.png?raw=true)


[Repositorio original de consigna](https://github.com/soyHenry/Datathon "Repositorio de consigna")
