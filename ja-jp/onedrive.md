# <a name="working-with-files-in-microsoft-graph"></a>Microsoft Graph でのファイルの作業

Microsoft Graph を使用して、OneDrive、OneDrive for Business、および SharePoint のドキュメント ライブラリに配置されるファイルに接続するアプリケーションを作成できます。Microsoft Graph を使用することで、ユーザーのドキュメントを単に格納することから、複雑なファイル共有の複雑なシナリオまで、Office 365 に格納されるファイルに関するさまざまなエクスペリエンスを構築できます。

Microsoft Graph では、ファイルを操作するための 2 種類のリソースが公開されています。

* [Drive](drive.md) - ドキュメント ライブラリやユーザーの OneDrive など、ファイルの論理コンテナーを表します。
* [DriveItem](driveitem.md) -ドキュメント、写真、ビデオ、フォルダーなど、ドライブ内のアイテムを表します。

ファイル間での相互作用のほとんどは、**DriveItem** リソース間での相互作用によって発生します。次に、DriveItem リソースの例を示します。

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

**Drive** リソースと **DriveItem** リソースでは、次の異なる 3 つの方法でデータを公開します。

* _プロパティ_ (**ID** や**名前**など) は単純な値 (文字列、数値、ブール値) を公開します。
* _ファセット_ (**ファイル**や**写真**など) は複雑な値を公開します。**ファイル**または**フォルダー**のファセットの有無は、**DriveItem** の動作およびプロパティを示します。
* _参照_ (**子**と**サムネイル**など) はその他のリソースのコレクションを指します。

## <a name="commonly-accessed-resources"></a>一般的にアクセスされるリソース

ファイルの相互作用に関するほとんどの API 要求では、**Drive** または **DriveItem** にアクセスするために以下の基本リソースのいずれかを使用します。

| パス    | リソース    |
|---------|-------------|
| `/me/drive` | ユーザーの OneDrive。 |
| `/me/drives` | そのユーザーに使用できる OneDrive リソースを列挙します。 |
| `/drives/{drive-id}` | ドライブの ID を使用して特定の **Drive** にアクセスします。 |
| `/drives/{drive-id}/root/children` | 特定の **Drive** のルートにある **DriveItem** リソースを列挙します。 |
| `/me/drive/items/{item-id}` | 一意の ID を使用してユーザーの OneDrive にある **DriveItem** にアクセスします。 |
| `/me/drive/special/{special-id}` | 既知の名前を使用してユーザーの OneDrive にある特別な (名前付き) フォルダーにアクセスします。 |
| `/users/{user-id}/drive` | 別のユーザーの一意の ID を使用してそのユーザーの OneDrive にアクセスします。 |
| `/groups/{group-id}/drive` | グループの一意の ID を使用してグループの既定のドキュメント ライブラリにアクセスします。 |
| `/shares/{share-id}` | **sharedId** や共有 URL を使用して **DriveItem** にアクセスします。 |

一意の ID を使用して **Drive** 内の **DriveItem** にアドレス指定することに加え、既知のリソースからの相対パスを使用することによってアプリで **DriveItem** にアドレス指定することもできます。パスを使用してアドレス指定するには、コロン (`:`) を使用して相対パスをエスケープします。次の表に、コロンを使用してパスでアイテムをアドレス指定するいくつかの方法を示します。

| パス | リソース |
|---|---|
| `/me/drive/root:/path/to/file` | ユーザーの OneDrive ルート フォルダーへの相対パスを使用して **DriveItem** にアクセスします。 |
| `/me/drive/items/{item-id}:/path/to/file` | 別のアイテム (**folder** ファセットを持つ **DriveItem**) への相対パスを使用して **DriveItem** にアクセスします。 |
| `/me/drive/root:/path/to/folder:/children` | ユーザーの OneDrive のルートへの相対パスを使用して **DriveItem** の子を一覧表示します。 |
| `/me/drive/items/{item-id}:/path/to/folder:/children` | 別のアイテムへの相対パスを使用して **DriveItem** の子を一覧表示します。 |

## <a name="drive-resource"></a>ドライブ リソース

[ドライブ リソース](drive.md)は、ユーザーの OneDrive または SharePoint ドキュメント ライブラリ内の最上位のオブジェクトです。ほぼすべてのファイル操作は、特定のドライブ リソースをアドレス指定することによって開始されます。

ドライブの一意の ID または [User](user.md)、[Group](group.md)、組織の既定のドライブを使用して、ドライブ リソースをアドレス指定できます。 

## <a name="driveitem-resource"></a>DriveItem リソース

[DriveItem](driveitem.md) は、ドライブのファイル システム内のオブジェクトです。`/items/{item-id}` 構文で **id** を使用して、または `/root:/path/to/item/` 構文でファイル システム パスを使用して、それらにアクセスできます。

DriveItem には、アイテムの id および機能に関するデータを提供する_ファセット_が存在します。

**folder** ファセットを持つ DriveItem は、アイテムのコンテナーとして機能し、フォルダーの下のアイテムのコレクションを指す**子**参照を持ちます。

## <a name="shared-folders-and-remote-items"></a>共有フォルダーおよびリモート アイテム

OneDrive 個人ユーザーは、別のドライブから自分の OneDrive に 1 つ以上の共有アイテムを追加できます。これらの共有のアイテムは、[remoteItem](remoteitem.md) ファセットを持つ**子**コレクションの **DriveItem** として表示されます。

共有フォルダーとリモート アイテムの操作方法の詳細については、「[リモート アイテムおよび共有フォルダー](remoteitem.md)」を参照してください。   

## <a name="sharing-and-permissions"></a>共有とアクセス許可

OneDrive と SharePoint のドキュメント ライブラリの最も一般的な操作の 1 つは、他のユーザーとコンテンツを共有することです。Microsoft Graph を使用することによって、アプリで[共有リンク](../api/item_createLink.md)を作成し、[アクセス許可を追加してドライブ内のアイテムに招待状を送信](../api/item_invite.md)することができます。

Microsoft Graph では、アプリで共有リンクから[共有コンテンツに直接アクセス](../api/shares_get.md)することもできます。

 
