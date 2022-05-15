---
title: ASP.NET Core Blazor Hybrid authentication and authorization
author: guardrex
description: Learn about Blazor Hybrid authentication and authorization scenarios.
monikerRange: '>= aspnetcore-6.0'
ms.author: riande
ms.custom: mvc
ms.date: 05/12/2022
no-loc: [".NET MAUI", "Mac Catalyst", "Blazor Hybrid", Home, Privacy, Kestrel, appsettings.json, "ASP.NET Core Identity", cookie, Cookie, Blazor, "Blazor Server", "Blazor WebAssembly", "Identity", "Let's Encrypt", Razor, SignalR]
uid: blazor/hybrid/security/index
zone_pivot_groups: blazor-hybrid-frameworks
---
# ASP.NET Core Blazor Hybrid authentication and authorization

This article describes ASP.NET Core's support for the configuration and management of security in Blazor Hybrid apps.

[!INCLUDE[](~/blazor/includes/blazor-hybrid-preview-notice.md)]

## Overview

Blazor Hybrid apps that render web content execute .NET code inside a platform :::no-loc text="Web View":::. The .NET code interacts with the web content via an interop channel between the two.

![The WebView and .NET code interoperate within the app to render web content.](~/blazor/hybrid/security/index/_static/figure01.png)

The web content rendered into the :::no-loc text="Web View"::: can come from assets provided by the app from either of the following locations:

* The `wwwroot` folder in the app.
* A source external to the app. For example, a network source, such as the Internet.

A trust boundary exists between the .NET code and the code that runs inside the :::no-loc text="Web View":::. .NET code is provided by the app and any trusted third-party packages that you've installed. After the app is built, the .NET code :::no-loc text="Web View"::: content sources can't change.

In contrast to the .NET code sources of content, content sources from the code that runs inside the :::no-loc text="Web View"::: can come not only from the app but also from external sources. For example, static assets from an external Content Delivery Network (CDN) might be used or rendered by an app's :::no-loc text="Web View":::.

Consider the code inside the :::no-loc text="Web View"::: as **untrusted** in the same way that code running inside the browser for a web app isn't trusted. The same threats and general security recommendations apply to untrusted resources in Blazor Hybrid apps as for other types of apps. If possible, avoid loading content from a third-party origin. To mitigate risk, you might be able to serve content directly from the app by downloading the external assets, verifying that they're safe to serve to users, and placing them into the app's `wwwroot` folder for packaging with the rest of the app.

If your app must reference content from an external origin, we recommended that you use common web security approaches to provide the app with an opportunity to block the content from loading if the content is compromised:

* Serve content securely with TLS/HTTPS.
* Institute a [Content Security Policy (CSP)](https://developer.mozilla.org/docs/Web/HTTP/CSP).
* Perform [subresource integrity](https://developer.mozilla.org/docs/Web/Security/Subresource_Integrity) checks.

Even all of the resources are packed into the app and don't load from any external origin, you still must be concerned about problems in the resources' code that run inside the :::no-loc text="Web View":::, as the resources might have vulnerabilities that could allow [cross-site scripting (XSS)](xref:blazor/security/server/threat-mitigation#cross-site-scripting-xss) attacks.

In general, the Blazor framework protects against XSS by dealing with HTML in safe ways. However, there are programming patterns that allow Razor components to inject raw HTML into rendered output, such as rendering content from an untrusted source, such as directly rendering database data. Any JavaScript library used by the app might manipulate HTML in an unsafe way to inadvertently render unsafe output or deliberately include code that renders malicious unsafe output.

For these reasons, it's best to apply the same protections against XSS that you would apply in a web app. Prevent loading scripts from unknown sources and don't implement potentially unsafe JavaScript features, such as [`eval`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/eval) and other unsafe JavaScript primitives. Establishing a CSP is recommended to control these security risks.

If the code inside the :::no-loc text="Web View"::: is compromised, the code gains access to all of the content inside the :::no-loc text="Web View"::: and might interact with the host via the interop channel. For that reason, any content coming from the :::no-loc text="Web View"::: (events, JS interop) must be treated as **untrusted** and validated in the same way as for other sensitive contexts, such as in a compromised Blazor Server app that can lead to malicious attacks on the host system.

Don't store sensitive information, such as credentials, security tokens, or sensitive user data, in the context of the :::no-loc text="Web View":::, as it makes the information available to an attacker if the :::no-loc text="Web View"::: is compromised. There are safer alternatives, such as handling the sensitive information directly within the native portion of the app.

## Integrate authentication

Authentication in Blazor Hybrid apps is handled by native platform libraries, as they offer enhanced security guarantees that the browser sandbox can't offer. Authentication of native apps uses an OS-specific mechanism or via a federated protocol, such as [OpenID Connect (OIDC)](https://openid.net/connect/). Follow the guidance for the identity provider that you've selected for the app and then further integrate identity with Blazor using the guidance in this article.

Integrating authentication must achieve the following goals for Razor components and services:

* Use the abstractions in the [`Microsoft.AspNetCore.Components.Authorization`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.Authorization) package, such as <xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeView>.
* React to changes in the authentication context.
* Access credentials provisioned by the app from the identity provider, such as access tokens to perform authorized API calls.

After authentication is added to a .NET MAUI, WPF, or Windows Forms app and users are able to log in and log out successfully, integrate authentication with Blazor to make the authenticated user available to Razor components and services. Perform the following steps:

* Reference the [`Microsoft.AspNetCore.Components.Authorization`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.Authorization) package.

  [!INCLUDE[](~/includes/package-reference.md)]

* Implement a custom <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationStateProvider>, which is the abstraction that Razor components use to access information about the authenticated user and to receive updates when the authentication state changes.
* Register the custom authentication state provider in the dependency injection container.

:::zone pivot="maui"

.NET MAUI apps use [Xamarin.Essentials: Web Authenticator](/xamarin/essentials/web-authenticator): The `WebAuthenticator` class allows the app to initiate browser-based authentication flows that listen for a callback to a specific URL registered with the app.

:::zone-end

:::zone pivot="wpf"

WPF apps use the [Microsoft identity platform](/azure/active-directory/develop/) to integrate with Azure Active Directory (AAD) and AAD B2C. For guidance and examples, see the following resources:

* [Sign-in a user with the Microsoft Identity Platform in a WPF Desktop application and call an ASP.NET Core Web API](/samples/azure-samples/active-directory-dotnet-native-aspnetcore-v2/1-desktop-app-calls-web-api/)
* [Add authentication to your Windows (WPF) app](/azure/developer/mobile-apps/azure-mobile-apps/quickstarts/wpf/authentication)
* [Tutorial: Sign in users and call Microsoft Graph in Windows Presentation Foundation (WPF) desktop app](/azure/active-directory/develop/tutorial-v2-windows-desktop)
* [Quickstart: Acquire a token and call Microsoft Graph API from a desktop application](/azure/active-directory/develop/desktop-app-quickstart?pivots=devlang-windows-desktop)
* [Quickstart: Set up sign in for a desktop app using Azure Active Directory B2C](/azure/active-directory-b2c/quickstart-native-app-desktop)
* [Configure authentication in a sample WPF desktop app by using Azure AD B2C](/azure/active-directory-b2c/configure-authentication-sample-wpf-desktop-app)

:::zone-end

:::zone pivot="winforms"

Windows Forms apps use ...

* \[]()
* \[]()

:::zone-end

## Create a custom `AuthenticationStateProvider` without user change updates

If the app authenticates the user immediately after the app launches and the authenticated user remains the same for the entirety of the app lifetime, user change notifications aren't required, and the app only provides information about the authenticated user. In this scenario, the user logs into the app when the app is opened, and the app displays the login screen again after the user logs out. The following `ExternalAuthStateProvider` is an example implementation of a custom <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationStateProvider> for this authentication scenario.

> [!NOTE]
> The following custom <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationStateProvider> doesn't declare a namespace in order to make the code example applicable to any Blazor Hybrid app. However, a best practice is to provide your app's namespace when you implement the example in a production app.

`ExternalAuthStateProvider.cs`:

```csharp
using System.Security.Claims;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Components.Authorization;

public class ExternalAuthStateProvider : AuthenticationStateProvider
{
    private readonly Task<AuthenticationState> authenticationState;

    public ExternalAuthStateProvider(AuthenticatedUser user) => 
        authenticationState = Task.FromResult(new AuthenticationState(user.Principal));

    public override Task<AuthenticationState> GetAuthenticationStateAsync() =>
        authenticationState;
}

public class AuthenticatedUser
{
    public ClaimsPrincipal Principal { get; set; } = new();
}
```

:::zone pivot="maui"

The following steps describe how to:

* Add required namespaces.
* Add the authorization services and Blazor abstractions to the service collection.
* Build the service collection.
* Resolve the `AuthenticatedUser` service to set the authenticated user's claims principal. See your identity provider's documentation for details.
* Return the built host.

In the `MauiProgram.CreateMauiApp` method of `MainWindow.cs`, add namespaces for <xref:Microsoft.AspNetCore.Components.Authorization?displayProperty=fullName> and <xref:System.Security.Claims?displayProperty=fullName>:

```csharp
using Microsoft.AspNetCore.Components.Authorization;
using System.Security.Claims;
```

Remove the following line of code that returns a built `Microsoft.Maui.Hosting.MauiApp`:

```diff
- return builder.Build();
```

Replace the preceding line of code with the following code. Add OpenID/MSAL code to authenticate the user. See your identity provider's documentation for details.

```csharp
builder.Services.AddAuthorizationCore();
builder.Services.AddScoped<AuthenticationStateProvider, ExternalAuthStateProvider>();
builder.Services.AddSingleton<AuthenticatedUser>();
var host = builder.Build();

var authenticatedUser = host.Services.GetRequiredService<AuthenticatedUser>();

/*
Provide OpenID/MSAL code to authenticate the user. See your identity provider's 
documentation for details.

The user is represented by a new ClaimsPrincipal based on a new ClaimsIdentity.
*/
var user = new ClaimsPrincipal(new ClaimsIdentity());

authenticatedUser.Principal = user;

return host;
```

:::zone-end

:::zone pivot="wpf"

The following steps describe how to:

* Add required namespaces.
* Add the authorization services and Blazor abstractions to the service collection.
* Build the service collection and add the built service collection as a resource to the app's `ResourceDictionary`.
* Resolve the `AuthenticatedUser` service to set the authenticated user's claims principal. See your identity provider's documentation for details.
* Return the built host.

In the `MainWindow`'s constructor (`MainWindow.xaml.cs`), add namespaces for <xref:Microsoft.AspNetCore.Components.Authorization?displayProperty=fullName> and <xref:System.Security.Claims?displayProperty=fullName>:

```csharp
using Microsoft.AspNetCore.Components.Authorization;
using System.Security.Claims;
```

Remove the following line of code that adds the built service collection as a resource to the app's `ResourceDictionary`:

```diff
- Resources.Add("services", serviceCollection.BuildServiceProvider());
```

Replace the preceding line of code with the following code. Add OpenID/MSAL code to authenticate the user. See your identity provider's documentation for details.

```csharp
serviceCollection.AddAuthorizationCore();
serviceCollection.AddScoped<AuthenticationStateProvider, ExternalAuthStateProvider>();
serviceCollection.AddSingleton<AuthenticatedUser>();
var services = serviceCollection.BuildServiceProvider();
Resources.Add("services", services);

var authenticatedUser = services.GetRequiredService<AuthenticatedUser>();

/*
Provide OpenID/MSAL code to authenticate the user. See your identity provider's 
documentation for details.

The user is represented by a new ClaimsPrincipal based on a new ClaimsIdentity.
*/
var user = new ClaimsPrincipal(new ClaimsIdentity());

authenticatedUser.Principal = user;
```

:::zone-end

:::zone pivot="winforms"

The following steps describe how to:

* Add required namespaces.
* Add the authorization services and Blazor abstractions to the service collection.
* Build the service collection and add the built service collection to the app's service provider.
* Resolve the `AuthenticatedUser` service to set the authenticated user's claims principal. See your identity provider's documentation for details.

In the `Form1`'s constructor (`Form1.cs`), add namespaces for <xref:Microsoft.AspNetCore.Components.Authorization?displayProperty=fullName> and <xref:System.Security.Claims?displayProperty=fullName>:

```csharp
using Microsoft.AspNetCore.Components.Authorization;
using System.Security.Claims;
```

Remove the following line of code that sets the built service collection to the app's service provider:

```diff
- blazorWebView1.Services = services.BuildServiceProvider();
```

Replace the preceding line of code with the following code. Add OpenID/MSAL code to authenticate the user. See your identity provider's documentation for details.

```csharp
services.AddAuthorizationCore();
services.AddScoped<AuthenticationStateProvider, ExternalAuthStateProvider>();
services.AddSingleton<AuthenticatedUser>();
var serviceCollection = services.BuildServiceProvider();
blazorWebView1.Services = serviceCollection;

var authenticatedUser = serviceCollection.GetRequiredService<AuthenticatedUser>();

/*
Provide OpenID/MSAL code to authenticate the user. See your identity provider's 
documentation for details.

The user is represented by a new ClaimsPrincipal based on a new ClaimsIdentity.
*/
var user = new ClaimsPrincipal(new ClaimsIdentity());

authenticatedUser.Principal = user;
```

:::zone-end

## Create a custom `AuthenticationStateProvider` with user change updates

To update the user while the Blazor app is running, call <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationStateProvider.NotifyAuthenticationStateChanged%2A> within the <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationStateProvider> implementation using ***either*** of the following approaches:

* [Signal an authentication update from outside of the `BlazorWebView`](#signal-an-authentication-update-from-outside-of-the-blazorwebview-option-1))
* [Handle authentication within the `BlazorWebView`](#handle-authentication-within-the-blazorwebview-option-2)

### Signal an authentication update from outside of the `BlazorWebView` (Option 1)

A custom <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationStateProvider> can use a global service to signal an authentication update. We recommend that the service offer an event that the <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationStateProvider> can subscribe to, where the event invokes <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationStateProvider.NotifyAuthenticationStateChanged%2A>.

> [!NOTE]
> The following custom <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationStateProvider> doesn't declare a namespace in order to make the code example applicable to any Blazor Hybrid app. However, a best practice is to provide your app's namespace when you implement the example in a production app.

`ExternalAuthStateProvider.cs`:

```csharp
using System;
using System.Security.Claims;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Components.Authorization;

public class ExternalAuthStateProvider : AuthenticationStateProvider
{
    private AuthenticationState currentUser;

    public ExternalAuthStateProvider(ExternalAuthService service)
    {
        currentUser = new AuthenticationState(service.CurrentUser);

        service.UserChanged += (newUser) =>
        {
            currentUser = new AuthenticationState(newUser);
            NotifyAuthenticationStateChanged(Task.FromResult(currentUser));
        };
    }

    public override Task<AuthenticationState> GetAuthenticationStateAsync() =>
        Task.FromResult(currentUser);
}

public class ExternalAuthService
{
    public event Action<ClaimsPrincipal>? UserChanged;
    private ClaimsPrincipal? currentUser;

    public ClaimsPrincipal CurrentUser
    {
        get { return currentUser ?? new(); }
        set
        {
            currentUser = value;

            if (UserChanged is not null)
            {
                UserChanged(currentUser);
            }
        }
    }
}
```

:::zone pivot="maui"

In the `MauiProgram.CreateMauiApp` method of `MainWindow.cs`, add a namespace for <xref:Microsoft.AspNetCore.Components.Authorization?displayProperty=fullName>:

```csharp
using Microsoft.AspNetCore.Components.Authorization;
```

Add the authorization services and Blazor abstractions to the service collection:

```csharp
builder.Services.AddAuthorizationCore();
builder.Services.AddScoped<AuthenticationStateProvider, ExternalAuthStateProvider>();
builder.Services.AddSingleton<ExternalAuthService>();
```

:::zone-end

:::zone pivot="wpf"

In the `MainWindow`'s constructor (`MainWindow.xaml.cs`), add a namespace for <xref:Microsoft.AspNetCore.Components.Authorization?displayProperty=fullName>:

```csharp
using Microsoft.AspNetCore.Components.Authorization;
```

Add the authorization services and the Blazor abstractions to the service collection:

```csharp
serviceCollection.AddAuthorizationCore();
serviceCollection.AddScoped<AuthenticationStateProvider, ExternalAuthStateProvider>();
serviceCollection.AddSingleton<ExternalAuthService>();
```

:::zone-end

:::zone pivot="winforms"

In the `Form1`'s constructor (`Form1.cs`), add a namespace for <xref:Microsoft.AspNetCore.Components.Authorization?displayProperty=fullName>:

```csharp
using Microsoft.AspNetCore.Components.Authorization;
```

Add the authorization services and Blazor abstractions to the service collection:

```csharp
services.AddAuthorizationCore();
services.AddScoped<AuthenticationStateProvider, ExternalAuthStateProvider>();
services.AddSingleton<ExternalAuthService>();
```

:::zone-end

Wherever the app authenticates a user, resolve the `ExternalAuthService` service:

```csharp
var authService = host.Services.GetRequiredService<ExternalAuthService>();
```

Execute your custom OpenID/MSAL code to authenticate the user. See your identity provider's documentation for details. The authenticated user (`authenticatedUser` in the following example) is a new <xref:System.Security.Claims.ClaimsPrincipal> based on a new <xref:System.Security.Claims.ClaimsIdentity>.

Set the current user to the authenticated user:

```csharp
authService.CurrentUser = authenticatedUser;
```

An alternative to the preceding approach is to set the user's principal on <xref:System.Threading.Thread.CurrentPrincipal?displayProperty=fullName> instead of setting it via a service, which avoids use of the dependency injection container:

```csharp
public class CurrentThreadUserAuthenticationStateProvider : AuthenticationStateProvider
{
    public override Task<AuthenticationState> GetAuthenticationStateAsync() =>
        Task.FromResult(
            new AuthenticationState(Thread.CurrentPrincipal as ClaimsPrincipal ?? 
                new ClaimsPrincipal(new ClaimsIdentity())));
}
```

Using the alternative approach, only authorization services (`.AddAuthorizationCore()`) and `CurrentThreadUserAuthenticationStateProvider` (`.AddScoped<AuthenticationStateProvider, CurrentThreadUserAuthenticationStateProvider>()`) are added to the service collection.

### Handle authentication within the `BlazorWebView` (Option 2)

A custom <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationStateProvider> can include additional methods to trigger log in and log out and update the user.

> [!NOTE]
> The following custom <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationStateProvider> doesn't declare a namespace in order to make the code example applicable to any Blazor Hybrid app. However, a best practice is to provide your app's namespace when you implement the example in a production app.

`ExternalAuthStateProvider.cs`:

```csharp
using System.Security.Claims;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Components.Authorization;

public class ExternalAuthStateProvider : AuthenticationStateProvider
{
    private ClaimsPrincipal currentUser = new ClaimsPrincipal(new ClaimsIdentity());

    public override Task<AuthenticationState> GetAuthenticationStateAsync() =>
        Task.FromResult(new AuthenticationState(currentUser));

    public Task LogInAsync()
    {
        var loginTask = LogInAsyncCore();
        NotifyAuthenticationStateChanged(loginTask);

        return loginTask;

        async Task<AuthenticationState> LogInAsyncCore()
        {
            var user = await LoginWithExternalProviderAsync();
            currentUser = user;

            return new AuthenticationState(currentUser);
        }
    }

    private Task<ClaimsPrincipal> LoginWithExternalProviderAsync()
    {
        /*
            Provide OpenID/MSAL code to authenticate the user. See your identity 
            provider's documentation for details.

            Return a new ClaimsPrincipal based on a new ClaimsIdentity.
        */
        var authenticatedUser = new ClaimsPrincipal(new ClaimsIdentity());

        return Task.FromResult(authenticatedUser);
    }

    public void Logout()
    {
        currentUser = new ClaimsPrincipal(new ClaimsIdentity());
        NotifyAuthenticationStateChanged(
            Task.FromResult(new AuthenticationState(currentUser)));
    }
}
```

In the preceding example:

* The call to `LogInAsyncCore` triggers the login process.
* The call to <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationStateProvider.NotifyAuthenticationStateChanged%2A> notifies that an update is in progress, which allows the app to provide a temporary UI during the login or logout process.
* Returning `loginTask` returns the task so that the component that triggered the login can await and react after the task is complete.
* The `LoginWithExternalProviderAsync` method is implemented by the developer to log in the user with the identity provider's SDK. For more information, see your identity provider's documentation. The authenticated user (`authenticatedUser`) is a new <xref:System.Security.Claims.ClaimsPrincipal> based on a new <xref:System.Security.Claims.ClaimsIdentity>.

:::zone pivot="maui"

In the `MauiProgram.CreateMauiApp` method of `MainWindow.cs`, add the authorization services and the Blazor abstraction to the service collection:

```csharp
builder.Services.AddAuthorizationCore();
builder.Services.AddScoped<AuthenticationStateProvider, ExternalAuthStateProvider>();
```

:::zone-end

:::zone pivot="wpf"

In the `MainWindow`'s constructor (`MainWindow.xaml.cs`), add the authorization services and the Blazor abstraction to the service collection:

```csharp
serviceCollection.AddAuthorizationCore();
serviceCollection.AddScoped<AuthenticationStateProvider, ExternalAuthStateProvider>();
```

:::zone-end

:::zone pivot="winforms"

In the `Form1`'s constructor (`Form1.cs`), add the authorization services and the Blazor abstraction to the service collection:

```csharp
services.AddAuthorizationCore();
services.AddScoped<AuthenticationStateProvider, ExternalAuthStateProvider>();
```

:::zone-end

The following `LoginComponent` component demonstrates how to log in a user. In a typical app, the `LoginComponent` component is only shown in a parent component if the user isn't logged into the app.

`Shared/LoginComponent.razor`:

```razor
@inject AuthenticationStateProvider AuthenticationStateProvider

<button @onclick="Login">Log in</button>

@code
{
    public async Task Login()
    {
        await ((ExternalAuthStateProvider)AuthenticationStateProvider)
            .LoginAsync();
    }
}
```

The following `LogoutComponent` component demonstrates how to log out a user. In a typical app, the `LogoutComponent` component is only shown in a parent component if the user is logged into the app.

`Shared/LogoutComponent.razor`:

```razor
@inject AuthenticationStateProvider AuthenticationStateProvider

<button @onclick="Logout">Log out</button>

@code
{
    public async Task Logout()
    {
        await ((ExternalAuthStateProvider)AuthenticationStateProvider)
            .Logout();
    }
}
```

## Accessing other authentication information

Blazor doesn't define an abstraction to deal with other credentials, such as access tokens to use for HTTP requests to web APIs. We recommend following the identity provider's guidance to manage the user's credentials with the primitives that the identity provider's SDK provides.

It's common for identity provider SDKs to use a token store for user credentials stored in the device. If the SDK's token store primitive is added to the service container, consume the SDK's primitive within the app.

The Blazor framework isn't aware of a user's authentication credentials and doesn't interact with credentials in any way, so the app's code is free to follow whatever approach you deem most convenient. However, follow the general security guidance in the next section, [Other authentication security considerations](#other-authentication-security-considerations), when implementing authentication code in an app.

## Other authentication security considerations

The authentication process is external to Blazor, and we recommend that developers access the identity provider's guidance for additional security guidance.

When implementing authentication:

* Avoid authentication in the context of the :::no-loc text="Web View":::. For example, avoid using a JavaScript OAuth library to perform the authentication flow. In a single-page app, authentication tokens aren't hidden in JavaScript and can be easily discovered by malicious users and used for nefarious purposes. Native apps don't suffer this risk because native apps are only able to obtain tokens outside of the browser context, which means that rogue third-party scripts can't steal the tokens and compromise the app.
* Avoid implementing the authentication workflow yourself. In most cases, platform libraries securely handle the authentication workflow, using the system's browser instead of using a custom :::no-loc text="Web View"::: that can be hijacked.
* Avoid using the platform's :::no-loc text="Web View"::: control to perform authentication. Instead, rely on the system's browser when possible.
* Avoid passing the tokens to the document context (JavaScript). In some situations, a JavaScript library within the document is required to perform an authorized call to an external service. Instead of making the token available to JavaScript via JS interop:
  * Provide a generated temporary token to the library and within the :::no-loc text="Web View":::.
  * Intercept the outgoing network request in code.
  * Replace the temporary token with the real token and confirm that the destination of the request is valid.

## Untrusted and unencoded content

Avoid allowing an app render untrusted and unencoded content from a database or other resource in its rendered UI, such as user-provided comments. Permitting untrusted, unencoded content to render can cause malicious code to execute.

## External content rendered in an `iframe`

When using an [`iframe`](https://developer.mozilla.org/docs/Web/HTML/Element/iframe) to display external content within a Blazor Hybrid page, we recommend that users leverage [sandboxing features](https://developer.mozilla.org/docs/Web/HTML/Element/iframe) to ensure that the content is isolated from the parent page containing the app. In the following example, the [`sandbox` attribute](https://developer.mozilla.org/docs/Web/HTML/Element/iframe) is present for the `<iframe>` tag to apply sandboxing features to the `foo.html` page:

```html
<iframe sandbox src="https://contoso.com/foo.html" />
```

> [!WARNING]
> The [`sandbox` attribute](https://developer.mozilla.org/docs/Web/HTML/Element/iframe) is ***not*** supported in early browser versions. For more information, see [Can I use: `sandbox`](https://caniuse.com/?search=sandbox).

## Links to external URLs

By default, links to URLs outside of the app are opened in an appropriate external app, not loaded within the :::no-loc text="Web View":::. We do ***not*** recommend overriding the default behavior.

The user might be able to indicate that they want the URL to load in the app because it's content that they trust. In that case, see the [Untrusted and unencoded content](#untrusted-and-unencoded-content) section.

## Keep the :::no-loc text="Web View"::: current deployed apps

By default, the [`BlazorWebView`](/maui/user-interface/controls/blazorwebview) control uses the currently-installed, platform-specific native :::no-loc text="Web View":::. Since the native :::no-loc text="Web View"::: is periodically updated with support for new APIs and fixes for security issues, it may be necessary to ensure that an app is using a :::no-loc text="Web View"::: version that meets the app's requirements.

Use one of the following approaches to keep the :::no-loc text="Web View"::: current in deployed apps:

* **On all platforms**: Check the :::no-loc text="Web View"::: version and prompt the user to take any necessary steps to update it.
* **Only on Windows**: Package a fixed-version :::no-loc text="Web View"::: within the app, using it in place of the system's shared :::no-loc text="Web View":::.

### Android

The Android :::no-loc text="Web View"::: is distributed and updated via the [Google Play Store](https://play.google.com/store/apps/details?id=com.google.android.webview). Check the :::no-loc text="Web View"::: version by reading the [`User-Agent`](https://developer.mozilla.org/docs/Web/HTTP/Headers/User-Agent) string. Read the :::no-loc text="Web View":::'s [`navigator.userAgent`](https://developer.mozilla.org/docs/Web/API/Navigator/userAgent) property using [JavaScript interop](xref:blazor/js-interop/index) and optionally cache the value using a singleton service if the user agent string is required outside of a Razor component context.

### iOS/Mac Catalyst

iOS and Mac Catalyst both use [`WKWebView`](https://developer.apple.com/documentation/webkit/wkwebview), a Safari-based control, which is updated by the operating system. Similar to the [Android](#android) case, determine the :::no-loc text="Web View"::: version by reading the :::no-loc text="Web View":::'s [`User-Agent`](https://developer.mozilla.org/docs/Web/HTTP/Headers/User-Agent) string.

### Windows (.NET MAUI, WPF, Windows Forms)

On Windows, the Chromium-based [Microsoft Edge `WebView2`](/microsoft-edge/webview2/) is required to run Blazor web apps.

By default, the newest installed version of `WebView2`, known as the *:::no-loc text="Evergreen distribution":::*, is used. If you wish to ship a specific version of `WebView2` with the app, use the *:::no-loc text="Fixed Version distribution":::*.

For more information on checking the currently-installed `WebView2` version and the distribution modes, see the [`WebView2` distribution documentation](/microsoft-edge/webview2/concepts/distribution).

## Additional resources

* <xref:blazor/security/index>
