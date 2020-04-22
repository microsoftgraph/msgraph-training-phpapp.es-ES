<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="33a8b-101">Para empezar, cree un nuevo proyecto Laravel.</span><span class="sxs-lookup"><span data-stu-id="33a8b-101">Begin by creating a new Laravel project.</span></span>

1. <span data-ttu-id="33a8b-102">Abra la interfaz de línea de comandos (CLI), vaya a un directorio donde tenga derechos para crear archivos y ejecute el siguiente comando para crear una nueva aplicación PHP.</span><span class="sxs-lookup"><span data-stu-id="33a8b-102">Open your command-line interface (CLI), navigate to a directory where you have rights to create files, and run the following command to create a new PHP app.</span></span>

    ```Shell
    laravel new graph-tutorial
    ```

1. <span data-ttu-id="33a8b-103">Navegue hasta el directorio **gráfico-tutorial** y escriba el siguiente comando para iniciar un servidor Web local.</span><span class="sxs-lookup"><span data-stu-id="33a8b-103">Navigate to the **graph-tutorial** directory and enter the following command to start a local web server.</span></span>

    ```Shell
    php artisan serve
    ```

1. <span data-ttu-id="33a8b-104">Abra el explorador y vaya a `http://localhost:8000`.</span><span class="sxs-lookup"><span data-stu-id="33a8b-104">Open your browser and navigate to `http://localhost:8000`.</span></span> <span data-ttu-id="33a8b-105">Si todo funciona, verá una página predeterminada de Laravel.</span><span class="sxs-lookup"><span data-stu-id="33a8b-105">If everything is working, you will see a default Laravel page.</span></span> <span data-ttu-id="33a8b-106">Si no ve esa página, compruebe los [documentos de Laravel](https://laravel.com/docs/7.x).</span><span class="sxs-lookup"><span data-stu-id="33a8b-106">If you don't see that page, check the [Laravel docs](https://laravel.com/docs/7.x).</span></span>

## <a name="install-packages"></a><span data-ttu-id="33a8b-107">Instalar paquetes</span><span class="sxs-lookup"><span data-stu-id="33a8b-107">Install packages</span></span>

<span data-ttu-id="33a8b-108">Antes de continuar, instale algunos paquetes adicionales que usará más adelante:</span><span class="sxs-lookup"><span data-stu-id="33a8b-108">Before moving on, install some additional packages that you will use later:</span></span>

- <span data-ttu-id="33a8b-109">[OAuth2-Client](https://github.com/thephpleague/oauth2-client) para controlar los flujos de tokens de inicio de sesión y OAuth.</span><span class="sxs-lookup"><span data-stu-id="33a8b-109">[oauth2-client](https://github.com/thephpleague/oauth2-client) for handling sign-in and OAuth token flows.</span></span>
- <span data-ttu-id="33a8b-110">[Microsoft-Graph](https://github.com/microsoftgraph/msgraph-sdk-php) para realizar llamadas a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="33a8b-110">[microsoft-graph](https://github.com/microsoftgraph/msgraph-sdk-php) for making calls to Microsoft Graph.</span></span>

1. <span data-ttu-id="33a8b-111">Ejecute el siguiente comando en su CLI.</span><span class="sxs-lookup"><span data-stu-id="33a8b-111">Run the following command in your CLI.</span></span>

    ```Shell
    composer require league/oauth2-client microsoft/microsoft-graph
    ```

## <a name="design-the-app"></a><span data-ttu-id="33a8b-112">Diseñar la aplicación</span><span class="sxs-lookup"><span data-stu-id="33a8b-112">Design the app</span></span>

1. <span data-ttu-id="33a8b-113">Cree un nuevo archivo en el directorio **./Resources/views** denominado `layout.blade.php` y agregue el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="33a8b-113">Create a new file in the **./resources/views** directory named `layout.blade.php` and add the following code.</span></span>

    :::code language="php" source="../demo/graph-tutorial/resources/views/layout.blade.php" id="LayoutSnippet":::

    <span data-ttu-id="33a8b-114">Este código agrega un [bootstrap](http://getbootstrap.com/) para los estilos sencillos y la [fuente maravilla](https://fontawesome.com/) para algunos iconos simples.</span><span class="sxs-lookup"><span data-stu-id="33a8b-114">This code adds [Bootstrap](http://getbootstrap.com/) for simple styling, and [Font Awesome](https://fontawesome.com/) for some simple icons.</span></span> <span data-ttu-id="33a8b-115">También define un diseño global con una barra de navegación.</span><span class="sxs-lookup"><span data-stu-id="33a8b-115">It also defines a global layout with a nav bar.</span></span>

1. <span data-ttu-id="33a8b-116">Cree un nuevo directorio en el `./public` directorio denominado `css`y, a continuación, cree un nuevo `./public/css` archivo en `app.css`el directorio denominado.</span><span class="sxs-lookup"><span data-stu-id="33a8b-116">Create a new directory in the `./public` directory named `css`, then create a new file in the `./public/css` directory named `app.css`.</span></span> <span data-ttu-id="33a8b-117">Agregue el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="33a8b-117">Add the following code.</span></span>

    :::code language="css" source="../demo/graph-tutorial/public/css/app.css":::

1. <span data-ttu-id="33a8b-118">Abra el `./resources/views/welcome.blade.php` archivo y reemplace el contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="33a8b-118">Open the `./resources/views/welcome.blade.php` file and replace its contents with the following.</span></span>

    :::code language="php" source="../demo/graph-tutorial/resources/views/welcome.blade.php" id="WelcomeSnippet":::

1. <span data-ttu-id="33a8b-119">Actualice la clase `Controller` base en **./app/http/Controllers/Controller.php** agregando la siguiente función a la clase.</span><span class="sxs-lookup"><span data-stu-id="33a8b-119">Update the base `Controller` class in **./app/Http/Controllers/Controller.php** by adding the following function to the class.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/Controller.php" id="LoadViewDataSnippet":::

1. <span data-ttu-id="33a8b-120">Cree un nuevo archivo en el `./app/Http/Controllers` directorio denominado `HomeController.php` y agregue el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="33a8b-120">Create a new file in the `./app/Http/Controllers` directory named `HomeController.php` and add the following code.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/HomeController.php":::

1. <span data-ttu-id="33a8b-121">Actualice la ruta en `./routes/web.php` para usar el nuevo controlador.</span><span class="sxs-lookup"><span data-stu-id="33a8b-121">Update the route in `./routes/web.php` to use the new controller.</span></span> <span data-ttu-id="33a8b-122">Reemplace todo el contenido de este archivo por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="33a8b-122">Replace the entire contents of this file with the following.</span></span>

    ```php
    <?php

    use Illuminate\Support\Facades\Route;

    Route::get('/', 'HomeController@welcome');
    ```

1. <span data-ttu-id="33a8b-123">Guarde todos los cambios y reinicie el servidor.</span><span class="sxs-lookup"><span data-stu-id="33a8b-123">Save all of your changes and restart the server.</span></span> <span data-ttu-id="33a8b-124">Ahora, la aplicación debe tener un aspecto muy diferente.</span><span class="sxs-lookup"><span data-stu-id="33a8b-124">Now, the app should look very different.</span></span>

    ![Una captura de pantalla de la Página principal rediseñada](./images/create-app-01.png)
