<properties 
	pageTitle="使用串流分析中的作業和服務記錄檔偵錯 | Microsoft Azure" 
	description="如何使用串流分析作業記錄檔" 
	keywords="服務記錄檔"
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

# 使用服務和作業記錄檔對串流分析工作進行偵錯

所有 Azure 服務都會將作業記錄訊息提供給使用者，以記錄與管理作業有關的詳細資料。在 Azure 串流分析中，這項資訊可用於偵錯用途，例如檢視工作狀態、工作進度與失敗訊息，以從一開始處理到輸出都可隨著時間追蹤作業進度。

## 在 Azure 管理入口網站中尋找作業記錄檔

作業記錄檔可用兩種方式來存取：

- 串流分析工作的儀表板
- Azure 傳統入口網站中的管理服務

## 串流分析工作的儀表板

串流分析工作相對應的記錄檔連結會顯示在該工作的 [儀表板] 索引標籤上。按一下該連結時，其會以能夠顯示該特定工作最新記錄檔的方式來設定篩選器。

  ![選取管理服務記錄檔](./media/stream-analytics-operation-logs/01-stream-analytics-operation-logs.png)

## 管理服務

若要以手動方式瀏覽至串流分析的作業記錄檔以及 Azure 傳統入口網站中的其他服務：

1.	按一下 [[Azure 傳統入口網站](https://manage.windowsazure.com)] 中的 [**管理服務**]。
2.	針對 [類型] 選取 [串流分析]，並針對 [服務名稱] 選取作業的名稱。

  ![選取串流分析](./media/stream-analytics-operation-logs/02-stream-analytics-operation-logs.png)

## 在 Azure 入口網站中尋找稽核記錄檔 ##

若要在 Azure 入口網站中尋找串流分析工作的作業記錄檔，請按一下 [**瀏覽**]，然後選取 [**稽核記錄檔**]。

  ![Azure 入口網站選取串流分析](./media/stream-analytics-operation-logs/06-stream-analytics-operation-logs.png)

這會開啟刀鋒視窗，當中會顯示您訂用帳戶中所有資源在過去 7 天的事件。按一下 [篩選] 命令，可以篩選查看指定類型或時間範圍的事件。

  ![Azure 入口網站選取串流分析](./media/stream-analytics-operation-logs/07-stream-analytics-operation-logs.png)

## 取得記錄檔詳細資料

您可以依照時間範圍和狀態來篩選，以檢視工作的記錄檔。

在 Azure 管理入口網站中，按一下視窗底部的 [詳細資料] 按鈕來檢視所選事件的詳細資訊。

  ![選取詳細資料](./media/stream-analytics-operation-logs/03-stream-analytics-operation-logs.png)

在 Azure 入口網站中，按一下記錄項目即可查看其中的詳細事件。

  ![Azure 入口網站選取詳細資料](./media/stream-analytics-operation-logs/08-stream-analytics-operation-logs.png)

從該處，按一下事件即可開啟 [詳細資料] 刀鋒視窗。

  ![Azure 入口網站選取詳細資料](./media/stream-analytics-operation-logs/09-stream-analytics-operation-logs.png)

## 偵錯失敗的工作

在 Azure 管理入口網站中，按一下搜尋圖示並鍵入 ‘failed’。這樣可以找出所有包含失敗項目的記錄檔。

  ![針對失敗的工作進行偵錯](./media/stream-analytics-operation-logs/04-stream-analytics-operation-logs.png)

在 Azure 入口網站中，您可以按訊息層級篩選，以檢視**嚴重**事件。

  ![Azure 入口網站偵錯](./media/stream-analytics-operation-logs/10-stream-analytics-operation-logs.png)

您可以選取任何一個失敗項目，並按一下 [詳細資料] 來取得有關此錯誤的詳細資訊。某些錯誤訊息也會提供降低此問題之風險的方法。

  ![Operation Details](./media/stream-analytics-operation-logs/05-stream-analytics-operation-logs.png)

萬一您需要連絡[支援服務](https://azure.microsoft.com/support/options/)或透過 [MSDN 論壇](https://social.msdn.microsoft.com/Forums/zh-TW/home?forum=AzureStreamAnalytics)將資訊提供給團隊，請注意作業詳細資料，特別是**相互關聯識別碼**。

## 取得說明
如需進一步的協助，請參閱我們的 [Azure Stream Analytics 論壇](https://social.msdn.microsoft.com/Forums/zh-TW/home?forum=AzureStreamAnalytics)

## 後續步驟

- [Azure Stream Analytics 介紹](stream-analytics-introduction.md)
- [開始使用 Azure Stream Analytics](stream-analytics-get-started.md)
- [調整 Azure Stream Analytics 工作](stream-analytics-scale-jobs.md)
- [Azure Stream Analytics 查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)
- [Azure 串流分析管理 REST API 參考](https://msdn.microsoft.com/library/azure/dn835031.aspx)

<!---HONumber=AcomDC_0921_2016-->