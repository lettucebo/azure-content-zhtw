<properties
	pageTitle="Azure AD v2.0 通訊協定 | Microsoft Azure"
	description="Azure AD v2.0 端點支援的通訊協定指南。"
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

# v2.0 通訊協定 - OAuth 2.0 與 OpenID Connect

v2.0 端點可以使用 Azure AD，利用業界標準通訊協定 (OpenID Connect 與 OAuth 2.0) 提供身分識別即服務。雖然此服務是標準相容，但這些通訊協定任兩個實作之間仍會有些微差異。若您選擇藉由直接傳送和處理 HTTP 要求，或使用協力廠商的開放原始碼程式庫來撰寫程式碼，而非使用我們的其中一個開放原始碼程式庫，則可參考這裡提供的實用資訊。
<!-- TODO: Need link to libraries above -->

> [AZURE.NOTE]
	v2.0 端點並非支援每個 Azure Active Directory 案例和功能。若要判斷是否應該使用 v2.0 端點，請閱讀相關的 [v2.0 限制](active-directory-v2-limitations.md)。

## 基本概念
幾乎在所有的 OAuth 和 OpenID Connect 流程中，都有四個參與交換的合作對象：

![OAuth 2.0 角色](../media/active-directory-v2-flows/protocols_roles.png)

- **授權伺服器**是 v2.0 端點。其負責確保使用者的身分識別、授與及撤銷資源存取權，以及核發權杖。其也稱作為識別提供者：安全地處理與使用者資訊、使用者存取權，以及流程中合作對象彼此間信任關係有關的任何項目。
- **資源擁有者**通常是使用者。其是擁有資料的一方，而且有權允許第三方存取該資料或資源。
- **OAuth 用戶端**是您的應用程式，透過其應用程式識別碼加以識別。其通常是與使用者互動的對象，而且會向授權伺服器要求權杖。用戶端必須獲得資源擁有者授權才能存取資源。
- **資源伺服器**是資源或資料所在位置。其信任授權伺服器會安全地驗證及授權 OAuth 用戶端，並使用 Bearer access\_token 來確保可以授與資源的存取權。


## App 註冊
所有使用 v2.0 端點的 app 都必須先在 [apps.dev.microsoft.com](https://apps.dev.microsoft.com) 註冊，才能使用 OAuth 或 OpenID Connect 進行互動。應用程式註冊處理序會收集與指派一些值給您的應用程式：

- 可唯一識別應用程式的**應用程式識別碼**
- 可將回應導回至應用程式的**重新導向 URI**或**封裝識別碼**
- 其他幾個狀況特定的值。

如需詳細資訊，請了解如何[註冊應用程式](active-directory-v2-app-registration.md)。

## 端點
註冊完成後，應用程式即會向 v2.0 端點傳送要求以與 Azure AD 通訊：

```
https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize
https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token
```

其中 `{tenant}` 可以接受下列這四個不同值的其中一個：

| 值 | 說明 |
| ----------------------- | ------------------------------- |
| `common` | 允許使用者使用個人的 Microsoft 帳戶和工作/學校帳戶，從 Azure Active Directory 登入應用程式。 |
| `organizations` | 僅允許使用者使用工作/學校帳戶，從 Azure Active Directory 登入應用程式。 |
| `consumers` | 僅允許使用者使用個人的 Microsoft 帳戶 (MSA) 登入應用程式。 |
| `8eaef023-2b34-4da1-9baa-8bc8c9d6a490` 或 `contoso.onmicrosoft.com` | 僅允許使用者使用工作/學校帳戶，從特定的 Azure Active Directory 租用戶登入應用程式。可以使用 Azure AD 租用戶的好記網域名稱或租用戶的 GUID 識別碼。 |

如需如何與這些端點互動的詳細資訊，請選擇下列特定的 app 類型。

## 權杖
OAuth 2.0 和 OpenID Connect 的 v2.0 實作會廣泛運用持有人權杖，包括以 JWT 表示的持有人權杖。持有人權杖是輕巧型的安全性權杖，授權「持有者」存取受保護的資源。從這個意義上說，「持有者」是可出示權杖的任何一方。雖然某一方必須先向 Azure AD 驗證以收到持有人權杖，但如果傳輸和儲存時未採取必要的步驟來保護權杖，它可能會被非預期的一方攔截和使用。雖然某些安全性權杖都有內建的機制，可防止未經授權的人士使用權杖，但持有者權杖沒有這項機制，而必須以安全通道來傳輸，例如傳輸層安全性 (HTTPS)。如果持有人權杖以純文字傳輸，惡意人士可能使用攔截式攻擊來取得權杖，然後未經授權存取受保護的資源。儲存或快取持有者權杖供以後使用時，也適用相同的安全性原則。務必確定您的應用程式以安全的方式傳輸和儲存持有人權杖。關於持有者權杖的其他安全性考量，請參閱 [RFC 6750 第 5 節](http://tools.ietf.org/html/rfc6750)。

如需 v2.0 端點中使用的不同類型權杖的詳細說明，請參閱 [v2.0 權杖參考](active-directory-v2-tokens.md)。

## 通訊協定

若您準備好查看部分範例要求，請開始使用以下的其中一個教學課程。每個教學課程皆對應至特定的驗證案例。若您在判斷正確流程時需要協助，請參閱[您可以使用 v2.0 建置的 app 類型](active-directory-v2-flows.md)。

- [使用 OAuth 2.0 建置行動與原生應用程式](active-directory-v2-protocols-oauth-code.md)
- [使用 OpenID Connect 建置 Web 應用程式](active-directory-v2-protocols-oidc.md)
- [使用 OAuth 2.0 隱含流程建置單一頁面應用程式](active-directory-v2-protocols-implicit.md)
- 使用 OAuth 2.0 用戶端認證流程建置精靈或伺服器端處理程序 (敬請期待)
- 透過 OAuth 2.0 代理者流程在 Web API 中取得權杖 (敬請期待)

<!-- - Get tokens using a username & password with the OAuth 2.0 Resource Owner Password Credentials Flow (coming soon) --> 

<!---HONumber=AcomDC_0921_2016-->