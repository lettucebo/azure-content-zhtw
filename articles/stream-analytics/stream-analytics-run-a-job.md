<properties 
	pageTitle="如何在串流分析中啟動串流工作 | Microsoft Azure" 
	description="如何在 Azure 串流分析中執行串流工作 | 學習路徑區段。"
    keywords="串流工作"
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

# 如何在 Azure 串流分析中執行串流工作

當已指定工作輸入、查詢及輸出時，即可開始串流分析工作。

開始您的工作：

1.	在 Azure 傳統入口網站的工作儀表板，按一下頁面底部的 [開始]。

    ![開始工作按鈕](./media/stream-analytics-run-a-job/1-stream-analytics-run-a-job.png)

    在 Azure 入口網站中，按一下工作頁面頂端的 [開始]。

    ![Azure 入口網站的開始工作按鈕](./media/stream-analytics-run-a-job/4-stream-analytics-run-a-job.png)

2.	指定 [開始輸出] 值來決定這項工作開始產生輸出的時間。先前尚未開始之工作的預設設定為 [工作開始時間]，這代表該工作會立即開始處理資料。您也可以指定在過去 (適用於取用歷史資料) 或未來 (用來延遲處理到未來某個時間) 的 [自訂] 時間。對於先前已經開始並停止的工作，您可選取 [上次停止時間]，以便從上次輸出的時間點繼續工作，並避免資料遺失。

    ![啟動串流工作的時間](./media/stream-analytics-run-a-job/2-stream-analytics-run-a-job.png)

    ![Azure 入口網站開始串流工作的時間](./media/stream-analytics-run-a-job/5-stream-analytics-run-a-job.png)

3.	確認選擇。工作狀態會變更為 [啟動中]，並會在工作開始後的短時間內變為 [執行中]。您可以在 [通知中樞] 中監視「開始」作業的進度：

    ![串流工作進度](./media/stream-analytics-run-a-job/3-stream-analytics-run-a-job.png)

    ![Azure 入口網站串流工作進度](./media/stream-analytics-run-a-job/6-stream-analytics-run-a-job.png)

## 取得說明
如需進一步的協助，請參閱我們的 [Azure Stream Analytics 論壇](https://social.msdn.microsoft.com/Forums/zh-TW/home?forum=AzureStreamAnalytics)

## 後續步驟

- [Azure Stream Analytics 介紹](stream-analytics-introduction.md)
- [開始使用 Azure Stream Analytics](stream-analytics-get-started.md)
- [調整 Azure Stream Analytics 工作](stream-analytics-scale-jobs.md)
- [Azure Stream Analytics 查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)
- [Azure 串流分析管理 REST API 參考](https://msdn.microsoft.com/library/azure/dn835031.aspx)

<!---HONumber=AcomDC_0921_2016-->