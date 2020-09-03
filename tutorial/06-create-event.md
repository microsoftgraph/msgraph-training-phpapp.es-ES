<!-- markdownlint-disable MD002 MD041 -->

En esta sección, agregará la capacidad de crear eventos en el calendario del usuario.

## <a name="create-new-event-form"></a>Crear nuevo formulario de eventos

1. Cree un nuevo archivo en el directorio **./Resources/views** denominado `newevent.blade.php` y agregue el siguiente código.

    :::code language="php" source="../demo/graph-tutorial/resources/views/newevent.blade.php" id="NewEventFormSnippet":::

## <a name="add-controller-actions"></a>Agregar acciones de controlador

1. Abra **./app/http/Controllers/CalendarController.php** y agregue la siguiente función para representar el formulario.

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/CalendarController.php" id="getNewEventFormSnippet":::

1. Agregue la siguiente función para recibir los datos del formulario cuando el usuario envíe y cree un nuevo evento en el calendario del usuario.

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/CalendarController.php" id="createNewEventSnippet":::

    Considere lo que hace este código.

    - Convierte la entrada del campo asistentes en una matriz de objetos [asistentes](https://docs.microsoft.com/graph/api/resources/attendee?view=graph-rest-1.0) del gráfico.
    - Genera un [evento](https://docs.microsoft.com/graph/api/resources/event?view=graph-rest-1.0) a partir de la entrada de formulario.
    - Envía una publicación al `/me/events` extremo y, a continuación, redirige de nuevo a la vista de calendario.

1. Actualice las rutas de **./Routes/Web.php** para agregar rutas para estas funciones nuevas en el controlador.

    ```php
    Route::get('/calendar/new', 'CalendarController@getNewEventForm');
    Route::post('/calendar/new', 'CalendarController@createNewEvent');
    ```

1. Guarde todos los cambios y reinicie el servidor. Use el botón **nuevo evento** para navegar hasta el nuevo formulario de evento.

1. Rellene los valores en el formulario. Usar una fecha de inicio de la semana actual. Seleccione **Crear**.

    ![Captura de pantalla del nuevo formulario de eventos](images/create-event-01.png)

1. Cuando la aplicación redirige a la vista de calendario, compruebe que el nuevo evento está presente en los resultados.
