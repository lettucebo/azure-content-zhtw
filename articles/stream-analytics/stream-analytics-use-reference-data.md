<properties
	pageTitle="在串流分析中使用參考資料和查閱資料表 | Microsoft Azure"
	description="在串流分析查詢中使用參考資料"
	keywords="查閱資料表, 參考資料"
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

# 在串流分析的輸入串流中使用參考資料或查詢資料表

參考資料 (也稱為查詢資料表) 基本上是靜態或不常變更的有限資料集，可用來執行查閱或與資料流相互關聯。若要使用 Azure 串流分析工作中的參考資料，您通常會在查詢中使用[參考資料聯結](https://msdn.microsoft.com/library/azure/dn949258.aspx)。串流分析會使用 Azure Blob 儲存體做為參考資料的儲存層，且可和 Azure Data Factory 參考資料一起轉換和/或複製到來自[任意數目的雲端架構和內部部署資料存放區](../data-factory/data-factory-data-movement-activities.md)的 Azure Blob 儲存體，做為參考資料。參考資料會依 Blob 名稱中指定之日期/時間的遞增順序，以 Blob 序列的形式建立模型 (在輸入組態中定義)。它「只」支援使用比序列中最後一個 Blob 指定之日期/時間「大」的日期/時間來新增到序列的結尾。

## 設定參考資料

若要設定參考資料，您必須先建立屬於「**參考資料**」類型的輸入。下表說明您在建立參考資料輸入及其描述時必須提供的每個屬性：

<table>
<tbody>
<tr>
<td>屬性名稱</td>
<td>說明</td>
</tr>
<tr>
<td>輸入別名</td>
<td>在工作查詢中將用來參考這個輸入的易記名稱。</td>
</tr>
<tr>
<td>儲存體帳戶</td>
<td>您的 blob 檔案所在的儲存體帳戶名稱。如果它與您的「串流分析工作」位於相同的訂用帳戶，您就可以從下拉式清單中選取它。</td>
</tr>
<tr>
<td>儲存體帳戶金鑰</td>
<td>與儲存體帳戶相關聯的密碼金鑰。如果儲存體帳戶與您的「串流分析」工作位於相同的訂用帳戶，就會自動填入此資訊。</td>
</tr>
<tr>
<td>儲存體容器</td>
<td>容器提供邏輯分組給儲存在 Microsoft Azure Blob 服務中的 blob。當您將 blob 上傳至 Blob 服務時，您必須指定該 blob 的容器。</td>
</tr>
<tr>
<td>路徑格式</td>
<td>用來在指定容器中找出 Blob 的檔案路徑。在該路徑內，您也可以指定下列 2 個變數的一個或多個執行個體：<BR>{date}、{time}<BR>範例 1：products/{date}/{time}/product-list.csv<BR>範例 2：products/{date}/product-list.csv
</tr>
<tr>
<td>日期格式 [選用]</td>
<td>如果您已經使用您指定之路徑模式內的 {date}，您可以從支援格式的下拉式清單選取組織檔案所使用的日期格式。範例：YYYY/MM/DD</td>
</tr>
<tr>
<td>時間格式 [選用]</td>
<td>如果您已經使用您指定之路徑模式內的 {time}，您可以從支援格式的下拉式清單選取組織檔案所使用的時間格式。範例：HH</td>
</tr>
<tr>
<td>事件序列化格式</td>
<td>為了確定您的查詢運作如預期，串流分析需要知道您的內送資料流使用哪一種序列化格式。參考資料的支援格式為 CSV 和 JSON。</td>
</tr>
<tr>
<td>編碼</td>
<td>UTF-8 是目前唯一支援的編碼格式</td>
</tr>
</tbody>
</table>

## 產生排程上的參考資料

如果您的參考資料是不常變更的資料集，可以啟用重新整理參考資料的支援，方法是使用 {date} 與 {time} 替代權杖指定輸入設定內的路徑模式。串流分析會根據此路徑模式採用更新的參考資料定義。例如，日期格式為 **“YYYY-MM-DD”** 且時間格式為 **“HH:mm”** 的模式 `sample/{date}/{time}/products.csv`，會指示「串流分析」在 UTC 時區 2015 年 4 月 16 日的下午 5:30 擷取更新的 Blob `sample/2015-04-16/17:30/products.csv`。

> [AZURE.NOTE] 目前串流分析作業只有在機器時間朝向 Blob 名稱中編碼的時間時，才會尋求 Blob 重新整理。例如，作業會儘速尋找 `sample/2015-04-16/17:30/products.csv`，但不早於 UTC 時區 2015 年 4 月 16 日的下午 5:30。它「決不會」尋找編碼時間早於最後一個探索到的檔案。
> 
> 例如，一旦作業找到 Blob `sample/2015-04-16/17:30/products.csv`，就會忽略任何編碼日期早於 2015 年 4 月 16 日下午 5:30 的檔案，所以如果在相同的容器中建立晚到的 `sample/2015-04-16/17:25/products.csv` Blob，作業便不會使用它。
> 
> 同樣地如果 `sample/2015-04-16/17:30/products.csv` 只會在 2015 年 4 月 16 日上午 10:03 產生，但容器中沒有日期較早的 Blob ，則作業會從 2015 年 4 月 16 日上午 10:03 開始使用這個檔案，並在那之前使用先前的參考資料。
> 
> 此情況有例外，就是當工作需要回到之前來重新處理資料，或當工作第一次啟動的時候。啟動時，工作會尋找在指定工作啟動時間之前產生的最新 Blob。這是為了確保在作業啟動時，會有**非空白**的參考資料集。如果無法找到，作業就會顯示下列診斷：`Initializing input without a valid reference data blob for UTC time <start time>`。


[Azure Data Factory](https://azure.microsoft.com/documentation/services/data-factory/) 可用來針對「串流分析」更新參考資料定義時所需的已更新 Blob，協調建立這些 Blob 的工作。Data Factory 是雲端架構資料整合服務，用來協調以及自動移動和轉換資料。Data Factory 可[連線到大量雲端式和內部部署的資料存放區](../data-factory/data-factory-data-movement-activities.md)，也可根據您指定的定期排程輕鬆移動資料。如需詳細資訊及逐步解說指南，瞭解該如何設定 Data Factory 管線才能產生依預先定義排程重新整理的「串流分析」參考資料，請查看此 [GitHub 範例](https://github.com/Azure/Azure-DataFactory/tree/master/Samples/ReferenceDataRefreshForASAJobs)。

## 重新整理參考資料的秘訣 ##

1. 覆寫參考資料 Blob 並不會造成「串流分析」重新載入 Blob，並且在某些情況下，它可能會造成工作失敗。若要變更參考資料，建議的方式是藉由使用工作輸入中定義的相同容器和路徑格式，並使用比序列中最後一個 Blob 指定之日期/時間「大」的日期/時間，加入新的 Blob。
2.	參考資料 Blobs 不是依 Blob 的「上次修改日期」時間排序，而是只依 Blob 名稱中使用 {date} 和 {time} 替代來指定的時間和日期排序。
3.	在一些情況下，工作必須回到之前，因此參照資料必須不被更改或刪除。

## 取得說明
如需進一步的協助，請參閱我們的 [Azure Stream Analytics 論壇](https://social.msdn.microsoft.com/Forums/zh-TW/home?forum=AzureStreamAnalytics)

## 後續步驟
以上就是串流分析 (物聯網資料串流分析專用的受管理服務) 的簡介。若要深入了解此服務，請參閱：

- [開始使用 Azure Stream Analytics](stream-analytics-get-started.md)
- [調整 Azure Stream Analytics 工作](stream-analytics-scale-jobs.md)
- [Azure Stream Analytics 查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)
- [Azure Stream Analytics 管理 REST API 參考](https://msdn.microsoft.com/library/azure/dn835031.aspx)

<!--Link references-->
[stream.analytics.developer.guide]: ../stream-analytics-developer-guide.md
[stream.analytics.scale.jobs]: stream-analytics-scale-jobs.md
[stream.analytics.introduction]: stream-analytics-introduction.md
[stream.analytics.get.started]: stream-analytics-get-started.md
[stream.analytics.query.language.reference]: http://go.microsoft.com/fwlink/?LinkID=513299
[stream.analytics.rest.api.reference]: http://go.microsoft.com/fwlink/?LinkId=517301

<!---HONumber=AcomDC_0921_2016-->