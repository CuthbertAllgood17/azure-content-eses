<properties
   pageTitle="¿Están sus datos preparados para la ciencia de datos?: evaluación de datos | Microsoft Azure"
   description="Aprenda los cuatro criterios para que los datos estén preparado para la ciencia de datos. El vídeo 2 de Ciencia de datos para principiantes tiene ejemplos concretos para ayudar con la evaluación de datos básicos."
   keywords="evaluación de los datos, datos pertinentes, evaluar los datos, preparar datos, criterios de datos, datos preparados"
   services="machine-learning"
   documentationCenter="na"
   authors="brohrer-ms"
   manager="paulettm"
   editor="cjgronlund"/>

<tags
   ms.service="machine-learning"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="06/27/2016"
   ms.author="cgronlun;brohrer;garye"/>


# ¿Están sus datos preparados para la ciencia de datos? Ciencia de datos para principiantes, vídeo 2

Aprenda cómo evaluar los datos para asegurarse de que cumplen los criterios básicos para la ciencia de datos.

> [AZURE.VIDEO data-science-for-beginners-series-is-your-data-ready-for-data-science]

## Transcripción: ¿Están sus datos preparados para la ciencia de datos?

Bienvenido a "¿Están sus datos preparados para la ciencia de datos?", el segundo vídeo de la serie *Ciencia de datos para principiantes*.

Antes de que la ciencia de datos pueda proporcionar las respuestas que desea, tendrá que darle materia prima de gran calidad con la que trabajar. Al igual que al hacer una pizza, cuanto mejor sean los ingredientes con los que empiece, mejor será el producto final.

## Criterios para los datos

Por tanto, en el caso de la ciencia de datos, hay algunos ingredientes que necesitamos recopilar.

Necesitamos datos que sean:

  *	Pertinentes
  *	Conectado
  *	Precisos
  *	Suficientes para trabajar con ellos

## ¿Son sus datos pertinentes?

El primer ingrediente: necesitamos que los datos sean relevantes.

![Datos pertinentes frente a datos no pertinentes: evaluar los datos](./media/machine-learning-data-science-for-beginners-is-your-data-ready-for-data-science/machine-learning-data-science-relevant-and-irrelevant-data.png)

Fíjese en la tabla de la izquierda. Hemos encontrado a siete personas fuera de los bares de Boston, medido su nivel de alcohol de sangre, el promedio de bateo de los Red Sox en su último partido y el precio de la leche en la tienda más cercana.

Todos ellos son datos perfectamente legítimos. El único problema es que no son pertinentes. No hay ninguna relación obvia entre estos números. Si le di el precio actual de la leche y el promedio de bateo de los Red Sox, no hay forma de que pueda adivinar mi contenido de alcohol en sangre.

Ahora observe la tabla de la derecha. Esta vez, medimos la masa corporal de cada persona y contamos el número de bebidas que consumieron. Los números de cada fila ahora son pertinentes entre sí. Si le di mi masa corporal y el número de margaritas he bebido, podría realizar una estimación de mi contenido de alcohol en sangre.

## ¿Tiene datos conectados?

El siguiente ingrediente son unos datos conectados.

![Datos conectados frente a datos desconectados: criterios de datos](./media/machine-learning-data-science-for-beginners-is-your-data-ready-for-data-science/machine-learning-data-science-connected-vs-disconnected-data.png)

Estos son algunos datos que afectan a la calidad de las hamburguesas: temperatura de la parrilla, el peso de la carne y la clasificación en la revista de comida local. Pero observe los espacios vacíos en la tabla de la izquierda.

A la mayoría de los conjuntos de datos les faltan algunos valores. Es habitual tener espacios vacíos como estos y hay formas de trabajar para solucionarlos. Pero si faltan demasiados, los datos se empiezan a parecerse a un queso suizo.

Si observa la tabla de la izquierda, faltan tantos datos que es difícil establecer cualquier tipo de relación entre la temperatura de la parrilla y el peso de la carne. Este es un ejemplo de datos desconectados.

Sin embargo, la tabla de la derecha está llena y completa; un ejemplo de datos conectados.

## ¿Son precisos los datos?

El siguiente ingrediente que necesitamos es la precisión. Aquí hay 4 dianas a las que nos gustaría acertar con flechas.

![Datos precisos frente a datos imprecisos: criterios de datos](./media/machine-learning-data-science-for-beginners-is-your-data-ready-for-data-science/machine-learning-data-science-inaccurate-vs-accurate-data.png)

Mire la diana en la parte superior derecha. Tenemos una agrupación apretada justo en el blanco. Esto, por supuesto, es preciso. Curiosamente, en el lenguaje de la ciencia de datos, nuestro rendimiento en la diana inferior derecha también se considera preciso.

Si señala el centro de estas flechas, verá que está muy cerca del blanco. Las flechas se reparten alrededor de la diana, por lo que se consideran imprecisas, pero están centradas alrededor del blanco, por lo que se consideran precisas.

Ahora examinemos la diana superior izquierda. Aquí nuestras flechas dieron todas muy cerca, en una agrupación apretada. Son precisas, pero no lo son porque el centro está muy lejos del blanco. Y, por supuesto, las flechas en la diana inferior izquierda son inexactas e imprecisas. Este arquero necesita más práctica.

## ¿Tiene suficientes datos con los que trabajar?

Por último, el cuarto ingrediente: necesitamos tener suficientes datos.

![¿Tiene suficientes datos para el análisis? Evaluación de datos](./media/machine-learning-data-science-for-beginners-is-your-data-ready-for-data-science/machine-learning-data-science-barely-enough-data.png)

Piense que cada punto de datos de la tabla es una pincelada en una pintura. Si solo tiene algunas, la pintura puede ser bastante difuminada; es difícil saber qué es.

Si agrega algunas pinceladas más, la pintura empieza a estar un poco más nítida.

Cuando ya casi haya suficiente pinceladas, puede ver lo suficiente para tomar algunas decisiones generales. ¿Es un lugar que me gustaría visitar? Parece soleado y con agua limpia; sí, ahí es a donde voy a ir de vacaciones.

A medida que agrega más datos, la imagen se vuelve más clara y puede tomar decisiones más detalladas. Ahora puedo ver los tres hoteles en la orilla izquierda. Me gustan mucho las características arquitectónicas del hotel en primer plano. Me quedaré en él, en el tercer piso.

Con datos pertinentes, conectados, precisos y suficientes, tenemos todos los ingredientes que necesitamos para llevar a cabo ciencia de datos de gran calidad.

Asegúrese de ver los otros cuatro vídeos de *Ciencia de datos para principiantes* en Aprendizaje automático de Microsoft Azure.


## Otros vídeos de la serie

*Ciencia de datos para principiantes* es una introducción rápida a la ciencia de datos en cinco vídeos de corta duración.

  * Vídeo 1: [Las cinco preguntas a las que puede responder la ciencia de datos](machine-learning-data-science-for-beginners-the-5-questions-data-science-answers.md). **Ya disponible.**
  * Vídeo 3: [Realización de preguntas que pueden responderse con datos](machine-learning-data-science-for-beginners-ask-a-question-you-can-answer-with-data.md). **Ya disponible.**
  * Vídeo 4: [Predicción de respuestas con un modelo sencillo](machine-learning-data-science-for-beginners-predict-an-answer-with-a-simple-model.md). **Ya disponible.**
  * Vídeo 5: [Copia del trabajo de otras personas para realizar ciencia de datos](machine-learning-data-science-for-beginners-copy-other-peoples-work-to-do-data-science.md). Disponible el 30 de junio.


## Pasos siguientes

  * [Prueba de su primer experimento de ciencia de datos con Aprendizaje automático de Azure](machine-learning-create-experiment.md)
  * [Introducción a Aprendizaje automático en Microsoft Azure](machine-learning-what-is-machine-learning.md)

<!---HONumber=AcomDC_0629_2016-->