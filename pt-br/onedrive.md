# <a name="working-with-files-in-microsoft-graph"></a>Trabalhando com arquivos no Microsoft Graph

Você pode usar o Microsoft Graph para criar um aplicativo que se conecta a arquivos entre bibliotecas de documentos do OneDrive, OneDrive for Business e SharePoint. Com o Microsoft Graph, é possível criar uma variedade de experiências com arquivos armazenados no Office 365, desde simplesmente armazenar documentos de usuários até cenários de compartilhamento de arquivos complexos.

O Microsoft Graph expõe dois tipos de recursos para trabalhar com arquivos:

* [Drive](drive.md) – Representa um contêiner lógico de arquivos, como uma biblioteca de documentos ou o OneDrive de um usuário.
* [DriveItem](driveitem.md) – Representa um item dentro de uma unidade, como um documento, uma foto, um vídeo ou uma pasta.

Grande parte da interação com os arquivos ocorre por meio da interação com recursos **DriveItem**. Veja a seguir um exemplo de recurso Driveitem:

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

Recursos **Drive** e **DriveItem** expõem dados de três maneiras diferentes:

* _Propriedades_ (como **id** e **name**) expõem valores simples (cadeias de caracteres, números, boolianos).
* _Facetas_ (como **file** e **photo**) expõem valores complexos. A presença de facetas **file** ou **folder** indica comportamentos e propriedades de um **DriveItem**.
* _Referências_ (como **children** e **thumbnails**) apontam para coleções de outros recursos.

## <a name="commonly-accessed-resources"></a>Recursos comumente acessados

A maioria das solicitações de API para interações de arquivo usa um destes recursos base para acessar **Drive** ou **DriveItem**.

| Caminho    | Recurso    |
|---------|-------------|
| `/me/drive` | OneDrive do usuário |
| `/me/drives` | Enumerar recursos do OneDrive disponíveis para o usuário. |
| `/drives/{drive-id}` | Acessar uma determinada **Unidade** com base na ID dessa unidade. |
| `/drives/{drive-id}/root/children` | Enumerar os recursos **DriveItem** na raiz de um determinada **Unidade**. |
| `/me/drive/items/{item-id}` | Acessar um **DriveItem** no OneDrive do usuário com base em sua ID exclusiva. |
| `/me/drive/special/{special-id}` | Acessar uma pasta especial (nomeada) no OneDrive do usuário com base em seu nome conhecido. |
| `/users/{user-id}/drive` | Acessar o OneDrive de outro usuário usando a ID exclusiva desse usuário. |
| `/groups/{group-id}/drive` | Acessar a biblioteca de documentos padrão de um grupo com base na ID exclusiva desse grupo. |
| `/shares/{share-id}` | Acessar **DriveItem** com base em sua **sharedId** ou URL de compartilhamento. |

Além de endereçar **DriveItem** dentro de **Drive** com base na ID exclusiva, seu aplicativo também pode endereçar **DriveItem** com base no caminho relativo de um recurso conhecido. Para endereçar usando um caminho, um caractere de dois pontos (`:`) é usado para escapar o caminho relativo. Essa tabela fornece um exemplo de diferentes maneiras de usar o caractere de dois-pontos para endereçar um item com base no caminho.

| Caminho | Recurso |
|---|---|
| `/me/drive/root:/path/to/file` | Acessar **DriveItem** com base no caminho relativo para a pasta raiz do OneDrive do usuário. |
| `/me/drive/items/{item-id}:/path/to/file` | Acessar **DriveItem** com base no caminho relativo para outro item (um **DriveItem** com uma faceta **folder**). |
| `/me/drive/root:/path/to/folder:/children` | Listar os filhos de um **DriveItem** com base no caminho relativo para a raiz do OneDrive do usuário. |
| `/me/drive/items/{item-id}:/path/to/folder:/children` | Listar os filhos de um **DriveItem** com base no caminho relativo para outro item. |

## <a name="drive-resource"></a>Recurso Drive

O [recurso Drive](drive.md) é o objeto de nível superior no OneDrive de um usuário ou em uma biblioteca de documentos do SharePoint. Quase todas as operações de arquivos serão iniciadas com o endereçamento de um recurso Drive específico.

Um recurso Drive pode ser endereçado pela ID exclusiva da unidade ou pela unidade padrão de um [Usuário](user.md), [Grupo](group.md) ou organização. 

## <a name="driveitem-resource"></a>Recurso DriveItem

[DriveItems](driveitem.md) são os objetos dentro do sistema de arquivos da unidade. Eles podem ser acessados por **id** usando a sintaxe `/items/{item-id}` ou por caminho no sistema de arquivos usando a sintaxe `/root:/path/to/item/`.

DriveItems têm _facetas_ que fornecem dados sobre a identidades e as capacidades do item.

DriveItems com uma faceta **folder** atuam como contêineres de itens e têm uma referência **children** que aponta para uma coleção de itens na pasta.

## <a name="shared-folders-and-remote-items"></a>Pastas compartilhadas e itens remotos

Um usuário pessoal do OneDrive pode adicionar um ou mais itens compartilhados de outra unidade a seu próprio OneDrive. Esses itens compartilhados aparecem como **DriveItem** na coleção **children** com uma faceta [remoteItem](remoteitem.md).

Para saber mais sobre como trabalhar com pastas compartilhadas e itens remotos, confira [Itens remotos e pastas compartilhadas](remoteitem.md).   

## <a name="sharing-and-permissions"></a>Compartilhamento e permissões

Uma das ações mais comuns para bibliotecas de documentos do OneDrive e do SharePoint é compartilhar conteúdo com outras pessoas. O Microsoft Graph permite que seu aplicativo crie [links de compartilhamento](../api/item_createLink.md), [adicione permissões e envie convites](../api/item_invite.md) para itens em uma unidade.

O Microsoft Graph também fornece uma maneira de seu aplicativo [acessar conteúdo compartilhado](../api/shares_get.md) diretamente de um link de compartilhamento.

 