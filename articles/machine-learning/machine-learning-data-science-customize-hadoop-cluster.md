<properties 
	pageTitle="自訂適用於 Team Data Science Process 的 Hadoop 叢集 | Microsoft Azure" 
	description="自訂 Azure HDInsight Hadoop 叢集中提供熱門 Python 模組。"
	services="machine-learning" 
	documentationCenter="" 
	authors="bradsev" 
	manager="jhubbard" 
	editor="cgronlun"  />

<tags 
	ms.service="machine-learning" 
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="09/19/2016" 
	ms.author="hangzh;bradsev" />

# 自訂適用於 Team Data Science Process 的 Azure HDInsight Hadoop 叢集 

本文將說明若將 HDInsight Hadoop 叢集佈建為 HDInsight 服務，如何藉由在每個節點上安裝 64 位元的 Anaconda (Python 2.7) 來自訂該叢集。它也會示範如何存取前端節點，以將自訂工作提交至叢集。這項自訂讓許多熱門的 Python 模組 (隨附於 Anaconda) 非常方便地在使用者定義函式 (UDF) 中使用，這類函式是設計來處理叢集中的 Hive 記錄。如需此案例中使用的程序的相關指示，請參閱[如何提交 Hive 查詢](machine-learning-data-science-move-hive-tables.md#submit)。

以下功能表所連結的主題會說明如何設定 [Team Data Science Process (TDSP)](data-science-process-overview.md) 所用的各種資料科學環境。

[AZURE.INCLUDE [data-science-environment-setup](../../includes/cap-setup-environments.md)]


## <a name="customize"></a>自訂 Azure HDInsight Hadoop 叢集

若要建立自訂 HDInsight Hadoop 叢集，使用者需要登入 [**Azure 傳統入口網站**](https://manage.windowsazure.com/)，按一下左下角的 [新增]，然後選取 [資料服務] -> [HDINSIGHT] -> [自訂建立]，以顯示 [叢集詳細資料] 視窗。

![建立工作區](./media/machine-learning-data-science-customize-hadoop-cluster/customize-cluster-img1.png)

在設定頁面 1 上輸入要建立的叢集名稱，並接受其他欄位的預設值。按一下箭號，前往下一個設定頁面。

![建立工作區](./media/machine-learning-data-science-customize-hadoop-cluster/customize-cluster-img1.png)

在組態頁面 2 上，輸入 [資料節點] 的數目，選取 [區域/虛擬網路]，然後選取 [前端節點] 和 [資料節點] 的大小。按一下箭號，以前往下一個設定頁面。

>[AZURE.NOTE] [區域/虛擬網路] 必須與即將用於 HDInsight Hadoop 叢集的儲存體帳戶區域相同。否則，在第四個組態頁面中，使用者想要使用的儲存體帳戶將出現在 [帳戶名稱] 下拉式清單中。

![建立工作區](./media/machine-learning-data-science-customize-hadoop-cluster/customize-cluster-img3.png)

在設定頁面 3 上，提供適用於 HDInsight Hadoop 叢集的使用者名稱和密碼。**請勿**選取 [進入 Hive/Oozie 中繼存放區]。然後按一下箭號，前往下一個設定頁面。

![建立工作區](./media/machine-learning-data-science-customize-hadoop-cluster/customize-cluster-img4.png)

在設定頁面 4 上，指定儲存體帳戶名稱，以及 HDInsight Hadoop 叢集的預設容器。如果使用者在 [預設容器] 下拉式清單中選取 [建立預設容器]，就會建立名稱與叢集相同的容器。按一下箭號，前往最後一個設定頁面。

![建立工作區](./media/machine-learning-data-science-customize-hadoop-cluster/customize-cluster-img5.png)

在最後的 [指令碼動作] 設定頁面中，按一下 [加入指令碼動作] 按鈕，然後使用下列值填入文字欄位。
 
* **名稱** - 可做為這個指令碼動作名稱的任何字串。
* **節點類型** - 選取 [所有節點]。
* **指令碼 URI** - *http://getgoing.blob.core.windows.net/publicscripts/Azure_HDI_Setup_Windows.ps1*
	* *publicscripts* 是儲存體帳戶中的公用容器
	* *getgoing* 可用來共用 PowerShell 指令碼檔案，以協助使用者在 Azure 中工作。
* **PARAMETERS** - (保留空白)

最後，按一下勾號，開始建立自訂的 HDInsight Hadoop 叢集。

![建立工作區](./media/machine-learning-data-science-customize-hadoop-cluster/script-actions.png)

## <a name="headnode"></a> 存取 Hadoop 叢集的前端節點

使用者必須先在 Azure 中啟用 Hadoop 叢集的遠端存取，才能透過 RDP 存取 Hadoop 叢集的前端節點。

1. 登入 [**Azure 傳統入口網站**](https://manage.windowsazure.com/)，選取左側的 [HDInsight]，從叢集清單中選取您的 Hadoop 叢集，按一下 [設定] 索引標籤，然後按一下頁面底部的 [啟用遠端] 圖示。
	
	![建立工作區](./media/machine-learning-data-science-customize-hadoop-cluster/enable-remote-access-1.png)

2. 在 [設定遠端桌面] 視窗中，輸入 [使用者名稱] 和 [密碼] 欄位，然後選取遠端存取的到期日。接著，按一下勾號，啟用對 Hadoop 叢集前端節點的遠端存取。

	![建立工作區](./media/machine-learning-data-science-customize-hadoop-cluster/enable-remote-access-2.png)
	
>[AZURE.NOTE] 適用於遠端存取的使用者名稱和密碼並非您在建立 Hadoop 叢集時所使用的使用者名稱和密碼。它們是一組個別的認證。此外，遠端存取的到期日必須在從目前日期起算的 7 天內。

啟用遠端存取之後，按一下頁面底部的 [連接]，遠端連至前端節點。您登入 Hadoop 叢集前端節點的方式是針對稍早指定的遠端存取使用者輸入認證。

![建立工作區](./media/machine-learning-data-science-customize-hadoop-cluster/enable-remote-access-3.png)

[Team Data Science Process (TDSP)](https://azure.microsoft.com/documentation/learning-paths/cortana-analytics-process/) 中說明了進階分析程序的後續步驟，其中可能包含將資料移至 HDInsight 並在其中處理資料與取樣，做為透過 Azure Machine Learning 從資料學習的準備。

請參閱[如何提交 Hive 查詢](machine-learning-data-science-move-hive-tables.md#submit)中的相關指示，以了解如何在用來處理儲存於叢集的 Hive 記錄的使用者定義函數 (UDF) 中，從叢集的前端節點存取隨附於 Anaconda 的 Python 模組。

 

<!---HONumber=AcomDC_0921_2016-->