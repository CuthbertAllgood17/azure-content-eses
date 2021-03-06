### Requisitos previos

- Una cuenta de [Salesforce](https://salesforce.com).  


Antes de poder usar la cuenta de Salesforce en una aplicación lógica, debe autorizar a la aplicación lógica a conectarse a dicha cuenta. Por suerte, esto se puede hacer fácilmente dentro de la aplicación lógica en el Portal de Azure.

Aquí se explica cómo autorizar a la aplicación lógica para conectarse a su cuenta de Salesforce:
1. Para crear una conexión a Salesforce, en el diseñador de aplicaciones lógicas, seleccione **Show Microsoft managed APIs** (Mostrar API administradas por Microsoft) en la lista desplegable y, luego, escriba *Salesforce* en el cuadro de búsqueda. Seleccione el desencadenador o la acción que quiera usar: ![](./media/connectors-create-api-salesforce/salesforce-1.png)  
2. Si no ha creado ninguna conexión a Salesforce antes, se le pedirá que indique sus credenciales de Salesforce. Estas credenciales se usarán para autorizar a la aplicación lógica para conectarse y tener acceso a los datos de su cuenta de Salesforce: ![](./media/connectors-create-api-salesforce/salesforce-2.png)  
3. Indique su nombre de usuario y contraseña de Salesforce para autorizar a la aplicación lógica: ![](./media/connectors-create-api-salesforce/salesforce-3.png)  
4. Permita la conexión a Salesforce: ![](./media/connectors-create-api-salesforce/salesforce-4.png)  
5. Observe que la conexión se ha creado y que puede continuar sin problemas con el resto de pasos en la aplicación lógica: ![](./media/connectors-create-api-salesforce/salesforce-5.png)  

<!---HONumber=AcomDC_0525_2016-->