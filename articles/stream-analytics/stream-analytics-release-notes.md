<properties 
	pageTitle="串流分析版本資訊 | Microsoft Azure" 
	description="串流分析版本資訊" 
	services="stream-analytics" 
	documentationCenter="" 
	authors="jeffstokes72" 
	manager="jhubbard" 
	editor="cgronlun"/>

<tags 
	ms.service="stream-analytics" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.tgt_pltfrm="na" 
	ms.workload="data-services" 
	ms.date="07/27/2016" 
	ms.author="jeffstok"/>

#串流分析版本資訊

## 串流分析 2016 年 4 月 15 日版本的注意事項 ##

此版本包含下列更新。

課程名稱 | 說明
---|---
Power BI 輸出的一般可用性 | [Power BI 輸出](stream-analytics-power-bi-dashboard.md)現在已正式運作已移除 Power BI 的 90 天授權到期。如需需要更新授權的案例的詳細資訊，請參閱＜建立 Power BI 儀表板＞的[更新授權](stream-analytics-power-bi-dashboard.md#Renew-authorization)一節。

## 串流分析 2016/03/03 版的注意事項 ##

此版本包含下列更新。

課程名稱 | 說明
---|---
新的串流分析查詢語言項目 | SAQL 現在有 [GetType](https://msdn.microsoft.com/library/azure/mt643890.aspx "GetType MSDN 頁面")、[TRY\_CAST](https://msdn.microsoft.com/library/azure/mt643735.aspx "TRY_CAST MSDN 頁面") 及 [REGEXMATCH](https://msdn.microsoft.com/library/azure/mt643891.aspx "REGEXMATCH MSDN 頁面")。

## 串流分析 2015 年 12 月 10 日版本的注意事項 ##

此版本包含下列更新。

課程名稱 | 說明
---|---
REST API 版本更新 | REST API 的版本已更新至 2015-10-01。您可在[串流分析管理 REST API 參考](https://msdn.microsoft.com/library/azure/dn835031.aspx)和[在串流分析中整合機器學習服務](stream-analytics-how-to-configure-azure-machine-learning-endpoints-in-stream-analytics.md)的 MSDN 中取得詳細資料。
Azure 機器學習的整合 | 此版本開始支援 Azure 機器學習的使用者定義函數。如需詳細資訊，請參閱[教學課程](stream-analytics-machine-learning-integration-tutorial.md)及[一般部落格公告](http://blogs.technet.com/b/machinelearning/archive/2015/12/10/apply-azure-ml-as-a-function-in-azure-stream-analytics.aspx)。

## 串流分析 2015/11/12 版本的注意事項 ##

此版本包含下列更新。

課程名稱 | 說明
---|---
SELECT 的新行為 | 「串流分析」中的 SELECT 已經延伸成允許使用 * 做為巢狀記錄的屬性存取子。如需進一步資訊，請參閱 [http://msdn.microsoft.com/library/mt622759.aspx](http://msdn.microsoft.com/library/mt622759.aspx "複雜資料類型")。

## 串流分析 2015/10/22 版本的注意事項 ##

此版本包含下列更新。

課程名稱 | 說明
---|---
其他查詢語言功能 | 串流分析已藉由納入下列功能來擴充查詢語言：[ABS](https://msdn.microsoft.com/library/azure/mt574054.aspx)、[CEILING](https://msdn.microsoft.com/library/azure/mt605286.aspx)、[EXP](https://msdn.microsoft.com/library/azure/mt605289.aspx)、[FLOOR](https://msdn.microsoft.com/library/azure/mt605240.aspx)、[POWER](https://msdn.microsoft.com/library/azure/mt605287.aspx)、[SIGN](https://msdn.microsoft.com/library/azure/mt605290.aspx)、[SQUARE](https://msdn.microsoft.com/library/azure/mt605288.aspx) 及 [SQRT](https://msdn.microsoft.com/library/azure/mt605238.aspx)。
移除了彙總限制 | 此版本移除一個查詢 15 個彙總的限制。現在每一個查詢的彙總數目沒有限制。
新增了 GROUP BY System.Timestamp 功能 | [GROUP BY](https://msdn.microsoft.com/library/azure/dn835023.aspx) 函式現在允許用於 window\_type 或 [System.Timestamp](https://msdn.microsoft.com/library/azure/mt598501.aspx)。
新增了適用於輪轉和跳動視窗的 OFFSET | 根據預設，[輪轉](https://msdn.microsoft.com/library/azure/dn835055.aspx)和[跳動](https://msdn.microsoft.com/library/azure/dn835041.aspx)視窗已對準零點時間 (國際標準時間 0001 年 1 月 1 日上午 12:00:00)。新的 (選擇性) 參數 'offsetsize' 允許指定自訂的 位移 (或對齊)。


## 串流分析 2015/09/29 版本的注意事項 ##

此版本包含下列更新。

課程名稱 | 說明
---|---
Azure IoT Suite 公用預覽 | Azure IoT Suite 公用預覽包含串流分析。
Azure 入口網站整合 | 串流分析除了會繼續出現在 Azure 管理入口網站之外，它現在也已整合到 [Azure 入口網站](https://azure.microsoft.com/overview/preview-portal/)中。請注意，Preview 入口網站中的「串流分析」功能目前是 Azure 管理入口網站所提供功能的子集，不支援瀏覽器中查詢測試、Power BI 輸出組態，以及在您可存取的訂用帳戶中瀏覽或建立新的輸入和輸出資源。
支援 DocumentDB 輸出 | 串流分析工作現在可以輸出到 [DocumentDB](https://azure.microsoft.com/services/documentdb/)。
支援 IoT 中樞輸入 | 串流分析作業現在可以內嵌來自 IoT 中樞的資料。
異質事件的時間戳記格式 | 當單一資料流包含數個在不同欄位有不同時間戳記的事件類型時，您現在已可以使用 [TIMESTAMP BY](http://msdn.microsoft.com/library/mt573293.aspx) 搭配運算式為每個案例指定不同的時間戳記欄位。

## 資料流分析 2015/09/10 版本的注意事項 ##

此版本包含下列更新。

課程名稱|說明
---|---
支援 PowerBI 群組|為了能夠與其他 Power BI 使用者共用資料，串流分析工作現在可寫入您 Power BI 帳戶內的 [PowerBI 群組](stream-analytics-define-outputs.md#power-bi)。

## 串流分析 2015 年 8 月 20 日版本的注意事項 ##

此版本包含下列更新。

課程名稱|說明
---|---
加入 LAST 函式 |在串流分析工作中已可使用 [LAST](http://msdn.microsoft.com/library/mt421186.aspx) 函式，讓您擷取指定時間範圍內事件資料流中的最新事件。
新的陣列函數|現在已可使用陣列函式 [GetArrayElement](http://msdn.microsoft.com/library/mt270218.aspx)、[GetArrayElements](http://msdn.microsoft.com/library/mt298451.aspx) 及 [GetArrayLength](http://msdn.microsoft.com/library/mt270226.aspx)。
新的記錄函數|現在已可使用記錄函式 [GetRecordProperties](http://msdn.microsoft.com/library/mt270221.aspx) 和 [GetRecordPropertyValue](http://msdn.microsoft.com/library/mt270220.aspx)。

## 串流分析 07/30/2015 版本的注意事項 ##

此版本包含下列更新。

課程名稱|說明
---|---
與 Azure 識別碼分離的 Power BI 組織識別碼|此功能可為任何 Azure 帳戶類型 (Live ID 或組織識別碼) 下的 ASA 工作啟用 [Power BI 輸出](stream-analytics-power-bi-dashboard.md)。此外，您可以擁有 Azure 帳戶的組織識別碼，並使用另一個識別碼用於授權 Power BI 輸出。
支援服務匯流排佇列輸出|現在您可以在「串流分析」工作中使用[服務匯流排佇列](stream-analytics-connect-data-event-outputs.md#service-bus-queues)輸出。
支援服務匯流排主題輸出|現在您可以在「串流分析」工作中使用[服務匯流排主題](stream-analytics-connect-data-event-outputs.md#service-bus-topics)輸出。

## 串流分析 2015/07/09 版本的注意事項 ##

此版本包含下列更新。


課程名稱|說明
---|---
自訂 Blob 輸出資料分割|Blob 儲存體輸出現在允許指定寫入輸出 Blob 的頻率，以及輸出資料路徑資料夾結構的結構和格式的選項。 

## 串流分析 2015/05/03 版本的注意事項 ##

此版本包含下列更新。


課程名稱|說明
---|---
增加次序錯誤允許的時間範圍的最大值|次序錯誤允許的時間範圍的大小上限現在是 59:59 (MM:SS)
JSON 輸出格式：以行分隔或陣列|現在當輸出至 Blob 儲存體或事件中心時，還有另一個選擇，就是以 JSON 物件的陣列輸出，或透過以新行分隔 JSON 物件來輸出。 

## 串流分析 2015/04/16 版本的注意事項 ##


課程名稱|說明
---|---
Azure 儲存體帳戶組態的延遲|第一次在某個區域中建立串流分析工作時，系統會提示您建立新的儲存體帳戶或指定現有帳戶，以監視該區域中的串流分析工作。由於設定監視作業會有所延遲，若在 30 分鐘內於相同區域中建立另一個串流分析工作，系統將提示您指定第二個儲存體帳戶，而不會在 [監視儲存體帳戶] 下拉式清單中顯示最近設定的帳戶。若要避免建立不必要的儲存體帳戶，請在第一次於某區域中建立工作的 30 分鐘後，再於該區域中佈建其他工作。
工作升級|串流分析目前無法對執行中工作的定義或組態進行即時編輯。若要變更執行中工作的輸入、輸出、查詢、調整或組態，您必須先停止工作。
從輸入來源推斷而來的資料類型|如果未使用 CREATE TABLE 陳述式，系統會從輸入格式衍生輸入類型，例如 CSV 的所有欄位都是字串。必須使用 CAST 函數，將欄位明確轉換成正確的類型，以避免發生類型不一致錯誤。
遺漏的欄位在輸出後會變成 null 值|參考的欄位如果在輸入來源中找不到，將會在輸出事件中產生 null 值。
WITH 陳述式前面必須是 SELECT 陳述式|在您的查詢中，SELECT 陳述式後面必須是 WITH 陳述式中定義的下列子查詢。
記憶體不足問題|串流分析工作如果可以容忍大量的失控事件和/或存在大量狀態的複雜查詢，可能會導致工作耗盡記憶體，進而造成工作重新開始。啟動作業和停止作業會顯示在工作的作業記錄中。若要避免此行為，請將查詢橫向擴充至多個資料分割。在未來版本中，將會藉由降低受影響之工作的效能 (而不是加以重新啟動)，來解決這項限制。
沒有裝載時間戳記的大型 Blob 輸入，可能會造成記憶體不足問題|Blob 儲存體用於大量的檔案時，如果未透過 TIMESTAMP BY 指定時間戳記欄位，則可能會造串流分析工作崩潰。要避免此問題，請讓每個 Blob 的大小保持在 10 MB 以下。
SQL Database事件容量限制|使用 SQL Database 做為輸出來源時，非常大量的輸出資料可能會導致串流分析工作無法在限定的時間內完成。要解決這個問題，請使用彙總或篩選運算子減少輸出量，或改以 Azure Blob 儲存體或事件中樞做為輸出來源。
PowerBI 資料集只能包含一個資料表|PowerBI 不允許指定的資料集中存在多個資料表。

## 取得說明
如需進一步的協助，請參閱我們的 [Azure Stream Analytics 論壇](https://social.msdn.microsoft.com/Forums/zh-TW/home?forum=AzureStreamAnalytics)

## 後續步驟

- [Azure Stream Analytics 介紹](stream-analytics-introduction.md)
- [開始使用 Azure Stream Analytics](stream-analytics-get-started.md)
- [調整 Azure Stream Analytics 工作](stream-analytics-scale-jobs.md)
- [Azure Stream Analytics 查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)
- [Azure 串流分析管理 REST API 參考](https://msdn.microsoft.com/library/azure/dn835031.aspx)
 

<!---HONumber=AcomDC_0921_2016-->