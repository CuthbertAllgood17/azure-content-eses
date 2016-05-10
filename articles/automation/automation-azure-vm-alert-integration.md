<properties
    pageTitle="Corrección de alertas de la máquina virtual de Azure con runbooks de automatización | Microsoft Azure"
    description="En este artículo se muestra cómo integrar las alertas de la máquina virtual de Azure con runbooks de Automatización de Azure y problemas de corrección automática"
    services="automation"
    documentationCenter=""
    authors="csand-msft"
    manager="stevenka"
    editor="tysonn" />    
<tags
    ms.service="automation"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="04/11/2016"
    ms.author="csand;magoedte" />

# Solución de Automatización de Azure: corrección de las alertas de la máquina virtual de Azure

Automatización de Azure y Máquinas virtuales de Azure han publicado una nueva característica que permite configurar las alertas de la máquina virtual (VM) para ejecutar runbooks de automatización. Esta nueva funcionalidad le permite realizar automáticamente la corrección estándar de alertas de la máquina virtual, como reiniciar o detener la máquina virtual.

Anteriormente, durante la creación de la regla de alertas de la máquina virtual podía [especificar un Webhook de automatización](https://azure.microsoft.com/blog/using-azure-automation-to-take-actions-on-azure-alerts/) en un runbook para ejecutar el runbook cada vez que se desencadene la alerta. Sin embargo, para esto es necesario que realice el trabajo de creación del runbook, la creación del Webhook para el runbook y, después, la copia y pegado del Webhook durante la creación de la regla de alertas. Con esta nueva versión, el proceso es mucho más fácil porque puede elegir directamente un runbook de una lista durante la creación de la regla de alertas, y puede elegir una cuenta de automatización que ejecute el runbook o crear fácilmente una cuenta.

En este artículo, le mostraremos lo fácil que es establecer una alerta de máquina virtual de Azure y configurar un runbook de automatización para ejecutar cada vez que se desencadena la alerta. Entre los escenarios de ejemplo se incluyen el tener que reiniciar una máquina virtual cuando el uso de memoria supera cierto límite debido a que una aplicación de la misma tiene una fuga de memoria, o el detener la máquina virtual si el tiempo de usuario de CPU ha sido inferior al 1% durante la última hora y no está en uso. También explicaremos cómo la creación automatizada de una entidad de servicio en la cuenta de automatización simplifica el uso de runbooks en la corrección de alertas de Azure.

## Creación de una alerta para una máquina virtual

Realice los pasos siguientes para configurar una alerta para iniciar un runbook cuando se haya alcanzado el umbral.

>[AZURE.NOTE] En esta versión solo se admiten máquinas virtuales de V2 y pronto se agregará el soporte técnico para máquinas virtuales clásicas.

1. Inicie sesión en el Portal de Azure y haga clic en **Máquinas virtuales**.  
2. Seleccione una de las máquinas virtuales. Aparecerá la hoja de panel de la máquina virtual y la hoja **Configuración** a la derecha.  
3. En la hoja **Configuración**, en la sección de supervisión, elija **Reglas de alerta**.
4. En la hoja **Reglas de alerta**, haga clic en **Agregar alerta**.

Esto abre la hoja **Agregar una regla de alerta**, donde puede configurar las condiciones de la alerta y elegir entre una o todas estas opciones: enviar un correo electrónico a alguien, utilizar un Webhook para reenviar la alerta a otro sistema o ejecutar un runbook de automatización para intentar corregir el problema.

## Configuración de un runbook

Para configurar un runbook parar que se ejecute cuando se cumpla el umbral de alerta de la máquina virtual, seleccione **Runbook de automatización**. En la hoja **Configurar runbook**, puede seleccionar el runbook que desea ejecutar y la cuenta de automatización en la que quiere ejecutarlo.

![Configuración de un runbook de automatización y creación de una nueva cuenta de automatización](media/automation-azure-vm-alert-integration/ConfigureRunbookNewAccount.png)

>[AZURE.NOTE] En esta versión puede elegir entre los tres runbooks que proporciona el servicio: reiniciar, detener o quitar (eliminar) la máquina virtual. La capacidad para seleccionar otros runbooks o uno de sus propios runbooks estará disponible en próximas versiones.

![Runbooks que puede elegir](media/automation-azure-vm-alert-integration/RunbooksToChoose.png)

Después de seleccionar uno de los tres runbooks disponibles, aparecerá la lista desplegable **Cuenta de automatización** y podrá seleccionar la cuenta de automatización en la que se ejecutará el runbook. Los runbooks se deben ejecutar en el contexto de una [cuenta de automatización](automation-security-overview.md) que se incluye en su suscripción de Azure. Puede seleccionar una cuenta de automatización que ya haya creado o puede crear una nueva.

Los runbooks proporcionados se autentican en Azure mediante una entidad de servicio. Si elige ejecutar el runbook en una de sus cuentas de automatización existentes, la entidad de servicio se creará automáticamente. Si decide crear una nueva cuenta de automatización, se creará automáticamente la cuenta y la entidad de servicio. En ambos casos, también se crearán dos recursos en la cuenta de automatización: un recurso de certificado denominado **AzureRunAsCertificate** y un recurso de conexión denominado **AzureRunAsConnection**. Los runbooks utilizarán **AzureRunAsConnection** para autenticarse con Azure para realizar la acción de administración en la máquina virtual.

>[AZURE.NOTE] La entidad de servicio se crea en el ámbito de la suscripción y se le asigna el rol Colaborador. Este rol es necesario para otorgar permiso a la cuenta para ejecutar runbooks de automatización para administrar máquinas virtuales de Azure. La creación de una cuenta de automatización o una entidad de servicio es un evento único. Una vez creados, puede utilizar esa cuenta para ejecutar runbooks para otras alertas de la máquina virtual de Azure.

Al hacer clic en **Aceptar** se configurará la alerta y si ha seleccionado la opción para crear una nueva cuenta de automatización, esta se creará junto con la entidad de servicio. Este proceso puede tardar unos segundos en completarse.

![Runbook que se está configurando](media/automation-azure-vm-alert-integration/RunbookBeingConfigured.png)

Una vez completada la configuración aparecerá el nombre del runbook en la hoja **Agregar una regla de alerta**.

![Runbook configurado](media/automation-azure-vm-alert-integration/RunbookConfigured.png)

Haga clic en **Aceptar** en la hoja **Agregar una regla de alerta** y se creará la regla de alerta y se activará si la máquina virtual está en estado de ejecución.

### Habilitamiento o deshabilitamiento de un runbook

Si tiene un runbook configurado para una alerta, puede deshabilitarlo sin quitar la configuración del mismo. Esto le permitirá mantener la alerta en ejecución y quizás probar algunas de las reglas de alerta y, a continuación, volver a habilitar el runbook.

## Resumen

Cuando configura una alerta en una máquina virtual de Azure, tiene la capacidad de configurar fácilmente un runbook de automatización para que realice la acción correctora automáticamente cuando se desencadene la alerta. En esta versión, puede elegir entre los runbooks para reiniciar, detener o eliminar una máquina virtual según el escenario de la alerta. Esto es solo el principio de la habilitación de escenarios en los que puede controlar las acciones (notificación, solución de problemas, corrección) que se realizarán automáticamente cuando se desencadene una alerta.

## Pasos siguientes

- Para empezar a trabajar con runbooks gráficos, consulte [Mi primer runbook gráfico](automation-first-runbook-graphical.md)
- Para empezar a trabajar con runbooks de flujo de trabajo de PowerShell, consulte [Mi primer runbook de flujo de trabajo de PowerShell](automation-first-runbook-textual.md)
- Para más información sobre los tipos de runbook, sus ventajas y sus limitaciones, consulte [Tipos de runbooks de Automatización de Azure](automation-runbook-types.md)

<!---HONumber=AcomDC_0420_2016-->