<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="cfe77-101">En esta sección agregará la capacidad de crear eventos en el calendario del usuario.</span><span class="sxs-lookup"><span data-stu-id="cfe77-101">In this section you will add the ability to create events on the user's calendar.</span></span>

## <a name="create-new-event-form"></a><span data-ttu-id="cfe77-102">Crear nuevo formulario de evento</span><span class="sxs-lookup"><span data-stu-id="cfe77-102">Create new event form</span></span>

1. <span data-ttu-id="cfe77-103">Cree un nuevo archivo en el directorio **./resources/views** denominado `newevent.blade.php` y agregue el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="cfe77-103">Create a new file in the **./resources/views** directory named `newevent.blade.php` and add the following code.</span></span>

    :::code language="php" source="../demo/graph-tutorial/resources/views/newevent.blade.php" id="NewEventFormSnippet":::

## <a name="add-controller-actions"></a><span data-ttu-id="cfe77-104">Agregar acciones de controlador</span><span class="sxs-lookup"><span data-stu-id="cfe77-104">Add controller actions</span></span>

1. <span data-ttu-id="cfe77-105">Abra **./app/Http/Controllers/CalendarController.php** y agregue la siguiente función para representar el formulario.</span><span class="sxs-lookup"><span data-stu-id="cfe77-105">Open **./app/Http/Controllers/CalendarController.php** and add the following function to render the form.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/CalendarController.php" id="getNewEventFormSnippet":::

1. <span data-ttu-id="cfe77-106">Agregue la siguiente función para recibir los datos del formulario cuando el usuario lo envíe y cree un nuevo evento en el calendario del usuario.</span><span class="sxs-lookup"><span data-stu-id="cfe77-106">Add the following function to receive the form data when the user's submits, and create a new event on the user's calendar.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/CalendarController.php" id="createNewEventSnippet":::

    <span data-ttu-id="cfe77-107">Ten en cuenta lo que hace este código.</span><span class="sxs-lookup"><span data-stu-id="cfe77-107">Consider what this code does.</span></span>

    - <span data-ttu-id="cfe77-108">Convierte la entrada del campo de asistentes en una matriz de objetos de [asistentes de](https://docs.microsoft.com/graph/api/resources/attendee?view=graph-rest-1.0) Graph.</span><span class="sxs-lookup"><span data-stu-id="cfe77-108">It converts the attendees field input to an array of Graph [attendee](https://docs.microsoft.com/graph/api/resources/attendee?view=graph-rest-1.0) objects.</span></span>
    - <span data-ttu-id="cfe77-109">Genera un evento a [partir de](https://docs.microsoft.com/graph/api/resources/event?view=graph-rest-1.0) la entrada del formulario.</span><span class="sxs-lookup"><span data-stu-id="cfe77-109">It builds an [event](https://docs.microsoft.com/graph/api/resources/event?view=graph-rest-1.0) from the form input.</span></span>
    - <span data-ttu-id="cfe77-110">Envía un POST al punto de conexión y, a `/me/events` continuación, redirige de nuevo a la vista de calendario.</span><span class="sxs-lookup"><span data-stu-id="cfe77-110">It sends a POST to the `/me/events` endpoint, then redirects back to the calendar view.</span></span>

1. <span data-ttu-id="cfe77-111">Guarde todos los cambios y reinicie el servidor.</span><span class="sxs-lookup"><span data-stu-id="cfe77-111">Save all of your changes and restart the server.</span></span> <span data-ttu-id="cfe77-112">Use el **botón Nuevo** evento para navegar al nuevo formulario de evento.</span><span class="sxs-lookup"><span data-stu-id="cfe77-112">Use the **New event** button to navigate to the new event form.</span></span>

1. <span data-ttu-id="cfe77-113">Rellene los valores del formulario.</span><span class="sxs-lookup"><span data-stu-id="cfe77-113">Fill in the values on the form.</span></span> <span data-ttu-id="cfe77-114">Use una fecha de inicio de la semana actual.</span><span class="sxs-lookup"><span data-stu-id="cfe77-114">Use a start date from the current week.</span></span> <span data-ttu-id="cfe77-115">Seleccione **Crear**.</span><span class="sxs-lookup"><span data-stu-id="cfe77-115">Select **Create**.</span></span>

    ![Captura de pantalla del nuevo formulario de evento](images/create-event-01.png)

1. <span data-ttu-id="cfe77-117">Cuando la aplicación redirige a la vista de calendario, comprueba que el nuevo evento está presente en los resultados.</span><span class="sxs-lookup"><span data-stu-id="cfe77-117">When the app redirects to the calendar view, verify that your new event is present in the results.</span></span>
