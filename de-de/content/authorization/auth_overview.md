# <a name="app-authentication-with-microsoft-graph"></a>App-Authentifizierung mit Microsoft Graph

Um auf die Microsoft-Daten eines Benutzers zuzugreifen, muss die Anwendung es den Benutzern ermöglichen, ihre Identität zu authentifizieren und ihr Einverständnis zu geben, dass die App Aktionen in ihrem Namen ausführen darf.

Der Microsoft Graph unterstützt zwei Authentifizierungsanbieter:

- Verwenden Sie für die Authentifizierung von Benutzern mit persönlichen Microsoft-Konten, beispielsweise ein _live.com-_ oder _outlook.com-_Konto, den Endpunkt [Azure Active Directory (Azure AD) v2.0](converged_auth).
- Verwenden Sie für die Authentifizierung von Benutzern mit Unternehmenskonten (Arbeit oder Bildungseinrichtung) [Azure AD](app_authorization).


> **Sie erstellen Apps für Unternehmenskunden?** Ihre App funktioniert möglicherweise nicht, wenn Ihr Unternehmenskunde Enterprise Mobility-Sicherheitsfunktionen wie <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-conditional-access-device-policies/" target="_newtab">bedingten Gerätezugriff</a> aktiviert.  

> Zur Unterstützung **aller Unternehmenskunden** über **alle Unternehmensszenarien** hinweg müssen Sie den Azure AD-Endpunkt verwenden und Ihre Apps mithilfe des [Azure-Verwaltungsportals](https://aka.ms/aadapplist) verwalten.

![Microsoft Graph-Anwendungsstapel. Die Authentifizierung ist als Schicht zwischen Ihrer App und den unterschiedlichen Microsoft Graph-Ressourcen dargestellt.](./images/MSGraph_DevStack_v2Auth.png)

## <a name="deciding-between-the-azure-ad-and-azure-ad-v2.0-endpoints"></a>Entscheiden zwischen dem Azure AD- und dem Azure AD v2.0-Endpunkt

In der folgenden Tabelle sind die wichtigsten Features zusammengefasst, die Azure AD- und Azure AD v2.0-Endpunkte unterstützen. Außerdem werde Links zu weiteren Informationen bereitgestellt. Die relative Wichtigkeit dieser Features – und somit die Auswahl des Authentifizierungsanbieters für Bereitstellung in Ihrer App – hängt hauptsächlich von folgenden Faktoren ab:

- Dem Kontotyp (Unternehmen oder Heimanwender), den Ihre App unterstützen muss
- Der Art der App, die Sie erstellen möchten
- Dem erforderlichen Authentifizierungsfluss 

<table style="width:100%">
  <tr>
    <th></th>
    <th>Dem Azure AD-Endpunkt</th> 
    <th>Dem Azure AD v2.0-Endpunkt</th>
   </tr>
  <tr>
    <td>Den unterstützten Grant-Typen</td>
    <td><p>Dem Authorisierungscode</p><p>Implizit</p><p>Client-Anmeldeinformationen</p><p>Anmeldeinformationen des Resourcenbesitzers</p></td> 
    <td><p>Dem Authorisierungscode</p><p>Implizit</p><p>Client-Anmeldeinformationen</p></td>
   </tr>
  <tr>
    <td>Unterstützte Typen hinzufügen</td>
    <td><p>Web-Apps</p><p>Web-APIs</p><p>Mobile und native Apps</p><p>Einzelseiten-App (Single Page App, SPA)</p><p>Eigenständige Web-Apps</p><p>Daemons/Server Side Apps</p><p><a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-authentication-scenarios/" target="_newtab">Informationen</a></p></td> 
    <td><p>Web-Apps</p><p>Web-APIs</p><p>Mobile und native Apps</p><p>Einzelseiten-App (Single Page App, SPA)</p><p>Daemons/Server Side Apps</p><p><a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-flows/" target="_newtab">Informationen</a></td>
   </tr>
  <tr>
    <td>Geräterichtlinien für bedingten Zugriff</td>
     <td><a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-conditional-access-device-policies/" target="_newtab">Unterstützt</a></td> 
    <td>Derzeit nicht unterstützt</td>
   </tr>
  <tr>
    <td>OAuth 2.0- und OpenID Connect-konform</td>
    <td>Nein</td> 
    <td>Ja</td>
  </tr>
  <tr>
    <td>Berechtigungen</td>
    <td>Statisch: Festgelegt während der App-Registrierung </td> 
    <td><a href ="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare/#scopes-not-resources" target="_newtab">Dynamisch:</a> Anforderung während App-Laufzeit; beinhaltet inkrementelle Zustimmung</td>
  </tr>
  <tr>
    <td>Kontotypen</td>
    <td> <p>Geschäft oder Schule</p></td> 
    <td><p>Geschäft oder Schule</p><p>Persönlich</p> </td>
  </tr>
  <tr>
    <td>App-ID </td>
    <td>Separate App-ID für jede Plattform</td> 
    <td><a href ="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare/#one-app-id-for-all-platforms" target="_newtab">Einzelne App-ID für verschiedene Plattformen</a></td>
  </tr>
  <tr>
    <td>Registrierungsportal </td>
    <td><a href ="https://manage.windowsazure.com/" target="_newtab">Microsoft Azure-Management</a></td> 
    <td><a href ="https://apps.dev.microsoft.com" target="_newtab">Microsoft App-Registrierung</a></td>
  </tr>
  <tr>
    <td>Client-Bibliotheken </td>
    <td>ADAL-SDKs (Active Directory Authentification) für die meisten Entwicklungsplattformen</td> 
    <td><p><a href="https://www.nuget.org/packages/Microsoft.Identity.Client" target="_newtab">Microsoft-Authentifizierungsbibliothek (Vorschau)</a></p><p><a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/#restrictions-on-libraries-amp-sdks" target="_newtab">Open Source OAuth 2.0-Bibliotheken</a></p> </td>
  </tr>
  <tr>
    <td>Sonstige Funktionen </td>
    <td><p>Gruppenansprüche für Azure AD-Benutzer</p><p>Anwendungsrollen und Rollenansprüche</p></td> 
    <td></td>
  </tr>
</table> 

Außerdem erfordern die beiden Authentifizierungsanbieter unterschiedliche Zustimmungsbereiche. Unterschiede hinsichtlich der Ansprüche, die in unterschiedlichen Tokens zurückgegeben werden, sind ebenfalls vorhanden. Weitere Informationen finden Sie unter [Bekannte Bereiche](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare/#well-known-scopes) und [Token-Ansprüche](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare/#token-claims) in [Was ist anders am v2.0-Endpunkt?](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare/).

Der Azure AD v2.0-Endpunkt befindet sich außerdem in aktiver Entwicklung, es werden also zusätzliche Features und unterstützte Szenarien hinzugefügt. Eine aktuelle Liste mit Einschränkungen für den Azure AD v2.0-Endpunkt finden Sie unter [Sollte ich den v2.0-Endpunkt verwenden?](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/).

## <a name="registering-your-app-for-authentication"></a>Registrieren Ihrer App zur Authentifizierung 

Wenn Sie sich für den Authentifizierungsanbieter entschieden haben, der die Anforderungen Ihrer App erfüllt, müssen Sie Ihre App im Portal des Authentifizierungsanbieters registrieren. Durch die Registrierung Ihrer App wird die Identität Ihrer App beim Authentifizierungsanbieter geschaffen. Außerdem kann die App so ihre Identität angeben, wenn sie Authentifizierungsanfragen vom Benutzer einreicht.

- Verwenden Sie das [Azure-Portal](https://portal.azure.com/), um ihre App bei Azure AD zu registrieren.

    <!--For Azure AD, you'll also need to [associate your Office 365 account with Azure AD subscription](../auth_azure_ad/associate_account.md) in order to manage your apps.-->

- Verwenden Sie das [Microsoft App-Registrierungsportal](auth_register_app_v2.md), um [Ihre App beim Azure AD v2.0-Endpunkt zu registrieren](https://apps.dev.microsoft.com).


## <a name="resources-for-implementing-authentication-in-your-microsoft-graph-app"></a>Ressourcen für die Implementierung von Authentifizierungen in Ihrer Microsoft Graph-App. 

Nachdem Sie Ihre App beim entsprechenden Authentifizierungsportal registriert haben und über die App-Registrierungsinformationen (App-ID, App-Geheimnis, falls zutreffend, und die Umleitungs-URI) verfügen, die Sie für die Erstellung der App-identität benötigen, können Sie die Authentifizierung in Ihrer App implementieren. 

Dies ist erneut wiederum abhängig vom App-Typ, den Sie erstellen, Ihrer Entwicklungsplattform, dem gewählten Authentifizierungsfluss und den speziellen Authentifizierungsanforderungen für Ihre App. 

### <a name="connect-samples-by-authentication-provider-and-platform"></a>Beispiele nach Authentifizierungsanbieter und -plattform verbinden

In der folgenden Tabbelle sind die Connect-Beispiele nach Authentifizierungsanbieter und -plattform aufgeführt. Weiterhin wird angegeben, ob sie sich über REST oder eine Microsoft Graph-Client-Bibliothek mit Microsoft Graph verbinden.

<table>
  <tr>
    <th>Plattform</th>
    <th>Dem Azure AD-Endpunkt</th> 
    <th>Dem Azure AD v2.0-Endpunkt</th>
  </tr>
  <tr>
    <td>Android</td>
    <td>
        <a href="https://github.com/microsoftgraph/android-java-connect-rest-sample">REST-Beispiel</a> oder <a href="https://github.com/microsoftgraph/android-java-connect-sample/tree/last_v1_auth">SDK-Beispiel</a>
    </td> 
    <td>
        <a href="https://github.com/microsoftgraph/android-java-connect-sample">SDK-Beispiel</a>
    </td> 
  </tr>
  <tr>
    <td>ASP.NET</td>
    <td>
        <a href="https://github.com/microsoftgraph/aspnet-connect-rest-sample">REST-Beispiel</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/aspnet-connect-sample">SDK-Beispiel</a>
    </td> 
  </tr>
  <tr>
    <td>iOS (Obj-C)</td>
    <td>
        <a href="https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample">REST-Beispiel</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/ios-objectivec-connect-sample">SDK-Beispiel</a>
    </td> 
  </tr>
  <tr>
    <td>iOS (Swift)</td>
    <td>
        <a href="https://github.com/microsoftgraph/ios-swift-connect-rest-sample">REST-Beispiel</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/ios-swift-connect-sample">SDK-Beispiel</a>
    </td> 
  </tr>
  <tr>
    <td>Node.js</td>
    <td>
        <a href="https://github.com/microsoftgraph/nodejs-connect-rest-sample/tree/last_v1_auth">REST-Beispiel</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/nodejs-connect-rest-sample">REST-Beispiel</a>
    </td> 
  </tr>
  <tr>
    <td>PHP</td>
    <td>
        <a href="https://github.com/microsoftgraph/php-connect-rest-sample/tree/last_v1_auth">REST-Beispiel</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/php-connect-rest-sample">REST-Beispiel</a>
    </td> 
  </tr>
  <tr>
    <td>Python</td>
    <td>
        <a href="https://github.com/microsoftgraph/python3-connect-rest-sample">REST-Beispiel</a>
    </td>     
    <td>
    </td> 
  </tr>
  <tr>
    <td>Ruby</td>
    <td>
        <a href="https://github.com/microsoftgraph/ruby-connect-rest-sample/tree/last_v1_auth">REST-Beispiel</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/ruby-connect-rest-sample">REST-Beispiel</a>
    </td> 
  </tr>
  <tr>
    <td>UWP</td>
    <td>
        <a href="https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample/tree/last_v1_auth">REST-Beispiel</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample">REST-Beispiel</a> oder <a href="https://github.com/microsoftgraph/uwp-csharp-connect-sample">SDK-Beispiel</a>
    </td> 
  </tr>
  <tr>
    <td>Xamarin</td>
    <td>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/xamarin-csharp-connect-sample">SDK-Beispiel</a>
    </td> 
  </tr>
</table>

Eine große Bandbreite an Projekten, die sich über ein breites Sortiment an Technologien mit Microsoft Graph verbinden, finden Sie im [Microsoft Graph-Repository](https://github.com/microsoftgraph) auf GitHub. 

### <a name="get-started"></a>Erste Schritte  

Der Abschnitt [Erste Schritte](http://graph.microsoft.io/en-us/docs/platform/get-started) enthält detaillierte Artikel, in den beschrieben wird, wie Sie die in der Tabelle aufgeführten Apps mit dem Azure AD v2.0-Endpunkt erstellen. Außerdem werden die Authentifizierungsbibliotheken besprochen, die auf jeder Plattform verwendet werden. 

## <a name="see-also"></a>Siehe auch

- <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-authentication-scenarios/#basics-of-authentication-in-azure-ad" target="_newtab">Authentifizierungsszenarien für Azure AD</a>

- <a href="https://azure.microsoft.com/en-us/documentation/articles/?product=active-directory&term=v2.0+endpoint" target="_newtab">Azure AD v2.0-Endpunktdokumentation auf Azure.com</a>
- <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-app-registration/#build-a-quick-start-app" target="_newtab">Schnellstarts für Azure AD v2.0-Code auf Azure.com</a>
