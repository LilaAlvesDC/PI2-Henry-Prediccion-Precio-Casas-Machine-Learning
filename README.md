### Modelo de predicción de precio de propiedades mediante análsis de Lenguaje Natural (NLP)

### NLP 
___
El Procesamiento del Lenguaje Natural (NLP por sus siglas en inglés) es el campo de estudio que se enfoca en la comprensión mediante ordenador del lenguaje humano. 

### Definición del proyecto
___
Una importante empresa inversora dentro del rubro de la inmobiliaria en Colombia, nos pide implementar un modelo de clasificación que permita clasificar el precio de las propiedades en venta, utilizando los datos que se han puesto a su disposición correspondientes al año 2020. Para esto, específicamente, debemos predecir la categorización de las propiedades entre baratas o caras, considerando como criterio el valor promedio de los precios (la media).

### Recolección de datos 
___
Se proveen los archivos dentro del archivo comprimido 'properties_colombia.zip':

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
___
En esta etapa importamos los dataset y analizamos su contenido. Había una gran proporción de datos nulos en los datos, salvo en las columnas 'title' y 'description' por esta razón, el proceso de limpieza se concentró en estas columnas. 

![enter image description here](https://github.com/LilaAlvesDC/P2-Prediccion-Precio-Casas-Machine-Learning/blob/main/_str/datos_nulos_train.JPG)

El proceso consistió en 
1. Convertir los precios expresados en USD a COP
2. Concatenar las columnas 'title' y 'description' en una sola llamada 'text'
3. Eliminar las filas que tuvieran datos nulos en la columna 'price'

## Eliminar outliers 
___
Para eliminar los outliers o valores atípicos extrajimos los datos dentro del rango intercuartílico de percentil 10. Como sigue: 

    Q1, Q2  =  percentile(dataset['price'], 10), percentile(dataset['price'], 90)

Determinamos el límite inferior: 

$$ q \lt  Q_1  -  1.5 * IQR $$

y el límite superior

$$ q \gt  Q_1  -  1.5 * IQR $$

Los valores de la columnas 'price' antes de remover los outliers 

![enter image description here](https://github.com/LilaAlvesDC/P2-Prediccion-Precio-Casas-Machine-Learning/blob/main/_str/Con_outliers.png?raw=true)

Los valores de la columnas 'price' después de remover de los outliers 

![enter image description here](https://github.com/LilaAlvesDC/P2-Prediccion-Precio-Casas-Machine-Learning/blob/main/_str/Sin_outliers.png?raw=true)

Los outliers identificados y removidos representaron el 0.0282 % de los datos, luego considerando los datos nulos, fueron descartados 2.85 % de los datos proporcionados. 


## Crear columna 'target'
___
Luego de unificar la divisa en que estaban expresados los precios, resolvimos calcular el precio promedio de las propiedades para crear la columna 'target'. 

La columna target etiqueta nuestra propiedades como cara o barata, asignando 1 o 0, respectivamente, tomando como referencia el precio promedio de las propiedades, de la siguiente manera:  

    dataset['target'] =  0
    
    for  i  in  range(len(dataset['price'])):
	    precio  =  dataset.loc[i, 'price']
		    if  precio  >=  precio_promedio:
			    dataset.loc[i, 'target'] =  1

Finalmente los datos de entrenamiento están distribuídos de la siguiente manera: 

![enter image description here](https://github.com/LilaAlvesDC/P2-Prediccion-Precio-Casas-Machine-Learning/blob/main/_str/Distribuci%C3%B3n%20de%20los%20datos%20.png?raw=true)

# Vectorización 
___
A continuación, vectorizamos el vocabulario de nuestros datos, este proceso consiste en asignar un índice a cada palabra y luego un índice a cada fila de datos, para construir la matriz de dispersión que posteriormente usaremos para entrenar nuestro modelo. 

Se utilizan las funciones CountVectorizer() para vectorizar y transform() para crear la matriz dispersión. 

    cv_tranformer  =  CountVectorizer(analyzer=text_processing).fit(data['text'])

    bolsa_palabras  =  cv_tranformer.transform(data['text'])
 

# TF - IDF 
---
(del inglés _Term frequency – Inverse document frequency_) frecuencia de término – frecuencia inversa de documento (o sea, la frecuencia de ocurrencia del término en la colección de documentos), es una medida numérica que expresa cuán relevante es una palabra para un documento en una colección.

    tfidf_transformer  =  TfidfTransformer().fit(bolsa_palabras(data))

***Tf-idf*** es el producto de dos medidas, frecuencia de término y frecuencia inversa de documento. Se calcula cómo: 


$${\displaystyle \mathrm {tfidf} (t,d,D)=\mathrm {tf} (t,d)\times \mathrm {idf} (t,D)}$$

donde la *frecuencia de término* es:

$${\displaystyle \mathrm {tf} (t,d)={\frac {\mathrm {f} (t,d)}{\max\{\mathrm {f} (t,d):t\in d\}}}}$$

y la *frecuencia inversa de documento* es:

$${\displaystyle \mathrm {idf} (t,D)=\log {\frac {|D|}{|\{d\in D:t\in d\}|}}}$$

## Entrenamiento del modelo 

Escogimos dos modelos de clasificación para predecir, estos son el *Modelo Naive Bayes* y el modelo *Gradiente Descendente Estocástico para Clasificación*. 

El *Modelo Naive Bayes* se basa en el teorema de Naive Bayes, donde se asume que laque las variables predictoras son independientes entre sí. En otras palabras, que la presencia de una cierta característica en un conjunto de datos no está en absoluto relacionada con la presencia de cualquier otra característica.

La fórmula del teorema de Bayes es: 

$$ {\displaystyle P(A_{i}|B)={\frac {P(B|A_{i})P(A_{i})}{P(B)}}} $$

El *Gradiente Descendente Estocástico para Clasificación*  implementa una rutina de aprendizaje de descenso de gradiente estocástico simple que admite diferentes funciones de pérdida y penalizaciones para la clasificación. la función SGDClassifier admite la clasificación de clases múltiples mediante la combinación de varios clasificadores binarios en un esquema de "uno contra todos" (OVA).


[Repositorio original de consigna](https://github.com/soyHenry/Datathon "Repositorio de consigna")
