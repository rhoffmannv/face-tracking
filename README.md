# Face Tracking

En este proyexto se implementa desde cero una red neuronal profunda para hacer seguimiento de caras.  
El proyecto contiene el *jupyter notebook* **Face Tracking.ipynb** con todos los pasos a seguir y explicaciones detalladas.

## Detalles del Proyecto

El proyecto se puede dividir a grandes rasgos en:

- Importación de librerías
- Creación de *dataset*
  - Captura de imágenes
  - Creación de anotaciones
  - Aumento de datos
- Preprocesamiento de datos
  - División de *dataset* por conjunto 
  - Procesamiento de imágenes
  - Procesamiento de etiquetas 
- Creación de modelo
  - Descarga de VGG16
  - Modelo de clasificación y de regresión
  - Definición de funciones de *loss*
- Entrenamiento de modelo
- Testing de modelo
- Implementación en tiempo real

## Importación de librerías

Entre las librerías utilizadas destacan:
  - TensorFlow
  - OpenCV
  - Numpy
  - LabelMe
  - Albumentations
  - Matplotlib

## Creación de dataset

### Captura de imágenes
- Se define el número de imágenes (*n_imagenes*) a generar inicialmente.
- Se crear carpeta para guardar imágenes.
- Se abre videocamara con OpenCV.
- Se capturan *n_imagenes* imágenes, una cada medio segundo.

### Creación de anotaciones
- Se crea carpeta para guardar anotaciones.
- Se usa programa LabelMe para crear manualmente las anotaciones.
  - Permite hacer rectángulo (*box*) fácilmente sobre imagen y colocarle *label* (etiqueta).
  - Guarda json por cada imagen con las coordenadas del rectángulo y el *label*.
 
### Aumento de datos
- Crear carpetas para las imágenes y las etiquetas "aumentadas".
- Uso de librería Albumentations.
  - Se crea objeto "augmentor" con parametros a usar para cambiar imagen original.
  - Transformaciones aleatorias a aplicar: cambio en tono de color, cambio en brillo, volteo horizontal, volteo vertical y recorte.
  - Se pasan también las coordenadas del rectángulo y las labels para que las modifique acorde a la transformación.  
  - Se aplica la transformación definida en "augmentor" *n_multiplicacion* veces sobre cada imagen.
  - Se obtienen *n_multiplicacion* nuevos *datapoint* por cada *datapoint* original, aplicando aleatoriamente las transformaciones definidas.
 
## Preprocesamiento de datos

### Separar en conjuntos de entrenamiento, validación y prueba
- Se crean carpetas para imagenes y anotaciones por cada conjunto.
- Se separan imagenes en conjuntos de entrenamiento, validación y prueba.
- Se mueven las imágenes a las carpetas correspondientes.
- Se mueven las etiquetas a las carpetas correspondientes.
    
### Procesamiento de imágenes
- Se cargan las imágenes desde su *path* a un tensor de TensorFlow.
- Se escalan a *120 x 120* pixeles.
- Se escalan los valores de cada pixel entre 0 y 1.

### Procesamiento de etiquetas
- Se cargan los archivos de anotaciones formato *json*.
- Se extraen la etiqueta y las coordenadas de los rectángulos y se crea en tensor a partir de ellos.

### Unir dataset
- Se unen los tensores correspondientes a las imágenes con los tensores de las etiquetas, por cada conjunto.

## Creación de modelo

### Descarga de VGG16
  - Se descarga la red neuronal convolucional VGG16 ya preentrenada, sin las capas finales.
    
### Modelo de clasificación y de regresión
El modelo consiste en dos partes:
   - Clasificación: Para determinar la etiqueta de la detección.
   - Regresión: Para predecir las coordenadas del rectángulo donde se encuentra el objeto.
   
Cada parte se basa en las *feature* entregadas por la red VGG16 y agrega capas sobre esta.
   - Clasificación: Agrega una capa de *Max Pooling*, luego una capa oculta *fully connected* y finalmente la capa de salida con una neurona y activación sigmoidea.
   - Regresión: Agrega una capa de *Max Pooling*, luego una capa oculta *fully connected* y finalmente la capa de salida con 4 neuronas de salida y función de activación sigmoidea.

### Definir optimizador y paramétros de entrenamiento
  - Se usa Adam como optimizador.
  - Se definen parámetros como el *learning rate*.
    
### Función de loss para regresión
Se usa la función de pérdida:

$$loss = \sum (x - \hat{x})^2 + (y - \hat{y})^2 + (w - \hat{w})^2 + (h - \hat{h})^2$$

> Donde *x* corresponde a la primera coordenada del punto superior izquierdo del rectángulo.  
> Donde *y* corresponde a la segunda coordenada del punto superior izquierdo del rectángulo.  
> Donde *w* corresponde al ancho del rectángulo.  
> Donde *h* corresponde al alto del rectángulo.  
> El "sombrero" hace referencia a la predicción.  

### Función de loss para clasificación
- Se usa Entropía cruzada para la pérdida de clasificación.

## Entrenamiento del modelo

### Creación de clase Modelo personalizada

Requiere definir los métodos:
- **compile**: Para declarar las funciones de *loss* y el optimizador a utilizar para entrenar.
- **train_step**: Para ajustar los pesos del modelo usando gradiente descendiente. Notar que se usa como función de *loss* una suma ponderada del *loss* de regresión y el de clasificación.
- **test_step**: Para realizar una predicciones sobre el conjunto de *test*. Guarda las *losses* en un diccionario.
- **call**: Para hacer predicciones con el modelo, se llama con el método *predict*.

### Ajuste de pesos
- Se define el directorio para el *callback* de Tensorboard (Permite ver los resultados del entrenamiento en tiempo real).
- Se ejecuta el método *fit* para ajustar los pesos con los datos de entrenamiento.
- Se ingresa el conjunto de validación para ir calculando el *loss* tanto en entrenamiento como en validación para poder evaluar si existe *overfitting* o *underfittting*.

### Gráficos de losses en entrenamiento y en validación
- Se hacen 3 gráficos de *losses*: *loss* total, *loss* de clasificación y *loss* de regresión.
- Por cada uno se grafican tanto el *loss* de entrenamiento como el *loss* de validación.

### Guardar modelo
- Se guardan los pesos del modelo entrenado para poder cargarlos posteriormente.

## Probar el modelo

###  Predicciones en el conjunto de prueba
- Se carga un batch de conjunto de prueba y se hace predicción con el modelo obtenido.
- Se grafican 4 imágenes del batch de prueba y se dibuja encima el rectángulo predicho por el modelo.

## Implementación del modelo

- Se abre videocamera con OpenCV.
- Se toman los *frame* y se recortan, tomando los primeros *450 x 450* pixeles.
- Se reescala y normaliza el *frame* para poder ingresarlo al modelo.
- Se realiza predicción con el modelo.
- Se dibuja rectangulo con cv2.rectangle y label con cv2.putText.
- Se muestra el *frame* modificado en pantalla.
- Salir presionando "q".
