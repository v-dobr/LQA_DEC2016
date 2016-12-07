# <a name="track-changes-for-a-drive"></a>ドライブの変更履歴を記録する

このメソッドでは、アプリがドライブおよびその子への変更履歴を時間の経過とともに記録できます。

アプリはまず、パラメーターを指定せずに `delta` を呼び出します。サービスはドライブの階層の列挙を開始し、次に説明するように、アイテムのページと `@odata.nextLink` または `@odata.deltaLink` のいずれかを返します。`@odata.nextLink` が返されなくなるまで、または変更の空のセットが応答で返されるまで、アプリは `@odata.nextLink` を使って呼び出しを続けます。

すべての変更を受信したら、それをローカルの状態に適用できます。今後の変更を確認するには、前回の応答の `@odata.deltaLink` を使ってもう一度 `delta` を呼び出します。

削除されたアイテムは [`deleted` ファセット](../resources/deleted.md)付きで返されます。このプロパティ セットを持つアイテムは、ローカル状態から削除する必要があります。 

**注:** すべての変更を同期した後にフォルダーが空の場合にのみ、ローカルでそのフォルダーを削除する必要があります。

## <a name="prerequisites"></a>前提条件
この API を実行するには、以下のいずれかの**スコープ**が必要です。

  * Files.Read
  * Files.Readwrite

## <a name="http-request"></a>HTTP 要求
<!-- { "blockType": "ignored" } -->
```http
GET /me/drive/root/delta
GET /drives/{drive-id}/root/delta
GET /groups/{group-id}/drive/root/delta
```

## <a name="optional-query-parameters"></a>オプションのクエリ パラメーター
このメソッドは、応答をカスタマイズするための [OData クエリ パラメーター](http://graph.microsoft.io/docs/overview/query_parameters)をサポートします。

## <a name="request-body"></a>要求本文
このメソッドには、要求本文を指定しません。

## <a name="response"></a>応答
成功した場合、このメソッドは応答本文で `200 OK` 応答コードと、[DriveItem](../resources/driveitem.md) リソースのコレクションを返します。

DriveItem のコレクションのほか、応答には次のプロパティのいずれかも含まれます。

| 名前                 | 値  | 説明                                                                                                                                      |
|:---------------------|:-------|:-------------------------------------------------------------------------------------------------------------------------------------------------|
| **@odata.nextLink**  | url    | 現在のセットに追加の変更がある場合に、次の使用可能な変更ページを取得するための URL です。                                        |
| **@odata.deltaLink** | url    | 現在のすべての変更が返された後に、**@odata.nextLink** の代わりに返される URL です。今後の次の一連の変更を読み取るために使用されます。  |


## <a name="example-(initial-request)"></a>例 (最初の要求)
ここでは、ローカルの状態を確立するために、この API を呼び出す方法の例です。

##### <a name="request"></a>要求
以下は最初の要求の例です。

<!-- {
  "blockType": "request",
  "name": "get_item_delta"
}-->
```http
GET https://graph.microsoft.com/v1.0/me/drive/root/delta
```

##### <a name="response"></a>応答
以下は、応答の例です。

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

この応答には変更の最初のページが含まれており、**@odata.nextLink** プロパティは、現在のアイテムのセットで使用可能なアイテムがさらにあることを示しています。アプリは、アイテムのすべてのページが取得されるまで、**@odata.nextLink** の URL の値を要求し続ける必要があります。

## <a name="example-(last-page-in-a-set)"></a>例 (セットの最後のページ)
以下に、ローカルの状態を更新するためにこの API を呼び出す方法の例を示します。

##### <a name="request"></a>要求
以下は最初の要求後の要求の例です。

<!-- {
  "blockType": "request",
  "name": "get_item_delta"
}-->
```http
GET https://graph.microsoft.com/v1.0/me/drive/root/delta(token='123123901209310923!23alksjd')
```

##### <a name="response"></a>応答
以下は、応答の例です。

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

この応答は、`folder2` という名前のアイテムが削除され、アイテム `file.txt` は最初の要求とローカルの状態を更新する今回の要求の間で追加または変更されたことを示しています。

アイテムの最後のページには **@odata.deltaLink** プロパティが含まれます。このプロパティは現在のアイテム セット以降の変更を後で取得するために使用される URL を提供します。

## <a name="remarks"></a>注釈

* 差分フィードは各変更を示すのではなく、各アイテムの最新の状態を示すものです。アイテムの名前が 2 回変更された場合、最新の名前で 1 回だけ表示されます。
* 差分フィードには、さまざまな理由から同じアイテムが複数回表示される場合があります。その場合は最後に出現したものを使用する必要があります。
* アイテムの `parentReference` プロパティには**パス**の値は含まれません。これは、フォルダー名を変更しても**デルタ**からそのフォルダーの子孫が返されることはないためです。**差分を使用する場合、アイテムは必ず ID で追跡する必要があります**。

指定したトークンの変更リストを、サービスが提供できない場合があります (長時間にわたって切断された後に、クライアントが古いトークンを再利用する場合や、サーバーの状態が変更され、新しいトークンが必要な場合など)。このような場合、サービスは次のエラー コードのいずれかを含むエラー応答で `HTTP 410 Gone` エラーを返し、また、新しい差分の列挙を最初から開始する新しい nextLink を含む `Location` ヘッダーを返します。完全な列挙処理が終了したら、返されたアイテムとローカルの状態を比較して、次の指示に従います。

| エラーの種類                       | 手順                                                                                                                                                                                                                    |
|:---------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `resyncChangesApplyDifferences`  | 最後に同期したときに、サービスがローカルの変更に対して最新の状態であったことが確実な場合、すべてのローカル アイテムをサーバーのバージョンと置き換えます (削除を含む)。サーバーが把握していないすべてのローカル変更をアップロードします。 |
| `resyncChangesUploadDifferences` | サービスが返さないすべてのローカル アイテムをアップロードして、サーバーのバージョンと異なるすべてのファイルをアップロードします (どちらがより最新の状態であるかわからない場合は、両方のコピーを保持する)。                                       |


OneDrive for Business および SharePoint では、`delta` は `root` フォルダーでのみサポートされ、他のフォルダーではサポートされません。また、DriveItem の次のプロパティも返しません。

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
