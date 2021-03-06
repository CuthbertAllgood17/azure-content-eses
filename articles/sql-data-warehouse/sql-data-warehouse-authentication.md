<properties
   pageTitle="Autenticación a Almacenamiento de datos SQL de Azure | Microsoft Azure"
   description="Autenticación de Azure Active Directory (AAD) y SQL Server a Almacenamiento de datos SQL de Azure"
   services="sql-data-warehouse"
   documentationCenter=""
   authors="byham"
   manager="barbkess"
   editor=""
   tags=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="data-management"
   ms.date="06/17/2016"
   ms.author="rickbyh;barbkess;sonyama"/>

# Autenticación a Almacenamiento de datos SQL de Azure

> [AZURE.SELECTOR]
- [Información general](sql-data-warehouse-connect-overview.md)
- [Autenticación](sql-data-warehouse-authentication.md)
- [Controladores](sql-data-warehouse-connection-strings.md)

Para conectarse a Almacenamiento de datos SQL, deberá pasar las credenciales de seguridad para realizar la autenticación. Después de establecer una conexión, también encontrará que determinados valores de conexión se configuran como parte del establecimiento de la sesión de la consulta.

Para obtener más información sobre la seguridad y cómo habilitar las conexiones a su almacenamiento de datos, consulte [Proteger una base de datos en Almacenamiento de datos SQL][].

## Autenticación de SQL
Para conectarse a Almacenamiento de datos SQL, deberá proporcionar la siguiente información:

- Nombre de servidor completo
- Especificar la autenticación de SQL
- Nombre de usuario
- Password
- Base de datos predeterminada (opcional)

Es importante tener en cuenta que los usuarios deben autenticarse mediante autenticación de SQL. No se admite la autenticación de confianza en este momento.

De forma predeterminada, su conexión se realizará a la base de datos principal y no a su base de datos de usuario. Para conectarse a la base de datos de usuario puede hacer dos cosas:

1. Especificar la base de datos predeterminada al registrar el servidor con el Explorador de objetos de SQL Server en SSDT o en la cadena de conexión de la aplicación. Por ejemplo, incluyendo el parámetro InitialCatalog para una conexión ODBC.
2. En primer lugar, resalte la base de datos de usuario antes de crear una sesión en SSDT.

> [AZURE.NOTE] Para obtener instrucciones sobre cómo conectarse a Almacenamiento de datos SQL con SSDT, vuelva al artículo [Query with Visual Studio][] \(Realización de consultas con Visual Studio).

De nuevo, es importante tener en cuenta que la instrucción de Transact-SQL **USE <your DB>** no se admite para cambiar la base de datos en una conexión


## Autenticación de Azure Active Directory (AAD)

La autenticación de Azure Active Directory es un mecanismo de conexión a Almacenamiento de datos de Microsoft Azure SQL mediante identidades de Azure Active Directory (Azure AD). Con la autenticación de Azure Active Directory puede administrar centralmente las identidades de los usuarios de la base de datos y otros servicios de Microsoft en una ubicación central. La administración de identificadores central ofrece un lugar único para administrar usuarios de Almacenamiento de datos SQL y simplifica la administración de permisos.

### Ventajas

Entre las ventajas se incluyen las siguientes:

- Ofrece una alternativa a la autenticación de SQL Server.
- Ayuda a detener la proliferación de identidades de usuario en los servidores de base de datos.
- Permite la rotación de contraseñas en un solo lugar
- Los clientes pueden administrar los permisos de la base de datos con grupos externos (AAD).
- Puede eliminar el almacenamiento de contraseñas mediante la habilitación de la autenticación integrada de Windows y otras formas de autenticación compatibles con Azure Active Directory.
- La autenticación de Azure Active Directory utiliza usuarios de base de datos independiente para autenticar las identidades en el nivel de base de datos.
- Azure Active Directory admite la autenticación basada en token para las aplicaciones que se conectan a Almacenamiento de datos SQL.

> [AZURE.IMPORTANT] Azure Active Directory es una característica de vista previa y está sujeta a los términos de la versión de vista previa incluidos en el contrato de licencia (por ejemplo, el Contrato Enterprise, el Contrato de Microsoft Azure o el contrato Microsoft Online Subscription), así como cualquier otro [Términos de Uso Complementarios para Vistas Previas de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) aplicable.

### Pasos de configuración

En los pasos de configuración se incluyen los siguientes procedimientos para configurar y usar la autenticación de Azure Active Directory.

1. Crear y rellenar un Azure Active Directory.
2. Opcional: asociar o cambiar el Active Directory que está asociado actualmente a la suscripción de Azure.
3. Crear un administrador de Azure Active Directory para Almacenamiento de datos SQL de Azure.
4. Configurar los equipos cliente.
5. Crear usuarios de base de datos independiente en la base de datos y asignados a identidades de Azure AD.
6. Conectarse a su almacenamiento de datos mediante identidades de Azure AD.

Las principales diferencias entre el uso de la autenticación de Azure Active Directory con Base de datos SQL de Azure y Almacenamiento de datos SQL de Azure es que debe utilizar SQL Server Data Tools en lugar de SQL Server Management Studio para conectarse a Almacenamiento de datos SQL. Almacenamiento de datos SQL requiere al menos la versión de abril de 2016 (14.0.60311.1) de SQL Server Data Tools para Visual Studio 2015. Actualmente los usuarios de Azure Active Directory no se muestran en el Explorador de objetos de SSDT. Como solución alternativa, vea los usuarios de [sys.database\_principals](https://msdn.microsoft.com/library/ms187328.aspx).
  
### Búsqueda de los detalles
- Siga los pasos detallados. Los pasos detallados para configurar y usar la autenticación de Azure Active Directory son casi idénticos para Base de datos SQL de Azure y Almacenamiento de datos SQL de Azure. Siga los pasos detallados del tema [Conexión a Base de datos SQL o a Almacenamiento de datos SQL mediante autenticación de Azure Active Directory](../sql-database/sql-database-aad-authentication.md).
- Cree roles de base de datos personalizados y agrégueles usuarios. A continuación, conceda permisos específicos a los roles. Para obtener más información, consulte [Introducción a los permisos de los motores de bases de datos](https://msdn.microsoft.com/library/mt667986.aspx).

## Pasos siguientes

Para empezar a realizar consultas en el almacenamiento de datos con Visual Studio y otras aplicaciones, consulte [Query with Visual Studio][] \(Realización de consultas con Visual Studio).

<!-- Article references -->
[Proteger una base de datos en Almacenamiento de datos SQL]: sql-data-warehouse-overview-manage-security.md
[Query with Visual Studio]: sql-data-warehouse-query-visual-studio.md

<!---HONumber=AcomDC_0622_2016-->