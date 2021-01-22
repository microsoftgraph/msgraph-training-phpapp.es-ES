<!-- markdownlint-disable MD002 MD041 -->

En esta sección agregará la capacidad de crear eventos en el calendario del usuario.

## <a name="create-new-event-form"></a>Crear nuevo formulario de evento

1. Cree un nuevo archivo en el directorio **./resources/views** denominado `newevent.blade.php` y agregue el siguiente código.

    :::code language="php" source="../demo/graph-tutorial/resources/views/newevent.blade.php" id="NewEventFormSnippet":::

## <a name="add-controller-actions"></a>Agregar acciones de controlador

1. Abra **./app/Http/Controllers/CalendarController.php** y agregue la siguiente función para representar el formulario.

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/CalendarController.php" id="getNewEventFormSnippet":::

1. Agregue la siguiente función para recibir los datos del formulario cuando el usuario lo envíe y cree un nuevo evento en el calendario del usuario.

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/CalendarController.php" id="createNewEventSnippet":::

    Ten en cuenta lo que hace este código.

    - Convierte la entrada del campo de asistentes en una matriz de objetos de [asistentes de](https://docs.microsoft.com/graph/api/resources/attendee?view=graph-rest-1.0) Graph.
    - Genera un evento a [partir de](https://docs.microsoft.com/graph/api/resources/event?view=graph-rest-1.0) la entrada del formulario.
    - Envía un POST al punto de conexión y, a `/me/events` continuación, redirige de nuevo a la vista de calendario.

1. Guarde todos los cambios y reinicie el servidor. Use el **botón Nuevo** evento para navegar al nuevo formulario de evento.

1. Rellene los valores del formulario. Use una fecha de inicio de la semana actual. Seleccione **Crear**.

    ![Captura de pantalla del nuevo formulario de evento](images/create-event-01.png)

1. Cuando la aplicación redirige a la vista de calendario, comprueba que el nuevo evento está presente en los resultados.
