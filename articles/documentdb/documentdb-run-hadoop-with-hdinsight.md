<properties
	pageTitle="使用 DocumentDB 和 HDInsight 執行 Hadoop 工作 | Microsoft Azure"
	description="了解如何使用 DocumentDB 和 Azure HDInsight 來執行簡單的 Hive、Pig 和 MapReduce 工作。"
	services="documentdb"
	authors="AndrewHoh"
	manager="jhubbard"
	editor="mimig"
	documentationCenter=""/>


<tags
	ms.service="documentdb"
	ms.workload="data-services"
	ms.tgt_pltfrm="na"
	ms.devlang="java"
	ms.topic="article"
	ms.date="09/20/2016"
	ms.author="anhoh"/>

#<a name="DocumentDB-HDInsight"></a>使用 DocumentDB 和 HDInsight 執行 Hadoop 工作

本教學課程會示範如何在 Azure HDInsight 上，使用 DocumentDB 的 Hadoop 連接器來執行 [Apache Hive][apache-hive]、[Apache Pig][apache-pig] 和 [Apache Hadoop][apache-hadoop] MapReduce 工作。DocumentDB 的 Hadoop 連接器可讓 DocumentDB 同時做為 Hive、Pig 和 MapReduce 工作的來源與接收。本教學課程將使用 DocumentDB 同時作為 Hadoop 工作的資料來源和目的地。

完成本教學課程後，您將能夠回答下列問題：

- 如何使用 Hive、Pig 或 MapReduce 工作從 DocumentDB 載入資料？
- 如何使用 Hive、Pig 或 MapReduce 工作將資料儲存在 DocumentDB 中？

我們建議使用者從觀看下列影片開始入門，在影片中我們會使用 DocumentDB 和 HDInsight 執行 Hive 工作。

> [AZURE.VIDEO use-azure-documentdb-hadoop-connector-with-azure-hdinsight]

接著回到本文，您將在本文中找到有關如何在 DocumentDB 資料上執行分析工作的完整詳細資訊。

> [AZURE.TIP] 本教學課程假設您先前已有使用 Apache Hadoop、Hive 和/或 Pig 的經驗。如果您不熟悉 Apache Hadoop、Hive 和 Pig，建議您造訪 [Apache Hadoop 文件][apache-hadoop-doc]。本教學課程也假設您先前已有使用 DocumentDB 的經驗，且擁有 DocumentDB 帳戶。如果您不熟悉 DocumentDB 或沒有 DocumentDB 帳戶，請造訪我們的[開始使用][getting-started]頁面。

是否沒有時間完成本教學課程，而只想取得 Hive、Pig 和 MapReduce 的完整範例 PowerShell 指令碼？ 沒問題，您可以在[這裡][documentdb-hdinsight-samples]取得。下載項目也包含這些範例的 hql、pig 和 java 檔案。

## <a name="NewestVersion"></a>最新版本

<table border='1'>
	<tr><th>Hadoop 連接器版本</th>
		<td>1.2.0</td></tr>
	<tr><th>指令碼 URI</th>
		<td>https://portalcontent.blob.core.windows.net/scriptaction/documentdb-hadoop-installer-v04.ps1</td></tr>
	<tr><th>修改日期</th>
		<td>2016 年 4 月 26 日</td></tr>
	<tr><th>支援的 HDInsight 版本</th>
		<td>3.1、3.2</td></tr>
	<tr><th>變更記錄檔</th>
		<td>將 DocumentDB Java SDK 更新為 1.6.0</br>
			將已分割的集合支援同時加入為來源和接收</br>
		</td></tr>
</table>

## <a name="Prerequisites"></a>必要條件
在依照本教學課程中的指示進行之前，請先確定您已備妥下列項目：

- DocumentDB 帳戶、資料庫和包含文件的集合。如需詳細資訊，請參閱[開始使用 DocumentDB][getting-started]。使用 [DocumentDB 匯入工具][documentdb-import-data]將範例資料匯入至您的 DocumentDB 帳戶。
- 輸送量。從 HDInsight 讀取和寫入將會計入您的集合的配置要求單位。如需詳細資訊，請參閱[佈建輸送量、要求單位和資料庫作業][documentdb-manage-throughput]。
- 每個輸出集合額內額外預存程序的容量。預存程序可用來傳送產生的文件。如需詳細資訊，請參閱[集合及佈建的輸送量][documentdb-manage-document-storage]。
- 由 Hive、Pig 或 MapReduce 工作產生的文件容量。如需詳細資訊，請參閱[管理 DocumentDB 容量和效能][documentdb-manage-collections]。
- [*選擇性*] 額外集合的容量。如需詳細資訊，請參閱[佈建的文件儲存體和索引額外負荷][documentdb-manage-document-storage]。

> [AZURE.WARNING] 為避免在任何工作期間建立新的集合，您可以將結果列印至 stdout、將輸出儲存到您的 WASB 容器，或指定現有的集合。如果是指定現有的集合，則會在集合內部建立新的文件，且只有當 *ids* 發生衝突時，現有的文件才會受到影響。**連接器會自動覆寫發生識別碼衝突的現有文件**。將更新插入選項設定為 false 便可關閉這項功能。如果更新插入為 false，但發生衝突，則 Hadoop 工作將會失敗。回報識別碼衝突錯誤。


## <a name="ProvisionHDInsight"></a>步驟 1︰建立新的 HDInsight 叢集
本教學課程會使用 Azure 入口網站的指令碼動作，來自訂您的 HDInsight 叢集。在本教學課程中，我們將使用 Azure 入口網站來建立 HDInsight 叢集。如需有關如何使用 PowerShell 指令程式或 HDInsight .NET SDK 的指示，請參閱[使用指令碼動作來自訂 HDInsight 叢集][hdinsight-custom-provision]一文。

1. 登入 [Azure 入口網站][azure-portal]。

2. 按一下左側導覽頂端的 [+ 新增]，然後在 [新增] 刀鋒視窗的頂端搜尋列中搜尋 **HDInsight**。

3. **Microsoft** 所發佈的 **HDInsight** 就會出現在 [結果] 頂端。對它按一下，然後按一下 [建立]。

4. 在 [新的 HDInsight 叢集建立] 刀鋒視窗中，輸入您的**叢集名稱**，然後選取您要用來佈建此資源的 [訂用帳戶]。

	<table border='1'>
		<tr><td>叢集名稱</td><td>為叢集命名。<br/>
		DNS 名稱的開頭與結尾都必須是英數字元，且可包含連字號。<br/>
		欄位必須是介於 3 到 63 個字元的字串。</td></tr>
		<tr><td>訂用帳戶名稱</td>
			<td>如果您有多個 Azure 訂用帳戶，請選取會裝載您的 HDInsight 叢集的訂用帳戶。</td></tr>
	</table>

5. 按一下 [選取叢集類型]，並將下列屬性設定為指定的值。

	<table border='1'>
		<tr><td>叢集類型</td><td><strong>Hadoop</strong></td></tr>
		<tr><td>叢集層</td><td><strong>標準</strong></td></tr>
		<tr><td>作業系統</td><td><strong>Windows</strong></td></tr>
		<tr><td>版本</td><td>最新版本</td></tr>
	</table>

	現在，按一下 [選取]。

	![提供 Hadoop HDInsight 初始叢集詳細資料][image-customprovision-page1]

6. 按一下 [認證] 以設定您的登入與遠端存取認證。選擇 [叢集登入使用者名稱] 和 [叢集登入密碼]。

	如果您想要遠端登入您的叢集，請選取刀鋒視窗底部的 [是]，並提供使用者名稱和密碼。

7. 按一下 [資料來源] 來設定用於存取資料的主要位置。選擇 [選取方法]，並指定現有儲存體帳戶或建立一個新帳戶。

8. 在相同的刀鋒視窗中，指定 [預設容器] 和 [位置]。然後，按一下 [選取]。

	> [AZURE.NOTE] 選取靠近您的 DocumentDB 帳戶區域的位置以提升效能

8. 按一下 [價格] 以選取節點的數量和類型。您可以保留預設組態，並在稍後調整背景工作節點的數量。

9. 按一下 [選擇性組態]，然後選取 [選擇性組態] 刀鋒視窗中的 [指令碼動作]。

	在 [指令碼動作] 中輸入下列用來自訂 HDInsight 叢集的資訊。

	<table border='1'>
		<tr><th>屬性</th><th>值</th></tr>
		<tr><td>名稱</td>
			<td>指定指令碼動作的名稱。</td></tr>
		<tr><td>指令碼 URI</td>
			<td>對自訂叢集所叫用的指令碼指定 URI。</br></br>
			請輸入： </br> <strong>https://portalcontent.blob.core.windows.net/scriptaction/documentdb-hadoop-installer-v04.ps1</strong>。</td></tr>
		<tr><td>前端</td>
			<td>按一下核取方塊以對前端節點執行 PowerShell 指令碼。</br></br>
			<strong>勾選此核取方塊</strong>。</td></tr>
		<tr><td>背景工作</td>
			<td>按一下核取方塊以對背景工作節點執行 PowerShell 指令碼。</br></br>
			<strong>勾選此核取方塊</strong>。</td></tr>
		<tr><td>Zookeeper</td>
			<td>按一下核取方塊以對 Zookeeper 執行 PowerShell 指令碼。</br></br>
			<strong>不需要</strong>。
			</td></tr>
		<tr><td>參數</td>
			<td>如果指令碼要求，請指定參數。</br></br>
			<strong>不需要參數</strong>。</td></tr>
	</table>

10. 建立新的**資源群組**或使用 Azure 訂用帳戶下的現有資源群組。

11. 現在，勾選 [釘選到儀表板] 來追蹤其部署，然後按一下 [建立]！

## <a name="InstallCmdlets"></a>步驟 2：安裝並設定 Azure PowerShell

1. 安裝 Azure PowerShell。您可以在[這裡][powershell-install-configure]找到指示。

	> [AZURE.NOTE] 或者，您可以使用 HDInsight 的線上 Hive 編輯器 (僅限 Hive 查詢)。若要這樣做，請登入 [Azure 入口網站][azure-portal]，按一下左側窗格中的 **HDInsight** 以檢視您的 HDInsight 叢集清單。按一下您要執行 Hive 查詢的叢集，然後按一下 [**查詢主控台**]。

2. 開啟 Azure PowerShell 整合式指令碼環境：
	- 在執行 Windows 8 或 Windows Server 2012 或更新版本的電腦上，您可以使用內建搜尋。在 [開始] 畫面中，輸入 **powershell ise**，並按一下 **Enter** 鍵。
	- 在執行比 Windows 8 或 Windows Server 2012 更舊版本的電腦上，使用 [開始] 功能表。在 [開始] 功能表中，在搜尋方塊中輸入 [**命令提示字元**]，然後按一下結果清單中的 [**命令提示字元**]。在命令提示字元中，輸入 **powershell\_ise**，並按一下 **Enter** 鍵。

3. 新增您的 Azure 帳戶。
	1. 在主控台窗格中，輸入 **Add-AzureAccount**，並按一下 **Enter** 鍵。
	2. 輸入與您 Azure 訂用帳戶相關聯的電子郵件地址，並按一下 [繼續]。
	3. 輸入您 Azure 訂用帳戶的密碼。
	4. 按一下 [**登入**]。

4. 下圖將指出 Azure PowerShell 指令碼環境的重要部分。

	![Azure PowerShell 的圖表][azure-powershell-diagram]

## <a name="RunHive"></a>步驟 3：使用 DocumentDB 和 HDInsight 執行 Hive 作業

> [AZURE.IMPORTANT] 以 < > 表示的所有變數都必須使用組態設定進行填寫。

1. 在 [PowerShell 指令碼] 窗格中設定下列變數。

		# Provide Azure subscription name, the Azure Storage account and container that is used for the default HDInsight file system.
		$subscriptionName = "<SubscriptionName>"
		$storageAccountName = "<AzureStorageAccountName>"
		$containerName = "<AzureStorageContainerName>"

		# Provide the HDInsight cluster name where you want to run the Hive job.
		$clusterName = "<HDInsightClusterName>"

2. <p>首先我們要建構查詢字串。我們將撰寫執行下列動作的 Hive 查詢：接受所有文件從 DocumentDB 集合系統產生的時間戳記 (_ts) 和唯一識別碼 (_rid) ，並計算所有文件 (以分鐘為單位)，然後將結果存回新的 DocumentDB 集合。</p>

    <p>首先，我們要在 DocumentDB 集合中建立 Hive 資料表。將下列程式碼片段加入 [PowerShell 指令碼] 窗格中 # 1 的程式碼片段<strong>後面</strong>。請確定包含選擇性的 DocumentDB.query 參數，將文件整理成只有 _ts 和 _rid。</p>

    > [AZURE.NOTE] **命名 DocumentDB.inputCollections 是正確的選擇。** 沒錯，我們允許在一筆輸入中加入多個集合：</br>

		'*DocumentDB.inputCollections*' = '*<DocumentDB Input Collection Name 1>*,*<DocumentDB Input Collection Name 2>*' A1A</br> The collection names are separated without spaces, using only a single comma.


		# Create a Hive table using data from DocumentDB. Pass DocumentDB the query to filter transferred data to _rid and _ts.
		$queryStringPart1 = "drop table DocumentDB_timestamps; "  +
                            "create external table DocumentDB_timestamps(id string, ts BIGINT) "  +
                            "stored by 'com.microsoft.azure.documentdb.hive.DocumentDBStorageHandler' "  +
                            "tblproperties ( " +
                                "'DocumentDB.endpoint' = '<DocumentDB Endpoint>', " +
                                "'DocumentDB.key' = '<DocumentDB Primary Key>', " +
                                "'DocumentDB.db' = '<DocumentDB Database Name>', " +
                                "'DocumentDB.inputCollections' = '<DocumentDB Input Collection Name>', " +
                                "'DocumentDB.query' = 'SELECT r._rid AS id, r._ts AS ts FROM root r' ); "

3.  接下來，我們將建立輸出集合的 Hive 資料表。輸出文件屬性包括月、日、小時、分鐘和總發生次數。

	> [AZURE.NOTE] **再重申一次，命名 DocumentDB.outputCollections 是正確的選擇。** 沒錯，我們允許在一筆輸出中加入多個集合：</br> 
	'*DocumentDB.outputCollections*' = '*<DocumentDB Output Collection Name 1>*,*<DocumentDB Output Collection Name 2>*'</br> 不使用空格，只使用單一逗點分隔的集合名稱。</br></br>文件將會是跨多個集合的分散式循環配置資源。第一批文件會儲存在一個集合中，然後第二批文件會儲存在下一個集合中，以此類推。

		# Create a Hive table for the output data to DocumentDB.
	    $queryStringPart2 = "drop table DocumentDB_analytics; " +
                              "create external table DocumentDB_analytics(Month INT, Day INT, Hour INT, Minute INT, Total INT) " +
                              "stored by 'com.microsoft.azure.documentdb.hive.DocumentDBStorageHandler' " +
                              "tblproperties ( " +
                                  "'DocumentDB.endpoint' = '<DocumentDB Endpoint>', " +
                                  "'DocumentDB.key' = '<DocumentDB Primary Key>', " +  
                                  "'DocumentDB.db' = '<DocumentDB Database Name>', " +
                                  "'DocumentDB.outputCollections' = '<DocumentDB Output Collection Name>' ); "

4. 最後，我們可以依照月、日、小時和分鐘計算文件，並將結果插回輸出 Hive 資料表中。

		# GROUP BY minute, COUNT entries for each, INSERT INTO output Hive table.
        $queryStringPart3 = "INSERT INTO table DocumentDB_analytics " +
                              "SELECT month(from_unixtime(ts)) as Month, day(from_unixtime(ts)) as Day, " +
                              "hour(from_unixtime(ts)) as Hour, minute(from_unixtime(ts)) as Minute, " +
                              "COUNT(*) AS Total " +
                              "FROM DocumentDB_timestamps " +
                              "GROUP BY month(from_unixtime(ts)), day(from_unixtime(ts)), " +
                              "hour(from_unixtime(ts)) , minute(from_unixtime(ts)); "

5. 新增下列指令碼片段，從先前的查詢建立 Hive 工作定義。

		# Create a Hive job definition.
		$queryString = $queryStringPart1 + $queryStringPart2 + $queryStringPart3
		$hiveJobDefinition = New-AzureHDInsightHiveJobDefinition -Query $queryString

	您也可以使用 -File 參數來指定 HDFS 上的 HiveQL 指令碼檔案。

6. 新增下列指令碼片段來儲存開始時間，並提交 Hive 工作。

		# Save the start time and submit the job to the cluster.
		$startTime = Get-Date
		Select-AzureSubscription $subscriptionName
		$hiveJob = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $hiveJobDefinition

7. 新增下列指令碼來等待 Hive 工作完成：

		# Wait for the Hive job to complete.
		Wait-AzureHDInsightJob -Job $hiveJob -WaitTimeoutInSeconds 3600

8. 新增下列項目來列印標準輸出及開始和結束時間。

		# Print the standard error, the standard output of the Hive job, and the start and end time.
		$endTime = Get-Date
		Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $hiveJob.JobId -StandardOutput
		Write-Host "Start: " $startTime ", End: " $endTime -ForegroundColor Green

9. **執行**新的指令碼！ **按一下**綠色的執行按鈕。

10. 查看結果。登入 [Azure 入口網站][azure-portal]。
	1. 按一下左側面板上的 [<strong>瀏覽</strong>]。</br>
	2. 按一下 [瀏覽] 面板右上方的 [<strong>所有項目</strong>]。</br>
	3. 尋找並按一下 [<strong>DocumentDB 帳戶</strong>]。</br>
	4. 接下來，依序尋找您的 [<strong>DocumentDB 帳戶</strong>]、[<strong>DocumentDB 資料庫</strong>] 和與 Hive 查詢中指定輸出集合相關聯的 [<strong>DocumentDB 集合</strong>]。</br>
	5. 最後，按一下 [<strong>開發人員工具</strong>] 底下的 <strong>Document Explorer</strong>。</br></p>

	您將會看到 Hive 查詢的結果。

	![Hive 查詢結果][image-hive-query-results]

## <a name="RunPig"></a>步驟 4：使用 DocumentDB 和 HDInsight 執行 Pig 作業

> [AZURE.IMPORTANT] 以 < > 表示的所有變數都必須使用組態設定進行填寫。

1. 在 [PowerShell 指令碼] 窗格中設定下列變數。

        # Provide Azure subscription name.
        $subscriptionName = "Azure Subscription Name"

        # Provide HDInsight cluster name where you want to run the Pig job.
        $clusterName = "Azure HDInsight Cluster Name"

2. <p>首先我們要建構查詢字串。我們將撰寫執行下列動作的 Pig 查詢：接受所有文件從 DocumentDB 集合系統產生的時間戳記 (_ts) 和唯一識別碼 (_rid) ，並計算所有文件 (以分鐘為單位)，然後將結果存回新的 DocumentDB 集合。</p>
    <p>首先，將文件從 DocumentDB 載入 HDInsight。將下列程式碼片段加入 [PowerShell 指令碼] 窗格中 # 1 的程式碼片段<strong>後面</strong>。請務必將 DocumentDB 查詢加入選擇性的 DocumentDB 查詢參數，以將文件整理成只有 _ts 和 _rid。</p>

    > [AZURE.NOTE] 沒錯，我們允許在一筆輸入中加入多個集合：</br> 
    '*<DocumentDB Input Collection Name 1>*,*<DocumentDB Input Collection Name 2>*'</br>不使用空格，只使用單一逗點分隔的集合名稱。</b>

	文件將會是跨多個集合的分散式循環配置資源。第一批文件會儲存在一個集合中，然後第二批文件會儲存在下一個集合中，以此類推。

		# Load data from DocumentDB. Pass DocumentDB query to filter transferred data to _rid and _ts.
        $queryStringPart1 = "DocumentDB_timestamps = LOAD '<DocumentDB Endpoint>' USING com.microsoft.azure.documentdb.pig.DocumentDBLoader( " +
                                                        "'<DocumentDB Primary Key>', " +
                                                        "'<DocumentDB Database Name>', " +
                                                        "'<DocumentDB Input Collection Name>', " +
                                                        "'SELECT r._rid AS id, r._ts AS ts FROM root r' ); "

3.  接下來，我們將依照月、日、小時、分鐘和總發生次數來計算文件。

		# GROUP BY minute and COUNT entries for each.
        $queryStringPart2 = "timestamp_record = FOREACH DocumentDB_timestamps GENERATE `$0#'id' as id:int, ToDate((long)(`$0#'ts') * 1000) as timestamp:datetime; " +
                            "by_minute = GROUP timestamp_record BY (GetYear(timestamp), GetMonth(timestamp), GetDay(timestamp), GetHour(timestamp), GetMinute(timestamp)); " +
                            "by_minute_count = FOREACH by_minute GENERATE FLATTEN(group) as (Year:int, Month:int, Day:int, Hour:int, Minute:int), COUNT(timestamp_record) as Total:int; "

4. 最後，將結果存回新的輸出集合。

    > [AZURE.NOTE] 沒錯，我們允許在一筆輸出中加入多個集合：</br> 
    '\<DocumentDB Output Collection Name 1\>,\<DocumentDB Output Collection Name 2\>'</br>不使用空格，只使用單一逗點分隔的集合名稱。</br> 
    文件將會是跨多個集合的分散式循環配置資源。第一批文件會儲存在一個集合中，然後第二批文件會儲存在下一個集合中，以此類推。

		# Store output data to DocumentDB.
        $queryStringPart3 = "STORE by_minute_count INTO '<DocumentDB Endpoint>' " +
                            "USING com.microsoft.azure.documentdb.pig.DocumentDBStorage( " +
                                "'<DocumentDB Primary Key>', " +
                                "'<DocumentDB Database Name>', " +
                                "'<DocumentDB Output Collection Name>'); "

5. 新增下列指令碼片段，從先前的查詢建立 Pig 工作定義。

        # Create a Pig job definition.
        $queryString = $queryStringPart1 + $queryStringPart2 + $queryStringPart3
        $pigJobDefinition = New-AzureHDInsightPigJobDefinition -Query $queryString -StatusFolder $statusFolder

	您也可以使用 -File 參數，在 HDFS 上指定一個 Pig 指令碼檔案。

6. 新增下列指令碼片段來儲存開始時間，並提交 Pig 工作。

		# Save the start time and submit the job to the cluster.
		$startTime = Get-Date
		Select-AzureSubscription $subscriptionName
		$pigJob = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $pigJobDefinition

7. 新增下列指令碼來等待 Pig 工作完成：

		# Wait for the Pig job to complete.
		Wait-AzureHDInsightJob -Job $pigJob -WaitTimeoutInSeconds 3600

8. 新增下列項目來列印標準輸出及開始和結束時間。

		# Print the standard error, the standard output of the Hive job, and the start and end time.
		$endTime = Get-Date
		Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $pigJob.JobId -StandardOutput
		Write-Host "Start: " $startTime ", End: " $endTime -ForegroundColor Green

9. **執行**新的指令碼！ **按一下**綠色的執行按鈕。

10. 查看結果。登入 [Azure 入口網站][azure-portal]。
	1. 按一下左側面板上的 [<strong>瀏覽</strong>]。</br>
	2. 按一下 [瀏覽] 面板右上方的 [<strong>所有項目</strong>]。</br>
	3. 尋找並按一下 [<strong>DocumentDB 帳戶</strong>]。</br>
	4. 接下來，依序尋找您的 [<strong>DocumentDB 帳戶</strong>]、[<strong>DocumentDB 資料庫</strong>] 和與 Pig 查詢中指定輸出集合相關聯的 [<strong>DocumentDB 集合</strong>]。</br>
	5. 最後，按一下 [<strong>開發人員工具</strong>] 底下的 <strong>Document Explorer</strong>。</br></p>

	您將會看到 Pig 查詢的結果。

	![Pig 查詢結果][image-pig-query-results]

## <a name="RunMapReduce"></a>步驟 5：使用 DocumentDB 和 HDInsight 執行 MapReduce 作業

1. 在 [PowerShell 指令碼] 窗格中設定下列變數。

		$subscriptionName = "<SubscriptionName>"   # Azure subscription name
		$clusterName = "<ClusterName>"             # HDInsight cluster name

2. 我們將執行 MapReduce 工作，針對您 DocumentDB 集合中每一個文件屬性的發生次數進行計算。在上述程式碼片段**之後**加入此指令碼程式碼片段。

		# Define the MapReduce job.
		$TallyPropertiesJobDefinition = New-AzureHDInsightMapReduceJobDefinition -JarFile "wasb:///example/jars/TallyProperties-v01.jar" -ClassName "TallyProperties" -Arguments "<DocumentDB Endpoint>","<DocumentDB Primary Key>", "<DocumentDB Database Name>","<DocumentDB Input Collection Name>","<DocumentDB Output Collection Name>","<[Optional] DocumentDB Query>"

	> [AZURE.NOTE] TallyProperties-v01.jar 隨附於 DocumentDB Hadoop 連接器的自訂安裝。

3. 新增下列命令來提交 MapReduce 工作。

		# Save the start time and submit the job.
		$startTime = Get-Date
		Select-AzureSubscription $subscriptionName
		$TallyPropertiesJob = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $TallyPropertiesJobDefinition | Wait-AzureHDInsightJob -WaitTimeoutInSeconds 3600  

	除了 MapReduce 工作定義，您也提供想要用來執行 MapReduce 工作的 HDInsight 叢集名稱，以及認證。Start-AzureHDInsightJob 是非同步呼叫。若要檢查工作是否完成，請使用 *Wait-AzureHDInsightJob* Cmdlet。

4. 新增下列命令來檢查 MapReduce 工作執行時有無任何錯誤。

		# Get the job output and print the start and end time.
		$endTime = Get-Date
		Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $TallyPropertiesJob.JobId -StandardError
		Write-Host "Start: " $startTime ", End: " $endTime -ForegroundColor Green

5. **執行**新的指令碼！ **按一下**綠色的執行按鈕。

6. 查看結果。登入 [Azure 入口網站][azure-portal]。
	1. 按一下左側面板上的 [<strong>瀏覽</strong>]。
	2. 按一下 [瀏覽] 面板右上方的 [<strong>所有項目</strong>]。
	3. 尋找並按一下 [<strong>DocumentDB 帳戶</strong>]。
	4. 接下來，依序尋找您的 [<strong>DocumentDB 帳戶</strong>]、[<strong>DocumentDB 資料庫</strong>] 和與 MapReduce 工作中指定輸出集合相關聯的 [<strong>DocumentDB 集合</strong>]。
	5. 最後，按一下 [<strong>開發人員工具</strong>] 底下的 <strong>Document Explorer</strong>。

	您將會看到 MapReduce 工作的結果。

	![MapReduce 查詢結果][image-mapreduce-query-results]

## <a name="NextSteps"></a>後續步驟

恭喜！ 您剛剛使用 Azure DocumentDB 和 HDInsight 執行您的第一個 Hive、Pig 和 MapReduce 工作。

我們已開放 Hadoop 連接器的原始碼。如果您有興趣，您可以在 [GitHub][documentdb-github] 上發表意見。

若要深入了解，請參閱下列文章：

- [使用 Documentdb 開發 Java 應用程式][documentdb-java-application]
- [在 HDInsight 上開發 Hadoop 的 Java MapReduce 程式][hdinsight-develop-deploy-java-mapreduce]
- [開始在 HDInsight 中搭配 Hive 使用 Hadoop 以分析行動電話使用][hdinsight-get-started]
- [搭配 HDInsight 使用 MapReduce][hdinsight-use-mapreduce]
- [〈搭配 HDInsight 使用 Hivet〉][hdinsight-use-hive]
- [搭配 HDInsight 使用 Pig][hdinsight-use-pig]
- [使用指令碼動作來自訂 HDInsight 叢集][hdinsight-hadoop-customize-cluster]

[apache-hadoop]: http://hadoop.apache.org/
[apache-hadoop-doc]: http://hadoop.apache.org/docs/current/
[apache-hive]: http://hive.apache.org/
[apache-pig]: http://pig.apache.org/
[getting-started]: documentdb-get-started.md

[azure-portal]: https://portal.azure.com/
[azure-powershell-diagram]: ./media/documentdb-run-hadoop-with-hdinsight/azurepowershell-diagram-med.png

[documentdb-hdinsight-samples]: http://portalcontent.blob.core.windows.net/samples/documentdb-hdinsight-samples.zip
[documentdb-github]: https://github.com/Azure/azure-documentdb-hadoop
[documentdb-java-application]: documentdb-java-application.md
[documentdb-manage-collections]: documentdb-manage.md#Collections
[documentdb-manage-document-storage]: documentdb-manage.md#IndexOverhead
[documentdb-manage-throughput]: documentdb-manage.md#ProvThroughput
[documentdb-import-data]: documentdb-import-data.md

[hdinsight-custom-provision]: ../hdinsight/hdinsight-provision-clusters.md#powershell
[hdinsight-develop-deploy-java-mapreduce]: ../hdinsight/hdinsight-develop-deploy-java-mapreduce-linux.md
[hdinsight-hadoop-customize-cluster]: ../hdinsight/hdinsight-hadoop-customize-cluster.md
[hdinsight-get-started]: ../hdinsight/hdinsight-hadoop-tutorial-get-started-windows.md
[hdinsight-storage]: ../hdinsight/hdinsight-hadoop-use-blob-storage.md
[hdinsight-use-hive]: ../hdinsight/hdinsight-use-hive.md
[hdinsight-use-mapreduce]: ../hdinsight/hdinsight-use-mapreduce.md
[hdinsight-use-pig]: ../hdinsight/hdinsight-use-pig.md

[image-customprovision-page1]: ./media/documentdb-run-hadoop-with-hdinsight/customprovision-page1.png
[image-hive-query-results]: ./media/documentdb-run-hadoop-with-hdinsight/hivequeryresults.PNG
[image-mapreduce-query-results]: ./media/documentdb-run-hadoop-with-hdinsight/mapreducequeryresults.PNG
[image-pig-query-results]: ./media/documentdb-run-hadoop-with-hdinsight/pigqueryresults.PNG

[powershell-install-configure]: ../powershell-install-configure.md

<!---HONumber=AcomDC_0921_2016-->