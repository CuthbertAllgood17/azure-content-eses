<properties
	pageTitle="Exploración y modelado avanzados de datos con Spark | Microsoft Azure"
	description="TBD."
	services="machine-learning"
	documentationCenter=""
	authors="bradsev,deguhath,gokuma"
	manager="paulettm"
	editor="cgronlun"  />

<tags
	ms.service="machine-learning"
	ms.workload="data-services"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="04/26/2016"
	ms.author="deguhath;bradsev" />

# Exploración y modelado avanzados de datos con Spark 

[AZURE.INCLUDE [machine-learning-spark-modeling](../../includes/machine-learning-spark-modeling.md)]

En este tutorial se usa Spark en HDInsight para realizar tareas de modelado por exploración de datos, clasificación binaria y regresión. Para ello, se usan la validación cruzada y la optimización de hiperparámetros en una muestra del conjunto de datos de carreras y tarifas de taxi de 2013 en la ciudad de Nueva York. Lo guía por los pasos del [proceso de la ciencia de los datos](http://aka.ms/datascienceprocess), de principio a fin, usando un clúster de Spark en HDInsight para el procesamiento y blobs de Azure para almacenar los datos y los modelos. El proceso analiza y visualiza los datos extraídos de un Blob de Almacenamiento de Azure y, después, los prepara para crear modelos predictivos. Se ha usado Python para codificar la solución y mostrar los trazados relevantes. Estos modelos se crean usando el kit de herramientas MLlib de Spark para realizar las tareas de clasificación binaria y modelado por regresión.

- La tarea de **clasificación binaria** consiste en predecir si se dará propina por la carrera o no. 
- La tarea de **regresión** consiste en predecir el importe de la propina en función de otras características de la propina. 

Los pasos de modelado también contienen código que muestra cómo entrenar, evaluar y guardar cada tipo de modelo. Este tema abarca algunos aspectos que coinciden con [Data exploration and modeling with Spark](machine-learning-data-science-spark-data-exploration-modeling.md) (Exploración y modelado de datos con Spark), aunque de forma "avanzada", ya que también usa la validación cruzada junto con el barrido de hiperparámetros para entrenar modelos de clasificación y regresión con precisión óptima.

La **validación cruzada (VC)** es una técnica que evalúa la calidad de la generalización que realizará un modelo entrenado con un conjunto conocido de datos para predecir las características de conjuntos de datos con los que no se haya entrenado. La idea general tras esta técnica es que se entrena un modelo con un conjunto conocido de datos y después se prueba la precisión de sus predicciones con un conjunto de datos independiente. En este caso, se usa una implementación habitual en la que se divide un conjunto de datos en K iteraciones y después se entrena el modelo como round-robin en todas las iteraciones menos una.

La **optimización de los hiperparámetros** es el problema de elegir un conjunto de hiperparámetros para un algoritmo de aprendizaje, normalmente con el fin de optimizar una medida del rendimiento del algoritmo con un conjunto de datos independiente. Los **hiperparámetros** son valores que deben especificarse fuera del procedimiento de entrenamiento del modelo. Las suposiciones que se hagan sobre estos valores pueden afectar a la flexibilidad y la precisión de los modelos. Los árboles de decisión tienen hiperparámetros, como por ejemplo la profundidad que desee y el número de hojas del árbol. Las máquinas de vectores de soporte (SVM) requieren que se establezca un término de penalización para las clasificaciones incorrectas.

Una forma habitual de realizar la optimización de hiperparámetros usada aquí es una búsqueda de cuadrícula o **barrido de parámetros**. Esto consiste en llevar a cabo una búsqueda exhaustiva en los valores de un subconjunto concreto del espacio de hiperparámetros para un algoritmo de aprendizaje. La validación cruzada puede proporcionar una métrica de rendimiento para ordenar los resultados óptimos generados por el algoritmo de búsqueda de cuadrícula. La VC que se usa con el barrido de hiperparámetros ayuda a limitar problemas tales como el sobreajuste del modelo a los datos de aprendizaje, para que el modelo conserve la capacidad de aplicarse al conjunto general de datos del que se extrajeron los datos de aprendizaje.

Los modelos que usamos incluyen regresión logística y lineal, bosques aleatorios y árboles impulsados por gradiente:

- La [regresión lineal con SGD](https://spark.apache.org/docs/latest/api/python/pyspark.mllib.html#pyspark.mllib.regression.LinearRegressionWithSGD) es un modelo de regresión lineal que usa un método de descenso de gradiente estocástico (SGD) para la optimización y el ajuste de la escala de las características para predecir las propinas. 
- La [regresión logística con LBFGS](https://spark.apache.org/docs/latest/api/python/pyspark.mllib.html#pyspark.mllib.classification.LogisticRegressionWithLBFGS) o regresión "logit" es un modelo de regresión que puede usarse cuando la variable dependiente es categórica para realizar la clasificación de los datos. LBFGS es un algoritmo de optimización cuasi Newton que aproxima el algoritmo Broyden–Fletcher Goldfarb–Shanno (BFGS) usando una cantidad limitada de memoria de proceso, y que se usa ampliamente en el aprendizaje automático.
- Los [bosques aleatorios](http://spark.apache.org/docs/latest/mllib-ensembles.html#Random-Forests) son conjuntos de árboles de decisión. Combinan varios árboles de decisión para reducir el riesgo de sobreajuste. Los bosques aleatorios se usan para clasificación y regresión, y pueden controlar características categóricas, amplían la configuración de clasificación multiclase, no requieren ajustar la escala de las características y pueden capturar errores de alineación e interacciones de las características. Los bosques aleatorios son uno de los modelos de aprendizaje automático de más éxito para clasificación y regresión.
- Los [árboles impulsados por gradiente](http://spark.apache.org/docs/latest/ml-classification-regression.html#gradient-boosted-trees-gbts) (GBT) son conjuntos de árboles de decisión. Los GBT entrenan árboles de decisión de forma iterativa para minimizar una función de pérdida. Los GBT se usan para clasificación y regresión, y pueden controlar características categóricas, no requieren ajustar la escala de las características y pueden capturar errores de alineación e interacciones de las características. También se pueden usar en una configuración de clasificación multiclase.

Se muestran ejemplos donde se usan la VC y el barrido de hiperparámetros para el problema de la clasificación binaria. Se presentan ejemplos más sencillos (sin el barrido de parámetros) en el tema principal para las tareas de regresión. Sin embargo, en el apéndice también se presentan la validación mediante red elástica para la regresión lineal y la VC con barrido de parámetros para la regresión con bosque aleatorio. La **red elástica** es un método de regresión regularizada para ajustar los modelos de regresión lineal que combina linealmente las métricas L1 y L2 como penalizaciones de los métodos de [lazo](https://en.wikipedia.org/wiki/Lasso%20%28statistics%29) y [arista](https://en.wikipedia.org/wiki/Tikhonov_regularization).



>[AZURE.NOTE] Aunque el kit de herramientas MLlib de Spark está diseñado para trabajar con grandes conjuntos de datos, con el fin de demostrar sus funcionalidades de modelado, se usa por comodidad una muestra relativamente pequeña (~30 MB, 170 filas; aproximadamente el 0,1 % del conjunto de datos original de Nueva York). El ejercicio que aquí se proporciona se ejecuta eficazmente en un clúster de HDInsight con 2 nodos de trabajo (en unos 10 minutos). Se puede usar el mismo código, con modificaciones menores, para procesar conjuntos de datos mayores, con las modificaciones correspondientes para almacenar datos en memoria caché o cambiar el tamaño del clúster.


## Requisitos previos

Necesita una cuenta de Azure y un clúster de Spark en HDInsight, versión Spark 1.5.2 (HDI 3.3), para comenzar este tutorial. Consulte [Overview of Data Science using Spark on Azure HDInsight](machine-learning-data-science-spark-overview.md) (Información general sobre la ciencia de los datos con Spark en HDInsight de Azure) para ver estos requisitos, obtener una descripción de los datos de taxis de Nueva York de 2013 usados aquí e instrucciones para ejecutar código desde un Notebook de Jupyter en el clúster de Spark. El Notebook **machine-learning-data-science-spark-advanced-data-exploration-modeling.ipynb** que contiene los ejemplos de código de este tema está disponible en [Github](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/Spark/Python).


[AZURE.INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]


## Configuración: ubicaciones de almacenamiento, bibliotecas y contexto de Spark

Spark puede leer y escribir en un blob de Almacenamiento de Azure (también conocido como WASB), por lo que cualquiera de los datos existentes almacenados allí pueden procesarse mediante Spark y volver a almacenarse en WASB.

Para guardar modelos o archivos en WASB, la ruta de acceso debe especificarse correctamente. Se puede hacer referencia el contenedor predeterminado asociado al clúster Spark con un ruta que comience con: "wasb///". Para hacer referencia a otras ubicaciones, se usa "wasb://".

### Establecimiento de rutas de directorio para las ubicaciones de almacenamiento de WASB

El ejemplo de código siguiente especifica la ubicación de los datos que se van a leer y la ruta de acceso del directorio de almacenamiento del modelo donde se guardará la salida del modelo.

	# SET PATHS TO FILE LOCATIONS: DATA AND MODEL STORAGE

	# LOCATION OF TRAINING DATA
	taxi_train_file_loc = "wasb://mllibwalkthroughs@cdspsparksamples.blob.core.windows.net/Data/NYCTaxi/JoinedTaxiTripFare.Point1Pct.Train.tsv";

	
	# SET THE MODEL STORAGE DIRECTORY PATH 
	# NOTE THAT THE FINAL BACKSLASH IN THE PATH IS NEEDED.
	modelDir = "wasb:///user/remoteuser/NYCTaxi/Models/";

	# PRINT START TIME
	import datetime
	datetime.datetime.now()

**SALIDA**

datetime.datetime(2016, 4, 18, 17, 36, 27, 832799)


### Importación de las bibliotecas necesarias y establecimiento del contexto de Spark 

Establezca el contexto de Spark e importe las bibliotecas necesarias con el código siguiente.

	# LOAD PYSPARK LIBRARIES
	import pyspark
	from pyspark import SparkConf
	from pyspark import SparkContext
	from pyspark.sql import SQLContext
	%matplotlib inline
	import matplotlib
	import matplotlib.pyplot as plt
	#matplotlib.style.use('ggplot')
	from pyspark.sql import Row
	from pyspark.sql.functions import UserDefinedFunction
	from pyspark.sql.types import *
	import atexit
	from numpy import array
	import numpy as np
	import datetime
	
	# SET SPARK CONTEXT
	sc = SparkContext(conf=SparkConf().setMaster('yarn-client'))
	sqlContext = SQLContext(sc)
	atexit.register(lambda: sc.stop())
	
	sc.defaultParallelism

**SALIDA**

4

## Ingesta de datos: 

Esta sección contiene el código de una serie de tareas necesarias para incorporar la muestra de datos que se va a modelar. Lea una muestra combinada del 0,1 % del archivo de carreras y tarifas de taxi (almacenado como un archivo .tsv), dé formato y limpie los datos, cree una trama de datos en memoria y almacénela en caché y, a continuación, regístrela como una tabla temporal en el contexto de SQL.

El primer paso en el proceso de la ciencia de los datos es incorporar los datos que se van a analizar desde orígenes o sistemas externos donde residen, a su entorno de exploración y modelado de datos. Este entorno es Spark en este tutorial. Esta sección contiene el código para completar una serie de tareas:

- incorporar la muestra de datos que se va a modelar
- leer el conjunto de datos de entrada (almacenado como un archivo .tsv)
- dar formato y limpiar los datos
- crear objetos en la memoria (RDD o tramas de datos) y almacenarlos en caché
- registrarlos como una tabla temporal en el contexto de SQL

Este es el código para la incorporación de los datos.


	# RECORD START TIME
	timestart = datetime.datetime.now()
	
	# IMPORT FILE FROM PUBLIC BLOB
	taxi_train_file = sc.textFile(taxi_train_file_loc)
	
	# GET SCHEMA OF THE FILE FROM HEADER
	schema_string = taxi_train_file.first()
	fields = [StructField(field_name, StringType(), True) for field_name in schema_string.split('\t')]
	fields[7].dataType = IntegerType() #Pickup hour
	fields[8].dataType = IntegerType() # Pickup week
	fields[9].dataType = IntegerType() # Weekday
	fields[10].dataType = IntegerType() # Passenger count
	fields[11].dataType = FloatType() # Trip time in secs
	fields[12].dataType = FloatType() # Trip distance
	fields[19].dataType = FloatType() # Fare amount
	fields[20].dataType = FloatType() # Surcharge
	fields[21].dataType = FloatType() # Mta_tax
	fields[22].dataType = FloatType() # Tip amount
	fields[23].dataType = FloatType() # Tolls amount
	fields[24].dataType = FloatType() # Total amount
	fields[25].dataType = IntegerType() # Tipped or not
	fields[26].dataType = IntegerType() # Tip class
	taxi_schema = StructType(fields)
	
	# PARSE FIELDS AND CONVERT DATA TYPE FOR SOME FIELDS
	taxi_header = taxi_train_file.filter(lambda l: "medallion" in l)
	taxi_temp = taxi_train_file.subtract(taxi_header).map(lambda k: k.split("\t"))\
	        .map(lambda p: (p[0],p[1],p[2],p[3],p[4],p[5],p[6],int(p[7]),int(p[8]),int(p[9]),int(p[10]),
	                        float(p[11]),float(p[12]),p[13],p[14],p[15],p[16],p[17],p[18],float(p[19]),
	                        float(p[20]),float(p[21]),float(p[22]),float(p[23]),float(p[24]),int(p[25]),int(p[26])))
	
	    
	# CREATE A CLEANED DATA-FRAME BY DROPPING SOME UN-NECESSARY COLUMNS & FILTERING FOR UNDESIRED VALUES OR OUTLIERS
	taxi_df_train_cleaned = taxi_train_df.drop('medallion').drop('hack_license').drop('store_and_fwd_flag').drop('pickup_datetime')\
	    .drop('dropoff_datetime').drop('pickup_longitude').drop('pickup_latitude').drop('dropoff_latitude')\
	    .drop('dropoff_longitude').drop('tip_class').drop('total_amount').drop('tolls_amount').drop('mta_tax')\
	    .drop('direct_distance').drop('surcharge')\
	    .filter("passenger_count > 0 and passenger_count < 8 AND payment_type in ('CSH', 'CRD') AND tip_amount >= 0 AND tip_amount < 30 AND fare_amount >= 1 AND fare_amount < 150 AND trip_distance > 0 AND trip_distance < 100 AND trip_time_in_secs > 30 AND trip_time_in_secs < 7200" )
	
	# CACHE DATA-FRAME IN MEMORY & MATERIALIZE DF IN MEMORY
	taxi_df_train_cleaned.cache()
	taxi_df_train_cleaned.count()
	
	# REGISTER DATA-FRAME AS A TEMP-TABLE IN SQL-CONTEXT
	taxi_df_train_cleaned.registerTempTable("taxi_train")
	
	# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 


**SALIDA**

Time taken to execute above cell: 17.73 seconds


## Visualización y exploración de datos 

Una vez incorporados los datos en Spark, el siguiente paso del proceso de la ciencia de los datos es conocer mejor los datos mediante la exploración y la visualización. En esta sección, examinaremos los datos de taxi mediante consultas SQL y trazaremos las variables de destino y las posibles características para su inspección visual. En concreto, trazaremos la frecuencia de los recuentos de pasajeros en las carreras de taxi, la frecuencia de los importes de las propinas y cómo varían las propinas según el tipo y el importe del pago.

### Trazado de un histograma de frecuencias de recuento de pasajeros en el ejemplo de carreras de taxi

El código usa una consulta de selección SQL para obtener una muestra de los datos y convierte los resultados en una trama de datos de Pandas para trazar.

	# PLOT FREQUENCY OF PASSENGER COUNTS IN TAXI TRIPS

	# SQL SQUERY
	sqlStatement = """
	    SELECT passenger_count, COUNT(*) as trip_counts 
	    FROM taxi_train 
	    WHERE passenger_count > 0 and passenger_count < 7
	    GROUP BY passenger_count 
	"""
	sqlResults = sqlContext.sql(sqlStatement)
	
	#CONVERT TO PANDAS DATA-FRAMES FOR PLOTTING IN PYTHON
	resultsPDDF = sqlResults.toPandas()
	
	# PLOT PASSENGER NUMBER VS. TRIP COUNTS
	x_labels = resultsPDDF['passenger_count'].values
	fig = resultsPDDF[['trip_counts']].plot(kind='bar', facecolor='lightblue')
	fig.set_xticklabels(x_labels)
	fig.set_title('Counts of trips by passenger count')
	fig.set_xlabel('Passenger counts')
	fig.set_ylabel('Trip counts')
	plt.show()

**SALIDA**

![Frecuencia de carreras por número de pasajeros](./media/machine-learning-data-science-spark-advanced-data-exploration-modeling/frequency-of-trips-by-passenger-count.png)


### Trazado de un histograma de importes de propinas y de las variaciones en las propinas según número de pasajeros y tarifas.

El código usa una consulta de selección SQL para obtener una muestra de los datos y convierte los resultados en una trama de datos de Pandas para trazar.

	# RECORD START TIME
	timestart = datetime.datetime.now()
	
	# SQL SQUERY
	sqlStatement = """
	    SELECT fare_amount, passenger_count, tip_amount, tipped
	    FROM taxi_train 
	    WHERE passenger_count > 0 AND passenger_count < 7
	    AND fare_amount > 0 AND fare_amount < 200
	    AND payment_type in ('CSH', 'CRD')
	    AND tip_amount > 0 AND tip_amount < 25
	"""
	sqlResults = sqlContext.sql(sqlStatement)
	
	# CONVERT TO PANDAS DATA-FRAME FOR PLOTTING IN PYTHON
	resultsPDDF= sqlResults.toPandas()
	
	# TIP BY PAYMENT TYPE AND PASSENGER COUNT
	ax1 = resultsPDDF[['tip_amount']].plot(kind='hist', bins=25, facecolor='lightblue')
	ax1.set_title('Tip amount distribution')
	ax1.set_xlabel('Tip Amount ($)')
	ax1.set_ylabel('Counts')
	plt.suptitle('')
	plt.show()
	
	# TIP BY PASSENGER COUNT
	ax2 = resultsPDDF.boxplot(column=['tip_amount'], by=['passenger_count'])
	ax2.set_title('Tip amount ($) by Passenger count')
	ax2.set_xlabel('Passenger count')
	ax2.set_ylabel('Tip Amount ($)')
	plt.suptitle('')
	plt.show()
	
	# TIP AMOUNT BY FARE AMOUNT, POINTS ARE SCALED BY PASSENGER COUNT
	ax = resultsPDDF.plot(kind='scatter', x= 'fare_amount', y = 'tip_amount', c='blue', alpha = 0.10, s=5*(resultsPDDF.passenger_count))
	ax.set_title('Tip amount by Fare amount ($)')
	ax.set_xlabel('Fare Amount')
	ax.set_ylabel('Tip Amount')
	plt.axis([-2, 120, -2, 30])
	plt.show()
	
	# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**Salida:**

![Distribución del importe de las propinas](./media/machine-learning-data-science-spark-advanced-data-exploration-modeling/tip-amount-distribution.png)

![Importe de las propinas por número de pasajeros](./media/machine-learning-data-science-spark-advanced-data-exploration-modeling/tip-amount-by-passenger-count.png)

![Importe de las propinas por importe de la tarifa](./media/machine-learning-data-science-spark-advanced-data-exploration-modeling/tip-amount-by-fare-amount.png)

Time taken to execute above cell: 10.42 seconds


## Diseño, transformación y preparación de los datos para el modelado de características

Esta sección describe y proporciona el código para los procedimientos que se usan para preparar los datos para su uso en el modelado de aprendizaje automático. Muestra cómo realizar las siguientes tareas:

- Creación de una nueva característica mediante la discretización de horas en cubos de tiempo de tráfico
- Indexación y codificación "uno de n" de características categóricas
- Creación de objetos de punto con etiqueta para la entrada en funciones de aprendizaje automático
- Creación de una submuestra aleatoria de datos y su división en conjuntos de entrenamiento y de pruebas
- Ajuste de la escala de las características
- Almacenamiento de objetos en caché


### Creación de una nueva característica mediante la discretización de horas en cubos de tiempo de tráfico

Este código muestra cómo crear una nueva característica mediante la discretización de horas en cubos de tiempo de tráfico y, después, cómo almacenar en caché la trama de datos resultante en memoria. Cuando se usan repetidamente conjuntos de datos distribuidos resistentes (RDD) y tramas de datos, el almacenamiento en caché mejora los tiempos de ejecución. Por lo tanto, almacenaremos en caché los RDD y las tramas de datos en varias fases del tutorial.

	# CREATE FOUR BUCKETS FOR TRAFFIC TIMES
	sqlStatement = """
	    SELECT *,
	    CASE
	     WHEN (pickup_hour <= 6 OR pickup_hour >= 20) THEN "Night" 
	     WHEN (pickup_hour >= 7 AND pickup_hour <= 10) THEN "AMRush" 
	     WHEN (pickup_hour >= 11 AND pickup_hour <= 15) THEN "Afternoon"
	     WHEN (pickup_hour >= 16 AND pickup_hour <= 19) THEN "PMRush"
	    END as TrafficTimeBins
	    FROM taxi_train 
	"""
	taxi_df_train_with_newFeatures = sqlContext.sql(sqlStatement)
	
	# CACHE DATA-FRAME IN MEMORY & MATERIALIZE DF IN MEMORY
	taxi_df_train_with_newFeatures.cache()
	taxi_df_train_with_newFeatures.count()

**SALIDA**

126050


### Indexación y codificación "uno de n" de características categóricas

Esta sección muestra cómo indexar o codificar las características categóricas para la entrada en las funciones de modelado. Las funciones de modelado y predicción de MLlib requieren características con datos de entrada categóricos indexados o codificados antes de usarlos.

Dependiendo del modelo, deberá indexarlos o codificarlos de maneras diferentes: Por ejemplo, los modelos de regresión logística y lineal requieren una codificación "uno de n", donde una característica con 3 categorías puede ampliarse a 3 columnas de característica, cada una de las cuales contiene 0 o 1 según la categoría de una observación. MLlib proporciona la función [OneHotEncoder](http://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.OneHotEncoder.html#sklearn.preprocessing.OneHotEncoder) para realizar la codificación "uno de n". Este codificador asigna una columna de índices de etiqueta a una columna de vectores binarios, con un solo valor uno como máximo. Esta codificación permite aplicar algoritmos que esperan características con valores numéricos, como la regresión logística, a características categóricas.

Este es el código para indexar y codificar características categóricas:


	# RECORD START TIME
	timestart = datetime.datetime.now()
	
	# LOAD PYSPARK LIBRARIES
	from pyspark.ml.feature import OneHotEncoder, StringIndexer, VectorAssembler, OneHotEncoder, VectorIndexer
	
	# INDEX AND ENCODE VENDOR_ID
	stringIndexer = StringIndexer(inputCol="vendor_id", outputCol="vendorIndex")
	model = stringIndexer.fit(taxi_df_train_with_newFeatures) # Input data-frame is the cleaned one from above
	indexed = model.transform(taxi_df_train_with_newFeatures)
	encoder = OneHotEncoder(dropLast=False, inputCol="vendorIndex", outputCol="vendorVec")
	encoded1 = encoder.transform(indexed)
	
	# INDEX AND ENCODE RATE_CODE
	stringIndexer = StringIndexer(inputCol="rate_code", outputCol="rateIndex")
	model = stringIndexer.fit(encoded1)
	indexed = model.transform(encoded1)
	encoder = OneHotEncoder(dropLast=False, inputCol="rateIndex", outputCol="rateVec")
	encoded2 = encoder.transform(indexed)
	
	# INDEX AND ENCODE PAYMENT_TYPE
	stringIndexer = StringIndexer(inputCol="payment_type", outputCol="paymentIndex")
	model = stringIndexer.fit(encoded2)
	indexed = model.transform(encoded2)
	encoder = OneHotEncoder(dropLast=False, inputCol="paymentIndex", outputCol="paymentVec")
	encoded3 = encoder.transform(indexed)
	
	# INDEX AND TRAFFIC TIME BINS
	stringIndexer = StringIndexer(inputCol="TrafficTimeBins", outputCol="TrafficTimeBinsIndex")
	model = stringIndexer.fit(encoded3)
	indexed = model.transform(encoded3)
	encoder = OneHotEncoder(dropLast=False, inputCol="TrafficTimeBinsIndex", outputCol="TrafficTimeBinsVec")
	encodedFinal = encoder.transform(indexed)
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 


**SALIDA**

Time taken to execute above cell: 1.22 seconds


### Creación de objetos de punto con etiqueta para la entrada en funciones de aprendizaje automático

Esta sección contiene código que muestra cómo indexar datos de texto categóricos como un tipo de datos de punto con etiqueta, y codificarlos para poder usarlos para entrenar y probar la regresión logística de MLlib y otros modelos de clasificación. Los objetos de punto con etiqueta son conjuntos de datos distribuidos resistentes (RDD) con el formato de datos de entrada que necesita la mayoría de los algoritmos de aprendizaje automático de MLlib. Un [punto con etiqueta](https://spark.apache.org/docs/latest/mllib-data-types.html#labeled-point) es un vector local, ya sea denso o disperso, asociado con una etiqueta o respuesta.


	# FUNCTIONS FOR BINARY CLASSIFICATION

	# LOAD PYSPARK LIBRARIES
	from pyspark.mllib.regression import LabeledPoint
	from numpy import array

	# INDEXING CATEGORICAL TEXT FEATURES FOR INPUT INTO TREE-BASED MODELS
	def parseRowIndexingBinary(line):
	    features = np.array([line.paymentIndex, line.vendorIndex, line.rateIndex, line.pickup_hour, line.weekday,
	                         line.passenger_count, line.trip_time_in_secs, line.trip_distance, line.fare_amount])
	    labPt = LabeledPoint(line.tipped, features)
	    return  labPt
	
	# ONE-HOT ENCODING OF CATEGORICAL TEXT FEATURES FOR INPUT INTO LOGISTIC RERESSION MODELS
	def parseRowOneHotBinary(line):
	    features = np.concatenate((np.array([line.pickup_hour, line.weekday, line.passenger_count,
	                                        line.trip_time_in_secs, line.trip_distance, line.fare_amount]), 
	                               line.vendorVec.toArray(), line.rateVec.toArray(), line.paymentVec.toArray()), axis=0)
	    labPt = LabeledPoint(line.tipped, features)
	    return  labPt


	# FUNCTIONS FOR REGRESSION WITH TIP AMOUNT AS TARGET VARIABLE

	# ONE-HOT ENCODING OF CATEGORICAL TEXT FEATURES FOR INPUT INTO TREE-BASED MODELS
	def parseRowIndexingRegression(line):
	    features = np.array([line.paymentIndex, line.vendorIndex, line.rateIndex, line.TrafficTimeBinsIndex, 
	                         line.pickup_hour, line.weekday, line.passenger_count, line.trip_time_in_secs, 
	                         line.trip_distance, line.fare_amount])
	    labPt = LabeledPoint(line.tip_amount, features)
	    return  labPt
	
	# INDEXING CATEGORICAL TEXT FEATURES FOR INPUT INTO LINEAR REGRESSION MODELS
	def parseRowOneHotRegression(line):
	    features = np.concatenate((np.array([line.pickup_hour, line.weekday, line.passenger_count,
	                                        line.trip_time_in_secs, line.trip_distance, line.fare_amount]), 
	                                        line.vendorVec.toArray(), line.rateVec.toArray(), 
	                                        line.paymentVec.toArray(), line.TrafficTimeBinsVec.toArray()), axis=0)
	    labPt = LabeledPoint(line.tip_amount, features)
	    return  labPt



### Creación de una submuestra aleatoria de datos y su división en conjuntos de entrenamiento y de pruebas

Este código crea una muestra aleatoria de los datos (aquí se usa el 25 %). Aunque no es necesario para este ejemplo debido al tamaño del conjunto de datos, se muestra cómo realizar la muestra para que sepa cómo hacerlo cuando lo necesite. Cuando las muestras son grandes, esto puede ahorrar mucho tiempo al entrenar modelos. Después, dividimos la muestra en una parte de entrenamiento (75 %) y una parte de pruebas (25 %) para el modelado de clasificación y regresión.


	# RECORD START TIME
	timestart = datetime.datetime.now()
	
	# SPECIFY SAMPLING AND SPLITTING FRACTIONS
	from pyspark.sql.functions import rand
	
	samplingFraction = 0.25;
	trainingFraction = 0.75; testingFraction = (1-trainingFraction);
	seed = 1234;
	encodedFinalSampled = encodedFinal.sample(False, samplingFraction, seed=seed)
	
	# SPLIT SAMPLED DATA-FRAME INTO TRAIN/TEST, WITH A RANDOM COLUMN ADDED FOR DOING CV (SHOWN LATER)
	# INCLUDE RAND COLUMN FOR CREATING CROSS-VALIDATION FOLDS
	dfTmpRand = encodedFinalSampled.select("*", rand(0).alias("rand"));
	trainData, testData = dfTmpRand.randomSplit([trainingFraction, testingFraction], seed=seed);
	
	# CACHE TRAIN AND TEST DATA
	trainData.cache()
	testData.cache()
	
	# FOR BINARY CLASSIFICATION TRAINING AND TESTING
	indexedTRAINbinary = trainData.map(parseRowIndexingBinary)
	indexedTESTbinary = testData.map(parseRowIndexingBinary)
	oneHotTRAINbinary = trainData.map(parseRowOneHotBinary)
	oneHotTESTbinary = testData.map(parseRowOneHotBinary)
	
	# FOR REGRESSION TRAINING AND TESTING
	indexedTRAINreg = trainData.map(parseRowIndexingRegression)
	indexedTESTreg = testData.map(parseRowIndexingRegression)
	oneHotTRAINreg = trainData.map(parseRowOneHotRegression)
	oneHotTESTreg = testData.map(parseRowOneHotRegression)
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**SALIDA**

Time taken to execute above cell: 0.4 seconds


### Ajuste de la escala de las características

El ajuste de la escala de las características, también conocido como normalización de los datos, garantiza que características con valores situados muy en los extremos no tengan un peso excesivo en la función objetivo. El código para ajustar la escala de las características usa [StandardScaler](https://spark.apache.org/docs/latest/api/python/pyspark.mllib.html#pyspark.mllib.feature.StandardScaler) para ajustarla a la varianza de la unidad. MLlib lo proporciona para su uso en la regresión lineal con descenso de gradiente estocástico (SGD), un popular algoritmo para entrenar una amplia variedad de otros modelos de aprendizaje automático tales como regresiones regularizadas o máquinas de vectores de soporte (SVM).


>[AZURE.NOTE] Hemos descubierto que el algoritmo LinearRegressionWithSGD es sensible al ajuste de la escala de las características.


Con este código, se ajusta la escala de variables para usarlas con el algoritmo SGD lineal regularizado.

	# RECORD START TIME
	timestart = datetime.datetime.now()
	
	# LOAD PYSPARK LIBRARIES
	from pyspark.mllib.regression import LabeledPoint
	from pyspark.mllib.linalg import Vectors
	from pyspark.mllib.feature import StandardScaler, StandardScalerModel
	from pyspark.mllib.util import MLUtils
	
	# SCALE VARIABLES FOR REGULARIZED LINEAR SGD ALGORITHM
	label = oneHotTRAINreg.map(lambda x: x.label)
	features = oneHotTRAINreg.map(lambda x: x.features)
	scaler = StandardScaler(withMean=False, withStd=True).fit(features)
	dataTMP = label.zip(scaler.transform(features.map(lambda x: Vectors.dense(x.toArray()))))
	oneHotTRAINregScaled = dataTMP.map(lambda x: LabeledPoint(x[0], x[1]))
	
	label = oneHotTESTreg.map(lambda x: x.label)
	features = oneHotTESTreg.map(lambda x: x.features)
	scaler = StandardScaler(withMean=False, withStd=True).fit(features)
	dataTMP = label.zip(scaler.transform(features.map(lambda x: Vectors.dense(x.toArray()))))
	oneHotTESTregScaled = dataTMP.map(lambda x: LabeledPoint(x[0], x[1]))
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**SALIDA**

Time taken to execute above cell: 7.33 seconds

### Almacenamiento de objetos en caché

Para reducir el tiempo necesario para entrenar y probar los algoritmos de aprendizaje automático, puede almacenar en caché los objetos de trama de datos de entrada usados para clasificación, regresión y características con ajuste de la escala.

	# RECORD START TIME
	timestart = datetime.datetime.now()
	
	# FOR BINARY CLASSIFICATION TRAINING AND TESTING
	indexedTRAINbinary.cache()
	indexedTESTbinary.cache()
	oneHotTRAINbinary.cache()
	oneHotTESTbinary.cache()
	
	# FOR REGRESSION TRAINING AND TESTING
	indexedTRAINreg.cache()
	indexedTESTreg.cache()
	oneHotTRAINreg.cache()
	oneHotTESTreg.cache()
	
	# SCALED FEATURES
	oneHotTRAINregScaled.cache()
	oneHotTESTregScaled.cache()
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**Salida:**

Time taken to execute above cell: 0.11 seconds


## Predicción de si se dio propina o no con modelos de clasificación binaria

Esta sección muestra cómo usar tres modelos para la tarea de clasificación binaria para predecir si se dio propina o no en una carrera de taxi. Los modelos que se presentan son:

- Regresión logística 
- Bosque aleatorio
- Árboles impulsados por gradiente

Cada sección de código de generación del modelo se dividirá en pasos:

1. Datos de **entrenamiento del modelo** con un conjunto de parámetros
2. **Evaluación del modelo** en un conjunto de datos de prueba con métricas
3. **Guardado del modelo** en un blob para su futuro consumo

Mostramos cómo realizar la validación cruzada (VC) con el barrido de parámetros de dos maneras:

1. Con código personalizado **genérico** que se puede aplicar a cualquier algoritmo de MLlib y a cualquier conjunto de parámetros en un algoritmo. 
1. Mediante la **función de canalización CrossValidator de pySpark**. Tenga en cuenta que aunque es útil, en nuestra experiencia, CrossValidator tiene algunas limitaciones para Spark 1.5.0: 

	- Los modelos de canalización no se pueden guardar ni conservar para usarlos en el futuro.
	- No se puede utilizar para todos los parámetros de un modelo.
	- No se puede usar para todos los algoritmos de MLlib.


### Validación cruzada y barrido de hiperparámetros genéricos usados con el algoritmo de regresión logística para la clasificación binaria

El código de esta sección muestra cómo entrenar, evaluar y guardar un modelo de regresión logística con [LBFGS](https://en.wikipedia.org/wiki/Broyden%E2%80%93Fletcher%E2%80%93Goldfarb%E2%80%93Shanno_algorithm), que predice si se dio propina o no en una carrera, en el conjunto de datos de carreras y tarifas de taxi de Nueva York. El modelo se entrena mediante la validación cruzada (VC) y el barrido de hiperparámetros, implementados con código personalizado que se puede aplicar a cualquiera de los algoritmos de aprendizaje en MLlib.


>[AZURE.NOTE] Se pueden tardar varios minutos en ejecutar este código de VC personalizado.


	# LOGISTIC REGRESSION CLASSIFICATION WITH CV AND HYPERPARAMETER SWEEPING

	# RECORD START TIME
	timestart = datetime.datetime.now()
	
	from pyspark.mllib.classification import LogisticRegressionWithLBFGS 
	from pyspark.mllib.evaluation import BinaryClassificationMetrics
	
	# CREATE PARAMETER GRID FOR LOGISTIC REGRESSION PARAMETER SWEEP
	from sklearn.grid_search import ParameterGrid
	grid = [{'regParam': [0.01, 0.1], 'iterations': [5, 10], 'regType': ["l1", "l2"], 'tolerance': [1e-3, 1e-4]}]
	paramGrid = list(ParameterGrid(grid))
	numModels = len(paramGrid)
	
	# SET NUM FOLDS AND NUM PARAMETER SETS TO SWEEP ON
	nFolds = 3;
	h = 1.0 / nFolds;
	metricSum = np.zeros(numModels);
	
	# BEGIN CV WITH PARAMETER SWEEP
	for i in range(nFolds):
	    # Create training and x-validation sets
	    validateLB = i * h
	    validateUB = (i + 1) * h
	    condition = (trainData["rand"] >= validateLB) & (trainData["rand"] < validateUB)
	    validation = trainData.filter(condition)
	    # Create labeled points from data-frames
	    if i > 0:
	        trainCVLabPt.unpersist()
	        validationLabPt.unpersist()
	    trainCV = trainData.filter(~condition)
	    trainCVLabPt = trainCV.map(parseRowOneHotBinary)
	    trainCVLabPt.cache()
	    validationLabPt = validation.map(parseRowOneHotBinary)
	    validationLabPt.cache()
	    # For parameter sets compute metrics from x-validation
	    for j in range(numModels):
	        regt = paramGrid[j]['regType']
	        regp = paramGrid[j]['regParam']
	        iters = paramGrid[j]['iterations']
	        tol = paramGrid[j]['tolerance']
	        # Train logistic regression model with hypermarameter set
	        model = LogisticRegressionWithLBFGS.train(trainCVLabPt, regType=regt, iterations=iters,  
	                                                  regParam=regp, tolerance = tol, intercept=True)
	        predictionAndLabels = validationLabPt.map(lambda lp: (float(model.predict(lp.features)), lp.label))
	        # Use ROC-AUC as accuracy metrics
	        validMetrics = BinaryClassificationMetrics(predictionAndLabels)
	        metric = validMetrics.areaUnderROC
	        metricSum[j] += metric
	
	avgAcc = metricSum / nFolds;
	bestParam = paramGrid[np.argmax(avgAcc)];
	
	# UNPERSIST OBJECTS
	trainCVLabPt.unpersist()
	validationLabPt.unpersist()
	    
	# TRAIN ON FULL TRAIING SET USING BEST PARAMETERS FROM CV/PARAMETER SWEEP
	logitBest = LogisticRegressionWithLBFGS.train(oneHotTRAINbinary, regType=bestParam['regType'], 
	                                              iterations=bestParam['iterations'], 
	                                              regParam=bestParam['regParam'], tolerance = bestParam['tolerance'], 
	                                              intercept=True)
	
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 


**SALIDA**

Time taken to execute above cell: 160.47 seconds


**Evaluación del modelo de clasificación binaria con métricas estándares y trazado de la curva ROC**

El código de esta sección muestra cómo evaluar un modelo de regresión logística con un conjunto de datos de prueba, incluido el trazado de la curva ROC.


	# RECORD START TIME
	timestart = datetime.datetime.now()
	
	# LOAD PYSPARK LIBRARIES
	from sklearn.metrics import roc_curve,auc
	from pyspark.mllib.evaluation import BinaryClassificationMetrics
	from pyspark.mllib.evaluation import MulticlassMetrics
	
	# PREDICT ON TEST DATA WITH BEST/FINAL MODEL
	predictionAndLabels = oneHotTESTbinary.map(lambda lp: (float(logitBest.predict(lp.features)), lp.label))
	
	# INSTANTIATE METRICS OBJECT
	metrics = BinaryClassificationMetrics(predictionAndLabels)

	# AREA UNDER PRECISION-RECALL CURVE
	print("Area under PR = %s" % metrics.areaUnderPR)

	# AREA UNDER ROC CURVE
	print("Area under ROC = %s" % metrics.areaUnderROC)
	metrics = MulticlassMetrics(predictionAndLabels)

	# OVERALL STATISTICS
	precision = metrics.precision()
	recall = metrics.recall()
	f1Score = metrics.fMeasure()
	print("Summary Stats")
	print("Precision = %s" % precision)
	print("Recall = %s" % recall)
	print("F1 Score = %s" % f1Score)
	
	
	# CREATE A PANDAS DATA-FRAME AND PLOT ROC-CURVE, FROM PREDICTED PROBS AND LABELS                                     
	logitBest.clearThreshold() # This clears threshold for classification (0.5) and outputs probabilities
	predictionAndLabels = oneHotTESTbinary.map(lambda lp: (float(logitBest.predict(lp.features)), lp.label))
	predictionAndLabelsDF = predictionAndLabels.toDF()
	test_predictions = predictionAndLabelsDF.toPandas()
	predictions_pddf = test_predictions.rename(columns={'_1': 'probability', '_2': 'label'})
	
	prob = predictions_pddf["probability"] 
	fpr, tpr, thresholds = roc_curve(predictions_pddf['label'], prob, pos_label=1);
	roc_auc = auc(fpr, tpr)
	
	# PLOT ROC CURVE
	plt.figure(figsize=(5,5))
	plt.plot(fpr, tpr, label='ROC curve (area = %0.2f)' % roc_auc)
	plt.plot([0, 1], [0, 1], 'k--')
	plt.xlim([0.0, 1.0])
	plt.ylim([0.0, 1.05])
	plt.xlabel('False Positive Rate')
	plt.ylabel('True Positive Rate')
	plt.title('ROC Curve')
	plt.legend(loc="lower right")
	plt.show()
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**SALIDA**

Area under PR = 0.985319161941

Area under ROC = 0.983511076103

Summary Stats

Precision = 0.984187223276

Recall = 0.984187223276

F1 Score = 0.984187223276

![Curva ROC de regresión logística para el enfoque genérico](./media/machine-learning-data-science-spark-advanced-data-exploration-modeling/logistic-regression-roc-curve.png)

Time taken to execute above cell: 5.02 seconds


**Persistencia del modelo en un blob para su futuro consumo**

El código de esta sección muestra cómo guardar el modelo de regresión logística para su consumo.

	# RECORD START TIME
	timestart = datetime.datetime.now()
	
	# LOAD PYSPARK LIBRARIES
	from pyspark.mllib.classification import LogisticRegressionModel
	
	# PERSIST MODEL
	datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
	logisticregressionfilename = "LogisticRegressionWithLBFGS_" + datestamp;
	dirfilename = modelDir + logisticregressionfilename;
	
	logitBest.save(sc, dirfilename);
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds";


**SALIDA**

Time taken to execute above cell: 9.96 seconds


### Uso de la función de canalización CrossValidator de MLlib con el modelo LogisticRegression (regresión elástica)

El código de esta sección muestra cómo entrenar, evaluar y guardar un modelo de regresión logística con [LBFGS](https://en.wikipedia.org/wiki/Broyden%E2%80%93Fletcher%E2%80%93Goldfarb%E2%80%93Shanno_algorithm), que predice si se dio propina o no en una carrera, en el conjunto de datos de carreras y tarifas de taxi de Nueva York. El modelo se entrena mediante la validación cruzada (VC) y el barrido de hiperparámetros, implementados con la función de canalización CrossValidator de MLlib para la VC con barrido de parámetros.


>[AZURE.NOTE] Se pueden tardar varios minutos en ejecutar este código de VC de MLlib.


	# RECORD START TIME
	timestart = datetime.datetime.now()
	
	# LOAD PYSPARK LIBRARIES
	from pyspark.ml.classification import LogisticRegression
	from pyspark.ml import Pipeline
	from pyspark.ml.evaluation import BinaryClassificationEvaluator
	from pyspark.ml.tuning import CrossValidator, ParamGridBuilder
	from sklearn.metrics import roc_curve,auc
	
	# DEFINE ALGORITHM / MODEL
	lr = LogisticRegression()
	
	# DEFINE GRID PARAMETERS
	paramGrid = ParamGridBuilder().addGrid(lr.regParam, (0.01, 0.1))\
	                              .addGrid(lr.maxIter, (5, 10))\
	                              .addGrid(lr.tol, (1e-4, 1e-5))\
	                              .addGrid(lr.elasticNetParam, (0.25,0.75))\
	                              .build()
	
	# DEFINE CV WITH PARAMETER SWEEP
	cv = CrossValidator(estimator= lr,
	                    estimatorParamMaps=paramGrid,
	                    evaluator=BinaryClassificationEvaluator(),
	                    numFolds=3)
	
	# CONVERT TO DATA-FRAME: THIS WILL NOT RUN ON RDDs
	trainDataFrame = sqlContext.createDataFrame(oneHotTRAINbinary, ["features", "label"])
	
	# TRAIN WITH CROSS-VALIDATION
	cv_model = cv.fit(trainDataFrame)
	

	## PREDICT AND EVALUATE ON TEST DATA-SET

	# USE TEST DATASET FOR PREDICTION
	testDataFrame = sqlContext.createDataFrame(oneHotTESTbinary, ["features", "label"])
	test_predictions = cv_model.transform(testDataFrame)
	
	# CONVERT RTO PANDAS DATA-FRAME FOR CALCULATING AND PLOTTING ROC CURVE
	predictions_pddf = test_predictions.toPandas()
	predictions_pddf.dtypes
	prob = [x[1] for x in predictions_pddf["probability"]]
	fpr, tpr, thresholds = roc_curve(predictions_pddf['label'], prob, pos_label=1);
	roc_auc = auc(fpr, tpr)
	
	# PLOT ROC CURVE
	plt.figure(figsize=(5,5))
	plt.plot(fpr, tpr, label='ROC curve (area = %0.2f)' % roc_auc)
	plt.plot([0, 1], [0, 1], 'k--')
	plt.xlim([0.0, 1.0])
	plt.ylim([0.0, 1.05])
	plt.xlabel('False Positive Rate')
	plt.ylabel('True Positive Rate')
	plt.title('ROC Curve')
	plt.legend(loc="lower right")
	plt.show()
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds";

**SALIDA**

![Curva ROC de regresión logística con CrossValidator de MLlib](./media/machine-learning-data-science-spark-advanced-data-exploration-modeling/mllib-crossvalidator-roc-curve.png)

Time taken to execute above cell: 118.25 seconds


### Clasificación de bosque aleatorio

El código de esta sección muestra cómo entrenar, evaluar y guardar una regresión con bosque aleatorio que predice si se dio propina o no en una carrera, en el conjunto de datos de carreras y tarifas de taxi de Nueva York.


	# RECORD START TIME
	timestart = datetime.datetime.now()
	
	# LOAD PYSPARK LIBRARIES
	from pyspark.mllib.tree import RandomForest, RandomForestModel
	from pyspark.mllib.util import MLUtils
	from pyspark.mllib.evaluation import BinaryClassificationMetrics
	from pyspark.mllib.evaluation import MulticlassMetrics
	
	# SPECIFY NUMBER OF CATEGORIES FOR CATEGORICAL FEATURES. FEATURE #0 HAS 2 CATEGORIES, FEATURE #2 HAS 2 CATEGORIES, AND SO ON
	categoricalFeaturesInfo={0:2, 1:2, 2:6, 3:4}
	
	# TRAIN RANDOMFOREST MODEL
	rfModel = RandomForest.trainClassifier(indexedTRAINbinary, numClasses=2, 
	                                       categoricalFeaturesInfo=categoricalFeaturesInfo,
	                                       numTrees=25, featureSubsetStrategy="auto",
	                                       impurity='gini', maxDepth=5, maxBins=32)
	## UN-COMMENT IF YOU WANT TO PRING TREES
	#print('Learned classification forest model:')
	#print(rfModel.toDebugString())
	
	# PREDICT ON TEST DATA AND EVALUATE
	predictions = rfModel.predict(indexedTESTbinary.map(lambda x: x.features))
	predictionAndLabels = indexedTESTbinary.map(lambda lp: lp.label).zip(predictions)
	
	# AREA UNDER ROC CURVE
	metrics = BinaryClassificationMetrics(predictionAndLabels)
	print("Area under ROC = %s" % metrics.areaUnderROC)
	
	# PERSIST MODEL IN BLOB
	datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
	rfclassificationfilename = "RandomForestClassification_" + datestamp;
	dirfilename = modelDir + rfclassificationfilename;
	
	rfModel.save(sc, dirfilename);
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 


**SALIDA**

Area under ROC = 0.985240932843

Time taken to execute above cell: 22.9 seconds


### Clasificación de árboles impulsados por gradiente

El código de esta sección muestra cómo entrenar, evaluar y guardar un modelo de árboles impulsados por gradiente que predice si se dio propina o no en una carrera, en el conjunto de datos de carreras de taxi y tarifas de Nueva York.


	# RECORD START TIME
	timestart = datetime.datetime.now()
	
	# LOAD PYSPARK LIBRARIES
	from pyspark.mllib.tree import GradientBoostedTrees, GradientBoostedTreesModel
	
	# SPECIFY NUMBER OF CATEGORIES FOR CATEGORICAL FEATURES. FEATURE #0 HAS 2 CATEGORIES, FEATURE #2 HAS 2 CATEGORIES, AND SO ON
	categoricalFeaturesInfo={0:2, 1:2, 2:6, 3:4}
	
	gbtModel = GradientBoostedTrees.trainClassifier(indexedTRAINbinary, categoricalFeaturesInfo=categoricalFeaturesInfo,
	                                                numIterations=10)
	## UNCOMMENT IF YOU WANT TO PRINT TREE DETAILS
	#print('Learned classification GBT model:')
	#print(bgtModel.toDebugString())
	
	# PREDICT ON TEST DATA AND EVALUATE
	predictions = gbtModel.predict(indexedTESTbinary.map(lambda x: x.features))
	predictionAndLabels = indexedTESTbinary.map(lambda lp: lp.label).zip(predictions)
	
	# Area under ROC curve
	metrics = BinaryClassificationMetrics(predictionAndLabels)
	print("Area under ROC = %s" % metrics.areaUnderROC)
	
	# PERSIST MODEL IN A BLOB
	datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
	btclassificationfilename = "GradientBoostingTreeClassification_" + datestamp;
	dirfilename = modelDir + btclassificationfilename;
	
	gbtModel.save(sc, dirfilename)
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**SALIDA**

Area under ROC = 0.985240932843

Time taken to execute above cell: 22.41 seconds


## Predicción de propinas con modelos de regresión (sin VC)

Esta sección muestra cómo usar tres modelos para la tarea de regresión para predecir el importe de la propina para una carrera de taxi en función de otras características de propina. Los modelos que se presentan son:

- Regresión lineal regularizada
- Bosque aleatorio
- Árboles impulsados por gradiente

Estos modelos se describieron en la introducción. Cada sección de código de generación del modelo se dividirá en pasos:

1. Datos de **entrenamiento del modelo** con un conjunto de parámetros
2. **Evaluación del modelo** en un conjunto de datos de prueba con métricas
3. **Guardado del modelo** en un blob para su futuro consumo   


>Nota de Azure: No se usa la validación cruzada con los tres modelos de regresión en esta sección. A cambio, se proporciona un ejemplo que muestra cómo usar la VC con una red elástica para la regresión lineal en el apéndice de este tema.


>Nota de Azure: Nuestra experiencia nos indica que puede haber problemas con la convergencia de los modelos LinearRegressionWithSGD, y es necesario cambiar u optimizar los parámetros cuidadosamente para obtener un modelo válido. El ajuste de la escala de las variables ayuda con la convergencia. La regresión con red elástica, que se muestra en el apéndice de este tema, también sirve para mejorar la convergencia.


### Regresión lineal con SGD

El código en esta sección muestra cómo usar características con ajuste de la escala para entrenar una regresión lineal que usa el descenso de gradiente estocástico (SGD) para la optimización, y cómo puntuar, evaluar y guardar el modelo en Almacenamiento de blobs de Azure (WASB).


	# LINEAR REGRESSION WITH SGD 

	# RECORD START TIME
	timestart = datetime.datetime.now()
	
	# LOAD PYSPARK LIBRARIES
	from pyspark.mllib.regression import LabeledPoint, LinearRegressionWithSGD, LinearRegressionModel
	from pyspark.mllib.evaluation import RegressionMetrics
	from scipy import stats
	
	# USE SCALED FEATURES TO TRAIN MODEL
	linearModel = LinearRegressionWithSGD.train(oneHotTRAINregScaled, iterations=100, step = 0.1, regType='l2', regParam=0.1, intercept = True)
	
	# SCORE ON SCALED TEST DATA-SET & EVALUATE
	predictionAndLabels = oneHotTESTregScaled.map(lambda lp: (float(linearModel.predict(lp.features)), lp.label))
	testMetrics = RegressionMetrics(predictionAndLabels)
	
	print("RMSE = %s" % testMetrics.rootMeanSquaredError)
	print("R-sqr = %s" % testMetrics.r2)
	
	# SAVE MODEL IN BLOB
	datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
	linearregressionfilename = "LinearRegressionWithSGD_" + datestamp;
	dirfilename = modelDir + linearregressionfilename;
	
	linearModel.save(sc, dirfilename)
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**SALIDA**

RMSE = 1.29395294535

R-sqr = 0.588405443258

Time taken to execute above cell: 36.14 seconds


### Regresión con bosque aleatorio

El código de esta sección muestra cómo entrenar, evaluar y guardar un modelo de bosque aleatorio que predice el importe de las propinas en los datos de carreras de taxi de Nueva York.


>[AZURE.NOTE] En el apéndice, se trata la validación cruzada con barrido de parámetros mediante código personalizado.


	#PREDICT TIP AMOUNTS USING RANDOM FOREST

	# RECORD START TIME
	timestart= datetime.datetime.now()
	
	# LOAD PYSPARK LIBRARIES
	from pyspark.mllib.tree import RandomForest, RandomForestModel
	from pyspark.mllib.util import MLUtils
	from pyspark.mllib.evaluation import RegressionMetrics
	
	
	# TRAIN MODEL
	categoricalFeaturesInfo={0:2, 1:2, 2:6, 3:4}
	rfModel = RandomForest.trainRegressor(indexedTRAINreg, categoricalFeaturesInfo=categoricalFeaturesInfo,
	                                    numTrees=25, featureSubsetStrategy="auto",
	                                    impurity='variance', maxDepth=10, maxBins=32)
	# UN-COMMENT IF YOU WANT TO PRING TREES
	#print('Learned classification forest model:')
	#print(rfModel.toDebugString())
	
	# PREDICT AND EVALUATE ON TEST DATA-SET
	predictions = rfModel.predict(indexedTESTreg.map(lambda x: x.features))
	predictionAndLabels = oneHotTESTreg.map(lambda lp: lp.label).zip(predictions)
	
	testMetrics = RegressionMetrics(predictionAndLabels)
	print("RMSE = %s" % testMetrics.rootMeanSquaredError)
	print("R-sqr = %s" % testMetrics.r2)
	
	# SAVE MODEL IN BLOB
	datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
	rfregressionfilename = "RandomForestRegression_" + datestamp;
	dirfilename = modelDir + rfregressionfilename;
	
	rfModel.save(sc, dirfilename);
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**SALIDA**

RMSE = 0.962262172157

R-sqr = 0.69142848223

Time taken to execute above cell: 29.3 seconds


### Regresión con árboles impulsados por gradiente

El código de esta sección muestra cómo entrenar, evaluar y guardar un modelo de árboles impulsados por gradiente que predice el importe de las propinas en los datos de carreras de taxi de Nueva York.


	#PREDICT TIP AMOUNTS USING GRADIENT BOOSTING TREES

	# RECORD START TIME
	timestart= datetime.datetime.now()
	
	# LOAD PYSPARK LIBRARIES
	from pyspark.mllib.tree import GradientBoostedTrees, GradientBoostedTreesModel
	from pyspark.mllib.util import MLUtils
	
	# TRAIN MODEL
	categoricalFeaturesInfo={0:2, 1:2, 2:6, 3:4}
	gbtModel = GradientBoostedTrees.trainRegressor(indexedTRAINreg, categoricalFeaturesInfo=categoricalFeaturesInfo, 
	                                                numIterations=10, maxBins=32, maxDepth = 4, learningRate=0.1)
	
	# EVALUATE A TEST DATA-SET
	predictions = gbtModel.predict(indexedTESTreg.map(lambda x: x.features))
	predictionAndLabels = indexedTESTreg.map(lambda lp: lp.label).zip(predictions)
	
	testMetrics = RegressionMetrics(predictionAndLabels)
	print("RMSE = %s" % testMetrics.rootMeanSquaredError)
	print("R-sqr = %s" % testMetrics.r2)
	
	# PLOT SCATTER-PLOT BETWEEN ACTUAL AND PREDICTED TIP VALUES
	test_predictions= sqlContext.createDataFrame(predictionAndLabels)
	test_predictions_pddf = test_predictions.toPandas()
	
	ax = test_predictions_pddf.plot(kind='scatter', figsize = (6,6), x='_1', y='_2', color='blue', alpha = 0.25, label='Actual vs. predicted');
	fit = np.polyfit(test_predictions_pddf['_1'], test_predictions_pddf['_2'], deg=1)
	ax.set_title('Actual vs. Predicted Tip Amounts ($)')
	ax.set_xlabel("Actual")
	ax.set_ylabel("Predicted")
	ax.plot(test_predictions_pddf['_1'], fit[0] * test_predictions_pddf['_1'] + fit[1], color='magenta')
	plt.axis([-1, 20, -1, 20])
	plt.show(ax)
	
	# SAVE MODEL IN BLOB
	datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
	btregressionfilename = "GradientBoostingTreeRegression_" + datestamp;
	dirfilename = modelDir + btregressionfilename;
	
	gbtModel.save(sc, dirfilename)
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 


**SALIDA**

RMSE = 0.962160568829

R-sqr = 0.717354800581

![Actual-vs-predicted-tip-amounts](./media/machine-learning-data-science-spark-advanced-data-exploration-modeling/actual-vs-predicted-tips.png)


## Apéndice: Tareas de regresión adicionales mediante validación cruzada con barridos de parámetros

Este apéndice contiene código que muestra cómo realizar la VC con red elástica para la regresión lineal y con barrido de parámetros mediante código personalizado para la regresión con bosque aleatorio.


### Validación cruzada con red elástica para la regresión lineal

El código de esta sección muestra cómo hacer la validación cruzada con red elástica para la regresión lineal y cómo evaluar el modelo con los datos de prueba.

	###  CV USING ELASTIC NET FOR LINEAR REGRESSION

	# RECORD START TIME
	timestart = datetime.datetime.now()
	
	# LOAD PYSPARK LIBRARIES
	from pyspark.ml.regression import LinearRegression
	from pyspark.ml import Pipeline
	from pyspark.ml.evaluation import RegressionEvaluator
	from pyspark.ml.tuning import CrossValidator, ParamGridBuilder
	
	# Define algo / model
	lr = LinearRegression()
	
	# Define grid parameters
	paramGrid = ParamGridBuilder().addGrid(lr.regParam, (0.01, 0.1))\
	                              .addGrid(lr.maxIter, (5, 10))\
	                              .addGrid(lr.tol, (1e-4, 1e-5))\
	                              .addGrid(lr.elasticNetParam, (0.25,0.75))\
	                              .build() 
	
	# Define pipeline, in this case, simply the modeling (without any transformations etc.)
	pipeline = Pipeline(stages=[lr])
	
	# Define CV with parameter sweep
	cv = CrossValidator(estimator= lr,
	                    estimatorParamMaps=paramGrid,
	                    evaluator=RegressionEvaluator(),
	                    numFolds=3)
	
	# Convert to data-frame, as CrossValidator won't run on RDDs
	trainDataFrame = sqlContext.createDataFrame(oneHotTRAINreg, ["features", "label"])
	
	# Train with cross-validation
	cv_model = cv.fit(trainDataFrame)
	
	timeend = datetime.datetime.now()
	timeend-timestart
	

	# EVALUATE MODEL ON TEST SET
	testDataFrame = sqlContext.createDataFrame(oneHotTESTreg, ["features", "label"])
	
	# MAKE PREDICTIONS ON TEST DOCUMENTS 
	#THE cvModel USES THE BEST MODEL FOUND (lrModel).
	test_predictions = cv_model.transform(testDataFrame)
	predictions_pddf = test_predictions.toPandas()
	
	corstats = stats.linregress(predictions_pddf['label'],predictions_pddf['prediction'])
	r2 = (corstats[2]*corstats[2])
	print("R-sqr = %s" % r2)
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 


**SALIDA**

R-sqr = 0.594830601664

Time taken to execute above cell: 129.51 seconds


### Validación cruzada con barrido de parámetros mediante código personalizado para la regresión con bosque aleatorio

El código de esta sección muestra cómo realizar la validación cruzada con barrido de parámetros mediante código personalizado para la regresión con bosque aleatorio y cómo evaluar el modelo con datos de prueba.


	# RECORD START TIME
	timestart= datetime.datetime.now()

	# LOAD PYSPARK LIBRARIES
	# GET ACCURARY FOR HYPERPARAMETERS BASED ON CROSS-VALIDATION IN TRAINING DATA-SET
	from pyspark.mllib.tree import RandomForest, RandomForestModel
	from pyspark.mllib.util import MLUtils
	from pyspark.mllib.evaluation import RegressionMetrics
	from sklearn.grid_search import ParameterGrid
	
	## CREATE PARAMETER GRID
	grid = [{'maxDepth': [5,10], 'numTrees': [25,50]}]
	paramGrid = list(ParameterGrid(grid))
	
	## SPECIFY LEVELS OF CATEGORICAL VARIBLES
	categoricalFeaturesInfo={0:2, 1:2, 2:6, 3:4}
	
	# SPECIFY NUMFOLDS AND ARRAY TO HOLD METRICS
	nFolds = 3;
	numModels = len(paramGrid)
	h = 1.0 / nFolds;
	metricSum = np.zeros(numModels);
	
	for i in range(nFolds):
	    # Create training and x-validation sets
	    validateLB = i * h
	    validateUB = (i + 1) * h
	    condition = (trainData["rand"] >= validateLB) & (trainData["rand"] < validateUB)
	    validation = trainData.filter(condition)
	    # Create labeled points from data-frames
	    if i > 0:
	        trainCVLabPt.unpersist()
	        validationLabPt.unpersist()
	    trainCV = trainData.filter(~condition)
	    trainCVLabPt = trainCV.map(parseRowIndexingRegression)
	    trainCVLabPt.cache()
	    validationLabPt = validation.map(parseRowIndexingRegression)
	    validationLabPt.cache()
	    # For parameter sets compute metrics from x-validation
	    for j in range(numModels):
	        maxD = paramGrid[j]['maxDepth']
	        numT = paramGrid[j]['numTrees']
	        # Train logistic regression model with hypermarameter set
	        rfModel = RandomForest.trainRegressor(trainCVLabPt, categoricalFeaturesInfo=categoricalFeaturesInfo,
	                                    numTrees=numT, featureSubsetStrategy="auto",
	                                    impurity='variance', maxDepth=maxD, maxBins=32)
	        predictions = rfModel.predict(validationLabPt.map(lambda x: x.features))
	        predictionAndLabels = validationLabPt.map(lambda lp: lp.label).zip(predictions)
	        # Use ROC-AUC as accuracy metrics
	        validMetrics = RegressionMetrics(predictionAndLabels)
	        metric = validMetrics.rootMeanSquaredError
	        metricSum[j] += metric
	
	avgAcc = metricSum/nFolds;
	bestParam = paramGrid[np.argmin(avgAcc)];
	
	# UNPERSIST OBJECTS
	trainCVLabPt.unpersist()
	validationLabPt.unpersist()
	        
	## TRAIN FINAL MODL WIHT BEST PARAMETERS
	rfModel = RandomForest.trainRegressor(indexedTRAINreg, categoricalFeaturesInfo=categoricalFeaturesInfo,
	                                    numTrees=bestParam['numTrees'], featureSubsetStrategy="auto",
	                                    impurity='variance', maxDepth=bestParam['maxDepth'], maxBins=32)

	# EVALUATE MODEL ON TEST DATA
	predictions = rfModel.predict(indexedTESTreg.map(lambda x: x.features))
	predictionAndLabels = indexedTESTreg.map(lambda lp: lp.label).zip(predictions)
	
	#PRINT TEST METRICS
	testMetrics = RegressionMetrics(predictionAndLabels)
	print("RMSE = %s" % testMetrics.rootMeanSquaredError)
	print("R-sqr = %s" % testMetrics.r2)
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 


**SALIDA**

RMSE = 0.990182456723

R-sqr = 0.609523627251

Time taken to execute above cell: 72.5 seconds


### Limpieza de objetos de la memoria e impresión de las ubicaciones de los modelos

Use `unpersist()` para eliminar objetos almacenados en la memoria caché.

	# UNPERSIST OBJECTS CACHED IN MEMORY

	# REMOVE ORIGINAL DFs
	taxi_df_train_cleaned.unpersist()
	taxi_df_train_with_newFeatures.unpersist()
	trainData.unpersist()
	trainData.unpersist()
	
	# FOR BINARY CLASSIFICATION TRAINING AND TESTING
	indexedTRAINbinary.unpersist()
	indexedTESTbinary.unpersist()
	oneHotTRAINbinary.unpersist()
	oneHotTESTbinary.unpersist()
	
	# FOR REGRESSION TRAINING AND TESTING
	indexedTRAINreg.unpersist()
	indexedTESTreg.unpersist()
	oneHotTRAINreg.unpersist()
	oneHotTESTreg.unpersist()
	
	# SCALED FEATURES
	oneHotTRAINregScaled.unpersist()
	oneHotTESTregScaled.unpersist()

	# PRINT MODEL FILE LOCATIONS FOR CONSUMPTION
	print "logisticRegFileLoc = modelDir + "" + logisticregressionfilename + """;
	print "linearRegFileLoc = modelDir + "" + linearregressionfilename + """;
	print "randomForestClassificationFileLoc = modelDir + "" + rfclassificationfilename + """;
	print "randomForestRegFileLoc = modelDir + "" + rfregressionfilename + """;
	print "BoostedTreeClassificationFileLoc = modelDir + "" + btclassificationfilename + """;
	print "BoostedTreeRegressionFileLoc = modelDir + "" + btregressionfilename + """;


**SALIDA**

PythonRDD[119] at RDD at PythonRDD.scala:43

logisticRegFileLoc = modelDir + "LogisticRegressionWithLBFGS\_2016-04-1817\_40\_35.796789"

linearRegFileLoc = modelDir + "LinearRegressionWithSGD\_2016-04-1817\_44\_00.993832"

randomForestClassificationFileLoc = modelDir + "RandomForestClassification\_2016-04-1817\_42\_58.899412"

randomForestRegFileLoc = modelDir + "RandomForestRegression\_2016-04-1817\_44\_27.204734"

BoostedTreeClassificationFileLoc = modelDir + "GradientBoostingTreeClassification\_2016-04-1817\_43\_16.354770"

BoostedTreeRegressionFileLoc = modelDir + "GradientBoostingTreeRegression\_2016-04-1817\_44\_46.206262"

## Pasos siguientes

Ahora que ha creado los modelos de clasificación y regresión con Spark MlLib, está listo para aprender a puntuar y evaluar estos modelos.

**Consumo de modelos:** para saber cómo puntuar y evaluar los modelos de clasificación y regresión creados en este tema, consulte [Score and evaluate Spark-built machine learning models](machine-learning-data-science-spark-model-consumption.md) (Evaluación y puntuación de modelos de aprendizaje automático creados con Spark).

<!---HONumber=AcomDC_0427_2016-->