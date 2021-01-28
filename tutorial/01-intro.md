<!-- markdownlint-disable MD002 MD041 -->

En este tutorial se explica cómo crear una aplicación web PHP que use la API de Microsoft Graph para recuperar información de calendario para un usuario.

> [!TIP]
> Si prefiere descargar el tutorial completo, puede descargarlo de dos maneras.
>
> - Descargue el [inicio rápido de PHP](https://developer.microsoft.com/graph/quick-start?platform=option-php) para obtener código de trabajo en minutos.
> - Descargue o clone el [repositorio de GitHub.](https://github.com/microsoftgraph/msgraph-training-phpapp)

## <a name="prerequisites"></a>Requisitos previos

Antes de iniciar este tutorial, debe tener [PHP,](http://php.net/downloads.php) [Composer](https://getcomposer.org/)y [Laravel](https://laravel.com/) instalados en el equipo de desarrollo.

También debe tener una cuenta personal de Microsoft con un buzón en Outlook.com o una cuenta de Microsoft para el trabajo o la escuela. Si no tienes una cuenta de Microsoft, hay un par de opciones para obtener una cuenta gratuita:

- Puede registrarse [para obtener una nueva cuenta personal de Microsoft.](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)
- Puede registrarse en el Programa de desarrolladores de [Office 365](https://developer.microsoft.com/office/dev-program) para obtener una suscripción gratuita a Office 365.

> [!NOTE]
> Este tutorial se escribió con PHP versión 8.0.1, Composer versión 2.0.8 y Laravel installer versión 4.1.1. Los pasos de esta guía pueden funcionar con otras versiones, pero no se han probado.

## <a name="feedback"></a>Comentarios

Envíe sus comentarios sobre este tutorial en el [repositorio de GitHub.](https://github.com/microsoftgraph/msgraph-training-phpapp)
