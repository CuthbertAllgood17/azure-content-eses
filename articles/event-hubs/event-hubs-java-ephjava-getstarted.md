<properties
	pageTitle="Introducción a los Centros de eventos en Java | Microsoft Azure"
	description="Siga este tutorial para empezar a usar Centros de eventos de Azure, a enviar eventos con C y a recibirlos con Java mediante EventProcessorHost."
	services="event-hubs"
	documentationCenter=""
	authors="fsautomata"
	manager="timlt"
	editor=""/>

<tags
	ms.service="event-hubs"
	ms.workload="core"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="06/16/2016"
	ms.author="jtaubensee"/>

# Introducción a los Centros de eventos

[AZURE.INCLUDE [service-bus-selector-get-started](../../includes/service-bus-selector-get-started.md)]

## Introducción

Centros de eventos es un sistema de recopilación de alta escalabilidad que puede recibir millones de eventos por segundo, habilitando una aplicación para procesar y analizar las grandes cantidades de datos generados por las aplicaciones y los dispositivos conectados. Una vez recopilados en los Centros de eventos, puede transformar y almacenar los datos usando cualquier proveedor de análisis en tiempo real o clúster de almacenamiento.

Para obtener más información, consulte [Información general sobre Centros de eventos][].

En este tutorial se muestra cómo insertar mensajes en un Centro de eventos mediante una aplicación de consola en Java y a recuperarlos en paralelo con la biblioteca Event Processor Host de Java.

Para completar este tutorial necesitará lo siguiente:

+ Un entorno de desarrollo de Java. Para este tutorial, asumimos que vamos a trabajar con [Eclipse](https://www.eclipse.org/).

+ Una cuenta de Azure activa. <br/>En caso de no tener ninguna, puede crear una cuenta gratuita en tan solo unos minutos. Para obtener más información, consulte <a href="http://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A0E0E5C02&amp;returnurl=http%3A%2F%2Fazure.microsoft.com%2Fes-ES%2Fdevelop%2Fmobile%2Ftutorials%2Fget-started%2F" target="_blank">Evaluación gratuita de Azure</a>.

[AZURE.INCLUDE [event-hubs-create-event-hub](../../includes/event-hubs-create-event-hub.md)]

[AZURE.INCLUDE [service-bus-event-hubs-get-started-send-java](../../includes/service-bus-event-hubs-get-started-send-java.md)]

[AZURE.INCLUDE [service-bus-event-hubs-get-started-receive-ephjava](../../includes/service-bus-event-hubs-get-started-receive-ephjava.md)]

## Ejecución de las aplicaciones

Ya está preparado para ejecutar las aplicaciones.

1.	Ejecute el proyecto **Receiver**.

	![][21]

2.	Ejecute el proyecto **Sender**.

	![][22]

## Pasos siguientes

Ahora que ha creado una aplicación de trabajo que crea un centro de eventos y envía y recibe datos, puede pasar a los siguientes escenarios:

- Una [aplicación de ejemplo completa que usa Centros de eventos][].
- El ejemplo [Escala horizontal del procesamiento de eventos con Centros de eventos][].
- Una [solución de mensajería en cola][] mediante las colas de Bus de servicio.

Para obtener más información, consulte el [Centro de desarrolladores de Java](/develop/java/).

<!-- Images. -->
[21]: ./media/event-hubs-java-ephjava-getstarted/ephjava.png
[22]: ./media/event-hubs-java-ephjava-getstarted/java-send.png

<!-- Links -->
[Azure classic portal]: https://manage.windowsazure.com/
[Información general sobre Centros de eventos]: event-hubs-overview.md
[aplicación de ejemplo completa que usa Centros de eventos]: https://code.msdn.microsoft.com/Service-Bus-Event-Hub-286fd097
[Escala horizontal del procesamiento de eventos con Centros de eventos]: https://code.msdn.microsoft.com/Service-Bus-Event-Hub-45f43fc3
[solución de mensajería en cola]: ../service-bus/service-bus-dotnet-multi-tier-app-using-service-bus-queues.md
 

<!---HONumber=AcomDC_0622_2016-->