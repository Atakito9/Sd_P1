# Sd_P1
## Practica 1

![](https://img.shields.io/badge/version-v1.2-blue)

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

## Inicio de la Practica
### Carga del archivo
Antes de empezar con el ejercicio, se crea una tupla para poder guardar cada uno de los datos de el csv y cuando se necesiten tenerlos mas a mano.
Primero se carga el archivo csv para sacar los datos del archivo y lo partimos en partes con los ;.
```python
# =================================================================
# 1. Constantes 
# =================================================================

MAG_TEMP = 83
MAG_PREC = 89
FLAG_VALID = 'V'

# =================================================================
# 2. Extraccion de datos 
# =================================================================
sc = SparkContext.getOrCreate()
file_name = "calidad_aire_datos_meteo_mes.csv"
raw_rdd = sc.textFile(f"Sd_P1/{file_name}")

SEP = ';'
safe_raw_rdd = raw_rdd.filter(lambda line: len(line.split(SEP)) >= 9 and line.split(SEP)[3].isdigit()) #Partir el archivo en partes EJ: 120;V;88;V...
```

Luego se guardan en una tupla para facil acceso:
```python
parse_lambda = lambda line: (
    lambda parts: {
        'provincia': parts[0],
        'fecha': f"{parts[5]}-{parts[6].zfill(2)}-{parts[7].zfill(2)}",
        'muni': parts[1],
        'est': parts[2],
        'mag': int(parts[3]),
        
        'vals': [
            float(parts[8 + (i * 2)].replace(',', '.'))
            for i in range(24)
            if (8 + (i * 2) + 1) < len(parts) and 
               parts[8 + (i * 2) + 1] == FLAG_VALID
        ]
    }
)(line.split(SEP))

data_rdd = safe_raw_rdd.map(parse_lambda)
```

### Ejercicio 1
En el primer apartado se cargan los datos meteorológicos de la Comunidad de Madrid y hacer diferentes análisis sobre estos usando RDDs. Se cuentan cuantos registros diarios de temperatura(magnitud 83) que presentan al menos una hora con dato válido.
##### Codigo
```python
# EJERCICIO 1
temp_validos_rdd = data_rdd.filter(lambda x: x['mag'] == MAG_TEMP and len(x['vals']) > 0)
total_validos = temp_validos_rdd.count()
```

## Ejercicio 2
En el segundo apartado se calcula para cada día disponible del fichero la temperatura máxima horaria válida registrada considerando únicamente los valores de magnitud de Temperatura(83). Descartando aquellas horas
cuyo campo de validación no sea "V". (Mostrando los valores en formato YYYY-MM-DD, y la temperatura máxima diaria correspondiente)
##### Codigo
```python
# EJERCICIO 2
# Mappear por temperatura maxima por dias
final_daily_max = temp_validos_rdd.map(lambda x: (x['fecha'], max(x['vals']))) \
                                  .reduceByKey(lambda a, b: max(a, b))
```

## Ejercicio 3
En el tercer apartado se calcula para cada día disponible en el fichero, la estación con mayor precipitación total registrada en la Comunidad de Madrid considerando únicamente los
valores correspondientes a la magnitud Precipitación (89) y sumando exclusivamente
aquellas horas cuyo campo de validación sea "V". (En el resultado se mostrará la fecha, el municipio, la estación y la precipitación total diaria correspondiente). 

En el apartado B se determina el mayor valor de precipitación diaria observado en todo el periodo analizado, indicando la fecha, el municipio y la estación en la que se produjo (En caso de empate entre varias estaciones, se seleccionará cualquiera de ellas)
##### Codigo
```python
# EJERCICIO 3
max_precip_por_dia = data_rdd.filter(lambda x: x['mag'] == MAG_PREC) \
                             .map(lambda x: (x['fecha'], (x['muni'], x['est'], sum(x['vals'])))) \
                             .reduceByKey(lambda x, y: x if x[2] >= y[2] else y)

# PARTE B: Máximo Absoluto del periodo
max_absoluto = max_precip_por_dia.max(key=lambda x: x[1][2])
```

## Ejercicio 4
En el cuarto apartado se compara los valores de temperatura media diaria (magnitud 83) registrados en dos estaciones concretas de la Comunidad de Madrid: municipio 6, estación 4 y municipio 5, estación 2. Muestra los resultados como un porcentaje del valor de la estación de referencia (municipio 6, estación 4), de modo que el porcentaje sea 100 si son iguales, 50 si la segunda estación tiene la mitad, 200 si tiene el doble, etc (En el resultado se  mostrará la fecha en formato YYYY-MM-DD y el porcentaje correspondiente)
##### Codigo
```python
# EJERCICIO 4
# Valor medio de las estaciones de municipios validos
get_daily_avg_lambda = lambda record: None if len(record['vals']) == 0 else (record['fecha'], sum(record['vals']) / len(record['vals']))

# 1. Se crean RDDs separados para cada estación específica
ref_station = temp_validos_rdd.filter(lambda x: int(x['muni']) == 6 and int(x['est']) == 4) \
                              .map(get_daily_avg_lambda).filter(lambda x: x is not None)

comp_station = temp_validos_rdd.filter(lambda x: int(x['muni']) == 5 and int(x['est']) == 2) \
                               .map(get_daily_avg_lambda).filter(lambda x: x is not None)
# 2. Unimos los datos por Fecha y calculamos sus porcentajes
percentage_rdd = ref_station.join(comp_station) \
                            .mapValues(lambda x: (x[1] / x[0]) * 100)
```

## Resultados al ejecutar el codigo

Ej 1 - Total registros de temperatura válidos: 224

Ej 3B - Máx Absoluto Precipitación -> Fecha: 2026-02-05 | Muni: 115 | Est: 3 | Total: 30.8 mm

FECHA        | T_MAX (Ej 2)    | PRECIP MAX (Ej 3A) [Muni, Est, mm]   | COMPARATIVA (Ej 4)

2026-02-01   | 11.7            | ('120', '1', 20.8)                   | 129.04%           
2026-02-02   | 12.2            | ('161', '1', 18.4)                   | 113.45%           
2026-02-03   | 9.8             | ('127', '4', 7.6)                    | 114.98%           
2026-02-04   | 11.3            | ('45', '2', 11.6)                    | 114.01%           
2026-02-05   | 15.5            | ('115', '3', 30.8)                   | 112.09%           
2026-02-06   | 10.3            | ('67', '1', 6.3)                     | 116.52%           
2026-02-07   | 9.0             | ('115', '3', 21.8)                   | 131.91%           
2026-02-08   | 12.7            | ('120', '1', 25.1)                   | 116.34%           
