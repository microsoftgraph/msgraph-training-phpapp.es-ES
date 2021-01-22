# <a name="how-to-run-the-completed-project"></a>Cómo ejecutar el proyecto completado

## <a name="prerequisites"></a>Requisitos previos

Para ejecutar el proyecto completado en esta carpeta, necesita lo siguiente:

- [PHP](http://php.net/downloads.php) instalado en el equipo de desarrollo. Si no tiene PHP, visite el vínculo anterior para ver las opciones de descarga. (**Nota: Este** tutorial se escribió con PHP versión 7.4.4. Los pasos de esta guía pueden funcionar con otras versiones, pero no se han probado).
- [Composer](https://getcomposer.org/) instalado en el equipo de desarrollo.
- [Laravel](https://laravel.com/) instalado en el equipo de desarrollo.
- Ya sea una cuenta personal de Microsoft con un buzón en Outlook.com o una cuenta de Microsoft para trabajo o escuela.

Si no tienes una cuenta de Microsoft, hay un par de opciones para obtener una cuenta gratuita:

- Puede registrarse [para obtener una nueva cuenta personal de Microsoft.](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)
- Puede registrarse en el Programa de desarrolladores de [Office 365](https://developer.microsoft.com/office/dev-program) para obtener una suscripción gratuita a Office 365.

## <a name="register-a-web-application-with-the-azure-active-directory-admin-center"></a>Registrar una aplicación web con el Centro de administración de Azure Active Directory

1. Abra un explorador y vaya al [centro de administración de Azure Active Directory](https://aad.portal.azure.com). Inicie sesión con una **cuenta personal** (también conocida como: cuenta Microsoft) o una **cuenta profesional o educativa**.

1. Seleccione **Azure Active Directory** en el panel de navegación izquierdo y, a continuación, seleccione **Registros de aplicaciones** en **Administrar**.

    ![Captura de pantalla de los registros de aplicaciones ](/tutorial/images/aad-portal-app-registrations.png)

1. Seleccione **Nuevo registro**. En la página **Registrar una aplicación**, establezca los valores siguientes.

    - Establezca **Nombre** como `PHP Graph Tutorial`.
    - Establezca **Tipos de cuenta admitidos** en **Cuentas en cualquier directorio de organización y cuentas personales de Microsoft**.
    - En **URI de redirección**, establezca la primera lista desplegable en `Web` y establezca el valor `http://localhost:8000/callback`.

    ![Captura de pantalla de la página Registrar una aplicación](/tutorial/images/aad-register-an-app.png)

1. Elija **Registrar**. En la **página Tutorial de Php Graph,** copie el valor del identificador de aplicación **(cliente)** y guárdelo, lo necesitará en el paso siguiente.

    ![Captura de pantalla del identificador de aplicación del nuevo registro de la aplicación](/tutorial/images/aad-application-id.png)

1. Seleccione **Certificados y secretos** en **Administrar**. Seleccione el botón **Nuevo secreto de cliente**. Escriba un valor en **Descripción** y seleccione una de las opciones de **Expira** y seleccione **Agregar**.

    ![Captura de pantalla del cuadro de diálogo Agregar un secreto de cliente](/tutorial/images/aad-new-client-secret.png)

1. Copie el valor del secreto de cliente antes de salir de esta página. Lo necesitará en el siguiente paso.

    > [!IMPORTANT]
    > El secreto de cliente no se vuelve a mostrar, así que asegúrese de copiarlo en este momento.

    ![Captura de pantalla del secreto de cliente recién agregado](/tutorial/images/aad-copy-client-secret.png)

## <a name="configure-the-sample"></a>Configuración del ejemplo

1. Cambie el nombre `example.env` del archivo a `.env` .
1. Edite `.env` el archivo y realice los siguientes cambios.
    1. Reemplace `YOUR_APP_ID_HERE` por el **id. de** aplicación que obtuvo del Portal de registro de aplicaciones.
    1. Reemplace `YOUR_APP_PASSWORD_HERE` con la contraseña que obtuvo del Portal de registro de aplicaciones.
1. En la interfaz de línea de comandos (CLI), vaya a este directorio y ejecute el siguiente comando para instalar los requisitos.

    ```Shell
    composer install
    ```

1. En la interfaz de línea de comandos (CLI), ejecute el siguiente comando para generar una clave de aplicación.

    ```Shell
    php artisan key:generate
    ```

## <a name="run-the-sample"></a>Ejecutar el ejemplo

1. Ejecute el siguiente comando en la CLI para iniciar la aplicación.

    ```Shell
    php artisan serve
    ```

1. Abra un explorador y vaya a `http://localhost:8000`.
