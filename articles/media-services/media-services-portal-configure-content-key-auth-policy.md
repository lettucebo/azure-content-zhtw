<properties 
	pageTitle="使用 Azure 入口網站設定內容金鑰授權原則 | Microsoft Azure" 
	description="了解如何設定內容金鑰的授權原則。" 
	services="media-services" 
	documentationCenter="" 
	authors="juliako" 
	manager="erikre" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.workload="media" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
 	ms.date="09/19/2016" 
	ms.author="juliako"/>



#設定內容金鑰授權原則
[AZURE.INCLUDE [媒體-服務-選取器-內容-金鑰-auth-原則](../../includes/media-services-selector-content-key-auth-policy.md)]


##Overview

Microsoft Azure 媒體服務可讓您傳遞受到進階加密標準 (AES) (使用 128 位元加密金鑰) 或 [Microsoft PlayReady DRM](https://www.microsoft.com/playready/overview/) 保護的 MPEG DASH、Smooth Streaming 和 HTTP Live Streaming (HLS) 串流。AMS 也可讓您傳遞使用 Widevine DRM 加密的 DASH 串流。PlayReady 和 Widevine 是依照 Common Encryption (ISO/IEC 23001-7 CENC) 規格加密。

媒體服務也提供**金鑰/授權傳遞服務**，用戶端可以從該處取得 AES 金鑰或 PlayReady/Widevine 授權，以便播放加密的內容。

本主題示範如何使用 **Azure 傳統入口網站**設定內容金鑰授權原則。金鑰稍後可以用來動態加密您的內容。請注意，目前您可以加密下列串流格式：HLS、MPEG DASH 和 Smooth Streaming。無法加密 HDS 串流格式，或漸進式下載。

當播放程式要求設定為動態加密的串流時，媒體服務會使用設定的金鑰，以 AES 或 DRM 加密來動態加密您的內容。為了將串流解密，播放程式將從金鑰傳遞服務要求金鑰。為了決定使用者是否有權取得金鑰，服務會評估為金鑰指定的授權原則。


如果您預計有多個內容金鑰，或想要指定**金鑰/授權傳遞服務** URL，而非媒體服務金鑰傳遞服務，請使用媒體服務 .NET SDK 或 REST API。

[使用媒體服務 .NET SDK 設定內容金鑰授權原則](media-services-dotnet-configure-content-key-auth-policy.md)

[使用媒體服務 REST API 設定內容金鑰授權原則](media-services-rest-configure-content-key-auth-policy.md)

###適用一些考量事項：

- 為了能夠使用動態封裝和動態加密，您至少有一個串流保留單元。如需詳細資訊，請參閱[如何調整媒體服務](media-services-portal-manage-streaming-endpoints.md)。
- 您的資產必須包含一組調適性位元速率 MP4 或調適性位元速率 Smooth Streaming 檔案。如需詳細資訊，請參閱[為資產編碼](media-services-encode-asset.md)。
- 金鑰傳遞服務會快取 ContentKeyAuthorizationPolicy 和其相關物件 (原則選項和限制) 15 分鐘。如果您建立 ContentKeyAuthorizationPolicy，並指定要使用 "Token" 的限制，那麼便測試它，然後將原則更新為"Open" 限制，將需要大約 15 分鐘，原則才會切換為 "Open" 版本的原則。


##作法：設定金鑰授權原則

若要設定金鑰授權原則，請選取 [**內容保護**] 頁面。

媒體服務支援多種方式來驗證提出金鑰要求的使用者。內容金鑰授權原則可以有 **open**、**token** 或 **IP** 授權限制 (**IP** 可以使用 REST 或 .NET SDK 設定)。

###Open 限制

**open** 限制表示系統將會傳送金鑰給提出金鑰要求的任何人。這項限制可用於測試用途。

![OpenPolicy][open_policy]

###權杖限制

若要選擇權杖限制原則，請按 [**權杖**] 按鈕。

**token** 限制原則必須伴隨著**安全權杖服務** (STS) 所發出的權杖。媒體服務支援**簡單 Web 權杖** ([SWT](https://msdn.microsoft.com/library/gg185950.aspx#BKMK_2)) 格式和 **JSON Web 權杖** (JWT) 格式的權杖。如需資訊，請參閱 [JWT 權杖驗證](http://www.gtrifonov.com/2015/01/03/jwt-token-authentication-in-azure-media-services-and-dynamic-encryption/)。

媒體服務不提供**安全權杖服務**。您可以建立自訂 STS，或利用 Microsoft Azure ACS 來發行權杖。STS 必須設定為建立使用指定的索引鍵和問題宣告您在權杖限制組態中指定簽署的權杖。如果權杖有效，且權杖中的宣告符合為內容金鑰設定的宣告，媒體服務金鑰傳遞服務會將加密金鑰傳回給用戶端。如需詳細資訊，請參閱[使用 Azure ACS 發行權杖](http://mingfeiy.com/acs-with-key-services)。

設定 **TOKEN** 限制原則時，您必須設定**驗證金鑰**、**簽發者**和**對象**的值。主要驗證金鑰包含簽署權杖使用的金鑰，簽發者是發行權杖的安全權杖服務。對象 (有時稱為範圍) 描述權杖或權杖獲授權存取之資源的用途。媒體服務金鑰傳遞服務會驗證權杖中的這些值符合在範本中的值。

###PlayReady

使用 **PlayReady** 保護內容時，您需要在驗證原則中指定的其中一件事是定義 PlayReady 授權範本的 XML 字串。依預設，會設定下列原則：

<PlayReadyLicenseResponseTemplate xmlns:i="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/PlayReadyTemplate/v1"> <LicenseTemplates> <PlayReadyLicenseTemplate><AllowTestDevices>true</AllowTestDevices> <ContentKey i:type="ContentEncryptionKeyFromHeader" /> <LicenseType>Nonpersistent</LicenseType> <PlayRight> <AllowPassingVideoContentToUnknownOutput>Allowed</AllowPassingVideoContentToUnknownOutput> </PlayRight> </PlayReadyLicenseTemplate> </LicenseTemplates> </PlayReadyLicenseResponseTemplate>

您可以按一下 [**匯入原則 xml**] 按鈕，並提供符合[這裡](https://msdn.microsoft.com/library/azure/dn783459.aspx)所定義之 XML 結構描述的不同 XML。


##後續步驟

檢閱媒體服務學習路徑。

[AZURE.INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

##提供意見反應

[AZURE.INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]





[open_policy]: ./media/media-services-portal-configure-content-key-auth-policy/media-services-protect-content-with-open-restriction.png
[token_policy]: ./media/media-services-key-authorization-policy/media-services-protect-content-with-token-restriction.png

<!---HONumber=AcomDC_0921_2016-->