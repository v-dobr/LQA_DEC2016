# <a name="app-authentication-with-microsoft-graph"></a>Microsoft Graph でのアプリ認証

ユーザーの Microsoft データにアクセスするために、アプリケーションは、ユーザーが自分自身の ID を認証し、自分に代わってアプリが操作を実行することを承諾できるようにする必要があります。

Microsoft Graph は 2 種類の認証プロバイダーをサポートします。

- _live.com_ または _outlook.com_ アカウントなどの個人用 Microsoft アカウントを持つユーザーを認証するには、[Azure Active Directory (Azure AD) v2.0 エンドポイント](converged_auth)を使用します。
- エンタープライズ (職場または学校など) のアカウントを持つユーザーを認証するには、[Azure AD](app_authorization) を使用します。


> **エンタープライズのお客様向けにアプリを作成していますか?**エンタープライズのお客様が、<a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-conditional-access-device-policies/" target="_newtab">デバイスへの条件付きアクセス</a>のようなエンタープライズ モビリティ セキュリティ機能をオンにしている場合、アプリが動作しない可能性があります。  

> **すべてのエンタープライズのお客様**の**すべてのエンタープライズ シナリオ**をサポートするには、Azure AD エンドポイントを使用し、[Azure 管理ポータル](https://aka.ms/aadapplist)でアプリを管理する必要があります。

![Microsoft Graph アプリケーションのスタック。認証が、アプリとさまざまな Microsoft Graph リソースとの間の層として示されている。](./images/MSGraph_DevStack_v2Auth.png)

## <a name="deciding-between-the-azure-ad-and-azure-ad-v2.0-endpoints"></a>Azure AD エンドポイントか Azure AD v2.0 エンドポイントかを決定する

次の表には、Azure AD エンドポイントと Azure AD v2.0 エンドポイントがサポートする主な機能と、追加情報へのリンクが記されています。これらの機能の相対的な重要性、およびどちらの認証プロバイダーをアプリに実装するかの選択は、主に次の要素に左右されます。

- アプリがサポートする必要があるアカウントの種類 (エンタープライズまたはコンシューマー)
- 構築するアプリの種類
- 必要な認証フロー 

<table style="width:100%">
  <tr>
    <th></th>
    <th>Azure AD エンドポイント</th> 
    <th>Azure AD v2.0 エンドポイント</th>
   </tr>
  <tr>
    <td>サポートされる許可の種類</td>
    <td><p>認証コード</p><p>暗黙的</p><p>クライアントの資格情報</p><p>リソース所有者のパスワード資格情報</p></td> 
    <td><p>認証コード</p><p>暗黙的</p><p>クライアントの資格情報</p></td>
   </tr>
  <tr>
    <td>サポートされるアプリの種類</td>
    <td><p>Web アプリ</p><p>Web API</p><p>モバイル アプリとネイティブ アプリ</p><p>シングル ページ アプリ (SPA)</p><p>スタンドアロン Web API</p><p>デーモン/サーバー側のアプリ</p><p><a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-authentication-scenarios/" target="_newtab">詳細情報</a></p></td> 
    <td><p>Web アプリ</p><p>Web API</p><p>モバイル アプリとネイティブ アプリ</p><p>シングル ページ アプリ (SPA)</p><p>デーモン/サーバー側のアプリ</p><p><a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-flows/" target="_newtab">詳細情報</a></td>
   </tr>
  <tr>
    <td>条件付きアクセス デバイス ポリシー</td>
     <td><a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-conditional-access-device-policies/" target="_newtab">サポートされています</a></td> 
    <td>現在サポートされていません</td>
   </tr>
  <tr>
    <td>OAuth 2.0 および OpenID Connect に準拠</td>
    <td>いいえ</td> 
    <td>はい</td>
  </tr>
  <tr>
    <td>アクセス許可</td>
    <td>静的:アプリの登録中に指定されます </td> 
    <td><a href ="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare/#scopes-not-resources" target="_newtab">動的:</a>アプリ ランタイム中の要求。段階的な同意が含まれます</td>
  </tr>
  <tr>
    <td>アカウントの種類</td>
    <td> <p>職場または学校</p></td> 
    <td><p>職場または学校</p><p>個人</p> </td>
  </tr>
  <tr>
    <td>アプリ ID </td>
    <td>各プラットフォームに個別のアプリ ID</td> 
    <td><a href ="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare/#one-app-id-for-all-platforms" target="_newtab">複数のプラットフォームに 1 つのアプリ ID</a></td>
  </tr>
  <tr>
    <td>登録ポータル </td>
    <td><a href ="https://manage.windowsazure.com/" target="_newtab">Microsoft Azure 管理</a></td> 
    <td><a href ="https://apps.dev.microsoft.com" target="_newtab">Microsoft アプリケーション登録</a></td>
  </tr>
  <tr>
    <td>クライアント ライブラリ </td>
    <td>ほとんどの開発プラットフォームに使用可能な Active Directory 認証 (ADAL) SDK</td> 
    <td><p><a href="https://www.nuget.org/packages/Microsoft.Identity.Client" target="_newtab">Microsoft Authentication Library (プレビュー)</a></p><p><a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/#restrictions-on-libraries-amp-sdks" target="_newtab">オープン ソース OAuth 2.0 ライブラリ (リスト)</a></p> </td>
  </tr>
  <tr>
    <td>その他の機能 </td>
    <td><p>Azure AD ユーザーへのグループ要求</p><p>アプリケーションの役割と役割要求</p></td> 
    <td></td>
  </tr>
</table> 

他にも、2 つの認証プロバイダーが求めるアクセス許可の範囲に違いがあり、さまざまなトークンで返される要求にも違いがあります。詳細については、「[v2.0 エンドポイントは何が違うのか?](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare/)」の「[よく知られている範囲](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare/#well-known-scopes)」と「[トークンの要求](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare/#token-claims)」をご覧ください。

また、Azure AD v2.0 エンドポイントは現在も開発途中で、他の機能やサポートされるシナリオが追加される予定です。現時点での Azure AD v2.0 エンドポイントの制限事項や制約をまとめたリストは、「[v2.0 エンドポイントを使用したほうがいいでしょうか?](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/)」をご覧ください。

## <a name="registering-your-app-for-authentication"></a>認証するためにアプリを登録する 

アプリの要件を満たしている認証プロバイダーを決めたら、その認証プロバイダーのポータルでアプリを登録する必要があります。アプリを登録すると、認証プロバイダーによってそのアプリの ID が確立され、ユーザーからの認証要求を送信するとき、アプリは自身の ID を指定できるようになります。

- Azure AD にアプリを登録するには、[Azure ポータル](https://portal.azure.com/)を使用します。

    <!--For Azure AD, you'll also need to [associate your Office 365 account with Azure AD subscription](../auth_azure_ad/associate_account.md) in order to manage your apps.-->

- [Azure AD v2.0 エンドポイントにアプリを登録](auth_register_app_v2.md)するには、[Microsoft アプリケーション登録ポータル](https://apps.dev.microsoft.com)を使用します。


## <a name="resources-for-implementing-authentication-in-your-microsoft-graph-app"></a>Microsoft Graph アプリに認証を実装するためのリソース 

該当する認証ポータルにアプリを登録し、アプリ ID の確立に必要なアプリ登録情報 (アプリ ID、該当する場合はアプリケーション シークレット、リダイレクト URI) を入手したら、アプリに認証を実装する準備ができたことになります。 

繰り返しになりますが、これは、構築しているアプリ、開発プラットフォーム、選んだ認証フローの種類、アプリ固有の認証要件によって異なります。 

### <a name="connect-samples-by-authentication-provider-and-platform"></a>認証プロバイダーおよびプラットフォーム別の Connect サンプル

次の表は、認証プロバイダーとプラットフォームごとに Connect サンプルをリストにしたもので、REST または Microsoft Graph クライアント ライブラリのどちらを使用して Microsoft Graph に接続するかを示しています。

<table>
  <tr>
    <th>プラットフォーム</th>
    <th>Azure AD エンドポイント</th> 
    <th>Azure AD v2.0 エンドポイント</th>
  </tr>
  <tr>
    <td>Android</td>
    <td>
        <a href="https://github.com/microsoftgraph/android-java-connect-rest-sample">REST サンプル</a>または <a href="https://github.com/microsoftgraph/android-java-connect-sample/tree/last_v1_auth">SDK サンプル</a>
    </td> 
    <td>
        <a href="https://github.com/microsoftgraph/android-java-connect-sample">SDK サンプル</a>
    </td> 
  </tr>
  <tr>
    <td>ASP.NET</td>
    <td>
        <a href="https://github.com/microsoftgraph/aspnet-connect-rest-sample">REST サンプル</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/aspnet-connect-sample">SDK サンプル</a>
    </td> 
  </tr>
  <tr>
    <td>iOS (Obj-C)</td>
    <td>
        <a href="https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample">REST サンプル</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/ios-objectivec-connect-sample">SDK サンプル</a>
    </td> 
  </tr>
  <tr>
    <td>iOS (Swift)</td>
    <td>
        <a href="https://github.com/microsoftgraph/ios-swift-connect-rest-sample">REST サンプル</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/ios-swift-connect-sample">SDK サンプル</a>
    </td> 
  </tr>
  <tr>
    <td>Node.js</td>
    <td>
        <a href="https://github.com/microsoftgraph/nodejs-connect-rest-sample/tree/last_v1_auth">REST サンプル</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/nodejs-connect-rest-sample">REST サンプル</a>
    </td> 
  </tr>
  <tr>
    <td>PHP</td>
    <td>
        <a href="https://github.com/microsoftgraph/php-connect-rest-sample/tree/last_v1_auth">REST サンプル</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/php-connect-rest-sample">REST サンプル</a>
    </td> 
  </tr>
  <tr>
    <td>Python</td>
    <td>
        <a href="https://github.com/microsoftgraph/python3-connect-rest-sample">REST サンプル</a>
    </td>     
    <td>
    </td> 
  </tr>
  <tr>
    <td>Ruby</td>
    <td>
        <a href="https://github.com/microsoftgraph/ruby-connect-rest-sample/tree/last_v1_auth">REST サンプル</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/ruby-connect-rest-sample">REST サンプル</a>
    </td> 
  </tr>
  <tr>
    <td>UWP</td>
    <td>
        <a href="https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample/tree/last_v1_auth">REST サンプル</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample">REST サンプル</a>または <a href="https://github.com/microsoftgraph/uwp-csharp-connect-sample">SDK サンプル</a>
    </td> 
  </tr>
  <tr>
    <td>Xamarin</td>
    <td>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/xamarin-csharp-connect-sample">SDK サンプル</a>
    </td> 
  </tr>
</table>

さまざまなテクノロジを介して Microsoft Graph に接続する広範囲に及ぶプロジェクトを調べるには、GitHub の [Microsoft Graph リポジトリ](https://github.com/microsoftgraph)にアクセスします。 

### <a name="get-started"></a>作業の開始  

「[作業の開始](http://graph.microsoft.io/en-us/docs/platform/get-started)」セクションには、表に一覧表示されているアプリを Azure AD v2.0 エンドポイントを使用して作成する方法を説明する詳しい記事があり、各プラットフォームで使用される認証ライブラリが説明されています。 

## <a name="see-also"></a>関連項目

- <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-authentication-scenarios/#basics-of-authentication-in-azure-ad" target="_newtab">Azure AD の認証シナリオ</a>

- <a href="https://azure.microsoft.com/en-us/documentation/articles/?product=active-directory&term=v2.0+endpoint" target="_newtab">Azure.com にある Azure AD v2.0 エンドポイントに関する文書</a>
- <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-app-registration/#build-a-quick-start-app" target="_newtab">Azure.com にある Azure AD v2.0 コード クイック スタート</a>
