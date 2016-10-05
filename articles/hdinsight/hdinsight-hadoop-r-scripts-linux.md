<properties
	pageTitle="在以 Linux 為基礎的 HDInsight 上安裝 R | Microsoft Azure"
	description="了解如何安裝和使用 R 來自訂以 Linux 為基礎的 Hadoop 叢集。"
	services="hdinsight"
	documentationCenter=""
	authors="Blackmist"
	manager="jhubbard"
	editor="cgronlun"/>

<tags
	ms.service="hdinsight"
	ms.workload="big-data"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/20/2016"
	ms.author="larryfr"/>

# 在 HDInsight Hadoop 叢集上安裝和使用 R

您可以使用**指令碼動作**叢集自訂，在 HDInsight 上 Hadoop 中的任何一種叢集上安裝 R。這可讓資料科學家和分析師使用 R 來部署強大的 MapReduce/YARN 程式設計架構，以處理部署在 HDInsight 中之 Hadoop 叢集上的大量資料。

> [AZURE.IMPORTANT] HDInsight 的[進階層](https://azure.microsoft.com/pricing/details/hdinsight/)供應項目包括隨附於 HDInsight 叢集的 R 伺服器。這可讓 R 指令碼使用 MapReduce 和 Spark 來執行分散式計算。如需詳細資訊，請參閱[開始使用 HDInsight 中的 R 伺服器](hdinsight-hadoop-r-server-get-started.md)。


## 什麼是 R？

<a href="http://www.r-project.org/" target="_blank">R Project for Statistical Computing</a> 是一個用於統計計算的開放原始碼語言和環境。R 提供數百個內建的統計函式及它自己的程式設計語言，此語言結合了函式型和物件導向程式設計的層面。它也提供廣泛的圖形功能。R 是廣泛各種不同領域中，大多數專業統計人員和科學家慣用的程式設計環境。

R 指令碼可以在 HDInsight 中的 Hadoop 叢集上執行，這些叢集是在建立時已藉由使用「指令碼動作」來自訂安裝 R 環境的叢集。R 與 Azure Blob 儲存體 (WASB) 相容，因此便可在 HDInsight 上使用 R 來處理儲存在該處的資料。

## 指令碼會執行哪些作業

用來在您的 HDInsight 叢集上安裝 R 的指令碼動作會安裝下列提供基本 R 安裝的 Ubuntu 封裝：

* [r-base](http://packages.ubuntu.com/precise/r-base)：基底 GNU R 封裝
* [r-base-dev](http://packages.ubuntu.com/precise/r-base-dev)：輔助 GNU R 封裝

下列 RHadoop 封裝也已安裝，以提供 MapReduce 和 HDFS 的整合：

* [rmr2](https://github.com/RevolutionAnalytics/rmr2)：允許 R 開發人員使用 Hadoop MapReduce
* [rhdfs](https://github.com/RevolutionAnalytics/rhdfs)：允許 R 開發人員使用 Hadoop HDFS (HDInsight 的 WASB)

此外，也會安裝下列 R 封裝：

| R 封裝 | 提供內容 |
| --------- | ---------------- |
| [rJava](https://cran.r-project.org/web/packages/rJava/index.html) | Java 介面的低階 R。 |
| [Rcpp](https://cran.r-project.org/web/packages/Rcpp/index.html) | R 與 C++ 整合。 |
| [RJSONIO](https://cran.r-project.org/web/packages/RJSONIO/index.html) | R 物件序列化/還原序列化為 JSON |
| [bitops](https://cran.r-project.org/web/packages/bitops/index.html) | 整數向量的位元運算函數。 |
| [digest](https://cran.r-project.org/web/packages/digest/index.html) | 建立 R 物件的密碼編譯雜湊摘要。 |
| [函數](https://cran.r-project.org/web/packages/functional/index.html) | Curry、撰寫和其他較高順序函數 |
| [reshape2](https://cran.r-project.org/web/packages/reshape2/index.html) | 有彈性地重新建構及彙總資料。 |
| [stringr](https://cran.r-project.org/web/packages/stringr/index.html) | 一般字串作業的簡單、一致包裝函式。 |
| [plyr](https://cran.r-project.org/web/packages/plyr/index.html) | 用於分割、套用和合併資料的工具。 |
| [caTools](https://cran.r-project.org/web/packages/caTools/index.html) | 用於移動視窗統計值、GIF、Base64、ROC AUC 等等的工具。 |
| [stringdist](https://cran.r-project.org/web/packages/stringdist/index.html) | 近似字串比對和字串距離函數。 |

## 使用指令碼動作安裝 R

下列指令碼動作是用來在 HDInsight 叢集上安裝 R。https://hdiconfigactions.blob.core.windows.net/linuxrconfigactionv01/r-installer-v01.sh
    
本節提供如何在使用 Azure 入口網站建立新叢集時使用指令碼的指示。

> [AZURE.NOTE] Azure PowerShell、Azure CLI、HDInsight .NET SDK 或 Azure Resource Manager 範本也可用來套用指令碼動作。您也可以將指令碼動作套用到執行中的叢集上。如需詳細資訊，請參閱[使用指令碼動作自訂 HDInsight 叢集](hdinsight-hadoop-customize-cluster-linux.md)。

1. 使用[佈建以 Linux 為基礎的 HDInsight 叢集](hdinsight-hadoop-provision-linux-clusters.md#portal)中的步驟開始佈建叢集，但是不完成佈建。

2. 在 [選用組態] 刀鋒視窗中，選取 [指令碼動作]，並提供下列資訊：

	* __名稱__：輸入指令碼動作的易記名稱。
	* __指令碼 URI__：https://hdiconfigactions.blob.core.windows.net/linuxrconfigactionv01/r-installer-v01.sh
	* __HEAD__：勾選此選項
	* __WORKER__：勾選此選項
	* __ZOOKEEPER__：勾選此選項以在 Zookeeper 節點上安裝。
	* __參數__：將此欄位保留空白

3. 在 [指令碼動作] 底部，使用 [選取] 按鈕以儲存組態。最後，使用 [選用組態] 刀鋒視窗底部的 [選取] 按鈕，儲存選用組態資訊。

4. 繼續如[佈建以 Linux 為基礎的 HDInsight 叢集](hdinsight-hadoop-provision-linux-clusters.md#portal)中所述佈建叢集。

## 執行 R 指令碼

叢集完成佈建之後，請使用下列步驟，在叢集上使用 R 執行 MapReduce 作業。

1. 使用 SSH 連線到 HDInsight 叢集

		ssh USERNAME@CLUSTERNAME-ssh.azurehdinsight.net

	如需搭配 HDInsight 使用 SSH 的詳細資訊，請參閱下列文章：

	* [從 Linux、Unix 或 OS X 在 HDInsight 上搭配使用 SSH 與以 Linux 為基礎的 Hadoop](hdinsight-hadoop-linux-use-ssh-unix.md)

	* [從 Windows 在 HDInsight 上搭配使用 SSH 與以 Linux 為基礎的 Hadoop](hdinsight-hadoop-linux-use-ssh-windows.md)

2. 從 `username@hn0-CLUSTERNAME:~$` 提示，輸入下列命令以啟動互動式 R 工作階段：

		R

3. 輸入下列 R 程式。這會產生數字 1 到 100，然後將它們乘以 2。

		library(rmr2)
		ints = to.dfs(1:100)
		calc = mapreduce(input = ints, map = function(k, v) cbind(v, 2*v))

	第一行會呼叫 RHadoop 程式庫 rmr2，用於 MapReduce 作業。

	第二行會產生值 1 - 100，然後使用 `to.dfs` 將它們儲存到 Hadoop 檔案系統。

	第三行會使用 rmr2 所提供的功能建立 MapReduce 程序並開始處理。您應該會在程序開始時看到幾行捲動過去。

4. 接下來，請使用下列項目查看儲存 MapReduce 輸出的目標暫存路徑：

		print(calc())

	應該類似 `/tmp/file5f615d870ad2`。若要檢視實際輸出，請使用下列項目：

		print(from.dfs(calc))

	輸出應該會看起來如下：

		[1,]  1 2
		[2,]  2 4
		.
		.
		.
		[98,]  98 196
		[99,]  99 198
		[100,] 100 200

5. 若要關閉 R，請輸入以下內容：

		q()


## 後續步驟

- [在 HDInsight 叢集上安裝及使用色調](hdinsight-hadoop-hue-linux.md)。Hue 是 Web UI，可讓您更輕鬆地建立、執行及儲存 Pig 和 Hive 工作，以及瀏覽您的 HDInsight 叢集的預設儲存體。

- [在 HDInsight 叢集上安裝 Giraph](hdinsight-hadoop-giraph-install.md)。在 HDInsight Hadoop 叢集上使用叢集自訂安裝 Giraph。Giraph 可讓您利用 Hadoop 執行圖形處理，且可以搭配 Azure HDInsight 一起使用。

- [在 HDInsight 叢集上安裝 Solr](hdinsight-hadoop-solr-install.md)。在 HDInsight Hadoop 叢集上使用叢集自訂安裝 Solr。Solr 可讓您對儲存的資料執行功能強大的搜尋作業。

- [在 HDInsight 叢集上安裝色調](hdinsight-hadoop-hue-linux.md)。在 HDInsight Hadoop 叢集上使用叢集自訂安裝 Hue。Hue 是一組 Web 應用程式，用來與 Hadoop 叢集互動。

[hdinsight-cluster-customize]: hdinsight-hadoop-customize-cluster-linux.md

<!---HONumber=AcomDC_0921_2016-->