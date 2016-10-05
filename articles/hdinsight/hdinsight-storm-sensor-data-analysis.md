<properties
   pageTitle="使用 Apache Storm 和 HBase 分析感應器資料 | Microsoft Azure"
   description="了解如何透過虛擬網路連線至 Apache Storm。搭配使用 Storm 和 HBase 來處理事件中樞的感應器資料，並使用 D3.js 將其視覺化呈現。"
   services="hdinsight"
   documentationCenter=""
   authors="Blackmist"
   manager="jhubbard"
   editor="cgronlun"/>

<tags
   ms.service="hdinsight"
   ms.devlang="java"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="big-data"
   ms.date="09/20/2016"
   ms.author="larryfr"/>

# 在 HDInsight (Hadoop) 中使用 Apache Storm、事件中樞和 HBase 分析感應器資料 

了解如何使用 Apache Storm on HDInsight 來處理 Azure 事件中樞的感應器資料，將此資料儲存在 Apache HBase on HDInsight 中，並使用當作 Azure Web 應用程式執行的 D3.js 將其視覺化呈現。

本文件中使用的 Azure Resource Manager 範本示範如何在一個資源群組中建立多個 Azure 資源。具體來說，它會建立一個 Azure 虛擬網路、兩個 HDInsight 叢集 (Storm 和 HBase) 以及一個 Azure Web 應用程式。即時 Web 儀表板的 node.js 實作會自動部署至 Web 應用程式。

> [AZURE.NOTE] 本文件中的資訊以及所提供的範例，都已使用以 Linux 為基礎的 HDInsight 3.3 及 3.4 叢集版本進行測試。

## 必要條件

* Azure 訂用帳戶。請參閱[取得 Azure 免費試用](http://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/)。

    > [AZURE.IMPORTANT] 您不需要現有的 HDInsight 叢集；本文件中的步驟將會建立下列資源︰
    >
    > * Azure 虛擬網路
    > * Storm on HDInsight 叢集 (以 Linux 為基礎，2 個背景工作節點)
    > * HBase on HDInsight 叢集 (以 Linux 為基礎，2 個背景工作節點)
    > * 裝載 Web 儀表板的 Azure Web 應用程式

* [Node.js](http://nodejs.org/)︰這用來預覽您的開發環境上的本機 Web 儀表板。

* [Java 和 JDK 1.7](http://www.oracle.com/technetwork/java/javase/downloads/index.html)︰用來開發 Storm 拓撲。

* [Maven](http://maven.apache.org/what-is-maven.html)︰用來建置和編譯專案。

* [Git](http://git-scm.com/)︰用來從 GitHub 下載專案。

* __SSH 用戶端__︰用來連接至以 Linux 為基礎的 HDInsight 叢集。如需搭配 HDInsight 使用 SSH 的詳細資訊，請參閱下列文件。

    * [從 Windows 用戶端搭配使用 SSH 與 HDInsight](hdinsight-hadoop-linux-use-ssh-windows.md)

    * [從 Linux、Unix 或 Mac 用戶端搭配使用 SSH 與 HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md)

    > [AZURE.NOTE] 您也必須能夠存取 `scp` 命令，該命令用於在本機開發環境與使用 SSH 的 HDInsight 叢集之間複製檔案。

## 架構

![架構圖表](./media/hdinsight-storm-sensor-data-analysis/devicesarchitecture.png)

此範例由下列元件組成：

* **Azure 事件中樞**：包含從感應器收集的資料。在此範例中，假設應用程式會產生資料。

* **Storm on HDInsight**：即時處理事件中樞的資料。

* **HBase on HDInsight**：針對 Storm 所處理過的資料，提供持續的 NoSQL 資料存放區。

* **Azure 虛擬網路服務**：確保 Storm on HDInsight 及 HBase on HDInsight 叢集之間能夠安全通訊。

    > [AZURE.NOTE] 需要虛擬網路才能使用 Java HBase 用戶端 API，因為它不會透過 HBase 叢集的公用閘道公開。將 HBase 和 Storm 叢集安裝到相同的虛擬網路，可讓 Storm 叢集 (或虛擬網路上的任何其他系統) 直接使用用戶端 API 存取 HBase。

* **儀表板網站**：即時建立資料圖表的範例儀表板。

	* 此網站採用 Node.js，因此可在任何用戶端作業系統上測試，或者也能部署至 Azure 網站。

	* [Socket.io](http://socket.io/) 用於 Storm 拓撲及網站間的即時通訊。

		> [AZURE.NOTE] 這是實作的詳細資料。您可以使用任何通訊架構，例如原始 WebSockets 或 SignalR。

	* [D3.js](http://d3js.org/) 用於為傳送至網站的資料繪圖。

拓撲會使用 [org.apache.storm.eventhubs.spout.EventHubSpout](http://storm.apache.org/releases/0.10.1/javadocs/org/apache/storm/eventhubs/spout/class-use/EventHubSpout.html) 類別從事件中樞讀取資料，並使用 [org.apache.storm.hbase.bolt.HBaseBolt](https://storm.apache.org/javadoc/apidocs/org/apache/storm/hbase/bolt/class-use/HBaseBolt.html) 類別將資料寫入至 HBase。與網站之間的通訊必須透過 [socket.io-client.java](https://github.com/nkzawa/socket.io-client.java) 來達成。

此拓撲的圖表如下。

![拓撲圖](./media/hdinsight-storm-sensor-data-analysis/sensoranalysis.png)

> [AZURE.NOTE] 這是相當簡化的拓撲樣貌。執行過程中，系統會針對正在讀取之事件中樞的每個分割區，為各個元件建立執行個體。這些執行個體會分散至叢集的節點上，而資料在節點間路由傳送的情形如下：
>
> * 從 Spout 傳送至剖析器的資料會經過負載平衡。
> * 從剖析器傳送至儀表板及 HBase的資料會依照裝置識別碼分組，讓相同裝置的訊息一律傳送至相同元件。

### 拓撲元件

* **事件中樞 Spout**：Spout 會當作 Apache Storm 0.10.0 版和更新版本的一部分提供。

    > [AZURE.NOTE] 此範例中使用的事件中樞 Spout 需要 HDInsight 叢集 3.3 或 3.4 版上的 Storm。如需如何使用事件中樞搭配舊版 Storm on HDInsight 的相關資訊，請參閱[使用 Storm on HDInsight 處理 Azure 事件中樞的事件](hdinsight-storm-develop-java-event-hub-topology.md)。

* **ParserBolt.java**：Spout 發出的資料為原始 JSON，有時會一次發出多個事件。此 Bolt 示範如何讀取 Spout 發出的資料，並將其以包含多個欄位之 Tuple 的形式發至新的串流。

* **DashboardBolt.java**：示範如何使用 Socket.io 用戶端程式庫，讓 Java 將資料即時傳送至 Web 儀表板。

這個範例會使用 [Flux](https://storm.apache.org/releases/0.10.0/flux.html) 架構，因此拓撲定義是包含在 YAML 檔案中。檔案有兩個：

* __no-hbase.yaml__ - 在開發環境中測試拓撲時會使用此檔案。它不會使用 HBase 元件，因為您無法從叢集所在的虛擬網路之外存取 HBase Java API。

* __with-hbase.yaml__ - 將拓撲部署至 Storm 叢集時會使用此檔案。它會使用 HBase 元件，因為它是在和 HBase 叢集相同的虛擬網路中執行。

## 準備您的環境

使用此範例之前，您必須建立 Azure 事件中樞以供 Storm 拓撲讀取。

### 設定事件中樞

事件中樞是此範例的資料來源。請使用下列步驟建立新的事件中心。

1. 從 [Azure 入口網站](https://portal.azure.com)，選取 [+ 新增] -> [物聯網] -> [事件中樞]。

2. 在 [建立命名空間] 刀鋒視窗中，執行下列工作︰

    1. 輸入命名空間的__名稱__。
    2. 選取定價層。此範例使用 [基本] 就已足夠。
    3. 選取要使用的 Azure [訂用帳戶]。
    4. 選取現有的資源群組，或建立一個新的群組。
    5. 選取事件中樞的 [位置]。
    6. 選取 [釘選到儀表板]，然後按一下 [建立]。

3. 建立程序完成時，就會顯示您的命名空間的 [事件中樞] 刀鋒視窗。從這裡選取 [+ 新增事件中樞]。在 [建立事件中樞] 刀鋒視窗中，輸入 [sensordata] 的名稱，然後選取 [建立]。讓其他欄位保持使用預設值。

4. 從您的命名空間的 [事件中樞] 刀鋒視窗中，選取 [事件中樞]。選取 [sensordata] 項目。

5. 從 sensordata 事件中樞的刀鋒視窗中，選取 [共用存取原則]。使用 [+ 新增] 連結新增下列原則︰


	| 原則名稱 | Claims |
    | ----- | ----- |
	| devices | 傳送 |
	| storm | 接聽 |

5. 選取這兩個原則，並記下 [主索引鍵] 值。在後續步驟中，您會需要這兩個原則的值。

## 下載並設定專案。

使用下列項目從 GitHub 下載專案。

	git clone https://github.com/Blackmist/hdinsight-eventhub-example

命令執行完畢後，您會擁有以下目錄架構：

	hdinsight-eventhub-example/
		TemperatureMonitor/ - this contains the topology
			resources/
                log4j2.xml - set logging to minimal
                no-hbase.yaml - topology definition for local testing
                with-hbase.yaml - topology definition that uses HBase in a virutal network
			src/ - the Java bolts
            dev.properties - contains configuration values for your environment
		dashboard/nodejs/ - this is the node.js web dashboard
		SendEvents/ - utilities to send fake sensor data

> [AZURE.NOTE] 本文件不會深入探究此範例包含的程式碼，不過會附上程式碼的完整註解。

開啟 **hdinsight-eventhub-example/TemperatureMonitor/dev.properties** 檔案，並對下列幾行新增事件中樞資訊︰

    eventhub.read.policy.name: storm
    eventhub.read.policy.key: KeyForTheStormPolicy
    eventhub.namespace: YourNamespace
    eventhub.name: sensordata

> [AZURE.NOTE] 此範例假設您使用了 __storm__ 做為具有 __Listen__ 宣告的原則名稱，而且您的事件中樞名稱為 __sensordata__。

 資訊新增完畢後，請儲存檔案。

## 編譯並在本機測試

測試前，您必須先開啟儀表板，檢視拓撲輸出的資料並產生要存放在事件中樞的資料。

> [AZURE.IMPORTANT] 此拓撲的 HBase 元件在本機測試時不在作用中，因為無法從包含叢集的 Azure 虛擬網路外部存取 HBase 叢集的 Java API。

### 啟動 Web 應用程式

1. 開啟新的命令提示字元或終端機，將目錄變更至 **hdinsight-eventhub-example/dashboard**，然後使用以下命令安裝 Web 應用程式需要的相依項目：

		npm install

2. 使用下列命令啟動 Web 應用程式：

		node server.js

	您應該會看見類似下方的訊息：

		Server listening at port 3000

2. 開啟 Web 瀏覽器，在網址列輸入 **http://localhost:3000/**。您應該會看到如下所示的頁面：

	![Web 儀表板](./media/hdinsight-storm-sensor-data-analysis/emptydashboard.png)

	保持開啟此命令提示字元或終端機。測試完成後，使用 Ctrl-C 鍵中斷 Web 伺服器。

### 開始產生資料

> [AZURE.NOTE] 本節中的步驟採用 Node.js，因此可使用於任何平台上。如需其他語言的範例，請查看 [SendEvents] 目錄。

1. 開啟新的命令提示字元、殼層或終端機，將目錄變更至 **hdinsight-eventhub-example/SendEvents/nodejs**，然後使用以下命令安裝應用程式需要的相依項目：

		npm install

2. 以文字編輯器開啟 **app.js** 檔案，並新增您稍早取得的事件中樞資訊：

		// ServiceBus Namespace
		var namespace = 'YourNamespace';
		// Event Hub Name
		var hubname ='sensordata';
		// Shared access Policy name and key (from Event Hub configuration)
		var my_key_name = 'devices';
		var my_key = 'YourKey';
    
    > [AZURE.NOTE] 此範例假設您使用了 __sensordata__ 做為事件中樞的名稱，並使用 __devices__ 做為具有 __Send__ 宣告的原則名稱。

2. 使用以下命令在事件中樞插入新項目：

		node app.js

	您應該會看見好幾行輸出資料，傳送至事件中樞的資料也會包含在內。此資訊類似以下內容：

		{"TimeStamp":"2015-02-10T14:43.05.00320Z","DeviceId":"0","Temperature":7}
		{"TimeStamp":"2015-02-10T14:43.05.00320Z","DeviceId":"1","Temperature":39}
		{"TimeStamp":"2015-02-10T14:43.05.00320Z","DeviceId":"2","Temperature":86}
		{"TimeStamp":"2015-02-10T14:43.05.00320Z","DeviceId":"3","Temperature":29}
		{"TimeStamp":"2015-02-10T14:43.05.00320Z","DeviceId":"4","Temperature":30}
		{"TimeStamp":"2015-02-10T14:43.05.00320Z","DeviceId":"5","Temperature":5}
		{"TimeStamp":"2015-02-10T14:43.05.00320Z","DeviceId":"6","Temperature":24}
		{"TimeStamp":"2015-02-10T14:43.05.00320Z","DeviceId":"7","Temperature":40}
		{"TimeStamp":"2015-02-10T14:43.05.00320Z","DeviceId":"8","Temperature":43}
		{"TimeStamp":"2015-02-10T14:43.05.00320Z","DeviceId":"9","Temperature":84}

### 啟動拓撲

2. 開啟新的命令提示字元、殼層或終端機，將目錄變更至 __hdinsight-eventhub-example/TemperatureMonitor__，然後使用下列命令啟動拓撲︰

        mvn compile exec:java -Dexec.args="--local -R /no-hbase.yaml --filter dev.properties"
    
    如果您要使用 PowerShell，請改用下列命令︰

        mvn compile exec:java "-Dexec.args=--local -R /no-hbase.yaml --filter dev.properties"

    > [AZURE.NOTE] 如果您是在 Linux/Unix/OS X 系統上，而且[已在開發環境中安裝 Storm](http://storm.apache.org/releases/0.10.0/Setting-up-development-environment.html)，您可以改用下列命令︰
    >
    > `mvn compile package` `storm jar target/WordCount-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux --local -R /no-hbase.yaml`

	這會以本機模式啟動 __no-hbase.yaml__ 檔案中定義的拓撲。__dev.properties__ 檔案中包含的值可提供事件中樞的連線資訊。啟動之後，拓撲會從事件中樞讀取項目，並將它們傳送到在本機電腦上執行的儀表板。您應該會看見 Web 儀表板中出現資料行，如下所示：

	![儀表板與資料](./media/hdinsight-storm-sensor-data-analysis/datadashboard.png)

3. 儀表板執行時，請使用前幾個步驟的 `node app.js` 命令，將新的資料傳送至事件中樞。由於溫度值是隨機產生，因此圖形必須更新才能顯示大量溫度變更。

    > [AZURE.NOTE] 在使用 `node app.js` 命令時，您必須位於 __hdinsight-eventhub-example/SendEvents/Nodejs__ 目錄。

3. 確定更新成功後，請使用 Ctrl+C 鍵停止拓撲。您也可以使用 Ctrl-C 鍵停止本機 Web 伺服器。

## 建立 Storm 和 HBase 叢集

若要在 HDInsight 上執行拓撲，並啟用 HBase bolt，您必須建立新的 Storm 叢集和 HBase 叢集。本節中的步驟使用 [Azure Resource Manager 範本](../resource-group-template-deploy.md)，在虛擬網路上建立新的 Azure 虛擬網路以及 Storm 和 HBase 叢集。這些範本也會建立 Azure Web 應用程式並在其中部署一份儀表板。

> [AZURE.NOTE] 使用虛擬網路，在 Storm 叢集上執行的拓撲便可以使用 HBase Java API 直接與 HBase 叢集通訊。

本文件中使用的 Resource Manager 範本位於公用 Blob 容器 (\_\_https://hditutorialdata.blob.core.windows.net/armtemplates/create-linux-based-hbase-storm-cluster-in-vnet.json__) 中。

1. 按一下以下按鈕，在 Azure 入口網站中登入 Azure 並開啟 Resource Manager 範本。

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fhditutorialdata.blob.core.windows.net%2Farmtemplates%2Fcreate-linux-based-hbase-storm-cluster-in-vnet.json" target="_blank"><img src="https://acom.azurecomcdn.net/80C57D/cdn/mediahandler/docarticles/dpsmedia-prod/azure.microsoft.com/documentation/articles/hdinsight-hbase-tutorial-get-started-linux/20160201111850/deploy-to-azure.png" alt="Deploy to Azure"></a>

2. 從 [參數] 刀鋒視窗，輸入下列項目：

    ![HDInsight 參數](./media/hdinsight-storm-sensor-data-analysis/parameters.png)
    
    * **BASECLUSTERNAME**︰此值將做為 Storm 和 HBase 叢集的基底名稱。例如，輸入 __hdi__ 將會建立名為 __storm-hdi__ 的 Storm 叢集以及名為 __hbase-hdi__ 的 HBase 叢集。
    * __CLUSTERLOGINUSERNAME__：Storm 和 HBase 叢集的系統管理員使用者名稱。
    * __CLUSTERLOGINPASSWORD__：Storm 和 HBase 叢集的系統管理員使用者密碼。
    * __SSHUSERNAME__︰要針對 Storm 和 HBase 叢集建立的 SSH 使用者。
    * __SSHPASSWORD__：Storm 和 HBase 叢集的 SSH 使用者的密碼。
    * __LOCATION__︰會在其中建立叢集的區域。
    
    按一下 [確定] 儲存參數。
    
3. 使用 [資源群組] 區段建立新的資源群組，或選取現有的資源群組。

4. 在 [資源群組位置] 下拉式功能表中，選取與您針對 __LOCATION__ 參數選取的相同位置。

5. 選取 [法律條款]，然後選取 [建立]。

6. 最後，核取 [釘選到儀表板]，然後選取 [建立]。這大約需要 20 分鐘來建立叢集。

建立資源後，您將會重新導向至資源群組的刀鋒視窗，其中內含叢集和 Web 儀表板。

![Vnet 和叢集的資源群組刀鋒視窗](./media/hdinsight-storm-sensor-data-analysis/groupblade.png)

> [AZURE.IMPORTANT] 請注意，HDInsight 叢集的名稱是 __storm-BASENAME__ 和 __hbase-BASENAME__，其中 BASENAME 是您提供給範本的名稱。連接到叢集時，您將在稍後步驟中使用這些名稱。另請注意，儀表板網站的名稱是 __basename-dashboard__。您稍後將在檢視儀表板時使用此名稱。

## 設定儀表板 Bolt

若要將資料傳送至部署為 Web 應用程式的儀表板，您必須修改 __dev.properties__ 檔案中的下面這行︰

    dashboard.uri: http://localhost:3000

將 `http://localhost:3000` 變更為 `http://BASENAME-dashboard.azurewebsites.net` 並儲存檔案。以您在先前步驟中提供的基底名稱取代 __BASENAME__。您也可以使用先前建立的資源群組來選取儀表板和檢視 URL。

## 建立 HBase 資料表

若要將資料儲存在 HBase 中，我們必須先建立資料表。您通常會想預先建立 Storm 需要寫入的資源，因為嘗試從 Storm 拓撲內部建立資源，可能會導致多個分散式程式碼複本嘗試建立相同的資源。在拓撲外部建立資源，而且只使用 Storm 僅行讀取/寫入和分析。

1. 使用您在叢集建立期間提供給範本的 SSH 使用者和密碼，透過 SSH 連線到 HBase 叢集。例如，如果使用 `ssh` 命令進行連線，您應使用下列語法：

        ssh USERNAME@hbase-BASENAME-ssh.azurehdinsight.net
    
    在此命令中，以建立叢集時所提供的 SSH 使用者名稱取代 __USERNAME__，並以您提供的基底名稱取代 __BASENAME__。出現提示時，請輸入 SSH 使用者的密碼。

2. 在 SSH 工作階段中，啟動 HBase Shell。

    	hbase shell
	
	載入 Shell 後，您會看到 `hbase(main):001:0>` 提示。

3. 在 HBase Shell 中輸入下列命令，以建立將儲存感應器資料的資料表：

    	create 'SensorData', 'cf'

4. 使用下列命令來確認資料表已建立：

    	scan 'SensorData'
		
	這會傳回類似下列範例的資訊，指出資料表中有 0 個資料列。
	
		ROW                   COLUMN+CELL                                       0 row(s) in 0.1900 seconds

5. 輸入下列命令來結束 HBase Shell：

		exit

## 設定 HBase Bolt

若要從 Storm 叢集寫入至 HBase，您必須將 HBase 叢集的組態詳細資料提供給 HBase bolt。這樣做的最簡單方式是從叢集下載 __hbase-site.xml__ 並納入您的專案中。您必須也在 __pom.xml__ 檔案中取消註解數個相依項目，以載入 storm-hbase 元件和必要的相依項目。

> [AZURE.IMPORTANT] 您還必須下載在 HDInsight 3.3 或 3.4 叢集上您的 Storm 上提供的 storm-hbase.jar 檔案；這個版本會編譯成使用 HBase 1.1.x，其用於 HDInsight 3.3 和 3.4 叢集上的 HBase。如果您從其他地方使用 storm-hbase 元件，該原則可能會針對舊版 HBase 進行編譯。

### 下載 hbase-site.xml

從命令提示字元中，使用 SCP 來下載 __hbase-site.xml__ 叢集的 datasourcegroups.xml 檔。在下列範例中，以建立叢集時所提供的 SSH 使用者取代 __USERNAME__，並以您稍早提供的基底名稱取代 __BASENAME__。出現提示時，請輸入 SSH 使用者的密碼。以 TemperatureMonitor 專案中此檔案的路徑取代 `/path/to/TemperatureMonitor/resources/hbase-site.xml`。

    scp USERNAME@hbase-BASENAME-ssh.azurehdinsight.net:/etc/hbase/conf/hbase-site.xml /path/to/TemperatureMonitor/resources/hbase-site.xml

這會將 __hbase-site.xml__ 下載到指定的路徑。

### 下載並安裝 storm-hbase 元件

1. 在命令提示字元中，使用 SCP 從 Storm 叢集下載 __storm-hbase.jar__ 檔案。在下列範例中，以建立叢集時所提供的 SSH 使用者取代 __USERNAME__，並以您稍早提供的基底名稱取代 __BASENAME__。出現提示時，請輸入 SSH 使用者的密碼。

        scp USERNAME@storm-BASENAME-ssh.azurehdinsight.net:/usr/hdp/current/storm-client/contrib/storm-hbase/storm-hbase*.jar .

    這會下載名為 `storm-hbase-####.jar` 的，其中 ### 是此叢集的 Storm 版本號碼。請記下這個號碼，以便稍後使用。

2. 使用下列命令，將此元件安裝到開發環境上的本機 Maven 儲存機制中。這可讓 Maven 在編譯專案時找到封裝。以檔案名稱中包含的版本號碼取代 __###__。

        mvn install:install-file -Dfile=storm-hbase-####.jar -DgroupId=org.apache.storm -DartifactId=storm-hbase -Dversion=#### -Dpackaging=jar
    
    如果您要使用 PowerShell，請使用下列命令︰

        mvn install:install-file "-Dfile=storm-hbase-####.jar" "-DgroupId=org.apache.storm" "-DartifactId=storm-hbase" "-Dversion=####" "-Dpackaging=jar"

### 在專案中啟用 storm-hbase 元件

1. 開啟 __TemperatureMonitor/pom.xml__ 檔案並刪除下列幾行︰

        <!-- uncomment this section to enable the hbase-bolt
        end comment for hbase-bolt section -->
    
    > [AZURE.IMPORTANT] 只要刪除這兩行；請勿刪除它們之間的任何行。
    
    這可啟用使用 hbase bolt 與 HBase 通訊時所需的數個元件。

2. 找出下列幾行，然後再以您稍早下載的 storm-hbase 檔案版本號碼取代 __###__。

        <dependency>
            <groupId>org.apache.storm</groupId>
            <artifactId>storm-hbase</artifactId>
            <version>####</version>
        </dependency>

    > [AZURE.IMPORTANT] 版本號碼必須符合將元件安裝在本機 Maven 儲存機制時使用的版本，因為 Maven 會在建置專案時使用此資訊來載入元件。

2. 儲存 __pom.xml__ 檔案。

## 建置、封裝及部署應用程式到 HDInsight

在開發環境中，使用下列步驟將 Storm 拓撲部署至 Storm 叢集。

1. 從 __TemperatureMonitor__ 目錄使用下列命令來執行新的組建，並從專案建立 JAR 封裝：

		mvn clean compile package

	這樣會在專案的 **target** 目錄中建立名為 **TemperatureMonitor-1.0-SNAPSHOT.jar** 的檔案。

2. 使用 scp 將 __TemperatureMonitor-1.0-SNAPSHOT.jar__ 檔案上傳至您的 Storm 叢集。在下列範例中，以建立叢集時所提供的 SSH 使用者取代 __USERNAME__，並以您稍早提供的基底名稱取代 __BASENAME__。出現提示時，請輸入 SSH 使用者的密碼。

        scp target\TemperatureMonitor-1.0-SNAPSHOT.jar USERNAME@storm-BASENAME-ssh.azurehdinsight.net:TemperatureMonitor-1.0-SNAPSHOT.jar
    
    > [AZURE.NOTE] 上傳檔案可能需要幾分鐘的時間，因為其大小為 MB。

    使用 scp 來上傳 __dev.properties__ 檔案，因為其中包含用來連線到事件中樞和儀表板的資訊。

        scp dev.properties USERNAME@storm-BASENAME-ssh.azurehdinsight.net:dev.properties

3. 上傳檔案後，使用 SSH 連線到叢集。

        ssh USERNAME@storm-BASENAME-ssh.azurehdinsight.net

4. 在 SSH 工作階段中，使用以下命令啟動拓撲。

        storm jar TemperatureMonitor-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux --remote -R /with-hbase.yaml --filter dev.properties
    
    這會使用 __with-hbase.yaml__ 檔案中的拓撲定義和 __dev.properties__ 檔案中的組態值啟動拓撲。

3. 拓撲啟動後，以瀏覽器開啟您發佈至 Azure 的網站，然後使用 `node app.js` 命令將資料傳送至事件中樞。您應該會看見顯示資訊的 Web 儀表板更新。

	![儀表板](./media/hdinsight-storm-sensor-data-analysis/datadashboard.png)

## 檢視 HBase 資料

使用 `node app.js` 將資料提交到拓撲之後，請使用下列步驟連接到 HBase，並確認資料已寫入您稍早建立的資料表。

1. 使用 SSH 連接到 HBase 叢集。

        ssh USERNAME@hbase-BASENAME-ssh.azurehdinsight.net

2. 在 SSH 工作階段中，啟動 HBase Shell。

    	hbase shell
	
	載入 Shell 後，您會看到 `hbase(main):001:0>` 提示。

2. 檢視資料表中的資料列︰

    	scan 'SensorData'
		
	應該會傳回類似以下的資訊，指出資料表中有 0 個資料列。
	
		hbase(main):002:0> scan 'SensorData'
        ROW                             COLUMN+CELL
        \x00\x00\x00\x00               column=cf:temperature, timestamp=1467290788277, value=\x00\x00\x00\x04
        \x00\x00\x00\x00               column=cf:timestamp, timestamp=1467290788277, value=2015-02-10T14:43.05.00320Z
        \x00\x00\x00\x01               column=cf:temperature, timestamp=1467290788348, value=\x00\x00\x00M
        \x00\x00\x00\x01               column=cf:timestamp, timestamp=1467290788348, value=2015-02-10T14:43.05.00320Z
        \x00\x00\x00\x02               column=cf:temperature, timestamp=1467290788268, value=\x00\x00\x00R
        \x00\x00\x00\x02               column=cf:timestamp, timestamp=1467290788268, value=2015-02-10T14:43.05.00320Z
        \x00\x00\x00\x03               column=cf:temperature, timestamp=1467290788269, value=\x00\x00\x00#
        \x00\x00\x00\x03               column=cf:timestamp, timestamp=1467290788269, value=2015-02-10T14:43.05.00320Z
        \x00\x00\x00\x04               column=cf:temperature, timestamp=1467290788356, value=\x00\x00\x00>
        \x00\x00\x00\x04               column=cf:timestamp, timestamp=1467290788356, value=2015-02-10T14:43.05.00320Z
        \x00\x00\x00\x05               column=cf:temperature, timestamp=1467290788326, value=\x00\x00\x00\x0D
        \x00\x00\x00\x05               column=cf:timestamp, timestamp=1467290788326, value=2015-02-10T14:43.05.00320Z
        \x00\x00\x00\x06               column=cf:temperature, timestamp=1467290788253, value=\x00\x00\x009
        \x00\x00\x00\x06               column=cf:timestamp, timestamp=1467290788253, value=2015-02-10T14:43.05.00320Z
        \x00\x00\x00\x07               column=cf:temperature, timestamp=1467290788229, value=\x00\x00\x00\x12
        \x00\x00\x00\x07               column=cf:timestamp, timestamp=1467290788229, value=2015-02-10T14:43.05.00320Z
        \x00\x00\x00\x08               column=cf:temperature, timestamp=1467290788336, value=\x00\x00\x00\x16
        \x00\x00\x00\x08               column=cf:timestamp, timestamp=1467290788336, value=2015-02-10T14:43.05.00320Z
        \x00\x00\x00\x09               column=cf:temperature, timestamp=1467290788246, value=\x00\x00\x001
        \x00\x00\x00\x09               column=cf:timestamp, timestamp=1467290788246, value=2015-02-10T14:43.05.00320Z
        10 row(s) in 0.1800 seconds

    > [AZURE.NOTE] 此掃描作業最多只會從資料表傳回 10 個資料列。

## 刪除叢集

[AZURE.INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

若要一次刪除叢集、儲存體和 Web 應用程式，請刪除包含它們的資源群組。

## 後續步驟

您現在已了解如何使用 Storm 來讀取事件中樞的資料、將資料儲存在 HBase 中，以及使用 Socket.io 和 D3.js 在外部儀表板上視覺化此資訊。

* 如需 Storm 拓撲搭配使用 HDInsight 的更多範例，請參閱：

    * [Storm on HDInsight 的範例拓撲](hdinsight-storm-example-topology.md)

* 如需關於 Apache Storm 的詳細資訊，請參閱 [Apache Storm](https://storm.incubator.apache.org/) 網站 (英文)。

* 如需關於 HBase on HDInsight 的詳細資訊，請參閱 [HBase 與 HDInsight 概觀](hdinsight-hbase-overview.md)。

* 如需關於 Socket.io 的詳細資訊，請參閱 [socket.io](http://socket.io/) 網站 (英文)。

* 如需關於 D3.js 的詳細資訊，請參閱 [D3.js：資料驅動型文件](http://d3js.org/) (英文)。

* 如需關於以 Java 建立拓撲的詳細資訊，請參閱[為 Apache Storm on HDInsight 開發 Java 拓撲](hdinsight-storm-develop-java-topology.md)。

* 如需關於以 .NET 建立拓撲的詳細資訊，請參閱[使用 Visual Studio 為 Apache Storm on HDInsight 開發 C# 拓撲](hdinsight-storm-develop-csharp-visual-studio-topology.md)。

[azure-portal]: https://portal.azure.com

<!---HONumber=AcomDC_0921_2016-->