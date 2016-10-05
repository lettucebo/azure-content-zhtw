<properties
	pageTitle="應用程式模型 v2.0 概觀 | Microsoft Azure"
	description="建置具備 Microsoft 帳戶和 Azure Active Directory 登入之應用程式的簡介。"
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

# 以單一 App 登入 Microsoft 帳戶和 Azure AD 使用者

在過去，想要同時支援 Microsoft 帳戶和 Azure Active Directory 的應用程式開發人員必須整合這兩個不同的系統。我們現在將介紹新的驗證 API 版本，可讓您透過 Azure AD 系統，使用這兩種帳戶類型登入使用者。這個聚合式驗證系統稱為 **v2.0 端點**。利用 v2.0 端點，一個簡單的整合便可讓您觸達橫跨個人和工作/學校帳戶的數百萬名使用者對象。

使用 v2.0 端點的 app 也可以利用這其中一種帳戶，從 [Microsoft Graph](https://graph.microsoft.io) 和 [Office 365](https://msdn.microsoft.com/office/office365/howto/authenticate-Office-365-APIs-using-v2) 取用 REST API。

<!-- For a quick introduction to the v2.0 endpoint, please view the [Getting Started with Microsoft Identities: Enterprise Grade Sign In For Your Apps](https://azure.microsoft.com/documentation/videos/build-2016-getting-started-with-microsoft-identities-enterprise-grade-sign-in-for-your-apps/) video. -->

## 開始使用
[AZURE.VIDEO build-2016-getting-started-with-microsoft-identities-enterprise-grade-sign-in-for-your-apps]

選擇下列您最愛的平台，使用我們的開放原始碼程式庫與架構來建置 app。或者，您可以使用 OAuth 2.0 和 OpenID Connect 通訊協定文件，直接傳送及接收通訊協定訊息，而不使用驗證庫。

<!-- TODO: Finalize this table  -->
[AZURE.INCLUDE [active-directory-v2-quickstart-table](../../includes/active-directory-v2-quickstart-table.md)]

## 新功能
當您想要了解 v2.0 端點可執行哪些動作以及不可執行哪些動作時，此處提供的概念性資訊非常實用。

- 如果您是在 v2.0 端點 2015 預覽期間建置應用程式，請務必[閱讀我們先前所做的重大通訊協定變更](active-directory-v2-preview-oidc-changes.md)。
- 了解[您可以使用 v2.0 端點建置的應用程式類型](active-directory-v2-flows.md)。
- 若是熟悉 Azure Active Directory 的開發人員，您應該查看 [v2.0 端點中的通訊協定更新和差異](active-directory-v2-compare.md)。
- 了解使用 v2.0 端點的[限制和條件約束](active-directory-v2-limitations.md)。

## 參考
下列連結有助於深入探索平台：

- 組建 2016：[Getting Started with Microsoft Identities: Enterprise Grade Sign In For Your Apps](https://azure.microsoft.com/documentation/videos/build-2016-getting-started-with-microsoft-identities-enterprise-grade-sign-in-for-your-apps/) (開始使用 Microsoft 身分識別︰應用程式的企業級登入)
- 取得使用 [azure-active-directory](http://stackoverflow.com/questions/tagged/azure-active-directory) 或 [adal](http://stackoverflow.com/questions/tagged/adal) 標記之堆疊溢位的相關說明。
- [v2.0 通訊協定參考](active-directory-v2-protocols.md)
- [v2.0 權杖參考](active-directory-v2-tokens.md)
- [v2.0 端點的範圍和同意](active-directory-v2-scopes.md)
- [Microsoft App 註冊入口網站](https://apps.dev.microsoft.com)
- [Office 365 REST API 參考](https://msdn.microsoft.com/office/office365/howto/authenticate-Office-365-APIs-using-v2)
- [Microsoft Graph](https://graph.microsoft.io)
- 以下是已通過 v2.0 端點測試的開放原始碼用戶端程式庫和範例。

  - [Java WSO2 身分識別伺服器](https://docs.wso2.com/display/IS500/Introducing+the+Identity+Server)
  - [Java Gluu 同盟](https://github.com/GluuFederation/oxAuth)
  - [Node.Js passport openidconnect](https://www.npmjs.com/package/passport-openidconnect)
  - [PHP OpenID Connect 基本用戶端](https://github.com/jumbojett/OpenID-Connect-PHP)
  - [iOS OAuth2 用戶端](https://github.com/nxtbgthng/OAuth2Client)
  - [Android OAuth2 用戶端](https://github.com/wuman/android-oauth-client)
  - [Android OpenID Connect 用戶端](https://github.com/kalemontes/OIDCAndroidLib)

<!---HONumber=AcomDC_0921_2016-->