<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="f7cde-101">En este ejercicio, creará un nuevo registro de aplicaciones Web de Azure AD mediante el portal de registro de aplicaciones (ARP).</span><span class="sxs-lookup"><span data-stu-id="f7cde-101">In this exercise, you will create a new Azure AD web application registration using the Application Registry Portal (ARP).</span></span>

1. <span data-ttu-id="f7cde-102">Abra un explorador y vaya al [portal de registro de aplicaciones](https://apps.dev.microsoft.com).</span><span class="sxs-lookup"><span data-stu-id="f7cde-102">Open a browser and navigate to the [Application Registration Portal](https://apps.dev.microsoft.com).</span></span> <span data-ttu-id="f7cde-103">Inicie sesión con una **cuenta personal** (también conocido como Microsoft Account) o una **cuenta profesional o educativa**.</span><span class="sxs-lookup"><span data-stu-id="f7cde-103">Login using a **personal account** (aka: Microsoft Account) or **Work or School Account**.</span></span>

1. <span data-ttu-id="f7cde-104">Seleccione **Agregar una aplicación** en la parte superior de la página.</span><span class="sxs-lookup"><span data-stu-id="f7cde-104">Select **Add an app** at the top of the page.</span></span>

    > [!NOTE]
    > <span data-ttu-id="f7cde-105">Si ve más de un botón **Agregar una aplicación** en la página, seleccione el que corresponda a la lista de **aplicaciones convergentes** .</span><span class="sxs-lookup"><span data-stu-id="f7cde-105">If you see more than one **Add an app** button on the page, select the one that corresponds to the **Converged apps** list.</span></span>

1. <span data-ttu-id="f7cde-106">En la página **registrar la aplicación** , establezca el tutorial **nombre** de la aplicación en **php Graph** y seleccione **crear**.</span><span class="sxs-lookup"><span data-stu-id="f7cde-106">On the **Register your application** page, set the **Application Name** to **PHP Graph Tutorial** and select **Create**.</span></span>

    ![Captura de pantalla de la creación de una nueva aplicación en el sitio web del portal de registro de aplicaciones](./images/arp-create-app-01.png)

1. <span data-ttu-id="f7cde-108">En la página **registro del tutorial de php Graph** , en la sección **propiedades** , copie el identificador de la **aplicación** ya que lo necesitará más adelante.</span><span class="sxs-lookup"><span data-stu-id="f7cde-108">On the **PHP Graph Tutorial Registration** page, under the **Properties** section, copy the **Application Id** as you will need it later.</span></span>

    ![Captura de pantalla del identificador de la aplicación recién creada](./images/arp-create-app-02.png)

1. <span data-ttu-id="f7cde-110">Desplácese hacia abajo hasta la sección secretos de la **aplicación** .</span><span class="sxs-lookup"><span data-stu-id="f7cde-110">Scroll down to the **Application Secrets** section.</span></span>

    1. <span data-ttu-id="f7cde-111">Seleccione **generar nueva contraseña**.</span><span class="sxs-lookup"><span data-stu-id="f7cde-111">Select **Generate New Password**.</span></span>
    1. <span data-ttu-id="f7cde-112">En el cuadro de diálogo **nueva contraseña generada** , copie el contenido del cuadro tal y como lo necesitará más adelante.</span><span class="sxs-lookup"><span data-stu-id="f7cde-112">In the **New password generated** dialog, copy the contents of the box as you will need it later.</span></span>

        > [!IMPORTANT]
        > <span data-ttu-id="f7cde-113">Esta contraseña no se vuelve a mostrar nunca, por lo que debe asegurarse de que la copia ahora.</span><span class="sxs-lookup"><span data-stu-id="f7cde-113">This password is never shown again, so make sure you copy it now.</span></span>

    ![Captura de pantalla de la contraseña de la aplicación recién creada](./images/arp-create-app-03.png)

1. <span data-ttu-id="f7cde-115">Desplácese hacia abajo hasta la sección **plataformas** .</span><span class="sxs-lookup"><span data-stu-id="f7cde-115">Scroll down to the **Platforms** section.</span></span>

    1. <span data-ttu-id="f7cde-116">Seleccione **Agregar plataforma**.</span><span class="sxs-lookup"><span data-stu-id="f7cde-116">Select **Add Platform**.</span></span>
    1. <span data-ttu-id="f7cde-117">En el cuadro de diálogo **Agregar plataforma** , seleccione **Web**.</span><span class="sxs-lookup"><span data-stu-id="f7cde-117">In the **Add Platform** dialog, select **Web**.</span></span>

        ![Captura de pantalla que crea una plataforma para la aplicación](./images/arp-create-app-04.png)

    1. <span data-ttu-id="f7cde-119">En el cuadro plataforma **Web** , escriba la dirección `http://localhost:8000/callback` URL de las **direcciones URL**de redireccionamiento.</span><span class="sxs-lookup"><span data-stu-id="f7cde-119">In the **Web** platform box, enter the URL `http://localhost:8000/callback` for the **Redirect URLs**.</span></span>

        ![Captura de pantalla de la plataforma web recién agregada para la aplicación](./images/arp-create-app-05.png)

1. <span data-ttu-id="f7cde-121">Desplácese hasta la parte inferior de la página y seleccione **Guardar**.</span><span class="sxs-lookup"><span data-stu-id="f7cde-121">Scroll to the bottom of the page and select **Save**.</span></span>