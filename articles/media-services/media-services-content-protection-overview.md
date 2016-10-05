<properties 
	pageTitle="保護內容概觀 | Microsoft Azure" 
	description="此文章簡介如何利用 Media Services 保護內容。" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
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

#保護內容概觀


Microsoft Azure 媒體服務可讓您保護媒體從離開電腦到進行儲存、處理和傳遞時的安全。媒體服務可讓您傳遞利用進階加密標準 (AES) (使用 128 位元加密金鑰) 動態加密和使用 PlayReady 和/或 Widevine DRM 的一般加密 (CENC) 的內容。媒體服務也提供服務，傳遞 AES 金鑰和 PlayReady 授權給授權用戶端。您也可以使用下列 AMS 合作夥伴來助您傳遞 Widevine 授權：[Axinom](http://www.axinom.com/press/ibc-axinom-drm-6/)、[EZDRM](http://ezdrm.com/)、[castLabs](http://castlabs.com/company/partners/azure/)。

- 下圖示範「PlayReady 和/或 Widevine DRM 動態一般加密」工作流程。如需詳細資訊，請參閱[使用 PlayReady 和/或 Widevine DRM 動態一般加密](media-services-protect-with-drm.md)。

![利用 PlayReady 保護](./media/media-services-content-protection-overview/media-services-content-protection-with-drm.png)


- 下圖示範「AES-128 動態加密」工作流程。如需詳細資訊，請參閱[使用 AES-128 動態加密和金鑰傳遞服務](media-services-protect-with-aes128.md)。

![利用 AES 128 保護](./media/media-services-content-protection-overview/media-services-content-protection-with-aes.png)

>[AZURE.NOTE]要想使用動態加密，您必須從負責傳送加密內容的資料流端點至少取得一個資料流保留單位。

##資產加密選項

根據您想要上傳、儲存和傳遞的內容類型，媒體服務會提供各種加密選項供您選擇。

###None

不使用加密。這是預設值。使用此選項時，您的內容在傳輸或儲存體中靜止時不會受到保護。

如果您計劃使用漸進式下載傳遞 MP4，請使用此選項來上傳內容。

###StorageEncrypted

使用 **StorageEncrypted**，使用 AES 256 位元加密對您的純文字內容進行本機加密，然後將其上傳到已靜止加密儲存的 Azure 儲存體。使用儲存體加密保護的資產會在編碼之前自動解除加密並放在加密的檔案系統中，並且選擇性地在上傳為新的輸出資產之前重新加密。儲存體加密的主要使用案例是當您想要使用強式加密保護靜止在磁碟上的高品質輸入媒體檔案時。

若要傳遞儲存體加密資產，您必須設定資產的傳遞原則，讓媒體服務知道您的內容傳遞方式。串流處理資產之前，串流伺服器會移除儲存體加密，並使用指定的傳遞原則來串流處理您的內容 (例如，AES、一般加密或不加密)。

####實作詳細資料

AMS 儲存體加密會將 **AES-CTR** 模式加密套用至整個檔案。AES CTR 模式是可加密任意長度資料且不需填補的區塊編碼器。其作業方式是使用 AES 演算法加密計數器區塊，並且對要加密或解密資料的 AES 輸出進行 XOR 處理。所使用計數器區塊的建構方式是將 InitializationVector 的值複製到計數器值的位元組 0 到 7，而且計數器值的位元組 8 到 15 設定為零。在 16 位元組的計數器區塊中，位元組 8 到 15 (即最低位元組) 是用作簡單 64 位元不帶正負號的整數，而且每個後續處理的資料區塊都會加一，並會保持網路位元組順序。請注意，如果此整數達到最大值 (0xFFFFFFFFFFFFFFFF)，則增加它時會將區塊計數器重設為零 (位元組 8 到 15)，而不影響計數器的其他 64 位元 (即位元組 0 到 7)。為了維護 AES CTR 模式加密的安全性，每個檔案之指定 KID 的 InitializationVector 值應該為唯一，而且檔案的長度應該小於 2 ^64 個區塊。這是為了確保計數器值絕不會與指定的索引鍵重複使用。如需 CTR 模式的詳細資訊，請參閱[此 Wiki 頁面](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#CTR) (Wiki 文章使用 "Nonce" 這個字，而非 "InitializationVector")。

如果您想要了解基本演算法的運作方式，請檢視下列方法的 AMS.NET 實作︰

- [ApplyEncryptionTransform](https://github.com/Azure/azure-sdk-for-media-services/blob/dev/src/net/Client/Common/Common.BlobTransfer/BlobTransferBase.cs)
- [AesCtr](https://github.com/Azure/azure-sdk-for-media-services/blob/dev/src/net/Client/Common/Common.FileEncryption/FileEncryptionTransform.cs)


###CommonEncryptionProtected

如果您想要使用一般加密來加密 (或上傳已加密) 內容，請使用 **CommonEncryptionProtected**。PlayReady 和 Widewine 是依照一般加密 (CENC) 規格，並且受 AMS 支援。

###EnvelopeEncryptionProtected

如果您想要保護利用進階加密標準 (AES) 所加密的 HTTP 即時串流 (HLS) 或上傳利用相同標準所加密的已保護 HTTP 即時串流 (HLS)，請使用 **EnvelopeEncryptionProtected**。請注意，如果您上傳已透過 AES 加密的 HLS，它必須是由 Transform Manager 加密。

##動態加密

Microsoft Azure 媒體服務可讓您傳遞利用進階加密標準 (AES) (使用 128 位元加密金鑰) 和 PlayReady 和/或 Widevine DRM 所動態加密的內容。

目前，您可以加密下列串流格式：HLS、MPEG DASH 和 Smooth Streaming。無法加密 HDS 串流格式，或漸進式下載。

如果您想要媒體服務加密資產，則需要建立加密金鑰 (CommonEncryption 或 EnvelopeEncryption) 與資產的關聯，同時設定金鑰的授權原則。

您也需要設定資產的傳遞原則。如果您想要串流處理儲存體加密的資產，請一定要透過設定資產傳遞原則來指定資產傳遞方式。

播放程式要求串流時，媒體服務便會使用 AES 或一般加密，使用指定的金鑰動態加密您的內容。為了將串流解密，播放程式將從金鑰傳遞服務要求金鑰。為了決定使用者是否有權取得金鑰，服務會評估為金鑰指定的授權原則。

>[AZURE.NOTE]若要利用動態加密，您必須先為想要從該處傳遞加密內容的串流端點取得至少一個隨選串流單元。如需詳細資訊，請參閱[如何調整媒體服務](media-services-portal-manage-streaming-endpoints.md)。

##授權和金鑰傳遞服務

媒體服務提供一種服務，可將 DRM (PlayReady 與 Widevine) 授權和 AES 清除金鑰傳遞給授權用戶端。若要設定您授權和金鑰的授權和驗證原則，才能使用 Azure 傳統入口網站、REST API 或 Media Services SDK for .NET。

請注意，如果您是使用入口網站，則可以設定一個 AES 原則 (將會套用到所有 AES 加密內容) 以及一個 PlayReady 原則 (將會套用到所有 PlayReady 加密內容)。如果您想要進一步控制組態，請使用 Media Services SDK for .NET。

##DRM 授權

###PlayReady 授權

媒體服務提供一種服務，來傳遞 PlayReady 授權。使用者播放程式 (例如 Silverlight) 嘗試播放 PlayReady 保護內容時，會將要求傳送到授權傳遞服務來取得授權。如果授權服務核准要求，就會發出傳送給用戶端並可用來解密和播放所指定內容的授權。

授權包含您要 PlayReady DRM 執行階段在使用者嘗試播放受保護內容時強制執行的權限和限制。媒體服務提供可讓您設定 PlayReady 授權的 API。如需詳細資訊，請參閱[媒體服務 PlayReady 授權範本概觀](media-services-playready-license-template-overview.md)。

###Widevine 授權

AMS 也可讓您傳遞使用 Widevine DRM 加密的 MPEG DASH。PlayReady 和 Widewine 是依照一般加密 (CENC) 規格加密。您可以使用 [AMS .NET SDK](https://www.nuget.org/packages/windowsazure.mediaservices/) (從版本 3.5.1 開始) 或 REST API 來設定 AssetDeliveryConfiguration 以使用 Widevine。

從媒體服務 .NET SDK 版本 3.5.2 開始，媒體服務讓您可設定 [Widevine 授權範本](media-services-widevine-license-template-overview.md)並取得 Widevine 授權。您也可以使用下列 AMS 合作夥伴來助您傳遞 Widevine 授權：[Axinom](http://www.axinom.com/press/ibc-axinom-drm-6/)、[EZDRM](http://ezdrm.com/)、[castLabs](http://castlabs.com/company/partners/azure/)。

##權杖限制

內容金鑰授權原則可能會有一個或多個授權限制：open 或 token 限制。權杖限制原則必須伴隨著安全權杖服務 (STS) 所發出的權杖。媒體服務支援簡單 Web 權杖 (SWT) 格式和 JSON Web 權杖 (JWT) 格式的權杖。媒體服務不提供安全權杖服務。您可以建立自訂 STS，或利用 Microsoft Azure ACS 來發行權杖。STS 必須設定為建立使用指定的索引鍵和問題宣告您在權杖限制組態中指定簽署的權杖。如果權杖有效，且權杖中的宣告符合針對金鑰 (或授權) 所設定的宣告，媒體服務金鑰傳遞服務會將所要求的金鑰 (或授權) 傳回給用戶端。

設定 token 限制原則時，您必須指定主要驗證金鑰、簽發者和對象參數。主要驗證金鑰包含簽署權杖使用的金鑰，簽發者是發行權杖的安全權杖服務。對象 (有時稱為範圍) 描述權杖或權杖獲授權存取之資源的用途。媒體服務金鑰傳遞服務會驗證權杖中的這些值符合在範本中的值。

##常見案例

###保護儲存體中的內容、提供動態加密串流處理媒體、使用 AMS 金鑰/授權傳遞服務

1. 將高品質夾層檔內嵌到資產。將儲存體加密選項套用到資產。
2. 設定串流端點。
1. 編碼為調適性位元速率 MP4 集。將儲存體加密選項套用到輸出資產。
1. 針對您想要在播放期間動態加密的資產，建立加密內容金鑰。
2. 設定內容金鑰授權原則。
1. 設定資產傳遞原則 (供動態封裝和動態加密使用)。
1. 透過建立 OnDemand 定位器，來發佈資產。
1. 串流發佈的內容。

如需詳細資訊，請參閱︰[利用 AES 保護](media-services-protect-with-aes128.md)和[利用 PlayReady 和 (或) Widevine 保護](media-services-protect-with-drm.md)。


###將媒體服務金鑰和授權傳遞服務運用到您自己的加密與串流處理服務

如需詳細資訊，請參閱[如何整合 Azure PlayReady 授權服務與您自己的加密程式/串流伺服器](http://mingfeiy.com/integrate-azure-playready-license-service-encryptorstreaming-server)。

###與合作夥伴整合

[使用 castLabs 將 DRM 授權傳遞到 Azure 媒體服務](media-services-castlabs-integration.md)

##媒體服務學習路徑

[AZURE.INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

##提供意見反應

[AZURE.INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]


##相關連結

[利用 Azure 媒體服務宣告 PlayReady 是一個服務和 AES 動態加密](http://mingfeiy.com/playready)

[Azure 媒體服務 PlayReady 授權傳遞定價說明](http://mingfeiy.com/playready-pricing-explained-in-azure-media-services)

[如何偵錯 Azure 媒體服務的 AES 加密串流](http://mingfeiy.com/debug-aes-encrypted-stream-azure-media-services)

[JWT 權杖驗證](http://www.gtrifonov.com/2015/01/03/jwt-token-authentication-in-azure-media-services-and-dynamic-encryption/)

[整合 Azure 媒體服務 OWIN MVC 型應用程式與 Azure Active Directory 並根據 JWT 宣告限制內容金鑰傳遞](http://www.gtrifonov.com/2015/01/24/mvc-owin-azure-media-services-ad-integration/)。

[使用 Azure ACS 發行權杖](http://mingfeiy.com/acs-with-key-services)。

[content-protection]: ./media/media-services-content-protection-overview/media-services-content-protection.png

<!---HONumber=AcomDC_0921_2016-->