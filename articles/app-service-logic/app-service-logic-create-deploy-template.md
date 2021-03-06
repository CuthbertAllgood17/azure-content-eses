<properties
   pageTitle="Creación de una plantilla de implementación de aplicación lógica | Microsoft Azure"
   description="Obtenga información acerca de cómo crear una plantilla de implementación de aplicación lógica y usarla para la administración de versiones."
   services="app-service\logic"
   documentationCenter=".net,nodejs,java"
   authors="jeffhollan"
   manager="erikre"
   editor=""/>

<tags
   ms.service="app-service-logic"
   ms.devlang="multiple"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="integration"
   ms.date="05/25/2016"
   ms.author="jehollan"/>

# Creación de una plantilla de implementación de aplicación lógica

Una vez creada una aplicación lógica, quizá desee crearla como una plantilla de Azure Resource Manager. De este modo, podrá implementar fácilmente la aplicación lógica en cualquier entorno o grupo de recursos en los que pueda necesitarla. Para obtener una introducción a las plantillas de Resource Manager, asegúrese de consultar los artículos [Creación de plantillas de Azure Resource Manager](../resource-group-authoring-templates.md) e [Implementación de recursos con plantillas de Azure Resource Manager](../resource-group-template-deploy.md).

## Plantilla de implementación de aplicación lógica

Una aplicación lógica consta de tres componentes básicos:

* **Recurso de aplicación lógica**. Este recurso contiene información acerca de cuestiones como el plan de precios, la ubicación y la definición del flujo de trabajo.
* **Definición del flujo de trabajo**. Esto es lo que se ve en la vista de código. Incluye la definición de los pasos necesarios para el flujo y cómo debe ejecutarse el motor. Se trata de la propiedad `definition` del recurso de la aplicación lógica.
* **Conexiones**. Son recursos independientes que almacenan los metadatos de manera segura acerca de las conexiones del conector, como una cadena de conexión y un token de acceso. Se hace referencia a ellas en una aplicación lógica, en la sección `parameters` del recurso de la aplicación lógica.

Puede ver todas estas piezas de las aplicaciones lógicas existentes mediante herramientas como el [Explorador de recursos de Azure](http://resources.azure.com).

Para crear una plantilla para una aplicación lógica que pueda utilizar con implementaciones de grupos de recursos, es preciso definir los recursos y parametrizarlos según sea necesario. Por ejemplo, si va a implementar en un entorno de desarrollo, prueba y producción, probablemente deseará usar cadenas de conexión diferentes para una base de datos SQL en cada entorno. O bien, puede que desee realizar la implementación en diferentes suscripciones o grupos de recursos.

## Creación de una plantilla de implementación de aplicación lógica

Algunas herramientas pueden ayudarle a crear una plantilla de implementación de aplicación lógica. Puede crearla manualmente; es decir, mediante el uso de los recursos ya mencionados aquí para crear parámetros según sea necesario. Otra opción es usar un módulo de PowerShell [Logic App Template Creator](https://github.com/jeffhollan/LogicAppTemplateCreator). Este módulo de código abierto evalúa primero la aplicación lógica y las conexiones que utiliza y luego genera los recursos de la plantilla con los parámetros necesarios para la implementación. Por ejemplo, si tiene una aplicación lógica que recibe un mensaje de una cola de Bus de servicio de Azure y agrega datos a una base de datos SQL de Azure, la herramienta conservará toda la lógica de orquestación y parametrizará las cadenas de conexión de SQL y del Bus de servicio con el fin de que se puedan establecer en la implementación.

>[AZURE.NOTE] Las conexiones deben estar dentro del mismo grupo de recursos que la aplicación lógica.

### Instalación del módulo de PowerShell de plantilla de aplicación lógica

La forma más fácil de instalar el módulo es a través de la [Galería de PowerShell](https://www.powershellgallery.com/packages/LogicAppTemplate/0.1) con el comando `Install-Module -Name LogicAppTemplate`.

También puede instalar el módulo de PowerShell manualmente:

1. Descargue la versión más reciente de [Logic App Template Creator](https://github.com/jeffhollan/LogicAppTemplateCreator/releases).  
1. Extraiga la carpeta en su carpeta del módulo de PowerShell (normalmente `%UserProfile%\Documents\WindowsPowerShell\Modules`).

Para que el módulo funcione con cualquier token de acceso de inquilino y suscripción, se recomienda utilizarlo con la herramienta de línea de comandos [ARMClient](https://github.com/projectkudu/ARMClient). Esta [entrada de blog ](http://blog.davidebbo.com/2015/01/azure-resource-manager-client.html) describe ARMClient con más detalle.

### Generación de una plantilla de aplicación lógica mediante PowerShell

Después de instalar PowerShell, puede generar una plantilla mediante el comando siguiente:

`armclient token $SubscriptionId | Get-LogicAppTemplate -LogicApp MyApp -ResourceGroup MyRG -SubscriptionId $SubscriptionId -Verbose | Out-File C:\template.json`

`$SubscriptionId` es el identificador de la suscripción de Azure. Esta línea primero obtiene acceso al token mediante ARMClient, luego lo canaliza hacia el script de PowerShell y finalmente crea la plantilla en un archivo JSON.

## Adición de parámetros a una plantilla de aplicación lógica

Después de crear la plantilla de aplicación lógica, podrá seguir agregando o modificando los parámetros que necesite. Por ejemplo, si la definición incluye un identificador de recurso a una función o un flujo de trabajo anidado de Azure en el que planea implementar una sola implementación, puede agregar más recursos a la plantilla y parametrizar los identificadores según sea necesario. Lo mismo sucede con las referencias a las API personalizadas o a los puntos de conexión de Swagger que espera implementar con cada grupo de recursos.

## Implementación de una plantilla de aplicación lógica

Puede implementar su plantilla mediante cualquier número de herramientas, entre las que se incluyen PowerShell, API de REST, Visual Studio, Release Management o la implementación de plantillas del Portal de Azure. Consulte el artículo [Implementación de recursos con plantillas de Azure Resource Manager](../resource-group-template-deploy.md) para obtener más información. También se recomienda crear un [archivo de parámetros](../resource-group-template-deploy.md#parameter-file) para almacenar los valores del parámetro.

### Autorización de conexiones de OAuth

Después de la implementación, la aplicación lógica funciona de un extremo a otro con parámetros válidos. Sin embargo, aún será necesario autorizar las conexiones de OAuth para generar un token de acceso válido. Para ello, abra la aplicación lógica en el diseñador y, a continuación, autorice las conexiones. O bien, si desea automatizarlo, puede usar un script para dar el consentimiento a cada conexión de OAuth. Hay un script de ejemplo en GitHub, en el proyecto [LogicAppConnectionAuth](https://github.com/logicappsio/LogicAppConnectionAuth).

## Visual Studio Release Management

Un escenario común para implementar y administrar un entorno es usar una herramienta como Visual Studio Release Management con una plantilla de implementación de aplicación lógica. Visual Studio Team Services incluye la tarea [Implementación de un grupo de recursos de Azure](https://github.com/Microsoft/vsts-tasks/tree/master/Tasks/DeployAzureResourceGroup), que se puede agregar a cualquier canalización de la compilación o versión. Es preciso tener una [entidad de servicio](https://blogs.msdn.microsoft.com/visualstudioalm/2015/10/04/automating-azure-resource-group-deployment-using-a-service-principal-in-visual-studio-online-buildrelease-management/) para implementar la autorización y, a continuación, generar la definición de la versión.

1. En Release Management, para crear una nueva definición, seleccione **Vacío** para comenzar con una definición vacía.

    ![Crear una definición nueva y vacía][1]

1. Elija los recursos que necesite. Es probable que sea la plantilla de aplicación lógica generada manualmente o como parte del proceso de compilación.
1. Agregue una tarea de **Implementación de un grupo de recursos de Azure**.
1. Configure una [entidad de servicio](https://blogs.msdn.microsoft.com/visualstudioalm/2015/10/04/automating-azure-resource-group-deployment-using-a-service-principal-in-visual-studio-online-buildrelease-management/) y haga referencia a los archivos de plantilla y de parámetros de la plantilla.
1. Siga compilando los pasos del proceso de lanzamiento de otros entornos, pruebas automatizadas o aprobadores según sea necesario.

<!-- Image References -->
[1]: ./media/app-service-logic-create-deploy-template/emptyReleaseDefinition.PNG

<!---HONumber=AcomDC_0615_2016-->