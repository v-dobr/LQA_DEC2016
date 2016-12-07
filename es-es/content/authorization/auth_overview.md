# <a name="app-authentication-with-microsoft-graph"></a>Autenticación de aplicaciones con Microsoft Graph

Para obtener acceso a los datos de un usuario de Microsoft, la aplicación deberá permitir que los usuarios autentiquen su identidad y le den su consentimiento para realizar acciones en su nombre.

Microsoft Graph admite dos proveedores de autenticación:

- Para autenticar los usuarios con cuentas personales de Microsoft, tales como las de _live.com_ o _outlook.com_, use el [punto de conexión de Azure Active Directory (AD Azure) v2.0](converged_auth).
- Para autenticar los usuarios con cuentas empresariales (es decir, profesionales o educativas), use [Azure AD](app_authorization).


> **¿Desea compilar aplicaciones para clientes empresariales?** Es posible que la aplicación no funcione si su cliente empresarial activa características de seguridad de movilidad empresarial como el <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-conditional-access-device-policies/" target="_newtab">acceso condicional al dispositivo</a>.  

> Para admitir **todos los clientes empresariales** en **todos los escenarios de empresa**, deberá usar el punto de conexión de Azure AD. Para administrar las aplicaciones, deberá usar el [Portal de administración de Azure](https://aka.ms/aadapplist).

![Pila de aplicaciones de Microsoft Graph, con la autenticación mostrada como nivel entre su aplicación y los distintos recursos de Microsoft Graph.](./images/MSGraph_DevStack_v2Auth.png)

## <a name="deciding-between-the-azure-ad-and-azure-ad-v2.0-endpoints"></a>Decidir entre los puntos de conexión de Azure AD y Azure AD v2.0

La tabla siguiente resume las características principales que admiten los puntos de conexión de Azure AD y Azure AD v2.0. Además, proporciona vínculos a información adicional. La importancia relativa de estas características y, por lo tanto, la elección del proveedor de autenticación para la implementación de la aplicación, dependerán principalmente de lo siguiente:

- El tipo de cuenta que necesiten admitir sus aplicaciones (empresarial o de consumidor)
- El tipo de aplicación que desee compilar
- El flujo de autenticación requerido 

<table style="width:100%">
  <tr>
    <th></th>
    <th>Punto de conexión de Azure AD</th> 
    <th>Punto de conexión v2.0 de Azure AD</th>
   </tr>
  <tr>
    <td>Tipos de concesión admitidos</td>
    <td><p>Código de autorización</p><p>Implícita</p><p>Credenciales de cliente</p><p>Credenciales de contraseña del propietario del recurso</p></td> 
    <td><p>Código de autorización</p><p>Implícita</p><p>Credenciales de cliente</p></td>
   </tr>
  <tr>
    <td>Tipos de aplicación admitidos</td>
    <td><p>Aplicaciones web</p><p>Interfaces de programación de aplicaciones web</p><p>Aplicaciones móviles y nativas</p><p>Aplicación de página única (SPA)</p><p>Interfaces de programación de aplicaciones web independientes</p><p>Aplicaciones demonio o del lado servidor</p><p><a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-authentication-scenarios/" target="_newtab">más información</a></p></td> 
    <td><p>Aplicaciones web</p><p>Interfaces de programación de aplicaciones web</p><p>Aplicaciones móviles y nativas</p><p>Aplicación de página única (SPA)</p><p>Aplicaciones demonio o del lado servidor</p><p><a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-flows/" target="_newtab">más información</a></td>
   </tr>
  <tr>
    <td>Directivas del dispositivo de acceso condicional</td>
     <td><a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-conditional-access-device-policies/" target="_newtab">Se admiten</a></td> 
    <td>No se admiten actualmente</td>
   </tr>
  <tr>
    <td>Conforme a OAuth 2.0 y OpenID Connect </td>
    <td>No</td> 
    <td>Sí</td>
  </tr>
  <tr>
    <td>Permisos</td>
    <td>Estáticos: Se especifican durante el registro de la aplicación </td> 
    <td><a href ="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare/#scopes-not-resources" target="_newtab">Dinámicos:</a> Solicitud durante la ejecución de la aplicación; incluye consentimiento incremental</td>
  </tr>
  <tr>
    <td>Tipos de cuenta</td>
    <td> <p>profesional o educativa</p></td> 
    <td><p>profesional o educativa</p><p>personal</p> </td>
  </tr>
  <tr>
    <td>Identificador de la aplicación </td>
    <td>Identificador de la aplicación independiente para cada plataforma</td> 
    <td><a href ="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare/#one-app-id-for-all-platforms" target="_newtab">Identificador de la aplicación único para varias plataformas</a></td>
  </tr>
  <tr>
    <td>Portal de registro </td>
    <td><a href ="https://manage.windowsazure.com/" target="_newtab">Administración de Microsoft Azure</a></td> 
    <td><a href ="https://apps.dev.microsoft.com" target="_newtab">Registro de aplicaciones de Microsoft</a></td>
  </tr>
  <tr>
    <td>Bibliotecas cliente </td>
    <td>Kits de desarrollo de software de autenticación de Active Directory (ADAL) para la mayoría de las plataformas de desarrollo</td> 
    <td><p><a href="https://www.nuget.org/packages/Microsoft.Identity.Client" target="_newtab">Biblioteca de autenticación de Microsoft (versión preliminar)</a></p><p><a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/#restrictions-on-libraries-amp-sdks" target="_newtab">Abrir bibliotecas de orígenes de OAuth 2.0 (lista)</a></p> </td>
  </tr>
  <tr>
    <td>Otras características </td>
    <td><p>Notificaciones de grupo para los usuarios de Azure AD</p><p>Roles de aplicación y notificaciones de rol</p></td> 
    <td></td>
  </tr>
</table> 

Además, existen diferencias en los ámbitos de permisos que requieren los dos proveedores de autenticación, así como diferencias en las notificaciones que se devuelven en varios tokens. Para obtener más información, consulte [Ámbitos conocidos](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare/#well-known-scopes) y [Notificaciones de token](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare/#token-claims) en [¿Qué hay diferente en el punto de conexión v2.0?](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare/)

Además, el punto de conexión v2.0 de Azure AD se encuentra en desarrollo activo y todavía deben agregarse más características y escenarios admitidos. Para obtener una lista actual de las limitaciones y las restricciones del punto de conexión v2.0 de Azure AD, consulte [¿Debería usar el punto de conexión v2.0?](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/)

## <a name="registering-your-app-for-authentication"></a>Registrar la aplicación para la autenticación 

Cuando decida qué proveedor de autenticación cumple los requisitos de su aplicación, deberá registrar su aplicación en el portal de ese proveedor de autenticación. El registro de la aplicación establece su identidad con el proveedor de autenticación y permite que especifique esa identidad al enviar solicitudes de autenticación del usuario.

- Para registrar la aplicación con Azure AD, use [Azure Portal](https://portal.azure.com/).

    <!--For Azure AD, you'll also need to [associate your Office 365 account with Azure AD subscription](../auth_azure_ad/associate_account.md) in order to manage your apps.-->

- Para [registrar su aplicación con el punto de conexión v2.0 de Azure AD](auth_register_app_v2.md), use el [Portal de registro de aplicaciones de Microsoft](https://apps.dev.microsoft.com).


## <a name="resources-for-implementing-authentication-in-your-microsoft-graph-app"></a>Recursos para la implementación de la autenticación en su aplicación de Microsoft Graph 

Después de registrar la aplicación con el portal de autenticación adecuado y de contar con la información de registro de la aplicación (identificador de la aplicación, secreto de aplicación, si procede, y el URI de redireccionamiento) que necesita para establecer la identidad de su aplicación, ya podrá implementar la autenticación en su aplicación. 

Una vez más, esto variará dependiendo del tipo de aplicación que esté compilando, de su plataforma de desarrollo, del flujo de autenticación que elija y de cualquier requisito de autenticación específico para su aplicación. 

### <a name="connect-samples-by-authentication-provider-and-platform"></a>Ejemplos de conexión por proveedor de autenticación y plataforma

En la siguiente tabla se enumeran los ejemplos de conexión por proveedor de autenticación y plataforma y se muestra una nota para indicar si se conectan a Microsoft Graph mediante REST o una biblioteca cliente de Microsoft Graph.

<table>
  <tr>
    <th>Plataforma</th>
    <th>Punto de conexión de Azure AD</th> 
    <th>Punto de conexión v2.0 de Azure AD</th>
  </tr>
  <tr>
    <td>Android</td>
    <td>
        <a href="https://github.com/microsoftgraph/android-java-connect-rest-sample">Ejemplo de REST</a> o <a href="https://github.com/microsoftgraph/android-java-connect-sample/tree/last_v1_auth">ejemplo de SDK</a>
    </td> 
    <td>
        <a href="https://github.com/microsoftgraph/android-java-connect-sample">Ejemplo de SDK</a>
    </td> 
  </tr>
  <tr>
    <td>ASP.NET</td>
    <td>
        <a href="https://github.com/microsoftgraph/aspnet-connect-rest-sample">Ejemplo de REST</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/aspnet-connect-sample">Ejemplo de SDK</a>
    </td> 
  </tr>
  <tr>
    <td>iOS (Obj-C)</td>
    <td>
        <a href="https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample">Ejemplo de REST</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/ios-objectivec-connect-sample">Ejemplo de SDK</a>
    </td> 
  </tr>
  <tr>
    <td>iOS (Swift)</td>
    <td>
        <a href="https://github.com/microsoftgraph/ios-swift-connect-rest-sample">Ejemplo de REST</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/ios-swift-connect-sample">Ejemplo de SDK</a>
    </td> 
  </tr>
  <tr>
    <td>Node.js</td>
    <td>
        <a href="https://github.com/microsoftgraph/nodejs-connect-rest-sample/tree/last_v1_auth">Ejemplo de REST</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/nodejs-connect-rest-sample">Ejemplo de REST</a>
    </td> 
  </tr>
  <tr>
    <td>PHP</td>
    <td>
        <a href="https://github.com/microsoftgraph/php-connect-rest-sample/tree/last_v1_auth">Ejemplo de REST</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/php-connect-rest-sample">Ejemplo de REST</a>
    </td> 
  </tr>
  <tr>
    <td>Python</td>
    <td>
        <a href="https://github.com/microsoftgraph/python3-connect-rest-sample">Ejemplo de REST</a>
    </td>     
    <td>
    </td> 
  </tr>
  <tr>
    <td>Ruby</td>
    <td>
        <a href="https://github.com/microsoftgraph/ruby-connect-rest-sample/tree/last_v1_auth">Ejemplo de REST</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/ruby-connect-rest-sample">Ejemplo de REST</a>
    </td> 
  </tr>
  <tr>
    <td>UWP</td>
    <td>
        <a href="https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample/tree/last_v1_auth">Ejemplo de REST</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample">Ejemplo de REST</a> o <a href="https://github.com/microsoftgraph/uwp-csharp-connect-sample">ejemplo de SDK</a>
    </td> 
  </tr>
  <tr>
    <td>Xamarin</td>
    <td>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/xamarin-csharp-connect-sample">Ejemplo de SDK</a>
    </td> 
  </tr>
</table>

Para explorar una amplia gama de proyectos que se conectan a Microsoft Graph mediante una gran variedad de tecnologías, visite el [repositorio de Microsoft Graph](https://github.com/microsoftgraph) en GitHub. 

### <a name="get-started"></a>Introducción  

En la sección [Introducción](http://graph.microsoft.io/en-us/docs/platform/get-started), se incluyen artículos detallados en los que se muestra cómo crear las aplicaciones que se enumeran en la tabla mediante el punto de conexión v2.0 de Azure AD y se incluyen las bibliotecas de autenticación de cada plataforma. 

## <a name="see-also"></a>Recursos adicionales

- <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-authentication-scenarios/#basics-of-authentication-in-azure-ad" target="_newtab">Escenarios de autenticación de Azure AD</a>

- <a href="https://azure.microsoft.com/en-us/documentation/articles/?product=active-directory&term=v2.0+endpoint" target="_newtab">Documentación acerca del punto de conexión v2.0 de Azure AD en Azure.com</a>
- <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-app-registration/#build-a-quick-start-app" target="_newtab">Inicios rápidos del código v2.0 de Azure AD en Azure.com</a>
