# <a name="track-changes-for-a-drive"></a>Nachverfolgen von Laufwerksänderungen

Mit dieser Methode kann Ihre App Änderungen nachverfolgen, die im Laufe der Zeit an Laufwerken sowie deren untergeordneten Elementen vorgenommen werden.

Die App ruft zunächst `delta` ohne Parameter auf. Der Dienst startet dann eine Enumeration der Laufwerkshierarchie und gibt Seiten mit Elementen sowie entweder einen `@odata.nextLink` oder einen `@odata.deltaLink` zurück, wie unten beschrieben. Die App sollte die Aufrufe solange mit dem `@odata.nextLink` fortführen, bis kein `@odata.nextLink` mehr zurückgegeben wird oder bis eine Antwort mit einem leeren Satz an Änderungen zurückgegeben wird.

Sobald alle Änderungen empfangen wurden, können Sie sie auf den lokalen Zustand anwenden. Wenn Sie zu einem späteren Zeitpunkt nochmals auf Änderungen prüfen möchten, rufen Sie erneut `delta` auf, mit dem `@odata.deltaLink` aus der vorherigen Antwort.

Gelöschte Elemente werden mit dem [`deleted`Facet](../resources/deleted.md) zurückgegeben. Elemente, für die diese Eigenschaft gesetzt ist, sollten Sie aus Ihrem lokalen Zustand entfernen. 

**Hinweis:** Löschen Sie Ordner nur lokal, wenn sie nach der Synchronisierung aller Änderungen leer sind.

## <a name="prerequisites"></a>Voraussetzungen
Einer der folgenden **Bereiche** ist erforderlich, um diese API auszuführen:

  * Files.Read
  * Files.Readwrite

## <a name="http-request"></a>HTTP-Anforderung
<!-- { "blockType": "ignored" } -->
```http
GET /me/drive/root/delta
GET /drives/{drive-id}/root/delta
GET /groups/{group-id}/drive/root/delta
```

## <a name="optional-query-parameters"></a>Optionale Abfrageparameter
Diese Methode unterstützt die [OData-Abfrageparameter](http://graph.microsoft.io/docs/overview/query_parameters) zur Anpassung der Antwort.

## <a name="request-body"></a>Anforderungstext
Geben Sie für diese Methode keinen Anforderungstext an.

## <a name="response"></a>Antwort
Bei Erfolg gibt diese Methode den Antwortcode `200 OK` und eine Sammlung von Ressourcen des Typs [DriveItem](../resources/driveitem.md) im Antworttext zurück.

Zusätzlich zu der Sammlung von DriveItems enthält die Antwort außerdem eine der folgenden Eigenschaften:

| Name                 | Wert  | Beschreibung                                                                                                                                      |
|:---------------------|:-------|:-------------------------------------------------------------------------------------------------------------------------------------------------|
| **@odata.nextLink**  | url    | Eine URL zum Abrufen der nächsten verfügbaren Seite von Änderungen, sofern weitere Änderungen im aktuellen Satz vorhanden sind                                        |
| **@odata.deltaLink** | url    | Eine URL, die anstelle eines **@odata.nextLink** zurückgegeben wird, sobald alle aktuellen Änderungen zurückgegeben wurden. Sie wird verwendet, um zu einem späteren Zeitpunkt den nächsten Satz von Änderungen zu lesen.  |


## <a name="example-initial-request"></a>Beispiel (ursprüngliche Anforderung)
Hier sehen Sie ein Beispiel für einen Aufruf dieser API zur Ermittlung Ihres lokalen Zustands.

##### <a name="request"></a>Anforderung
Unten ein Beispiel für eine ursprüngliche Anforderung:

<!-- {
  "blockType": "request",
  "name": "get_item_delta"
}-->
```http
GET https://graph.microsoft.com/v1.0/me/drive/root/delta
```

##### <a name="response"></a>Antwort
Nachfolgend sehen Sie ein Beispiel der Antwort.

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

Diese Antwort enthält die erste Seite von Änderungen. Die Eigenschaft **@odata.nextLink** gibt an, dass im aktuellen Satz von Elementen weitere Elemente verfügbar sind. Die App sollte nun solange den URL-Wert von **@odata.nextLink** anfordern, bis alle Seiten von Elementen abgerufen wurden.

## <a name="example-last-page-in-a-set"></a>Beispiel (letzte Seite in einem Satz)
Unten sehen Sie ein Beispiel für einen Aufruf dieser API zur Aktualisierung Ihres lokalen Zustands.

##### <a name="request"></a>Anforderung
Hier sehen Sie ein Beispiel für eine Anforderung nach der ursprünglichen Anforderung:

<!-- {
  "blockType": "request",
  "name": "get_item_delta"
}-->
```http
GET https://graph.microsoft.com/v1.0/me/drive/root/delta(token='123123901209310923!23alksjd')
```

##### <a name="response"></a>Antwort
Nachfolgend sehen Sie ein Beispiel der Antwort.

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

Diese Antwort gibt an, dass im Zeitraum zwischen der ursprünglichen Anforderung und dieser Anforderung zur Aktualisierung des lokalen Zustands das Element `folder2` gelöscht wurde und das Element `file.txt` entweder hinzugefügt oder geändert wurde.

Die letzte Seite mit Elementen enthält die Eigenschaft **@odata.deltaLink**. Sie spezifiziert die URL, über die zu einem späteren Zeitpunkt Änderungen abgerufen werden können, die bis dahin an dem jetzt aktuellen Elementsatz vorgenommen wurden.

## <a name="remarks"></a>Bemerkungen

* Der „delta“-Feed zeigt den aktuellen Zustand jedes Elements, nicht jede Änderung. Wenn ein Element beispielsweise zweimal umbenannt wurde, wird es nur einmal angezeigt, mit seinem neuesten Namen.
* Ein Element kann mehrmals in einem „delta“-Feed aufgeführt werden, aus jeweils unterschiedlichen Gründen. Verwenden Sie das letzte Vorkommen in der Auflistung.
* Die Eigenschaft `parentReference` von Elementen enthält keinen Wert für **path**. Der Grund: Wenn ein Ordner umbenannt wird, gibt **delta** keine Nachfolger dieses Ordners zurück. **Bei Verwendung von „delta“ sollten Sie Elemente immer anhand ihrer ID nachverfolgen.**

Es kann passieren, dass der Dienst für ein Token keine Änderungsliste zurückgeben kann. (Das ist beispielsweise der Fall, wenn ein Client nach einer längeren Verbindungsunterbrechung versucht, ein altes Token wiederzuverwenden, oder wenn sich der Zustand eines Servers geändert hat und daher ein neues Token erforderlich ist.) In solchen Fällen gibt der Dienst einen Fehler des Typs `HTTP 410 Gone` zurück. Die Fehlerantwort enthält einen der unten aufgeführten Fehlercodes und einen Header `Location` mit einem neuen „nextLink“, der eine vollständig neue „delta“-Enumeration startet. Vergleichen Sie nach Abschluss der vollständigen Enumeration die zurückgegebenen Elemente mit Ihrem lokalen Zustand, und befolgen Sie diese Anweisungen:

| Fehlertyp                       | Anweisungen                                                                                                                                                                                                                    |
|:---------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `resyncChangesApplyDifferences`  | Ersetzen Sie alle lokalen Elemente durch die Serverversion (einschließlich gelöschter Elemente), wenn Sie sicher sind, dass der Dienst bei der letzten Synchronisierung mit den lokalen Änderungen aktualisiert wurde. Laden Sie alle lokalen Änderungen hoch, die dem Server noch nicht bekannt sind. |
| `resyncChangesUploadDifferences` | Laden Sie alle lokalen Elemente hoch, die der Dienst nicht zurückgegeben hat, und laden Sie alle Dateien hoch, die sich von der Version auf dem Server unterscheiden (wobei Sie beide Kopien beibehalten, wenn Sie nicht sicher sind, welche die aktuelle ist).                                       |


In OneDrive for Business und SharePoint wird `delta` nur für den Ordner `root` unterstützt, für andere Ordner jedoch nicht. Zudem gibt es keine der folgenden „DriveItem“-Eigenschaften zurück:

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
