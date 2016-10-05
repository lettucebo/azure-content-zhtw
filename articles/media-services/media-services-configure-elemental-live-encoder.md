<properties 
	pageTitle="設定 Elemental Live 編碼器來傳送單一位元速率的即時串流 | Microsoft Azure" 
	description="本主題會示範如何設定 Elemental Live 編碼器，藉此將單一位元速率的即時串流傳送到 AMS 通道，其已針對即時編碼而啟用。" 
	services="media-services" 
	documentationCenter="" 
	authors="cenkdin" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.workload="media" 
	ms.tgt_pltfrm="na" 
	ms.devlang="ne" 
	ms.topic="article" 
	ms.date="09/19/2016"
	ms.author="cenkdin;anilmur;juliako"/>

#使用 Elemental Live 編碼器來傳送單一位元速率的即時串流

> [AZURE.SELECTOR]
- [Elemental Live](media-services-configure-elemental-live-encoder.md)
- [Tricaster](media-services-configure-tricaster-live-encoder.md)
- [Wirecast](media-services-configure-wirecast-live-encoder.md)
- [FMLE](media-services-configure-fmle-live-encoder.md)

本主題會示範如何設定 [Elemental Live](http://www.elementaltechnologies.com/products/elemental-live) 編碼器，藉此將單一位元速率的即時串流傳送到 AMS 通道，其已針對即時編碼而啟用。如需詳細資訊，請參閱[使用啟用的通道來以 Azure 媒體服務執行即時編碼](media-services-manage-live-encoder-enabled-channels.md)。

本教學課程示範如何使用 Azure 媒體服務總管 (AMSE) 工具管理 Azure 媒體服務 (AMS)。此工具只會在 Windows 電腦上執行。如果您是用 Mac 或 Linux，請使用 Azure 傳統入口網站建立[通道](media-services-portal-creating-live-encoder-enabled-channel.md#create-a-channel)和[程式](media-services-portal-creating-live-encoder-enabled-channel.md#create-and-manage-a-program)。

##必要條件

- 必須具備使用 Elemental Live Web 介面來建立即時事件的工作知識。
- [建立 Azure 媒體服務帳戶](media-services-create-account.md)
- 請確定執行的串流端點至少有一個配置的串流單位。如需詳細資訊，請參閱[在媒體服務帳戶中管理串流端點](media-services-portal-manage-streaming-endpoints.md)。
- 安裝最新版的 [AMSE](https://github.com/Azure/Azure-Media-Services-Explorer) 工具。
- 啟動工具並連接到您的 AMS 帳戶。

##秘訣

- 請盡可能使用實體的有線網際網路連線。
- 判斷頻寬需求的一項法則是將串流位元速率加倍。雖然這不是強制性需求，卻有助於減輕網路阻塞的影響。
- 使用軟體式編碼器時，請關閉任何不必要的程式。

## Elemental Live 與 RTP 嵌入

本節說明如何設定可透過 RTP 傳送單一位元速率的即時串流 Elemental Live 編碼器。如需詳細資訊，請參閱[透過 RTP 的 MPEG-TS 串流](media-services-manage-live-encoder-enabled-channels.md#channel)。

### 建立通道

1.  在 AMSE 工具中，瀏覽至 [Live] 索引標籤，然後在通道區域內按一下滑鼠右鍵。從功能表選取 [建立通道...]。

![Elemental](./media/media-services-elemental-live-encoder/media-services-elemental1.png)

2. 指定通道名稱，描述欄位是選擇性的。在 [頻道設定] 下方，針對 [即時編碼] 選項選取 [**標準**]，並將 [輸入通訊協定] 設為 [**RTP (MPEG-TS)**]。您可以將所有其他設定保留現狀。


請確認已選取 [**立即啟動新頻道**]。

3. 按一下 [**建立頻道**]。

![Elemental](./media/media-services-elemental-live-encoder/media-services-elemental12.png)

>[AZURE.NOTE] 通道約需 20 分鐘的時間即可啟動。

當頻道啟動時，您可以[設定編碼器](media-services-configure-elemental-live-encoder.md#configure_elemental_rtp)。

>[AZURE.IMPORTANT] 請注意，只要通道進入就緒狀態，就會開始計費。如需詳細資訊，請參閱[頻道的狀態](media-services-manage-live-encoder-enabled-channels.md#states)。

###<a id=configure_elemental_rtp></a>設定 Elemental Live 編碼器 

在本教學課程中會使用下列輸出設定。本章節的其餘部分將詳細說明組態步驟。

**視訊**：
 
- 轉碼器：H.264
- 設定檔：高 (層級 4.0)
- 位元速率：5000 kbps
- 主要畫面格：2 秒 (60 秒)
- 畫面播放速率：30
 
**音訊**：

- 轉碼器：AAC (LC)
- 位元速率：192 kbps
- 取樣速率：44.1 kHz


####組態步驟

1. 瀏覽至 **Elemental Live** Web 介面，並設定用於 **UDP/TS** 串流的編碼器。

2. 建立新事件後，請向下捲動至輸出群組並新增 **UDP/TS** 輸出群組。

3. 選取 [**新增串流**] 然後按一下 [**加入輸出**] 建立新的輸出。
	
	![Elemental](./media/media-services-elemental-live-encoder/media-services-elemental13.png)
	
	>[AZURE.NOTE] 建議將 Elemental 事件的時間代碼設定為「系統時鐘」，以協助編碼器在串流失敗時重新連接。

4. 現在已建立好輸出，請按一下 [**新增串流**]。現在可以設定輸出設定。
5. 向下捲動到剛剛建立的「串流 1」，按一下左側的 [**視訊**] 索引標籤並展開 [**進階**] 設定區段。

	![Elemental](./media/media-services-elemental-live-encoder/media-services-elemental4.png)

	雖然 Elemental Live 有各種不同的可用自訂，建議對串流至 AMS 入門使用下列設定。
	
	- 解析度：1280 x 720
	- 畫面播放速率：30
	- GOP 大小：60 個畫面格
	- 交錯式模式：漸進式
	- 位元速率：5000000 位元/秒 (這可以根據網路限制調整)
	

	![Elemental](./media/media-services-elemental-live-encoder/media-services-elemental5.png)

6. 取得通道的輸入 URL。
	
	瀏覽回到 AMSE 工具，並檢查通道的完成狀態。狀態從 [**啟動中**] 變更為 [**執行中**] 後，您便可取得輸入 URL。
	  
	頻道執行時，以滑鼠右鍵按一下頻道名稱，向下瀏覽讓滑鼠游標停留在 [**複製輸入 URL 到剪貼簿**]，然後選取 [**主要輸入 URL**]。
	
	![Elemental](./media/media-services-elemental-live-encoder/media-services-elemental6.png)
	
1. 在 Elemental 的 [**主要目的地**] 欄位中貼上這項資訊。所有其他設定可以保留預設值。
	
	![Elemental](./media/media-services-elemental-live-encoder/media-services-elemental14.png)

	如需額外的備援，請對次要輸入 URL 重複這些步驟，方法是為 UDP/TS 串流建立個別的「輸出」索引標籤。
	
7. 按一下 [**建立**] \(如果已建立新事件) 或 [**更新**] \(如果是編輯原有的事件)，然後繼續啟動編碼器。

>[AZURE.IMPORTANT] 按一下 Elemental Live Web 介面上的 [**開始**] 之前，您**必須**確保頻道已就緒。
>此外，請確保不要讓通道處於就緒狀態而無任何事件超過 15 分鐘。

在串流已經執行了 30 秒後，導覽回 AMSE 工具和測試播放。

###測試播放
  
1. 瀏覽至 AMSE 工具，然後以滑鼠右鍵按一下要測試的通道。在功能表中，將滑鼠游標停留在 [播放預覽]，並選取 [使用 Azure 媒體播放器]。

	![Elemental](./media/media-services-elemental-live-encoder/media-services-elemental8.png)

如果播放器中出現串流，則編碼器已妥善設定為連接到 AMS。

如果收到錯誤，則必須重設通道，且編碼器設定需要調整。請參閱[疑難排解](media-services-troubleshooting-live-streaming.md)主題中的指引。

###建立程式

1. 一旦確認通道播放沒問題後，請建立程式。在 AMSE 工具的 [Live] 索引標籤下，於程式區域內按一下滑鼠右鍵，並選取 [建立新的程式]。

	![Elemental](./media/media-services-elemental-live-encoder/media-services-elemental9.png)

2. 為程式命名，並視需要調整**封存時間長度** (預設為 4 小時)。您也可以指定儲存體位置，或保留為預設值。
3. 勾選 [現在啟動程式] 方塊。
4. 按一下 [建立程式]。
  
	注意：建立程式時所使用的時間會比建立通道時更少。
 
5. 一旦程式開始執行，請在程式上按一下滑鼠右鍵，並瀏覽至 [播放程式]，然後選取 [使用 Azure 媒體播放器] 確認播放。
6. 一經確認後，再次於該程式上按一下滑鼠右鍵，並選取 [複製輸出 URL 到剪貼簿] \(或從 [程式資訊和設定] 功能表選項擷取這項資訊)。

串流現在已經可以內嵌於播放程式中，或散發給某個對象，以供即時檢視。

## 疑難排解

請參閱[疑難排解](media-services-troubleshooting-live-streaming.md)主題中的指引。


##媒體服務學習路徑

[AZURE.INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

##提供意見反應

[AZURE.INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]

<!---HONumber=AcomDC_0921_2016-->