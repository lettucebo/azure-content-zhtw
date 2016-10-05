<properties
	pageTitle="開始使用 Azure AD Xamarin | Microsoft Azure"
	description="如何建置 Xamarin 應用程式來與 Azure AD 整合進行登入，並使用 OAuth 呼叫受 Azure AD 保護的 API。"
	services="active-directory"
	documentationCenter="xamarin"
	authors="dstrockis"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="mobile-xamarin"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.date="09/16/2016"
	ms.author="dastrock"/>


# 將 Azure AD 整合至 Xamarin 應用程式

[AZURE.INCLUDE [active-directory-devquickstarts-switcher](../../includes/active-directory-devquickstarts-switcher.md)]

[AZURE.INCLUDE [active-directory-devguide](../../includes/active-directory-devguide.md)]

Xamarin 可讓您使用 C# 撰寫可在 iOS、Android 和 Windows (行動裝置和電腦) 上執行的行動應用程式。如果您正在建置使用 Xamarin 的應用程式，Azure AD 可讓您簡單又直截了當以 Active Directory 帳戶驗證您的使用者。它也可讓您的應用程式安全地使用任何受 Azure AD 保護的 Web API，例如 Office 365 API 或 Azure API。

對於需要存取受保護資源的 Xamarin 應用程式，Azure AD 提供 Active Directory 驗證程式庫 (ADAL)。ADAL 存在的唯一目的是為了讓您的應用程式輕鬆取得存取權杖。為了示範究竟有多麼簡單，我們將建置一個可執行下列動作的「目錄搜尋程式」應用程式：

-	可在 iOS、Android、Windows 桌面、Windows Phone 和 Windows 市集上執行。
- 使用單一可攜式類別庫 (PCL) 來驗證使用者並取得 Azure AD Graph API 的權杖
-	搜尋目錄以尋找具有指定 UPN 的使用者。

若要建立完整可用的應用程式，您必須：

2. 設定您的 Xamarin 開發環境
2. 向 Azure AD 註冊您的應用程式。
3. 安裝及設定 ADAL。
5. 使用 ADAL 來取得 Azure AD 的權杖。

若要開始使用，請[下載基本架構專案](https://github.com/AzureADQuickStarts/NativeClient-MultiTarget-DotNet/archive/skeleton.zip)或[下載完整的範例](https://github.com/AzureADQuickStarts/NativeClient-MultiTarget-DotNet/archive/complete.zip)。每一個都是 Visual Studio 2013 解決方案。您還需要一個可以建立使用者並註冊應用程式的 Azure AD 租用戶。如果您還沒有租用戶，[了解如何取得租用戶](active-directory-howto-tenant.md)。

## *0.設定您的 Xamarin 開發環境*
因為本教學課程包含 iOS、Android 和 Windows 專案，所以您需要 Visual Studio 和 Xamarin。若要建立必要的環境，請依照 MSDN 上 Visual Studio 和 Xamarin [設定和安裝](https://msdn.microsoft.com/library/mt613162.aspx)的完整指示。這些指示包含的資料，可供您於等待安裝程式完成時檢閱，以深入了解 Xamarin。

完成必要設定之後，您可以在 Visual Studio 中開啟解決方案並開始使用。您會看到六個專案：五個平台特定專案和一個可跨所有平台共用的可攜式類別庫，`DirectorySearcher.cs`

## *1.註冊目錄搜尋應用程式*
若要啟用您的應用程式以取得權杖，您必須先在 Azure AD 租用戶中進行註冊，並對其授與存取 Azure AD Graph API 的權限：

-	登入 [Azure 管理入口網站](https://manage.windowsazure.com)
-	在左側導覽中按一下 **Active Directory**
-	選取要在其中註冊應用程式的租用戶。
-	按一下 [**應用程式**] 索引標籤，然後按一下最下面抽屜的 [**新增**]。
-	遵照提示進行，並建立新的**原生用戶端應用程式**。
    -	應用程式的 [**名稱**] 將對使用者說明您的應用程式
    -	「**重新導向 Uri**」是配置和字串的組合，Azure AD 可用它來傳回權杖回應。輸入值，例如 `http://DirectorySearcher`。
-	完成註冊後，AAD 會為您的應用程式指派一個唯一用戶端識別碼。您在後續章節中將會用到這個值，所以請從 [**設定**] 索引標籤中複製此值。
- 此外，在 [**設定**] 索引標籤上找到 [其他應用程式的權限] 區段。在 [**委派權限**] 下，為 [Azure Active Directory] 應用程式新增 [**存取您的組織目錄**] 權限。這樣做可讓您的應用程式查詢 Graph API 的使用者。

## *2.安裝及設定 ADAL*
既然您在 Azure AD 中已經擁有應用程式，您可以安裝 ADAL，並撰寫身分識別相關程式碼。為了讓 ADAL 能與 Azure AD 通訊，您需要提供一些應用程式註冊的相關資訊。
-	從使用套件管理器主控台將 ADAL 加入方案的每一個專案來開始。

`
PM> Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory -ProjectName DirectorySearcherLib
`

`
PM> Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory -ProjectName DirSearchClient-Android
`

`
PM> Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory -ProjectName DirSearchClient-Desktop
`

`
PM> Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory -ProjectName DirSearchClient-iOS
`

`
PM> Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory -ProjectName DirSearchClient-Universal
`

- 您應該會注意到每個專案會新增兩個程式庫參考：ADAL 的 PCL 部份和平台特定部分。

-	在 DirectorySearcherLib 專案中，開啟 `DirectorySearcher.cs`。變更類別成員值，以反映您在 Azure 入口網站中所輸入的值。每當使用 ADAL 時，您的程式碼便會參考這些值。
    -	`tenant` 是指您的 Azure AD 租用戶網域，例如 contoso.onmicrosoft.com
    -	`clientId` 是指您從入口網站複製的應用程式 clientId。
    - `returnUri` 是指您在入口網站中輸入的 redirectUri，例如 `http://DirectorySearcher`。

## *3.使用 ADAL 來取得 AAD 的權杖*
*幾乎*所有應用程式的驗證邏輯採用 `DirectorySearcher.SearchByAlias(...)`。平台特定專案所需的只是將內容參數傳遞至 `DirectorySearcher` PCL。

- 首先，開啟 `DirectorySearcher.cs` 並在 `SearchByAlias(...)` 方法中加入新的參數。`IPlatformParameters` 是個內容參數，其中包含 ADAL 必須執行驗證的平台特定物件。

```C#
public static async Task<List<User>> SearchByAlias(string alias, IPlatformParameters parent)
{
```

-	接下來，初始化 `AuthenticationContext` - ADAL 的主要類別。這裡是您將與 Azure AD 通訊所需座標傳遞給 ADAL 的位置。然後呼叫 `AcquireTokenAsync(...)`，它可接受 `IPlatformParameters` 物件，並叫用將權杖傳回應用程式所需的驗證流程。

```C#
...
    AuthenticationResult authResult = null;
    try
    {
        AuthenticationContext authContext = new AuthenticationContext(authority);
        authResult = await authContext.AcquireTokenAsync(graphResourceUri, clientId, returnUri, parent);
    }
    catch (Exception ee)
    {
        results.Add(new User { error = ee.Message });
        return results;
    }
...
```
- `AcquireTokenAsync(...)` 會先嘗試為要求的資源 (在此範例中會是 Graph API) 傳回權杖，而不提示使用者輸入其認證 (透過快取或重新整理舊的權杖)。只有在必要時，它才會在取得要求的權杖之前，對使用者顯示 Azure AD 登入頁面。


- 您可以接著將存取權杖附加至 Graph API 要求的授權標頭中：

```C#
...
    request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", authResult.AccessToken);
...
```

`DirectorySearcher` PCL 和應用程式的身分識別相關程式碼到此告一段落。剩下的工作只是在每個平台檢視中呼叫 `SearchByAlias(...)` 方法，並在必要的地方加入程式碼以便正確地處理 UI 生命週期。

####Android：
- 在 `MainActivity.cs` 中，在按鈕點擊處理常式中新增 `SearchByAlias(...)` 的呼叫：

```C#
List<User> results = await DirectorySearcher.SearchByAlias(searchTermText.Text, new PlatformParameters(this));
```
- 您還必須覆寫 `OnActivityResult` 生命週期方法，將任何驗證重新導向轉送回到適當的方法。ADAL 針對 Android 中的此項作業提供協助程式方法：

```C#
...
protected override void OnActivityResult(int requestCode, Result resultCode, Intent data)
{
    base.OnActivityResult(requestCode, resultCode, data);
    AuthenticationAgentContinuationHelper.SetAuthenticationAgentContinuationEventArgs(requestCode, resultCode, data);
}
...
```

####Windows 桌面：
- 在 `MainWindow.xaml.cs` 中，只需進行 `SearchByAlias(...)` 呼叫，即可在桌面的 `PlatformParameters` 物件中傳遞 `WindowInteropHelper`：

```C#
List<User> results = await DirectorySearcher.SearchByAlias(
  SearchTermText.Text,
  new PlatformParameters(PromptBehavior.Auto, this.Handle));
```

####iOS：
- 在 `DirSearchClient_iOSViewController.cs` 中，iOS `PlatformParameters` 物件只需參考檢視控制器：

```C#
List<User> results = await DirectorySearcher.SearchByAlias(
  SearchTermText.Text,
  new PlatformParameters(PromptBehavior.Auto, this.Handle));
```

####Windows Universal：
- 在 Windows Universal 中，開啟 `MainPage.xaml.cs` 並實作 `Search` 方法，這會視需要在共用專案中使用協助程式方法來更新 UI。

```C#
...
    List<User> results = await DirectorySearcherLib.DirectorySearcher.SearchByAlias(SearchTermText.Text, new PlatformParameters(PromptBehavior.Auto, false));
...
```

恭喜！ 您現在擁有一個能夠在五個不同平台之間驗證使用者並使用 OAuth 2.0 安全地呼叫 Web API 的運作中 Xamarin 應用程式。如果您還沒有這麼做，現在正是將一些使用者植入租用戶的好時機。執行 DirectorySearcher 應用程式，並使用其中一個使用者登入。根據 UPN 搜尋其他使用者。

ADAL 可讓您輕鬆地將常見的身分識別功能納入您的應用程式。它會為您處理一切麻煩的事，包括快取管理、OAuth 通訊協定支援、向使用者顯示登入 UI、重新整理過期權杖等等。您唯一需要知道的就是單一 API 呼叫，`authContext.AcquireToken*(…)`。

[這裡](https://github.com/AzureADQuickStarts/NativeClient-MultiTarget-DotNet/archive/complete.zip)提供完成的範例供您參考 (不含您的設定值)。您現在可以繼續其他身分識別案例。您可以嘗試：

[使用 Azure AD 保護 .NET Web API >>](active-directory-devquickstarts-webapi-dotnet.md)

[AZURE.INCLUDE [active-directory-devquickstarts-additional-resources](../../includes/active-directory-devquickstarts-additional-resources.md)]

<!---HONumber=AcomDC_0921_2016-->