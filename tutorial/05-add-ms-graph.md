<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="72832-101">En este ejercicio, incorporará Microsoft Graph a la aplicación.</span><span class="sxs-lookup"><span data-stu-id="72832-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="72832-102">Para esta aplicación, usará la biblioteca [de Microsoft-Graph](https://github.com/microsoftgraph/msgraph-sdk-php) para realizar llamadas a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="72832-102">For this application, you will use the [microsoft-graph](https://github.com/microsoftgraph/msgraph-sdk-php) library to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="72832-103">Obtener eventos de calendario de Outlook</span><span class="sxs-lookup"><span data-stu-id="72832-103">Get calendar events from Outlook</span></span>

1. <span data-ttu-id="72832-104">Cree un nuevo directorio en el directorio **./app** denominado `TimeZones` , cree un nuevo archivo en ese directorio denominado `TimeZones.php` y agregue el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="72832-104">Create a new directory in the **./app** directory named `TimeZones`, then create a new file in that directory named `TimeZones.php`, and add the following code.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/TimeZones/TimeZones.php":::

    <span data-ttu-id="72832-105">Esta clase implementa una asignación simplista de los nombres de zona horaria de Windows a los identificadores de zona horaria de IANA.</span><span class="sxs-lookup"><span data-stu-id="72832-105">This class implements a simplistic mapping of Windows time zone names to IANA time zone identifiers.</span></span>

1. <span data-ttu-id="72832-106">Cree un nuevo archivo en el directorio **./app/http/Controllers** denominado `CalendarController.php` y agregue el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="72832-106">Create a new file in the **./app/Http/Controllers** directory named `CalendarController.php`, and add the following code.</span></span>

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

    <span data-ttu-id="72832-107">Tenga en cuenta lo que está haciendo este código.</span><span class="sxs-lookup"><span data-stu-id="72832-107">Consider what this code is doing.</span></span>

    - <span data-ttu-id="72832-108">La dirección URL a la que se llamará es `/v1.0/me/calendarView` .</span><span class="sxs-lookup"><span data-stu-id="72832-108">The URL that will be called is `/v1.0/me/calendarView`.</span></span>
    - <span data-ttu-id="72832-109">Los `startDateTime` `endDateTime` parámetros y definen el inicio y el final de la vista.</span><span class="sxs-lookup"><span data-stu-id="72832-109">The `startDateTime` and `endDateTime` parameters define the start and end of the view.</span></span>
    - <span data-ttu-id="72832-110">El `$select` parámetro limita los campos devueltos para cada evento a solo aquellos que la vista usará realmente.</span><span class="sxs-lookup"><span data-stu-id="72832-110">The `$select` parameter limits the fields returned for each events to just those the view will actually use.</span></span>
    - <span data-ttu-id="72832-111">El `$orderby` parámetro ordena los resultados por la fecha y hora en que se crearon, con el elemento más reciente en primer lugar.</span><span class="sxs-lookup"><span data-stu-id="72832-111">The `$orderby` parameter sorts the results by the date and time they were created, with the most recent item being first.</span></span>
    - <span data-ttu-id="72832-112">El `$top` parámetro limita los resultados a 25 eventos.</span><span class="sxs-lookup"><span data-stu-id="72832-112">The `$top` parameter limits the results to 25 events.</span></span>
    - <span data-ttu-id="72832-113">El `Prefer: outlook.timezone=""` encabezado hace que las horas de inicio y finalización de la respuesta se ajusten a la zona horaria preferida del usuario.</span><span class="sxs-lookup"><span data-stu-id="72832-113">The `Prefer: outlook.timezone=""` header causes the start and end times in the response to be adjusted to the user's preferred time zone.</span></span>

1. <span data-ttu-id="72832-114">Actualice las rutas en **./Routes/Web.php** para agregar una ruta a este nuevo controlador.</span><span class="sxs-lookup"><span data-stu-id="72832-114">Update the routes in **./routes/web.php** to add a route to this new controller.</span></span>

    ```php
    Route::get('/calendar', 'CalendarController@calendar');
    ```

1. <span data-ttu-id="72832-115">Inicie sesión y haga clic en el vínculo de **calendario** en la barra de navegación.</span><span class="sxs-lookup"><span data-stu-id="72832-115">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="72832-116">Si todo funciona, debería ver un volcado JSON de eventos en el calendario del usuario.</span><span class="sxs-lookup"><span data-stu-id="72832-116">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="72832-117">Mostrar los resultados</span><span class="sxs-lookup"><span data-stu-id="72832-117">Display the results</span></span>

<span data-ttu-id="72832-118">Ahora puede Agregar una vista para mostrar los resultados de forma más fácil de uso.</span><span class="sxs-lookup"><span data-stu-id="72832-118">Now you can add a view to display the results in a more user-friendly manner.</span></span>

1. <span data-ttu-id="72832-119">Cree un nuevo archivo en el directorio **./Resources/views** denominado `calendar.blade.php` y agregue el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="72832-119">Create a new file in the **./resources/views** directory named `calendar.blade.php` and add the following code.</span></span>

    :::code language="php" source="../demo/graph-tutorial/resources/views/calendar.blade.php" id="CalendarSnippet":::

    <span data-ttu-id="72832-120">Se recorrerá en bucle una colección de eventos y se agregará una fila de tabla para cada uno.</span><span class="sxs-lookup"><span data-stu-id="72832-120">That will loop through a collection of events and add a table row for each one.</span></span>

1. <span data-ttu-id="72832-121">Quite la `return response()->json($events);` línea de la `calendar` acción en **./app/http/Controllers/CalendarController.php**y reemplácela por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="72832-121">Remove the `return response()->json($events);` line from the `calendar` action in **./app/Http/Controllers/CalendarController.php**, and replace it with the following code.</span></span>

    ```php
    $viewData['events'] = $events;
    return view('calendar', $viewData);
    ```

1. <span data-ttu-id="72832-122">Actualice la página y la aplicación ahora debería representar una tabla de eventos.</span><span class="sxs-lookup"><span data-stu-id="72832-122">Refresh the page and the app should now render a table of events.</span></span>

    ![Captura de pantalla de la tabla de eventos](./images/add-msgraph-01.png)
