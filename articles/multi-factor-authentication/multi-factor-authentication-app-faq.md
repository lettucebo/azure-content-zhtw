<properties
	pageTitle="Microsoft Authenticator 應用程式常見問題集"
	description="提供關於 Microsoft 驗證應用程式與 Azure Multi-Factor Authentication 的常見問題與答案清單。"
	services="multi-factor-authentication"
	documentationCenter=""
	authors="kgremban"
	manager="femila"
	editor="pblachar, librown"/>

<tags
	ms.service="multi-factor-authentication"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/20/2016"
	ms.author="kgremban"/>

# Microsoft Authenticator 應用程式常見問題集

Microsoft Authenticator 應用程式取代了 Azure Authenticator 應用程式，當您使用 Azure Multi-Factor Authentication 時，建議使用此應用程式。此應用程式適用於 Windows Phone、Android 和 iOS。

## 常見問題集

- **我已經使用 Microsoft Authenticator 應用程式做為安全性驗證碼。如何切換到單鍵推播通知？**

	如果您使用 Microsoft 帳戶做為個人帳戶，想要切換到推播通知，您必須再次新增您的帳戶。這是因為應用程式使用一次性密碼。使用您的帳戶重新註冊裝置，並設定推播通知。

	如果您的帳戶沒有啟用雙步驟驗證，請參閱[關於雙步驟驗證](https://support.microsoft.com/help/12408/microsoft-account-about-two-step-verification)來判斷這是否適合您。

- **我何時能夠在 iPhone 或 iPad 上使用單鍵推播通知？**

	這項功能在 8 月底供 Microsoft 帳戶廣泛使用之前為 Beta 版。如果您想要加入我們的 Beta 方案，請傳送電子郵件至 msauthenticator@microsoft.com。在郵件中包含您的名字、姓氏和 Apple ID。

- **單鍵推播通知適用於非 Microsoft 帳戶嗎？**

	不適用，推播通知只適用於 Microsoft 帳戶與 Azure Active Directory 帳戶。如果您的工作或學校使用 Azure AD 帳戶，它們可能會停用這項功能。

- **我從備份還原了我的裝置，但我的帳戶代碼遺漏或無法運作。這是怎麼回事？**

	基於安全考量，我們目前不會從應用程式的備份還原帳戶。如果您從備份還原 iOS 應用程式，您的帳戶仍會顯示，但無法用來接收登入驗證或產生安全性驗證碼。還原應用程式之後，請刪除您的帳戶，然後重新新增。

## 相關主題

- [Azure Multi-Factor Authentication 常見問題集](multi-factor-authentication-faq.md)
- Microsoft 帳戶的[關於雙步驟驗證](https://support.microsoft.com/help/12408/microsoft-account-about-two-step-verification)
- [身分識別驗證 app：常見問題集](https://support.microsoft.com/help/12414/microsoft-account-identity-verification-apps-faq)

<!---HONumber=AcomDC_0921_2016-->