<properties
	pageTitle="Tutorial: integración de Azure Active Directory con HackerOne | Microsoft Azure"
	description="Aprenda a configurar el inicio de sesión único entre Azure Active Directory y HackerOne."
	services="active-directory"
	documentationCenter=""
	authors="jeevansd"
	manager="stevenpo"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="04/06/2016"
	ms.author="jeedes"/>


# Tutorial: Integración de Azure Active Directory con HackerOne

En este tutorial, integrará HackerOne con Azure Active Directory (Azure AD).

La integración de HackerOne con Azure AD proporciona las siguientes ventajas:

- Puede controlar en Azure AD quién tiene acceso a HackerOne.
- Puede permitir que los usuarios inicien sesión automáticamente en HackerOne (inicio de sesión único) con sus cuentas de Azure AD.
- Puede administrar sus cuentas en una ubicación central: el Portal de Azure clásico.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](active-directory-appssoaccess-whatis.md).

## Requisitos previos

Para configurar la integración de Azure AD con HackerOne, necesita los siguientes elementos:

- Una suscripción de Azure
- Una suscripción habilitada para el inicio de sesión único en HackerOne


> [AZURE.NOTE] Para probar los pasos de este tutorial, no se recomienda el uso de un entorno de producción.


Para probar los pasos de este tutorial, debe seguir estas recomendaciones:

- No debe usar el entorno de producción, a menos que sea necesario.
- Si no dispone de un entorno de prueba de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/).


## Descripción del escenario
En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba. <br> La situación descrita en este tutorial consta de dos bloques de creación principales:

1. Incorporación de HackerOne desde la galería
2. Configuración y comprobación del inicio de sesión único de Azure AD


## Incorporación de HackerOne desde la galería
Para integrar HackerOne en Azure AD, deberá agregar HackerOne desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar HackerOne desde la galería, realice los pasos siguientes:**

1. En el **Portal de Azure clásico**, en el panel de navegación izquierdo, haga clic en **Active Directory**. <br><br> ![Active Directory][1]<br>

2. En la lista **Directory**, seleccione el directorio cuya integración desee habilitar.

3. Para abrir la vista de aplicaciones, en la vista de directorios, haga clic en **Aplicaciones** en el menú superior.<br><br> ![Aplicaciones][2]<br>
4. Haga clic en **Agregar** en la parte inferior de la página.<br><br> ![Aplicaciones][3]<br>
5. En el cuadro de diálogo **¿Qué desea hacer?**, haga clic en **Agregar una aplicación de la galería**.<br><br> ![Aplicaciones][4]<br>
6. En el cuadro de búsqueda, escriba **HackerOne**.<br><br> ![Creación de un usuario de prueba de Azure AD](./media/active-directory-saas-hackerone-tutorial/tutorial_hackerone_01.png)<br>
7. En el panel de resultados, seleccione **HackerOne** y luego haga clic en **Completar** para agregar la aplicación.<br><br>


##  Configuración y comprobación del inicio de sesión único de Azure AD
A continuación, configure y pruebe el inicio de sesión único de Azure AD con HackerOne basado en un usuario de prueba llamado "Britta Simon".

Para que el inicio de sesión único funcione, Azure AD debe saber cuál es el usuario homólogo de HackerOne para un usuario de Azure AD. Es decir, es necesario establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de HackerOne.<br> Esta relación de vínculo se establece asignando el valor del **nombre de usuario** en Azure AD como el valor del **nombre de usuario** en HackerOne.

Para configurar y probar el inicio de sesión único de Azure AD con HackerOne, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configuring-azure-ad-single-single-sign-on)**: para permitir a los usuarios usar esta característica.
2. **[Creación de un usuario de prueba de Azure AD](#creating-an-azure-ad-test-user)**: para probar el inicio de sesión único de Azure AD con Britta Simon.
3. **[Creación de un usuario de prueba de HackerOne](#creating-a-hackerone-test-user)**: para tener un homólogo de Britta Simon en Certify que esté vinculado a su representación en Azure AD.
4. **[Asignación del usuario de prueba de Azure AD](#assigning-the-azure-ad-test-user)**: para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Prueba del inicio de sesión único](#testing-single-sign-on)**: para comprobar si funciona la configuración.

### Configuración del inicio de sesión único de Azure AD

A continuación, habilitará el inicio de sesión único de Azure AD en el portal clásico y configurará el inicio de sesión único en la aplicación HackerOne.

Como parte de este procedimiento, se requiere crear un archivo de certificado codificado en base 64. Si no está familiarizado con este procedimiento, consulte [Conversión de un certificado binario en un archivo de texto](http://youtu.be/PlgrzUZ-Y1o).

**Para configurar el inicio de sesión único de Azure AD con HackerOne, realice los pasos siguientes:**

1. En el Portal de Azure clásico, en la página de integración de aplicaciones de **HackerOne**, haga clic en **Configurar inicio de sesión único** para abrir el cuadro de diálogo **Configurar inicio de sesión único**. <br><br> ![Configurar inicio de sesión único][6] <br>

2. En la página **¿Cómo desea que los usuarios inicien sesión en HackerOne?**, seleccione **Inicio de sesión único de Azure AD** y después haga clic en **Siguiente**.<br><br> ![Configurar inicio de sesión único](./media/active-directory-saas-hackerone-tutorial/tutorial_hackerone_03.png) <br>

3. En la página de diálogo **Configurar las opciones de la aplicación**, realice los pasos siguientes y luego haga clic en **Siguiente**: <br><br>![Configurar inicio de sesión único](./media/active-directory-saas-hackerone-tutorial/tutorial_hackerone_04.png) <br>


    a. En el cuadro de texto **URL de inicio de sesión**, escriba la dirección URL que usan los usuarios para iniciar sesión en la aplicación HackerOne con el siguiente patrón: **“https://hackerone.com/<company name>/authentication”**.

    b. Póngase en contacto con el equipo de soporte técnico de HackerOne mediante [support@hackerone.com](mailto:support@hackerone.com) para obtener la dirección URL del inquilino si no la sabe.

	c. En el cuadro de texto **Identificador**, escriba la dirección URL del inquilino.

	d. Haga clic en **Siguiente**.


4. En la página **Configurar inicio de sesión único en HackerOne**, lleve a cabo estos pasos y luego haga clic en **Siguiente**:<br><br>![Configurar inicio de sesión único](./media/active-directory-saas-hackerone-tutorial/tutorial_hackerone_05.png) <br>

    a. Haga clic en **Descargar certificado** y después guarde el archivo en el equipo.

    b. Haga clic en **Siguiente**.


1. Inicie la sesión en el inquilino de HackerOne como administrador.

1. En el menú de la parte superior, haga clic en **Configuración**.<br><br>![Configurar inicio de sesión único](./media/active-directory-saas-hackerone-tutorial/tutorial_hackerone_001.png) <br>

1. A continuación, vaya a "**Autenticación**" y haga clic en el botón "**Agregar configuración de SAML**".<br><br>![Configurar inicio de sesión único](./media/active-directory-saas-hackerone-tutorial/tutorial_hackerone_003.png) <br>


1. En el diálogo **Configuración de SAML**, realice los pasos siguientes: <br><br>![Configurar inicio de sesión único](./media/active-directory-saas-hackerone-tutorial/tutorial_hackerone_004.png) <br><br>

    a. En el cuadro de texto **Dominio de correo electrónico**, escriba un dominio registrado.

	b. En el Portal de Azure clásico, copie la **Dirección URL del servicio de inicio de sesión único** y péguela en el cuadro de texto Dirección URL de inicio de sesión único.

    c. Cree un archivo **codificado en base 64** a partir del certificado descargado.

       >[AZURE.TIP] Para obtener más información, consulte [Conversión de un certificado binario en un archivo de texto](http://youtu.be/PlgrzUZ-Y1o).
	
    d. Abra el certificado codificado en base 64 en el Bloc de notas, copie el contenido en el Portapapeles y péguelo en el cuadro de texto **Certificado X509**.

    e. Haga clic en **Guardar**


1. En el diálogo Configuración de autenticación, realice los pasos siguientes: <br><br>![Configurar inicio de sesión único](./media/active-directory-saas-hackerone-tutorial/tutorial_hackerone_005.png) <br><br>

    a. Haga clic en **Ejecutar prueba**.

    b. Si el valor del campo **Estado** es igual al de **Último estado de la prueba: creado**, póngase en contacto con el equipo de soporte técnico de HackerOne mediante [support@hackerone.com](mailto:support@hackerone.com) para solicitar una revisión de la configuración.


6. En el Portal de Azure clásico, seleccione la confirmación de la configuración de inicio de sesión único y haga clic en **Siguiente**. <br><br>![Inicio de sesión único de Azure AD][10]<br>

7. En la página **Confirmación del inicio de sesión único**, haga clic en **Completar**. <br>![Inicio de sesión único de Azure AD][11]




### Creación de un usuario de prueba de Azure AD
A continuación, debe crear un usuario de prueba llamado Britta Simon en el Portal clásico.<br> En la lista Usuarios, seleccione **Britta Simon**.<br><br>![Creación de un usuario de Azure AD][20]<br>

**Siga estos pasos para crear un usuario de prueba de SECURE DELIVER en Azure AD:**

1. En el **Portal de Azure clásico**, en el panel de navegación izquierdo, haga clic en **Active Directory**. <br><br>![Creación de un usuario de prueba de Azure AD](./media/active-directory-saas-hackerone-tutorial/create_aaduser_09.png) <br>

2. En la lista **Directory**, seleccione el directorio cuya integración desee habilitar.

3. Para mostrar la lista de usuarios, en el menú de la parte superior, haga clic en **Usuarios**.<br><br> ![Creación de un usuario de prueba de Azure AD](./media/active-directory-saas-hackerone-tutorial/create_aaduser_03.png) <br>

4. Para abrir el cuadro de diálogo **Agregar usuario**, en la barra de herramientas de la parte inferior, haga clic en **Agregar usuario**. <br><br> ![Creación de un usuario de prueba de Azure AD](./media/active-directory-saas-hackerone-tutorial/create_aaduser_04.png) <br>

5. En la página del cuadro de diálogo **Proporcione información sobre este usuario**, realice los pasos siguientes: <br><br> ![Creación de un usuario de prueba de Azure AD](./media/active-directory-saas-hackerone-tutorial/create_aaduser_05.png) <br>

    a. En Tipo de usuario, seleccione Nuevo usuario de la organización.

    b. En el cuadro de texto **Nombre de usuario**, escriba **BrittaSimon**.

    c. Haga clic en **Siguiente**.

6.  En la página del cuadro de diálogo **Perfil de usuario**, realice los siguientes pasos: <br><br>![Creación de un usuario de prueba de Azure AD](./media/active-directory-saas-hackerone-tutorial/create_aaduser_06.png) <br>

    a. En el cuadro de texto **Nombre**, escriba **Britta**.

    b. En el cuadro de texto **Apellidos**, escriba **Simon**.

    c. En el cuadro de texto **Nombre para mostrar**, escriba **Britta Simon**.

    d. En la lista **Rol**, seleccione **Usuario**.

    e. Haga clic en **Siguiente**.

7. En la página de diálogo **Obtener contraseña temporal**, haga clic en **Crear**. <br><br> ![Creación de un usuario de prueba de Azure AD](./media/active-directory-saas-hackerone-tutorial/create_aaduser_07.png) <br>

8. En la página de diálogo **Obtener contraseña temporal**, realice los pasos siguientes: <br><br>![Creación de un usuario de prueba de Azure AD](./media/active-directory-saas-hackerone-tutorial/create_aaduser_08.png) <br>

    a. Anote el valor del campo **Nueva contraseña**.

    b. Haga clic en **Completo**.


### Creación de un usuario de prueba de HackerOne

A continuación, creará un usuario llamado Simon Britta en HackerOne. HackerOne admite el aprovisionamiento Just-In-Time, que está habilitado de forma predeterminada.

No hay ningún elemento de acción para usted en esta sección. Al acceder a HackerOne, se crea un usuario nuevo si todavía no existe uno. [Configuración del inicio de sesión único de Azure AD](#configuring-azure-ad-single-single-sign-on).

> [AZURE.NOTE] Si necesita crear manualmente un usuario, debe ponerse en contacto con el equipo de soporte técnico de Certify.




### Asignación del usuario de prueba de Azure AD

A continuación, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a HackerOne. <br><br>![Asignar usuario][200] <br>

**Para asignar a Britta Simon a HackerOne, realice los pasos siguientes:**

1. En el Portal de Azure clásico, para abrir la vista de aplicaciones, en la vista del directorio, haga clic en **Aplicaciones** en el menú superior. <br><br>![Asignar usuario][201] <br>

2. En la lista de aplicaciones, seleccione **HackerOne**. <br><br>![Configurar inicio de sesión único](./media/active-directory-saas-hackerone-tutorial/tutorial_hackerone_50.png) <br>

1. En el menú de la parte superior, haga clic en **Usuarios**. <br><br>![Asignar usuario][203] <br>

1. En la lista Usuarios, seleccione **Britta Simon**.

2. En la barra de herramientas de la parte inferior, haga clic en **Asignar**. <br><br>![Asignar usuario][205]



### Prueba del inicio de sesión único

Por último, probará la configuración de inicio de sesión único de Azure AD mediante el panel de acceso.<br> Al hacer clic en el icono de HackerOne en el Panel de acceso, debería iniciar sesión automáticamente en su aplicación HackerOne.


## Recursos adicionales

* [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](active-directory-saas-tutorial-list.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](active-directory-appssoaccess-whatis.md)


<!--Image references-->

[1]: ./media/active-directory-saas-hackerone-tutorial/tutorial_general_01.png
[2]: ./media/active-directory-saas-hackerone-tutorial/tutorial_general_02.png
[3]: ./media/active-directory-saas-hackerone-tutorial/tutorial_general_03.png
[4]: ./media/active-directory-saas-hackerone-tutorial/tutorial_general_04.png

[6]: ./media/active-directory-saas-hackerone-tutorial/tutorial_general_05.png
[10]: ./media/active-directory-saas-hackerone-tutorial/tutorial_general_06.png
[11]: ./media/active-directory-saas-hackerone-tutorial/tutorial_general_07.png
[20]: ./media/active-directory-saas-hackerone-tutorial/tutorial_general_100.png

[200]: ./media/active-directory-saas-hackerone-tutorial/tutorial_general_200.png
[201]: ./media/active-directory-saas-hackerone-tutorial/tutorial_general_201.png
[203]: ./media/active-directory-saas-hackerone-tutorial/tutorial_general_203.png
[204]: ./media/active-directory-saas-hackerone-tutorial/tutorial_general_204.png
[205]: ./media/active-directory-saas-hackerone-tutorial/tutorial_general_205.png

<!---HONumber=AcomDC_0413_2016-->