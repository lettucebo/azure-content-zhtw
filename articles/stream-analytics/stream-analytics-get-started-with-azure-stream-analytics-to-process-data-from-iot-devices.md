<properties
	pageTitle="開始使用 Azure 串流分析來處理 IoT 裝置的資料 | 串流分析"
	description="IoT 感應器標記和具有串流分析的資料串流與即時資料處理"
    keywords="IoT 解決方案，開始使用 IoT"
	services="stream-analytics"
	documentationCenter=""
	authors="jeffstokes72"
	manager="jhubbard"
	editor="cgronlun"
/>

<tags 
	ms.service="stream-analytics" 
	ms.devlang="na" 
	ms.topic="hero-article" 
	ms.tgt_pltfrm="na" 
	ms.workload="data-services" 
	ms.date="09/26/2016"
	ms.author="jeffstok"
/>

# 開始使用 Azure 串流分析來處理 IoT 裝置的資料

在本教學課程中，您將學習建立串流處理邏輯，以從物聯網 (IoT) 裝置收集資料。我們將使用真實世界的物聯網 (IoT) 使用案例以示範如何快速且經濟地建置您的解決方案。

## 必要條件

-   [Azure 訂閱](https://azure.microsoft.com/pricing/free-trial/)
-   範例查詢和資料檔案可從 [GitHub](https://aka.ms/azure-stream-analytics-get-started-iot) 下載

## 案例

Contoso 是一家工業自動化空間的公司，他們已完全自動化其製造程序。這家製造廠的機械裝置具有感應器，可以發出即時資料串流。在此案例中，生產現場經理想要取得感應器資料的即時深入資訊，來尋找模式並且對其採取動作。我們將對感應器資料使用串流分析查詢語言 (SAQL)，以在內送資料串流中找出感興趣的模式。

這裡的資料是從 Texas Instrument Sensor Tag 裝置產生。

![Texas Instruments Sensor Tag](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-01.jpg)

資料的裝載的格式是 JSON，如下所示：

    
	{
    	"time": "2016-01-26T20:47:53.0000000",  
	    "dspl": "sensorE",  
    	"temp": 123,  
	    "hmdt": 34  
	}  
    
在真實世界的案例中，您會有 100 個此類感應器，產生事件作為串流。在理想情況下會有執行一些程式碼的閘道器，將這些事件推送至 [Azure 事件中樞](https://azure.microsoft.com/services/event-hubs/)或 [Azure IoT 中樞](https://azure.microsoft.com/services/iot-hub/)。串流分析作業會從事件中樞擷取這些事件，並針對串流執行即時分析查詢。接著，您可以將結果傳送至其中一個[支援輸出](stream-analytics-define-outputs.md)。

為了簡化使用經驗，在此快速入門指南中，我們提供您從實際 SensorTag 裝置擷取的範例資料檔，您可以對其執行不同的查詢，並查看其結果。在後續教學課程中，您將了解如何將您的作業連接至輸入和輸出，並將它們部署到 Azure 服務。

## 建立串流分析工作

在 [Azure 入口網站](http://manage.windowsazure.com)中選取 [串流分析]，然後按一下頁面左下角的 [新增] 來建立新的分析作業。

![建立新的串流分析作業](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-02.png)

按一下 [快速建立]。

對於 [區域監視儲存體帳戶] 設定，請選取 [建立新的儲存體帳戶]，並給予它任何唯一的名稱。Azure 串流分析會使用此帳戶來儲存您未來所有工作的監視資訊。

> [AZURE.NOTE] 您應該只對每個區域建立此儲存體帳戶一次，此儲存體會跨該區域中建立的所有串流分析作業共用。

按一下頁面底部的 [建立串流分析作業]。

![儲存體帳戶組態](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-03.jpg)

## Azure 串流分析查詢

按一下 [查詢] 索引標籤以移至 [查詢編輯器]。[查詢] 索引標籤包含會針對傳入事件資料進行轉換的 T-SQL 查詢。

## 封存您的原始資料

查詢的最簡單形式是傳遞，此形式會將所有輸入資料封存至其指定的輸出。

![封存作業查詢](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-04.png)

立即從 [GitHub](https://aka.ms/azure-stream-analytics-get-started-iot) 將範例資料檔案下載至您的電腦中的位置。從 **PassThrough.txt** 檔案複製並貼上查詢。按一下下面的 [測試] 按鈕，然後從您的下載位置選取名為 **HelloWorldASA-InputStream.json** 的資料檔案。

![串流分析中的 [測試] 按鈕](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-05.png)

![測試輸入串流](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-06.png)

您可以在瀏覽器中看到查詢的結果，如下所示。

![測試結果](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-07.png)

## 根據條件篩選資料

讓我們試著根據條件篩選結果。我們想要僅針對來自 "SensorA" 的事件顯示結果。查詢位於 **Filtering.txt** 檔案。

![篩選資料串流](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-08.png)

請注意，這裡我們要比較字串值且其區分大小寫。按一下 [重新執行] 按鈕以執行查詢。查詢應該僅針對 1860 個事件傳回 389 個資料列。

![查詢測試的第二個輸出結果](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-09.png)

## 觸發商務工作流程的警示

現在，我們要讓查詢更詳細。對於每一個類型的感應器，如果我們想要監視每 30 秒時段的平均溫度，並且只顯示平均溫度高於 100 度的結果，我們會撰寫下列查詢，然後按一下 [重新執行] 來查看結果。查詢位於 **ThresholdAlerting.txt** 檔案。

![30 秒篩選查詢](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-10.png)

如您所見，現在結果只包含 245 個資料列，而且會列出平均溫度高於 100 的感應器。在此查詢中，我們已依據 **dspl** 分組事件串流，dspl 是感應器名稱，並且有 30 秒的**輪轉視窗**。當我們執行這類暫時查詢時，表明我們想要的進行時間很重要。使用 **TIMESTAMP BY** 子句，我們已指定 [時間] 資料行做為所有暫時計算的進行時間方式。如需詳細資訊，請閱讀有關[時間管理](https://msdn.microsoft.com/library/azure/mt582045.aspx)和[時間範圍函式](https://msdn.microsoft.com/library/azure/dn835019.aspx)的 MSDN 主題。

![溫度超過 100](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-11.png)

## 偵測不存在的事件

我們要如何撰寫查詢來尋找輸入不足事件？ 方法非常簡單。我們來找出上次感應器已傳送資料，但是接下來一分鐘未傳送任何事件的情形。查詢位於 **AbsenseOfEvent.txt** 檔案。

![偵測不存在的事件](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-12.png)

此處，我們使用 **LEFT OUTER JOIN** 至相同的資料串流 (自我聯結)。對於內部聯結，只會在找到相符項目時傳回結果。但是對於 **LEFT OUTER** 聯結，如果聯結左側的事件不相符，則會針對右側資料列的所有資料行傳回具有 NULL 的資料列。這項技術對於尋找不存在的事件很有用。如需 [JOIN](https://msdn.microsoft.com/library/azure/dn835026.aspx) 的詳細資訊，請參閱我們的 MSDN 文件。

![聯結結果](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-13.png)

## 結論

本教學課程的目的在於示範如何撰寫不同的串流分析查詢語言查詢，並在瀏覽器中查看結果。但是，這只是剛開始。您還可以使用串流分析執行更多功能。串流分析支援各種輸入和輸出，並甚至可以運用 Azure Machine Learning 中的函式，因此是分析資料串流的健全工具。您可以使用我們的[學習圖](https://azure.microsoft.com/documentation/learning-paths/stream-analytics/)開始探索更多串流分析，如需撰寫查詢的詳細資訊，請閱讀[常見查詢模式](./stream-analytics-stream-analytics-query-patterns.md)上的文章。

<!---HONumber=AcomDC_0928_2016-->