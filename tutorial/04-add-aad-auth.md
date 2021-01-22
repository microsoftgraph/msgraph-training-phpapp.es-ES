<!-- markdownlint-disable MD002 MD041 -->

En este ejercicio, ampliará la aplicación del ejercicio anterior para admitir la autenticación con Azure AD. Esto es necesario para obtener el token de acceso OAuth necesario para llamar a Microsoft Graph. En este paso, integrará la biblioteca [oauth2-client](https://github.com/thephpleague/oauth2-client) en la aplicación.

1. Abra el **archivo .env** en la raíz de la aplicación PHP y agregue el siguiente código al final del archivo.

    :::code language="ini" source="../demo/graph-tutorial/example.env" range="48-54":::

1. Reemplace por el identificador de aplicación del Portal de registro de aplicaciones y `YOUR_APP_ID_HERE` reemplace por la contraseña que `YOUR_APP_PASSWORD_HERE` generó.

    > [!IMPORTANT]
    > Si usas el control de código fuente como Git, ahora sería un buen momento para excluir el archivo del control de código fuente para evitar la pérdida involuntaria del identificador de la aplicación y la `.env` contraseña.

## <a name="implement-sign-in"></a>Implementar el inicio de sesión

1. Crea un nuevo archivo en el directorio **./app/Http/Controllers** denominado `AuthController.php` y agrega el siguiente código.

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

    Esto define un controlador con dos acciones: `signin` y `callback` .

    La acción genera la dirección URL de inicio de sesión de Azure AD, guarda el valor generado por el cliente `signin` de OAuth y, a continuación, redirige el explorador a la página de inicio de sesión `state` de Azure AD.

    La `callback` acción es donde Azure redirige una vez completado el inicio de sesión. Esa acción se asegura de que el valor coincide con el valor guardado y, a continuación, usa el código de autorización enviado por `state` Azure para solicitar un token de acceso. A continuación, redirige de nuevo a la página principal con el token de acceso en el valor de error temporal. Usará esta opción para comprobar que el inicio de sesión funciona antes de seguir adelante.

1. Agregue las rutas a **./routes/web.php**.

    ```php
    Route::get('/signin', 'AuthController@signin');
    Route::get('/callback', 'AuthController@callback');
    ```

1. Inicie el servidor y vaya a `https://localhost:8000` . Haga clic en el botón de inicio de sesión y se le `https://login.microsoftonline.com` redirigirá. Inicie sesión con su cuenta de Microsoft.

1. Examine la solicitud de consentimiento. La lista de permisos corresponde a la lista de ámbitos de permisos configurados en **.env**.

    - **Mantenga el acceso a los datos** a los que le ha concedido acceso: ( ) MSAL solicita este permiso para recuperar `offline_access` tokens de actualización.
    - **Inicie sesión y lea su perfil:** ( ) Este permiso permite a la aplicación obtener el perfil y la foto de perfil del usuario que ha iniciado `User.Read` sesión.
    - **Lea la configuración del buzón:** ( ) Este permiso permite a la aplicación leer la configuración del buzón del usuario, incluido el formato de zona `MailboxSettings.Read` horaria y hora.
    - **Tener acceso completo a los calendarios:** ( ) Este permiso permite a la aplicación leer eventos en el calendario del usuario, agregar nuevos eventos y modificar los `Calendars.ReadWrite` existentes.

1. Consentimiento de los permisos solicitados. El explorador redirige a la aplicación y muestra el token.

### <a name="get-user-details"></a>Obtener detalles del usuario

En esta sección actualizará el método para obtener el perfil del usuario `callback` de Microsoft Graph.

1. Agregue las siguientes instrucciones en la parte `use` superior de **/app/Http/Controllers/AuthController.php**, debajo de la `namespace App\Http\Controllers;` línea.

    ```php
    use Microsoft\Graph\Graph;
    use Microsoft\Graph\Model;
    ```

1. Reemplace el `try` bloque del método por el código `callback` siguiente.

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

El nuevo código crea un objeto, asigna el token de acceso y, a continuación, lo `Graph` usa para solicitar el perfil del usuario. Agrega el nombre para mostrar del usuario a la salida temporal para realizar pruebas.

## <a name="storing-the-tokens"></a>Almacenar los tokens

Ahora que puedes obtener tokens, es el momento de implementar una forma de almacenarlos en la aplicación. Dado que se trata de una aplicación de ejemplo, por motivos de simplicidad, las almacenarás en la sesión. Una aplicación del mundo real usaría una solución de almacenamiento seguro más confiable, como una base de datos.

1. Crea un nuevo directorio en el directorio **./app** denominado , luego crea un nuevo archivo en ese directorio denominado `TokenStore` y agrega el siguiente `TokenCache.php` código.

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

1. Agregue la siguiente `use` instrucción en la parte superior de **./app/Http/Controllers/AuthController.php**, debajo de la `namespace App\Http\Controllers;` línea.

    ```php
    use App\TokenStore\TokenCache;
    ```

1. Reemplace el `try` bloque de la función existente por lo `callback` siguiente.

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/AuthController.php" id="StoreTokensSnippet":::

## <a name="implement-sign-out"></a>Implementar el cerrar sesión

Antes de probar esta nueva característica, agrega una forma de cerrar sesión.

1. Agregue la siguiente acción a la `AuthController` clase.

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/AuthController.php" id="SignOutSnippet":::

1. Agregue esta acción **a ./routes/web.php**.

    ```php
    Route::get('/signout', 'AuthController@signout');
    ```

1. Reinicie el servidor y vaya a través del proceso de inicio de sesión. Debería volver a la página principal, pero la interfaz de usuario debe cambiar para indicar que ha iniciado sesión.

    ![Captura de pantalla de la página principal después de iniciar sesión](./images/add-aad-auth-01.png)

1. Haz clic en el avatar del usuario en la esquina superior derecha para acceder al vínculo **Cerrar** sesión. Al **hacer clic en** Cerrar sesión, se restablece la sesión y se vuelve a la página principal.

    ![Captura de pantalla del menú desplegable con el vínculo Cerrar sesión](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>Actualización de tokens

En este momento, la aplicación tiene un token de acceso, que se envía en el `Authorization` encabezado de las llamadas API. Este es el token que permite que la aplicación acceda a Microsoft Graph en nombre del usuario.

Sin embargo, este token es de corta duración. El token expira una hora después de su emisión. Aquí es donde el token de actualización resulta útil. El token de actualización permite a la aplicación solicitar un nuevo token de acceso sin necesidad de que el usuario vuelva a iniciar sesión. Actualice el código de administración de tokens para implementar la actualización de tokens.

1. Abra **./app/TokenStore/TokenCache.php** y agregue la siguiente función a la `TokenCache` clase.

    :::code language="php" source="../demo/graph-tutorial/app/TokenStore/TokenCache.php" id="UpdateTokensSnippet":::

1. Reemplace la función `getAccessToken` existente por lo siguiente.

    :::code language="php" source="../demo/graph-tutorial/app/TokenStore/TokenCache.php" id="GetAccessTokenSnippet":::

Este método comprueba primero si el token de acceso ha expirado o está próximo a expirar. Si es así, usa el token de actualización para obtener nuevos tokens y, a continuación, actualiza la memoria caché y devuelve el nuevo token de acceso.
