<properties 
	pageTitle="如何在串流分析中編寫查詢 | Microsoft Azure" 
	description="在串流分析中編寫查詢及查詢資料 | 學習路徑區段。"
	keywords="如何編寫查詢, 查詢資料, 編寫查詢, 編寫查詢"
	documentationCenter=""
	services="stream-analytics"
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

# 如何在串流分析中撰寫查詢

在 Azure 串流分析中編寫串流處理邏輯的查詢會實作為「常設查詢」，它在工作開始前就已經獲得定義，且會在到達工作時針對資料來執行。資料轉換會以類似 SQL 的查詢語言來表示，大部分是 T-SQL 子集並加入某些語言擴充功能 (例如 [Windowing](https://msdn.microsoft.com/library/azure/dn835019.aspx)) 來表示時間語意。

## 編寫查詢： ##

1. 在 Azure 管理入口網站的串流分析工作中，按一下 [查詢]。

    ![選取查詢](./media/stream-analytics-write-queries/1-stream-analytics-write-queries.png)

    在 Azure 入口網站，按一下 [查詢]。

    ![選取查詢預覽](./media/stream-analytics-write-queries/query-preview-portal.png)

2.	新工作有可以協助您入門的查詢範本。查詢範本會執行「傳遞」查詢，其會將所有欄位從輸入事件投影至輸出中。

    - 如果您已經為工作定義了至少一個輸入和輸出，便可以將預留位置 "[YourOutputAlias]" 和 "[YourInputAlias]" 欄位取代為您想先用的輸入與輸出別名。此外，您仍可在 Azure 傳統入口網站中撰寫並測試您的查詢，而不需在工作中定義輸入及輸出。
    - 如果您想要執行比簡單的傳遞更多的處理，您可以編輯查詢定義。若要開始撰寫查詢，請到[這裡](stream-analytics-stream-analytics-query-patterns.md)看看一些擷取的常見查詢模式。
  
    ![查詢資料視窗](./media/stream-analytics-write-queries/2-stream-analytics-write-queries.png)

## 如何驗證查詢資料在運作中： ##

您可以透過在瀏覽器中執行一或多個包含測試資料的本機 JSON 檔案，來測試查詢是否如預期執行。這不會開始工作，也不會以任何方式計費。

> [AZURE.NOTE] Azure 入口網站目前不支援瀏覽器內查詢測試。

1.	請確定查詢中沒有任何錯誤 (否則 [測試] 按鈕將會停用) 然後按一下 [測試] 按鈕。

    ![查詢資料測試](./media/stream-analytics-write-queries/3-stream-analytics-write-queries.png)

2.	將會提示您將檔案指定給在查詢中被參考的每個輸入。在此範例中，範本查詢將維持現狀，所以對話方塊提示會提示輸入名為 "yourinputalias" 的輸入。

    ![輪詢資料查詢](./media/stream-analytics-write-queries/4-stream-analytics-write-queries.png)

3.	瀏覽至測試檔案。[github](https://github.com/Azure/azure-stream-analytics/tree/master/Sample 資料) 上有幾個範例檔案，而您也可以透過輸入索引標籤上的範例資料函式，從您自己的資料流輸入來擷取範例資料。

    ![查詢輸入](./media/stream-analytics-write-queries/5-stream-analytics-write-queries.png)

4.	關閉對話方塊之後，將會針對測試資料執行查詢，而您可在 [查詢] 頁面底部查看結果。

    ![查詢摘要](./media/stream-analytics-write-queries/6-stream-analytics-write-queries.png)

## 取得說明
如需進一步的協助，請參閱我們的 [Azure Stream Analytics 論壇](https://social.msdn.microsoft.com/Forums/zh-TW/home?forum=AzureStreamAnalytics)

## 後續步驟

- [Azure Stream Analytics 介紹](stream-analytics-introduction.md)
- [開始使用 Azure Stream Analytics](stream-analytics-get-started.md)
- [調整 Azure Stream Analytics 工作](stream-analytics-scale-jobs.md)
- [Azure Stream Analytics 查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)
- [Azure 串流分析管理 REST API 參考](https://msdn.microsoft.com/library/azure/dn835031.aspx)

<!---HONumber=AcomDC_0921_2016-->