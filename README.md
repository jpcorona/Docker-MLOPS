Mlflow es un proyecto open source que facilita la gestión de modelos de machine learning incluyendo la experimentación, la reproducibilidad así como el despliegue de los mismos. El proyecto fue desarrollado por el equipo creador de spark y está disponible como un módulo de python el cuál es muy sencillo instalar mediante la siguiente sentencia:

$ pip install mlflow==1.5.0


MLFlow esta compuesto de tres componentes principales:

MLflow tracking: Es una componente para loggeo de métricas, parámetros, versiones de código, así como archivos de salida de las ejecuciones de nuestro código. Existen clientes para Python, R y Java.
MLflow projects: Es una estructura de archivos que permiten organizar Y gestionar el modelo de machine learning de manera reproducible. Consiste en tres archivos de salida generados: conda.yaml, MLproject y model.pkl
MLflow models: Es un estándar de empaquetado para la ejecución de un modelo y consumo a traves de una REST API. Librerias y frameworks soportados: sklearn, h20, mleap, keras, tensorflow, onnx, etc.
A su vez mlflow se apoya de otras componentes adicionales para poder funcionar las cuales son: backend-store (almacenamiento de metricas) y default-artifact-root (archivos y modelos resultados).

La backend-store puede ser de los siguientes tipos: archivos planos, postgres, mysql, etc.

La default-artifact puede ser de los siguientes tipos: archivos locales, azure blob storage, aws s3, ftp, etc.

Toda la documentación de mlflow se puede encontrar en la siguiente liga:

https://mlflow.org/docs/latest/index.html

Docker: Proyecto open source que habilita el despliegue de contenedores de Linux siendo una alternativa muy eficiente en muchos casos a la virtualización de máquinas completas.


Docker compose: Es una herramienta para definir aplicaciones multi-contenedor mediante un archivo en formato yaml.

https://docs.docker.com/compose/

Ya que contamos con una base teórica de las componentes que se utilizarán en el proyecto, comencemos.

Docker compose: Es una herramienta para definir aplicaciones multi-contenedor mediante un archivo en formato yaml.

https://docs.docker.com/compose/

Ya que contamos con una base teórica de las componentes que se utilizarán en el proyecto, comencemos.

Arquitectura de proyecto:

![image](https://user-images.githubusercontent.com/30583333/187587363-31f73e8e-ea70-483e-bbdc-fdf55a90e505.png)


#Paso 1: Levantar un entorno inicial mediante docker compose.
Para poder levantar este entorno necesitamos los siguientes componentes:

El entorno de desarrollo de jupyter

MLflow server: Gestión de modelos.

Postgres: Base de datos de apoyo para mlflow server.

FTP server: Servidor de archivos de apoyo para mlflow server. Nos basamos en la imagen de https://github.com/mauler/docker-simple-ftp-server

Portainer: Contenedor con UI de gestión del entorno de docker.

Jupyter: Para el entorno de jupyter nos basaremos en la imagen especificada en este repo: https://github.com/jupyter/docker-stacks, le agregamos mlflow así como el notebook con el modelo ejemplo.
