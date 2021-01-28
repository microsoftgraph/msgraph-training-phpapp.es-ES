<!-- markdownlint-disable MD002 MD041 -->

Comience por crear un nuevo proyecto de Laravel.

1. Abra la interfaz de línea de comandos (CLI), vaya a un directorio donde tenga derechos para crear archivos y ejecute el siguiente comando para crear una nueva aplicación PHP.

    ```Shell
    laravel new graph-tutorial
    ```

1. Vaya al directorio **graph-tutorial** y escriba el siguiente comando para iniciar un servidor web local.

    ```Shell
    php artisan serve
    ```

1. Abra el explorador y vaya a `http://localhost:8000`. Si todo funciona, verá una página Laravel predeterminada. Si no ve esa página, compruebe los documentos [de Laravel](https://laravel.com/docs/8.x).

## <a name="install-packages"></a>Instalar paquetes

Antes de seguir adelante, instala algunos paquetes adicionales que usarás más adelante:

- [oauth2-client para controlar](https://github.com/thephpleague/oauth2-client) los flujos de tokens de OAuth y de inicio de sesión.
- [microsoft-graph para](https://github.com/microsoftgraph/msgraph-sdk-php) realizar llamadas a Microsoft Graph.

1. Ejecute el siguiente comando en la CLI.

    ```Shell
    composer require league/oauth2-client microsoft/microsoft-graph
    ```

## <a name="design-the-app"></a>Diseñar la aplicación

1. Cree un nuevo archivo en el directorio **./resources/views** con nombre `layout.blade.php` y agregue el siguiente código.

    :::code language="php" source="../demo/graph-tutorial/resources/views/layout.blade.php" id="LayoutSnippet":::

    Este código agrega [Bootstrap para](http://getbootstrap.com/) estilos sencillos y [Font Awesome](https://fontawesome.com/) para algunos iconos simples. También define un diseño global con una barra de navegación.

1. Cree un nuevo directorio en el directorio denominado y, a continuación, `./public` cree un nuevo archivo en el directorio denominado `css` `./public/css` `app.css` . Agregue el siguiente código.

    :::code language="css" source="../demo/graph-tutorial/public/css/app.css":::

1. Abra el `./resources/views/welcome.blade.php` archivo y reemplace su contenido por lo siguiente.

    :::code language="php" source="../demo/graph-tutorial/resources/views/welcome.blade.php" id="WelcomeSnippet":::

1. Actualice la clase base `Controller` **en ./app/Http/Controllers/Controller.php** agregando la siguiente función a la clase.

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/Controller.php" id="LoadViewDataSnippet":::

1. Cree un nuevo archivo en el `./app/Http/Controllers` directorio denominado y agregue el siguiente `HomeController.php` código.

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/HomeController.php":::

1. Actualice la ruta para `./routes/web.php` usar el nuevo controlador. Reemplace todo el contenido de este archivo por lo siguiente.

    ```php
    <?php

    use Illuminate\Support\Facades\Route;

    Route::get('/', 'HomeController@welcome');
    ```

1. Abra **./app/Providers/RouteServiceProvider.php** y descomprima la `$namespace` declaración.

    ```php
    /**
     * This namespace is applied to your controller routes.
     *
     * In addition, it is set as the URL generator's root namespace.
     *
     * @var string
     */
    protected $namespace = 'App\Http\Controllers';
    ```

1. Guarde todos los cambios y reinicie el servidor. Ahora, la aplicación debería tener un aspecto muy diferente.

    ![Captura de pantalla de la página principal rediseñada](./images/create-app-01.png)
