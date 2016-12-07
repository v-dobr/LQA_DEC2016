# <a name="working-with-files-in-microsoft-graph"></a>Trabajar con archivos en Microsoft Graph

Puede utilizar Microsoft Graph para crear una aplicación que se conecte con archivos a través de las bibliotecas de documentos de OneDrive, OneDrive para la Empresa y SharePoint. Con Microsoft Graph puede crear una variedad de experiencias con archivos almacenados en Office 365, desde simplemente almacenar documentos de usuario a escenarios complejos de uso compartido de archivos.

Microsoft Graph expone dos tipos de recursos para trabajar con archivos:

* [Drive](drive.md) - Representa un contenedor lógico de archivos, como una biblioteca de documentos o OneDrive de un usuario.
* [DriveItem](driveitem.md) - Representa un elemento dentro de una unidad, como un documento, una foto, un vídeo o una carpeta.

La mayor parte de la interacción con archivos se produce mediante la interacción con recursos **DriveItem**. A continuación se muestra un ejemplo de un recurso DriveItem:

```json
{
  "@content.downloadUrl":"http://public-sn3302.files.1drv.com/y2pcT7OaUEExF7EHOlpTjCE55mIUoiX7H3sx1ff6I-nP35XUTBqZlnkh9FJhWb_pf9sZ7LEpEchvDznIbQig0hWBeidpwFkOqSKCwQylisarN6T0ecAeMvantizBUzM2PA1",
  "createdDateTime": "2016-09-16T03:37:04.72Z",
  "cTag": "aYzpENDY0OEYwNkM5MUQ5RDNEITU0OTI3LjI1Ng",
  "eTag": "aRDQ2NDhGMDZDOTFEOUQzRCE1NDkyNy4w",
  "id":"D4648F06C91D9D3D!54927",
  "lastModifiedBy": {
    "user": {
      "displayName": "Daron Spektor",
      "id": "d4648f06c91d9d3d"
    }
  },
  "name":"BritishShorthair.jpg",
  "size":35212,
  "image":{
    "height":398,
    "width":273
  },
  "file": {
    "hashes":{
      "sha1Hash":"wmgPQ6jrSeMX7JP1XmstQEGM2fc="
    }
  }
}
```

Los recursos **Drive** y **DriveItem** exponen los datos de tres maneras diferentes:

* Las _propiedades_ (como **id** y **name**) exponen valores simples (cadenas, números y booleanos).
* Las _facetas_ (como **file** y **photo**) exponen valores complejos. La presencia de facetas **file** o **folder** indica comportamientos y propiedades de un **DriveItem**.
* Las _referencias_ (como **children** y **thumbnails**) indican colecciones de otros recursos.

## <a name="commonly-accessed-resources"></a>Recursos de acceso frecuente

La mayoría de las solicitudes de API para las interacciones de archivo utilizarán uno de estos recursos base para acceder a un recurso **Drive** o **DriveItem**.

| Ruta de acceso    | Recurso    |
|---------|-------------|
| `/me/drive` | OneDrive del usuario |
| `/me/drives` | Enumera los recursos de OneDrive disponibles para el usuario. |
| `/drives/{drive-id}` | Accede a un **Drive** específico mediante el identificador de la unidad. |
| `/drives/{drive-id}/root/children` | Enumera los recursos **DriveItem** en la raíz de un **Drive** específico. |
| `/me/drive/items/{item-id}` | Accede a un **DriveItem** en el OneDrive del usuario mediante su identificador único. |
| `/me/drive/special/{special-id}` | Accede a una carpeta con nombre especial en el OneDrive del usuario mediante su nombre conocido. |
| `/users/{user-id}/drive` | Accede al OneDrive de otro usuario mediante el identificador del usuario. |
| `/groups/{group-id}/drive` | Accede a la biblioteca de documentos predeterminada de un grupo mediante el identificador único del grupo. |
| `/shares/{share-id}` | Accede a un **DriveItem** mediante su **sharedId** o su dirección URL compartida. |

Además de dirigirse a un **DriveItem** dentro de un **Drive** mediante el identificador único, su aplicación también puede dirigirse a un **DriveItem** mediante la ruta de acceso relativa de un recurso conocido. Para acceder mediante una ruta de acceso, se utiliza el carácter de los dos puntos (`:`) para salir de la ruta de acceso relativa. Esta tabla proporciona un ejemplo de las diferentes maneras de utilizar el carácter de los dos puntos para dirigirse a un elemento mediante la ruta de acceso.

| Ruta de acceso | Recurso |
|---|---|
| `/me/drive/root:/path/to/file` | Accede a un **DriveItem** mediante la ruta de acceso relativa a la carpeta raíz del OneDrive del usuario. |
| `/me/drive/items/{item-id}:/path/to/file` | Accede a un **DriveItem** mediante la ruta de acceso relativa a otro elemento (un **DriveItem** con una faceta **folder**). |
| `/me/drive/root:/path/to/folder:/children` | Enumera los elementos secundarios de un **DriveItem** mediante la ruta de acceso relativa a la raíz del OneDrive del usuario. |
| `/me/drive/items/{item-id}:/path/to/folder:/children` | Enumera los elementos secundarios de un **DriveItem** mediante la ruta de acceso relativa a otro elemento. |

## <a name="drive-resource"></a>Recurso Drive

El [recurso Drive](drive.md) es el objeto de nivel superior dentro del OneDrive de un usuario o de una biblioteca de documentos de SharePoint. Casi todas las operaciones de archivos comienzan con el direccionamiento a un recurso de unidad específico.

Un recurso Drive puede tratarse mediante el identificador único de la unidad o mediante la unidad predeterminada de un [usuario](user.md), [grupo](group.md) u organización. 

## <a name="driveitem-resource"></a>Recurso DriveItem

Los [DriveItems](driveitem.md) son los objetos que hay en el sistema de archivos de una unidad. Se puede acceder a ellos a través de su **identificador**, mediante la sintaxis `/items/{item-id}`, o a través de su ruta de acceso al sistema de archivos, mediante la sintaxis `/root:/path/to/item/`. 

Los DriveItems tienen _facetas_ que proporcionan datos sobre la identidad y las capacidades del elemento.

Los DriveItems con una faceta **folder** actúan como contenedores de elementos y tienen una referencia **children** que indica una colección de elementos dentro de la carpeta.

## <a name="shared-folders-and-remote-items"></a>Carpetas compartidas y elementos remotos

Los usuarios con cuenta personal de OneDrive pueden agregar uno o más elementos compartidos desde otra unidad a su propio OneDrive. Estos elementos compartidos aparecen como un **DriveItem** en la colección **children** con una faceta [remoteItem](remoteitem.md).

Para obtener más información acerca de cómo trabajar con carpetas compartidas y objetos remotos, consulte [Elementos remotos y carpetas compartidas](remoteitem.md).   

## <a name="sharing-and-permissions"></a>Uso compartido y permisos

Una de las acciones más comunes en las bibliotecas de documentos de OneDrive y SharePoint es compartir contenido con otras personas. Microsoft Graph permite que su aplicación pueda crear [vínculos para compartir](../api/item_createLink.md), [agregar permisos y enviar invitaciones](../api/item_invite.md) a los elementos de una unidad.

Microsoft Graph también permite que su aplicación pueda [acceder a contenido compartido](../api/shares_get.md) directamente desde un vínculo para compartir.

 
