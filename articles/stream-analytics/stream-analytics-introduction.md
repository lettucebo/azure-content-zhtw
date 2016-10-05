<properties 
	pageTitle="串流分析簡介 | Microsoft Azure" 
	description="了解串流分析，這是可協助您即時分析物聯網 (IoT) 資料流的受管理服務。" 
	keywords="分析服務, 受管理服務, 串流處理, 串流分析, 什麼是串流分析"
	services="stream-analytics" 
	documentationCenter="" 
	authors="jeffstokes72" 
	manager="jhubbard" 
	editor="cgronlun"/>

<tags 
	ms.service="stream-analytics" 
	ms.devlang="na" 
	ms.topic="get-started-article" 
	ms.tgt_pltfrm="na" 
	ms.workload="data-services" 
	ms.date="09/26/2016" 
	ms.author="jeffstok"/>


# 什麼是串流分析？

Azure 串流分析是完全受管理、符合成本效益的即時事件處理引擎，可解除鎖定資料的深入見解。串流分析可讓您輕易地對串流自裝置、感應器、網站、社交媒體、應用程式、基礎結構系統和其他項目的資料設定即時分析計算。

只要在 Azure 入口網站中按幾下，您就可以撰寫串流分析作業，指定串流資料的輸入來源、工作結果的輸出接收，以及利用類似 SQL 的語言表示的資料轉換。您可以在 Azure 入口網站中監視和調整工作的規模/速度，可從數 KB 調整到一 GB 或每秒處理更多事件。

串流分析運用多年的 Microsoft Research 工作成果為時效性處理作業開發出高度量身訂做的串流引擎，以及這類作業的直覺式規格語言整合功能。

## 串流分析的用途為何？
現今，大量的資料都需要透過網路高速流動。組織若能即時處理串流資料並採取相關動作，將可大幅改善效率，並在市場上脫穎而出。各種產業都有應用即時串流分析的狀況：金融服務公司所提供的個人化、即時股票交易分析和警示、即時詐騙偵測、資料及身分識別保護服務、可靠地擷取和分析內嵌在實體物件 (物聯網，又稱 IoT) 中的感應器和傳動器所產生的資料、網頁點選流分析，以及客戶經驗若在某個時間範圍內降低時會發出警示的客戶關係管理 (CRM) 應用程式。企業正尋求以最具彈性、可靠且符合成本效益的方式執行這類即時事件資料流資料分析，以在高度競爭的現代商務世界中獲致成功。

## 主要功能和優點
-   **容易使用：**串流分析用來說明轉換的簡單、宣告式查詢模型。為了最佳化以方便使用，串流分析可使用 T-SQL 變數，並且為客戶省去處理串流處理系統之技術複雜性的需求。使用瀏覽器中查詢編輯器中的[串流分析查詢語言](https://msdn.microsoft.com/library/azure/dn834998.aspx)，您可讓 intelli-sense 自動完成，以協助您快速而輕鬆地實作時間序列查詢，包括以時間為基礎的聯結、視窗彙總、暫時性篩選器，以及其他常見作業，例如聯結、彙總、預測及篩選器等。此外，對範例資料檔案進行瀏覽器中查詢測試，也能加快反覆開發作業的速度。

-   **延展性：**串流分析能夠處理高達 1 GB/秒的高事件輸送量。與 [Azure 事件中樞](https://azure.microsoft.com/services/event-hubs/)和 [Azure IoT 中樞](https://azure.microsoft.com/services/iot-hub/)整合，可讓解決方案每秒擷取數百萬來自連線裝置、點擊資料流及記錄檔等項目的事件。為達到此目的，串流分析運用事件中樞資料分割功能，每個資料分割可產生 1 MB/秒。使用者可將計算分割成查詢定義內的數個邏輯步驟，且每個步驟都能夠進一步分割以增加研展性。

-   **可靠性、可重複性及快速修復：**串流分析雲端受管理服務可透過內建的復原功能，在失敗時防止資料遺失並讓業務繼續營運。此服務具備內部維護狀態的能力，因此會提供可重複的結果，確保能封存事件並在日後重新套用處理，且一律取得相同的結果。這項功能可讓客戶及時回頭檢視，並在執行根本原因分析和假設狀況分析等案例時調查計算方式。

-   **低成本：**屬於雲端服務的串流分析已最佳化，可讓使用者以非常低廉的成本開始運作並維護即時分析解決方案。該服務的建置可讓您根據串流單位使用量和系統處理的資料量執行隨用隨付。使用量衍生自所處理的事件量，以及佈建在叢集內，用來處理個別串流分析工作的計算能力。

-   **參考資料：**串流分析可讓使用者指定及使用參考資料。這可能是歷史資料，或只是變動頻率較低的非串流資料。系統可以簡化參考資料的使用，使資料被視為任何其他要與即時擷取的其他事件資料流聯結以執行轉換的傳入事件資料流。

-   **使用者定義函式︰**串流分析已與 Azure Machine Learning 整合，可隨著串流分析查詢定義 Machine Learning 中的函式呼叫。這能擴充串流分析的功能，以便利用現有的 Azure Machine Learning 解決方案。如需進一步資訊，請檢閱 [Machine Learning 整合教學課程](stream-analytics-machine-learning-integration-tutorial.md)。

-   **連接性：**串流分析會直接連接到 Azure 事件中樞和 Azure IoT 中樞進行串流擷取，並連接到 Azure Blob 服務以取得歷史資料。結果可以從串流分析寫入到 Azure 儲存體 Blob 或資料表、Azure SQL DB、Azure Data Lake Store、DocumentDB、事件中樞、Azure 服務匯流排主題或佇列以及 Power BI，它可在其中視覺化、由工作流程進一步處理、透過 [Azure HDInsight](https://azure.microsoft.com/services/hdinsight/) 在批次分析中使用，或以一系列事件的方式再次處理。使用事件中樞時，您可以將多個串流分析與其他資料來源和處理引擎組合在一起，而不會失去計算的串流本質。

## 取得說明
如需進一步的協助，請參閱我們的 [Azure Stream Analytics 論壇](https://social.msdn.microsoft.com/Forums/zh-TW/home?forum=AzureStreamAnalytics)

## 後續步驟
以上就是串流分析 (物聯網資料串流分析專用的受管理服務) 的簡介。若要深入了解此服務，請參閱：

- [開始使用 Azure Stream Analytics](stream-analytics-get-started.md)
- [調整 Azure Stream Analytics 工作](stream-analytics-scale-jobs.md)
- [Azure Stream Analytics 查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)
- [Azure 串流分析管理 REST API 參考](https://msdn.microsoft.com/library/azure/dn835031.aspx)

<!---HONumber=AcomDC_0928_2016-->