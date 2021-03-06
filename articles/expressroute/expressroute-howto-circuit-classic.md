<properties
   pageTitle="Creación y modificación de un circuito ExpressRoute con el modelo de implementación clásica y PowerShell | Microsoft Azure"
   description="Este artículo le guiará por los pasos necesarios para crear y aprovisionar un circuito ExpressRoute. También se muestra cómo comprobar el estado, actualizar, o eliminar y desaprovisionar un circuito ExpressRoute."
   documentationCenter="na"
   services="expressroute"
   authors="ganesr"
   manager="carmonm"
   editor=""
   tags="azure-service-management"/>
<tags
   ms.service="expressroute"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="04/15/2016"
   ms.author="ganesr"/>

# Creación y modificación de un circuito ExpressRoute

> [AZURE.SELECTOR]
[Azure Portal - Resource Manager](expressroute-howto-circuit-portal-resource-manager.md)
[PowerShell - Resource Manager](expressroute-howto-circuit-arm.md)
[PowerShell - Classic](expressroute-howto-circuit-classic.md)

Este artículo le guiará por los pasos necesarios para crear un circuito ExpressRoute de Azure con los cmdlets de PowerShell y el modelo clásico de implementación. Los siguientes pasos también le mostrarán cómo comprobar el estado, actualizar, o eliminar y desaprovisionar un circuito ExpressRoute.

**Información acerca de los modelos de implementación de Azure**

[AZURE.INCLUDE [vpn-gateway-clasic-rm](../../includes/vpn-gateway-classic-rm-include.md)]


## Antes de empezar

- Necesitará la versión más reciente de los módulos de Azure PowerShell. Puede descargar los módulos de PowerShell más recientes desde la sección de PowerShell en la [página de descargas de Azure](https://azure.microsoft.com/downloads/). Para obtener instrucciones detalladas sobre cómo configurar el equipo para usar los módulos de Azure PowerShell, siga las instrucciones de la página [Cómo instalar y configurar Azure PowerShell](../powershell-install-configure.md).

- Asegúrese de haber revisado los [requisitos previos](expressroute-prerequisites.md) y los [flujos de trabajo](expressroute-workflows.md) antes de comenzar la configuración.

## Creación y aprovisionamiento de un circuito ExpressRoute

### 1\. Importación del módulo de PowerShell para ExpressRoute

 Tiene que importar los módulos de Azure y ExpressRoute en la sesión de PowerShell para poder usar los cmdlets de ExpressRoute. Para ello, ejecute los siguientes comandos:

	Import-Module 'C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\Azure.psd1'
	Import-Module 'C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\ExpressRoute\ExpressRoute.psd1'

### 2\. Obtención de la lista de proveedores, ubicaciones y anchos de banda admitidos

Para crear un circuito ExpressRoute, necesita la lista de proveedores de conectividad, ubicaciones y opciones de ancho de banda admitidas.

El cmdlet de PowerShell `Get-AzureDedicatedCircuitServiceProvider` devuelve esta información, que se usará en pasos posteriores:

	Get-AzureDedicatedCircuitServiceProvider

Compruebe si aparece su proveedor de conectividad. Tome nota de la siguiente información, porque la necesitará más adelante cuando cree un circuito:

- Nombre

- PeeringLocations

- BandwidthsOffered

Ahora está listo para crear un circuito ExpressRoute.

### 3\. Creación de un circuito ExpressRoute

En el ejemplo siguiente se muestra cómo crear un circuito ExpressRoute de 200 Mbps a través de Equinix en Silicon Valley. Si usa otro proveedor y otra configuración, sustituya esa información al realizar la solicitud.

>[AZURE.IMPORTANT] El circuito ExpressRoute se facturará a partir del momento en que se emita una clave de servicio. Asegúrese de realizar esta operación cuando el proveedor de conectividad esté listo para aprovisionar el circuito.


A continuación se muestra un ejemplo de solicitud para una nueva clave de servicio:

	$Bandwidth = 200
	$CircuitName = "MyTestCircuit"
	$ServiceProvider = "Equinix"
	$Location = "Silicon Valley"

	New-AzureDedicatedCircuit -CircuitName $CircuitName -ServiceProviderName $ServiceProvider -Bandwidth $Bandwidth -Location $Location -sku Standard -BillingType MeteredData

O bien, si desea crear un circuito ExpressRoute con el complemento Premium, use el siguiente ejemplo. Consulte la página [P+F de ExpressRoute](expressroute-faqs.md) para más información sobre el complemento Premium.

	New-AzureDedicatedCircuit -CircuitName $CircuitName -ServiceProviderName $ServiceProvider -Bandwidth $Bandwidth -Location $Location -sku Premium - BillingType MeteredData


La respuesta contendrá la clave del servicio. Puede obtener una descripción detallada de todos los parámetros ejecutando lo siguiente:

	get-help new-azurededicatedcircuit -detailed

### 4\. Lista de todos los circuitos ExpressRoute

Para obtener una lista de todos los circuitos ExpressRoute que haya creado, ejecute el comando `Get-AzureDedicatedCircuit`:


	Get-AzureDedicatedCircuit

La respuesta será similar al siguiente ejemplo:

	Bandwidth                        : 200
	CircuitName                      : MyTestCircuit
	Location                         : Silicon Valley
	ServiceKey                       : *********************************
	ServiceProviderName              : equinix
	ServiceProviderProvisioningState : NotProvisioned
	Sku                              : Standard
	Status                           : Enabled

Esta información se puede recuperar en cualquier momento con el cmdlet `Get-AzureDedicatedCircuit`. Si se realiza la llamada sin parámetros, se obtendrá una lista de todos los circuitos. La clave de servicio se mostrará en el campo *ServiceKey*.

	Get-AzureDedicatedCircuit

	Bandwidth                        : 200
	CircuitName                      : MyTestCircuit
	Location                         : Silicon Valley
	ServiceKey                       : *********************************
	ServiceProviderName              : equinix
	ServiceProviderProvisioningState : NotProvisioned
	Sku                              : Standard
	Status                           : Enabled

Puede obtener una descripción detallada de todos los parámetros ejecutando lo siguiente:

	get-help get-azurededicatedcircuit -detailed

### 5\. Envío de la clave de servicio al proveedor de conectividad para el aprovisionamiento


*ServiceProviderProvisioningState* da información sobre el estado actual del aprovisionamiento en el lado del proveedor de servicios. *Status* proporciona el estado relativo al lado de Microsoft. Para más información sobre los estados de aprovisionamiento del circuito, consulte el artículo [Flujos de trabajo de ExpressRoute para aprovisionamiento de circuitos y estados de circuitos de ExpressRoute](expressroute-workflows.md#expressroute-circuit-provisioning-states).

Cuando se crea un nuevo circuito ExpressRoute, dicho circuito estará en el siguiente estado:

	ServiceProviderProvisioningState : NotProvisioned
	Status                           : Enabled


El circuito pasará al estado siguiente cuando el proveedor de conectividad se encuentre en el proceso de habilitarlo:

	ServiceProviderProvisioningState : Provisioning
	Status                           : Enabled

Un circuito ExpressRoute tiene que estar en el siguiente estado para poder usarlo:

	ServiceProviderProvisioningState : Provisioned
	Status                           : Enabled


### 6\. Comprobación periódica del estado y la condición de la clave del circuito

Esto le permitirá saber cuándo ha habilitado el circuito el proveedor. Después de configurar el circuito, *ServiceProviderProvisioningState* aparece como *Provisioned*, tal como se muestra en el ejemplo siguiente:

	Get-AzureDedicatedCircuit

	Bandwidth                        : 200
	CircuitName                      : MyTestCircuit
	Location                         : Silicon Valley
	ServiceKey                       : *********************************
	ServiceProviderName              : equinix
	ServiceProviderProvisioningState : Provisioned
	Sku                              : Standard
	Status                           : Enabled

### 7\. Creación de la configuración de enrutamiento

Consulte el artículo [Configuración de enrutamiento de circuitos ExpressRoute (crear y modificar emparejamientos de circuito)](expressroute-howto-routing-classic.md) para obtener instrucciones paso a paso.

>[AZURE.IMPORTANT] Estas instrucciones se aplican solo a los circuitos creados con proveedores de servicios que ofrecen servicios de conectividad de nivel 2. Si usa un proveedor de servicios que ofrece servicios administrados de nivel 3 (normalmente VPN IP, como MPLS), el mismo proveedor de conectividad configurará y administrará el enrutamiento.

### 8\. Vinculación de una red virtual a un circuito ExpressRoute

A continuación, vincule una red virtual a su circuito ExpressRoute. Consulte [Vinculación de una red virtual a un circuito ExpressRoute](expressroute-howto-linkvnet-classic.md) para obtener instrucciones paso a paso. Si necesita crear una red virtual con el modelo de implementación clásica de ExpressRoute, consulte [Configuración de una red virtual para ExpressRoute en el Portal clásico](expressroute-howto-vnet-portal-classic.md) para obtener instrucciones.

## Obtención del estado de un circuito ExpressRoute

Esta información se puede recuperar en cualquier momento con el cmdlet `Get-AzureCircuit`. Si se realiza la llamada sin parámetros, se obtendrá una lista de todos los circuitos.

	Get-AzureDedicatedCircuit

	Bandwidth                        : 200
	CircuitName                      : MyTestCircuit
	Location                         : Silicon Valley
	ServiceKey                       : *********************************
	ServiceProviderName              : equinix
	ServiceProviderProvisioningState : Provisioned
	Sku                              : Standard
	Status                           : Enabled

	Bandwidth                        : 1000
	CircuitName                      : MyAsiaCircuit
	Location                         : Singapore
	ServiceKey                       : #################################
	ServiceProviderName              : equinix
	ServiceProviderProvisioningState : Provisioned
	Sku                              : Standard
	Status                           : Enabled

Puede obtener información sobre un circuito ExpressRoute específico, pasando la clave de servicio como un parámetro a la llamada:

	Get-AzureDedicatedCircuit -ServiceKey "*********************************"

	Bandwidth                        : 200
	CircuitName                      : MyTestCircuit
	Location                         : Silicon Valley
	ServiceKey                       : *********************************
	ServiceProviderName              : equinix
	ServiceProviderProvisioningState : Provisioned
	Sku                              : Standard
	Status                           : Enabled


Puede obtener una descripción detallada de todos los parámetros ejecutando lo siguiente:

	get-help get-azurededicatedcircuit -detailed

## Modificación de un circuito ExpressRoute

Puede modificar determinadas propiedades de un circuito ExpressRoute sin afectar a la conectividad.

Puede hacer lo siguiente sin experimentar tiempo de inactividad:

- Habilitar o deshabilitar el complemento ExpressRoute Premium en su circuito ExpressRoute.
- Aumentar el ancho de banda de su circuito ExpressRoute. Tenga en cuenta que no se admite la degradación del ancho de banda de un circuito.
- Cambio del plan de medición de datos limitados a datos ilimitados. Tenga en cuenta que no es posible cambiar el plan de medición de datos ilimitados a datos limitados.
- Puede habilitar y deshabilitar *Allow Classic Operations* (Permitir operaciones clásicas).

Consulte la página [P+F de ExpressRoute](expressroute-faqs.md) para más información sobre los límites y las limitaciones.

### Para habilitar el complemento ExpressRoute Premium

Puede habilitar el complemento ExpressRoute Premium en el circuito existente mediante el siguiente cmdlet de PowerShell:

	Set-AzureDedicatedCircuitProperties -ServiceKey "*********************************" -Sku Premium

	Bandwidth                        : 1000
	CircuitName                      : TestCircuit
	Location                         : Silicon Valley
	ServiceKey                       : *********************************
	ServiceProviderName              : equinix
	ServiceProviderProvisioningState : Provisioned
	Sku                              : Premium
	Status                           : Enabled

El circuito tendrá ahora las características del complemento ExpressRoute Premium habilitadas. Tenga en cuenta que la facturación de la capacidad del complemento Premium comienza en cuanto el comando se ejecuta correctamente.

### Para deshabilitar el complemento ExpressRoute Premium

>[AZURE.IMPORTANT] Esta operación puede producir un error si usa recursos que son más grandes de lo que está permitido para el circuito estándar.

Tenga en cuenta lo siguiente:

- Debe asegurarse de que el número de redes virtuales vinculadas al circuito es inferior a 10 antes de realizar la degradación de Premium a estándar. Si no lo hace, se producirá un error en la solicitud de actualización y se le facturará con las tarifas Premium.

- Tiene que desvincular todas las redes virtuales en otras regiones geopolíticas. Si no lo hace, se producirá un error en la solicitud de actualización y se le facturará con las tarifas Premium.

- La tabla de enrutamiento tiene que tener menos de 4.000 rutas para el emparejamiento entre pares privados. Si el tamaño de la tabla de ruta sobrepasa las 4.000 rutas, la sesión BGP se anulará y no se volverá a habilitar hasta que el número de prefijos anunciados esté por debajo de 4.000.

Puede deshabilitar el complemento ExpressRoute Premium en el circuito existente mediante el siguiente cmdlet de PowerShell:

	Set-AzureDedicatedCircuitProperties -ServiceKey "*********************************" -Sku Standard

	Bandwidth                        : 1000
	CircuitName                      : TestCircuit
	Location                         : Silicon Valley
	ServiceKey                       : *********************************
	ServiceProviderName              : equinix
	ServiceProviderProvisioningState : Provisioned
	Sku                              : Standard
	Status                           : Enabled



### Para actualizar el ancho de banda del circuito ExpressRoute

Consulte la página [P+F de ExpressRoute](expressroute-faqs.md) para conocer las opciones de ancho de banda compatibles con su proveedor. Puede elegir cualquier tamaño mayor que el tamaño de su circuito existente.

>[AZURE.IMPORTANT] No podrá reducir el ancho de banda de un circuito ExpressRoute sin interrupciones. Degradar de ancho de banda requiere que cancele el aprovisionamiento del circuito ExpressRoute y, a continuación, vuelva a aprovisionar un nuevo circuito ExpressRoute.

Después de decidir el tamaño que necesita, puede usar el comando siguiente para cambiar el tamaño del circuito:

	Set-AzureDedicatedCircuitProperties -ServiceKey ********************************* -Bandwidth 1000

	Bandwidth                        : 1000
	CircuitName                      : TestCircuit
	Location                         : Silicon Valley
	ServiceKey                       : *********************************
	ServiceProviderName              : equinix
	ServiceProviderProvisioningState : Provisioned
	Sku                              : Standard
	Status                           : Enabled

El circuito habrá cambiado de tamaño en el lado de Microsoft. Debe ponerse en contacto con su proveedor de conectividad para actualizar las configuraciones de su parte para que coincidan con este cambio. Tenga en cuenta que le facturará la opción de ancho de banda actualizada desde este momento.

## Eliminación y la cancelación de un circuito ExpressRoute

Tenga en cuenta lo siguiente:

- Para realizar esta operación correctamente, tiene que desvincular todas las redes virtuales del circuito ExpressRoute. Si se produce un error en esta operación compruebe si tiene alguna red virtual vinculada al circuito.

- Si está habilitado el estado de aprovisionamiento del proveedor de servicios del circuito ExpressRoute, el estado cambiará de habilitado a *deshabilitado*. Tiene que cooperar con su proveedor de servicios para desaprovisionar el circuito en su lado. Hasta que el proveedor de servicios finalice la cancelación del aprovisionamiento del circuito y nos envíe una notificación continuaremos reservando recursos y facturándole por ello.

- Si el proveedor de servicios ha desaprovisionado el circuito (el estado de aprovisionamiento del proveedor de servicios está establecido en *no aprovisionado*) antes de ejecutar el cmdlet anterior, cancelaremos el aprovisionamiento del circuito y dejaremos de facturarle.

Puede eliminar el circuito ExpressRout con la ejecución del siguiente comando:

	Remove-AzureDedicatedCircuit -ServiceKey "*********************************"



## Pasos siguientes

Después de crear el circuito, asegúrese de hacer lo siguiente:

- [Crear y modificar el enrutamiento para el circuito ExpressRoute](expressroute-howto-routing-classic.md)
- [Vincular la red virtual a su circuito ExpressRoute](expressroute-howto-linkvnet-classic.md)

<!---HONumber=AcomDC_0518_2016-->