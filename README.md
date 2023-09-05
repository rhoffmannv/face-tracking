# Face Tracking

En este proyexto se implementa desde cero una red neuronal profunda para hacer seguimiento de caras.  
El proyecto contiene el *jupyter notebook* **Face Tracking.ipynb** con todos los pasos a seguir y explicaciones detalladas.

# Detalles del Proyecto

El proyecto se puede dividir a grandes rasgos en:

- Importación de librerías
  - TensorFlow
  - OpenCV
  - Numpy
  - LabelMe
  - Albumentations
  - Matplotlib
- Creación de dataset
  - Captura de imágenes
  - Creación de anotaciones
  - Aumento de datos
- Preprocesamiento de datos
- Creación de modelo
  - Descarga de VGG16
  - Modelo de clasificación y de regresión
- Entrenamiento de modelo
- Testing de modelo
- Implementación en tiempo real
