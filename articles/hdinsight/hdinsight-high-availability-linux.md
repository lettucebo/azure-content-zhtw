<properties
	pageTitle="以 Linux 為基礎的 HDInsight (Hadoop) 高可用性功能 |Microsoft Azure"
	description="了解如何使用額外的前端節點，讓以 Linux 為基礎的 HDInsight 叢集可以提高可靠性和可用性。您將了解這會如何影響例如 Ambari 和 Hive 等 Hadoop 服務，以及如何使用 SSH 分別連線到每個前端節點。"
	services="hdinsight"
	editor="cgronlun"
	manager="jhubbard"
	authors="Blackmist"
	documentationCenter=""
	tags="azure-portal"/>

<tags
	ms.service="hdinsight"
	ms.workload="big-data"
	ms.tgt_pltfrm="na"
	ms.devlang="multiple"
	ms.topic="article"
	ms.date="09/13/2016"
	ms.author="larryfr"/>

#HDInsight 上 Hadoop 叢集的可用性和可靠性

Hadoop 可藉由在叢集中的節點散佈服務和資料的複製檔，達到高可用性和可靠性。不過 Hadoop 的標準散佈功能通常只能有一個前端節點。任何單一前端節點的中斷情況都可能導致叢集停止運作。

若要解決這種潛在問題，在 Azure 上以 Linux 為基礎的 HDInsight 叢集可提供兩個前端節點，以提升 Hadoop 服務和作業執行的可用性和可靠性。

> [AZURE.NOTE] 此文件中使用的步驟是針對以 Linux 為基礎的 HDInsight 叢集。如果您使用以 Windows 為主的叢集，如需 Windows 特定的資訊，請參閱 [HDInsight 上以 Windows 為主的 Hadoop 叢集可用性和可靠性](hdinsight-high-availability.md)。

##了解節點

使用 Azure 虛擬機器在 HDInsight 叢集中的節點實作。當節點失敗時，該節點會離線，並且會建立一個新的節點，來取代失敗的節點。當節點離線時，在新節點上線之前，將會使用相同類型的另一個節點。

> [AZURE.NOTE] 當節點失敗時，如果是正在分析資料，其作業的進度會遺失。失敗節點正在處理的作業將重新提交至另一個節點。

下列各節會討論個別的節點類型所搭配使用的 HDInsight。並非所有的節點類型都可用於某個叢集類型。例如，Hadoop 叢集類型就不會有任何 Nimbus 節點。如需 HDInsight 叢集類型所使用的節點詳細資訊，請參閱[在 HDInsight 建立以 Linux 為基礎的 Hadoop 叢集](hdinsight-hadoop-provision-linux-clusters.md#cluster-types)的〈叢集類型〉一節。

###前端節點

Hadoop 的某些實作具有單一前端節點，其可裝載服務和元件，能順利管理背景工作節點的失敗。但任何在前端節點上執行的主要服務中斷都有可能導致叢集停止工作。

HDInsight 叢集提供次要的前端節點，在主要節點失敗時，可讓主要服務和元件繼續在次要節點上執行。

> [AZURE.IMPORTANT] 這兩個前端節點會同時為作用中和在叢集中執行。某些服務，例如 HDFS 或 YARN，在任何給定的時間僅能在其中一個前端節點上為「作用中」(而在其他前端節點上為「待命」)。HiveServer2 或 Hive MetaStore 等其他服務可同時在這兩個前端節點上作用。

前端節點 (和 HDInsight 中的其他節點) 以數值做為節點主機名稱的一部分。例如，`hn0-CLUSTERNAME` 或 `hn4-CLUSTERNAME`。

> [AZURE.IMPORTANT] 請勿將數值與節點 (不論是主要或次要) 相關聯；數值只是用來為每個節點提供唯一名稱。

###Nimbus 節點

對於 Storm 叢集，Nimbus 節點會透過在背景工作節點之間散佈並監視處理，來提供 Hadoop jobtracker 類似的功能。HDInsight 提供 2 個 Nimbus 節點給 Storm 叢集類型。

###Zookeeper 節點

[ZooKeeper](http://zookeeper.apache.org/) 節點 (ZK) 用於前端節點上主要服務的前置選擇，並可確保服務、資料 (背景工作角色) 節點和閘道器知道主要服務在哪一個前端節點上為作用中。根據預設，HDInsight 會提供 3 個 ZooKeeper 節點。

###背景工作節點

當作業提交至叢集時，背景工作節點會執行實際的資料分析。如果背景工作節點失敗，它所執行的工作將會提交至另一個背景工作節點。根據預設，HDInsight 會建立 4 個背景工作節點。不過，您可以視需要在叢集建立期間和在叢集建立之後變更該數字。

###邊緣節點

邊緣節點並不積極地參與叢集內的資料分析，而是開發人員或資料科學家在使用 Hadoop 時搭配使用。邊緣節點和叢集中的其他節點一樣，存在於相同的 Azure 虛擬網路中，也可直接存取其他所有節點。由於邊緣節點不涉及叢集資料的分析，因此其可以從關鍵 Hadoop 服務或分析作業中取走資源，而不會產生任何問題。

目前，HDInsight 上的 R 伺服器是唯一可依預設提供邊緣節點的叢集類型。對於 HDInsight 上的 R 伺服器來說，邊緣節點是用來在提交至叢集進行分散式處理之前，在本機節點上測試 R 程式碼之用。

[建立以 Linux 為基礎，且 Hue 在邊緣節點上的 HDInsight 叢集](https://azure.microsoft.com/documentation/templates/hdinsight-linux-with-hue-on-edge-node/)即是可用來建立具有邊緣節點的 Hadoop 叢集類型的範例範本。


## 存取節點

經由公用閘道器提供透過網際網路存取叢集，並僅限於連接至前端節點 (若 R 伺服器是在 HDInsight 叢集上) 和邊緣節點。由於公用閘道器會將要求路由至裝載所要求服務的前端節點，因此在前端節點上執行服務的存取不會受到具多個前端節點的影響。比方說，如果 Ambari 目前裝載在次要前端節點上，閘道器會將 Ambari 收到的要求路由至該節點。

使用 SSH 存取叢集時，透過連接埠 22 (SSH 的預設值) 的連接將會連接到主要前端節點；透過連接埠 23 的連接將會連接到次要前端節點。例如，`ssh username@mycluster-ssh.azurehdinsight.net` 會連線到名為 __mycluster__ 之叢集的主要前端節點。

> [AZURE.NOTE] 這也適用於根據 SSH 的通訊協定，例如 SSH 檔案傳輸通訊協定 (SFTP)。

在 HDInsight 叢集上 R 伺服器提供的邊緣節點也可透過連接埠 22 使用 SSH 直接存取。例如，`ssh username@RServer.mycluster.ssh.azurehdinsight.net` 會連接到 R 伺服器 (於名為 __mycluster__ 的 HDInsight 叢集上) 的邊緣節點。

### 內部完整網域名稱 (FQDN)

HDInsight 叢集中的節點具有內部 IP 位址和 FQDN，這些只能自叢集存取 (例如連接到前端節點的 SSH 工作階段或叢集上執行的作業。) 使用內部 FQDN 或 IP 位址存取叢集上的服務時，您應該使用 Ambari 來驗證要存取服務時所使用的 IP 或 FQDN。

例如 Oozie 服務只能在一個前端節點上執行，而且從 SSH 工作階段使用 `oozie` 命令需要有該服務的 URL。這個可以使用下列命令從 Ambari 擷取：

	curl -u admin:PASSWORD "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/configurations?type=oozie-site&tag=TOPOLOGY_RESOLVED" | grep oozie.base.url

這會傳回如下的值，其中包含要使用 `oozie` 命令的內部 URL：

	"oozie.base.url": "http://hn0-CLUSTERNAME-randomcharacters.cx.internal.cloudapp.net:11000/oozie"

### 存取其他節點類型

您可以使用下列方法連線到並非直接透過網際網路存取的節點。

* __SSH__︰一旦使用 SSH 連接到前端節點，您便可以使用來自前端節點的 SSH 來連接至叢集中的其他節點。
* __SSH 通道__︰如果您需要存取裝載在其中一個節點上的 Web 服務，且該服務不會在網際網路上公開，您就必須[使用 SSH 通道](hdinsight-linux-ambari-ssh-tunnel.md)。
* __Azure 虛擬網路__：如果您的 HDInsight 叢集是 Azure 虛擬網路的一部分，則任何在相同虛擬網路上的資源都可以直接存取叢集內所有的節點。

## 如何檢查服務狀態

Ambari Web UI 或 Ambari REST API 都可用來檢查在前端節點執行的服務狀態。

###Ambari Web UI

Ambari Web UI 可在 https://CLUSTERNAME.azurehdinsight.net 檢視。將 **CLUSTERNAME** 取代為您叢集的名稱。如果出現提示，請輸入叢集的 HTTP 使用者認證。預設 HTTP 使用者名稱為 **admin** ，密碼是您在建立叢集時輸入的密碼。

當您來到 [Ambari] 頁面上時，會在該頁面的左邊列出已安裝的服務。

![已安裝的服務](./media/hdinsight-high-availability-linux/services.png)

服務旁可能會出現一系列圖示以表示狀態。可以使用在頁面頂端的 [警示] 連結，檢視與服務相關的任何警示。您可以選取每個服務來檢視其詳細資訊。

雖然 [服務] 頁面會提供資訊之狀態和每個服務的組態，但並不提供該服務正在哪些前端節點上執行的資訊。若要檢視這項資訊，請使用在頁面頂端的 [主機] 連結。這會顯示叢集內的主機，包括前端節點在內。

![主機清單](./media/hdinsight-high-availability-linux/hosts.png)

選取其中一個前端節點的連結將會顯示該節點上執行的服務與元件。

![元件狀態](./media/hdinsight-high-availability-linux/nodeservices.png)

###Ambari REST API

Ambari REST API 可透過網際網路提供，而且公用閘道器會處理路由要求，將其傳送到目前裝載 REST API 的前端節點。

您可以使用下列命令透過 Ambari REST API 來檢查服務的狀態：

	curl -u admin:PASSWORD https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/services/SERVICENAME?fields=ServiceInfo/state

* 將 **PASSWORD** 取代為 HTTP 使用者 (系統管理員) 帳戶密碼。

* 將 **CLUSTERNAME** 取代為叢集的名稱。

* 將 **SERVICENAME** 取代為要檢查其狀態的服務名稱

例如，若要檢查在名為 **mycluster** 的叢集上的 **HDFS** 服務狀態，其中密碼是 **password**，您可使用下列命令：

	curl -u admin:password https://mycluster.azurehdinsight.net/api/v1/clusters/mycluster/services/HDFS?fields=ServiceInfo/state

回應將如下所示：

	{
	  "href" : "http://hn0-CLUSTERNAME.randomcharacters.cx.internal.cloudapp.net:8080/api/v1/clusters/mycluster/services/HDFS?fields=ServiceInfo/state",
	  "ServiceInfo" : {
	    "cluster_name" : "mycluster",
	    "service_name" : "HDFS",
	    "state" : "STARTED"
	  }
	}

該 URL 會告訴我們服務目前正在名為 __hn0-CLUSTERNAME__ 的前端節點上執行。

該狀態會告訴我們此服務目前正在執行，或**已啟動**。

如果您不知道有哪些服務安裝在該叢集上，您可以使用下列命令擷取清單：

	curl -u admin:PASSWORD https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/services

####服務元件

服務可能包含您想要個別檢查狀態的元件。例如，HDFS 包含 NameNode 元件。若要檢視在元件上的資訊，該命令為：

	curl -u admin:PASSWORD https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/services/SERVICE/components/component

如果您不知道服務提供了哪些元件，您可以使用下列命令擷取清單：

	curl -u admin:PASSWORD https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/services/SERVICE/components/component
    
## 如何存取前端節點上的記錄檔

###SSH

透過 SSH 連接至前端節點時，可以在 **/var/log** 找到記錄檔。例如，**/var/log/hadoop-yarn/yarn** 包含 YARN 的記錄檔。

每個前端節點可以有唯一的記錄項目，所以您應該檢查兩者的記錄檔。

###SFTP

您也可以使用 SSH 檔案傳輸通訊協定或安全檔案傳輸通訊協定 (SFTP)，來連線至前端節點，並直接下載記錄檔。

類似於使用 SSH 用戶端，當連接到叢集時，您必須提供 SSH 的使用者帳戶名稱和叢集的 SSH 位址。例如，`sftp username@mycluster-ssh.azurehdinsight.net`。您也必須在出現提示時，提供帳戶密碼或使用 `-i` 參數提供公用金鑰。

連線之後，您會看到 `sftp>` 提示。您可以從該提示變更目錄、上傳和下載檔案。例如：下列命令會將目錄變更至 **/var/log/hadoop/hdfs** 目錄，然後在目錄中下載所有檔案。

    cd /var/log/hadoop/hdfs
    get *

如需可用命令清單，請在 `sftp>` 提示中輸入 `help`。

> [AZURE.NOTE] 使用 SFTP 連線時，也提供圖形化介面可讓您以視覺化方式檢視檔案系統。例如：[MobaXTerm](http://mobaxterm.mobatek.net/) 可讓您使用類似於「Windows 檔案總管」的介面瀏覽檔案系統。


###Ambari

> [AZURE.NOTE] 因為個別服務的網站不會在網際網路上公開，所以透過 Ambari 存取記錄檔需要 SSH 通道。如需使用 SSH 通道的相關資訊，請參閱[使用 SSH 通道來存取 Ambari Web UI、ResourceManager、JobHistory、NameNode、Oozie 及其他 Web UI](hdinsight-linux-ambari-ssh-tunnel.md)。

從 Ambari Web UI 中，選取您想要檢視記錄檔的服務 (例如 YARN)，然後使用 [快速連結] 選取要檢視哪一個前端節點的記錄檔。

![使用快速連結檢視記錄檔](./media/hdinsight-high-availability-linux/viewlogs.png)

## 如何設定節點大小 ##

只能在叢集建立期間選取節點的大小。您可以在 [HDInsight 定價頁面](https://azure.microsoft.com/pricing/details/hdinsight/)找到 HDInsight 可用之不同 VM 大小的清單，包括個別的核心、記憶體和本機儲存體。

建立新的叢集時，您可以指定該節點的大小。下列提供如何使用 [Azure 入口網站][preview-portal]、[Azure PowerShell][azure-powershell] 和 [Azure CLI][azure-cli] 指定大小的相關資訊：

* **Azure 入口網站**：建立新叢集時，會提供您設定叢集前端、背景工作和 ZooKeeper 節點大小的選項 (定價層)：

	![可選取節點大小的 [叢集映像建立精靈]](./media/hdinsight-high-availability-linux/headnodesize.png)

* **Azure CLI**︰當使用 `azure hdinsight cluster create` 命令時，您可以使用 `--headNodeSize`、`--workerNodeSize` 和 `--zookeeperNodeSize` 參數設定前端、 背景工作與 ZooKeeper 節點的大小。

* **Azure PowerShell**︰當使用 `New-AzureRmHDInsightCluster` Cmdlet時，您可以使用 `-HeadNodeVMSize`、`-WorkerNodeSize` 和 `-ZookeeperNodeSize` 參數設定前端、背景工作與 ZooKeeper 節點的大小。

##後續步驟

在本文件中，您已了解到 Azure HDInsight 如何讓 Hadoop 更加可用。若要深入了解這份文件中所敘述的項目，請使用下列連結。

- [Ambari REST 參考](https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md)

- [安裝和設定 Azure CLI](../xplat-cli-install.md)

- [安裝並設定 Azure PowerShell](../powershell-install-configure.md)

- [使用 Ambari 管理 HDInsight](hdinsight-hadoop-manage-ambari.md)

- [佈建以 Linux 為基礎的 HDInsight 叢集](hdinsight-hadoop-provision-linux-clusters.md)

[preview-portal]: https://portal.azure.com/
[azure-powershell]: ../powershell-install-configure.md
[azure-cli]: ../xplat-cli-install.md

<!---HONumber=AcomDC_0921_2016-->