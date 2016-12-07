# <a name="working-with-files-in-microsoft-graph"></a>Arbeiten mit Dateien in Microsoft Graph

Mithilfe von Microsoft Graph können Sie eine App erstellen, die eine Verbindung mit Dateien in OneDrive-, OneDrive for Business- und SharePoint-Dokumentbibliotheken herstellt. Mit Microsoft Graph können Sie verschiedene Möglichkeiten zum Umgang mit in Office 365 gespeicherten Dateien einrichten, angefangen vom einfachen Speichern von Dokumenten bis hin zu komplexen Dateifreigabeszenarien.

Microsoft Graph legt zwei Ressourcentypen für die Arbeit mit Dateien Offen:

* [Drive](drive.md): Stellt einen logischen Container von Dateien wie z. B. eine Dokumentbibliothek oder OneDrive eines Benutzers dar.
* [DriveItem](driveitem.md): Stellt ein Element innerhalb eines Laufwerks dar, wie z. B. ein Dokument, ein Foto, ein Video oder einen Ordner.

Die meisten Interaktionen mit Dateien finden über Interaktionen mit **DriveItem**-Ressourcen statt. Es folgt ein Beispiel für eine DriveItem-Ressource:

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

**Drive**- und **DriveItem**-Ressourcen legen Daten auf drei verschiedene Arten offen:

* _Eigenschaften_ (wie **id** und **name**) legen einfache Werte (Zeichenfolgen, Zahlen, boolesche Werte) offen.
* _Facets_ (wie **file** und **photo**) legen komplexe Werte offen. Vorhandene **file**- oder **folder**-Facets geben Verhaltensweisen und Eigenschaften eines **DriveItem** an.
* _Referenzen_ (wie **children** und **thumbnails**) verweisen auf Sammlungen anderer Ressourcen.

## <a name="commonly-accessed-resources"></a>Häufig verwendete Ressourcen

Die meisten API-Anforderungen für Dateiinteraktionen verwenden eine der folgenden grundlegenden Ressourcen für den Zugriff auf **Drive** oder **DriveItem**.

| Pfad    | Ressource    |
|---------|-------------|
| `/me/drive` | OneDrive eines Benutzers |
| `/me/drives` | Listet die für den Benutzer verfügbaren OneDrive-Ressourcen auf. |
| `/drives/{drive-id}` | Greift auf ein bestimmtes **Drive** über die Laufwerks-ID zu. |
| `/drives/{drive-id}/root/children` | Listet die **DriveItem**-Ressourcen in der Root eines spezifischen **Drive** auf. |
| `/me/drive/items/{item-id}` | Greift auf ein **DriveItem** auf dem OneDrive des Benutzers über die eindeutige ID zu. |
| `/me/drive/special/{special-id}` | Greift auf einen bestimmten (benannten) Ordner auf dem OneDrive des Benutzers über seinen bekannten Namen zu. |
| `/users/{user-id}/drive` | Greift auf das OneDrive eines anderen Benutzers mithilfe der eindeutigen ID des Benutzers zu. |
| `/groups/{group-id}/drive` | Greift auf die Standarddokumentbibliothek für eine Gruppe mithilfe der eindeutigen ID der Gruppe zu. |
| `/shares/{share-id}` | Greift auf ein **DriveItem** über seine **SharedId** oder eine Freigabe-URL zu. |

Zusätzlich zur Adressierung eines **DriveItem** innerhalb eines **Drive** durch eine eindeutige ID kann die App ein **DriveItem** auch über einen relativen Pfad von einer bekannten Ressource adressieren. Zur Adressierung über einen Pfad wird das Doppelpunktzeichen (`:`) für das Escape des relativen Pfads verwendet. Diese Tabelle enthält ein Beispiel für unterschiedliche Methoden für die Verwendung des Doppelpunkts zur Adressierung eines Elements nach Pfad.

| Pfad | Ressource |
|---|---|
| `/me/drive/root:/path/to/file` | Greift auf ein **DriveItem** über den Pfad relativ zum OneDrive-Stammordner des Benutzers zu. |
| `/me/drive/items/{item-id}:/path/to/file` | Greift auf ein **DriveItem** über einen Pfad relativ zu einem anderen Element zu (einem **DriveItem** mit einem **folder**-Facet). |
| `/me/drive/root:/path/to/folder:/children` | Listet die untergeordneten Elemente eines **DriveItem** über einen Pfad relativ zum Stammordner des OneDrive des Benutzers auf. |
| `/me/drive/items/{item-id}:/path/to/folder:/children` | Listet die untergeordneten Elemente eines **DriveItem** über einen Pfad relativ zu einem anderen Element auf. |

## <a name="drive-resource"></a>Drive-Ressource

Die [Drive-Ressource](drive.md) ist das Objekt der obersten Ebene innerhalb des OneDrive eines Benutzers oder einer SharePoint-Dokumentbibliothek. Fast alle Dateivorgänge beginnen damit, dass eine bestimmte Drive-Ressource  adressiert wird.

Eine Drive-Ressource kann entweder von der eindeutigen ID des Laufwerks oder von dem Standardlaufwerk eines [Benutzers](user.md), einer [Gruppe](group.md), oder einer Organisation adressiert werden. 

## <a name="driveitem-resource"></a>DriveItem-Ressource

[DriveItems](driveitem.md) sind die Objekte innerhalb des Laufwerks-Dateisystems. Der Zugriff darauf erfolgt über ihre **id** unter Verwendung der `/items/{item-id}`-Syntax oder den Dateisystempfad unter Verwendung der`/root:/path/to/item/`-Syntax.

DriveItems verfügen _Facets_, die Daten zu den Identitäten und Funktionen der Elemente bereitstellen.

DriveItems mit einem **folder**-Facet fungieren als Container von Elementen und enthalten einen **children**-Verweis, der auf eine Sammlung von Elementen unter dem Ordner zeigt.

## <a name="shared-folders-and-remote-items"></a>Freigegebene Ordner und Remoteelemente

Persönliche OneDrive-Benutzer können ein oder mehrere freigegebene Elemente von einem anderen Laufwerk zu ihrem eigenen OneDrive hinzufügen. Diese freigegebenen Elemente werden als **DriveItem** in der **children**-Sammlung mit einem [remoteItem](remoteitem.md)-Facet angezeigt.

Weitere Informationen zum Arbeiten mit freigegebenen Ordnern und Remoteelementen finden Sie unter [Remoteelemente und freigegebene Ordner](remoteitem.md).   

## <a name="sharing-and-permissions"></a>Freigabe und Berechtigungen

Eine der am häufigsten verwendeten Aktionen für OneDrive und SharePoint-Dokumentbibliotheken besteht im Freigeben von Inhalten für andere Personen. Über Microsoft Graph kann Ihre App [Freigabelinks](../api/item_createLink.md) erstellen, [Genehmigungen hinzufügen und Einladungen an Elemente eines Laufwerks senden](../api/item_invite.md).

Microsoft Graph bietet der App auch eine Möglichkeit, [Zugriff auf freigegebene Inhalte](../api/shares_get.md) direkt über einen Freigabelinks zu erhalten.

 
