<properties 
	pageTitle="了解串流分析工作監視 | Microsoft Azure" 
	description="了解串流分析工作監視" 
	keywords="查詢監視"
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
	ms.date="09/19/2016" 
	ms.author="jeffstok"/>

# 了解串流分析工作監視功能，以及如何監視查詢

## 簡介：監視頁面

Azure 管理入口網站和 Azure 入口網站都會顯示關鍵效能計量，讓您能用來監視和疑難排解您的查詢及工作效能。

在 Azure 管理入口網站中，按一下執行中串流分析工作的 [監視] 索引標籤以查看這些度量。[監視] 頁面中出現了最多 1 分鐘的效能度量延遲。

  ![監視工作儀表板](./media/stream-analytics-monitoring/01-stream-analytics-monitoring.png)

在 Azure 入口網站中，瀏覽至您有興趣查看度量的串流分析工作，並檢視 [**監視**] 區段。

  ![Azure 入口網站監視工作儀表板](./media/stream-analytics-monitoring/06-stream-analytics-monitoring.png)

串流分析工作第一次在區域中建立時，您必須設定適合該地區的診斷功能。若要這樣做，請按一下 [監視] 區段的任何位置，就會顯示 [診斷] 刀鋒視窗。您可以在這裡啟用診斷，並指定監視資料的儲存體帳戶。

  ![Azure 入口網站設定查詢診斷](./media/stream-analytics-monitoring/07-stream-analytics-monitoring.png)

## 可供串流分析使用的度量


| 度量 | 定義 |
|--------|-------------|
| SU % 使用率 | 從工作的 [調整] 索引標籤指派給工作的串流處理單元使用率。若此指標達到 80% 以上，則代表事件處理作業極有可能延遲或暫停。 |
| 輸入事件 | 「串流分析」工作所接收到的資料量 (以事件數為單位)。這可以用來驗證傳送到輸入來源的事件。 |
| 輸出事件 | 「串流分析」工作所傳送的資料量 (以事件數為單位)。 |
| 順序錯亂事件 | 所收到順序錯亂的事件數目，這些事件會根據事件順序原則，予以捨棄或指定調整後的時間戳記。順序錯亂容錯視窗設定的組態可能會造成影響。 |
| 資料轉換錯誤 | 串流分析工作所造成的錯誤訊息數目。 |
| 執行階段錯誤 | 在執行串流分析工作期間所發生的錯誤數目。 |
| 延遲輸入事件 | 從來源延遲抵達的事件數目，這些事件已根據延遲抵達容錯視窗設定的事件順序原則組態卸除或調整其時間戳記。 |

## 在 Azure 管理入口網站中自訂監視 ##

圖表上最多可以顯示 6 個度量。

若要切換顯示相對值 (各個度量的最終值) 和絕對值 (顯示的 Y 軸)，請選取圖表頂端的 [相對] 或 [絕對]。

  ![查詢監視相對絕對](./media/stream-analytics-monitoring/02-stream-analytics-monitoring.png)

您可以在監視器圖表中以 1 小時、12 小時、24 小時或 7 天的彙總檢視度量。

若要變更度量圖表顯示的時間範圍，請選取圖表頂端的 [1 hour]、[24 小時] 或 [7 days]。

  ![查詢監視時間範圍](./media/stream-analytics-monitoring/03-stream-analytics-monitoring.png)

您可以設定規則，在工作超過定義臨界值時透過電子郵件通知您。

## 在 Azure 入口網站中自訂監視 ##

您可以在 [編輯圖表] 設定中調整圖表類型、顯示的度量和時間範圍。如需詳細資訊，請參閱[如何自訂監視](../azure-portal/insights-how-to-customize-monitoring.md)。

  ![Azure 入口網站查詢監視時間範圍](./media/stream-analytics-monitoring/08-stream-analytics-monitoring.png)

## 工作狀態

在 Azure 傳統入口網站中可檢視串流分析工作的狀態，在此您會看見工作清單。按一下 Azure 傳統入口網站中的 [串流分析] 圖示可檢視工作清單。

| 狀態 | 定義 |
|--------|------------|
| 建立時間 | 工作已建立，但是尚未啟動。 |
| 啟動中 | 使用者按一下啟動工作，且工作正在啟動 |
| 執行中 | 已配置工作，正在處理輸入，或正在等候處理輸入。如果作業顯示執行中狀態，卻沒有產生輸出，可能是資料處理時間範圍很大或查詢邏輯很複雜。另一個原因可能是目前沒有任何資料傳送至工作。 |
| 停止中 | 使用者按一下停止工作，且工作正在停止。 |
| 已停止 | 工作已停止。 |
| 已降級 | 此狀態表示串流分析工作發生暫時性錯誤 (例如，輸入/輸出錯誤、處理錯誤、轉換錯誤等)。工作仍在執行，不過已產生許多錯誤。這項工作需要客戶注意，且客戶可以看到錯誤的作業記錄檔。 |
| Failed | 這表示工作已因錯誤而失敗，而且已停止處理。客戶必須查看作業記錄檔以偵錯錯誤。 |
| 刪除中 | 這表示正在刪除工作。 |

## 診斷

在 Azure 管理入口網站中，工作儀表板提供您要在其中尋找診斷的資訊，也就是輸入、輸出和/或作業記錄檔。您可以按一下連結移至適當位置來查看診斷。

  ![查詢監視錯誤](./media/stream-analytics-monitoring/04-stream-analytics-monitoring.png)

按一下輸入或輸出資源可提供詳細的診斷資訊。這樣會在工作執行時重新整理為最新的診斷資訊。

  ![查詢診斷](./media/stream-analytics-monitoring/05-stream-analytics-monitoring.png)

## 取得說明
如需進一步的協助，請參閱我們的 [Azure Stream Analytics 論壇](https://social.msdn.microsoft.com/Forums/zh-TW/home?forum=AzureStreamAnalytics)

## 後續步驟

- [Azure Stream Analytics 介紹](stream-analytics-introduction.md)
- [開始使用 Azure Stream Analytics](stream-analytics-get-started.md)
- [調整 Azure Stream Analytics 工作](stream-analytics-scale-jobs.md)
- [Azure Stream Analytics 查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)
- [Azure 串流分析管理 REST API 參考](https://msdn.microsoft.com/library/azure/dn835031.aspx)

<!---HONumber=AcomDC_0921_2016-->