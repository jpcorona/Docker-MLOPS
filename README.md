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

Dockefile jupyter modificado
=
FROM jupyter/datascience-notebook:9b06df75e445

WORKDIR /home/jp/

COPY requirements.txt requirements.txt
COPY ejemplo.ipynb ejemplo.ipynb

RUN pip install -r requirements.txt

EXPOSE 8888

Archivo de requirements de python
=
mlflow==1.5.0

seaborn==0.9.0

scikit-learn==0.21.2

pysftp==0.2.9

MLFlow server
=
Para el entorno de mlflow nos basamos en una imagen de python y le agregamos mlflow.
FROM  python:3.7.5-slim

COPY requirements.txt requirements.txt

RUN pip install -r requirements.txt

EXPOSE 5000

ENTRYPOINT sleep 25 && mlflow server --backend-store-uri postgresql://mlflow_user:mlflow@postgres/mlflow_db --default-artifact-root ftp://test:test@ftpd/home/test/ --host 0.0.0.0

requirements.txt para mlflow server
=
mlflow==1.5.0
psycopg2-binary==2.8.4


Dockerfile para postgres
=
FROM postgres:latest

COPY init.sql /docker-entrypoint-initdb.d/

init.sql
=
CREATE DATABASE mlflow_db;
CREATE USER mlflow_user WITH PASSWORD 'mlflow';
GRANT ALL PRIVILEGES ON DATABASE mlflow_db TO mlflow_user;


Docker Compose
=
Una vez que tenemos las imagenes construidas, pasamos a levantar el entorno con docker-compose ejecutando el siguiente comando en terminal en el directorio raiz del proyecto.
$ docker-compose up

docker-compose.yaml para levantar todo el entorno
=
version: '3'
services:
  mlflow:    
    build: ./mlflow-server
    ports:
      - "5000:5000"
    depends_on:
      - "ftpd"
      - "postgres"
    volumes: 
      - /usr/bin/docker:/usr/bin/docker
      - /var/run/docker.sock:/var/run/docker.sock
  ftpd:
    image: "mauler/simple-ftp-server"
    environment:
      - FTP_USER=test
      - FTP_PASS=test
    ports:
      - "21:21"        
  postgres:
    build: ./postgres-mlflow
    ports:
      - "5432:5432"
  jupyter:
    environment:
      - JUPYTER_ENABLE_LAB=yes  
    build: ./jupyter-mlflow
    ports:
      - "8888:8888"
  portainer:    
    image: "portainer/portainer:1.23.0"
    volumes: 
      - /var/run/docker.sock:/var/run/docker.sock
    ports: 
      - "9000:9000"
      - "8000:8000"
      
 Esperamos unos segundos y verificamos que los contenedores se hayan levantado correctamente mediante portainer accesible en http://localhost:9000
 
 
Paso 2: Desarrollo del modelo de machine learning:
=
Dado que el objetivo de este post es demostrar el flujo completo desde el desarrollo hasta la puesta en operación de un modelo, vamos a tomar un ejemplo del dataset de diabetes incluido en la carpeta de ejemplos del repositorio de mlflow (este ejercicio ya viene cargado en la imagen de jupyter construida).

Para poder acceder a jupyter, buscamos en los logs de docker compose y encontramos una linea semejante a la de la imagen, la copiamos y sustituimos la parte entre parentesis por localhost o 127.0.0.1.

![image](https://user-images.githubusercontent.com/30583333/187963790-00f048b5-89d8-41b6-89b7-d2c3aac604e4.png)
Log de docker-compose

La cadena anterior quedaría: http://localhost:8888/?token=….. etc.

Pegamos esta cadena en nuestro browser y este terminará dirigiéndonos a localhost:8888/lab. Posteriormente abrimos el archivo ejemplo ipynb y ejecutamos el notebook.
![image](https://user-images.githubusercontent.com/30583333/187963946-15042afe-4ea1-4b04-a37c-1eaa9c9d7b3b.png)


3. Registro de modelos en MLFlow
=
Una vez ejecutado el código, el modelo estará registrado en mlflow donde estarán loggedas las métricas especificadas en el notebook mediante los comandos:

mlflow.log_param
=
mlflow.log_metric
=
mlflow.sklearn.log_model
=
![image](https://user-images.githubusercontent.com/30583333/187964331-100abd07-4dc5-4e87-82f1-0bc1abd6f245.png)
UI principal mlflow http://localhost:5000


![image](https://user-images.githubusercontent.com/30583333/187964382-ebb0093c-e184-4538-a065-16f88b524dc4.png)

Parámetros y métricas de ejecución

![image](https://user-images.githubusercontent.com/30583333/187964431-720c7038-4438-4ae0-af96-abcd4cbed5c9.png)
Modelo loggeado


Paso 4: Creación de una imagen de docker con el modelo entrenado.
=
Para este paso, nos conectamos mediante bash al contenedor del servidor de mlflow mediante el siguiente comando:

$ docker exec -it ml-deployment_mlflow_1 /bin/bash


Una vez dentro del contenedor ejecutamos el siguiente comando:
=
$ mlflow models build-docker -m ftp://test:test@ftpd/home/test/0/e7df5edb12d547538e94ce06d7970754/artifacts/model/model

Siendo la cadena ftp://….. la ubicación del modelo entrenado en en el servidor de ftp.

MLflow construirá una imagen de docker basada en ubuntu (default de mlflow) con el modelo entrenado adentro y la hará disponible en el docker-registry local. El contenedor de Mlflow es capaz de acceder al docker engine ya que tanto el binario como el socket estan mapeados como un volumen. Este proceso tarda unos minutos.

![image](https://user-images.githubusercontent.com/30583333/187964655-fbf77c00-e430-4b3d-bd39-2402ce4aa4b9.png)

Construccion de la imagen con el modelo entrenado
=

Paso 5: Creación del contenedor con el modelo entrenado.
=
Una vez que este disponible la imagen dentro del registry de docker, procedemos a instanciarla para llevar a cabo predicciones. Ejecutamos con el siguiente comando desde nuestro sistema operativo base:

$ docker run -it -p 5002:8080 mlflow-pyfunc-servable:latest

Paso 6. Llevar a cabo una predicción.
=
Existen muchas herramientas para llevar a cabo pruebas sobre APIs, nosotros utilizaremos POSTMAN que es un servicio web para llevar a cabo testing sobre Rest APIS. Especificamos metodo POST y el URL: http://localhost:5002/invocations, el tipo de contenido “application/json” así como el body del request (datos de entrada a predecir).

![image](https://user-images.githubusercontent.com/30583333/187964798-81034766-b9ac-4698-8898-cf1e9ef33629.png)
Predicción del modelo entrenado sobre un registro de 10 variables de entrada.

Consideraciones
=
Este proyecto esta construido para demostrar el flujo completo desde el desarrollo de un modelo hasta su operación, sin embargo se tienen que resaltar aspectos importantes si se desea implementarlo a escala.

Necesaria incorporación de herramientas de seguridad y auditoría.
Necesaria la incorporación de bases de datos robustas y almacenamiento de archivos con respaldos.
Necesaria la incorporación de control de versiones y un flujo de CI/CD, Ansible, etc.
Despliegue del modelo sobre infraestructura de microservicios de kubernetes para garantizar su alta disponibilidad.
Necesaria Incorporación de herramienta descriptiva de API mediante swagger.
Necesaria incorporación de un API manager para llevar la adecuada contabilidad y acceso al modelo.

Conclusiones Finales
=
En este post se mostró como gestionar un modelo de machine learning de punta a punta, sin embargo, también se mostró que para llevar esto a una escala productiva es necesario incorporar más elementos que brinden alta disponibilidad, seguridad y respaldo.


Recursos adicionales
=
https://mc.ai/setup-mlflow-in-production/
