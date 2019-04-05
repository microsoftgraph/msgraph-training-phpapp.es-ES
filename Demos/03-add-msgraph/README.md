# <a name="how-to-run-the-completed-project"></a><span data-ttu-id="c56cd-101">Cómo ejecutar el proyecto completado</span><span class="sxs-lookup"><span data-stu-id="c56cd-101">How to run the completed project</span></span>

## <a name="prerequisites"></a><span data-ttu-id="c56cd-102">Requisitos previos</span><span class="sxs-lookup"><span data-stu-id="c56cd-102">Prerequisites</span></span>

<span data-ttu-id="c56cd-103">Para ejecutar el proyecto completado en esta carpeta, necesita lo siguiente:</span><span class="sxs-lookup"><span data-stu-id="c56cd-103">To run the completed project in this folder, you need the following:</span></span>

- <span data-ttu-id="c56cd-104">[Php](http://php.net/downloads.php) instalado en el equipo de desarrollo.</span><span class="sxs-lookup"><span data-stu-id="c56cd-104">[PHP](http://php.net/downloads.php) installed on your development machine.</span></span> <span data-ttu-id="c56cd-105">Si no tiene PHP, visite el vínculo anterior para las opciones de descarga.</span><span class="sxs-lookup"><span data-stu-id="c56cd-105">If you do not have PHP, visit the previous link for download options.</span></span> <span data-ttu-id="c56cd-106">(**Nota:** este tutorial se ha escrito con la versión 7,2 de php.</span><span class="sxs-lookup"><span data-stu-id="c56cd-106">(**Note:** This tutorial was written with PHP version 7.2.</span></span> <span data-ttu-id="c56cd-107">Los pasos de esta guía pueden funcionar con otras versiones, pero no se han probado.</span><span class="sxs-lookup"><span data-stu-id="c56cd-107">The steps in this guide may work with other versions, but that has not been tested.)</span></span>
- <span data-ttu-id="c56cd-108">[](https://getcomposer.org/) Composer instalado en el equipo de desarrollo.</span><span class="sxs-lookup"><span data-stu-id="c56cd-108">[Composer](https://getcomposer.org/) installed on your development machine.</span></span>
- <span data-ttu-id="c56cd-109">[Laravel](https://laravel.com/) instalado en el equipo de desarrollo.</span><span class="sxs-lookup"><span data-stu-id="c56cd-109">[Laravel](https://laravel.com/) installed on your development machine.</span></span>
- <span data-ttu-id="c56cd-110">Una cuenta de Microsoft personal con un buzón de correo en Outlook.com o una cuenta profesional o educativa de Microsoft.</span><span class="sxs-lookup"><span data-stu-id="c56cd-110">Either a personal Microsoft account with a mailbox on Outlook.com, or a Microsoft work or school account.</span></span>

<span data-ttu-id="c56cd-111">Si no tiene una cuenta de Microsoft, hay un par de opciones para obtener una cuenta gratuita:</span><span class="sxs-lookup"><span data-stu-id="c56cd-111">If you don't have a Microsoft account, there are a couple of options to get a free account:</span></span>

- <span data-ttu-id="c56cd-112">Puede [registrarse para obtener una nueva cuenta Microsoft personal](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span><span class="sxs-lookup"><span data-stu-id="c56cd-112">You can [sign up for a new personal Microsoft account](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span></span>
- <span data-ttu-id="c56cd-113">Puede [registrarse para el programa de desarrolladores de office 365](https://developer.microsoft.com/office/dev-program) para obtener una suscripción gratuita a Office 365.</span><span class="sxs-lookup"><span data-stu-id="c56cd-113">You can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

## <a name="register-a-web-application-with-the-azure-active-directory-admin-center"></a><span data-ttu-id="c56cd-114">Registro de una aplicación web con el centro de administración de Azure Active Directory</span><span class="sxs-lookup"><span data-stu-id="c56cd-114">Register a web application with the Azure Active Directory admin center</span></span>

1. <span data-ttu-id="c56cd-115">Abra un explorador y vaya al [centro de administración de Azure Active Directory](https://aad.portal.azure.com).</span><span class="sxs-lookup"><span data-stu-id="c56cd-115">Open a browser and navigate to the [Azure Active Directory admin center](https://aad.portal.azure.com).</span></span> <span data-ttu-id="c56cd-116">Inicie sesión con una **cuenta personal** (también conocido como Microsoft Account) o una **cuenta profesional o educativa**.</span><span class="sxs-lookup"><span data-stu-id="c56cd-116">Login using a **personal account** (aka: Microsoft Account) or **Work or School Account**.</span></span>

1. <span data-ttu-id="c56cd-117">Seleccione **Azure Active Directory** en el panel de navegación de la izquierda y, después, seleccione **registros de aplicaciones (vista previa)** en **administrar**.</span><span class="sxs-lookup"><span data-stu-id="c56cd-117">Select **Azure Active Directory** in the left-hand navigation, then select **App registrations (Preview)** under **Manage**.</span></span>

    ![<span data-ttu-id="c56cd-118">Una captura de pantalla de los registros de la aplicación</span><span class="sxs-lookup"><span data-stu-id="c56cd-118">A screenshot of the App registrations</span></span> ](/tutorial/images/aad-portal-app-registrations.png)

1. <span data-ttu-id="c56cd-119">Seleccione **registro nuevo**.</span><span class="sxs-lookup"><span data-stu-id="c56cd-119">Select **New registration**.</span></span> <span data-ttu-id="c56cd-120">En la página **registrar una aplicación** , establezca los valores de la siguiente manera.</span><span class="sxs-lookup"><span data-stu-id="c56cd-120">On the **Register an application** page, set the values as follows.</span></span>

    - <span data-ttu-id="c56cd-121">Establezca **el nombre** en `PHP Graph Tutorial`.</span><span class="sxs-lookup"><span data-stu-id="c56cd-121">Set **Name** to `PHP Graph Tutorial`.</span></span>
    - <span data-ttu-id="c56cd-122">Establezca **tipos de cuenta compatibles** en **cuentas de cualquier directorio de la organización y cuentas personales de Microsoft**.</span><span class="sxs-lookup"><span data-stu-id="c56cd-122">Set **Supported account types** to **Accounts in any organizational directory and personal Microsoft accounts**.</span></span>
    - <span data-ttu-id="c56cd-123">En **URI**de redireccionamiento, establezca la primera lista desplegable para `Web` y establezca el `http://localhost:8000/callback`valor en.</span><span class="sxs-lookup"><span data-stu-id="c56cd-123">Under **Redirect URI**, set the first drop-down to `Web` and set the value to `http://localhost:8000/callback`.</span></span>

    ![Captura de pantalla de la página registrar una aplicación](/tutorial/images/aad-register-an-app.png)

1. <span data-ttu-id="c56cd-125">Elija **registrar**.</span><span class="sxs-lookup"><span data-stu-id="c56cd-125">Choose **Register**.</span></span> <span data-ttu-id="c56cd-126">En la página **tutorial de php Graph** , copie el valor del **identificador de la aplicación (cliente)** y guárdelo, lo necesitará en el paso siguiente.</span><span class="sxs-lookup"><span data-stu-id="c56cd-126">On the **PHP Graph Tutorial** page, copy the value of the **Application (client) ID** and save it, you will need it in the next step.</span></span>

    ![Captura de pantalla del identificador de la aplicación del nuevo registro de la aplicación](/tutorial/images/aad-application-id.png)

1. <span data-ttu-id="c56cd-128">Seleccione **certificados & secretos** en **administrar**.</span><span class="sxs-lookup"><span data-stu-id="c56cd-128">Select **Certificates & secrets** under **Manage**.</span></span> <span data-ttu-id="c56cd-129">Seleccione el botón **nuevo secreto de cliente** .</span><span class="sxs-lookup"><span data-stu-id="c56cd-129">Select the **New client secret** button.</span></span> <span data-ttu-id="c56cd-130">Escriba un valor en **Descripción** y seleccione una de las opciones para **Expires** y elija **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="c56cd-130">Enter a value in **Description** and select one of the options for **Expires** and choose **Add**.</span></span>

    ![Captura de pantalla del cuadro de diálogo Agregar un secreto de cliente](/tutorial/images/aad-new-client-secret.png)

1. <span data-ttu-id="c56cd-132">Copie el valor de secreto de cliente antes de salir de esta página.</span><span class="sxs-lookup"><span data-stu-id="c56cd-132">Copy the client secret value before you leave this page.</span></span> <span data-ttu-id="c56cd-133">Lo necesitará en el paso siguiente.</span><span class="sxs-lookup"><span data-stu-id="c56cd-133">You will need it in the next step.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="c56cd-134">Este secreto de cliente no se vuelve a mostrar nunca, así que asegúrese de copiarlo ahora.</span><span class="sxs-lookup"><span data-stu-id="c56cd-134">This client secret is never shown again, so make sure you copy it now.</span></span>

    ![Captura de pantalla del secreto de cliente recién agregado](/tutorial/images/aad-copy-client-secret.png)

## <a name="configure-the-sample"></a><span data-ttu-id="c56cd-136">Configuración del ejemplo</span><span class="sxs-lookup"><span data-stu-id="c56cd-136">Configure the sample</span></span>

1. <span data-ttu-id="c56cd-137">Cambie el nombre `.env.example` del archivo `.env`a.</span><span class="sxs-lookup"><span data-stu-id="c56cd-137">Rename the `.env.example` file to `.env`.</span></span>
1. <span data-ttu-id="c56cd-138">Edite `.env` el archivo y realice los cambios siguientes.</span><span class="sxs-lookup"><span data-stu-id="c56cd-138">Edit the `.env` file and make the following changes.</span></span>
    1. <span data-ttu-id="c56cd-139">Reemplace `YOUR_APP_ID_HERE` por el **identificador de aplicación** que obtuvo desde el portal de registro de aplicaciones.</span><span class="sxs-lookup"><span data-stu-id="c56cd-139">Replace `YOUR_APP_ID_HERE` with the **Application Id** you got from the App Registration Portal.</span></span>
    1. <span data-ttu-id="c56cd-140">Reemplace `YOUR_APP_PASSWORD_HERE` por la contraseña que obtuvo en el portal de registro de aplicaciones.</span><span class="sxs-lookup"><span data-stu-id="c56cd-140">Replace `YOUR_APP_PASSWORD_HERE` with the password you got from the App Registration Portal.</span></span>
1. <span data-ttu-id="c56cd-141">En la interfaz de línea de comandos (CLI), navegue a este directorio y ejecute el siguiente comando para instalar los requisitos.</span><span class="sxs-lookup"><span data-stu-id="c56cd-141">In your command-line interface (CLI), navigate to this directory and run the following command to install requirements.</span></span>

    ```Shell
    composer install
    ```

1. <span data-ttu-id="c56cd-142">En la interfaz de línea de comandos (CLI), ejecute el siguiente comando para generar una clave de aplicación.</span><span class="sxs-lookup"><span data-stu-id="c56cd-142">In your command-line interface (CLI), run the following command to generate an application key.</span></span>

    ```Shell
    php artisan key:generate
    ```

## <a name="run-the-sample"></a><span data-ttu-id="c56cd-143">Ejecutar el ejemplo</span><span class="sxs-lookup"><span data-stu-id="c56cd-143">Run the sample</span></span>

1. <span data-ttu-id="c56cd-144">Ejecute el siguiente comando en su CLI para iniciar la aplicación.</span><span class="sxs-lookup"><span data-stu-id="c56cd-144">Run the following command in your CLI to start the application.</span></span>

    ```Shell
    php artisan serve
    ```

1. <span data-ttu-id="c56cd-145">Abra un explorador y vaya a `http://localhost:8000`.</span><span class="sxs-lookup"><span data-stu-id="c56cd-145">Open a browser and browse to `http://localhost:8000`.</span></span>