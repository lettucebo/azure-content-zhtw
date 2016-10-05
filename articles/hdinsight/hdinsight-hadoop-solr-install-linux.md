<properties
	pageTitle="使用指令碼動作在以 Linux 為基礎的 HDInsight 上安裝 Solr | Microsoft Azure"
	description="在本主題中，您將學習如何使用指令碼動作在以 Linux 為基礎的 HDInsight Hadoop 叢集上安裝 Solr。"
	services="hdinsight"
	documentationCenter=""
	authors="Blackmist"
	manager="jhubbard"
	editor="cgronlun"
	tags="azure-portal"/>

<tags
	ms.service="hdinsight"
	ms.workload="big-data"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/13/2016"
	ms.author="larryfr"/>

# 在 HDInsight Hadoop 叢集上安裝和使用 Solr

在本主題中，您將了解如何使用指令碼動作，在 Azure HDInsight 上安裝 Solr。Solr 是強大的搜尋平台，可對 Hadoop 管理的資料執行企業級搜尋功能。在 HDInsight 叢集上安裝 Solr 之後，您也將學習如何使用 Solr 搜尋資料。

> [AZURE.NOTE] 此文件中的步驟需要以 Linux 為基礎的 HDInsight 叢集。如需搭配以 Windows 為基礎的叢集使用 Solr 的詳細資訊，請參閱[在 HDInsight Hadoop 叢集上安裝和使用 Solr (Windows)](hdinsight-hadoop-solr-install.md)。

本主題中使用的範例指令碼會以特定組態建立 Solr 叢集。如果您想要以不同的集合、分區、結構描述和複本等項目設定 Solr 叢集，則必須相應修改指令碼和 Solr 二進位檔。

## <a name="whatis"></a>什麼是 Solr？

[Apache Solr](http://lucene.apache.org/solr/features.html) 是可對資料執行強大全文搜尋作業的企業搜尋平台。Hadoop 可儲存和管理大量資料，而 Apache Solr 則是提供搜尋功能以便快速擷取資料。本主題提供如何自訂 HDInsight 叢集以安裝 Solr 的指示。

> [AZURE.WARNING] 透過 HDInsight 叢集提供的元件會受到完整支援，且 Microsoft 支援服務將協助釐清與解決這些元件的相關問題。
>
> 自訂元件 (例如 Solr) 則獲得商務上合理的支援，協助您進一步對問題進行疑難排解。如此可能會進而解決問題，或要求您利用可用管道，以找出開放原始碼技術，從中了解該技術的深度專業知識。例如，有許多社群網站可以使用，像是：[HDInsight 的 MSDN 論壇](https://social.msdn.microsoft.com/Forums/azure/zh-TW/home?forum=hdinsight)、[http://stackoverflow.com](http://stackoverflow.com)。另外，Apache 專案在 [http://apache.org](http://apache.org) 上有專案網站，例如 [Hadoop](http://hadoop.apache.org/)。

## 指令碼會執行哪些作業

此指令碼可以對 HDInsight 叢集進行下列變更：

* 將 Solr 安裝至 `/usr/hdp/current/solr`
* 建立新的使用者 __solrusr__，用來執行 Solr 服務
* 將 __solruser__ 設為 `/usr/hdp/current/solr` 的擁有者
* 新增 [Upstart](http://upstart.ubuntu.com/) 組態，如果叢集節點重新啟動，就會啟動 Solr。Solr 也會在安裝之後於叢集節點上自動啟動

## <a name="install"></a>使用指令碼動作安裝 Solr

在 HDInsight 叢集上安裝 Solr 的範例指令碼位於下列位置。

    https://hdiconfigactions.blob.core.windows.net/linuxsolrconfigactionv01/solr-installer-v01.sh

本節提供如何在使用 Azure 入口網站建立新叢集時使用範例指令碼的指示。

> [AZURE.NOTE] Azure PowerShell、Azure CLI、HDInsight .NET SDK 或 Azure Resource Manager 範本也可用來套用指令碼動作。您也可以將指令碼動作套用到執行中的叢集上。如需詳細資訊，請參閱[使用指令碼動作自訂 HDInsight 叢集](hdinsight-hadoop-customize-cluster-linux.md)。

1. 使用[佈建以 Linux 為基礎的 HDInsight 叢集](hdinsight-provision-linux-clusters.md#portal)中的步驟開始佈建叢集，但是不完成佈建。

2. 在 [選用組態] 刀鋒視窗中，選取 [指令碼動作]，並提供下列資訊：

	* __名稱__：輸入指令碼動作的易記名稱。
	* __指令碼 URI__：https://hdiconfigactions.blob.core.windows.net/linuxsolrconfigactionv01/solr-installer-v01.sh
	* __HEAD__：勾選此選項
	* __WORKER__：勾選此選項
	* __ZOOKEEPER__：勾選此選項以在 Zookeeper 節點上安裝
	* __參數__：將此欄位保留空白

3. 在 [指令碼動作] 底部，使用 [選取] 按鈕以儲存組態。最後，使用 [選用組態] 刀鋒視窗底部的 [選取] 按鈕，儲存選用組態資訊。

4. 繼續如[佈建以 Linux 為基礎的 HDInsight 叢集](hdinsight-hadoop-create-linux-clusters-portal.md)中所述佈建叢集。

## <a name="usesolr"></a>如何在 HDInsight 中使用 Solr？

### 索引資料

您必須從以某些資料檔案編製 Solr 的索引來開始。然後，您可以使用 Solr 來對已編製索引的資料執行搜尋查詢。使用下列步驟以將某些範例資料新增至 Solr，然後查詢它：

1. 使用 SSH 連線到 HDInsight 叢集

		ssh USERNAME@CLUSTERNAME-ssh.azurehdinsight.net

	如需搭配 HDInsight 使用 SSH 的詳細資訊，請參閱下列文章：

	* [從 Linux、Unix 或 OS X 在 HDInsight 上搭配使用 SSH 與以 Linux 為基礎的 Hadoop](hdinsight-hadoop-linux-use-ssh-unix.md)

	* [從 Windows 在 HDInsight 上搭配使用 SSH 與以 Linux 為基礎的 Hadoop](hdinsight-hadoop-linux-use-ssh-windows.md)

	> [AZURE.IMPORTANT] 本文件中稍後的步驟會使用 SSL 通道以連線至 Solr Web UI。為了使用這些步驟，您必須建立 SSL 通道，然後設定您的瀏覽器以使用它。
	>
	> 如需詳細資訊，請參閱[使用 SSH 通道來存取 Ambari Web UI、ResourceManager、JobHistory、NameNode、Oozie 及其他 Web UI](hdinsight-linux-ambari-ssh-tunnel.md)

2. 使用下列命令以具備 Solr 索引範例資料：

		cd /usr/hdp/current/solr/example/exampledocs
		java -jar post.jar solr.xml monitor.xml

	您會在主控台上看到下列輸出：

		POSTing file solr.xml
		POSTing file monitor.xml
		2 files indexed.
		COMMITting Solr index changes to http://localhost:8983/solr/update..
		Time spent: 0:00:01.624

	post.jar 公用程式使用 **solr.xml** 和 **monitor.xml** 這兩個範例文件對 Solr 編製索引。這些項目會儲存在 Solr 內的 __collection1__ 中。

3. 使用下列項目以查詢 Solr 公開的 REST API：

		curl "http://localhost:8983/solr/collection1/select?q=*%3A*&wt=json&indent=true"

	這會對任何符合 __*:*__ (在查詢字串中編碼為 *%3A*) 的文件針對 __collection1__ 發出查詢，回應應該以 JSON 傳回。回應看起來應該如下所示：

			"response": {
			    "numFound": 2,
			    "start": 0,
			    "maxScore": 1,
			    "docs": [
			      {
			        "id": "SOLR1000",
			        "name": "Solr, the Enterprise Search Server",
			        "manu": "Apache Software Foundation",
			        "cat": [
			          "software",
			          "search"
			        ],
			        "features": [
			          "Advanced Full-Text Search Capabilities using Lucene",
			          "Optimized for High Volume Web Traffic",
			          "Standards Based Open Interfaces - XML and HTTP",
			          "Comprehensive HTML Administration Interfaces",
			          "Scalability - Efficient Replication to other Solr Search Servers",
			          "Flexible and Adaptable with XML configuration and Schema",
			          "Good unicode support: héllo (hello with an accent over the e)"
			        ],
			        "price": 0,
			        "price_c": "0,USD",
			        "popularity": 10,
			        "inStock": true,
			        "incubationdate_dt": "2006-01-17T00:00:00Z",
			        "_version_": 1486960636996878300
			      },
			      {
			        "id": "3007WFP",
			        "name": "Dell Widescreen UltraSharp 3007WFP",
			        "manu": "Dell, Inc.",
			        "manu_id_s": "dell",
			        "cat": [
			          "electronics and computer1"
			        ],
			        "features": [
			          "30" TFT active matrix LCD, 2560 x 1600, .25mm dot pitch, 700:1 contrast"
			        ],
			        "includes": "USB cable",
			        "weight": 401.6,
			        "price": 2199,
			        "price_c": "2199,USD",
			        "popularity": 6,
			        "inStock": true,
			        "store": "43.17614,-90.57341",
			        "_version_": 1486960637584081000
			      }
			    ]
			  }

### 使用 Solr 儀表板

Solr 儀表板是 Web UI，可讓您透過網頁瀏覽器使用 Solr。Solr 儀表板不會直接從您的 HDInsight 叢集公開至網際網路上，必須使用 SSH 通道來存取。如需使用 SSH 通道的詳細資訊，請參閱[使用 SSH 通道來存取 Ambari Web UI、ResourceManager、JobHistory、NameNode、Oozie 及其他 Web UI](hdinsight-linux-ambari-ssh-tunnel.md)

一旦您建立 SSH 通道，請使用下列步驟以使用 Solr 儀表板：

1. 決定主要前端節點的主機名稱：

    1. 在連接埠 22 上使用 SSH 連接到叢集。例如，`ssh USERNAME@CLUSTERNAME-ssh.azurehdinsight.net`，其中 __USERNAME__ 是您的 SSH 使用者名稱，__CLUSTERNAME__ 是您的叢集名稱。

        如需使用 SSH 的詳細資訊，請參閱下列文件：

        * [從 Linux、Unix 或 Mac OS X 用戶端搭配使用 SSH 與以 Linux 為基礎的 HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md)

        * [從 Windows 用戶端搭配使用 SSH 與以 Linux 為基礎的 HDInsight](hdinsight-hadoop-linux-use-ssh-windows.md)
    
    3. 使用下列命令以取得完整主機名稱︰

            hostname -f

        此命令會傳回類似以下的名稱：

            hn0-myhdi-nfebtpfdv1nubcidphpap2eq2b.ex.internal.cloudapp.net
    
        這是應該用於下列步驟中的主機名稱。
    
1. 在瀏覽器中，連接到 __http://HOSTNAME:8983/solr/#/__，其中 __HOSTNAME\_\_ 是您在先前步驟中決定的名稱。

    此要求應該會透過 SSH 通道路由傳送至您的 HDInsight 叢集的前端節點。您應該會看到如下所示的頁面：

	![Solr 儀表板的映像](./media/hdinsight-hadoop-solr-install-linux/solrdashboard.png)

2. 從左窗格使用 [核心選取器] 下拉式清單，以選取 [collection1]。數個項目應該會出現在 __collection1__ 底下。

3. 從 __collection1__ 底下的項目中，選取 [查詢] 。使用下列值來填入搜尋頁面：

	* 在 [**q**] 文字方塊中輸入 ***:***。如此便會傳回已在 Solr 中編製索引的所有文件。如果您想要搜尋文件內的特定字串，您可以在此輸入該字串。

	* 在 [**wt**] 文字方塊中，選取輸出格式。預設值是 [**json**]。

	最後，選取搜尋頁面底部的 [執行查詢] 按鈕。

	![使用指令碼動作以自訂叢集](./media/hdinsight-hadoop-solr-install-linux/hdi-solr-dashboard-query.png)

	輸出中會傳回兩個我們之前用於對 Solr 編製索引的文件。輸出結果類似下面：

			"response": {
			    "numFound": 2,
			    "start": 0,
			    "maxScore": 1,
			    "docs": [
			      {
			        "id": "SOLR1000",
			        "name": "Solr, the Enterprise Search Server",
			        "manu": "Apache Software Foundation",
			        "cat": [
			          "software",
			          "search"
			        ],
			        "features": [
			          "Advanced Full-Text Search Capabilities using Lucene",
			          "Optimized for High Volume Web Traffic",
			          "Standards Based Open Interfaces - XML and HTTP",
			          "Comprehensive HTML Administration Interfaces",
			          "Scalability - Efficient Replication to other Solr Search Servers",
			          "Flexible and Adaptable with XML configuration and Schema",
			          "Good unicode support: héllo (hello with an accent over the e)"
			        ],
			        "price": 0,
			        "price_c": "0,USD",
			        "popularity": 10,
			        "inStock": true,
			        "incubationdate_dt": "2006-01-17T00:00:00Z",
			        "_version_": 1486960636996878300
			      },
			      {
			        "id": "3007WFP",
			        "name": "Dell Widescreen UltraSharp 3007WFP",
			        "manu": "Dell, Inc.",
			        "manu_id_s": "dell",
			        "cat": [
			          "electronics and computer1"
			        ],
			        "features": [
			          "30" TFT active matrix LCD, 2560 x 1600, .25mm dot pitch, 700:1 contrast"
			        ],
			        "includes": "USB cable",
			        "weight": 401.6,
			        "price": 2199,
			        "price_c": "2199,USD",
			        "popularity": 6,
			        "inStock": true,
			        "store": "43.17614,-90.57341",
			        "_version_": 1486960637584081000
			      }
			    ]
			  }

### 啟動和停止 Solr

如果您需要手動停止或啟動 Solar，請使用下列命令：

	sudo stop solr

	sudo start solr

## 備份已編製索引的資料

您最好從 Solr 叢集節點將已編製索引的資料備份到 Azure Blob 儲存體。請執行下列步驟來進行此作業：

1. 使用 SSH 連線到叢集，然後使用下列命令來取得前端節點的主機名稱：

        hostname -f
        
2. 使用下列命令來建立已編製索引之資料的快照。以上一個命令傳回的名稱取代 __HOSTNAME__：

		curl http://HOSTNAME:8983/solr/replication?command=backup

	您應該會看到如下所示的回應：

		<?xml version="1.0" encoding="UTF-8"?>
		<response>
		  <lst name="responseHeader">
		    <int name="status">0</int>
		    <int name="QTime">9</int>
		  </lst>
		  <str name="status">OK</str>
		</response>

2. 接下來，將目錄變更為 __/usr/hdp/current/solr/example/solr__。在這裡每個集合會有子目錄。每個集合目錄包含__資料__目錄，這是該集合的快照所在的位置。

	例如，如果您使用先前的步驟來編製範例文件的索引，__/usr/hdp/current/solr/example/solr/collection1/data__ 目錄現在應該包含一個名為 __snapshot.###########__ 的目錄，其中 # 是快照的日期和時間。

3. 使用如下的命令，建立快照資料夾的壓縮封存：

		tar -zcf snapshot.20150806185338855.tgz snapshot.20150806185338855

	這會建立名為 __snapshot.20150806185338855.tgz__ 的新封存，其中包含 __snapshot.20150806185338855__ 目錄的內容。

3. 然後您可以使用下列命令，將封存儲存至叢集的主要儲存體：

	hadoop fs -copyFromLocal snapshot.20150806185338855.tgz /example/data

	> [AZURE.NOTE] 您可能想要建立用來儲存 Solr 快照的專用目錄。例如，`hadoop fs -mkdir /solrbackup`。

如需有關使用 Solr 備份和還原的詳細資訊，請參閱[製作和還原 SolrCores 的備份](https://cwiki.apache.org/confluence/display/solr/Making+and+Restoring+Backups+of+SolrCores)。


## 另請參閱

- [在 HDInsight 叢集上安裝及使用 Hue](hdinsight-hadoop-hue-linux.md)。Hue 是 Web UI，可讓您更輕鬆地建立、執行及儲存 Pig 和 Hive 工作，以及瀏覽您的 HDInsight 叢集的預設儲存體。

- [在 HDInsight 叢集上安裝 R][hdinsight-install-r]。在 HDInsight Hadoop 叢集上使用叢集自訂安裝 R。R 是一個用於統計計算的開放原始碼語言和環境。它提供數百個內建的統計函式及它自己的程式設計語言，此語言結合了函式型和物件導向程式設計的層面。它也提供廣泛的圖形功能。

- [在 HDInsight 叢集上安裝 Giraph](hdinsight-hadoop-giraph-install-linux.md)。在 HDInsight Hadoop 叢集上使用叢集自訂安裝 Giraph。Giraph 可讓您利用 Hadoop 執行圖形處理，且可以搭配 Azure HDInsight 一起使用。

- [在 HDInsight 叢集上安裝 Hue](hdinsight-hadoop-hue-linux.md)。在 HDInsight Hadoop 叢集上使用叢集自訂安裝 Hue。Hue 是一組 Web 應用程式，用來與 Hadoop 叢集互動。



[hdinsight-install-r]: hdinsight-hadoop-r-scripts-linux.md
[hdinsight-cluster-customize]: hdinsight-hadoop-customize-cluster-linux.md

<!---HONumber=AcomDC_0921_2016-->