<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="71ef0-101">En este ejercicio, incorporará Microsoft Graph a la aplicación.</span><span class="sxs-lookup"><span data-stu-id="71ef0-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="71ef0-102">Para esta aplicación, usará la biblioteca [de Microsoft-Graph](https://github.com/microsoftgraph/msgraph-sdk-php) para realizar llamadas a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="71ef0-102">For this application, you will use the [microsoft-graph](https://github.com/microsoftgraph/msgraph-sdk-php) library to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="71ef0-103">Obtener eventos de calendario de Outlook</span><span class="sxs-lookup"><span data-stu-id="71ef0-103">Get calendar events from Outlook</span></span>

<span data-ttu-id="71ef0-104">Empecemos agregando un controlador para la vista de calendario.</span><span class="sxs-lookup"><span data-stu-id="71ef0-104">Let's start by adding a controller for the calendar view.</span></span> <span data-ttu-id="71ef0-105">Cree un nuevo archivo en la `./app/Http/Controllers` carpeta denominada `CalendarController.php`y agregue el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="71ef0-105">Create a new file in the `./app/Http/Controllers` folder named `CalendarController.php`, and add the following code.</span></span>

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use Microsoft\Graph\Graph;
use Microsoft\Graph\Model;
use App\TokenStore\TokenCache;

class CalendarController extends Controller
{
  public function calendar()
  {
    $viewData = $this->loadViewData();

    // Get the access token from the cache
    $tokenCache = new TokenCache();
    $accessToken = $tokenCache->getAccessToken();

    // Create a Graph client
    $graph = new Graph();
    $graph->setAccessToken($accessToken);

    $queryParams = array(
      '$select' => 'subject,organizer,start,end',
      '$orderby' => 'createdDateTime DESC'
    );

    // Append query parameters to the '/me/events' url
    $getEventsUrl = '/me/events?'.http_build_query($queryParams);

    $events = $graph->createRequest('GET', $getEventsUrl)
      ->setReturnType(Model\Event::class)
      ->execute();

    return response()->json($events);
  }
}
```

<span data-ttu-id="71ef0-106">Tenga en cuenta lo que está haciendo este código.</span><span class="sxs-lookup"><span data-stu-id="71ef0-106">Consider what this code is doing.</span></span>

- <span data-ttu-id="71ef0-107">La dirección URL a la que se `/v1.0/me/events`llamará es.</span><span class="sxs-lookup"><span data-stu-id="71ef0-107">The URL that will be called is `/v1.0/me/events`.</span></span>
- <span data-ttu-id="71ef0-108">El `$select` parámetro limita los campos devueltos para cada evento a solo aquellos que la vista usará realmente.</span><span class="sxs-lookup"><span data-stu-id="71ef0-108">The `$select` parameter limits the fields returned for each events to just those the view will actually use.</span></span>
- <span data-ttu-id="71ef0-109">El `$orderby` parámetro ordena los resultados por la fecha y hora en que se crearon, con el elemento más reciente en primer lugar.</span><span class="sxs-lookup"><span data-stu-id="71ef0-109">The `$orderby` parameter sorts the results by the date and time they were created, with the most recent item being first.</span></span>

<span data-ttu-id="71ef0-110">Actualice las rutas en `./routes/web.php` para agregar una ruta a este nuevo controlador</span><span class="sxs-lookup"><span data-stu-id="71ef0-110">Update the routes in `./routes/web.php` to add a route to this new controller</span></span>

```php
Route::get('/calendar', 'CalendarController@calendar');
```

<span data-ttu-id="71ef0-111">Ahora puede probar esto.</span><span class="sxs-lookup"><span data-stu-id="71ef0-111">Now you can test this.</span></span> <span data-ttu-id="71ef0-112">Inicie sesión y haga clic en el vínculo de **calendario** en la barra de navegación.</span><span class="sxs-lookup"><span data-stu-id="71ef0-112">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="71ef0-113">Si todo funciona, debería ver un volcado JSON de eventos en el calendario del usuario.</span><span class="sxs-lookup"><span data-stu-id="71ef0-113">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="71ef0-114">Mostrar los resultados</span><span class="sxs-lookup"><span data-stu-id="71ef0-114">Display the results</span></span>

<span data-ttu-id="71ef0-115">Ahora puede Agregar una vista para mostrar los resultados de forma más fácil de uso.</span><span class="sxs-lookup"><span data-stu-id="71ef0-115">Now you can add a view to display the results in a more user-friendly manner.</span></span> <span data-ttu-id="71ef0-116">Cree un nuevo archivo en el `./resources/views` directorio denominado `calendar.blade.php` y agregue el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="71ef0-116">Create a new file in the `./resources/views` directory named `calendar.blade.php` and add the following code.</span></span>

```php
@extends('layout')

@section('content')
<h1>Calendar</h1>
<table class="table">
  <thead>
    <tr>
      <th scope="col">Organizer</th>
      <th scope="col">Subject</th>
      <th scope="col">Start</th>
      <th scope="col">End</th>
    </tr>
  </thead>
  <tbody>
    @isset($events)
      @foreach($events as $event)
        <tr>
          <td>{{ $event->getOrganizer()->getEmailAddress()->getName() }}</td>
          <td>{{ $event->getSubject() }}</td>
          <td>{{ \Carbon\Carbon::parse($event->getStart()->getDateTime())->format('n/j/y g:i A') }}</td>
          <td>{{ \Carbon\Carbon::parse($event->getEnd()->getDateTime())->format('n/j/y g:i A') }}</td>
        </tr>
      @endforeach
    @endif
  </tbody>
</table>
@endsection
```

<span data-ttu-id="71ef0-117">Se recorrerá en bucle una colección de eventos y se agregará una fila de tabla para cada uno.</span><span class="sxs-lookup"><span data-stu-id="71ef0-117">That will loop through a collection of events and add a table row for each one.</span></span> <span data-ttu-id="71ef0-118">Quite la `return response()->json($events);` línea de la `calendar` acción en `./app/Http/Controllers/CalendarController.php`y reemplácela por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="71ef0-118">Remove the `return response()->json($events);` line from the `calendar` action in `./app/Http/Controllers/CalendarController.php`, and replace it with the following code.</span></span>

```php
$viewData['events'] = $events;
return view('calendar', $viewData);
```

<span data-ttu-id="71ef0-119">Actualice la página y la aplicación ahora debería representar una tabla de eventos.</span><span class="sxs-lookup"><span data-stu-id="71ef0-119">Refresh the page and the app should now render a table of events.</span></span>

![Captura de pantalla de la tabla de eventos](./images/add-msgraph-01.png)