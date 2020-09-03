# <a name="contributing-to-microsoft-graph-training-repositories"></a>Contribuir a repositorios de aprendizaje de Microsoft Graph

Gracias por colaborar en este proyecto. Antes de enviar la solicitud de incorporación deble, asegúrese de tener en cuenta lo siguiente.

## <a name="overview"></a>Información general

El código de este repositorio sirve para tres propósitos:

- Los archivos de Markdown en la carpeta [tutorial](/tutorial) se publican como un tutorial en la página de [tutoriales de Microsoft Graph](https://docs.microsoft.com/graph/tutorials) .
- El proyecto de ejemplo de la carpeta [Demo](/demo) es el origen de un [Inicio rápido de Microsoft Graph](https://developer.microsoft.com/graph/quick-start).**\***
- El proyecto de ejemplo de la carpeta demo también se puede descargar directamente desde GitHub y debe ejecutarse tal cual después de una configuración sencilla.

> **\*** No todos los repositorios de formación están disponibles como inicios rápidos (todavía).

Es importante tener esto en cuenta, ya que los cambios en un punto *pueden* requerir cambios en otro para mantener sincronizados los elementos. Whereever posible, los archivos de Markdown hacen referencia a los archivos de código fuente directamente (mediante una `:::code` Sintaxis personalizada), de modo que la actualización del código en el origen actualice automáticamente el código en Markdown.

## <a name="updating-code"></a>Actualizar código

La `:::code` sintaxis usada en Markdown depende de comentarios específicos en el archivo de código fuente. Estos comentarios son similares a los siguientes:

```csharp
// <MySnippet>
Console.WriteLine("Hello World!");
// </MySnippet>
```

Si actualiza el código entre estos comentarios de "marcador", los archivos de Markdown obtendrán automáticamente esos cambios cuando se publiquen en el sitio de documentación de Microsoft Graph. Si actualiza el código fuera de esos comentarios, es muy probable que necesite actualizar el Markdown correspondiente.

## <a name="adding-features"></a>Adición de características

Si bien se agradece el entusiasmo, no envíe solicitudes de incorporación de inserción para agregar nuevas características al ejemplo. Como este repositorio es principalmente un tutorial de creación de la primera aplicación, el conjunto de características es limitado, por diseño.

## <a name="submitting-pull-requests"></a>Enviar solicitudes de inserción

Envíe todas las solicitudes de incorporación de `master` conmutación a la sucursal.

## <a name="when-do-changes-get-published"></a>¿Cuándo se publican los cambios?

La publicación de actualizaciones en el sitio de [tutoriales de Microsoft Graph](https://docs.microsoft.com/graph/tutorials) no es automática. En primer lugar, los cambios deben promoverse a la `live` rama y, a continuación, los administradores del sitio deben desencadenar una compilación. Normalmente, esto se realiza en función de las necesidades.

## <a name="code-of-conduct"></a>Código de conducta

Este proyecto ha adoptado el [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/) (Código de conducta de código abierto de Microsoft). Para obtener más información, consulte las [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) (Preguntas más frecuentes del código de conducta) o póngase en contacto con [opencode@microsoft.com](mailto:opencode@microsoft.com) con otras preguntas o comentarios.
