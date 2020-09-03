<!-- markdownlint-disable MD002 MD041 -->

En este ejercicio, incorporará Microsoft Graph a la aplicación. Para esta aplicación, usará la biblioteca [de Microsoft-Graph](https://github.com/microsoftgraph/msgraph-sdk-php) para realizar llamadas a Microsoft Graph.

## <a name="get-calendar-events-from-outlook"></a>Obtener eventos de calendario de Outlook

1. Cree un nuevo directorio en el directorio **./app** denominado `TimeZones` , cree un nuevo archivo en ese directorio denominado `TimeZones.php` y agregue el siguiente código.

    :::code language="php" source="../demo/graph-tutorial/app/TimeZones/TimeZones.php":::

    Esta clase implementa una asignación simplista de los nombres de zona horaria de Windows a los identificadores de zona horaria de IANA.

1. Cree un nuevo archivo en el directorio **./app/http/Controllers** denominado `CalendarController.php` y agregue el siguiente código.

    ```php
    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;
    use Microsoft\Graph\Graph;
    use Microsoft\Graph\Model;
    use App\TokenStore\TokenCache;
    use App\TimeZones\TimeZones;

    class CalendarController extends Controller
    {
      public function calendar()
      {
        $viewData = $this->loadViewData();

        $graph = $this->getGraph();

        // Get user's timezone
        $timezone = TimeZones::getTzFromWindows($viewData['userTimeZone']);

        // Get start and end of week
        $startOfWeek = new \DateTimeImmutable('sunday -1 week', $timezone);
        $endOfWeek = new \DateTimeImmutable('sunday', $timezone);

        $queryParams = array(
          'startDateTime' => $startOfWeek->format(\DateTimeInterface::ISO8601),
          'endDateTime' => $endOfWeek->format(\DateTimeInterface::ISO8601),
          // Only request the properties used by the app
          '$select' => 'subject,organizer,start,end',
          // Sort them by start time
          '$orderby' => 'start/dateTime',
          // Limit results to 25
          '$top' => 25
        );

        // Append query parameters to the '/me/calendarView' url
        $getEventsUrl = '/me/calendarView?'.http_build_query($queryParams);

        $events = $graph->createRequest('GET', $getEventsUrl)
          // Add the user's timezone to the Prefer header
          ->addHeaders(array(
            'Prefer' => 'outlook.timezone="'.$viewData['userTimeZone'].'"'
          ))
          ->setReturnType(Model\Event::class)
          ->execute();

        return response()->json($events);
      }

      private function getGraph(): Graph
      {
        // Get the access token from the cache
        $tokenCache = new TokenCache();
        $accessToken = $tokenCache->getAccessToken();

        // Create a Graph client
        $graph = new Graph();
        $graph->setAccessToken($accessToken);
        return $graph;
      }
    }
    ```

    Tenga en cuenta lo que está haciendo este código.

    - La dirección URL a la que se llamará es `/v1.0/me/calendarView` .
    - Los `startDateTime` `endDateTime` parámetros y definen el inicio y el final de la vista.
    - El `$select` parámetro limita los campos devueltos para cada evento a solo aquellos que la vista usará realmente.
    - El `$orderby` parámetro ordena los resultados por la fecha y hora en que se crearon, con el elemento más reciente en primer lugar.
    - El `$top` parámetro limita los resultados a 25 eventos.
    - El `Prefer: outlook.timezone=""` encabezado hace que las horas de inicio y finalización de la respuesta se ajusten a la zona horaria preferida del usuario.

1. Actualice las rutas en **./Routes/Web.php** para agregar una ruta a este nuevo controlador.

    ```php
    Route::get('/calendar', 'CalendarController@calendar');
    ```

1. Inicie sesión y haga clic en el vínculo de **calendario** en la barra de navegación. Si todo funciona, debería ver un volcado JSON de eventos en el calendario del usuario.

## <a name="display-the-results"></a>Mostrar los resultados

Ahora puede Agregar una vista para mostrar los resultados de forma más fácil de uso.

1. Cree un nuevo archivo en el directorio **./Resources/views** denominado `calendar.blade.php` y agregue el siguiente código.

    :::code language="php" source="../demo/graph-tutorial/resources/views/calendar.blade.php" id="CalendarSnippet":::

    Se recorrerá en bucle una colección de eventos y se agregará una fila de tabla para cada uno.

1. Quite la `return response()->json($events);` línea de la `calendar` acción en **./app/http/Controllers/CalendarController.php**y reemplácela por el código siguiente.

    ```php
    $viewData['events'] = $events;
    return view('calendar', $viewData);
    ```

1. Actualice la página y la aplicación ahora debería representar una tabla de eventos.

    ![Captura de pantalla de la tabla de eventos](./images/add-msgraph-01.png)
