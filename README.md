Mlflow es un proyecto open source que facilita la gestión de modelos de machine learning incluyendo la experimentación, la reproducibilidad así como el despliegue de los mismos. El proyecto fue desarrollado por el equipo creador de spark y está disponible como un módulo de python el cuál es muy sencillo instalar mediante la siguiente sentencia:

$ pip install mlflow==1.5.0


MLFlow esta compuesto de tres componentes principales:

MLflow tracking: Es una componente para loggeo de métricas, parámetros, versiones de código, así como archivos de salida de las ejecuciones de nuestro código. Existen clientes para Python, R y Java.
MLflow projects: Es una estructura de archivos que permiten organizar Y gestionar el modelo de machine learning de manera reproducible. Consiste en tres archivos de salida generados: conda.yaml, MLproject y model.pkl
MLflow models: Es un estándar de empaquetado para la ejecución de un modelo y consumo a traves de una REST API. Librerias y frameworks soportados: sklearn, h20, mleap, keras, tensorflow, onnx, etc.
A su vez mlflow se apoya de otras componentes adicionales para poder funcionar las cuales son: backend-store (almacenamiento de metricas) y default-artifact-root (archivos y modelos resultados).

La backend-store puede ser de los siguientes tipos: archivos planos, postgres, mysql, etc.

La default-artifact puede ser de los siguientes tipos: archivos locales, azure blob storage, aws s3, ftp, etc.
