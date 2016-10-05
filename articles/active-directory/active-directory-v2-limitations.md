<properties
	pageTitle="v2.0 端點限制 | Microsoft Azure"
	description="Azure AD v2.0 端點的限制清單。"
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

# 我應該使用 v2.0 端點嗎？

當建立與 Azure Active Directory 整合的應用程式時，您必須判斷 v2.0 端點和驗證通訊協定是否符合您的需求。仍然完整支援原始的 Azure AD 應用程式模型，而且在某些方面比 v2.0 的功能更豐富。不過，v2.0 端點為開發人員[帶來極大的好處](active-directory-v2-compare.md)， 可能吸引您採用新的程式設計模型。隨著時間經過，v2.0 會成長到含括所有的 Azure AD 功能，屆時您只需要使用 v2.0 端點即可。

目前，v2.0 端點可讓您達成兩項核心功能：

- 讓使用者以個人帳戶和公司帳戶登入。
- 呼叫[融合的 Outlook API](https://dev.outlook.com)。

這兩種功能在 Web、行動及個人電腦應用程式中的實作方式都一樣。如果這些相當有限的功能對您的應用程式有意義，我們建議您使用 v2.0 端點。如果您的應用程式需要 Microsoft 服務的任何其他功能，我們建議您繼續使用 Azure AD 的實際可行端點和 Microsoft 帳戶。未來，v2.0 端點將兼容 Azure AD 和 Microsoft 帳戶，我們將協助所有開發人員轉換至 v2.0 端點。

在此同時，本文旨在協助您判斷 v2.0 端點是否適合您。我們將持續更新本文，以反映 v2.0 端點目前的狀態，請記得回來重新評估您對於 v2.0 功能的需求。

如果您有一個與 Azure AD 搭配的現有應用程式未使用 v2.0 端點，則不需要從頭開始。未來，我們設法讓您的現有 Azure AD 應用程式可以使用 v2.0 端點。

## 應用程式的限制
v2.0 端點目前不支援下列類型的應用程式。如需受支援的應用程式類型描述，請參閱[這篇文章](active-directory-v2-flows.md)。

##### 獨立的 Web API
在 v2.0 端點中，您能夠[建置使用 OAuth 2.0 保護的 Web API](active-directory-v2-flows.md#web-apis)。不過，該 Web API 只能從共用相同應用程式識別碼的應用程式接收權杖。不支援建置 Web API 供具有不同應用程式識別碼的用戶端存取。該用戶端無法要求或取得您的 Web API 的權限。

若要了解如何建置 Web API 以接受具有相同應用程式識別碼的用戶端傳回的權杖，請參閱[使用者入門](active-directory-appmodel-v2-overview.md#getting-started)一節中的 v2.0 端點 Web API 範例。

##### 精靈/伺服器端應用程式
包含長時執行處理序或不需要使用者操作的應用程式，也需他法存取受保護的資源，例如 Web API。這些應用程式可以透過 OAuth 2.0 用戶端認證流程，利用應用程式身分識別 (而非使用者委派身分識別) 驗證及取得權杖。

v2.0 端點目前不支援此流程，也就是說應用程式只能在互動式使用者登入流程之後取得權杖。近期內即會加入用戶端認證流程。如果想要查看使用原始 Azure AD 端點的用戶端認證流程，請參閱 [GitHub 上的精靈範例](https://github.com/AzureADSamples/Daemon-DotNet)。

##### Web API 代理者流程
許多架構包含需要呼叫另一個下游 Web API 的 Web API，而這兩者都受到 v2.0 端點的保護。此情況常見於有 Web API 後端的原生用戶端，而後端會再呼叫 Microsoft Online 服務或另一個支援 Azure AD 的自訂建置 Web API。

使用 OAuth 2.0 Jwt 持有人認證授與可支援此案例，亦稱為「代理者流程」。不過，v2.0 端點目前不支援「代理者」流程。若要查看此流程在正式運作的 Azure AD 服務中如何運作，請參閱 [GitHub 上的代理者程式碼範例](https://github.com/AzureADSamples/WebAPI-OnBehalfOf-DotNet)。

## 應用程式註冊的限制
目前，所有想要與 v2.0 端點整合的應用程式，都必須在 [apps.dev.microsoft.com](https://apps.dev.microsoft.com) 建立新的應用程式註冊。任何現有的 Azure AD 或 Microsoft 帳戶應用程式都與 v2.0 端點不相容，除新的應用程式註冊入口網站以外，在任何入口網站中註冊的應用程式也不相容。我們打算設法將現有的應用程式當做 v2.0 應用程式來使用，但目前還沒有將應用程式移轉至 v2.0 端點的途徑。

同樣地，在新的應用程式註冊入口網站中註冊的應用程式，將無法配合原始 Azure AD 驗證端點來運作。不過，您可以使用在應用程式註冊入口網站中建立的應用程式，成功地與 Microsoft 帳戶驗證端點 `https://login.live.com` 整合。

在新的應用程式註冊入口網站中註冊的應用程式，目前僅限一組受限的 redirect\_uri 值。Web 應用程式和服務的 redirect\_uri 必須以配置或 `https` 開頭，而所有其他平台的 redirect\_uri 則必須使用 `urn:ietf:oauth:2.0:oob` 的硬式編碼值。

## 重新導向 URI 的限制
對於 Web 應用程式，redirect\_uri 值必須全部共用單一 DNS 網域。例如，不可以登錄具有具有 redirect\_uri 的 Web 應用程式︰

`https://login-east.contoso.com` `https://login-west.contoso.com`

註冊系統會比較現有 redirect\_uri 的完整 DNS 名稱與您要新增的 redirect\_uri 的 DNS 名稱。如果新 redirect\_uri 的完整 DNS 名稱與現有 redirect\_uri 的 DNS 名稱不完全相符，或如果新 redirect\_uri 的完整 DNS 名稱不是現有 redirect\_uri 的子網域，則新增要求將會失敗。例如，如果應用程式目前有 redirect\_uri：

`https://login.contoso.com`

然後可以新增︰

`https://login.contoso.com/new`

這完全符合 DNS 名稱，或︰

`https://new.login.contoso.com`

這是 login.contoso.com 的 DNS 子網域。如果您希望應用程式以 login-east.contoso.com 和 login-west.contoso.com 做為 redirect\_uri，您必須依序新增下列 redirect\_uri︰

`https://contoso.com` `https://login-east.contoso.com` `https://login-west.contoso.com`

可以新增後面兩個，因為它們是第一個 redirect\_uri (contoso.com) 的子網域。即將推出的版本將會移除這項限制。

若要了解如何在新的應用程式註冊入口網站中註冊應用程式，請參閱[這篇文章](active-directory-v2-app-registration.md)。

## 服務和 API 的限制
v2.0 端點目前支援登入所有在新的應用程式註冊入口網站註冊的應用程式，如[支援的驗證流程](active-directory-v2-flows.md)清單所列。不過，這些應用程式將只能取得 OAuth 2.0 存取權杖，獲得非常有限的資源。v2.0 端點只會為下列項目發行 access\_token：

- 要求權杖的應用程式。如果邏輯應用程式是由數個不同的元件或層級組成，應用程式就可以為自己取得 access\_token。若要查看此案例的執行狀況，請參閱[使用者入門](active-directory-appmodel-v2-overview.md#getting-started)教學課程。
- Outlook 郵件、行事曆和連絡人 REST API，全都位於 https://outlook.office.com。若要了解如何撰寫存取這些 API 的應用程式，請參閱 [Office 使用者入門](https://www.msdn.com/office/office365/howto/authenticate-Office-365-APIs-using-v2)教學課程。
- Microsoft Graph API。若要了解 Microsoft Graph 和可用的所有資料，請造訪 [https://graph.microsoft.io](https://graph.microsoft.io)。

目前不支援其他服務。未來將增加更多 Microsoft Online 服務，並支援您自己的自訂建置 Web API 和服務。

## 程式庫與 SDK 的限制
為了協助您嘗試，我們提供與 v2.0 端點相容的 Active Directory 驗證程式庫實驗版。不過，此版本的 ADAL 處於預覽狀態 - 目前不支援，接下來幾個月將有大幅變更。如果您想要儘快讓應用程式搭配 v2.0 端點一起執行，我們的[使用者入門](active-directory-appmodel-v2-overview.md#getting-started)一節中有使用 ADAL for .NET、iOS、Android 和 Javascript 的程式碼範例。

如果您想要在實際執行應用程式中使用 v2.0 端點，您有下列選項：

- 如果您要建置 Web 應用程式，您可以安心地使用我們正式運作的伺服器端中介軟體來執行登入和權杖驗證。其中包括適用於 ASP.NET 的 OWIN Open ID Connect 中介軟體和我們的 NodeJS Passport 外掛程式。[使用者入門](active-directory-appmodel-v2-overview.md#getting-started)一節也提供使用這些中介軟體的範例程式碼。
- 至於其他平台及原生與行動應用程式，您也可以直接在應用程式程式碼中傳送和接收通訊協定訊息，以便與 v2.0 端點整合。v2.0 OpenID Connect 和 OAuth 通訊協定[有明確的說明文件](active-directory-v2-protocols.md)，可協助您執行這種整合。
- 最後，您可以使用開放原始碼 Open ID Connect 和 OAuth 程式庫來與 v2.0 端點整合。v2.0 通訊協定與許多開放原始碼通訊協定程式庫都應該相容，而不需要重大變更。這類程式庫的可用性依語言和平台而不同，[Open ID Connect](http://openid.net/connect/) 和 [OAuth 2.0](http://oauth.net/2/) 網站維護一份常用的實作清單。以下是已通過 v2.0 端點測試的開放原始碼用戶端程式庫和範例。

  - [Java WSO2 身分識別伺服器](https://docs.wso2.com/display/IS500/Introducing+the+Identity+Server)
  - [Java Gluu 同盟](https://github.com/GluuFederation/oxAuth)
  - [Node.Js passport openidconnect](https://www.npmjs.com/package/passport-openidconnect)
  - [PHP OpenID Connect 基本用戶端](https://github.com/jumbojett/OpenID-Connect-PHP)
  - [iOS OAuth2 用戶端](https://github.com/nxtbgthng/OAuth2Client)
  - [Android OAuth2 用戶端](https://github.com/wuman/android-oauth-client)
  - [Android OpenID Connect 用戶端](https://github.com/kalemontes/OIDCAndroidLib)

## 通訊協定的限制
v2.0 端點僅支援 Open ID Connect 和 OAuth 2.0。不過，並非每個通訊協定的所有特性與功能都已納入到 v2.0 端點。部分範例包括：

- OpenID Connect `end_sesssion_endpoint`
- OAuth 2.0 用戶端認證授與

若要進一步了解 v2.0 端點支援的通訊協定功能範圍，請參閱 [OpenID Connect 與 OAuth 2.0 通訊協定參照](active-directory-v2-protocols.md)。

## 進階的 Azure AD 開發人員功能
Azure Active Directory 服務中有一組開發人員功能 (v2.0 端點尚不支援這些功能)，包括：

- Azure AD 使用者的群組宣告
- 應用程式角色和角色宣告

<!---HONumber=AcomDC_0921_2016-->