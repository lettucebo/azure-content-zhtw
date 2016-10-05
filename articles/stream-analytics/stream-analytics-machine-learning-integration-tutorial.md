<properties
	pageTitle="使用 Azure 串流分析和 Azure Machine Learning 的情緒分析 | Microsoft Azure"
	description="如何在串流分析工作中使用使用者定義的函數和機器學習服務"
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
	ms.date="08/08/2016" 
	ms.author="jeffstok"
/>

# 使用 Azure 串流分析和 Azure Machine Learning 的情緒分析 #

本文旨在協助您透過 Azure Machine Learning 整合，快速設定簡單的 Azure 串流分析工作。我們將使用 Cortana 智慧資源庫的情緒分析機器學習服務模型，來分析串流文字資料並即時判斷情緒分數。本文的資訊可協助了解下列各種案例，例如對串流 Twitter 資料進行即時情緒分析、由支援人員分析客戶聊天記錄、評估論壇、部落格、影片上的評論，以及其他許多即時預測評分案例。

本文提供含文字的範例 CSV 檔案做為 Azure Blob 儲存體的輸入，如下圖所示。工作會將情緒分析模型做為使用者定義的函數 (UDF) 套用到 Blob 存放區中的範例文字資料。最終結果則會放置在相同 Blob 存放區中的另一個 CSV 檔中。

![串流分析機器學習服務](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-integration-tutorial-figure-2.png)

下圖示範這項設定。如需更真實的案例，可將 Blob 儲存體更換為 Azure 事件中樞輸入內的串流 Twitter 資料。此外，也可針對彙總情緒建置 [Microsoft Power BI](https://powerbi.microsoft.com/) 即時視覺效果。

![串流分析機器學習服務](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-integration-tutorial-figure-1.png)

## 必要條件

完成本文示範工作的必要條件如下︰

-	有效的 Azure 訂用帳戶。
-	內附資料的 CSV 檔。您可以從 [GitHub](https://github.com/jeffstokes72/azure-stream-analytics-repository/blob/master/sampleinputs.csv) 下載檔案 (如圖 1 所示)，或者建立您自己的檔案。在本文中，我們假設您使用從 GitHub 下載的檔案。

總的來說，若要完成本文示範工作，您將執行下列作業︰

1.	將 CSV 輸入檔上傳至 Azure Blob 儲存體。
2.	將 Cortana 智慧資源庫的情緒分析模型加入 Azure Machine Learning 工作區。
3.	在機器學習服務工作區中將此模型部署為 Web 服務。
4.	建立串流分析工作，以函數形式呼叫此 Web 服務來判斷所輸入文字的情緒。
5.	啟動串流分析工作並觀察輸出。

## 將 CSV 輸入檔上傳至 Blob 儲存體

在此步驟中，您可以使用任何 CSV 檔案，例如 GitHub 上已指定為可供下載的檔案。您可以使用 [Azure 儲存體總管](http://storageexplorer.com/)或 Visual Studio 上傳檔案，或使用自訂程式碼。我們使用根據 Visual Studio 的範例。

1.	在 Visual Studio 中，按一下 [Azure] > [儲存體] > [附加外部儲存體]。輸入 [帳戶名稱] 和 [帳戶金鑰]。

    ![串流分析機器學習服務, 伺服器總管](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-integration-tutorial-server-explorer.png)

2.	展開在步驟 1 附加的儲存體，按一下 [建立 Blob 容器]，然後輸入邏輯名稱。建立容器之後，開啟容器以檢視其內容(此時是空的)。

    ![串流分析機器學習服務, 建立 blob](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-integration-tutorial-create-blob.png)

3.	若要上傳 CSV 檔案，請按一下 [上傳 Blob]，然後按一下本機磁碟中的檔案。

## 新增 Cortana 智慧資源庫中的情緒分析模型

1.	下載 Cortana 智慧資源庫中的[預測情緒分析模型](https://gallery.cortanaintelligence.com/Experiment/Predictive-Mini-Twitter-sentiment-analysis-Experiment-1)。
2.	在 Machine Learning Studio 中，按一下 [在 Studio 中開啟]。

    ![串流分析機器學習服務, 開啟 Machine Learning Studio](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-integration-tutorial-open-ml-studio.png)

3.	登入以移至工作區。選取最適合您所在地的位置。
4.	按一下頁面底部的 [執行]。
5.	程序成功執行之後，按一下 [部署 Web 服務]。
6.	情緒分析模型現在已可供使用。若要驗證，請按一下 [測試] 按鈕並輸入文字，例如 "I love Microsoft"。 測試應該會傳回類似下列的結果：

`'Predictive Mini Twitter sentiment analysis Experiment' test returned ["4","0.715057671070099"]...`

![串流分析機器學習服務, 分析資料](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-integration-tutorial-analysis-data.png)

在 [應用程式] 欄中，按一下 [Excel 2010 或較舊版本活頁簿] 連結，以取得稍後要設定串流分析工作的 API 金鑰和 URL。(如果要使用另一個 Azure 帳戶工作區中的機器學習服務模型，才必須執行此步驟。本文假設這正是我們為了解決該案例而必須執行的動作)。

記下所下載 Excel 檔中的 Web 服務 URL 和存取金鑰，如下所示：

![串流分析機器學習服務, 快速概覽](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-integration-tutorial-quick-glance.png)

## 建立使用機器學習服務模型的串流分析工作

1.	移至 [Azure 入口網站](https://manage.windowsazure.com)。
2.	按一下 [新增] > [資料服務] > [串流分析] > [快速建立]。在 [工作名稱] 中輸入您的工作名稱，在 [區域] 中輸入適當的工作區域，然後在 [區域監視儲存體帳戶] 中選取帳戶。
3.	建立工作之後，在 [輸入] 索引標籤上按一下 [加入輸入]。

    ![串流分析機器學習服務, 加入機器學習服務輸入](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-integration-tutorial-add-input-screen.png)

4.	在 [加入輸入] 精靈的第一頁按一下 [資料串流]，然後按 [下一步]。在下一個頁面按一下 [Blob 儲存體] 做為輸入，然後按 [下一步]。
5.	在精靈的 [Blob 儲存體設定] 頁面上，提供您先前上傳資料時所定義的儲存體帳戶 Blob 容器名稱。按 [下一步]。對於 [事件序列化格式] 按一下 [CSV]。接受 [序列化設定] 頁面其餘的預設值。按一下 [確定]。
6.	在 [輸出] 索引標籤上，按一下 [加入輸出]。

    ![串流分析機器學習服務, 加入輸出](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-integration-tutorial-add-output-screen.png)

7.	按一下 [Blob 儲存體]，然後輸入相同的參數 (除了容器)。我們先前已設定為要從 **CSV** 檔所上傳到的 "test" 容器中讀取**輸入**值。對於 [輸出]，請輸入 “testoutput”。容器名稱必須不同。確認此容器存在。
8.	按 [下一步] 以設定輸出的 [序列化設定]。和在 [輸入] 時一樣，按一下 [CSV] 並按一下 [確定] 按鈕。
9.	在 [函數] 索引標籤上，按一下 [新增機器學習服務函數]。

    ![串流分析機器學習服務, 加入機器學習服務函數](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-integration-tutorial-add-ml-function.png)

10.	在 [機器學習 Web 服務設定] 頁面上，找到機器學習服務工作區、Web 服務和預設端點。在本文中，如果您知道 URL 而且擁有 API 金鑰，請手動套用設定，以便熟悉工作區 Web 服務的設定方式。輸入端點的 [URL] 和 [API 金鑰]。按一下 [確定]。

    ![串流分析機器學習服務, Machine Learning Web 服務](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-integration-tutorial-ml-web-service.png)

11.	在 [查詢] 索引標籤上，如下修改查詢︰

 ```
 	WITH subquery AS (  
 		SELECT text, sentiment(text) as result from input  
  	)  
 
 	Select text, result.[Scored Labels]  
 	Into output  
 	From subquery  
 ```    
12.	按一下 [儲存] 以儲存查詢。

## 啟動串流分析工作並觀察輸出

1.	按一下工作頁面底部的 [啟動]。
2.	在 [開始查詢] 對話方塊上，按一下 [自訂時間]，然後選取您將 CSV 上傳至 Blob 儲存體之前的時間。按一下 [確定]。

    ![串流分析機器學習服務, 自訂時間](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-integration-tutorial-custom-time.png)

3.	使用您用來上傳 CSV 檔案的工具 (例如 Visual Studio)，移至 Blob 儲存體。
4.	在工作啟動後幾分鐘，就會建立輸出容器，並將 CSV 檔案上傳到當中。
5.	在預設 CSV 編輯器中開啟檔案。應該會顯示類似下面的內容：

    ![串流分析機器學習服務, CSV 檢視](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-integration-tutorial-csv-view.png)

## 結論

本文示範如何建立串流分析工作讀取串流處理的文字資料，並即時對資料套用情緒分析。您不必擔心需要建立麻煩的情緒分析模型，就能完成全部的工作。這是 Cortana 智慧套件的其中一項優點。

您也能檢視與 Azure Machine Learning 函數相關的度量。若要進行，請按一下 [監視器] 索引標籤。此時會顯示三個與函數相關的度量。

- **函數要求**指出傳送至 Machine Learning Web 服務的要求數目。
- **函數事件**指出要求中的事件數目。根據預設，每個對 Machine Learning Web 服務的要求最多可包含 1,000 個事件。
- **失敗的函數要求**指出對 Machine Learning Web 服務的失敗要求數目。

    ![串流分析機器學習服務, Machine Learning 監視器檢視](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-integration-tutorial-ml-monitor-view.png)

<!---HONumber=AcomDC_0921_2016-->