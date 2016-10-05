<properties 
	pageTitle="Azure 隨選媒體編碼器的概觀和比較 | Microsoft Azure" 
	description="本主題概要說明並比較 Azure 隨選媒體編碼器。" 
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

#Azure 隨選媒體編碼器的概觀和比較

##編碼概觀

Azure 媒體服務提供多個用於將雲端中之媒體編碼的選項。

開始使用媒體服務時，請務必了解轉碼器和檔案格式之間的差異。轉碼器是實作壓縮/解壓縮演算法的軟體，而檔案格式是保存已壓縮視訊的容器。

媒體服務提供動態封裝，這讓您以媒體服務支援的串流格式 (MPEG DASH、HLS、Smooth Streaming、HDS) 提供調適性位元速率 MP4 或 Smooth Streaming 編碼內容，而不必重新封裝成這些串流格式。

若要利用[動態封裝](media-services-dynamic-packaging-overview.md)，您需要執行下列動作：

- 將您的夾層 (來源) 檔編碼為一組調適性位元速率 MP4 檔案或調適性位元速率 Smooth Streaming 檔案 (編碼步驟稍後示範於本教學課程中)。
- 為您計畫從該處傳遞內容的串流端點取得至少一個隨選串流單元。如需詳細資訊，請參閱[如何調整隨選串流保留單元](media-services-portal-manage-streaming-endpoints.md)。

媒體服務支援本文中所描述的下列隨選編碼器：

- [Media Encoder Standard](media-services-encode-asset.md#media-encoder-standard)
- [Media Encoder Premium Workflow](media-services-encode-asset.md#media-encoder-premium-workflow)

本文概略敘述隨選媒體編碼器，並提供文章連結以提供更詳細資訊。本主題也提供各種編碼器的比較。

請注意，每個媒體服務帳戶預設一次可以有一個進行中的編碼工作。您可以保留編碼單位，這樣就可以同時執行多個編碼工作，其中一個用於您購買的每一個編碼保留單位。如需相關資訊，請參閱[調整編碼單位](media-services-scale-media-processing-overview.md)。

##Media Encoder Standard

###使用方式

[如何使用 Media Encoder Standard 進行編碼](media-services-dotnet-encode-with-media-encoder-standard.md)

###格式

[格式和轉碼器](media-services-media-encoder-standard-formats.md)

###預設值

Media Encoder Standard 使用[這裡](http://go.microsoft.com/fwlink/?linkid=618336&clcid=0x409)描述的其中一個編碼器預設值進行設定。

###輸入和輸出中繼資料

[這裡](http://msdn.microsoft.com/library/azure/dn783120.aspx)說明編碼器輸入中繼資料。

[這裡](http://msdn.microsoft.com/library/azure/dn783217.aspx)說明編碼器輸出中繼資料。

###產生縮圖

如需相關資訊，請參閱[如何使用媒體編碼器標準產生縮圖](media-services-custom-mes-presets-with-dotnet.md#thumbnails)。

###修剪視訊 (裁剪)

如需相關資訊，請參閱[如何使用媒體編碼器標準修剪視訊](media-services-custom-mes-presets-with-dotnet.md#trim_video)。

###建立疊加層

如需相關資訊，請參閱[如何使用媒體編碼器標準建立覆疊](media-services-custom-mes-presets-with-dotnet.md#overlay)。

###另請參閱

[媒體服務部落格](https://azure.microsoft.com/blog/2015/07/16/announcing-the-general-availability-of-media-encoder-standard/)
 
##Media Encoder Premium Workflow

###Overview

[介紹 Azure 媒體服務中的 Premium 編碼](https://azure.microsoft.com/blog/2015/03/05/introducing-premium-encoding-in-azure-media-services/)

###使用方式

Media Encoder Premium Workflow 使用複雜的工作流程設定。您可以使用[工作流程設計工具](media-services-workflow-designer.md)建立和更新工作流程檔案。

[如何使用 Azure 媒體服務中的 Premium 編碼](https://azure.microsoft.com/blog/2015/03/06/how-to-use-premium-encoding-in-azure-media-services/)

###已知問題

如果您的輸入視訊不包含隱藏式字幕，輸出資產仍然會包含空白 TTML 檔案。


##<a id="compare_encoders"></a>比較編碼器

###<a id="billing"></a>每個編碼器所使用的計費計量表

媒體處理器名稱|適用的價格|注意事項
---|---|---
**Media Encoder Standard** |ENCODER|在 ENCODER 資料行之下，編碼工作會以[這裡][1]指定的費率，根據輸出資產的大小 (以 GB 為單位) 收費。
**Media Encoder Premium 工作流程** |PREMIUM ENCODER|在 PREMIUM ENCODER 資料行之下，編碼工作會以[這裡][1]指定的費率，根據輸出資產的大小 (以 GB 為單位) 收費。


本節比較**媒體編碼器標準**和**媒體編碼器高階工作流程**的編碼功能。


###輸入容器/檔案格式

輸入容器/檔案格式|Media Encoder Standard|Media Encoder Premium Workflow
---|---|---
Adobe® Flash® F4V |是|是
MXF/SMPTE 377M |是|是
GXF |是|是
MPEG-2 傳輸資料流 |是|是
MPEG-2 程式資料流 |是|是
MPEG-4/MP4 |是|是
Windows Media/ASF |是|是
AVI (未壓縮 8 位元/10 位元)|是|是
3GPP/3GPP2 |是|否
Smooth Streaming 檔案格式 (PIFF 1.3)|是|否
[Microsoft Digital Video Recording(DVR-MS)](https://msdn.microsoft.com/library/windows/desktop/dd692984)|是|否
Matroska/WebM |是|否
QuickTime (.mov) |是|否

###輸入視訊轉碼器

輸入視訊轉碼器|Media Encoder Standard|Media Encoder Premium Workflow
---|---|---
AVC 8 位元/10 位元，高達 4:2:2，包括 AVCIntra |8 位元 4:2:0 和 4:2:2|是
Avid DNxHD (使用 MXF) |是|是
DVCPro/DVCProHD (使用 MXF) |是|是
JPEG2000 |是|是
MPEG-2 (高達 422 Profile 和 High Level，包括 XDCAM、XDCAM HD、XDCAM IMX、CableLabs ® 和 D10 等變種)|最高 422 設定檔|是
MPEG-1 |是|是
Windows Media 視訊/VC-1 |是|是
Canopus HQ/HQX |否|否
Mpeg-4 第 2 部分 |是|否
[Theora](https://en.wikipedia.org/wiki/Theora) |是|否
Apple ProRes 422 |是|否
Apple ProRes 422 LT |是|否
Apple ProRes 422 HQ |是|否
Apple ProRes Proxy|是|否
Apple ProRes 4444 |是|否
Apple ProRes 4444 XQ |是|否

###輸入音訊轉碼器

輸入音訊轉碼器|Media Encoder Standard|Media Encoder Premium Workflow
---|---|---
AES (SMPTE 331M 和 302M，AES3-2003) |否|是
Dolby® E |否|是
Dolby® Digital (AC3) |否|是
Dolby® Digital Plus (E-AC3) |否|是
AAC (AAC-LC、AAC-HE 和 AAC-HEv2；高達 5.1)|是|是
MPEG Layer 2|是|是
MP3 (MPEG-1 音訊層 3)|是|是
Windows Media 音訊|是|是
WAV/PCM|是|是
[FLAC](https://en.wikipedia.org/wiki/FLAC)</a>|是|否
[Opus](https://en.wikipedia.org/wiki/Opus_(audio_format)) |是|否
[Vorbis](https://en.wikipedia.org/wiki/Vorbis)</a>|是|否


###輸出容器/檔案格式

輸出容器/檔案格式|Media Encoder Standard|Media Encoder Premium Workflow
---|---|---
Adobe® Flash® F4V|否|是
MXF (OP1a、XDCAM 和 AS02)|否|是
DPP (包括 AS11)|否|是
GXF|否|是
MPEG-4/MP4|是|是
MPEG-TS|是|是
Windows Media/ASF|否|是
AVI (未壓縮 8 位元/10 位元)|否|是
Smooth Streaming 檔案格式 (PIFF 1.3)|否|是

###輸出視訊轉碼器

輸出視訊轉碼器|Media Encoder Standard|Media Encoder Premium Workflow
---|---|---
AVC (H.264；8 位元；高達 High Profile、Level 5.2；4K Ultra HD；AVC Intra)|僅限 8 位元 4:2:0|是
Avid DNxHD (使用 MXF)|否|是
DVCPro/DVCProHD (使用 MXF)|否|是
MPEG-2 (高達 422 Profile 和 High Level，包括 XDCAM、XDCAM HD、XDCAM IMX、CableLabs ® 和 D10 等變種)|否|是
MPEG-1|否|是
Windows Media 視訊/VC-1|否|是
JPEG 縮圖建立|否|是

###輸出音訊轉碼器

輸出音訊轉碼器|Media Encoder Standard|Media Encoder Premium Workflow
---|---|---
AES (SMPTE 331M 和 302M，AES3-2003)|否|是
Dolby® Digital (AC3)|否|是
Dolby® Digital Plus (E-AC3) 高達 7.1|否|是
AAC (AAC-LC、AAC-HE 和 AAC-HEv2；高達 5.1)|是|是
MPEG Layer 2|否|是
MP3 (MPEG-1 音訊層 3)|否|是
Windows Media 音訊|否|是


##錯誤碼  

下表列出在編碼工作執行期間發生錯誤的情況下可能傳回的錯誤碼。若要取得 .NET 程式碼中的錯誤詳細資料，請使用 [ErrorDetails](http://msdn.microsoft.com/library/microsoft.windowsazure.mediaservices.client.errordetail.aspx) 類別。若要取得 REST 程式碼中的錯誤詳細資料，請使用 [ErrorDetail](https://msdn.microsoft.com/library/jj853026.aspx) REST API。

ErrorDetail.Code|導致發生錯誤的可能原因
-----|-----------------------
不明| 執行工作時發生不明錯誤
ErrorDownloadingInputAssetMalformedContent|涵蓋下載輸入資產中之錯誤 (例如無效的檔案名稱、長度為零檔案、錯誤格式等等) 的錯誤類別。
ErrorDownloadingInputAssetServiceFailure|涵蓋服務端問題 (例如下載時發生網路或儲存體錯誤) 的錯誤類別。
ErrorParsingConfiguration|工作 <see cref="MediaTask.PrivateData"/> (組態) 無效時的錯誤類別，例如組態不是有效的系統預設值或包含無效的 XML。
ErrorExecutingTaskMalformedContent|在工作執行期間因輸入媒體檔案內部問題導致失敗的錯誤類別。
ErrorExecutingTaskUnsupportedFormat|媒體處理器無法處理提供之檔案 (不支援的媒體格式或與組態不符) 的錯誤類別。例如，嘗試從只有影片的資產產生只含音訊的輸出
ErrorProcessingTask|媒體處理器在處理和內容不相關的工作時發生的其他錯誤類別。
ErrorUploadingOutputAsset|上傳輸出資產時的錯誤類別
ErrorCancelingTask|涵蓋嘗試取消工作時失敗的錯誤類別
TransientError|涵蓋暫時性問題 (例如 Azure 儲存體暫時性網路問題) 的錯誤類別


若要獲得**媒體服務**小組的協助，請開啟[支援票證](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade)。



##媒體服務學習路徑

[AZURE.INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

##提供意見反應

[AZURE.INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]


##相關文章

- [透過自訂 Media Encoder Standard 預設值來執行進階編碼工作](media-services-custom-mes-presets-with-dotnet.md)
- [配額和限制](media-services-quotas-and-limitations.md)

 
<!--Reference links in article-->
[1]: http://azure.microsoft.com/pricing/details/media-services/

<!---HONumber=AcomDC_0921_2016-->