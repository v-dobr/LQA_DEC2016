# <a name="working-with-files-in-microsoft-graph"></a>Utiliser des fichiers dans Microsoft Graph

Vous pouvez utiliser Microsoft Graph pour créer une application qui accède aux fichiers dans les bibliothèques de documents OneDrive, OneDrive Entreprise et SharePoint. Avec Microsoft Graph, vous pouvez créer différentes expériences avec des fichiers stockés dans Office 365, en stockant simplement les documents des utilisateurs ou en partageant des fichiers complexes.

Microsoft Graph propose deux types de ressources pour l’utilisation des fichiers :

* [Drive](drive.md) : représente un conteneur logique de fichiers, tel qu’une bibliothèque de documents ou le lecteur OneDrive d’un utilisateur.
* [DriveItem](driveitem.md) : représente un élément présent sur un lecteur, comme un document, une photo, une vidéo ou un dossier.

La plupart des interactions avec les fichiers s’effectue en interagissant avec des ressources **DriveItem**. Voici un exemple d’une ressource DriveItem :

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

Les ressources **Drive** et **DriveItem** affichent les données de trois manières différentes :

* Les _Propriétés_ (comme **id** et **nom**) exposent des valeurs simples (chaînes, nombres, valeurs booléennes).
* Les _Facettes_ (comme **fichier** et **photo**) exposent des valeurs complexes. La présence de facettes **fichier** ou **dossier** indique les comportements et les propriétés d’un objet **DriveItem**.
* Les _Références_ (comme **enfants** et **miniatures**) renvoient aux collections d’autres ressources.

## <a name="commonly-accessed-resources"></a>Ressources fréquemment employées

La plupart des requêtes d’API qui demandent des interactions entre les fichiers utilisent l’une de ces ressources de base pour accéder à un objet **Drive** ou **DriveItem**.

| Chemin    | Ressource    |
|---------|-------------|
| `/me/drive` | Lecteur OneDrive d’un utilisateur |
| `/me/drives` | Énumère les ressources OneDrive mises à la disposition de l’utilisateur. |
| `/drives/{drive-id}` | Accède à un objet **Drive** spécifique via l’ID du lecteur. |
| `/drives/{drive-id}/root/children` | Énumère les ressources **DriveItem** de la racine d’un objet **Drive** spécifique. |
| `/me/drive/items/{item-id}` | Accède à un objet **DriveItem** dans le lecteur OneDrive de l’utilisateur via son ID unique. |
| `/me/drive/special/{special-id}` | Accède à un dossier spécial (nommé) dans le lecteur OneDrive de l’utilisateur via son nom connu. |
| `/users/{user-id}/drive` | Accède au lecteur OneDrive d’un autre utilisateur via l’ID unique de l’utilisateur. |
| `/groups/{group-id}/drive` | Accède à la bibliothèque de documents par défaut d’un groupe via l’ID unique du groupe. |
| `/shares/{share-id}` | Accède à un objet **DriveItem** via son ID partagé (**sharedId**) ou son URL de partage. |

En plus d’utiliser l’ID unique de l’objet **DriveItem** pour y accéder dans l’objet **Drive**, votre application peut également accéder à l’objet **DriveItem** via son chemin d’accès relatif à partir d’une ressource connue. Pour utiliser un chemin d’accès, le signe deux-points (`:`) permet d’ignorer le chemin d’accès relatif. Ce tableau fournit un exemple qui montre comment utiliser le caractère deux-points pour accéder à un élément via son chemin d’accès.

| Chemin | Ressource |
|---|---|
| `/me/drive/root:/path/to/file` | Accède à un objet **DriveItem** via le chemin d’accès au dossier racine du lecteur OneDrive de l’utilisateur. |
| `/me/drive/items/{item-id}:/path/to/file` | Accède à un objet **DriveItem** via le chemin d’accès relatif à un autre élément (un objet **DriveItem** avec une facette **dossier**). |
| `/me/drive/root:/path/to/folder:/children` | Répertorie les enfants d’un objet **DriveItem** via le chemin d’accès relatif à la racine du lecteur OneDrive de l’utilisateur. |
| `/me/drive/items/{item-id}:/path/to/folder:/children` | Répertorie les enfants d’un objet **DriveItem** via le chemin d’accès relatif à un autre élément. |

## <a name="drive-resource"></a>Ressource Drive

La [ressource Drive](drive.md) est l’objet de niveau supérieur présent dans le lecteur OneDrive d’un utilisateur ou dans une bibliothèque de documents SharePoint. Presque toutes les opérations de fichiers commencent par accéder à une ressource spécifique du lecteur.

Une ressource de lecteur peut être accessible via l’ID unique du lecteur ou via le lecteur par défaut d’un [utilisateur](user.md), d’un [groupe](group.md) ou d’une organisation. 

## <a name="driveitem-resource"></a>Ressource DriveItem

Les objets [DriveItem](driveitem.md) sont les objets présents dans le système de fichiers d’un lecteur. Ils sont accessibles via leur **ID** à l’aide de la syntaxe `/items/{item-id}`, ou via le chemin d’accès du système de fichiers à l’aide de la syntaxe `/root:/path/to/item/`.

Les objets DriveItem ont des _facettes_ qui fournissent des informations sur l’identité de l’élément et ses fonctions.

Les objets DriveItem avec une facette **dossier** servent de conteneurs d’éléments et ont une référence **enfants**, qui renvoie à une collection d’éléments situés au-dessous du dossier.

## <a name="shared-folders-and-remote-items"></a>Dossiers partagés et éléments distants

Les utilisateurs personnels OneDrive peuvent ajouter à leur propre lecteur OneDrive un ou plusieurs éléments partagés à partir d’un autre lecteur. Ces éléments partagés apparaissent comme un objet **DriveItem** dans la collection d’objets **enfants** ayant une facette [remoteItem](remoteitem.md).

Pour plus d’informations sur l’utilisation des dossiers partagés et des éléments distants, consultez la rubrique relative aux [éléments distants et dossiers partagés](remoteitem.md).   

## <a name="sharing-and-permissions"></a>Partage et autorisations

L’une des actions les plus courantes des bibliothèques de documents OneDrive et SharePoint consiste à partager du contenu avec d’autres personnes. Microsoft Graph permet à votre application de créer des [liens de partage](../api/item_createLink.md), d’[ajouter des autorisations et d’envoyer des invitations](../api/item_invite.md) à des éléments d’un lecteur.

Microsoft Graph permet également à votre application d’[accéder au contenu partagé](../api/shares_get.md) directement à partir d’un lien de partage.

 
