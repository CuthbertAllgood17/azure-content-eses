<properties
   pageTitle="Visualización de operaciones de implementación con la API de REST | Microsoft Azure"
   description="Describe cómo usar la API de REST de Azure Resource Manager para detectar los problemas de la implementación de Resource Manager."
   services="azure-resource-manager,virtual-machines"
   documentationCenter=""
   tags="top-support-issue"
   authors="tfitzmac"
   manager="timlt"
   editor="tysonn"/>

<tags
   ms.service="azure-resource-manager"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="vm-multiple"
   ms.workload="infrastructure"
   ms.date="06/13/2016"
   ms.author="tomfitz"/>

# Visualización de operaciones de implementación con la API de REST de Azure Resource Manager

> [AZURE.SELECTOR]
- [Portal](resource-manager-troubleshoot-deployments-portal.md)
- [PowerShell](resource-manager-troubleshoot-deployments-powershell.md)
- [CLI de Azure](resource-manager-troubleshoot-deployments-cli.md)
- [API DE REST](resource-manager-troubleshoot-deployments-rest.md)

Si recibe un error al implementar recursos en Azure, quizás desee conocer más detalles acerca de las operaciones de implementación ejecutadas. La API de REST proporciona operaciones que permiten buscar los errores y determinar las posibles correcciones.

[AZURE.INCLUDE [resource-manager-troubleshoot-introduction](../includes/resource-manager-troubleshoot-introduction.md)]

Puede evitar algunos errores si valida la plantilla y la infraestructura antes de la implementación. También puede registrar información adicional sobre solicitudes y respuestas durante la implementación que podría resultarle de ayuda más adelante para solucionar los problemas. Para obtener más información sobre la validación, así como del registro de la información de solicitudes y respuestas, consulte [Deploy a resource group with Azure Resource Manager template](resource-group-template-deploy-rest.md) (Implementación de un grupo de recursos con la plantilla de Azure Resource Manager).

## Solución de problemas de la API de REST

1. Implemente los recursos con la operación [Crear una implementación de plantilla](https://msdn.microsoft.com/library/azure/dn790564.aspx). Para conservar información que puede ser útil para la depuración, establezca la propiedad **debugSetting** de la solicitud JSON en **requestContent** o **responseContent**. 

        PUT https://management.azure.com/subscriptions/{subscription-id}/resourcegroups/{resource-group-name}/providers/microsoft.resources/deployments/{deployment-name}?api-version={api-version}
          <common headers>
          {
            "properties": {
              "templateLink": {
                "uri": "http://mystorageaccount.blob.core.windows.net/templates/template.json",
                "contentVersion": "1.0.0.0",
              },
              "mode": "Incremental",
              "parametersLink": {
                "uri": "http://mystorageaccount.blob.core.windows.net/templates/parameters.json",
                "contentVersion": "1.0.0.0",      
              },
              "debugSetting": {
                "detailLevel": "requestContent, responseContent"
              }
            }
          }

    De forma predeterminada, el valor **debugSetting** está establecido en **none**. Al especificar el valor **debugSetting**, considere cuidadosamente el tipo de información que se pasa durante la implementación. Al registrar información sobre la solicitud o la respuesta, se podrían exponer datos confidenciales que se recuperan en las operaciones de implementación.

2. Obtenga información sobre la implementación con la operación [Obtener información sobre una implementación de plantilla](https://msdn.microsoft.com/library/azure/dn790565.aspx).

        GET https://management.azure.com/subscriptions/{subscription-id}/resourcegroups/{resource-group-name}/providers/microsoft.resources/deployments/{deployment-name}?api-version={api-version}

    En la respuesta, observe en particular los elementos **provisioningState**, **correlationId** y **error**. **correlationId** se usa para realizar un seguimiento de eventos relacionados y puede ser útil al colaborar con el soporte técnico para solucionar un problema.
    
        { 
          ...
          "properties": {
            "provisioningState":"Failed",
            "correlationId":"d5062e45-6e9f-4fd3-a0a0-6b2c56b15757",
            ...
            "error":{
              "code":"DeploymentFailed","message":"At least one resource deployment operation failed. Please list deployment operations for details. Please see http://aka.ms/arm-debug for usage details.",
              "details":[{"code":"Conflict","message":"{\r\n  "error": {\r\n    "message": "Conflict",\r\n    "code": "Conflict"\r\n  }\r\n}"}]
            }  
          }
        }

3. Obtenga información sobre las operaciones de implementación con la operación [Enumerar todas las operaciones de implementación de plantilla](https://msdn.microsoft.com/library/azure/dn790518.aspx).

        GET https://management.azure.com/subscriptions/{subscription-id}/resourcegroups/{resource-group-name}/providers/microsoft.resources/deployments/{deployment-name}/operations?$skiptoken={skiptoken}&api-version={api-version}

    La respuesta incluirá información de la solicitud o la respuesta según lo especificado en la propiedad **debugSetting** durante la implementación.
    
        {
          ...
          "properties": 
          {
            ...
            "request":{
              "content":{
                "location":"West US",
                "properties":{
                  "accountType": "Standard_LRS"
                }
              }
            },
            "response":{
              "content":{
                "error":{
                  "message":"Conflict","code":"Conflict"
                }
              }
            }
          }
        }

4. Obtenga eventos de los registros de auditoría para la implementación con la operación [Enumerar los eventos de administración de una suscripción](https://msdn.microsoft.com/library/azure/dn931934.aspx).

        GET https://management.azure.com/subscriptions/{subscription-id}/providers/microsoft.insights/eventtypes/management/values?api-version={api-version}&$filter={filter-expression}&$select={comma-separated-property-names}


## Pasos siguientes

- Para obtener ayuda con la resolución de errores de implementación concretos, consulte [Solución de problemas comunes al implementar recursos en Azure con Azure Resource Manager](resource-manager-common-deployment-errors.md).
- Para obtener más información acerca de cómo usar los registros de auditoría para supervisar otros tipos de acciones, consulte [Operaciones de auditoría con Resource Manager](resource-group-audit.md).
- Para validar la implementación antes de ejecutarla, consulte [Deploy a resource group with Azure Resource Manager template](resource-group-template-deploy.md) (Implementación de un grupo de recursos con la plantilla de Azure Resource Manager).

<!---HONumber=AcomDC_0615_2016-->