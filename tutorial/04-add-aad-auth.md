<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="a2aac-101">En este ejercicio, ampliará la aplicación del ejercicio anterior para admitir la autenticación con Azure AD.</span><span class="sxs-lookup"><span data-stu-id="a2aac-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="a2aac-102">Esto es necesario para obtener el token de acceso OAuth necesario para llamar a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="a2aac-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="a2aac-103">En este paso, integrará la biblioteca [oauth2-client](https://github.com/thephpleague/oauth2-client) en la aplicación.</span><span class="sxs-lookup"><span data-stu-id="a2aac-103">In this step you will integrate the [oauth2-client](https://github.com/thephpleague/oauth2-client) library into the application.</span></span>

1. <span data-ttu-id="a2aac-104">Abra el **archivo .env** en la raíz de la aplicación PHP y agregue el siguiente código al final del archivo.</span><span class="sxs-lookup"><span data-stu-id="a2aac-104">Open the **.env** file in the root of your PHP application, and add the following code to the end of the file.</span></span>

    :::code language="ini" source="../demo/graph-tutorial/example.env" range="51-57":::

1. <span data-ttu-id="a2aac-105">Reemplace por el identificador de aplicación del Portal de registro de aplicaciones y `YOUR_APP_ID_HERE` reemplace por la contraseña que `YOUR_APP_SECRET_HERE` generó.</span><span class="sxs-lookup"><span data-stu-id="a2aac-105">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_SECRET_HERE` with the password you generated.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="a2aac-106">Si usas el control de código fuente como Git, ahora sería un buen momento para excluir el archivo del control de código fuente para evitar la pérdida involuntaria del identificador de la aplicación y la `.env` contraseña.</span><span class="sxs-lookup"><span data-stu-id="a2aac-106">If you're using source control such as git, now would be a good time to exclude the `.env` file from source control to avoid inadvertently leaking your app ID and password.</span></span>

1. <span data-ttu-id="a2aac-107">Cree un nuevo archivo en la **carpeta ./config** con nombre `azure.php` y agregue el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="a2aac-107">Create a new file in the **./config** folder named `azure.php` and add the following code.</span></span>

    :::code language="php" source="../demo/graph-tutorial/config/azure.php":::

## <a name="implement-sign-in"></a><span data-ttu-id="a2aac-108">Implementar el inicio de sesión</span><span class="sxs-lookup"><span data-stu-id="a2aac-108">Implement sign-in</span></span>

1. <span data-ttu-id="a2aac-109">Crea un nuevo archivo en el directorio **./app/Http/Controllers** denominado `AuthController.php` y agrega el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="a2aac-109">Create a new file in the **./app/Http/Controllers** directory named `AuthController.php` and add the following code.</span></span>

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
          'clientId'                => config('azure.appId'),
          'clientSecret'            => config('azure.appSecret'),
          'redirectUri'             => config('azure.redirectUri'),
          'urlAuthorize'            => config('azure.authority').config('azure.authorizeEndpoint'),
          'urlAccessToken'          => config('azure.authority').config('azure.tokenEndpoint'),
          'urlResourceOwnerDetails' => '',
          'scopes'                  => config('azure.scopes')
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
            'clientId'                => config('azure.appId'),
            'clientSecret'            => config('azure.appSecret'),
            'redirectUri'             => config('azure.redirectUri'),
            'urlAuthorize'            => config('azure.authority').config('azure.authorizeEndpoint'),
            'urlAccessToken'          => config('azure.authority').config('azure.tokenEndpoint'),
            'urlResourceOwnerDetails' => '',
            'scopes'                  => config('azure.scopes')
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

    <span data-ttu-id="a2aac-110">Esto define un controlador con dos acciones: `signin` y `callback` .</span><span class="sxs-lookup"><span data-stu-id="a2aac-110">This defines a controller with two actions: `signin` and `callback`.</span></span>

    <span data-ttu-id="a2aac-111">La acción genera la dirección URL de inicio de sesión de Azure AD, guarda el valor generado por el cliente `signin` de OAuth y, a continuación, redirige el explorador a la página de inicio de sesión `state` de Azure AD.</span><span class="sxs-lookup"><span data-stu-id="a2aac-111">The `signin` action generates the Azure AD signin URL, saves the `state` value generated by the OAuth client, then redirects the browser to the Azure AD signin page.</span></span>

    <span data-ttu-id="a2aac-112">La `callback` acción es donde Azure redirige una vez completado el inicio de sesión.</span><span class="sxs-lookup"><span data-stu-id="a2aac-112">The `callback` action is where Azure redirects after the signin is complete.</span></span> <span data-ttu-id="a2aac-113">Esa acción se asegura de que el valor coincide con el valor guardado y, a continuación, usa el código de autorización enviado por `state` Azure para solicitar un token de acceso.</span><span class="sxs-lookup"><span data-stu-id="a2aac-113">That action makes sure the `state` value matches the saved value, then users the authorization code sent by Azure to request an access token.</span></span> <span data-ttu-id="a2aac-114">A continuación, redirige de nuevo a la página principal con el token de acceso en el valor de error temporal.</span><span class="sxs-lookup"><span data-stu-id="a2aac-114">It then redirects back to the home page with the access token in the temporary error value.</span></span> <span data-ttu-id="a2aac-115">Usará esto para comprobar que el inicio de sesión funciona antes de seguir.</span><span class="sxs-lookup"><span data-stu-id="a2aac-115">You'll use this to verify that sign-in is working before moving on.</span></span>

1. <span data-ttu-id="a2aac-116">Agregue las rutas a **./routes/web.php**.</span><span class="sxs-lookup"><span data-stu-id="a2aac-116">Add the routes to **./routes/web.php**.</span></span>

    ```php
    Route::get('/signin', 'AuthController@signin');
    Route::get('/callback', 'AuthController@callback');
    ```

1. <span data-ttu-id="a2aac-117">Inicie el servidor y vaya a `https://localhost:8000` .</span><span class="sxs-lookup"><span data-stu-id="a2aac-117">Start the server and browse to `https://localhost:8000`.</span></span> <span data-ttu-id="a2aac-118">Haga clic en el botón de inicio de sesión y se le `https://login.microsoftonline.com` redirigirá.</span><span class="sxs-lookup"><span data-stu-id="a2aac-118">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="a2aac-119">Inicie sesión con su cuenta de Microsoft.</span><span class="sxs-lookup"><span data-stu-id="a2aac-119">Login with your Microsoft account.</span></span>

1. <span data-ttu-id="a2aac-120">Examine la solicitud de consentimiento.</span><span class="sxs-lookup"><span data-stu-id="a2aac-120">Examine the consent prompt.</span></span> <span data-ttu-id="a2aac-121">La lista de permisos corresponde a la lista de ámbitos de permisos configurados en **.env**.</span><span class="sxs-lookup"><span data-stu-id="a2aac-121">The list of permissions correspond to list of permissions scopes configured in **.env**.</span></span>

    - <span data-ttu-id="a2aac-122">**Mantenga el acceso a los datos** a los que le ha concedido acceso: ( ) MSAL solicita este permiso para recuperar `offline_access` tokens de actualización.</span><span class="sxs-lookup"><span data-stu-id="a2aac-122">**Maintain access to data you have given it access to:** (`offline_access`) This permission is requested by MSAL in order to retrieve refresh tokens.</span></span>
    - <span data-ttu-id="a2aac-123">**Inicie sesión y lea su perfil:** ( ) Este permiso permite a la aplicación obtener el perfil y la foto de perfil del usuario que ha iniciado `User.Read` sesión.</span><span class="sxs-lookup"><span data-stu-id="a2aac-123">**Sign you in and read your profile:** (`User.Read`) This permission allows the app to get the logged-in user's profile and profile photo.</span></span>
    - <span data-ttu-id="a2aac-124">**Lea la configuración del buzón:** ( ) Este permiso permite a la aplicación leer la configuración del buzón del usuario, incluido el formato de zona `MailboxSettings.Read` horaria y hora.</span><span class="sxs-lookup"><span data-stu-id="a2aac-124">**Read your mailbox settings:** (`MailboxSettings.Read`) This permission allows the app to read the user's mailbox settings, including time zone and time format.</span></span>
    - <span data-ttu-id="a2aac-125">Tener acceso completo a los **calendarios:** ( ) Este permiso permite a la aplicación leer eventos en el calendario del usuario, agregar nuevos eventos y modificar los `Calendars.ReadWrite` existentes.</span><span class="sxs-lookup"><span data-stu-id="a2aac-125">**Have full access to your calendars:** (`Calendars.ReadWrite`) This permission allows the app to read events on the user's calendar, add new events, and modify existing ones.</span></span>

1. <span data-ttu-id="a2aac-126">Consentimiento de los permisos solicitados.</span><span class="sxs-lookup"><span data-stu-id="a2aac-126">Consent to the requested permissions.</span></span> <span data-ttu-id="a2aac-127">El explorador redirige a la aplicación mostrando el token.</span><span class="sxs-lookup"><span data-stu-id="a2aac-127">The browser redirects to the app, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="a2aac-128">Obtener detalles del usuario</span><span class="sxs-lookup"><span data-stu-id="a2aac-128">Get user details</span></span>

<span data-ttu-id="a2aac-129">En esta sección actualizará el método para obtener el perfil del usuario `callback` de Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="a2aac-129">In this section you'll update the `callback` method to get the user's profile from Microsoft Graph.</span></span>

1. <span data-ttu-id="a2aac-130">Agregue las siguientes instrucciones en la parte `use` superior de **/app/Http/Controllers/AuthController.php**, debajo de la `namespace App\Http\Controllers;` línea.</span><span class="sxs-lookup"><span data-stu-id="a2aac-130">Add the following `use` statements to the top of **/app/Http/Controllers/AuthController.php**, beneath the `namespace App\Http\Controllers;` line.</span></span>

    ```php
    use Microsoft\Graph\Graph;
    use Microsoft\Graph\Model;
    ```

1. <span data-ttu-id="a2aac-131">Reemplace el `try` bloque del método por el código `callback` siguiente.</span><span class="sxs-lookup"><span data-stu-id="a2aac-131">Replace the `try` block in the `callback` method with the following code.</span></span>

    ```php
    try {
      // Make the token request
      $accessToken = $oauthClient->getAccessToken('authorization_code', [
        'code' => $authCode
      ]);

      $graph = new Graph();
      $graph->setAccessToken($accessToken->getToken());

      $user = $graph->createRequest('GET', '/me?$select=displayName,mail,mailboxSettings,userPrincipalName')
        ->setReturnType(Model\User::class)
        ->execute();

      // TEMPORARY FOR TESTING!
      return redirect('/')
        ->with('error', 'Access token received')
        ->with('errorDetail', 'User:'.$user->getDisplayName().', Token:'.$accessToken->getToken());
    }
    ```

<span data-ttu-id="a2aac-132">El nuevo código crea un objeto, asigna el token de acceso y, a continuación, lo `Graph` usa para solicitar el perfil del usuario.</span><span class="sxs-lookup"><span data-stu-id="a2aac-132">The new code creates a `Graph` object, assigns the access token, then uses it to request the user's profile.</span></span> <span data-ttu-id="a2aac-133">Agrega el nombre para mostrar del usuario a la salida temporal para realizar pruebas.</span><span class="sxs-lookup"><span data-stu-id="a2aac-133">It adds the user's display name to the temporary output for testing.</span></span>

## <a name="storing-the-tokens"></a><span data-ttu-id="a2aac-134">Almacenar los tokens</span><span class="sxs-lookup"><span data-stu-id="a2aac-134">Storing the tokens</span></span>

<span data-ttu-id="a2aac-135">Ahora que puede obtener tokens, es el momento de implementar una forma de almacenarlos en la aplicación.</span><span class="sxs-lookup"><span data-stu-id="a2aac-135">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="a2aac-136">Dado que se trata de una aplicación de ejemplo, por motivos de simplicidad, las almacenarás en la sesión.</span><span class="sxs-lookup"><span data-stu-id="a2aac-136">Since this is a sample app, for simplicity's sake, you'll store them in the session.</span></span> <span data-ttu-id="a2aac-137">Una aplicación del mundo real usaría una solución de almacenamiento seguro más confiable, como una base de datos.</span><span class="sxs-lookup"><span data-stu-id="a2aac-137">A real-world app would use a more reliable secure storage solution, like a database.</span></span>

1. <span data-ttu-id="a2aac-138">Crea un nuevo directorio en el directorio **./app** denominado , luego crea un nuevo archivo en ese directorio denominado `TokenStore` y agrega el siguiente `TokenCache.php` código.</span><span class="sxs-lookup"><span data-stu-id="a2aac-138">Create a new directory in the **./app** directory named `TokenStore`, then create a new file in that directory named `TokenCache.php`, and add the following code.</span></span>

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
          'userEmail' => null !== $user->getMail() ? $user->getMail() : $user->getUserPrincipalName(),
          'userTimeZone' => $user->getMailboxSettings()->getTimeZone()
        ]);
      }

      public function clearTokens() {
        session()->forget('accessToken');
        session()->forget('refreshToken');
        session()->forget('tokenExpires');
        session()->forget('userName');
        session()->forget('userEmail');
        session()->forget('userTimeZone');
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

1. <span data-ttu-id="a2aac-139">Agregue la siguiente `use` instrucción en la parte superior de **./app/Http/Controllers/AuthController.php**, debajo de la `namespace App\Http\Controllers;` línea.</span><span class="sxs-lookup"><span data-stu-id="a2aac-139">Add the following `use` statement to the top of **./app/Http/Controllers/AuthController.php**, beneath the `namespace App\Http\Controllers;` line.</span></span>

    ```php
    use App\TokenStore\TokenCache;
    ```

1. <span data-ttu-id="a2aac-140">Reemplace el `try` bloque de la función existente por lo `callback` siguiente.</span><span class="sxs-lookup"><span data-stu-id="a2aac-140">Replace the `try` block in the existing `callback` function with the following.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/AuthController.php" id="StoreTokensSnippet":::

## <a name="implement-sign-out"></a><span data-ttu-id="a2aac-141">Implementar el cerrar sesión</span><span class="sxs-lookup"><span data-stu-id="a2aac-141">Implement sign-out</span></span>

<span data-ttu-id="a2aac-142">Antes de probar esta nueva característica, agregue una forma de cerrar sesión.</span><span class="sxs-lookup"><span data-stu-id="a2aac-142">Before you test this new feature, add a way to sign out.</span></span>

1. <span data-ttu-id="a2aac-143">Agregue la siguiente acción a la `AuthController` clase.</span><span class="sxs-lookup"><span data-stu-id="a2aac-143">Add the following action to the `AuthController` class.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/AuthController.php" id="SignOutSnippet":::

1. <span data-ttu-id="a2aac-144">Agregue esta acción **a ./routes/web.php**.</span><span class="sxs-lookup"><span data-stu-id="a2aac-144">Add this action to **./routes/web.php**.</span></span>

    ```php
    Route::get('/signout', 'AuthController@signout');
    ```

1. <span data-ttu-id="a2aac-145">Reinicie el servidor y vaya a través del proceso de inicio de sesión.</span><span class="sxs-lookup"><span data-stu-id="a2aac-145">Restart the server and go through the sign-in process.</span></span> <span data-ttu-id="a2aac-146">Debería volver a la página principal, pero la interfaz de usuario debe cambiar para indicar que ha iniciado sesión.</span><span class="sxs-lookup"><span data-stu-id="a2aac-146">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

    ![Captura de pantalla de la página principal después de iniciar sesión](./images/add-aad-auth-01.png)

1. <span data-ttu-id="a2aac-148">Haz clic en el avatar del usuario en la esquina superior derecha para acceder al vínculo **Cerrar** sesión.</span><span class="sxs-lookup"><span data-stu-id="a2aac-148">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="a2aac-149">Al **hacer clic en** Cerrar sesión, se restablece la sesión y se vuelve a la página principal.</span><span class="sxs-lookup"><span data-stu-id="a2aac-149">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

    ![Captura de pantalla del menú desplegable con el vínculo Cerrar sesión](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="a2aac-151">Actualización de tokens</span><span class="sxs-lookup"><span data-stu-id="a2aac-151">Refreshing tokens</span></span>

<span data-ttu-id="a2aac-152">En este momento, la aplicación tiene un token de acceso, que se envía en el `Authorization` encabezado de las llamadas API.</span><span class="sxs-lookup"><span data-stu-id="a2aac-152">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="a2aac-153">Este es el token que permite que la aplicación acceda a Microsoft Graph en nombre del usuario.</span><span class="sxs-lookup"><span data-stu-id="a2aac-153">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="a2aac-154">Sin embargo, este token es de corta duración.</span><span class="sxs-lookup"><span data-stu-id="a2aac-154">However, this token is short-lived.</span></span> <span data-ttu-id="a2aac-155">El token expira una hora después de su emisión.</span><span class="sxs-lookup"><span data-stu-id="a2aac-155">The token expires an hour after it is issued.</span></span> <span data-ttu-id="a2aac-156">Aquí es donde el token de actualización resulta útil.</span><span class="sxs-lookup"><span data-stu-id="a2aac-156">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="a2aac-157">El token de actualización permite a la aplicación solicitar un nuevo token de acceso sin necesidad de que el usuario vuelva a iniciar sesión.</span><span class="sxs-lookup"><span data-stu-id="a2aac-157">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span> <span data-ttu-id="a2aac-158">Actualice el código de administración de tokens para implementar la actualización de tokens.</span><span class="sxs-lookup"><span data-stu-id="a2aac-158">Update the token management code to implement token refresh.</span></span>

1. <span data-ttu-id="a2aac-159">Abra **./app/TokenStore/TokenCache.php** y agregue la siguiente función a la `TokenCache` clase.</span><span class="sxs-lookup"><span data-stu-id="a2aac-159">Open **./app/TokenStore/TokenCache.php** and add the following function to the `TokenCache` class.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/TokenStore/TokenCache.php" id="UpdateTokensSnippet":::

1. <span data-ttu-id="a2aac-160">Reemplace la función `getAccessToken` existente por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="a2aac-160">Replace the existing `getAccessToken` function with the following.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/TokenStore/TokenCache.php" id="GetAccessTokenSnippet":::

<span data-ttu-id="a2aac-161">Este método comprueba primero si el token de acceso ha expirado o está a punto de expirar.</span><span class="sxs-lookup"><span data-stu-id="a2aac-161">This method first checks if the access token is expired or close to expiring.</span></span> <span data-ttu-id="a2aac-162">Si es así, usa el token de actualización para obtener nuevos tokens y, a continuación, actualiza la memoria caché y devuelve el nuevo token de acceso.</span><span class="sxs-lookup"><span data-stu-id="a2aac-162">If it is, then it uses the refresh token to get new tokens, then updates the cache and returns the new access token.</span></span>
