<properties
   pageTitle="Protección de un clúster de Service Fabric | Microsoft Azure"
   description="Describe los escenarios de seguridad de un clúster de Service Fabric y las diferentes tecnologías que se utilizan para implementarlos."
   services="service-fabric"
   documentationCenter=".net"
   authors="ChackDan"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="06/01/2016"
   ms.author="chackdan"/>

# Escenarios de seguridad de los clústeres de Service Fabric

Un clúster de Service Fabric es un recurso que usted posee. Para evitar el acceso no autorizado al recurso, debe protegerlo, especialmente cuando se están ejecutando en él cargas de trabajo de producción. En este artículo se proporciona información general de los escenarios de seguridad de los clústeres que se ejecutan en Azure y Windows Server, así como de las diversas tecnologías que se utilizan para implementar estos escenarios. Estos son los escenarios de seguridad de clúster:

- Seguridad de nodo a nodo
- Seguridad de cliente a nodo
- Control de acceso basado en roles (RBAC)

## Seguridad de nodo a nodo
Protege la comunicación entre las máquinas virtuales y los equipos del clúster. De esta forma se garantiza que solo los equipos que están autorizados a unirse al clúster pueden participar en el hospedaje de aplicaciones y servicios en el clúster.

![Diagrama de comunicación de nodo a nodo][Node-to-Node]

Los clústeres que se ejecutan en Azure o los independientes que se ejecutan en Windows pueden utilizar una [seguridad basada en certificados](https://msdn.microsoft.com/library/ff649801.aspx) o [Seguridad de Windows](https://msdn.microsoft.com/library/ff649396.aspx). La seguridad basada en certificados se configura al crear el clúster (a través del Portal de Azure o las plantillas ARM) especificando un certificado principal y uno secundario opcional. Los certificados principales y secundarios que especifique deben ser diferentes a los certificados de cliente de solo lectura y los de cliente de administración que determine para la [seguridad de cliente a nodo](#client-to-node-security). Consulte [Protección de clústeres de Service Fabric mediante certificados](service-fabric-secure-azure-cluster-with-certs.md) o [Configuración de un clúster de Service Fabric con una plantilla del Administrador de recursos de Azure](service-fabric-cluster-creation-via-arm.md) para obtener información sobre cómo configurar la seguridad basada en certificados en un clúster que se ejecuta en Azure.

## Seguridad de cliente a nodo
Autentica los clientes y protege la comunicación entre un cliente y los nodos individuales del clúster. Este tipo de seguridad autentica y protege las comunicaciones de cliente, lo que garantiza que solo los usuarios autorizados pueden acceder al clúster y a las aplicaciones implementadas en él. Los clientes se identifican exclusivamente mediante sus credenciales de seguridad de Windows o las credenciales de seguridad del certificado.

![Diagrama de comunicación de cliente a nodo][Client-to-Node]

Los clústeres que se ejecutan en Azure o los independientes que se ejecutan en Windows pueden utilizar una [seguridad basada en certificados](https://msdn.microsoft.com/library/ff649801.aspx) o [Seguridad de Windows](https://msdn.microsoft.com/library/ff649396.aspx). La seguridad basada en certificados de cliente a nodo se configura al crear el clúster (a través del Portal de Azure o las plantillas ARM) especificando un certificado de cliente de administración y uno de cliente de solo lectura. Los certificados de cliente de administración y de cliente de solo lectura que especifique deben ser diferentes a los certificados principales y secundarios que determine para la [seguridad de nodo a nodo](#node-to-node-security). Los clientes que se conectan al clúster mediante el certificado de administración o el principal tienen acceso completo a las funcionalidades de administración. Los clientes que se conectan al clúster mediante el certificado de cliente de solo lectura tienen acceso de este tipo a las funcionalidades de administración. Consulte [Protección de clústeres de Service Fabric mediante certificados](service-fabric-secure-azure-cluster-with-certs.md) o [Configuración de un clúster de Service Fabric con una plantilla del Administrador de recursos de Azure](service-fabric-cluster-creation-via-arm.md) para obtener información sobre cómo configurar la seguridad basada en certificados en un clúster que se ejecuta en Azure.

Service Fabric usa certificados de servidor X.509 que se especifican como parte de las configuraciones del tipo de nodo cuando se crea un clúster. Al final de este artículo, se proporciona una descripción rápida de qué son estos certificados y cómo se pueden adquirir o crear.

Los clústeres que se ejecutan en Azure también pueden proteger el acceso a los puntos de conexión de administración con Azure Active Directory (AAD). Consulte [VISTA PREVIA: Crear un clúster de Service Fabric con Azure Active Directory para la autenticación de cliente](service-fabric-cluster-security-client-auth-with-aad.md) para obtener información sobre cómo crear los artefactos de AAD necesarios, cómo rellenarlos durante la creación de los clústeres y cómo conectarse a estos clústeres posteriormente.

## Control de acceso basado en roles (RBAC)
El control de acceso permite al administrador de clústeres limitar el acceso a determinadas operaciones de clúster para distintos grupos de usuarios, lo que aumenta la seguridad del clúster. Se admiten dos tipos de control de acceso diferentes para los clientes que se conectan a un clúster: administrador y usuario. Los administradores tienen acceso total a las capacidades de administración (incluidas las capacidades de lectura y escritura). Los usuarios, de forma predeterminada, tienen acceso de solo lectura a las capacidades de administración (por ejemplo, capacidad de consulta) y a la capacidad para resolver las aplicaciones y los servicios. Especifique los roles de cliente de usuario y administrador cuando cree el clúster con certificados independientes para cada uno. Para obtener más información sobre la configuración de control de acceso predeterminada y cómo cambiarla, consulte [Control de acceso basado en roles para clientes de Service Fabric](service-fabric-cluster-security-roles.md).


## Certificados X.509 y Service Fabric
Los certificados digitales X509 se usan habitualmente para autenticar a clientes y servidores y para cifrar y firmar mensajes digitalmente. Para obtener más información sobre estos certificados, vaya a la sección [Trabajar con certificados](http://msdn.microsoft.com/library/ms731899.aspx).

Algunos puntos importantes que hay que tener en cuenta:

- Los certificados usados en clústeres que ejecutan cargas de trabajo de producción deberán crearse mediante un servicio de certificados de Windows Server correctamente configurado u obtenerse mediante una [entidad de certificación (CA)](https://en.wikipedia.org/wiki/Certificate_authority) autorizada.
- No use nunca certificados temporales o de pruebas en producción creados con herramientas como MakeCert.exe.
- Puede usar un certificado autofirmado, pero solo para los clústeres de prueba y no para los que se encuentran en fase de producción.

### Certificados de servidor X.509

Los certificados de servidor tienen la tarea principal de autenticar un servidor (nodo) en los clientes o de autenticar un servidor (nodo) en un servidor (nodo). Una de las comprobaciones iniciales cuando un nodo o un cliente autentica un nodo consiste en comprobar el valor del nombre común en el campo Sujeto. Este nombre común o uno de los nombres alternativos del sujeto del certificado debe estar presente en la lista de nombres comunes permitidos.

En el siguiente artículo se describe cómo generar certificados con nombres alternativos de sujeto (SAN): [ Cómo agregar un nombre alternativo del sujeto a un certificado LDAP seguro](http://support.microsoft.com/kb/931351).

El campo de sujeto puede contener varios valores, cada uno con un prefijo de inicialización para indicar el tipo de valor. Normalmente, la inicialización es "CN" para nombre común, por ejemplo, "CN = www.contoso.com". El campo Sujeto también puede estar en blanco. Si el campo opcional Nombre alternativo de sujeto está relleno, debe contener tanto el nombre común del certificado como una entrada por nombre alternativo de sujeto. Estos se especifican como valores de nombre DNS.

El valor del campo Propósitos planteados del certificado debe incluir un valor apropiado, como "Autenticación de servidor" o "Autenticación de cliente".

### Certificados de cliente X.509

Los certificados de cliente normalmente no los emite una entidad de certificación de terceros. En su lugar, el almacén Personal de la ubicación del usuario actual contiene normalmente los certificados de cliente colocados ahí por una entidad de certificación raíz, con un propósito planteado de "Autenticación de cliente". El cliente puede usar este tipo de certificado cuando se requiere autenticación mutua.

>[AZURE.NOTE] Todas las operaciones de administración en el clúster de Service Fabric requieren certificados de servidor. No se pueden usar certificados de cliente para la administración.

<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->


## Pasos siguientes
Cuando el clúster esté configurado, obtenga sobre las actualizaciones del clúster:

- [Proceso de actualización del clúster de Service Fabric y expectativas](service-fabric-cluster-upgrade.md)

Más información sobre la seguridad de las aplicaciones:

- [Seguridad de las aplicaciones y RunAs](service-fabric-application-runas-security.md)

- [Comunicaciones seguras del servicio](service-fabric-reliable-services-secure-communication.md)

<!--Image references-->
[Node-to-Node]: ./media/service-fabric-cluster-security/node-to-node.png
[Client-to-Node]: ./media/service-fabric-cluster-security/client-to-node.png

<!---HONumber=AcomDC_0608_2016-->