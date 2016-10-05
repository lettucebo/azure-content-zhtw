<properties
   pageTitle="使用 Apache Ambari REST API 監視和管理 HDInsight 叢集 | Microsoft Azure"
   description="了解如何使用 Ambari 來監視和管理以 Linux 為基礎的 HDInsight 叢集。在本文件中，您將學習如何使用 HDInsight 叢集隨附的 Ambari REST API。"
   services="hdinsight"
   documentationCenter=""
   authors="Blackmist"
   manager="jhubbard"
   editor="cgronlun"
	tags="azure-portal"/>

<tags
   ms.service="hdinsight"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="big-data"
   ms.date="09/20/2016"
   ms.author="larryfr"/>

#使用 Ambari REST API 管理 HDInsight 叢集

[AZURE.INCLUDE [ambari-selector](../../includes/hdinsight-ambari-selector.md)]

Apache Ambari 提供容易使用的 Web UI 和 REST API，可簡化 Hadoop 叢集的管理和監視。以 Linux 為基礎的 HDInsight 叢集上有 Ambari，用來監視叢集並進行組態變更。在本文件中，您會了解使用 cURL 執行一般工作來處理 Ambari REST API 的基本概念。

##必要條件

* [cURL](http://curl.haxx.se/)：cURL 是跨平台公用程式，可以用來從命令列使用 REST API。在本文件中，它用來與 Ambari REST API 進行通訊。
* [jq](https://stedolan.github.io/jq/)：jq 是跨平台命令列公用程式，可與 JSON 文件搭配使用。在這份文件中，它用來剖析從 Ambari REST API 傳回的 JSON 文件。
* [Azure CLI](../xplat-cli-install.md)：跨平台命令列公用程式，可與 Azure 服務搭配使用。

    [AZURE.INCLUDE [use-latest-version](../../includes/hdinsight-use-latest-cli.md)]

##<a id="whatis"></a>什麼是 Ambari？

[Apache Ambari](http://ambari.apache.org) 提供簡單易用的 Web UI，以供用來佈建、管理及監視 Hadoop 叢集，讓 Hadoop 管理起來更為簡單。開發人員可以使用 [Ambari REST API](https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md) 將這些功能整合到應用程式。

以 Linux 為基礎的 HDInsight 叢集預設會提供 Ambari。

##REST API

在 HDInsight 上 Ambari REST API 的基礎 URI 是 https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME，其中 __CLUSTERNAME__ 是叢集的名稱。

> [AZURE.IMPORTANT] URI (CLUSTERNAME.azurehdinsight.net) 完整網域名稱 (FQDN) 部分中的叢集名稱區分大小寫，URI 中的其他部分也區分大小寫。例如，如果您的叢集名稱為 MyCluster，有效的 URI 如下：
>
> `https://mycluster.azurehdinsight.net/api/v1/clusters/MyCluster` `https://MyCluster.azurehdinsight.net/api/v1/clusters/MyCluster`
>
> 下列 URI 會傳回錯誤，因為名稱的第二個項目不是正確的大小寫。
>
> `https://mycluster.azurehdinsight.net/api/v1/clusters/mycluster` `https://MyCluster.azurehdinsight.net/api/v1/clusters/mycluster`

連線到 HDInsight 上的 Ambari 需要 HTTPS。在驗證連線時，您必須使用系統管理員帳戶名稱 (預設值是 __admin__) 和建立叢集時所提供的密碼。

下列範例使用 cURL 來對 REST API 提出 GET 要求。將 __PASSWORD__ 取代為叢集的系統管理員密碼。將 __CLUSTERNAME__ 取代為叢集的名稱：

    curl -u admin:PASSWORD -G "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME"
    
其回應是開頭資訊類似下列內容的 JSON 文件︰

    {
    "href" : "http://10.0.0.10:8080/api/v1/clusters/CLUSTERNAME",
    "Clusters" : {
        "cluster_id" : 2,
        "cluster_name" : "CLUSTERNAME",
        "health_report" : {
        "Host/stale_config" : 0,
        "Host/maintenance_state" : 0,
        "Host/host_state/HEALTHY" : 7,
        "Host/host_state/UNHEALTHY" : 0,
        "Host/host_state/HEARTBEAT_LOST" : 0,
        "Host/host_state/INIT" : 0,
        "Host/host_status/HEALTHY" : 7,
        "Host/host_status/UNHEALTHY" : 0,
        "Host/host_status/UNKNOWN" : 0,
        "Host/host_status/ALERT" : 0

由於這是 JSON，使用 JSON 剖析器來處理資料會較為容易。例如，下列範例使用 jq 來只顯示 `health_report` 元素。

    curl -u admin:PASSWORD -G "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME" | jq '.Clusters.health_report'

##範例：取得叢集節點的 FQDN

使用 HDInsight 時，您可能需要知道叢集節點的完整網域名稱 (FQDN)。您可以使用下列項目，輕鬆地擷取叢集中不同節點的 FQDN：

* __前端節點__：`curl -u admin:PASSWORD -G "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/services/HDFS/components/NAMENODE" | jq '.host_components[].HostRoles.host_name'`
* __背景工作節點__：`curl -u admin:PASSWORD -G "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/services/HDFS/components/DATANODE" | jq '.host_components[].HostRoles.host_name'`
* __Zookeeper 節點__：`curl -u admin:PASSWORD -G "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/services/ZOOKEEPER/components/ZOOKEEPER_SERVER" | jq '.host_components[].HostRoles.host_name'`

請注意，上述每個範例都遵循以下相同模式︰

1. 對我們知道是在這些節點上執行的元件進行查詢。
2. 擷取 `host_name` 元素，其包含這些節點的 FQDN。

傳回文件的 `host_components` 元素包含多個項目。使用 `.host_components[]`，然後指定元素內的路徑，將對每個項目執行迴圈並從特定路徑提取值。如果您只想要一個值，例如第一個 FQDN 項目，您可以以集合形式傳回項目，然後選取特定項目：

    jq '[.host_components[].HostRoles.host_name][0]'

這會傳回來自集合的第一個 FQDN。

##範例：取得預設儲存體帳戶和容器

建立 HDInsight 叢集時，您必須使用 Azure 儲存體帳戶和 Blob 容器做為叢集的預設儲存體。在建立叢集之後，您可以使用 Ambari 來擷取這項資訊。例如，如果您想要以程式設計方式將資料直接寫入容器。

以下將擷取叢集預設儲存體的 WASB URI：

    curl -u admin:PASSWORD -G "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/configurations/service_config_versions?service_name=HDFS&service_config_version=1" | jq '.items[].configurations[].properties["fs.defaultFS"] | select(. != null)'
    
> [AZURE.NOTE] 這個方法會傳回套用至伺服器的第一個組態 (`service_config_version=1`)，其包含這項資訊。如果您擷取在叢集建立後已修改過的值，您可能需要列出組態版本並擷取最新的版本。

這會傳回類似下列範例的值，其中 __CONTAINER__ 為預設容器和 __ACCOUNTNAME__ 是 Azure 儲存體帳戶名稱：

    wasbs://CONTAINER@ACCOUNTNAME.blob.core.windows.net

之後您可以使用這項資訊搭配 [Azure CLI](../xplat-cli-install.md) 來上傳或從容器下載資料。

1. 取得儲存體帳戶的資源群組。將 __ACCOUNTNAME__ 取代為從 Ambari 擷取的儲存體帳戶名稱：

        azure storage account list --json | jq '.[] | select(.name=="ACCOUNTNAME").resourceGroup'
    
    這會傳回帳戶的資源群組名稱。
    
    > [AZURE.NOTE] 如果此命令未傳回任何項目，您需要將 Azure CLI 變更為 Azure 資源管理員模式，然後再重新執行命令。若要切換至 Azure Resource Manager 模式，請使用下列命令：
    >
    > `azure config mode arm`
    
2. 取得儲存體帳戶的金鑰。將 __GROUPNAME__ 取代為上一個步驟的資源群組。將 __ACCOUNTNAME__ 取代為儲存體帳戶名稱：

        azure storage account keys list -g GROUPNAME ACCOUNTNAME --json | jq '.storageAccountKeys.key1'

    此範例會傳回帳戶的主要金鑰。
    
3. 使用上傳命令來將檔案儲存在容器：

        azure storage blob upload -a ACCOUNTNAME -k ACCOUNTKEY -f FILEPATH --container __CONTAINER__ -b BLOBPATH
        
    將 __ACCOUNTNAME__ 取代為儲存體帳戶名稱。將 __ACCOUNTKEY__ 取代為先前擷取的索引鍵。__FILEPATH__ 是您想要上傳的檔案的路徑，而 __BLOBPATH__ 是在容器中的路徑。

    例如，如果您想要檔案出現在 HDInsight 中的 wasbs://example/data/filename.txt，則 __BLOBPATH__ 會是 `example/data/filename.txt`。

##範例︰更新 Ambari 組態

1. 取得目前的組態，Ambari 會將其儲存為「所需的組態」:

        curl -u admin:PASSWORD -G "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME?fields=Clusters/desired_configs"
        
    此範例會傳回 JSON 文件，其中包含叢集上安裝之元件的目前組態 (由「tag」值識別)。下列範例是從 Spark 叢集類型傳回之資料的摘要。
    
        "spark-metrics-properties" : {
            "tag" : "INITIAL",
            "user" : "admin",
            "version" : 1
        },
        "spark-thrift-fairscheduler" : {
            "tag" : "INITIAL",
            "user" : "admin",
            "version" : 1
        },
        "spark-thrift-sparkconf" : {
            "tag" : "INITIAL",
            "user" : "admin",
            "version" : 1
        }

    您需要從此清單中複製元件的名稱 (例如，__spark\_thrift\_sparkconf__ 和 __tag__ 值。
    
2. 使用下列命令以擷取元件和標記的組態。將 __spark-thrift-sparkconf__ 和 __INITIAL__ 取代為您想要擷取其組態的元件和標籤。

        curl -u admin:PASSWORD -G "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/configurations?type=spark-thrift-sparkconf&tag=INITIAL" | jq --arg newtag $(echo version$(date +%s%N)) '.items[] | del(.href, .version, .Config) | .tag |= $newtag | {"Clusters": {"desired_config": .}}' > newconfig.json
    
    Curl 擷取 JSON 文件，然後使用 jq 來對資料進行修改以便建立範本。範本則可用來新增/修改組態值。它特別具有下列功能：
    
    * 建立唯一的值，其中包含字串 "version" 和日期，會儲存在 __newtag__。
    * 建立新的所需組態的根文件。
    * 取得 `.items[]` 陣列的內容，並且新增在 __desired\_config__ 元素下。
    * 刪除 __href__、__version__ 和 __Config__ 元素，因為提交新組態時不需要這些元素。
    * 新增 __tag__ 元素，並將其值設定為 __version#################__。數字部分是根據目前的日期。每個組態都必須有唯一的標記。
    
    最後，將資料儲存至 __newconfig.json__ 文件。此文件結構應會顯示為類似下列範例：
    
        {
            "Clusters": {
                "desired_config": {
                "tag": "version1459260185774265400",
                "type": "spark-thrift-sparkconf",
                "properties": {
                    ....
                 },
                 "properties_attributes": {
                     ....
                 }
            }
        }

3. 開啟 __newconfig.json__ 文件並且在 __properties__ 物件中修改/加入值。下列範例將 __"spark.yarn.am.memory"__ 的值從 __"1g"__ 變更為 __"3g"__，並且針對 __"spark.kryoserializer.buffer.max"__ 以值 __"256m"__ 加入新元素。

        "spark.yarn.am.memory": "3g",
        "spark.kyroserializer.buffer.max": "256m",

    完成修改之後，請儲存檔案。

4. 使用以下命令將更新的組態提交至 Ambari。

        cat newconfig.json | curl -u admin:PASSWORD -H "X-Requested-By: ambari" -X PUT -d "@-" "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME"
        
    此命令使用管線將 __newconfig.json__ 檔案的內容傳送至 CURL 要求，後者會將它提交至叢集中作為所需的新組態。cURL 要求會傳回 JSON 文件。這份文件中的 __versionTag__ 元素應符合您所提交的版本，__configs__ 物件將會包含您所要求的組態變更。

###範例︰重新啟動服務元件

此時，如果您看一下 Ambari Web UI，Spark 服務就會指出它需要重新啟動，新組態才會生效。使用下列步驟重新啟動服務。

1. 使用以下命令以啟用 Spark 服務的維護模式：

        echo '{"RequestInfo": {"context": "turning on maintenance mode for SPARK"},"Body": {"ServiceInfo": {"maintenance_state":"ON"}}}' | curl -u admin:PASSWORD -H "X-Requested-By: ambari" -X PUT -d "@-" "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/services/SPARK"

    此命令會將 JSON 文件傳送到開啟維護模式的伺服器 (包含在 `echo` 陳述式)。您可以使用下列要求驗證服務現在處於維護模式中：
    
        curl -u admin:PASSWORD -H "X-Requested-By: ambari" "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/services/SPARK" | jq .ServiceInfo.maintenance_state
        
    這會傳回 `"ON"` 的值。

3. 接下來，使用以下命令以關閉服務：

        echo '{"RequestInfo": {"context" :"Stopping the Spark service"}, "Body": {"ServiceInfo": {"state": "INSTALLED"}}}' | curl -u admin:PASSWORD -H "X-Requested-By: ambari" -X PUT -d "@-" "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/services/SPARK"
        
    此命令會傳回類似以下的回應。
    
        {
            "href" : "http://10.0.0.18:8080/api/v1/clusters/CLUSTERNAME/requests/29",
            "Requests" : {
                "id" : 29,
                "status" : "Accepted"
            }
        }
    
    這個 URI 所傳回的 `href` 值會使用叢集節點的內部 IP 位址。若要從叢集之外使用它，請將 '10.0.0.18:8080' 部分取代為叢集的 FQDN。例如，以下命令會擷取要求的狀態。
    
        curl -u admin:PASSWORD -H "X-Requested-By: ambari" "https://CLUSTERNAME/api/v1/clusters/CLUSTERNAME/requests/29" | jq .Requests.request_status
    
    如果此命令傳回 `"COMPLETED"` 值，則表示要求已完成。

4. 前一個要求完成後，使用以下命令來啟動服務。

        echo '{"RequestInfo": {"context" :"Restarting the Spark service"}, "Body": {"ServiceInfo": {"state": "STARTED"}}}' | curl -u admin:PASSWORD -H "X-Requested-By: ambari" -X PUT -d "@-" "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/services/SPARK"

    在服務重新啟動後，它就會成為新的組態設定。

5. 最後，使用以下命令來關閉維護模式。

        echo '{"RequestInfo": {"context": "turning off maintenance mode for SPARK"},"Body": {"ServiceInfo": {"maintenance_state":"OFF"}}}' | curl -u admin:PASSWORD -H "X-Requested-By: ambari" -X PUT -d "@-" "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/services/SPARK"

##後續步驟

如需 REST API 的完整參考，請參閱 [Ambari API 參考 V1](https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md)。

> [AZURE.NOTE] 某些 Ambari 功能已停用，因為這些功能是由 HDInsight 雲端服務所管理；例如，在叢集中新增或移除主機，或新增服務。

<!---HONumber=AcomDC_0921_2016-->