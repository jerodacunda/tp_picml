# Pre-analysis
--------------

## Introducción

En este proyecto buscamos replicar una parte del paper 'Brain age predicts mortality' (https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5984097/). Este paper consta de dos partes: en la primera se muestra una forma de predecir la edad de pacientes a partir de neuroimágenes, la cual llama 'brain-predicted age'; y en la segunda parte muestra una asociación entre el hecho de que la 'brain-predicted age' diste de la edad real, y la mortalidad u otros aspectos de deterioro físico y mental. Nos enfocaremos en la primera parte y replicaremos la prediccion de edad planteada, utilizando un dataset a nuestro alcance y siguiendo las indicaciones del paper. Nuestra hipotesis es que será capaz de predecir la edad ya que en el paper se muestran resultados que sostienen esta teoría, prediciendo la edad exitosamente. Además, la replicabilidad es una característica clave en el análisis de datos, por lo tanto si el modelo es bueno sería razonable que obtengamos resultados similares a pesar de utilizar un dataset diferente.

## Métodos

### Dataset

El dataset que usaremos será el AOMIC-ID1000. Nos concentraremos en las imágenes MRI de peso T1. En particular utilizaremos las imágenes fMRIprep, las cuales están preprocesadas y por lo tanto normalizadas espacialmente.
Específicamente tomaremos para cada sujeto las imágenes:

* {sujeto}_space-MNI152NLin2009cAsym_label-GM_probseg.nii.gz
* {sujeto}_space-MNI152NLin2009cAsym_label-WM_probseg.nii.gz

las cuales están situadas en AOMIC-ID1000/derivatives/fmriprep/sub-(i)/anat.



### Features

De las imágenes mencionadas trabajaremos con las segmentaciones de materia blanca (WM) y materia gris (GM), las cuales remarcamos que ya se encuentran normalizadas espacialmente. A éstas le vamos a aplicar una serie de parcelaciones, a las cuales vamos a extraer el promedio (mean) y desviación estándar (std). Luego de explorar diferentes parcelaciones, seleccionamos las siguientes:
- **Schaefer**: Esta es una opción sólida porque está basada en redes funcionales y ha sido validada en numerosos estudios. Captura la organización funcional del cerebro en diferentes resoluciones. Utilizaremos Schaefer100x7 y Schaefer200x7.
- **Brainnetome**: Es un atlas anatómico *cross-validated* basado en conectividad, con información de las conexiones anatómicas y funcionales. Utilizaremos Brainnetome246.


### Modelo

Definiremos el modelo de acuerdo a lo indicado en el paper. En éste se utiliza un Gaussian Process regression para poder predecir las edades. El mismo será entrenado con los features mencionados y los datos de los sujetos encontrados en el dataset, generando una predicción de la edad como salida. Para el entrenamiento, el GPR toma como parámetro un kernel que define su función de covarianza. Este kernel toma a su vez uno o más hiperparámetros que se van optimizando durante el entrenamiento. El kernel puede definirse como combinación de distintos kernels. En nuestro caso, lo definiremos combinando:
- **Constant kernel**: se usa como factor de un kernel *producto* y escala la magnitud del otro factor. El hiperparámetro a optimizar es una constante iniciada en 1 y define la covarianza.
- **RBF**: los Gaussian Process con este kernel como función de covarianza permiten derivar los cuadrados medios absolutos de todos los órdenes. Su hiperparámetro a optimizar es la *lenght_scale*, que permite definir cómo escalan las dimensiones de los datos y también está inicializado en 1.
  
Entonces nuestro kernel queda definido así (los parámetros "fixed" no se optimizan durante el entrenamiento): 
- *ConstantKernel(1.0, constant_value_bounds="fixed") * RBF(1.0, length_scale_bounds="fixed")*

Para explorar sobre los valores y hallar los óptimos, vamos a utilizar grid search, es decir, plantear todas las combinaciones posibles.
  


### Evaluación y Testeo
Para definir los conjuntos de training y test data, nos basamos en el N elegido en el paper, el cual toma aproximadamente el 75% de los datos para entrenamiento y el resto para testeo. Considerando que hay 928 sujetos en nuestro dataset, elegimos un N=696 para entrenamiento y N=232 para test data.

Luego lo validaremos utilizando una 10-fold cross validation aplicada a los datos de entrenamiento. En esta validación se va a comparar la edad predicha y la edad verdadera. 

Finalmente pondremos a prueba el modelo  utilizando al conjunto de datos de prueba. Así generamos una prediccion sobre la edad de estos últimos, para luego compararla con la edad real y analizar qué tan bien funcionó. 

En la figura 1 del paper pueden verse en más detalle los pasos del modelo.
