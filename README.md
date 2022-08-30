#Crea y ejecuta un contenedor Docker para un modelo de ML
La idea es hacer una construcción rápida y fácil de un contenedor Docker con un modelo simple de aprendizaje automático y ejecutarlo. Para comenzar a construir un contenedor Docker para un modelo de aprendizaje automático, consideremos tres archivos:

Dockerfile
entrenar.py
inferencia.py
El entrenar.py es un script de python que ingiere y normaliza los datos de EEG en un archivo csv (entrenar.csv) y entrena dos modelos para clasificar los datos (usando scikit-learn). El script guarda dos modelos: análisis discriminante lineal (clf_lda) y perceptrón multicapa de redes neuronales (clf_NN).

Se llamará a inferencia.py para realizar la inferencia por lotes cargando los dos modelos que se han creado previamente. La aplicación normalizará los nuevos datos de EEG provenientes de un archivo csv (test.csv), realizará inferencias en el conjunto de datos e imprimirá la precisión de la clasificación y las predicciones.

