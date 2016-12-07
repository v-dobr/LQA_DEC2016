# <a name="track-changes-for-a-drive"></a>Suivre les modifications sur un lecteur

Cette méthode permet à votre application de suivre les modifications apportées à un lecteur et à ses enfants au fil du temps.

Votre application commence par appeler le service `delta` sans aucun paramètre. Le service commence à énumérer la hiérarchie du lecteur, renvoyant les pages des éléments et un `@odata.nextLink`, ou un `@odata.deltaLink`, comme décrit ci-dessous. Votre application doit continuer à appeler avec `@odata.nextLink` jusqu’à ce que plus aucun `@odata.nextLink` ne soit renvoyé, ou jusqu’à ce que vous receviez une réponse avec un ensemble de modifications vide.

Une fois que vous avez reçu toutes les modifications, vous pouvez les appliquer à votre état local. Pour vérifier les modifications apportées à l’avenir, appelez une nouvelle fois `delta` à l’aide de `@odata.deltaLink` reçu dans la réponse précédente.

Les éléments supprimés sont renvoyés avec la [facette `deleted`](../resources/deleted.md). Les éléments qui possèdent ce jeu de propriétés doivent être supprimés de votre état local. 

**Remarque :** supprimez uniquement un dossier local s’il est vide une fois toutes les modifications synchronisées.

## <a name="prerequisites"></a>Conditions préalables
L’une des **étendues** suivantes est nécessaire pour exécuter cette API :

  * Files.Read
  * Files.Readwrite

## <a name="http-request"></a>Requête HTTP
<!-- { "blockType": "ignored" } -->
```http
GET /me/drive/root/delta
GET /drives/{drive-id}/root/delta
GET /groups/{group-id}/drive/root/delta
```

## <a name="optional-query-parameters"></a>Paramètres facultatifs de la requête
Cette méthode prend en charge les [paramètres de requête OData](http://graph.microsoft.io/docs/overview/query_parameters) pour vous aider à personnaliser la réponse.

## <a name="request-body"></a>Corps de la demande
N’indiquez pas le corps de la demande pour cette méthode.

## <a name="response"></a>Réponse
En cas de réussite, cette méthode renvoie un code de réponse `200 OK` et une collection de ressources [DriveItem](../resources/driveitem.md) dans le corps de la réponse.

Outre la collection de DriveItems, la réponse comprend également l’une des propriétés suivantes :

| Nom                 | Valeur  | Description                                                                                                                                      |
|:---------------------|:-------|:-------------------------------------------------------------------------------------------------------------------------------------------------|
| **@odata.nextLink**  | url    | URL destinée à récupérer la page de modifications disponible suivante, si d’autres modifications ont été apportées dans la série en cours.                                        |
| **@odata.deltaLink** | url    | URL renvoyée à la place de **@odata.nextLink** une fois que toutes les modifications en cours ont été renvoyées. Permet de lire la prochaine série de modifications apportées à l’avenir.  |


## <a name="example-(initial-request)"></a>Exemple (requête initiale)
Voici comment appeler cette API pour établir votre état local.

##### <a name="request"></a>Demande
Voici un exemple de la requête initiale.

<!-- {
  "blockType": "request",
  "name": "get_item_delta"
}-->
```http
GET https://graph.microsoft.com/v1.0/me/drive/root/delta
```

##### <a name="response"></a>Réponse
Voici un exemple de réponse.

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

Cette réponse comprend la première page de modifications. La propriété **@odata.nextLink** indique qu’il y a davantage d’éléments disponibles dans l’ensemble des éléments en cours. Votre application doit continuer à demander la valeur de l’URL de **@odata.nextLink** jusqu’à ce vous ayez récupéré toutes les pages d’éléments.

## <a name="example-(last-page-in-a-set)"></a>Exemple (dernière page d’une série)
Voici comment appeler cette API pour mettre à jour votre état local.

##### <a name="request"></a>Demande
Voici un exemple de requête après la requête initiale.

<!-- {
  "blockType": "request",
  "name": "get_item_delta"
}-->
```http
GET https://graph.microsoft.com/v1.0/me/drive/root/delta(token='123123901209310923!23alksjd')
```

##### <a name="response"></a>Réponse
Voici un exemple de réponse.

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

Cette réponse indique que l’élément nommé `folder2` a été supprimé et que l’élément `file.txt` a été ajouté ou modifié entre la requête initiale et cette requête afin de mettre à jour l’état local.

La dernière page d’éléments comprendra la propriété **@odata.deltaLink** qui fournit l’URL pouvant servir ultérieurement à récupérer les modifications à partir du jeu d’éléments en cours.

## <a name="remarks"></a>Remarques

* Le flux delta affiche le dernier état de chaque élément, et non chaque modification. Si un élément a été renommé deux fois, il apparaîtrait une seule fois uniquement, avec son nom le plus récent.
* Le même élément peut s’afficher plusieurs fois dans un flux delta pour diverses raisons. Vous devez utiliser la dernière occurrence consultée.
* La propriété `parentReference` des éléments n’indiquera pas de valeur pour **path**. En effet, le fait de renommer un dossier ne renvoie pas les descendants du dossier renvoyés depuis **delta**. **Lorsque vous utilisez delta, vous devez toujours suivre les éléments par ID**.

Il se peut que le service ne puisse pas fournir une liste des modifications pour un jeton donné (par exemple, si un client tente de réutiliser un ancien jeton après avoir été déconnecté pendant un certain temps, ou si l’état du serveur a été modifié et qu’un nouveau jeton est requis). Dans ces scénarios, le service renverra une erreur `HTTP 410 Gone` avec une réponse d’erreur contenant l’un des codes d’erreur ci-dessous, et un en-tête `Location` contenant un nouveau nextLink qui recommence une énumération delta. Une fois l’énumération complètement terminée, comparez les éléments renvoyés avec votre état local et suivez les instructions suivantes.

| Type d’erreur                       | Instructions                                                                                                                                                                                                                    |
|:---------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `resyncChangesApplyDifferences`  | Remplacez les éléments locaux par la version du serveur (y compris les suppressions) si vous êtes sûr que le service était à jour avec vos modifications locales lors de la dernière synchronisation. Téléchargez les modifications locales que le serveur ignore. |
| `resyncChangesUploadDifferences` | Téléchargez les éléments locaux que le service n’a pas renvoyés, et téléchargez les fichiers qui diffèrent de la version du serveur (tout en conservant les deux copies si vous ne savez pas quelle est la plus récente).                                       |


Dans OneDrive Entreprise et SharePoint, `delta` est uniquement pris en charge dans le dossier `root`, et non dans les autres dossiers. Il ne renverra pas non plus les propriétés DriveItem suivantes :

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
