<properties
   pageTitle="在 Azure Active Directory 應用程式庫中列出您的應用程式"
   description="如何列出 Azure Active Directory 組件庫中支援單一登入的應用程式 | Microsoft Azure"
   services="active-directory"
   documentationCenter="dev-center-name"
   authors="msmbaldwin"
   manager="mbaldwin"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="identity"
   ms.date="09/16/2016"
   ms.author="mbaldwin"/>


# 在 Azure Active Directory 應用程式庫中列出您的應用程式

若要在 [Azure AD 資源庫](https://azure.microsoft.com/marketplace/active-directory/all/)中列出某個支援單一登入搭配 Azure Active Directory 的應用程式 ，該應用程式首先必須實作下列其中一項整合模式：

* **OpenID Connect** - 直接整合 Azure AD，使用 OpenID Connect 進行驗證，使用 Azure AD 同意 API 進行設定。如果您是剛開始整合，而您的應用程式不支援 SAML，這是建議的模式。

* **SAML** – 您的應用程式已能夠使用 SAML 通訊協定設定協力廠商識別提供者。

每個模式的列出需求如下。

##OpenID Connect 整合

若要整合您的應用程式與 Azure AD，請遵循[開發人員指示](active-directory-authentication-scenarios.md)。接著完成下面問題，並傳送至 waadpartners@microsoft.com。

* 提供可由 Azure AD 小組搭配您的應用程式來測試整合的測試租用戶或帳戶的認證。

* 提供有關 Azure AD 小組如何使用 [Azure AD 同意架構](active-directory-integrating-applications.md#overview-of-the-consent-framework)登入並連接 Azure AD 執行個體到您的應用程式的指示。

* 提供 Azure AD 小組搭配您的應用程式測試單一登入所需的任何進一步指示。

* 提供下列資訊：

> 公司名稱：
> 
> 公司網站：
> 
> 應用程式名稱：
> 
> 應用程式描述 (256 個字元的限制)：
> 
> 應用程式網站 (資訊)：
> 
> 應用程式技術支援網站或連絡資訊：
> 
> 應用程式的用戶端識別碼 (如 https://manage.windowsazure.com 中的應用程式詳細資料所示)：
> 
> 客戶前往註冊和 (或) 購買應用程式的應用程式註冊 URL：
> 
> 選擇要為您的應用程式列出的最多三個類別 (如需可用的類別，請參閱 Azure Active Directory Marketplace)：
> 
> 附加應用程式小型圖示 (PNG 檔案、45px x 45px、背景純色)：
> 
> 附加應用程式大型圖示 (PNG 檔案、215px x 215px、背景純色)：
> 
> 附加應用程式標誌 (PNG 檔案、150px x 122px、透明背景色彩)：

##SAML 整合

使用[這些指示來新增自訂應用程式](active-directory-saas-custom-apps.md)，支援 SAML 2.0 的任何應用程式都可直接與 Azure AD 租用戶整合。一旦您測試過應用程式可與 Azure AD 整合後，請將下列資訊傳送至 <waadpartners@microsoft.com>。

* 提供可由 Azure AD 小組搭配您的應用程式來測試整合的測試租用戶或帳戶的認證。

* 提供您的應用程式的 SAML 登入 URL、簽發者 URL (實體 ID) 和回覆 URL (判斷提示取用者服務) 的值，如[此處](active-directory-saas-custom-apps.md)所述。如果您通常會提供這些值做為一個 SAML 中繼資料檔的一部分，也請一併傳送該檔案。

* 提供如何在使用 SAML 2.0 的應用程式中設定 Azure AD 做為身分識別提供者的簡短描述。如果您的應用程式支援透過自助系統管理入口網站來設定 Azure AD 做為身分識別提供者，請確認以上提供的認證包含執行這項設定所需的能力。

* 提供下列資訊：

> 公司名稱：
> 
> 公司網站：
> 
> 應用程式名稱：
> 
> 應用程式描述 (256 個字元的限制)：
> 
> 應用程式網站 (資訊)：
> 
> 應用程式技術支援網站或連絡資訊：
> 
> 客戶前往註冊和 (或) 購買應用程式的應用程式註冊 URL：
> 
> 選擇要為您的應用程式列出的最多三個類別 (如需可用的類別，請參閱 [Azure Active Directory Marketplace](https://azure.microsoft.com/marketplace/active-directory/))：
> 
> 附加應用程式小型圖示 (PNG 檔案、45px x 45px、背景純色)：
> 
> 附加應用程式大型圖示 (PNG 檔案、215px x 215px、背景純色)：
> 
> 附加應用程式標誌 (PNG 檔案、150px x 122px、透明背景色彩)：

<!---HONumber=AcomDC_0921_2016-->