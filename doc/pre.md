# Pre-analysis
--------------

## Introducción

En este proyecto buscamos replicar una parte del paper 'Brain age predicts mortality' (https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5984097/). Este paper consta de dos partes: en la primera se muestra una forma de predecir la edad de pacientes a partir de neuroimágenes, la cual llama 'brain-predicted age'; y en la segunda parte muestra una asociación entre el hecho de que la 'brain-predicted age' diste de la edad real, y la mortalidad u otros aspectos de deterioro físico y mental. Nos enfocaremos en la primera parte y replicaremos la prediccion de edad planteada, utilizando un dataset a nuestro alcance y siguiendo las indicaciones del paper. Nuestra hipotesis es que será capaz de predecir la edad ya que en el paper se muestran resultados que sostienen esta teoría, prediciendo la edad exitosamente. Además, la replicabilidad es una característica clave en el análisis de datos, por lo tanto si el modelo es bueno sería razonable que obtengamos resultados similares a pesar de utilizar un dataset diferente.

## Métodos

### Dataset

El dataset que usaremos será el AOMIC-ID1000. Nos concentraremos en las imágenes MRI de peso T1. En particular utilizaremos las imágenes fMRIprep, las cuales están preprocesadas y por lo tanto normalizadas espacialmente.
Específicamente tomaremos para cada sujeto i las imágenes sub-(i)_desc-preproc_T1w.nii.gz situada en AOMIC-ID1000/derivatives/fmriprep/sub-(i)/anat

### Features

De las imágenes mencionadas trabajaremos con las segmentaciones de materia blanca (WM) y materia gris (GM), las cuales remarcamos que ya se encuentran normalizadas espacialmente. A éstas le vamos a aplicar una parcelación y extraer el promedio (mean) y desviación estándar (std). 

### Modelo

Definiremos el modelo de acuerdo a lo indicado en el paper. En este se utiliza un Gaussian Process regression para poder predecir las edades. El mismo será entrenado con los features mencionados y los datos de los sujetos encontrados en el dataset, generando una predicción de la edad como salida. 


### Evaluación y Testeo
Para definir los conjuntos de training y test data, nos basamos en el N elegido en el paper, el cual toma aproximadamente el 75% de los datos para entrenamiento y el resto para testeo. Considerando que hay 928 sujetos en nuestro dataset, elegimos un N=696 para entrenamiento y N=232 para test data.

Luego lo validaremos utilizando una 10-fold cross validation aplicada a los datos de entrenamiento. En esta validación se va a comparar la edad predicha y la edad verdadera. 

Finalmente pondremos a prueba el modelo  utilizando al conjunto de datos de prueba. Así generamos una prediccion sobre la edad de estos últimos, para luego compararla con la edad real y analizar qué tan bien funcionó. 

En la figura 1 del paper pueden verse en más detalle los pasos del modelo.
