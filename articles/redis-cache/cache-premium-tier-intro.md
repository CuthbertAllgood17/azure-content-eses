<properties 
	pageTitle="Introducción al nivel Premium de Caché en Redis de Azure | Microsoft Azure" 
	description="Aprenda a crear y a administrar la persistencia de Redis, la agrupación en clústeres de Redis y la compatibilidad de red virtual para las instancias de Caché en Redis de Azure de nivel Premium." 
	services="redis-cache" 
	documentationCenter="" 
	authors="steved0x" 
	manager="douge" 
	editor=""/>

<tags 
	ms.service="cache" 
	ms.workload="tbd" 
	ms.tgt_pltfrm="cache-redis" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="06/29/2016" 
	ms.author="sdanie"/>

# Introducción al nivel Premium de Caché en Redis de Azure
Caché en Redis de Azure es una memoria caché distribuida y administrada que ayuda a compilar aplicaciones muy útiles y altamente escalables mediante el acceso ultrarrápido a los datos.

Premium es un nuevo nivel destinado a las empresas que incluye todas las características del nivel Estándar y otras adicionales, como un mejor rendimiento, cargas de trabajo más grandes, recuperación ante desastres, importación/exportación y seguridad mejorada. Siga leyendo para obtener más información acerca de las características adicionales de la memoria caché del nivel Premium.

## Mejor rendimiento en comparación con el nivel Estándar o Básico
**Mejor rendimiento respecto a los niveles Standard o Basic.** Las memorias caché de nivel Premium se implementan en hardware con procesadores más rápidos que ofrece un mejor rendimiento en comparación con el nivel Standard o Basic. Además, el rendimiento de dichas memorias caché es superior y sus latencias son más bajas.

**El rendimiento de una memoria caché de nivel Premium es superior al de una memoria caché de nivel Standard del mismo tamaño .** Por ejemplo, el rendimiento de una memoria caché de 53 GB P4 (Premium) es de 250 000 solicitudes por segundo, en comparación con 150 000 para una memoria C6 (Standard).

Consulte el artículo [P+F de Caché en Redis de Azure](cache-faq.md#what-redis-cache-offering-and-size-should-i-use) para obtener más detalles sobre el tamaño, el rendimiento y el ancho de banda de las memorias caché Premium.

## Persistencia de datos de Redis
El nivel Premium permite conservar los datos de la memoria caché en una cuenta de almacenamiento de Azure. En las memorias caché de nivel Basic o Standard, todos los datos se almacenan únicamente en la memoria. En caso de problemas con la infraestructura subyacente, podría producirse una pérdida de los datos. Se recomienda usar la característica de persistencia de datos de Redis del nivel Premium para aumentar la resistencia frente a la pérdida de datos. Caché en Redis de Azure ofrece las opciones RDB y AOF (próximamente) en la [persistencia de Redis](http://redis.io/topics/persistence).

Para obtener instrucciones acerca de cómo configurar la persistencia, consulte [Configuración de la persistencia para una Caché en Redis de Azure de nivel Premium](cache-how-to-premium-persistence.md).

##Clúster Redis
Si desea crear memorias caché de más de 53 GB o particionar los datos entre varios nodos de Redis, puede usar la agrupación en clústeres Redis, disponible en el nivel Premium. Cada nodo consta de un par de cachés principal-réplica administrado por Azure para ofrecer una alta disponibilidad.

**La agrupación en clústeres de Redis proporciona una escalabilidad y un rendimiento máximos.** El rendimiento aumenta de manera lineal a medida que aumenta el número de particiones (nodos) del clúster. Por ejemplo, si se crea un clúster P4 de 10 particiones, el rendimiento posible es de 250 000 * 10 = 2,5 millones de solicitudes por segundo. Consulte el artículo [P+F de Caché en Redis de Azure](cache-faq.md#what-redis-cache-offering-and-size-should-i-use) para obtener más detalles sobre el tamaño, el rendimiento y el ancho de banda de las memorias caché Premium.

Para empezar a usar la agrupación en clústeres, consulte [Configuración de la agrupación en clústeres para Caché en Redis de Azure de nivel Premium](cache-how-to-premium-clustering.md).

##Aislamiento y seguridad mejorados

Las memorias caché creadas en los niveles Basic o Standard están disponibles en Internet. El acceso a las mismas se restringe mediante la clave de acceso. Con el nivel Premium puede asegurarse de que solo los clientes de una red determinada puedan tener acceso a la memoria caché. Puede implementar la Caché en Redis en una [Red Virtual de Azure (VNet)](https://azure.microsoft.com/services/virtual-network/). Además, puede usar todas las características de la red virtual, como las subredes y las directivas de control de acceso, entre otras, para restringir aún más el acceso a Redis.

Para obtener más información, vea [Configuración de la compatibilidad de Red virtual para una Caché en Redis de Azure Premium](cache-how-to-premium-vnet.md).

## Importación/Exportación

Importación/Exportación es una operación de administración de datos de Caché en Redis de Azure que permite importar datos en Caché en Redis de Azure o exportar datos desde Caché en Redis de Azure mediante la importación y exportación de una instantánea de base de datos de Caché en Redis (RDB) desde una caché premium a un blob en páginas en una cuenta de Almacenamiento de Azure. Este método le permite migrar entre diferentes instancias de Caché en Redis de Azure o rellenar la caché de datos antes de su uso.

La importación se puede usar para traer los archivos RDB compatibles de Redis desde cualquier servidor de Redis que se ejecute en cualquier nube o entorno, incluidas las instancias de Redis que se ejecutan en Linux, Windows o cualquier proveedor en la nube como, por ejemplo, Amazon Web Services. La importación de datos supone una manera fácil de crear una caché con datos rellenados previamente. Durante el proceso de importación, Caché en Redis de Azure carga los archivos RDB desde Almacenamiento de Azure en la memoria y, a continuación, inserta las claves en la memoria caché.

La exportación permite exportar los datos almacenados en la Caché en Redis de Azure a archivos RDB compatibles. Puede utilizar esta característica para mover datos desde una instancia de Caché en Redis de Azure a otra o a otro servidor de Redis. Durante el proceso de exportación, se crea un archivo temporal en la máquina virtual que hospeda la instancia del servidor de Caché en Redis de Azure y el archivo se carga en la cuenta de almacenamiento designada. Una vez completada la operación de exportación (de manera correcta o incorrecta), se elimina el archivo temporal.

Para obtener más información, consulte [How to import data into and export data from Azure Redis Cache](cache-how-to-import-export-data.md) (Importación de datos en Caché en Redis de Azure y exportación de datos desde este servicio).

## Reboot

El nivel premium permite reiniciar uno o varios nodos de la memoria caché a petición. De este modo, podrá probar la resiliencia de la aplicación en caso de error. Puede reiniciar los siguientes nodos.

-	Nodo maestro de la memoria caché
-	Nodo subordinado de la memoria caché
-	Nodos maestro y subordinado de la memoria caché
-	Cuando se utiliza una caché premium con agrupación en clústeres, puede reiniciar el nodo maestro o el subordinado (o ambos) para particiones individuales en la memoria caché

Para obtener más información, consulte [Reboot](cache-administration.md#reboot) (Reinicio) y [Reboot FAQ](cache-administration.md#reboot-faq) (P+F de Reinicio).

## Programar actualizaciones

La característica de actualizaciones programadas permite designar una ventana de mantenimiento para la memoria caché. Cuando se especifica la ventana de mantenimiento, las actualizaciones del servidor Redis se realizan en ese período. Para designar una ventana de mantenimiento, seleccione los días deseados y especifique la hora de inicio de la ventana de mantenimiento para cada día. Tenga en cuenta que la hora de la ventana de mantenimiento está en formato UTC.

Para obtener más información, consulte [Schedule updates](cache-administration.md#schedule-updates) (Programar actualizaciones) y [Schedule updates FAQ](cache-administration.md#schedule-updates-faq) (P+F de la programación de actualizaciones).

>[AZURE.NOTE] Las actualizaciones del servidor Redis solo se actualizan durante la ventana de mantenimiento programada. La ventana de mantenimiento no se aplica a las actualizaciones de Azure o a las actualizaciones del sistema operativo de la máquina virtual.

## Para escalar al nivel premium

Para escalar al nivel premium, basta con elegir uno de los niveles premium en la hoja **Cambiar el plan de tarifa**. También puede escalar la memoria caché al nivel premium con PowerShell y la CLI. Para obtener instrucciones detalladas, consulte [Escalado de Caché en Redis de Azure](cache-how-to-scale.md) y [Automatización de una operación de escalado](cache-how-to-scale.md#how-to-automate-a-scaling-operation).

## Pasos siguientes

Cree una memoria caché y explore las nuevas características del nivel Premium.

-	[Cómo configurar la persistencia para una memoria Caché en Redis de Azure Premium](cache-how-to-premium-persistence.md)
-	[Cómo configurar la compatibilidad de red virtual para una memoria Caché en Redis de Azure Premium](cache-how-to-premium-vnet.md)
-	[Cómo configurar la agrupación en clústeres para una memoria Caché en Redis de Azure Premium](cache-how-to-premium-clustering.md)
-	[How to import data into and export data from Azure Redis Cache (Importación de datos en Caché en Redis de Azure y exportación de datos desde este servicio).](cache-how-to-import-export-data.md)
-	[How to administer Azure Redis Cache](cache-administration.md) (Administración de la Caché en Redis de Azure)
  

<!---HONumber=AcomDC_0629_2016-->