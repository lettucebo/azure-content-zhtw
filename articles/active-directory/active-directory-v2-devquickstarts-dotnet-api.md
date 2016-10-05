<properties
	pageTitle="Azure AD v2.0 .NET Web API| Microsoft Azure"
	description="如何建置 .NET MVC Web 應用程式，以從個人 Microsoft 帳戶及公司或學校帳戶接受權杖。"
	services="active-directory"
	documentationCenter=".net"
	authors="dstrockis"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.date="09/16/2016"
	ms.author="dastrock"/>

# 保護 MVC web API

Azure Active Directory 的 v2.0 端點可讓您使用 [OAuth 2.0](active-directory-v2-protocols.md#oauth2-authorization-code-flow) 存取權杖保護 Web API，具有個人 Microsoft 帳戶以及公司或學校帳戶的使用者，也能夠安全地存取您的 Web API。

> [AZURE.NOTE]
	v2.0 端點並非支援每個 Azure Active Directory 案例和功能。如果要判斷是否應該使用 v2.0 端點，請閱讀 [v2.0 限制](active-directory-v2-limitations.md)。

在 ASP.NET Web API 中，您可以使用隨附於 .NET Framework 4.5 的 Microsoft OWIN 中介軟體來完成此項作業。在這裡，我們將使用 OWIN 組建可讓用戶端透過使用者待辦事項清單建立和讀取工作的「待辦事項清單」MVC Web API。Web API 會驗證傳入的要求是否包含有效的存取權杖，以及拒絕所有受保護路由上未通過驗證的要求。

## 下載
本教學課程的程式碼保留在 [GitHub](https://github.com/AzureADQuickStarts/AppModelv2-WebAPI-DotNet)。若要遵循執行，您可以[用 .zip 格式下載應用程式的基本架構](https://github.com/AzureADQuickStarts/AppModelv2-WebAPI-DotNet/archive/skeleton.zip)，或複製基本架構：

```
git clone --branch skeleton https://github.com/AzureADQuickStarts/AppModelv2-WebAPI-DotNet.git
```

基本架構應用程式包含簡易 API 的所有未定案程式碼，但是不包含所有身分識別相關的部分。如果您不想要跟著步驟自行製作，您可以改為複製或[下載完成的範例](https://github.com/AzureADQuickStarts/AppModelv2-WebAPI-DotNet/archive/skeleton.zip)。

```
git clone https://github.com/AzureADQuickStarts/AppModelv2-WebAPI-DotNet.git
```

## 註冊應用程式
在 [apps.dev.microsoft.com](https://apps.dev.microsoft.com) 建立新的應用程式，或遵循下列[詳細步驟](active-directory-v2-app-registration.md)。請確定：

- 將指派給您應用程式的**應用程式識別碼**複製起來，您很快會需要用到這些識別碼。

本 Visual Studio 方案也包含了「TodoListClient」，這是一個簡單的 WPF 應用程式。TodoListClient 可用來示範使用者如何登入，以及如何向您的 Web API 發出要求。在此案例中，TodoListClient 和 TodoListService 都由相同的應用程式代表。如需設定 TodoListClient，您也應該：

- 為您的應用程式新增**行動**平台。
- 從入口網站複製完整的**重新導向 URI**。您必須使用 `urn:ietf:wg:oauth:2.0:oob` 的預設值。


## 安裝 OWIN

您現在已註冊了應用程式，需要加以設定才能與 the v2.0 端點通訊，以便驗證傳入的要求和權杖。

- 若要開始，請開啟解決方案，並將 OWIN 中介軟體 NuGet 封裝加入至使用封裝管理員主控台的 TodoListService 專案。

```
PM> Install-Package Microsoft.Owin.Security.OAuth -ProjectName TodoListService
PM> Install-Package Microsoft.Owin.Security.Jwt -ProjectName TodoListService
PM> Install-Package Microsoft.Owin.Host.SystemWeb -ProjectName TodoListService
```

## 設定 OAuth 驗證

- 將 OWIN 啟動類別加入至名為 `Startup.cs` 的 TodoListService 專案。以滑鼠右鍵按一下專案 --> [**新增**] -->[ **新增項目**] --> 搜尋 "OWIN"。OWIN 中介軟體將會在應用程式啟動時叫用 `Configuration(…)` 方法。
- 將類別宣告變更為 `public partial class Startup`，我們已為您在另一個檔案中實作了此類別的一部分。在 `Configuration(…)` 方法中，請呼叫 ConfgureAuth(...) 以設定您的 Web 應用程式驗證。

```C#
public partial class Startup
{
    public void Configuration(IAppBuilder app)
    {
        ConfigureAuth(app);
    }
}
```

- 開啟檔案 `App_Start\Startup.Auth.cs` 並實作 `ConfigureAuth(…)` 方法，以便設定 Web API 接受來自 v2.0 端點的權杖。

```C#
public void ConfigureAuth(IAppBuilder app)
{
		var tvps = new TokenValidationParameters
		{
				// In this app, the TodoListClient and TodoListService
				// are represented using the same Application Id - we use
				// the Application Id to represent the audience, or the
				// intended recipient of tokens.

				ValidAudience = clientId,

				// In a real applicaiton, you might use issuer validation to
				// verify that the user's organization (if applicable) has
				// signed up for the app.  Here, we'll just turn it off.

				ValidateIssuer = false,
		};

		// Set up the OWIN pipeline to use OAuth 2.0 Bearer authentication.
		// The options provided here tell the middleware about the type of tokens
		// that will be recieved, which are JWTs for the v2.0 endpoint.

		// NOTE: The usual WindowsAzureActiveDirectoryBearerAuthenticaitonMiddleware uses a
		// metadata endpoint which is not supported by the v2.0 endpoint.  Instead, this
		// OpenIdConenctCachingSecurityTokenProvider can be used to fetch & use the OpenIdConnect
		// metadata document.

		app.UseOAuthBearerAuthentication(new OAuthBearerAuthenticationOptions
		{
				AccessTokenFormat = new Microsoft.Owin.Security.Jwt.JwtFormat(tvps, new OpenIdConnectCachingSecurityTokenProvider("https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration")),
		});
}
```

- 現在，您可以使用 `[Authorize]` 屬性並搭配 OAuth 2.0 承載驗證來保護您的控制器和動作。使用授權標記裝飾 `Controllers\TodoListController.cs` 類別。這樣會強制使用者在存取該頁面之前必須先登入。

```C#
[Authorize]
public class TodoListController : ApiController
{
```

- 當授權的呼叫者成功叫用其中一個 `TodoListController` API 時，此動作可能需要存取呼叫者的相關資訊。OWIN 可讓您透過 `ClaimsPrincpal` 物件存取持有人權杖內部的宣告。  

```C#
public IEnumerable<TodoItem> Get()
{
    // You can use the ClaimsPrincipal to access information about the
    // user making the call.  In this case, we use the 'sub' or
    // NameIdentifier claim to serve as a key for the tasks in the data store.

    Claim subject = ClaimsPrincipal.Current.FindFirst(ClaimTypes.NameIdentifier);

    return from todo in todoBag
           where todo.Owner == subject.Value
           select todo;
}
```

-	最後，請開啟 TodoListService 專案根目錄中的 `web.config` 檔案，並在 [`<appSettings>`] 區段中輸入您的設定值。
  -	您的 `ida:Audience` 是您在入口網站中為應用程式輸入的**應用程式識別碼**。

## 設定用戶端應用程式
在看到 Todo 清單服務運作之前，您需要設定 Todo 清單用戶端，以便它可以取得 v2.0 端點的權杖，並對服務進行呼叫。

- 在 TodoListClient 專案中，開啟 `App.config` 並在 [`<appSettings>`] 區段中輸入您的設定值。
  -	您從入口網站複製的 `ida:ClientId` 應用程式識別碼。
	- `ida:RedirectUri` 是來自入口網站的**重新導向 Uri**。

最後，清除、建置並執行每個專案！ 您現在已有 .NET MVC Web API，可從個人 Microsoft 帳戶及公司或學校帳戶接受權杖。登入 TodoListClient，然後呼叫 web api 以將工作加入至使用者的待辦事項清單。

如需參考，[此處以 .zip 格式提供](https://github.com/AzureADQuickStarts/AppModelv2-WebAPI-DotNet/archive/complete.zip)完整範例 (不含您的組態值)，您也可以從 GitHub 予以複製：

```git clone --branch complete https://github.com/AzureADQuickStarts/AppModelv2-WebAPI-DotNet.git```

## 後續步驟
您現在可以繼續探索其他主題。您可以嘗試：

[從 Web 應用程式呼叫 Web API >>](active-directory-v2-devquickstarts-webapp-webapi-dotnet.md)

如需其他資源，請參閱：
- [v2.0 開發人員指南 >>](active-directory-appmodel-v2-overview.md)
- [StackOverflow "azure-active-directory" 標記 >>](http://stackoverflow.com/questions/tagged/azure-active-directory)

## 取得產品的安全性更新

我們鼓勵您造訪[此頁面](https://technet.microsoft.com/security/dd252948)並訂閱資訊安全摘要報告警示，以在安全性事件發生時收到通知。

<!---HONumber=AcomDC_0921_2016-->