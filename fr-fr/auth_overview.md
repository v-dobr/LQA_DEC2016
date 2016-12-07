# <a name="app-authentication-with-microsoft-graph"></a>Authentification d’application avec Microsoft Graph

Pour accéder aux données Microsoft d’un utilisateur, votre application doit permettre aux utilisateurs d’authentifier leur identité et d’octroyer leur consentement pour que l’application effectue des actions en leur nom.

Microsoft Graph prend en charge deux fournisseurs d’authentification :

- Pour authentifier les utilisateurs avec des comptes personnels Microsoft, tels que _live.com_ ou _outlook.com_, utilisez le [point de terminaison Azure Active Directory (AD Azure) v2.0](converged_auth).
- Pour authentifier les utilisateurs avec des comptes Entreprise (autrement dit, professionnels ou scolaires), utilisez [Azure AD](app_authorization).


> **Vous voulez créer des applications pour des entreprises ?** Il est possible que votre application ne fonctionne pas si l’entreprise a activé les fonctionnalités de sécurité pour la mobilité en entreprise comme l’<a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-conditional-access-device-policies/" target="_newtab">accès conditionnel des appareils</a>.  

> Pour prendre en charge **toutes les entreprises clientes** dans **tous les scénarios**, utilisez le point de terminaison Azure AD et gérez vos applications avec le [portail de gestion Azure](https://aka.ms/aadapplist).

![Pile des applications Microsoft Graph, avec l’authentification indiquée comme une couche entre votre application et les différentes ressources Microsoft Graph.](./images/MSGraph_DevStack_v2Auth.png)

## <a name="deciding-between-the-azure-ad-and-azure-ad-v2.0-endpoints"></a>Fonctionnalités des points de terminaison Azure AD et Azure AD v2.0

Le tableau suivant résume les principales fonctionnalités prises en charge par les points de terminaison Azure AD et Azure AD v2.0 et fournit des liens vers des informations supplémentaires. L’importance relative de ces fonctionnalités (et par conséquent, le fournisseur d’authentification que vous choisissez d’implémenter dans votre application) dépendra principalement des éléments suivants :

- Le type de compte (Entreprise ou consommateur) que vos applications doivent prendre en charge
- Le type d’application que vous souhaitez créer
- Le flux d’authentification requis 

<table style="width:100%">
  <tr>
    <th></th>
    <th>Point de terminaison Azure AD</th> 
    <th>Point de terminaison Azure AD v2.0</th>
   </tr>
  <tr>
    <td>Types d’octroi pris en charge</td>
    <td><p>Code d’autorisation</p><p>Implicite</p><p>Informations d’identification du client</p><p>Informations d’identification du mot de passe du propriétaire de la ressource</p></td> 
    <td><p>Code d’autorisation</p><p>Implicite</p><p>Informations d’identification du client</p></td>
   </tr>
  <tr>
    <td>Types d’application pris en charge</td>
    <td><p>Applications web</p><p>API web</p><p>Applications mobiles et natives</p><p>SPA</p><p>API web autonomes</p><p>Applications côté serveur/démon</p><p><a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-authentication-scenarios/" target="_newtab">plus d'informations</a></p></td> 
    <td><p>Applications web</p><p>API web</p><p>Applications mobiles et natives</p><p>SPA</p><p>Applications côté serveur/démon</p><p><a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-flows/" target="_newtab">plus d'informations</a></td>
   </tr>
  <tr>
    <td>Stratégies d’appareil d’accès conditionnel</td>
     <td><a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-conditional-access-device-policies/" target="_newtab">Pris en charge</a></td> 
    <td>N'est pas pris en charge actuellement</td>
   </tr>
  <tr>
    <td>Compatible avec OAuth 2.0 et OpenID Connect</td>
    <td>Non</td> 
    <td>Oui</td>
  </tr>
  <tr>
    <td>Autorisations</td>
    <td>Statique : Spécifié lors de l’inscription des applications </td> 
    <td><a href ="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare/#scopes-not-resources" target="_newtab">Dynamique :</a> Demande lors de l’exécution de l’application ; inclut le consentement incrémentiel</td>
  </tr>
  <tr>
    <td>Types de compte</td>
    <td> <p>professionnel ou scolaire</p></td> 
    <td><p>professionnel ou scolaire</p><p>personnel</p> </td>
  </tr>
  <tr>
    <td>ID de l’application </td>
    <td>ID d’application distinct pour chaque plateforme</td> 
    <td><a href ="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare/#one-app-id-for-all-platforms" target="_newtab">ID d’application unique pour plusieurs plateformes</a></td>
  </tr>
  <tr>
    <td>Portail d’inscription </td>
    <td><a href ="https://manage.windowsazure.com/" target="_newtab">Gestion de Microsoft Azure</a></td> 
    <td><a href ="https://apps.dev.microsoft.com" target="_newtab">Inscription de l’application Microsoft</a></td>
  </tr>
  <tr>
    <td>Bibliothèques clientes </td>
    <td>Kits de développement logiciel (SDK) d’authentification Active Directory pour la plupart des plateformes de développement</td> 
    <td><p><a href="https://www.nuget.org/packages/Microsoft.Identity.Client" target="_newtab">Bibliothèque d’authentification de Microsoft (aperçu)</a></p><p><a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/#restrictions-on-libraries-amp-sdks" target="_newtab">Ouvrir des bibliothèques OAuth 2.0 source (liste)</a></p> </td>
  </tr>
  <tr>
    <td>Autres fonctionnalités </td>
    <td><p>Revendications de groupe pour les utilisateurs Azure AD</p><p>Rôles d’application et revendications de rôle</p></td> 
    <td></td>
  </tr>
</table> 

En outre, il existe des différences dans les étendues d’autorisation requises par les deux fournisseurs d’authentification, ainsi que des différences entre les revendications renvoyées dans divers jetons. Pour plus d’informations, voir les rubriques relatives aux [étendues connues](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare/#well-known-scopes) et aux [revendications de jeton](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare/#token-claims) dans les [différences du point de terminaison v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare/).

En outre, le point de terminaison Azure AD v2.0 est en cours de développement actif, avec des fonctionnalités supplémentaires et des scénarios pris en charge à ajouter. Pour une liste actuelle des limitations et restrictions relatives au point de terminaison Azure AD v2.0, voir la rubrique relative à l’[utilisation du point de terminaison v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/).

## <a name="registering-your-app-for-authentication"></a>Inscription de votre application pour l’authentification 

Lorsque vous choisissez le fournisseur d’authentification qui répond aux exigences de votre application, vous devez inscrire votre application sur le portail de ce fournisseur d’authentification. L’inscription de votre application établit l’identité de votre application avec le fournisseur d’authentification et permet à votre application de spécifier son identité lors de l’envoi de demandes d’authentification de l’utilisateur.

- Pour inscrire votre application avec Azure AD, utilisez le [portail Azure](https://portal.azure.com/).

    <!--For Azure AD, you'll also need to [associate your Office 365 account with Azure AD subscription](../auth_azure_ad/associate_account.md) in order to manage your apps.-->

- Pour [inscrire votre application avec le point de terminaison Azure AD v2.0](auth_register_app_v2.md), utilisez le [portail d’inscription de l’application Microsoft](https://apps.dev.microsoft.com).


## <a name="resources-for-implementing-authentication-in-your-microsoft-graph-app"></a>Ressources pour l’implémentation de l’authentification dans votre application Microsoft Graph 

Une fois que vous avez inscrit votre application avec le portail d’authentification approprié et que vous disposez des informations d’inscription de l’application (ID d’application, question secrète de l’application, le cas échéant, et URI de redirection) nécessaires pour établir l’identité de votre application, vous pouvez implémenter l’authentification dans votre application. 

Là encore, cela dépendra du type d’application que vous créez, de votre plateforme de développement, du flux d’authentification que vous choisissez et des besoins d’authentification spécifiques de votre application. 

### <a name="connect-samples-by-authentication-provider-and-platform"></a>Exemples de connexion par fournisseur d’authentification et plateforme

Le tableau suivant répertorie les exemples de connexion par fournisseur d’authentification et plateforme, et les remarques relatives à la connexion à Microsoft Graph avec REST ou une bibliothèque cliente Microsoft Graph.

<table>
  <tr>
    <th>Plateforme</th>
    <th>Point de terminaison Azure AD</th> 
    <th>Point de terminaison Azure AD v2.0</th>
  </tr>
  <tr>
    <td>Android</td>
    <td>
        <a href="https://github.com/microsoftgraph/android-java-connect-rest-sample">Exemple REST</a> ou <a href="https://github.com/microsoftgraph/android-java-connect-sample/tree/last_v1_auth">Exemple SDK</a>
    </td> 
    <td>
        <a href="https://github.com/microsoftgraph/android-java-connect-sample">Exemple SDK</a>
    </td> 
  </tr>
  <tr>
    <td>ASP.NET</td>
    <td>
        <a href="https://github.com/microsoftgraph/aspnet-connect-rest-sample">Exemple REST</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/aspnet-connect-sample">Exemple SDK</a>
    </td> 
  </tr>
  <tr>
    <td>iOS (Obj-C)</td>
    <td>
        <a href="https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample">Exemple REST</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/ios-objectivec-connect-sample">Exemple SDK</a>
    </td> 
  </tr>
  <tr>
    <td>iOS (Swift)</td>
    <td>
        <a href="https://github.com/microsoftgraph/ios-swift-connect-rest-sample">Exemple REST</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/ios-swift-connect-sample">Exemple SDK</a>
    </td> 
  </tr>
  <tr>
    <td>Node.js</td>
    <td>
        <a href="https://github.com/microsoftgraph/nodejs-connect-rest-sample/tree/last_v1_auth">Exemple REST</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/nodejs-connect-rest-sample">Exemple REST</a>
    </td> 
  </tr>
  <tr>
    <td>PHP</td>
    <td>
        <a href="https://github.com/microsoftgraph/php-connect-rest-sample/tree/last_v1_auth">Exemple REST</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/php-connect-rest-sample">Exemple REST</a>
    </td> 
  </tr>
  <tr>
    <td>Python</td>
    <td>
        <a href="https://github.com/microsoftgraph/python3-connect-rest-sample">Exemple REST</a>
    </td>     
    <td>
    </td> 
  </tr>
  <tr>
    <td>Ruby</td>
    <td>
        <a href="https://github.com/microsoftgraph/ruby-connect-rest-sample/tree/last_v1_auth">Exemple REST</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/ruby-connect-rest-sample">Exemple REST</a>
    </td> 
  </tr>
  <tr>
    <td>UWP</td>
    <td>
        <a href="https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample/tree/last_v1_auth">Exemple REST</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample">Exemple REST</a> ou <a href="https://github.com/microsoftgraph/uwp-csharp-connect-sample">Exemple SDK</a>
    </td> 
  </tr>
  <tr>
    <td>Xamarin</td>
    <td>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/xamarin-csharp-connect-sample">Exemple SDK</a>
    </td> 
  </tr>
</table>

Pour explorer un grand nombre de projets qui se connectent à Microsoft Graph via un large éventail de technologies, lisez l’article relatif au [référentiel Microsoft Graph](https://github.com/microsoftgraph) sur GitHub. 

### <a name="get-started"></a>Prise en main  

La section [Prise en main](http://graph.microsoft.io/en-us/docs/platform/get-started) contient des articles qui décrivent comment créer les applications répertoriées dans le tableau en utilisant le point de terminaison Azure AD v2.0 et présente les bibliothèques d’authentification utilisées sur chaque plateforme. 

## <a name="see-also"></a>Voir aussi

- <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-authentication-scenarios/#basics-of-authentication-in-azure-ad" target="_newtab">Scénarios d’authentification pour Azure AD</a>

- <a href="https://azure.microsoft.com/en-us/documentation/articles/?product=active-directory&term=v2.0+endpoint" target="_newtab">Documentation du point de terminaison Azure AD v2.0 sur Azure.com</a>
- <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-app-registration/#build-a-quick-start-app" target="_newtab">Démarrage rapide du code Azure AD v2.0 sur Azure.com</a>
