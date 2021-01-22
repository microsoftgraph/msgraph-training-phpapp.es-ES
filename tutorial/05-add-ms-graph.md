<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="9a7ee-101">En este ejercicio incorporará Microsoft Graph en la aplicación.</span><span class="sxs-lookup"><span data-stu-id="9a7ee-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="9a7ee-102">Para esta aplicación, usará la biblioteca [de Microsoft Graph](https://github.com/microsoftgraph/msgraph-sdk-php) para realizar llamadas a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="9a7ee-102">For this application, you will use the [microsoft-graph](https://github.com/microsoftgraph/msgraph-sdk-php) library to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="9a7ee-103">Obtener eventos de calendario de Outlook</span><span class="sxs-lookup"><span data-stu-id="9a7ee-103">Get calendar events from Outlook</span></span>

1. <span data-ttu-id="9a7ee-104">Crea un nuevo directorio en el directorio **./app** denominado , luego crea un nuevo archivo en ese directorio denominado `TimeZones` y agrega el siguiente `TimeZones.php` código.</span><span class="sxs-lookup"><span data-stu-id="9a7ee-104">Create a new directory in the **./app** directory named `TimeZones`, then create a new file in that directory named `TimeZones.php`, and add the following code.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/TimeZones/TimeZones.php":::

    <span data-ttu-id="9a7ee-105">Esta clase implementa una asignación simple de nombres de zona horaria de Windows a identificadores de zona horaria IANA.</span><span class="sxs-lookup"><span data-stu-id="9a7ee-105">This class implements a simplistic mapping of Windows time zone names to IANA time zone identifiers.</span></span>

1. <span data-ttu-id="9a7ee-106">Crea un nuevo archivo en el directorio **./app/Http/Controllers** denominado `CalendarController.php` y agrega el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="9a7ee-106">Create a new file in the **./app/Http/Controllers** directory named `CalendarController.php`, and add the following code.</span></span>

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

    <span data-ttu-id="9a7ee-107">Ten en cuenta lo que hace este código.</span><span class="sxs-lookup"><span data-stu-id="9a7ee-107">Consider what this code is doing.</span></span>

    - <span data-ttu-id="9a7ee-108">La dirección URL a la que se llamará es `/v1.0/me/calendarView` .</span><span class="sxs-lookup"><span data-stu-id="9a7ee-108">The URL that will be called is `/v1.0/me/calendarView`.</span></span>
    - <span data-ttu-id="9a7ee-109">Los `startDateTime` parámetros y definen el inicio y el final de la `endDateTime` vista.</span><span class="sxs-lookup"><span data-stu-id="9a7ee-109">The `startDateTime` and `endDateTime` parameters define the start and end of the view.</span></span>
    - <span data-ttu-id="9a7ee-110">El `$select` parámetro limita los campos devueltos para cada evento a solo aquellos que la vista usará realmente.</span><span class="sxs-lookup"><span data-stu-id="9a7ee-110">The `$select` parameter limits the fields returned for each events to just those the view will actually use.</span></span>
    - <span data-ttu-id="9a7ee-111">El parámetro ordena los resultados por la fecha y hora en que se crearon, siendo el `$orderby` elemento más reciente el primero.</span><span class="sxs-lookup"><span data-stu-id="9a7ee-111">The `$orderby` parameter sorts the results by the date and time they were created, with the most recent item being first.</span></span>
    - <span data-ttu-id="9a7ee-112">El `$top` parámetro limita los resultados a 25 eventos.</span><span class="sxs-lookup"><span data-stu-id="9a7ee-112">The `$top` parameter limits the results to 25 events.</span></span>
    - <span data-ttu-id="9a7ee-113">El encabezado hace que las horas de inicio y finalización de la respuesta se ajusten a la zona horaria preferida `Prefer: outlook.timezone=""` del usuario.</span><span class="sxs-lookup"><span data-stu-id="9a7ee-113">The `Prefer: outlook.timezone=""` header causes the start and end times in the response to be adjusted to the user's preferred time zone.</span></span>

1. <span data-ttu-id="9a7ee-114">Actualice las rutas en **./routes/web.php** para agregar una ruta a este controlador nuevo.</span><span class="sxs-lookup"><span data-stu-id="9a7ee-114">Update the routes in **./routes/web.php** to add a route to this new controller.</span></span>

    ```php
    Route::get('/calendar', 'CalendarController@calendar');
    ```

1. <span data-ttu-id="9a7ee-115">Inicie sesión y haga clic en el **vínculo** Calendario de la barra de navegación.</span><span class="sxs-lookup"><span data-stu-id="9a7ee-115">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="9a7ee-116">Si todo funciona, debería ver un volcado JSON de eventos en el calendario del usuario.</span><span class="sxs-lookup"><span data-stu-id="9a7ee-116">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="9a7ee-117">Mostrar los resultados</span><span class="sxs-lookup"><span data-stu-id="9a7ee-117">Display the results</span></span>

<span data-ttu-id="9a7ee-118">Ahora puede agregar una vista para mostrar los resultados de una manera más fácil de usar.</span><span class="sxs-lookup"><span data-stu-id="9a7ee-118">Now you can add a view to display the results in a more user-friendly manner.</span></span>

1. <span data-ttu-id="9a7ee-119">Cree un nuevo archivo en el directorio **./resources/views** con nombre `calendar.blade.php` y agregue el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="9a7ee-119">Create a new file in the **./resources/views** directory named `calendar.blade.php` and add the following code.</span></span>

    :::code language="php" source="../demo/graph-tutorial/resources/views/calendar.blade.php" id="CalendarSnippet":::

    <span data-ttu-id="9a7ee-120">Esto recorrerá una colección de eventos y agregará una fila de tabla para cada uno.</span><span class="sxs-lookup"><span data-stu-id="9a7ee-120">That will loop through a collection of events and add a table row for each one.</span></span>

1. <span data-ttu-id="9a7ee-121">Actualice las rutas en **./routes/web.php** para agregar rutas para `/calendar/new` .</span><span class="sxs-lookup"><span data-stu-id="9a7ee-121">Update the routes in **./routes/web.php** to add routes for `/calendar/new`.</span></span> <span data-ttu-id="9a7ee-122">Implementará estas funciones en la siguiente sección, pero la ruta debe definirse ahora porque **calendar.blade.php** hace referencia a ella.</span><span class="sxs-lookup"><span data-stu-id="9a7ee-122">You will implement these functions in the next section, but the route need to be defined now because **calendar.blade.php** references it.</span></span>

    ```php
    Route::get('/calendar/new', 'CalendarController@getNewEventForm');
    Route::post('/calendar/new', 'CalendarController@createNewEvent');
    ```

1. <span data-ttu-id="9a7ee-123">Quite la `return response()->json($events);` línea de la acción en `calendar` **./app/Http/Controllers/CalendarController.php** y reempláctela por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="9a7ee-123">Remove the `return response()->json($events);` line from the `calendar` action in **./app/Http/Controllers/CalendarController.php**, and replace it with the following code.</span></span>

    ```php
    $viewData['events'] = $events;
    return view('calendar', $viewData);
    ```

1. <span data-ttu-id="9a7ee-124">Actualice la página y la aplicación ahora debería representar una tabla de eventos.</span><span class="sxs-lookup"><span data-stu-id="9a7ee-124">Refresh the page and the app should now render a table of events.</span></span>

    ![Captura de pantalla de la tabla de eventos](./images/add-msgraph-01.png)
