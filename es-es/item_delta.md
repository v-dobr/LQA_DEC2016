# <a name="track-changes-for-a-drive"></a>Control de cambios de una unidad

Este método permite que su aplicación realice el control de cambios de una unidad y de sus elementos secundarios con el tiempo.

La aplicación comienza llamando a `delta` sin ningún parámetro. El servicio comienza enumerando la jerarquía de la unidad, devuelve páginas de elementos y también `@odata.nextLink` o `@odata.deltaLink`, tal y como se describe a continuación. La aplicación debe continuar la llamada con `@odata.nextLink` hasta que ya no se devuelva `@odata.nextLink` o hasta que la respuesta contenga un conjunto de cambios vacío.

Una vez que termine de recibir todos los cambios, puede aplicarlos a su estado local. Para comprobar si hay cambios en el futuro, llame a `delta` con el `@odata.deltaLink` de la respuesta anterior.

Los elementos eliminados se devuelven con la [`deleted`faceta](../resources/deleted.md). Los elementos con este conjunto de propiedades deben ser eliminados de su estado local. 

**Nota:** solo debe eliminar una carpeta de forma local si está vacía después de sincronizar todos los cambios.

## <a name="prerequisites"></a>Requisitos previos
Se requiere uno de los siguientes **ámbitos** para ejecutar esta API:

  * Files.Read
  * Files.Readwrite

## <a name="http-request"></a>Solicitud HTTP
<!-- { "blockType": "ignored" } -->
```http
GET /me/drive/root/delta
GET /drives/{drive-id}/root/delta
GET /groups/{group-id}/drive/root/delta
```

## <a name="optional-query-parameters"></a>Parámetros de consulta opcionales
Este método admite los [parámetros de consulta de OData](http://graph.microsoft.io/docs/overview/query_parameters) a modo de ayuda para personalizar la respuesta.

## <a name="request-body"></a>Cuerpo de solicitud
No proporcione un cuerpo de solicitud para este método.

## <a name="response"></a>Respuesta
Si se ejecuta correctamente, este método devuelve un código de respuesta `200 OK` y una colección de recursos [DriveItem](../resources/driveitem.md) en el cuerpo de la respuesta.

Además de la colección de DriveItems, la respuesta también incluirá una de las siguientes propiedades:

| Nombre                 | Valor  | Descripción                                                                                                                                      |
|:---------------------|:-------|:-------------------------------------------------------------------------------------------------------------------------------------------------|
| **@odata.nextLink**  | url    | Una dirección URL que recupera la siguiente página de cambios disponible, si hay cambios adicionales en el conjunto actual.                                        |
| **@odata.deltaLink** | url    | Una dirección URL que se devuelve en lugar de **@odata.nextLink** cuando se han devuelto todos los cambios actuales. Se utiliza para leer el siguiente conjunto de cambios en el futuro.  |


## <a name="example-initial-request"></a>Ejemplo (solicitud inicial)
Aquí tiene un ejemplo de cómo llamar a esta API para establecer su estado local.

##### <a name="request"></a>Solicitud
Aquí tiene un ejemplo de la solicitud inicial.

<!-- {
  "blockType": "request",
  "name": "get_item_delta"
}-->
```http
GET https://graph.microsoft.com/v1.0/me/drive/root/delta
```

##### <a name="response"></a>Respuesta
Aquí tiene un ejemplo de la respuesta.

<!-- {
  "blockType": "response",
  "truncated": true,
  "@odata.type": "microsoft.graph.driveItem",
  "isCollection": true
} -->
```http
HTTP/1.1 200 OK
Content-type: application/json

{
    "value": [
        {
            "id": "0123456789abc",
            "name": "folder2",
            "folder": { }
        },
        {
            "id": "123010204abac",
            "name": "file.txt",
            "file": { }
        },
        {
            "id": "2353010204ddgg",
            "name": "file5.txt",
            "deleted": { }
        }
    ],
    "@odata.nextLink": "https://graph.microsoft.com/v1.0/me/drive/delta(token=1230919asd190410jlka)"
}
```

Esta respuesta incluye la primera página de cambios y la propiedad **@odata.nextLink** indica que hay más elementos disponibles en el conjunto actual de elementos. Su aplicación debe continuar solicitando el valor de dirección URL **@odata.nextLink** hasta que se recuperen todas las páginas de elementos.

## <a name="example-last-page-in-a-set"></a>Ejemplo (última página de un conjunto)
Aquí tiene un ejemplo de cómo llamar a esta API para actualizar su estado local.

##### <a name="request"></a>Solicitud
Aquí tiene un ejemplo después de la solicitud inicial.

<!-- {
  "blockType": "request",
  "name": "get_item_delta"
}-->
```http
GET https://graph.microsoft.com/v1.0/me/drive/root/delta(token='123123901209310923!23alksjd')
```

##### <a name="response"></a>Respuesta
Aquí tiene un ejemplo de la respuesta.

<!-- {
  "blockType": "response",
  "truncated": true,
  "@odata.type": "microsoft.graph.driveItem",
  "isCollection": true
} -->
```http
HTTP/1.1 200 OK
Content-type: application/json

{
    "value": [
        {
            "id": "0123456789abc",
            "name": "folder2",
            "folder": { },
            "deleted": { }
        },
        {
            "id": "123010204abac",
            "name": "file.txt",
            "file": { }
        }
    ],
    "@odata.deltaLink": "https://graph.microsoft.com/v1.0/me/drive/root/delta?(token='1230919asd190410jlka')"
}
```

Esta respuesta indica que el elemento llamado `folder2` se ha eliminado y el elemento `file.txt` se ha agregado o se ha modificado entre la solicitud inicial y esta solicitud para actualizar el estado local.

La última página de elementos incluirá la propiedad **@odata.deltaLink**, que proporciona la dirección URL que puede utilizarse posteriormente para recuperar los cambios desde el conjunto actual de elementos.

## <a name="remarks"></a>Comentarios

* La fuente delta muestra el estado más reciente de cada elemento, no cada cambio. Si se ha cambiado el nombre de un elemento dos veces, solo aparecerá una vez con el nombre más reciente.
* El mismo elemento puede aparecer más de una vez en una fuente delta por diversas razones. Debe utilizar la última repetición que vea.
* La propiedad `parentReference` de los elementos no incluirá un valor de **ruta de acceso**. Esto ocurre porque el cambio de nombre de una carpeta no devuelve los descendientes de la carpeta desde **delta**. **Cuando utilice delta, debe hacer siempre un seguimiento de los elementos mediante id**.

En algunos casos, puede que el servicio no pueda proporcionar una lista de cambios de un token determinado (por ejemplo, si un cliente intenta reutilizar un token anterior después de haber estado desconectado durante mucho tiempo o si ha cambiado el estado del servidor y se requiere un nuevo token). En estos casos, el servicio devolverá un error `HTTP 410 Gone` con una respuesta de error que contiene uno de los códigos de error siguientes y un encabezado `Location` con un nuevo nextLink que comienza una nueva enumeración delta desde cero. Cuando finalice la enumeración completa, compare los elementos devueltos con su estado local y siga estas instrucciones.

| Tipo de error                       | Instrucciones                                                                                                                                                                                                                    |
|:---------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `resyncChangesApplyDifferences`  | Reemplace cualquier elemento local con la versión del servidor (incluyendo eliminaciones) si está seguro de que el servicio estaba actualizado con sus cambios locales cuando lo sincronizó por última vez. Cargue cualquier cambio local que no conozca el servidor. |
| `resyncChangesUploadDifferences` | Cargue cualquier elemento local que el servicio no devolvió y cargue cualquier archivo que difiera de la versión del servidor (mantenga ambas copias si no está seguro de cuál está más actualizada).                                       |


En OneDrive para la Empresa y SharePoint, solo se admite `delta` en la carpeta `root`, no en otras carpetas. Tampoco devolverá las siguientes propiedades de DriveItem:

* **createdBy**
* **cTag**
* **eTag**
* **fileSystemInfo**
* **lastModifiedBy**
* **parentReference**
* **size**


<!-- {
  "type": "#page.annotation",
  "description": "Get item delta",
  "keywords": "",
  "section": "documentation",
  "tocPath": ""
}-->
