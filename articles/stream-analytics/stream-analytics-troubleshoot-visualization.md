<properties
	pageTitle="以視覺化方式呈現串流分析工作並進行疑難排解 | Microsoft Azure"
	description="了解如何使用診斷圖表功能以視覺化方式呈現「串流分析」工作管線，來進行自助疑難排解。"
	keywords=""
	documentationCenter=""
	services="stream-analytics"
	authors="jeffstokes72"
	manager="jhubbard"
	editor="cgronlun"
/>

<tags
	ms.service="stream-analytics"
	ms.devlang="na"
	ms.topic="article"
	ms.tgt_pltfrm="na"
	ms.workload="data-services"
	ms.date="07/27/2016"
	ms.author="jeffstok"
/>


# 以視覺化方式呈現串流分析工作並進行疑難排解

在「串流分析」中，與其他雲端型技術一樣，有時需要進行疑難排解來仔細研究為何某個工作沒有產生預期的輸出 (或就此而言，產生任何輸出)。考量到這一點，「串流分析」提供了以視覺化方式呈現串流工作的功能。此功能可作為便利的模型工具，並且對於需要為其工作提供佐證文件的使用者而言，也有附帶的好處。

在視覺化面板中，可以看到輸入項以及所執行的查詢，還有所有已設定的輸出項。連線或設定問題會變得更清楚，此外，以視覺方式呈現您的組態也可能會有幫助。

## 使用診斷圖表工具

若要存取這個視覺化檢視，只要按一下「串流分析」工作之 [設定] 刀鋒視窗中的 [診斷圖表] 按鈕即可。

![stream-analytics-troubleshoot-visualization-diagnosis-diagram](./media/stream-analytics-troubleshoot-visualization/stream-analytics-troubleshoot-visualization-diagnosis-diagram1.png)

每個輸入項和輸出項都會以色彩標示，以指出該元件目前的狀態，如以下所示。

![stream-analytics-troubleshoot-visualization-input-map](./media/stream-analytics-troubleshoot-visualization/stream-analytics-troubleshoot-visualization-input-map.png)

當使用者想要查看中繼查詢步驟以了解工作內的資料流模式時，視覺化工具可提供將查詢細分成組成步驟及流程順序的檢視。按一下每個查詢步驟，即會在查詢編輯窗格中顯示對應的區段，如下圖所示。

![stream-analytics-troubleshoot-visualization-intermediate-steps](./media/stream-analytics-troubleshoot-visualization/stream-analytics-troubleshoot-visualization-intermediate-steps.png)




## 後續步驟

- [Azure Stream Analytics 介紹](stream-analytics-introduction.md)
- [開始使用 Azure Stream Analytics](stream-analytics-get-started.md)
- [調整 Azure Stream Analytics 工作](stream-analytics-scale-jobs.md)
- [Azure Stream Analytics 查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)
- [Azure Stream Analytics 管理 REST API 參考](https://msdn.microsoft.com/library/azure/dn835031.aspx)

<!---HONumber=AcomDC_0921_2016-->