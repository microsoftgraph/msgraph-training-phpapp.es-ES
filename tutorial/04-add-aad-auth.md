<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="064ee-101">En este ejercicio, ampliará la aplicación del ejercicio anterior para admitir la autenticación con Azure AD.</span><span class="sxs-lookup"><span data-stu-id="064ee-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="064ee-102">Esto es necesario para obtener el token de acceso de OAuth necesario para llamar a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="064ee-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="064ee-103">En este paso, integrará la biblioteca [de OAuth2-Client](https://github.com/thephpleague/oauth2-client) en la aplicación.</span><span class="sxs-lookup"><span data-stu-id="064ee-103">In this step you will integrate the [oauth2-client](https://github.com/thephpleague/oauth2-client) library into the application.</span></span>

1. <span data-ttu-id="064ee-104">Abra el archivo **. env** en la raíz de la aplicación PHP y agregue el siguiente código al final del archivo.</span><span class="sxs-lookup"><span data-stu-id="064ee-104">Open the **.env** file in the root of your PHP application, and add the following code to the end of the file.</span></span>

    :::code language="ini" source="../demo/graph-tutorial/.env.example" id="OAuthSettingsSnippet":::

1. <span data-ttu-id="064ee-105">Reemplace `YOUR_APP_ID_HERE` por el identificador de la aplicación del portal de registro de la aplicación y reemplace `YOUR_APP_PASSWORD_HERE` por la contraseña que ha generado.</span><span class="sxs-lookup"><span data-stu-id="064ee-105">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_PASSWORD_HERE` with the password you generated.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="064ee-106">Si usa un control de código fuente como GIT, ahora sería un buen momento para excluir el `.env` archivo del control de código fuente para evitar la pérdida inadvertida del identificador de la aplicación y la contraseña.</span><span class="sxs-lookup"><span data-stu-id="064ee-106">If you're using source control such as git, now would be a good time to exclude the `.env` file from source control to avoid inadvertently leaking your app ID and password.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="064ee-107">Implementar el inicio de sesión</span><span class="sxs-lookup"><span data-stu-id="064ee-107">Implement sign-in</span></span>

1. <span data-ttu-id="064ee-108">Cree un nuevo archivo en el directorio **./app/http/Controllers** denominado `AuthController.php` y agregue el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="064ee-108">Create a new file in the **./app/Http/Controllers** directory named `AuthController.php` and add the following code.</span></span>

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

    <span data-ttu-id="064ee-109">Esto define un controlador con dos acciones: `signin` y `callback` .</span><span class="sxs-lookup"><span data-stu-id="064ee-109">This defines a controller with two actions: `signin` and `callback`.</span></span>

    <span data-ttu-id="064ee-110">La `signin` acción genera la dirección URL de inicio de sesión de Azure ad, guarda el `state` valor generado por el cliente de OAuth y, a continuación, redirige el explorador a la página de inicio de sesión de Azure ad.</span><span class="sxs-lookup"><span data-stu-id="064ee-110">The `signin` action generates the Azure AD signin URL, saves the `state` value generated by the OAuth client, then redirects the browser to the Azure AD signin page.</span></span>

    <span data-ttu-id="064ee-111">La `callback` acción es donde Azure redirige después de que se complete el inicio de sesión.</span><span class="sxs-lookup"><span data-stu-id="064ee-111">The `callback` action is where Azure redirects after the signin is complete.</span></span> <span data-ttu-id="064ee-112">Esa acción garantiza que el `state` valor coincida con el valor guardado y, a continuación, los usuarios el código de autorización enviado por Azure para solicitar un token de acceso.</span><span class="sxs-lookup"><span data-stu-id="064ee-112">That action makes sure the `state` value matches the saved value, then users the authorization code sent by Azure to request an access token.</span></span> <span data-ttu-id="064ee-113">A continuación, se redirige de nuevo a la Página principal con el token de acceso en el valor de error temporal.</span><span class="sxs-lookup"><span data-stu-id="064ee-113">It then redirects back to the home page with the access token in the temporary error value.</span></span> <span data-ttu-id="064ee-114">Usará esto para comprobar que el inicio de sesión está funcionando antes de continuar.</span><span class="sxs-lookup"><span data-stu-id="064ee-114">You'll use this to verify that sign-in is working before moving on.</span></span>

1. <span data-ttu-id="064ee-115">Agregue las rutas a **./Routes/Web.php**.</span><span class="sxs-lookup"><span data-stu-id="064ee-115">Add the routes to **./routes/web.php**.</span></span>

    ```php
    Route::get('/signin', 'AuthController@signin');
    Route::get('/callback', 'AuthController@callback');
    ```

1. <span data-ttu-id="064ee-116">Inicie el servidor y vaya a `https://localhost:8000` .</span><span class="sxs-lookup"><span data-stu-id="064ee-116">Start the server and browse to `https://localhost:8000`.</span></span> <span data-ttu-id="064ee-117">Haga clic en el botón de inicio de sesión y se le redirigirá a `https://login.microsoftonline.com` .</span><span class="sxs-lookup"><span data-stu-id="064ee-117">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="064ee-118">Inicie sesión con su cuenta de Microsoft.</span><span class="sxs-lookup"><span data-stu-id="064ee-118">Login with your Microsoft account.</span></span>

1. <span data-ttu-id="064ee-119">Examine la solicitud de consentimiento.</span><span class="sxs-lookup"><span data-stu-id="064ee-119">Examine the consent prompt.</span></span> <span data-ttu-id="064ee-120">La lista de permisos se corresponde con la lista de ámbitos de permisos configurados en **. env**.</span><span class="sxs-lookup"><span data-stu-id="064ee-120">The list of permissions correspond to list of permissions scopes configured in **.env**.</span></span>

    - <span data-ttu-id="064ee-121">**Mantener el acceso a los datos a los que ha dado acceso a:** ( `offline_access` ) este permiso se solicita mediante la MSAL para recuperar los tokens de actualización.</span><span class="sxs-lookup"><span data-stu-id="064ee-121">**Maintain access to data you have given it access to:** (`offline_access`) This permission is requested by MSAL in order to retrieve refresh tokens.</span></span>
    - <span data-ttu-id="064ee-122">**Inicie sesión y lea su perfil:** ( `User.Read` ) este permiso permite a la aplicación obtener el perfil y la foto de perfil del usuario que ha iniciado sesión.</span><span class="sxs-lookup"><span data-stu-id="064ee-122">**Sign you in and read your profile:** (`User.Read`) This permission allows the app to get the logged-in user's profile and profile photo.</span></span>
    - <span data-ttu-id="064ee-123">**Leer la configuración del buzón de correo:** ( `MailboxSettings.Read` ) este permiso permite que la aplicación Lea la configuración del buzón del usuario, incluido el formato de hora y zona horaria.</span><span class="sxs-lookup"><span data-stu-id="064ee-123">**Read your mailbox settings:** (`MailboxSettings.Read`) This permission allows the app to read the user's mailbox settings, including time zone and time format.</span></span>
    - <span data-ttu-id="064ee-124">**Tener acceso completo a sus calendarios:** ( `Calendars.ReadWrite` ) este permiso permite a la aplicación leer eventos en el calendario del usuario, agregar nuevos eventos y modificar los existentes.</span><span class="sxs-lookup"><span data-stu-id="064ee-124">**Have full access to your calendars:** (`Calendars.ReadWrite`) This permission allows the app to read events on the user's calendar, add new events, and modify existing ones.</span></span>

1. <span data-ttu-id="064ee-125">Consentimiento para los permisos solicitados.</span><span class="sxs-lookup"><span data-stu-id="064ee-125">Consent to the requested permissions.</span></span> <span data-ttu-id="064ee-126">El explorador redirige a la aplicación, que muestra el token.</span><span class="sxs-lookup"><span data-stu-id="064ee-126">The browser redirects to the app, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="064ee-127">Obtener detalles del usuario</span><span class="sxs-lookup"><span data-stu-id="064ee-127">Get user details</span></span>

<span data-ttu-id="064ee-128">En esta sección, actualizará el `callback` método para obtener el perfil del usuario de Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="064ee-128">In this section you'll update the `callback` method to get the user's profile from Microsoft Graph.</span></span>

1. <span data-ttu-id="064ee-129">Agregue las siguientes `use` instrucciones a la parte superior de **/App/http/Controllers/AuthController.php**, debajo de la `namespace App\Http\Controllers;` línea.</span><span class="sxs-lookup"><span data-stu-id="064ee-129">Add the following `use` statements to the top of **/app/Http/Controllers/AuthController.php**, beneath the `namespace App\Http\Controllers;` line.</span></span>

    ```php
    use Microsoft\Graph\Graph;
    use Microsoft\Graph\Model;
    ```

1. <span data-ttu-id="064ee-130">Reemplace el `try` bloque en el `callback` método por el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="064ee-130">Replace the `try` block in the `callback` method with the following code.</span></span>

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

<span data-ttu-id="064ee-131">El nuevo código crea un `Graph` objeto, asigna el token de acceso y, a continuación, lo usa para solicitar el perfil del usuario.</span><span class="sxs-lookup"><span data-stu-id="064ee-131">The new code creates a `Graph` object, assigns the access token, then uses it to request the user's profile.</span></span> <span data-ttu-id="064ee-132">Agrega el nombre para mostrar del usuario a la salida temporal para las pruebas.</span><span class="sxs-lookup"><span data-stu-id="064ee-132">It adds the user's display name to the temporary output for testing.</span></span>

## <a name="storing-the-tokens"></a><span data-ttu-id="064ee-133">Almacenamiento de tokens</span><span class="sxs-lookup"><span data-stu-id="064ee-133">Storing the tokens</span></span>

<span data-ttu-id="064ee-134">Ahora que puede obtener tokens, es el momento de implementar una forma de almacenarlos en la aplicación.</span><span class="sxs-lookup"><span data-stu-id="064ee-134">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="064ee-135">Dado que se trata de una aplicación de ejemplo, por razones de simplicidad, se almacenará en la sesión.</span><span class="sxs-lookup"><span data-stu-id="064ee-135">Since this is a sample app, for simplicity's sake, you'll store them in the session.</span></span> <span data-ttu-id="064ee-136">Una aplicación real usaría una solución de almacenamiento seguro más confiable, como una base de datos.</span><span class="sxs-lookup"><span data-stu-id="064ee-136">A real-world app would use a more reliable secure storage solution, like a database.</span></span>

1. <span data-ttu-id="064ee-137">Cree un nuevo directorio en el directorio **./app** denominado `TokenStore` , cree un nuevo archivo en ese directorio denominado `TokenCache.php` y agregue el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="064ee-137">Create a new directory in the **./app** directory named `TokenStore`, then create a new file in that directory named `TokenCache.php`, and add the following code.</span></span>

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

1. <span data-ttu-id="064ee-138">Agregue la siguiente `use` instrucción a la parte superior de **./app/http/Controllers/AuthController.php**, debajo de la `namespace App\Http\Controllers;` línea.</span><span class="sxs-lookup"><span data-stu-id="064ee-138">Add the following `use` statement to the top of **./app/Http/Controllers/AuthController.php**, beneath the `namespace App\Http\Controllers;` line.</span></span>

    ```php
    use App\TokenStore\TokenCache;
    ```

1. <span data-ttu-id="064ee-139">Reemplace el `try` bloque de la `callback` función existente por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="064ee-139">Replace the `try` block in the existing `callback` function with the following.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/AuthController.php" id="StoreTokensSnippet":::

## <a name="implement-sign-out"></a><span data-ttu-id="064ee-140">Implementación de cierre de sesión</span><span class="sxs-lookup"><span data-stu-id="064ee-140">Implement sign-out</span></span>

<span data-ttu-id="064ee-141">Antes de probar esta nueva característica, agregue una forma de cerrar sesión.</span><span class="sxs-lookup"><span data-stu-id="064ee-141">Before you test this new feature, add a way to sign out.</span></span>

1. <span data-ttu-id="064ee-142">Agregue la siguiente acción a la `AuthController` clase.</span><span class="sxs-lookup"><span data-stu-id="064ee-142">Add the following action to the `AuthController` class.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/AuthController.php" id="SignOutSnippet":::

1. <span data-ttu-id="064ee-143">Agregue esta acción a **./Routes/Web.php**.</span><span class="sxs-lookup"><span data-stu-id="064ee-143">Add this action to **./routes/web.php**.</span></span>

    ```php
    Route::get('/signout', 'AuthController@signout');
    ```

1. <span data-ttu-id="064ee-144">Reinicie el servidor y pase por el proceso de inicio de sesión.</span><span class="sxs-lookup"><span data-stu-id="064ee-144">Restart the server and go through the sign-in process.</span></span> <span data-ttu-id="064ee-145">Deberás volver a la Página principal, pero la interfaz de usuario debe cambiar para indicar que has iniciado sesión.</span><span class="sxs-lookup"><span data-stu-id="064ee-145">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

    ![Una captura de pantalla de la Página principal después de iniciar sesión](./images/add-aad-auth-01.png)

1. <span data-ttu-id="064ee-147">Haga clic en el avatar de usuario en la esquina superior derecha para acceder al vínculo **Cerrar sesión** .</span><span class="sxs-lookup"><span data-stu-id="064ee-147">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="064ee-148">Al hacer clic en **cerrar** sesión se restablece la sesión y se vuelve a la Página principal.</span><span class="sxs-lookup"><span data-stu-id="064ee-148">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

    ![Captura de pantalla del menú desplegable con el vínculo cerrar sesión](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="064ee-150">Actualizar tokens</span><span class="sxs-lookup"><span data-stu-id="064ee-150">Refreshing tokens</span></span>

<span data-ttu-id="064ee-151">En este punto, la aplicación tiene un token de acceso, que se envía en el `Authorization` encabezado de las llamadas a la API.</span><span class="sxs-lookup"><span data-stu-id="064ee-151">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="064ee-152">Este es el token que permite que la aplicación tenga acceso a Microsoft Graph en nombre del usuario.</span><span class="sxs-lookup"><span data-stu-id="064ee-152">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="064ee-153">Sin embargo, este token es de corta duración.</span><span class="sxs-lookup"><span data-stu-id="064ee-153">However, this token is short-lived.</span></span> <span data-ttu-id="064ee-154">El token expira una hora después de su emisión.</span><span class="sxs-lookup"><span data-stu-id="064ee-154">The token expires an hour after it is issued.</span></span> <span data-ttu-id="064ee-155">Aquí es donde el token de actualización se vuelve útil.</span><span class="sxs-lookup"><span data-stu-id="064ee-155">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="064ee-156">El token de actualización permite que la aplicación solicite un nuevo token de acceso sin que el usuario tenga que iniciar sesión de nuevo.</span><span class="sxs-lookup"><span data-stu-id="064ee-156">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span> <span data-ttu-id="064ee-157">Actualice el código de administración de tokens para implementar la actualización de tokens.</span><span class="sxs-lookup"><span data-stu-id="064ee-157">Update the token management code to implement token refresh.</span></span>

1. <span data-ttu-id="064ee-158">Abra **./app/TokenStore/TokenCache.php** y agregue la siguiente función a la `TokenCache` clase.</span><span class="sxs-lookup"><span data-stu-id="064ee-158">Open **./app/TokenStore/TokenCache.php** and add the following function to the `TokenCache` class.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/TokenStore/TokenCache.php" id="UpdateTokensSnippet":::

1. <span data-ttu-id="064ee-159">Reemplace la función `getAccessToken` existente por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="064ee-159">Replace the existing `getAccessToken` function with the following.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/TokenStore/TokenCache.php" id="GetAccessTokenSnippet":::

<span data-ttu-id="064ee-160">Este método comprueba primero si el token de acceso ha expirado o está próximo a expirar.</span><span class="sxs-lookup"><span data-stu-id="064ee-160">This method first checks if the access token is expired or close to expiring.</span></span> <span data-ttu-id="064ee-161">Si es así, usa el token de actualización para obtener nuevos tokens y, a continuación, actualiza la memoria caché y devuelve el nuevo token de acceso.</span><span class="sxs-lookup"><span data-stu-id="064ee-161">If it is, then it uses the refresh token to get new tokens, then updates the cache and returns the new access token.</span></span>
