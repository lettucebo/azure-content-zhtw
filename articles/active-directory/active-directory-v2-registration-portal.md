<properties
	pageTitle="應用程式註冊入口網站說明主題 | Microsoft Azure"
	description="Microsoft App 註冊入口網站中各種功能的說明。"
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

# App 註冊參考
本文件提供可在 Microsoft App 註冊入口網站 [https://apps.dev.microsoft.com](https://apps.dev.microsoft.com) 中找到的各種功能的內容和說明。

## 我的應用程式
此清單包含所有已註冊且可與 Azure AD v2.0 端點搭配使用的應用程式。這些應用程式能夠讓使用者使用個人的帳戶 Microsoft 帳戶登入，以及使用工作/學校帳戶從 Azure Active Directory 登入。若要深入了解 Azure AD v2.0 端點，請參閱我們的 [v2.0 概觀](active-directory-appmodel-v2-overview.md)。這些應用程式也可以用來與 Microsoft 帳戶驗證端點 `https://login.live.com` 整合。

## Live SDK 應用程式
此清單包含所有已註冊且只能與 Microsoft 帳戶搭配使用的應用程式。在任何情況下，它們都不會啟用來與 Azure Active Directory 搭配使用。您可以在此處找到任何先前已於 MSA 開發人員入口網站 `https://account.live.com/developers/applications` 註冊的應用程式。您先前曾在 `https://account.live.com/developers/applications` 執行的所有函式現在均可在這個新的入口網站 `https://apps.dev.microsoft.com` 中執行。如果您有任何進一步有關 Microsoft 帳戶應用程式的問題，請與我們連絡。

## 應用程式密碼
應用程式密碼是可讓您的應用程式利用 Azure AD 執行可靠的[用戶端驗證](http://tools.ietf.org/html/rfc6749#section-2.3)的認證。在 OAuth 和 OpenID Connect 中，應用程式密碼通常稱為 `client_secret`。在 v2.0 通訊協定中，任何在 Web 可定址位置 (使用 `https` 配置) 中接收安全性權杖的應用程式都必須使用應用程式密碼，藉由兌換該安全性權杖來向 Azure AD 識別它自己。此外，裝置上收到權杖的任何原生用戶端都將禁止使用應用程式密碼來執行用戶端驗證，以防止在不安全的環境中儲存機密資料。

每個 app 最後都能在任何指定時間點包含兩個有效的應用程式密碼。藉由維護兩個密碼，您就能夠在應用程式的整個環境中執行定期的金鑰變換。一旦將應用程式的全部內容移轉至新的密碼之後，您可能會刪除舊的密碼，並佈建一個新密碼。

此時，App 註冊入口網站中只允許兩種類型的應用程式密碼。選擇 [產生新密碼] 將會在各自的資料存放區中產生並儲存共用的密碼，讓您可以在應用程式中加以使用。選擇 [產生新的金鑰組] 會建立新的公開/私密金鑰組，可下載並用來向 Azure AD 進行用戶端驗證。

## 設定檔
App 註冊入口網站的設定檔區段可以用來自訂您應用程式的登入頁面。此時，您可以改變登入頁面的應用程式標誌、服務條款 URL 及隱私權聲明。標誌必須是 GIF、PNG 或 JPEG 檔案中透明的 48 x 48 或 50 x 50 像素影像，且大小等於或小於 15 KB。請嘗試變更值，然後檢視產生的登入頁面！

## Live SDK 支援
當您啟用「Live SDK 支援」時，即會將您建立的任何應用程式密碼佈建到 Azure AD 與 Microsoft 帳戶資料存放區。這讓您的應用程式能夠直接與 Microsoft 帳戶服務 (login.live.com) 整合。如果您想要使用 Microsoft 帳戶直接建置 app (而不使用 Azure AD v2.0 端點)，則應確定已啟用 Live SDK 支援。

停用 Live SDK 支援將可確保系統只會將應用程式密碼寫入 Azure AD 資料存放區。Azure AD 資料存放區包含企業級法規，使其符合特定的標準，例如 FISMA 相容性。如果您啟用 Live SDK 支援，您的應用程式可能不會遵守這其中的部分標準。

如果您只打算使用 Azure AD v2.0 端點，您可以安全地停用 Live SDK 支援。

<!---HONumber=AcomDC_0921_2016-->