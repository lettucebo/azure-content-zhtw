<properties
	pageTitle="使用 Azure Functions 從 Azure 串流分析輸出至 Azure Redis 快取 | Microsoft Azure"
	description="了解如何使用連接到服務匯流排佇列的 Azure 函式，從串流分析工作的輸出填入 Azure Redis 快取。"
	keywords="data stream, redis cache, service bus queue, 資料流, redis 快取, 服務匯流排佇列"
	services="stream-analytics"
	authors="ryancrawcour"
	manager="jhubbard"
    documentationCenter=""
	/>

<tags
	ms.service="stream-analytics"
	ms.devlang="na"
	ms.topic="article"
	ms.tgt_pltfrm="na"
	ms.workload="na"
	ms.date="08/09/2016"
	ms.author="ryancraw"/>

# 如何使用 Azure Functions 在 Azure Redis 快取中儲存 Azure 串流分析的資料

Azure 串流分析可讓您快速開發及部署低成本的解決方案，即時獲取裝置、感應器、基礎結構與應用程式，或任何資料流的深入解析。它可啟用不同的使用案例，例如即時管理與監視、命令與控制、詐騙偵測、Connected Cars 等等。在許多類似情況下，您可能希望將 Azure 串流分析輸出的資料儲存到分散式資料存放區，例如 Azure Redis 快取。

假設您是電信公司的員工。您正在嘗試偵測 SIM 卡詐騙，並尋找於相同時間來自相同身分，卻位於不同地理位置的多個通話。您的工作是將所有潛在的詐騙電話儲存在 Azure Redis 快取中。在此部落格中，我們將提供如何輕鬆地完成此工作的指引。

## 必要條件
完成 ASA 的[即時詐騙偵測][fraud-detection]逐步解說

## 架構概觀
![架構的螢幕擷取畫面](./media/stream-analytics-functions-redis/architecture-overview.png)

如上圖所示，串流分析允許查詢串流輸入資料並將之傳送至輸出。根據輸出，Azure Functions 可以接著觸發某些事件類型。

在此部落格中，我們會著重在此管線中 Azure Functions 的部分，若要更明確地說，則是觸發將詐騙資料儲存到快取的事件。完成[即時詐騙偵測][fraud-detection]教學課程之後，您將會有已設定並正在執行的輸入 (事件中樞)、查詢，以及輸出 (Blob 儲存體)。在此部落格中，我們將輸出改為使用「服務匯流排佇列」。接下來，我們將一個 Azure 函式與這個佇列連接。

## 建立並連接服務匯流排佇列輸出
若要建立服務匯流排佇列，請遵循[開始使用服務匯流排佇列][servicebus-getstarted]中＜.NET＞小節的步驟 1 和 2。現在讓我們將佇列連接到先前在詐騙偵測逐步解說中所建立的串流分析工作。



1. 在 Azure 入口網站中，移至工作的 [輸出] 刀鋒視窗，然後選取頁面頂端的 [新增]。

	![加入輸出](./media/stream-analytics-functions-redis/adding-outputs.png)

2. 選擇 [服務匯流排佇列] 為 [接收]，並遵循螢幕上的指示。請務必選擇您在[開始使用服務匯流排佇列][servicebus-getstarted]中建立的服務匯流排佇列命名空間。當您完成時，請按一下「右鍵」。
3. 指定下列值：
	- **事件序列化程式格式**：JSON
	- **編碼**：UTF8
	- **格式**：列分隔

4. 按一下 [建立] 按鈕以新增這個來源，並確認串流分析可以成功連接到儲存體帳戶。

5. 在 [查詢] 索引標籤上，以下列項目取代目前的查詢。使用您在步驟 3 中所建立的輸出名稱取代 [您的服務匯流排名稱]。

    ```    

	    SELECT 
            System.Timestamp as Time, CS1.CallingIMSI, CS1.CallingNum as CallingNum1, 
            CS2.CallingNum as CallingNum2, CS1.SwitchNum as Switch1, CS2.SwitchNum as Switch2

        INTO [YOUR SERVICE BUS NAME]
    
        FROM CallStream CS1 TIMESTAMP BY CallRecTime
	    JOIN CallStream CS2 TIMESTAMP BY CallRecTime
            ON CS1.CallingIMSI = CS2.CallingIMSI AND DATEDIFF(ss, CS1, CS2) BETWEEN 1 AND 5
    
        WHERE CS1.SwitchNum != CS2.SwitchNum
    
    ```

## 建立 Azure Redis 快取
遵循[如何使用 Azure Redis 快取][use-rediscache]中＜.NET＞小節，直到＜設定快取用戶端＞一節的內容，來建立 Azure Redis 快取。完成後，您將具有新的 Redis 快取。在 [所有設定] 底下，選取 [存取金鑰] 並記下 [主要連接字串] 的內容。

![架構的螢幕擷取畫面](./media/stream-analytics-functions-redis/redis-cache-keys.png)

## 建立 Azure 函式
遵循[建立您的第一個 Azure 函式][functions-getstarted]教學課程以開始使用 Azure Functions。如果您已經有想要使用的 Azure 函式，則請直接跳到[寫入至 Redis 快取](#Writing-to-Redis-Cache)

1. 在入口網站中，從左側導覽中選取 [應用程式服務]，然後按一下您的 Azure 函數應用程式名稱，以取得函式的應用程式網站。![應用程式服務函式清單的螢幕擷取畫面](./media/stream-analytics-functions-redis/app-services-function-list.png)

2. 按一下 [新增函式] > [ServiceBusQueueTrigger – C#]。針對下列欄位，請遵循下列指示︰
	- **佇列名稱**：與您在[開始使用服務匯流排佇列][servicebus-getstarted]中建立佇列時所輸入的名稱相同 (不是服務匯流排的名稱)。請確定您式使用連線到串流分析輸出的佇列。
	- **服務匯流排連接**︰選取 [加入連接字串]。若要尋找連接字串，請移至傳統入口網站，依序選取 [服務匯流排]、您所建立的服務匯流排，以及畫面底部的 [連接資訊]。請確定您是位於此頁面的主畫面上。請複製並貼上連接字串。請隨意輸入任何連接名稱。
	
		![服務匯流排連線的螢幕擷取畫面](./media/stream-analytics-functions-redis/servicebus-connection.png)
	- **AccessRights**︰選擇 [管理]


3. 按一下 [建立]

## 寫入至 Redis 快取
我們現在已經建立一個可從服務匯流排佇列進行讀取的 Azure 函式。我們還需使用函式將此資料寫入 Redis 快取。

1. 選取您剛建立的 **ServiceBusQueueTrigger**，並按一下右上角的 [函數應用程式設定]。選取 [移至 App Service 設定] > [設定] > [應用程式設定]

2. 在 [連接字串] 區段中，於 [名稱] 區段中建立一個名稱。在 [值] 區段中，貼上您在 [建立 Redis 快取] 步驟中找到的主要連接字串。在 [SQL 資料庫]的位置選取 [自訂]。

3. 按一下頂端的 [儲存]。

	![服務匯流排連線的螢幕擷取畫面](./media/stream-analytics-functions-redis/function-connection-string.png)

4. 現在回到 [App Service 設定]，然後選取 [工具] > [App Service 編輯器 (預覽)] > [開啟] > [前往]。

	![服務匯流排連線的螢幕擷取畫面](./media/stream-analytics-functions-redis/app-service-editor.png)

5. 在您選擇的編輯器中，使用下列項目建立名為 **project.json** 的 JSON 檔案，並將它儲存至本機磁碟。

        {
            "frameworks": {
		        "net46": {
		            "dependencies": {
				        "StackExchange.Redis":"1.1.603",
				        "Newtonsoft.Json": "9.0.1"
			        }
		        }
	        }
        }

6. 將此檔案上傳至您函式的根目錄 (不是 WWWROOT)。您應該會看到名為 **project.lock.json** 的檔案自動顯示，確認 Nuget 封裝 "StackExchange.Redis" 和 "Newtonsoft.Json" 已經匯入。

7. 在 **run.csx** 檔案中，使用下列程式碼取代預先產生的程式碼。在 lazyConnection 函式中，使用您在**將資料儲存至 Redis 快取**的步驟 2 中所建立的名稱取代 "CONN NAME"。

````

	using System;
    using System.Threading.Tasks;
    using StackExchange.Redis;
    using Newtonsoft.Json;
    using System.Configuration;
    
    public static void Run(string myQueueItem, TraceWriter log)
    {
        log.Info($"Function processed message: {myQueueItem}");

        // Connection refers to a property that returns a ConnectionMultiplexer
        IDatabase db = Connection.GetDatabase();
        log.Info($"Created database {db}");
    
        // Parse JSON and extract the time
        var message = JsonConvert.DeserializeObject<dynamic>(myQueueItem);
        string time = message.time;
        string callingnum1 = message.callingnum1;

        // Perform cache operations using the cache object...
        // Simple put of integral data types into the cache
        string key = time + " - " + callingnum1;
        db.StringSet(key, myQueueItem);
        log.Info($"Object put in database. Key is {key} and value is {myQueueItem}");

        // Simple get of data types from the cache
        string value = db.StringGet(key);
        log.Info($"Database got: {value}"); 
    }

    // Connect to the Service Bus
    private static Lazy<ConnectionMultiplexer> lazyConnection = 
		new Lazy<ConnectionMultiplexer>(() =>
    		{
				var cnn = ConfigurationManager.ConnectionStrings["CONN NAME"].ConnectionString
        		return ConnectionMultiplexer.Connect();
    		});
    
    public static ConnectionMultiplexer Connection
    {
        get
        {
            return lazyConnection.Value;
        }
    }

````

## 啟動串流分析工作

1. 啟動 telcodatagen.exe 應用程式。使用方式如下：````telcodatagen.exe [#NumCDRsPerHour] [SIM Card Fraud Probability] [#DurationHours]````

2. 從入口網站中的 [串流分析工作] 刀鋒視窗中，按一下頁面頂端的 [啟動] 。

	![開始工作的螢幕擷取畫面](./media/stream-analytics-functions-redis/starting-job.png)

3. 在顯示的 [開始工作] 刀鋒視窗中，選取 [立即] 然後按一下畫面底部的 [啟動] 按鈕。工作狀態會變更為 [啟動中]，稍後則會變更為 [執行中]。
 
	![開始工作時間範圍的螢幕擷取畫面](./media/stream-analytics-functions-redis/start-job-time.png)

## 執行解決方案並檢查結果
回到您的 **ServiceBusQueueTrigger** 頁面上，您現在應該會看到記錄陳述式。這些記錄檔顯示您從服務匯流排佇列取得某些項目、將該項目放入資料庫，並使用時間做為索引鍵將該項目擷取出來！

若要確認您的資料位於 Redis 快取中，請移至新入口網站中的 Redis 快取頁面 (如先前在[建立 Azure Redis 快取](#Create-an-Azure-Redis-Cache)步驟中所示)，然後選取主控台。

您現在可以撰寫 Redis 命令以確認資料實際位於快取中。

![Redis 主控台的螢幕擷取畫面](./media/stream-analytics-functions-redis/redis-console.png)

## 後續步驟
我們對於 Azure Functions 及串流分析可以協力執行的新功能感到十分興奮，並希望這會為您帶來新的可能性。如果您對自己所需要的功能有任何意見反應，歡迎使用 [Azure UserVoice 網站](https://feedback.azure.com/forums/270577-stream-analytics)。

如果您不熟悉 Microsoft Azure，我們邀請您透過註冊[免費 Azure 試用帳戶](https://azure.microsoft.com/pricing/free-trial/)來親自嘗試。如果您不熟悉串流分析，我們邀請您[建立您的第一個串流分析工作](stream-analytics-create-a-job.md)。

如果您需要任何協助或有任何問題，請在 [MSDN](https://social.msdn.microsoft.com/Forums/zh-TW/home?forum=AzureStreamAnalytics) 或 [Stackoverflow](http://stackoverflow.com/questions/tagged/azure-stream-analytics) 論壇中提出。

您也可以查看下列資源︰

- [Azure Functions 開發人員參考](../azure-functions/functions-reference.md)
- [Azure Functions C# 開發人員參考](../azure-functions/functions-reference-csharp.md)
- [Azure Functions F# 開發人員參考](../azure-functions/functions-reference-fsharp.md)
- [Azure Functions NodeJS 開發人員參考](../azure-functions/functions-reference.md)
- [Azure Functions 觸發程序和繫結](../azure-functions/functions-triggers-bindings.md)
- [如何監視 Azure Redis 快取](../redis-cache/cache-how-to-monitor.md)

若要隨時掌握所有最新消息及功能，請在 Twitter 上追蹤 [@AzureStreaming](https://twitter.com/AzureStreaming)。


[fraud-detection]: stream-analytics-real-time-fraud-detection.md
[servicebus-getstarted]: ../service-bus/service-bus-dotnet-get-started-with-queues.md
[use-rediscache]: ../redis-cache/cache-dotnet-how-to-use-azure-redis-cache.md
[functions-getstarted]: ../azure-functions/functions-create-first-azure-function.md

<!---HONumber=AcomDC_0921_2016-->