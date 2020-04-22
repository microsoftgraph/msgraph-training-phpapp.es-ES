<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="44907-101">En este ejercicio, ampliará la aplicación del ejercicio anterior para admitir la autenticación con Azure AD.</span><span class="sxs-lookup"><span data-stu-id="44907-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="44907-102">Esto es necesario para obtener el token de acceso de OAuth necesario para llamar a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="44907-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="44907-103">En este paso, integrará la biblioteca [de OAuth2-Client](https://github.com/thephpleague/oauth2-client) en la aplicación.</span><span class="sxs-lookup"><span data-stu-id="44907-103">In this step you will integrate the [oauth2-client](https://github.com/thephpleague/oauth2-client) library into the application.</span></span>

1. <span data-ttu-id="44907-104">Abra el `.env` archivo en la raíz de la aplicación PHP y agregue el siguiente código al final del archivo.</span><span class="sxs-lookup"><span data-stu-id="44907-104">Open the `.env` file in the root of your PHP application, and add the following code to the end of the file.</span></span>

    :::code language="ini" source="../demo/graph-tutorial/.env.example" id="OAuthSettingsSnippet":::

1. <span data-ttu-id="44907-105">Reemplace `YOUR_APP_ID_HERE` por el identificador de la aplicación del portal de registro de la `YOUR_APP_PASSWORD_HERE` aplicación y reemplace por la contraseña que ha generado.</span><span class="sxs-lookup"><span data-stu-id="44907-105">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_PASSWORD_HERE` with the password you generated.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="44907-106">Si usa un control de código fuente como GIT, ahora sería un buen momento para excluir el `.env` archivo del control de código fuente para evitar la pérdida inadvertida del identificador de la aplicación y la contraseña.</span><span class="sxs-lookup"><span data-stu-id="44907-106">If you're using source control such as git, now would be a good time to exclude the `.env` file from source control to avoid inadvertently leaking your app ID and password.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="44907-107">Implementar el inicio de sesión</span><span class="sxs-lookup"><span data-stu-id="44907-107">Implement sign-in</span></span>

1. <span data-ttu-id="44907-108">Cree un nuevo archivo en el directorio **./app/http/Controllers** denominado `AuthController.php` y agregue el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="44907-108">Create a new file in the **./app/Http/Controllers** directory named `AuthController.php` and add the following code.</span></span>

    ```php
    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;

    class AuthController extends Controller
    {
      public function signin()
      {
        // Initialize the OAuth client
        $oauthClient = new \League\OAuth2\Client\Provider\GenericProvider([
          'clientId'                => env('OAUTH_APP_ID'),
          'clientSecret'            => env('OAUTH_APP_PASSWORD'),
          'redirectUri'             => env('OAUTH_REDIRECT_URI'),
          'urlAuthorize'            => env('OAUTH_AUTHORITY').env('OAUTH_AUTHORIZE_ENDPOINT'),
          'urlAccessToken'          => env('OAUTH_AUTHORITY').env('OAUTH_TOKEN_ENDPOINT'),
          'urlResourceOwnerDetails' => '',
          'scopes'                  => env('OAUTH_SCOPES')
        ]);

        $authUrl = $oauthClient->getAuthorizationUrl();

        // Save client state so we can validate in callback
        session(['oauthState' => $oauthClient->getState()]);

        // Redirect to AAD signin page
        return redirect()->away($authUrl);
      }

      public function callback(Request $request)
      {
        // Validate state
        $expectedState = session('oauthState');
        $request->session()->forget('oauthState');
        $providedState = $request->query('state');

        if (!isset($expectedState)) {
          // If there is no expected state in the session,
          // do nothing and redirect to the home page.
          return redirect('/');
        }

        if (!isset($providedState) || $expectedState != $providedState) {
          return redirect('/')
            ->with('error', 'Invalid auth state')
            ->with('errorDetail', 'The provided auth state did not match the expected value');
        }

        // Authorization code should be in the "code" query param
        $authCode = $request->query('code');
        if (isset($authCode)) {
          // Initialize the OAuth client
          $oauthClient = new \League\OAuth2\Client\Provider\GenericProvider([
            'clientId'                => env('OAUTH_APP_ID'),
            'clientSecret'            => env('OAUTH_APP_PASSWORD'),
            'redirectUri'             => env('OAUTH_REDIRECT_URI'),
            'urlAuthorize'            => env('OAUTH_AUTHORITY').env('OAUTH_AUTHORIZE_ENDPOINT'),
            'urlAccessToken'          => env('OAUTH_AUTHORITY').env('OAUTH_TOKEN_ENDPOINT'),
            'urlResourceOwnerDetails' => '',
            'scopes'                  => env('OAUTH_SCOPES')
          ]);

          try {
            // Make the token request
            $accessToken = $oauthClient->getAccessToken('authorization_code', [
              'code' => $authCode
            ]);

            // TEMPORARY FOR TESTING!
            return redirect('/')
              ->with('error', 'Access token received')
              ->with('errorDetail', $accessToken->getToken());
          }
          catch (League\OAuth2\Client\Provider\Exception\IdentityProviderException $e) {
            return redirect('/')
              ->with('error', 'Error requesting access token')
              ->with('errorDetail', $e->getMessage());
          }
        }

        return redirect('/')
          ->with('error', $request->query('error'))
          ->with('errorDetail', $request->query('error_description'));
      }
    }
    ```

    <span data-ttu-id="44907-109">Esto define un controlador con dos acciones: `signin` y `callback`.</span><span class="sxs-lookup"><span data-stu-id="44907-109">This defines a controller with two actions: `signin` and `callback`.</span></span>

    <span data-ttu-id="44907-110">La `signin` acción genera la dirección URL de inicio de sesión de Azure `state` ad, guarda el valor generado por el cliente de OAuth y, a continuación, redirige el explorador a la página de inicio de sesión de Azure ad.</span><span class="sxs-lookup"><span data-stu-id="44907-110">The `signin` action generates the Azure AD signin URL, saves the `state` value generated by the OAuth client, then redirects the browser to the Azure AD signin page.</span></span>

    <span data-ttu-id="44907-111">La `callback` acción es donde Azure redirige después de que se complete el inicio de sesión.</span><span class="sxs-lookup"><span data-stu-id="44907-111">The `callback` action is where Azure redirects after the signin is complete.</span></span> <span data-ttu-id="44907-112">Esa acción garantiza que el `state` valor coincida con el valor guardado y, a continuación, los usuarios el código de autorización enviado por Azure para solicitar un token de acceso.</span><span class="sxs-lookup"><span data-stu-id="44907-112">That action makes sure the `state` value matches the saved value, then users the authorization code sent by Azure to request an access token.</span></span> <span data-ttu-id="44907-113">A continuación, se redirige de nuevo a la Página principal con el token de acceso en el valor de error temporal.</span><span class="sxs-lookup"><span data-stu-id="44907-113">It then redirects back to the home page with the access token in the temporary error value.</span></span> <span data-ttu-id="44907-114">Usará esto para comprobar que el inicio de sesión está funcionando antes de continuar.</span><span class="sxs-lookup"><span data-stu-id="44907-114">You'll use this to verify that sign-in is working before moving on.</span></span>

1. <span data-ttu-id="44907-115">Agregue las rutas a **./Routes/Web.php**.</span><span class="sxs-lookup"><span data-stu-id="44907-115">Add the routes to **./routes/web.php**.</span></span>

    ```php
    Route::get('/signin', 'AuthController@signin');
    Route::get('/callback', 'AuthController@callback');
    ```

1. <span data-ttu-id="44907-116">Inicie el servidor y vaya a `https://localhost:8000`.</span><span class="sxs-lookup"><span data-stu-id="44907-116">Start the server and browse to `https://localhost:8000`.</span></span> <span data-ttu-id="44907-117">Haga clic en el botón de inicio de sesión y se le `https://login.microsoftonline.com`redirigirá a.</span><span class="sxs-lookup"><span data-stu-id="44907-117">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="44907-118">Inicie sesión con su cuenta de Microsoft y dé su consentimiento a los permisos solicitados.</span><span class="sxs-lookup"><span data-stu-id="44907-118">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="44907-119">El explorador redirige a la aplicación, que muestra el token.</span><span class="sxs-lookup"><span data-stu-id="44907-119">The browser redirects to the app, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="44907-120">Obtener detalles del usuario</span><span class="sxs-lookup"><span data-stu-id="44907-120">Get user details</span></span>

<span data-ttu-id="44907-121">En esta sección, actualizará el `callback` método para obtener el perfil del usuario de Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="44907-121">In this section you'll update the `callback` method to get the user's profile from Microsoft Graph.</span></span>

1. <span data-ttu-id="44907-122">Agregue las siguientes `use` instrucciones a la parte superior de **/App/http/Controllers/AuthController.php**, debajo `namespace App\Http\Controllers;` de la línea.</span><span class="sxs-lookup"><span data-stu-id="44907-122">Add the following `use` statements to the top of **/app/Http/Controllers/AuthController.php**, beneath the `namespace App\Http\Controllers;` line.</span></span>

    ```php
    use Microsoft\Graph\Graph;
    use Microsoft\Graph\Model;
    ```

1. <span data-ttu-id="44907-123">Reemplace el `try` bloque en el `callback` método por el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="44907-123">Replace the `try` block in the `callback` method with the following code.</span></span>

    ```php
    try {
      // Make the token request
      $accessToken = $oauthClient->getAccessToken('authorization_code', [
        'code' => $authCode
      ]);

      $graph = new Graph();
      $graph->setAccessToken($accessToken->getToken());

      $user = $graph->createRequest('GET', '/me')
        ->setReturnType(Model\User::class)
        ->execute();

      // TEMPORARY FOR TESTING!
      return redirect('/')
        ->with('error', 'Access token received')
        ->with('errorDetail', 'User:'.$user->getDisplayName().', Token:'.$accessToken->getToken());
    }
    ```

<span data-ttu-id="44907-124">El nuevo código crea un `Graph` objeto, asigna el token de acceso y, a continuación, lo usa para solicitar el perfil del usuario.</span><span class="sxs-lookup"><span data-stu-id="44907-124">The new code creates a `Graph` object, assigns the access token, then uses it to request the user's profile.</span></span> <span data-ttu-id="44907-125">Agrega el nombre para mostrar del usuario a la salida temporal para las pruebas.</span><span class="sxs-lookup"><span data-stu-id="44907-125">It adds the user's display name to the temporary output for testing.</span></span>

## <a name="storing-the-tokens"></a><span data-ttu-id="44907-126">Almacenamiento de tokens</span><span class="sxs-lookup"><span data-stu-id="44907-126">Storing the tokens</span></span>

<span data-ttu-id="44907-127">Ahora que puede obtener tokens, es el momento de implementar una forma de almacenarlos en la aplicación.</span><span class="sxs-lookup"><span data-stu-id="44907-127">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="44907-128">Dado que se trata de una aplicación de ejemplo, por razones de simplicidad, se almacenará en la sesión.</span><span class="sxs-lookup"><span data-stu-id="44907-128">Since this is a sample app, for simplicity's sake, you'll store them in the session.</span></span> <span data-ttu-id="44907-129">Una aplicación real usaría una solución de almacenamiento seguro más confiable, como una base de datos.</span><span class="sxs-lookup"><span data-stu-id="44907-129">A real-world app would use a more reliable secure storage solution, like a database.</span></span>

1. <span data-ttu-id="44907-130">Cree un nuevo directorio en el directorio **./app** denominado `TokenStore`, cree un nuevo archivo en ese directorio denominado `TokenCache.php`y agregue el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="44907-130">Create a new directory in the **./app** directory named `TokenStore`, then create a new file in that directory named `TokenCache.php`, and add the following code.</span></span>

    ```php
    <?php

    namespace App\TokenStore;

    class TokenCache {
      public function storeTokens($accessToken, $user) {
        session([
          'accessToken' => $accessToken->getToken(),
          'refreshToken' => $accessToken->getRefreshToken(),
          'tokenExpires' => $accessToken->getExpires(),
          'userName' => $user->getDisplayName(),
          'userEmail' => null !== $user->getMail() ? $user->getMail() : $user->getUserPrincipalName()
        ]);
      }

      public function clearTokens() {
        session()->forget('accessToken');
        session()->forget('refreshToken');
        session()->forget('tokenExpires');
        session()->forget('userName');
        session()->forget('userEmail');
      }

      public function getAccessToken() {
        // Check if tokens exist
        if (empty(session('accessToken')) ||
            empty(session('refreshToken')) ||
            empty(session('tokenExpires'))) {
          return '';
        }

        return session('accessToken');
      }
    }
    ```

1. <span data-ttu-id="44907-131">Agregue la siguiente `use` instrucción a la parte superior de **./app/http/Controllers/AuthController.php**, debajo `namespace App\Http\Controllers;` de la línea.</span><span class="sxs-lookup"><span data-stu-id="44907-131">Add the following `use` statement to the top of **./app/Http/Controllers/AuthController.php**, beneath the `namespace App\Http\Controllers;` line.</span></span>

    ```php
    use App\TokenStore\TokenCache;
    ```

1. <span data-ttu-id="44907-132">Reemplace el `try` bloque de la función `callback` existente por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="44907-132">Replace the `try` block in the existing `callback` function with the following.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/AuthController.php" id="StoreTokensSnippet":::

## <a name="implement-sign-out"></a><span data-ttu-id="44907-133">Implementación de cierre de sesión</span><span class="sxs-lookup"><span data-stu-id="44907-133">Implement sign-out</span></span>

<span data-ttu-id="44907-134">Antes de probar esta nueva característica, agregue una forma de cerrar sesión.</span><span class="sxs-lookup"><span data-stu-id="44907-134">Before you test this new feature, add a way to sign out.</span></span>

1. <span data-ttu-id="44907-135">Agregue la siguiente acción a la `AuthController` clase.</span><span class="sxs-lookup"><span data-stu-id="44907-135">Add the following action to the `AuthController` class.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/AuthController.php" id="SignOutSnippet":::

1. <span data-ttu-id="44907-136">Agregue esta acción a **./Routes/Web.php**.</span><span class="sxs-lookup"><span data-stu-id="44907-136">Add this action to **./routes/web.php**.</span></span>

    ```php
    Route::get('/signout', 'AuthController@signout');
    ```

1. <span data-ttu-id="44907-137">Reinicie el servidor y pase por el proceso de inicio de sesión.</span><span class="sxs-lookup"><span data-stu-id="44907-137">Restart the server and go through the sign-in process.</span></span> <span data-ttu-id="44907-138">Deberás volver a la Página principal, pero la interfaz de usuario debe cambiar para indicar que has iniciado sesión.</span><span class="sxs-lookup"><span data-stu-id="44907-138">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

    ![Una captura de pantalla de la Página principal después de iniciar sesión](./images/add-aad-auth-01.png)

1. <span data-ttu-id="44907-140">Haga clic en el avatar de usuario en la esquina superior derecha para acceder al vínculo **Cerrar sesión** .</span><span class="sxs-lookup"><span data-stu-id="44907-140">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="44907-141">Al hacer clic en **cerrar** sesión se restablece la sesión y se vuelve a la Página principal.</span><span class="sxs-lookup"><span data-stu-id="44907-141">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

    ![Captura de pantalla del menú desplegable con el vínculo cerrar sesión](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="44907-143">Actualizar tokens</span><span class="sxs-lookup"><span data-stu-id="44907-143">Refreshing tokens</span></span>

<span data-ttu-id="44907-144">En este punto, la aplicación tiene un token de acceso, que se envía `Authorization` en el encabezado de las llamadas a la API.</span><span class="sxs-lookup"><span data-stu-id="44907-144">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="44907-145">Este es el token que permite que la aplicación tenga acceso a Microsoft Graph en nombre del usuario.</span><span class="sxs-lookup"><span data-stu-id="44907-145">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="44907-146">Sin embargo, este token es de corta duración.</span><span class="sxs-lookup"><span data-stu-id="44907-146">However, this token is short-lived.</span></span> <span data-ttu-id="44907-147">El token expira una hora después de su emisión.</span><span class="sxs-lookup"><span data-stu-id="44907-147">The token expires an hour after it is issued.</span></span> <span data-ttu-id="44907-148">Aquí es donde el token de actualización se vuelve útil.</span><span class="sxs-lookup"><span data-stu-id="44907-148">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="44907-149">El token de actualización permite que la aplicación solicite un nuevo token de acceso sin que el usuario tenga que iniciar sesión de nuevo.</span><span class="sxs-lookup"><span data-stu-id="44907-149">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span> <span data-ttu-id="44907-150">Actualice el código de administración de tokens para implementar la actualización de tokens.</span><span class="sxs-lookup"><span data-stu-id="44907-150">Update the token management code to implement token refresh.</span></span>

1. <span data-ttu-id="44907-151">Abra **./app/TokenStore/TokenCache.php** y agregue la siguiente función a la `TokenCache` clase.</span><span class="sxs-lookup"><span data-stu-id="44907-151">Open **./app/TokenStore/TokenCache.php** and add the following function to the `TokenCache` class.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/TokenStore/TokenCache.php" id="UpdateTokensSnippet":::

1. <span data-ttu-id="44907-152">Reemplace la función `getAccessToken` existente por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="44907-152">Replace the existing `getAccessToken` function with the following.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/TokenStore/TokenCache.php" id="GetAccessTokenSnippet":::

<span data-ttu-id="44907-153">Este método comprueba primero si el token de acceso ha expirado o está próximo a expirar.</span><span class="sxs-lookup"><span data-stu-id="44907-153">This method first checks if the access token is expired or close to expiring.</span></span> <span data-ttu-id="44907-154">Si es así, usa el token de actualización para obtener nuevos tokens y, a continuación, actualiza la memoria caché y devuelve el nuevo token de acceso.</span><span class="sxs-lookup"><span data-stu-id="44907-154">If it is, then it uses the refresh token to get new tokens, then updates the cache and returns the new access token.</span></span>
