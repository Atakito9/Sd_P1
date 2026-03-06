# Sd_P1
### Practica 1

![](https://img.shields.io/badge/version-v1.2_test2-blue)

![](https://upload.wikimedia.org/wikipedia/commons/thumb/3/38/Jupyter_logo.svg/500px-Jupyter_logo.svg.png)

Apache Spark es un framework de código abierto para computación distribuida. Spark suele instalarse en un cluster, y puede interactuar con diversos sistemas distribuidos, como HDFS (Hadoop Distributed File System), o sistemas similares, como Amazon S3.

Spark se puede además integrar con sistemas de gestión de cluster, como Apache Mesos, o usar su propio sistema de cluster nativo.

En esta práctica instalaremos Spark y crearemos trabajos que mandaremos al sistema distribuido usando el API para Python (PySpark).

Usaremos el workshop de Jupyter para calcular datos meteorologicos desde un CSV

## Versiones del codigo - Estable v1.2
Existen dos versiones del codigo actualmente: Ej0 y Ej1.
La version final, sin hilos y sin bucles for es el codigo Ej1.ipynb

[Link Ej1.ipynb](https://github.com/Atakito9/Sd_P1/blob/main/Ej1.ipynb "Link Ej1.ipynb")

## TO-DO
- [x] Javascript y Conexion a Jupyter con spark
    - [x] docker compose up & apt install spark
- [x] Ejercicio 0 - Convertir lista CSV a Texto legible con ;
    - [x] Revisar codigo de Ej0.ipynb
    - [x] Crear Ej_final.ipynb con los cambios necesarios (1 bucle for y ningun def)
- [x] Ejercicio 1 - Separar datos dentro de el CSV
    - [x] Hallar todos los datos y separar por valido e invalido
    - [x] NEW!: Eliminar el conjunto de hilos(def) por otro tipo de estructura
- [x] Ejercicio 2 - Calcular la temperatura maxima valida
    - [x] A partir de la tupla hallamos las temperaturas maximas y lo ordenamos
    - [x] NEW!: Eliminar el bucle for para imprimir por otra estructura
- [x] Ejercicio 3 - Calcular las estaciones con mayor precipitacion
    - [x] Usar la tupla otra vez para reducir e ordenar
    - [x] NEW!: Eliminar el bucle for para imprimir por otra estructura
- [x] Ejercicio 4 - Comparar valores de dos estaciones concretas
    - [x] Hallar valor medio de las temperaturas
    - [x] Calcular el porcentaje
    - [x] NEW! Cambiar el def para calcular el valor medio
    - [x] NEW! Eliminar el bucle for para imprimir
- [x] Adaptar todo el codigo a los cambios

⚠︎ El proyecto usa librerias de spark, usa `sudo apt install spark` para poder usar

## Preparacion

Para conectarse a Jupyter Notebook creamos una carpeta spark y ponemos este codigo
```yml
services:
  notebook:
    image: quay.io/jupyter/pyspark-notebook
    restart: on-failure
    command: start-notebook.py --IdentityProvider.token=''
    ports:
      - 8888:8888
```
Esto servira para lanzar un puente para conectarse a Jupyter, en la carpeta donde esta este archivo lanzamos una terminal y ejecutamos el archivo:
`docker compose up` <- Iniciar spark y conectarse al servidor de localhost a partir de este link - https://localhost:8888/lab

RECUERDA, haz un git clone del repositorio dentro de Jupyter para que funcionen los comandos de spark.

## Ejercicio 1

```python
# Inicialización del Contexto
sc = SparkContext.getOrCreate()
file_name = "calidad_aire_datos_meteo_mes.csv"

# Constantes
MAG_TEMP = 83
MAG_PREC = 89
FLAG_VALID = 'V'

# Carga del archivo que esta dentro de jupyter
raw_rdd = sc.textFile(f"Sd_P1/{file_name}")
```
En este ejercicio se utilizará spark para cargar los datos meteorológicos de la Comunidad de Madrid y hacer diferentes análisis sobre estos usando RDDs.

En el primer apartado se contará cuantos registros diarios de temperatura(magnitud 83) presentan al menos una hora con dato válido.


Descripción del código

En el segundo apartado se calculará para cada día disponible del fichero la temperatura máxima horaria válida registrada considerando únicamente los valores de magnitud de Temperatura(83). Descartando aquellas horas
cuyo campo de validación no sea "V". (Mostrando los valores en formato YYYY-MM-DD, y la temperatura máxima diaria correspondiente)

Descripción del código

En el tercer apartado se calcula para cada día disponible en el fichero, la estación con mayor precipitación total registrada en la Comunidad de Madrid considerando únicamente los
valores correspondientes a la magnitud Precipitación (89) y sumando exclusivamente
aquellas horas cuyo campo de validación sea "V". (En el resultado se mostrará la fecha, el municipio, la estación y la precipitación total diaria correspondiente). También se ha de determinar el mayor valor de precipitación diaria observado en todo el periodo analizado, indicando la fecha, el municipio y la estación en la que se produjo (En caso de empate entre varias estaciones, se seleccionará cualquiera de ellas)

Descripción del código

En el cuarto apartado se compara los valores de temperatura media diaria
(magnitud 83) registrados en dos estaciones concretas de la Comunidad de Madrid: municipio 6, estación 4 y municipio 5, estación 2. Muestra los resultados como un porcentaje del valor de la estación de referencia (municipio 6, estación 4), de modo que el porcentaje sea 100 si son iguales, 50 si la segunda estación tiene la mitad, 200 si tiene el doble, etc (En el resultado se  mostrará la fecha en formato YYYY-MM-DD y el porcentaje correspondiente)

Descripción del código