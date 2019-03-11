<!-- markdownlint-disable MD002 MD041 -->

En este ejercicio, incorporará Microsoft Graph a la aplicación. Para esta aplicación, usará la biblioteca [de Microsoft-Graph](https://github.com/microsoftgraph/msgraph-sdk-php) para realizar llamadas a Microsoft Graph.

## <a name="get-calendar-events-from-outlook"></a>Obtener eventos de calendario de Outlook

Empecemos agregando un controlador para la vista de calendario. Cree un nuevo archivo en la `./app/Http/Controllers` carpeta denominada `CalendarController.php`y agregue el código siguiente.

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

Tenga en cuenta lo que está haciendo este código.

- La dirección URL a la que se `/v1.0/me/events`llamará es.
- El `$select` parámetro limita los campos devueltos para cada evento a solo aquellos que la vista usará realmente.
- El `$orderby` parámetro ordena los resultados por la fecha y hora en que se crearon, con el elemento más reciente en primer lugar.

Actualice las rutas en `./routes/web.php` para agregar una ruta a este nuevo controlador

```php
Route::get('/calendar', 'CalendarController@calendar');
```

Ahora puede probar esto. Inicie sesión y haga clic en el vínculo de **calendario** en la barra de navegación. Si todo funciona, debería ver un volcado JSON de eventos en el calendario del usuario.

## <a name="display-the-results"></a>Mostrar los resultados

Ahora puede Agregar una vista para mostrar los resultados de forma más fácil de uso. Cree un nuevo archivo en el `./resources/views` directorio denominado `calendar.blade.php` y agregue el siguiente código.

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

Se recorrerá en bucle una colección de eventos y se agregará una fila de tabla para cada uno. Quite la `return response()->json($events);` línea de la `calendar` acción en `./app/Http/Controllers/CalendarController.php`y reemplácela por el código siguiente.

```php
$viewData['events'] = $events;
return view('calendar', $viewData);
```

Actualice la página y la aplicación ahora debería representar una tabla de eventos.

![Captura de pantalla de la tabla de eventos](./images/add-msgraph-01.png)