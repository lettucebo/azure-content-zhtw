<properties
	pageTitle="調整串流分析工作以增加輸送量 | Microsoft Azure"
	description="了解如何透過設定輸入資料分割、微調查詢定義，及設定工作串流處理單元來調整串流分析工作。"
	keywords="資料串流處理, 串流資料處理, 微調分析"
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

# 調整 Azure 串流分析工作，以提高串流資料處理的輸送量

了解如何微調分析工作及計算串流分析的「串流處理單元」，以及如何藉由設定輸入資料分割、微調查詢定義，及設定工作串流處理單元來調整串流分析工作。

## 串流分析工作由哪些部分所組成？
串流分析工作的定義包含輸入、查詢及輸出。輸入是工作從中讀取串流之處，查詢則是用來轉換輸入串流，而輸出是工作將工作結果傳送到的位置。

一個工作至少需要一個輸入來源來進行資料串流。資料流輸入來源可以儲存在 Azure 服務匯流排事件中樞或 Azure Blob 儲存體。如需詳細資訊，請參閱 [Azure 串流分析簡介](stream-analytics-introduction.md)和[開始使用 Azure 串流分析](stream-analytics-get-started.md)。

## 設定串流處理單位
串流處理單位 (SU) 代表執行 Azure 串流分析工作的資源和能力。SU 會根據 CPU、記憶體，以及讀寫率的混合量值，提供一個方式來描述相關的事件處理容量。每個串流處理單元會對應至約 1 MB/秒的輸送量。

選擇特定工作所需的 SU 數量取決於輸入的磁碟分割設定以及針對工作定義的查詢。您可以使用 Azure 傳統入口網站，為工作選取串流處理單元的配額上限。在特定區域中，每個 Azure 訂用帳戶的所有分析工作的預設資料流單位配額上限為 50。若要增加訂用帳戶的資料流單位，請連絡 [Microsoft 支援服務](http://support.microsoft.com)。

工作可以使用的串流處理單元數目，取決於輸入的資料分割設定以及為工作定義的查詢。此外請注意，必須使用有效的串流處理單元的值。有效值從 1、3、6 開始，然後以 6 向上遞增，如下所示。

![Azure 串流分析的串流處理單元調整][img.stream.analytics.streaming.units.scale]

本文將說明如何計算及微調查詢，以增加分析工作的輸送量。

## 窘迫平行作業
窘迫平行作業是我們在 Azure 串流分析中調整性最高的案例。它將對於查詢執行個體之輸入的某個資料分割，連接到輸出的某個資料分割。為達到此平行處理原則，您必須滿足幾項需求：

1.  如果您的查詢邏輯所相依於的索引鍵，就是同一個執行個體所處理的索引鍵，則您必須確保事件移至您輸入的資料分割。在事件中樞的案例中，這代表事件資料必須要有 **PartitionKey** 組，或是您可以使用已分割的傳送者。對於 Blob 而言，這代表事件是傳送到相同的資料分割資料夾。如果您的查詢邏輯並不需要由同一個查詢執行個體所處的相同索引鍵，您可以忽略這項需求。例如，簡單的選取/專案/篩選查詢。
2.	當資料的配置方式像是它必須在輸入端時，我們就需要確保您的查詢已分割。這需要您在所有步驟中使用 [Partition By]。您可以使用多重步驟，但這些步驟都必須以相同的索引鍵來分割。另外要注意的是，目前的資料分割索引鍵必須設定為 **PartitionId**，才能擁有完全平行作業。
3.	目前只有事件中樞和 Blob 支援已分割的輸出。對於事件中樞輸出，您必須將 [PartitionKey] 欄位設定為「PartitionId」。而對於 Blob，您不需要採取任何動作。
4.	另外要注意的是，輸入資料分割的數目必須與輸出資料分割的數目相同。Blob 輸出目前並不支援資料分割，但沒有關係，因為它會繼承上游查詢的資料分割配置。允許完全平行作業的資料分割值範例：
	1.	8 個事件中樞輸入資料分割，和 8 個事件中樞輸出資料分割
	2.	8 個事件中樞輸入資料分割，和 Blob 輸出
	3.	8 個 Blob 輸入資料分割，和 Blob 輸出
	4.	8 個 Blob 輸入資料分割，和 8 個事件中樞輸出資料分割

以下是幾個窘迫平行的範例案例。

### 簡單查詢
輸入 – 有 8 個資料分割輸出的事件中樞 – 有 8 個資料分割的事件中樞

**查詢：**

    SELECT TollBoothId
    FROM Input1 Partition By PartitionId
    WHERE TollBoothId > 100

這個查詢是簡單的篩選，我們不必擔心要分割我們傳送至事件中樞的輸入。您會注意到查詢擁有 [PartitionId] 的 [Partition By]，因此我們滿足了上述的需求 #2。對於輸出，我們必須設定作業中的事件中樞輸出，以便把 [PartitionKey] 欄位設定為「PartitionId」。再檢查一次，輸入資料分割 == 輸出資料分割。這個拓撲的確是窘迫平行。

### 利用群組索引鍵的查詢
輸入 – 有 8 個資料分割輸出的事件中樞 – Blob

**查詢：**

    SELECT COUNT(*) AS Count, TollBoothId
    FROM Input1 Partition By PartitionId
    GROUP BY TumblingWindow(minute, 3), TollBoothId, PartitionId

這個查詢有群組索引鍵，且同一個查詢執行個體必須處理相同的索引鍵。這代表我們必須以資料分割的方式，把事件傳送至事件中樞。我們該關心哪個索引鍵？ **PartitionId** 是作業邏輯概念，我們真正關心的索引鍵是 **TollBoothId**。這代表我們應該要把我們傳送到事件中樞之事件資料的 [PartitionKey]，設定為事件的 [TollBoothId]。查詢擁有 [PartitionId] 的 [Partition By]，因此沒問題了。對於輸出，因為它是 Blob，所以我們不需要擔心設定 **PartitionKey** 的問題。而對於需求 #4，同樣地，因為這是 Blob，所以我們不需要擔心。這個拓撲的確是窘迫平行。

### 利用群組索引鍵的多重步驟查詢 ###
輸入 – 有 8 個資料分割輸出的事件中樞 – 有 8 個資料分割的事件中樞

**查詢：**

    WITH Step1 AS (
    SELECT COUNT(*) AS Count, TollBoothId, PartitionId
    FROM Input1 Partition By PartitionId
    GROUP BY TumblingWindow(minute, 3), TollBoothId, PartitionId
    )
    
    SELECT SUM(Count) AS Count, TollBoothId
    FROM Step1 Partition By PartitionId
    GROUP BY TumblingWindow(minute, 3), TollBoothId, PartitionId

這個查詢有群組索引鍵，且同一個查詢執行個體必須處理相同的索引鍵。我們可以使用與上一個查詢相同的策略。這個查詢有多重步驟。每個步驟都有 [PartitionId] 的 [Partition By] 嗎？ 有，所以沒問題了。針對輸出，我們必須把 [PartitionKey] 設定為[PartitionId]，就跟上面討論的一樣，且我們也得讓資料分割的數目跟輸入的一樣。這個拓撲的確是窘迫平行。


## 並非窘迫平行的範例案例

### 不相符的資料分割數目 ###
輸入 – 有 8 個資料分割輸出的事件中樞 – 有 32 個資料分割的事件中樞

查詢是什麼並不重要，因為在這個案例中，輸入資料分割數目 ! = 輸出資料分割數目。

### 不把事件中樞或 Blob 當做輸出來使用
輸入 – 有 8 個資料分割輸出的事件中樞 – PowerBI

PowerBI 輸出目前並不支援資料分割。

### 有不同 Partition By 值的多重步驟查詢
輸入 – 有 8 個資料分割輸出的事件中樞 – 有 8 個資料分割的事件中樞

**查詢：**

    WITH Step1 AS (
    SELECT COUNT(*) AS Count, TollBoothId, PartitionId
    FROM Input1 Partition By PartitionId
    GROUP BY TumblingWindow(minute, 3), TollBoothId, PartitionId
    )
    
    SELECT SUM(Count) AS Count, TollBoothId
    FROM Step1 Partition By TollBoothId
    GROUP BY TumblingWindow(minute, 3), TollBoothId
    
如您所見，第二個步驟把[TollBoothId] 當做資料分割索引鍵來使用。這跟第一個步驟中的不同，因此需要我們來更換一下。

我們有幾個串流分析工作的範例和反例，能夠達到窘迫平行拓撲，且有可能會達到最大級別。對於不符合任何這些情況的作業，將來會有更新來詳述如何以最大幅度調整某些其他的 Canonical 串流分析案例。

現在，請利用下列的一般指引：

## 計算工作的最大串流處理單元
資料流分析工作可使用的串流處理單元總數，取決於為工作定義之查詢中的步驟數目，和每個步驟的資料分割數目。

### 查詢中的步驟
一個查詢可以有一或多個步驟。每個步驟都是使用 **WITH** 關鍵字定義的子查詢。唯一不在 **WITH** 關鍵字範圍內的查詢，也被視為步驟，例如下列查詢中的 **SELECT** 陳述式：

	WITH Step1 AS (
		SELECT COUNT(*) AS Count, TollBoothId
		FROM Input1 Partition By PartitionId
		GROUP BY TumblingWindow(minute, 3), TollBoothId, PartitionId
	)

	SELECT SUM(Count) AS Count, TollBoothId
	FROM Step1
	GROUP BY TumblingWindow(minute,3), TollBoothId

上述查詢有兩個步驟。

> [AZURE.NOTE] 在本文稍後將說明此範例查詢。

### 分割步驟

要分割步驟必須符合下列條件：

- 必須分割輸入來源。如需詳細資訊，請參閱[事件中樞程式設計指南](../event-hubs/event-hubs-programming-guide.md)。
- 查詢的 **SELECT** 陳述式必須讀取某個已分割的輸入來源。
- 步驟內的查詢必須有 **Partition By** 關鍵字

分割查詢時將會處理輸入事件並將其彙總在個別的資料分割群組中，並且會為每個群組產生輸出事件。如果需要結合彙總，您必須建立第二個非資料分割的步驟以進行彙總。

### 計算工作的最大串流處理單元

所有非資料分割的步驟可以整合，而將串流分析工作的串流處理單元調整為最多 6 個。若要新增更多串流處理單元，則必須分割步驟。每個資料分割可以有 6 個串流處理單元。

<table border="1">
<tr><th>工作的查詢</th><th>工作的最大串流處理單元</th></td>

<tr><td>
<ul>
<li>查詢包含一個步驟。</li>
<li>步驟未分割。</li>
</ul>
</td>
<td>6</td></tr>

<tr><td>
<ul>
<li>輸入資料流分割時會除以 3。</li>
<li>查詢包含一個步驟。</li>
<li>步驟進行分割。</li>
</ul>
</td>
<td>18</td></tr>

<tr><td>
<ul>
<li>查詢包含兩個步驟。</li>
<li>兩個步驟都不會分割。</li>
</ul>
</td>
<td>6</td></tr>



<tr><td>
<ul>
<li>資料流輸入分割時會除以 3。</li>
<li>查詢包含兩個步驟。輸入步驟會分割，而第二個步驟則否。</li>
<li>SELECT 陳述式會讀取分割的輸入。</li>
</ul>
</td>
<td>24 (18 個用於分割的步驟 + 6 個用於非分割的步驟)</td></tr>
</table>

### 調整的範例
下列查詢會以三個收費亭為準，計算在 3 分鐘的時間範圍內通過收費站的車流量。此查詢可以調整為最多 6 個串流處理單元。

	SELECT COUNT(*) AS Count, TollBoothId
	FROM Input1
	GROUP BY TumblingWindow(minute, 3), TollBoothId, PartitionId

若要讓查詢使用更多串流處理單元，必須同時分割資料流輸入和查詢。假設資料流資料分割設為 3，則下列修改的查詢最多可以調整為 18 個串流處理單元：

	SELECT COUNT(*) AS Count, TollBoothId
	FROM Input1 Partition By PartitionId
	GROUP BY TumblingWindow(minute, 3), TollBoothId, PartitionId

分割查詢時將會處理輸入事件並將其彙總在個別的資料分割群組中。也會為每個群組產生輸出事件。當 [群組依據]欄位不是資料流輸入中的資料分割索引鍵時，資料分割可能會導致某些非預期的結果。舉例來說，前一個範例查詢中的 [TollBoothId] 欄位，並不是 Input1 資料分割索引鍵。收費亭 #1 中的資料可能分散在多個資料分割中。

每個 Input1 資料分割都將由串流分析個別處理，並且會建立相同的收費亭在相同輪動時間範圍內之車流量計數的多筆記錄。如果輸入資料分割索引鍵無法變更，此問題可藉由新增額外的非資料分割步驟來解決，例如：

	WITH Step1 AS (
		SELECT COUNT(*) AS Count, TollBoothId
		FROM Input1 Partition By PartitionId
		GROUP BY TumblingWindow(minute, 3), TollBoothId, PartitionId
	)

	SELECT SUM(Count) AS Count, TollBoothId
	FROM Step1
	GROUP BY TumblingWindow(minute, 3), TollBoothId

此查詢可以調整為 24 個串流處理單元。

>[AZURE.NOTE] 如果您要聯結兩個串流，請確定串流是由執行聯結之資料行的分割索引鍵進行分割的，且兩個串流中有相同數目的資料分割。


## 設定資料流分析工作資料分割

**若要調整工作的串流處理單元**

1. 登入[管理入口網站](https://manage.windowsazure.com)。
2. 按一下左窗格中的 [串流分析]。
3. 按一下您要調整的資料流分析工作。
4. 按一下頁面頂端的 [調整]。

![Azure 串流分析的串流處理單元調整][img.stream.analytics.streaming.units.scale]

在 Azure 入口網站的 [設定] 下可以調整設定：

![Azure 入口網站的串流分析工作組態][img.stream.analytics.preview.portal.settings.scale]

## 監視工作效能

使用管理入口網站可讓您追蹤工作的輸送量 (事件數/秒)：

![Azure 串流分析監視工作][img.stream.analytics.monitor.job]

計算工作負載的預期輸送量 (事件數/秒)。如果輸送量少於預期，請微調輸入資料分割、微調查詢，並為工作新增更多串流處理單元。

## 大規模串流分析輸送量：Raspberry Pi 案例


為了解串流分析工作在跨多個串流處理單位處理輸送量的典型案例中會如何調整，以下實驗會將感應器資料 (用戶端) 傳送到事件中樞、處理資料，然後把警示或統計資料當成輸出傳送到另一個事件中樞。

用戶端正在將綜合感應器資料以 JSON 格式傳送到串流分析的事件中樞，而資料輸出的格式也是 JSON。範例資料看起來如下–

    {"devicetime":"2014-12-11T02:24:56.8850110Z","hmdt":42.7,"temp":72.6,"prss":98187.75,"lght":0.38,"dspl":"R-PI Olivier's Office"}

查詢：「關燈時傳送警示」

    SELECT AVG(lght),
	 “LightOff” as AlertText
	FROM input TIMESTAMP
	BY devicetime
	 WHERE
		lght< 0.05 GROUP BY TumblingWindow(second, 1)

測量輸送量：此內容中的輸送量是串流分析在固定時間 (10 分鐘) 內處理的輸入資料量。若要讓輸入資料的輸送量處理達到最佳，必須同時分割資料流輸入和查詢。同時在查詢中包含 **COUNT()** 來測量已處理的輸入事件數目。為確保工作不會單純只等待輸入事件到來，輸入事件中樞的每個資料分割都已預先載入足夠的輸入資料 (大約 300MB)。

以下是事件中樞中數目持續增加的串流處理單位與對應的資料分割計數。

<table border="1">
<tr><th>輸入資料分割</th><th>輸出資料分割</th><th>串流處理單位</th><th>持續的輸送量
</th></td>

<tr><td>12</td>
<td>12</td>
<td>6</td>
<td>4.06 MB/秒</td>
</tr>

<tr><td>12</td>
<td>12</td>
<td>12</td>
<td>8.06 MB/秒</td>
</tr>

<tr><td>48</td>
<td>48</td>
<td>48</td>
<td>38.32 MB/秒</td>
</tr>

<tr><td>192</td>
<td>192</td>
<td>192</td>
<td>172.67 MB/秒</td>
</tr>

<tr><td>480</td>
<td>480</td>
<td>480</td>
<td>454.27 MB/秒</td>
</tr>

<tr><td>720</td>
<td>720</td>
<td>720</td>
<td>609.69 MB/秒</td>
</tr>
</table>

![img.stream.analytics.perfgraph][img.stream.analytics.perfgraph]

## 取得說明
如需進一步的協助，請參閱我們的 [Azure Stream Analytics 論壇](https://social.msdn.microsoft.com/Forums/zh-TW/home?forum=AzureStreamAnalytics)。


## 後續步驟

- [Azure Stream Analytics 介紹](stream-analytics-introduction.md)
- [開始使用 Azure Stream Analytics](stream-analytics-get-started.md)
- [Azure Stream Analytics 查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)
- [Azure Stream Analytics 管理 REST API 參考](https://msdn.microsoft.com/library/azure/dn835031.aspx)


<!--Image references-->

[img.stream.analytics.monitor.job]: ./media/stream-analytics-scale-jobs/StreamAnalytics.job.monitor.png
[img.stream.analytics.configure.scale]: ./media/stream-analytics-scale-jobs/StreamAnalytics.configure.scale.png
[img.stream.analytics.perfgraph]: ./media/stream-analytics-scale-jobs/perf.png
[img.stream.analytics.streaming.units.scale]: ./media/stream-analytics-scale-jobs/StreamAnalyticsStreamingUnitsExample.jpg
[img.stream.analytics.preview.portal.settings.scale]: ./media/stream-analytics-scale-jobs/StreamAnalyticsPreviewPortalJobSettings.png

<!--Link references-->

[microsoft.support]: http://support.microsoft.com
[azure.management.portal]: http://manage.windowsazure.com
[azure.event.hubs.developer.guide]: http://msdn.microsoft.com/library/azure/dn789972.aspx

[stream.analytics.introduction]: stream-analytics-introduction.md
[stream.analytics.get.started]: stream-analytics-get-started.md
[stream.analytics.query.language.reference]: http://go.microsoft.com/fwlink/?LinkID=513299
[stream.analytics.rest.api.reference]: http://go.microsoft.com/fwlink/?LinkId=517301
 

<!---HONumber=AcomDC_0921_2016-->