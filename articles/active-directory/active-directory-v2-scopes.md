<properties
	pageTitle="Azure AD v2.0 範圍、權限和同意 | Microsoft Azure"
	description="Azure AD v2.0 端點中的授權說明，包括範圍、權限及同意。"
	services="active-directory"
	documentationCenter=""
	authors="dstrockis"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/16/2016"
	ms.author="dastrock"/>

# v2.0 端點的範圍、權限和同意

與 Azure AD 整合的應用程式會遵循可讓使用者控制應用程式如何存取其資料的特定授權模型。這個授權模型的 v2.0 實作已更新，並變更 app 必須與 Azure AD 互動的方式。本主題涵蓋此授權模型的基本概念，包括範圍、權限及同意。

> [AZURE.NOTE]
	v2.0 端點並非支援每個 Azure Active Directory 案例和功能。若要判斷是否應該使用 v2.0 端點，請閱讀相關的 [v2.0 限制](active-directory-v2-limitations.md)。

## 範圍和權限

Azure AD 會實作 [OAuth 2.0](active-directory-v2-protocols.md) 授權通訊協定，此方法可讓協力廠商 app 代表使用者存取 Web 主控的資源。任何與 Azure AD 整合的 Web 主控資源都會有資源識別碼或**應用程式識別碼 URI**。例如，Microsoft 的 Web 主控資源包括：

- Office 365 整合郵件 API：`https://outlook.office.com`
- Azure AD Graph API：`https://graph.windows.net`
- Microsoft Graph：`https://graph.microsoft.com`

這也適用於已與 Azure AD 整合的任何協力廠商資源。這些資源也可以定義一組可用來將該資源的功能分割成較小區塊的權限。例如，Microsoft Graph 已定義數個權限：

- 讀取使用者的行事曆
- 寫入使用者的行事曆
- 以使用者身分傳送郵件
- [\+ 更多功能](https://graph.microsoft.io)

藉由定義這些權限，資源可以更細微地掌控其資料及如何對外界公開資料的方式。協力廠商 app 接著可以向使用者要求這些權限，而使用者必須先核准權限，app 才可以代表他們執行動作。將資源的功能切割成較小的權限集，即可將協力廠商應用程式建置為只要求他們所需的特定權限，以便執行其職責。它也可讓使用者完全知道應用程式將如何使用其資料，如此他們才會對應用程式的行為不受惡意攻擊影響更具信心

在 Azure AD 和 OAuth 中，這些權限也稱為**範圍**。您也可能會看到它們稱為 **oAuth2Permissions**。在 Azure AD 中範圍會以字串值表示。依據 Microsoft Graph 範例，每個權限的範圍值如下：

- 讀取使用者的行事曆：`Calendar.Read`
- 寫入使用者的行事曆：`Mail.ReadWrite`
- 以使用者身分傳送郵件：`Mail.Send`

如下所述，在對 v2.0 端點的要求中指定範圍，應用程式即可要求這些權限。


## 同意

在 [OpenID Connect 或 OAuth 2.0](active-directory-v2-protocols.md) 授權要求中，應用程式可以使用 `scope` 查詢參數來要求它所需的權限。例如，當使用者登入應用程式時，應用程式會傳送如下所示的要求 (包含分行符號以便讀取)：

```
GET https://login.microsoftonline.com/common/oauth2/v2.0/authorize?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&response_type=code
&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
&response_mode=query
&scope=
https%3A%2F%2Fgraph.microsoft.com%2Fcalendar.read%20
https%3A%2F%2Fgraph.microsoft.com%2Fmail.send
&state=12345
```

`scope` 參數是應用程式所要求的範圍清單 (以空格分隔)。將範圍值附加至資源的識別項 (應用程式識別碼 URI) 可指出每個個別範圍。上述要求指出 app 需要讀取使用者的行事曆，以及以使用者身分傳送郵件的權限。

在使用者輸入其認證之後，v2.0 端點會檢查是否有相符的**使用者同意**記錄。如果使用者未曾同意所要求權限的任何一項，v2.0 端點會要求使用者授與要求的權限。

![公司帳戶同意螢幕擷取畫面](../media/active-directory-v2-flows/work_account_consent.png)

當使用者核准權限時，就會記錄同意，使用者在後續登入時就不需要重新同意。

## 使用權限

在使用者同意您的應用程式的權限之後，您的應用程式即可取得存取權杖，而這些權杖代表您的應用程式存取資源的權限。指定的存取權杖只能用於單一資源，但其內部編碼會是您的應用程式已獲得該資源的特有權限。若要取得存取權杖，您的應用程式可以對 v2.0 權杖端點提出要求：

```
POST common/oauth2/v2.0/token HTTP/1.1
Host: https://login.microsoftonline.com
Content-Type: application/json

{
	"grant_type": "authorization_code",
	"client_id": "6731de76-14a6-49ae-97bc-6eba6914391e",
	"scope": "https://outlook.office.com/mail.read https://outlook.office.com/mail.send",
	"code": "AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq..."
	"redirect_uri": "https://localhost/myapp",
	"client_secret": "zc53fwe80980293klaj9823"  // NOTE: Only required for web apps
}
```

然後，結果產生的存取權杖可使用於資源的 HTTP 要求 - 它會可靠地指出您的應用程式具有適當權限可執行指定工作的資源。

如需 OAuth 2.0 通訊協定以及如何取得存取權杖的詳細資訊，請參閱 [v2.0 端點通訊協定參考](active-directory-v2-protocols.md)。

## OpenId Connect 範圍

OpenID Connect 的 v2.0 實作有一些定義妥善但不會套用至任何特定資源的範圍 - `openid`、`email`、`profile` 和 `offline_access`。

#### OpenId

如果應用程式使用 [OpenID Connect](active-directory-v2-protocols.md#openid-connect-sign-in-flow) 執行登入，則必須要求 `openid` 範圍。`openid` 範圍會在公司帳戶同意畫面上顯示為「讓您登入」權限，而在個人 Microsoft 帳戶同意畫面上顯示為「檢視您的設定檔並使用您的 Microsoft 帳戶連接到應用程式和服務」權限。此權限可讓 app 以 `sub` 宣告的形式接收使用者的唯一識別碼。它也會提供使用者資訊端點的應用程式存取權。`openid` 範圍也可用於 v2.0 權杖端點來取得 id\_token，而該權杖可用來保護應用程式的不同元件之間的 HTTP 呼叫。

#### 電子郵件

`email` 範圍可以連同 `openid` 範圍和任何其他範圍一起納入。它會以 `email` 宣告的形式提供使用者主要電子郵件地址的 app 存取權。如果電子郵件地址與使用者帳戶相關聯 (情況並非永遠如此)，則 `email` 宣告只會包含在權杖中。如果使用 `email` 範圍，您的 app 應該準備好要處理 `email` 宣告不存在於權杖中的情況。

#### 設定檔

`profile` 範圍可以連同 `openid` 範圍和任何其他範圍一起納入。它會提供大量使用者資訊的應用程式存取權。這包括但不限於使用者的名字、姓氏、慣用使用者名稱、物件識別碼等等。如需指定使用者的可用 id\_token 中可用的設定檔宣告完整清單，請參閱 [v2.0 權杖參照](active-directory-v2-tokens.md)。

#### Offline\_access

[`offline_access` 範圍](http://openid.net/specs/openid-connect-core-1_0.html#OfflineAccess)可讓您的 app 代表使用者存取資源的期間延長。在公司帳戶同意畫面上，此範圍會顯示為「隨時存取您的資料」權限。在個人 Microsoft 帳戶同意畫面上，則會顯示為「隨時存取您的資訊」權限。當使用者核准 `offline_access` 範圍時，您的應用程式將會啟用以接收來自 v2.0 權杖端點的重新整理權杖。重新整理權杖屬於長效權杖，可讓您的應用程式在舊的存取權杖過期時取得新的存取權杖。

如果您的應用程式未要求 `offline_access` 範圍，則不會收到 refresh\_token。這表示當您在 [OAuth 2.0 授權碼流程](active-directory-v2-protocols.md#oauth2-authorization-code-flow)中兌換 authorization\_code 時，您只會從 `/token` 端點接收 access\_token。該 access\_token 會短時間維持有效 (通常是一小時)，但最後終將過期。屆時，您的應用程式必須將使用者重新導向回到 `/authorize` 端點以擷取新的 authorization\_code。在此重新導向期間，視應用程式的類型而定，使用者或許不需要再次輸入其認證或重新同意權限。

如需如何取得及使用重新整理權杖的詳細資訊，請參閱 [v2.0 通訊協定參考](active-directory-v2-protocols.md)。

<!---HONumber=AcomDC_0921_2016-->