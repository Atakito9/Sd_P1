# Sd_P1
### Practica 1

![](https://img.shields.io/badge/version-v0.alph-blue)

![](https://upload.wikimedia.org/wikipedia/commons/thumb/d/db/Apache_Software_Foundation_Logo_%282016%29.svg/960px-Apache_Software_Foundation_Logo_%282016%29.svg.png?20160227223120)

Apache Spark es un framework de código abierto para computación distribuida. Spark suele instalarse en un cluster, y puede interactuar con diversos sistemas distribuidos, como HDFS (Hadoop Distributed File System), o sistemas similares, como Amazon S3.

Spark se puede además integrar con sistemas de gestión de cluster, como Apache Mesos, o usar su propio sistema de cluster nativo.

En esta práctica instalaremos Spark y crearemos trabajos que mandaremos al sistema distribuido usando el API para Python (PySpark).

Usaremos el workshop de Jupyter para calcular datos meteorologicos desde un CSV

## TO-DO
- [x] Javascript y Conexion a Jupyter con spark
- [ ] Ejercicio 0 - Convertir lista CSV a Texto legible con ;
    - [ ] Revisar codigo de Ej0.ipynb
- [ ] Ejercicio 1 -

`docker compose up` <- Iniciar spark y conectarse al servidor de localhost
RECUERDA, haz un git clone del repositorio dentro de Jupyter para que funcionen los comandos de spark.

```javascript
from pyspark import SparkContext

# Inicialización del Contexto dentro de Jupyter
sc = SparkContext("local[*]", "Sd_P1")
file_name = "calidad_aire_datos_meteo_mes.csv"

# Constantes
MAG_TEMP = 83  # Magnitud Temperatura 
MAG_PREC = 89  # Magnitud Precipitación 
FLAG_VALID = 'V' # Bandera de validación [cite: 81]

# Cargar archivo dentro de Jupyter
raw_rdd = sc.textFile(f"file:///home/jovyan/Sd_P1/{file_name}")
```
