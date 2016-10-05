<properties
	pageTitle="串流分析上的 Power BI 儀表板 | Microsoft Azure"
	description="使用即時串流 Power BI 儀表板來收集商業智慧及分析來自串流分析工作的大量資料。"
	keywords="分析儀表板, 即時儀表板"
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

#  串流分析及 Power BI：適用於串流資料的即時分析儀表板

Azure 串流分析可讓您使用其中一種業界頂尖智慧型工具 Microsoft Power BI。了解如何使用 Azure 串流分析來分析大量串流資料，以及取得即時 Power BI 分析儀表板中的深度資訊。

使用 [Microsoft Power BI](https://powerbi.com/) 來快速建置即時儀表板。[觀賞案例說明影片](https://www.youtube.com/watch?v=SGUpT-a99MA)。

在這篇文章中，您將了解如何使用 Power BI 做為 Azure 串流分析工作的輸出，來建立您的自訂商業智慧型工具，以及如何利用即時儀表板。

## 必要條件

* Microsoft Azure 帳戶
* 串流分析工作從中使用串流資料的輸入。串流分析可接收來自 Azure 事件中樞或 Azure Blob 儲存體的輸入。
* Power BI 的公司帳戶或學校帳戶

## 建立 Azure 串流分析工作

在 [Azure 傳統入口網站](https://manage.windowsazure.com) 中，依序按一下 [新增]、[資料服務]、[串流分析]、[快速建立]。

指定下列值，然後按一下 [建立 Stream Analytics 工作]：

* **工作名稱** - 輸入工作名稱。例如 **DeviceTemperatures**。
* **區域** - 選取要執行此工作的區域。請考慮將工作和事件中樞放在相同的區域以確保更好的效能，以及在區域之間傳輸資料時無須付費。
* **儲存體帳戶** - 選擇您為在此區域內執行的所有 Stream Analytics 工作儲存監視資料時所要使用的儲存體帳戶。您可以選擇現有的儲存體帳戶，或建立新帳戶。

按一下左窗格中的 [Stream Analytics]，以列出 Stream Analytics 工作。

![graphic1][graphic1]

> [AZURE.TIP] 新工作會以 [未啟動] 的狀態列出。請注意，頁面底部的 [啟動] 按鈕會停用。這是正常行為，因為您必須先設定工作輸入、輸出、查詢等項目，才能啟動工作。

## 指定工作輸入

在本教學課程中，我們假設您搭配 JSON 序列化與 UTF-8 編碼使用事件中樞做為輸入。

* 按一下工作名稱。
* 按一下頁面頂端的 [輸入]，然後按一下 [新增輸入]。開啟的對話方塊會逐步引導您完成設定輸入。
*	選取 [資料流]，然後按一下右鍵。
*	選取 [事件中樞]，然後按一下右鍵。
*	在第三頁上輸入或選取下列值：
  *	**輸入別名** - 輸入此工作輸入的易記名稱。請注意，此名稱將用於後續的查詢。
  * **事件中樞** - 如果您建立的「事件中樞」與「串流分析」工作位於相同的訂用帳戶，請選取事件中樞所在的命名空間。
*	如果事件中樞位於不同的訂閱中，請選取 [使用其他訂閱中的事件中樞]，並手動輸入 [服務匯流排命名空間]、[事件中樞名稱]、[事件中樞原則名稱]、[事件中樞原則索引鍵] 和 [事件中樞資料分割計數] 的資訊。

> [AZURE.NOTE]	此範例會使用預設的 16 個資料分割數目。

* **事件中樞名稱** - 選取 Azure 事件中樞的名稱。
* **事件中樞原則名稱** - 為目前使用中的事件中樞選取事件中樞原則。請確定此原則具有管理權限。
*	**事件中樞用戶群組** – 您可以保留空白，或指定事件中樞上的用戶群組。請注意，事件中樞的每個用戶群組一次只能有 5 個讀取器。因此，請為您的工作選擇正確的用戶群組。如果您將欄位保留空白，它會使用預設的用戶群組。

*	按一下向右按鈕。
*	指定下列值：
  *	**事件序列化程式格式** - JSON
  *	**編碼** - UTF8
*	按一下核取按鈕以新增此來源，並確認資料流分析可成功連接到事件中心。

## 新增 Power BI 輸出

1.  按一下頁面頂端的 [輸出]，然後按一下 [新增輸出]。您會看到 Power BI 被列為輸出選項。

    ![graphic2][graphic2]

2.  選取 [Power BI]，然後按一下右鍵。
3.  您會看到類似下列畫面：

    ![graphic3][graphic3]

4.  在此步驟中，提供串流分析工作輸出的公司帳戶或學校帳戶。如果您已具有 Power BI 帳戶，請選取 [**立即授權**]。如果沒有，請選擇 [**立即註冊**]。[下面的部落格文章為您逐步解說如何註冊 Power BI](http://blogs.technet.com/b/powerbisupport/archive/2015/02/06/power-bi-sign-up-walkthrough.aspx)。

    ![graphic11][graphic11]

5.  接下來您會看到一個畫面，如下所示：

    ![graphic4][graphic4]

提供值，如下所示：

* **輸出別名** – 您可以填寫任何容易記住的輸出別名。如果您決定讓工作擁有多個輸出，填寫輸出別名會非常有幫助。在這個情況下，您必須參考此查詢中的輸出。比方說，我們要使用的輸出別名值 = "OutPbi"。
* **資料集名稱** - 提供一個 Power BI 輸出應該要有的資料集名稱。例如，"pbidemo"。
*	**資料表名稱** - 提供 Power BI 輸出資料集的資料表名稱。暫時稱之為 "pbidemo"。目前，串流分析工作的 Power BI 輸出中，一個資料集只能有一個資料表。
*	**工作區** – 在將用來建立資料集的 Power BI 租用戶中選取工作區。

>	[AZURE.NOTE] 您不應該在 Power BI 帳戶中明確地建立資料集和資料表，這些資料集和資料表會在您啟動串流分析工作時自動建立，且串流分析工作會將輸出提取至 Power BI。如果您的工作佇列並未傳回任何結果，則不會建立資料集和資料表。

*	依序按一下 [確定]、[測試連線]，輸出組態就已經完成。

>	[AZURE.WARNING] 也請注意 Power BI 是否已經具有與您在串流分析工作中提供的名稱相同皂資料集和資料表名稱；若是如此，可能會覆寫現有的資料。


## 撰寫查詢

移至工作的 [查詢] 索引標籤。撰寫查詢，就是 Power BI 中要有的輸出。例如，可能是類似下列 SQL 查詢的內容：

    SELECT
    	MAX(hmdt) AS hmdt,
    	MAX(temp) AS temp,
    	System.TimeStamp AS time,
    	dspl
    INTO
        OutPBI
    FROM
    	Input TIMESTAMP BY time
    GROUP BY
    	TUMBLINGWINDOW(ss,1),
    	dspl



開始您的工作。驗證事件中樞是否正在接收事件，而且您的查詢會產生預期的結果。如果您的查詢輸出 0 個資料列，系統就不會自動建立 Power BI 資料集和資料表。

## 在 Power BI 中建立儀表板

移至 [Powerbi.com](https://powerbi.com)，然後使用公司帳戶或學校帳戶登入。如果串流分析工作查詢輸出任何結果，就會看到您的資料集已經建立完成：

![graphic5][graphic5]

如果想建立儀表板，請移至儀表板選項，然後建立新的儀表板。

![graphic6][graphic6]

在這個範例中，我們將它標籤為「示範儀表板 」。

現在按一下串流分析工作建立的資料集 (就是目前範例中的 pbidemo)。系統會開啟一個頁面並在這個資料集上方建立一張圖表。以下是您可以建立的報表範例：

選取 Σ 溫度和時間欄位。這兩個欄位會自動顯示在圖表中的 [值] 和 [軸]：

![graphic7][graphic7]

這樣一來，一張圖表就大功告成了，如下所示：

![graphic8][graphic8]

在 [值] 區段中，按一下溫度的下拉式清單，然後針對溫度選擇 [平均值]，接下來在圖表上，按一下 [視覺效果] 再選擇 [折線圖]：

![graphic9][graphic9]

您現在會看到一張折線圖，裡面列出各個時間的平均值。使用釘選選項，如下所示。您可以釘選先前建立的儀表板：

![graphic10][graphic10]

現在當您檢視儀表板以及這個釘選的報表時，會看到報表的數據不斷即時更新。試著變更事件中的資料 – 最高溫度或類似資料，您會看到儀表板即時反映最新的數據。

請注意，本教學課程示範如何為一個資料集建立一種圖表類型。Power BI 可協助您為組織建立其他客戶商業智慧型工具。如需其他 Power BI 儀表板範例，請觀賞[開始使用 Power BI](https://youtu.be/L-Z_6P56aas?t=1m58s) 視訊。

如需有關設定 Power BI 輸出及利用 Power BI 群組的進一步資訊，請檢閱[了解串流分析輸出](stream-analytics-define-outputs.md "了解串流分析輸出")的 [Power BI 小節](stream-analytics-define-outputs.md#power-bi)。另一個深入了解如何使用 Power BI 建立儀表板的實用資源，可參考 [Power BI 中的儀表板](https://powerbi.microsoft.com/documentation/powerbi-service-dashboards/)。

## 限制和最佳作法

Power BI 同時採用了並行處理和輸送量條件約束，如下所述：[https://powerbi.microsoft.com/pricing](https://powerbi.microsoft.com/pricing "Power BI 價格")

由於這些 Power BI 本身以最自然的方式符合案例需求，其中 Azure 串流分析會大量降低資料載入的作業。
我們建議使用 TumblingWindow 或 HoppingWindow 來確保資料推送最多為每秒推送 1 次，並且您的查詢會符合輸送量的要求範圍 – 您可以使用下列方程式，以秒為單位計算要提供給視窗的值：

![equation1](./media/stream-analytics-power-bi-dashboard/equation1.png)

舉例來說，如果您有 1,000 個每秒傳送資料的裝置，並且在支援每小時 1,000,000 個資料列的 Power BI Pro SKU 上，而您想要取得 Power BI 上每個裝置的平均資料，則您可以讓每一裝置最多每 4 秒推送一次 (如下所示)：

![equation2](./media/stream-analytics-power-bi-dashboard/equation2.png)

這表示我們會將原始查詢變更為：

    SELECT
    	MAX(hmdt) AS hmdt,
    	MAX(temp) AS temp,
    	System.TimeStamp AS time,
    	dspl
    INTO
    	OutPBI
    FROM
    	Input TIMESTAMP BY time
    GROUP BY
    	TUMBLINGWINDOW(ss,4),
    	dspl

### PowerBI 檢視重新整理

常見的問題是「為什麼 PowerBI 中的儀表板不會自動更新？」。

為達此目的，在 PowerBI 中利用問與答提出像「時間戳記是今天的溫度最大值」的問題，並將該磚釘選至儀表板。

### 更新授權

如果您在建立工作之後或上次驗證過後變更了密碼，則需要重新驗證您的 Power BI 帳戶。如果您在 Azure Active Directory (AAD) 租用戶上設定 Multi-Factor Authentication (MFA)，則也需要每 2 週更新一次 Power BI 授權。此問題發生時的徵兆就是沒有工作輸出，且作業記錄檔中出現「驗證使用者錯誤」：

![graphic12][graphic12]

同樣地，如果作業在權杖已過期時嘗試啟動，將會發生錯誤，而作業啟動將會失敗。錯誤看起來如下所示︰

![PowerBI 驗證錯誤](./media/stream-analytics-power-bi-dashboard/stream-analytics-power-bi-dashboard-token-expire.png)
 

若要解決這個問題，請停止執行工作並移至 Power BI 輸出。按一下 [更新授權] 連結，並從 [上次停止時間] 重新啟動您的工作以避免資料遺失。

![PowerBI 驗證更新](./media/stream-analytics-power-bi-dashboard/stream-analytics-power-bi-dashboard-token-renew.png)

透過 Power BI 重新整理授權後，您會看見授權區域中出現綠色警示︰

![PowerBI 驗證更新](./media/stream-analytics-power-bi-dashboard/stream-analytics-power-bi-dashboard-token-renewed.png)

## 取得說明
如需進一步的協助，請參閱我們的 [Azure Stream Analytics 論壇](https://social.msdn.microsoft.com/Forums/zh-TW/home?forum=AzureStreamAnalytics)

## 後續步驟

- [Azure Stream Analytics 介紹](stream-analytics-introduction.md)
- [開始使用 Azure Stream Analytics](stream-analytics-get-started.md)
- [調整 Azure Stream Analytics 工作](stream-analytics-scale-jobs.md)
- [Azure Stream Analytics 查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)
- [Azure 串流分析管理 REST API 參考](https://msdn.microsoft.com/library/azure/dn835031.aspx)


[graphic1]: ./media/stream-analytics-power-bi-dashboard/1-stream-analytics-power-bi-dashboard.png
[graphic2]: ./media/stream-analytics-power-bi-dashboard/2-stream-analytics-power-bi-dashboard.png
[graphic3]: ./media/stream-analytics-power-bi-dashboard/3-stream-analytics-power-bi-dashboard.png
[graphic4]: ./media/stream-analytics-power-bi-dashboard/4-stream-analytics-power-bi-dashboard.png
[graphic5]: ./media/stream-analytics-power-bi-dashboard/5-stream-analytics-power-bi-dashboard.png
[graphic6]: ./media/stream-analytics-power-bi-dashboard/6-stream-analytics-power-bi-dashboard.png
[graphic7]: ./media/stream-analytics-power-bi-dashboard/7-stream-analytics-power-bi-dashboard.png
[graphic8]: ./media/stream-analytics-power-bi-dashboard/8-stream-analytics-power-bi-dashboard.png
[graphic9]: ./media/stream-analytics-power-bi-dashboard/9-stream-analytics-power-bi-dashboard.png
[graphic10]: ./media/stream-analytics-power-bi-dashboard/10-stream-analytics-power-bi-dashboard.png
[graphic11]: ./media/stream-analytics-power-bi-dashboard/11-stream-analytics-power-bi-dashboard.png
[graphic12]: ./media/stream-analytics-power-bi-dashboard/12-stream-analytics-power-bi-dashboard.png
[graphic13]: ./media/stream-analytics-power-bi-dashboard/13-stream-analytics-power-bi-dashboard.png

<!---HONumber=AcomDC_0921_2016-->