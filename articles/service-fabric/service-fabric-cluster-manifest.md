<properties
   pageTitle="設定獨立叢集 |Microsoft Azure"
   description="本文說明如何設定獨立或私人的 Service Fabric 叢集。"
   services="service-fabric"
   documentationCenter=".net"
   authors="dsk-2015"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="06/23/2016"
   ms.author="dkshir"/>


# 獨立 Windows 叢集的組態設定

本文說明如何使用 **ClusterConfig.JSON** 檔案來設定獨立 Service Fabric 叢集。當您[下載獨立 Service Fabric 封裝](service-fabric-cluster-creation-for-windows-server.md#downloadpackage)時，即會將這個檔案下載至您的工作電腦。ClusterConfig.JSON 檔案可讓您針對 Service Fabric 叢集指定如下的資訊：Service Fabric 節點及其 IP 位址、叢集上不同類型的節點、安全性組態，以及關於容錯/升級網域的網路拓撲。

我們後續將會檢查此檔案的不同區段。

## 一般叢集組態
這涵蓋廣泛的叢集特定組態，如以下 JSON 片段所示。

    "name": "SampleCluster",
    "clusterConfigurationVersion": "1.0.0",
    "apiVersion": "2016-09-26",

若要為 Service Fabric 叢集提供任何易記名稱，您可以將它指派給 **name** 變數。您可以根據安裝程式來變更 **clusterManifestVersion**；必須先更新它，才能升級 Service Fabric 組態。您可以保留 **apiVersion** 設為預設值。


<a id="clusternodes"></a>
## 叢集上的節點
您可以使用 **nodes** 區段，在 Service Fabric 叢集上設定節點，如下列程式碼片段所示。

    "nodes": [{
        "nodeName": "vm0",
        "iPAddress": "localhost",
        "nodeTypeRef": "NodeType0",
        "faultDomain": "fd:/dc1/r0",
        "upgradeDomain": "UD0"
    }, {
        "nodeName": "vm1",
        "iPAddress": "localhost",
        "nodeTypeRef": "NodeType1",
        "faultDomain": "fd:/dc1/r1",
        "upgradeDomain": "UD1"
    }, {
        "nodeName": "vm2",
        "iPAddress": "localhost",
        "nodeTypeRef": "NodeType2",
        "faultDomain": "fd:/dc1/r2",
        "upgradeDomain": "UD2"
    }],

Service Fabric 叢集至少需要 3 個節點。您可以根據安裝程式，在此區段中新增更多節點。下表說明每個節點的組態設定。

|**節點組態**|**說明**|
|-----------------------|--------------------------|
|nodeName|您可以為節點提供易記名稱。|
|iPAddress|開啟命令視窗並輸入 `ipconfig`，以找出節點的 IP 位址。記下 IPV4 位址，並將它指派給 **iPAddress** 變數。|
|nodeTypeRef|每個節點都可以指派不同的節點類型。[節點類型](#nodetypes)將於下一節中定義。|
|faultDomain|容錯網域可讓叢集管理員定義實體節點，這類節點可能會在相同時間因為共用實體的相依性 (例如電源和網路來源) 而發生錯誤。|
|upgradeDomain|升級網域說明大約會在相同時間關閉以進行 Service Fabric 升級的節點集。因為它們不會受到任何實體需求所限制，您可以選擇要將哪些節點指派給哪些升級網域。| 


## 叢集 **properties**

ClusterConfig.JSON 中的 **properties** 區段用來設定叢集，如下所示。

### **diagnosticsStore**
您可以使用 **diagnosticsStore** 區段來設定參數，以啟用診斷以及疑難排解節點與叢集的失敗，如下列程式碼片段所示。

    "diagnosticsStore": {
        "metadata":  "Please replace the diagnostics store with an actual file share accessible from all cluster machines.",
        "dataDeletionAgeInDays": "7",
        "storeType": "FileShare",
        "IsEncrypted": "false",
        "connectionstring": "c:\\ProgramData\\SF\\DiagnosticsStore"
    }

**metadata** 是叢集診斷的說明，而且可根據您的安裝程式來設定。這些變數有助於收集 ETW 追蹤記錄檔、損毀傾印，以及效能計數器。如需 ETW 追蹤記錄檔的詳細資訊，請參閱 [Tracelog](https://msdn.microsoft.com/library/windows/hardware/ff552994.aspx) 和 [ETW 追蹤](https://msdn.microsoft.com/library/ms751538.aspx)。包含[損毀傾印](https://blogs.technet.microsoft.com/askperf/2008/01/08/understanding-crash-dump-files/)和[效能計數器](https://msdn.microsoft.com/library/windows/desktop/aa373083.aspx)的所有記錄檔可導向至電腦上的 **connectionString** 資料夾。您也可以使用 **AzureStorage** 儲存診斷。請參閱下面的範例程式碼片段。

	"diagnosticsStore": {
        "metadata":  "Please replace the diagnostics store with an actual file share accessible from all cluster machines.",
        "dataDeletionAgeInDays": "7",
        "storeType": "AzureStorage",
        "IsEncrypted": "false",
        "connectionstring": "xstore:DefaultEndpointsProtocol=https;AccountName=[AzureAccountName];AccountKey=[AzureAccountKey]"
    }

### **security** 
**security** 區段對於安全獨立的 Service Fabric 叢集是必要的項目。下列程式碼片段示範此區段的一部分。

    "security": {
        "metadata": "This cluster is secured using X509 certificates.",
        "ClusterCredentialType": "X509",
        "ServerCredentialType": "X509",
		. . .
	}

**metadata** 是安全叢集的說明，而且可根據您的安裝程式來設定。**ClusterCredentialType** 和 **ServerCredentialType** 決定叢集和節點將實作的安全性類型。如果是憑證式安全性，可設定為 *X509*，如果是以 Azure Active Directory 為基礎的安全性，可設定為 *Windows*。其餘的 **security** 區段則是根據安全性類型。如需如何填滿其餘 **security** 區段的相關資訊，請參閱[獨立叢集中的憑證式安全性](service-fabric-windows-cluster-x509-security.md)或[獨立叢集中的 Windows 安全性](service-fabric-windows-cluster-windows-security.md)。

### **reliabilityLevel**
**reliabilityLevel** 定義可以在叢集主要節點上執行的系統服務複本數目。這會增加這些服務以及叢集的可靠性。您可以針對這些服務的 3、5、7 或 9 個複本，將此變數分別設定為 *Bronze*、*Silver*、*Gold* 或 *Platinum*。請參閱下列範例。

	"reliabilityLevel": "Bronze",
	
請注意，由於主要節點執行的是系統服務的單一複本，所以 *Bronze* 可靠性層級至少要有 3 個主要節點、*Silver* 可靠性層級至少要有 5 個主要節點、*Gold* 可靠性層級至少要有 7 個主要節點，*Platinum* 可靠性層級至少要有 9 個主要節點。


<a id="nodetypes"></a>
### **nodeTypes**
**nodeTypes** 區段說明叢集所擁有的節點類型。至少必須針對叢集指定一個節點類型，如下列程式碼片段所示。

	"nodeTypes": [{
        "name": "NodeType0",
        "clientConnectionEndpointPort": "19000",
        "clusterConnectionEndpoint": "19001",
        "httpGatewayEndpointPort": "19080",
        "applicationPorts": {
			"startPort": "20001",
            "endPort": "20031"
        },
        "ephemeralPorts": {
            "startPort": "20032",
            "endPort": "20062"
        },
        "isPrimary": true
    }]

**name** 是此特定節點類型的易記名稱。若要建立此節點類型的節點，您必須將此節點類型的易記名稱指定為該節點的 **nodeTypeRef** 變數，如先前[叢集上的節點](#clusternodes)一節中所述。針對每個節點類型，您可以定義用於連接到此叢集的各種端點。您可以為這些連接端點選擇任意的連接埠號碼，只要它們不會與此叢集中的任何其他端點發生衝突即可。在包含多個節點類型的叢集中，將會有一個主要節點類型，其中已將 **isPrimary** 設定為 *true*。其餘的節點會將 **isPrimary** 設定為 *false*。如需根據您的叢集容量設定 **nodeTypes** 和 **reliabilityLevel** 值的詳細資訊，以及了解主要和非主要節點類型之間的差異，請參閱 [Service Fabric 叢集容量規劃考量](service-fabric-cluster-capacity.md)。


### **fabricSettings**
此區段可讓您設定 Service Fabric 資料和記錄檔的根目錄。您只能在初始叢集建立期間自訂這些項目。如需這個區段的範例程式碼片段，請參閱下列內容。

    "fabricSettings": [{
        "name": "Setup",
        "parameters": [{
            "name": "FabricDataRoot",
            "value": "C:\ProgramData\SF"
        }, {
            "name": "FabricLogRoot",
            "value": "C:\ProgramData\SF\Log"
    }]

建議您使用非作業系統磁碟機做為 FabricDataRoot 和 FabricLogRoot，因為這類磁碟機比較不會讓作業系統當機。請注意，如果您只自訂資料根目錄，則記錄根目錄將會以資料根目錄的下一個層級來取代。


## 後續步驟

當您根據獨立叢集安裝程式設定完整的 ClusterConfig.JSON 檔案之後，就可以遵循[在內部部署或雲端建立 Azure Service Fabric 叢集](service-fabric-cluster-creation-for-windows-server.md)一文來部署叢集，然後繼續[使用 Service Fabric Explorer 視覺化叢集](service-fabric-visualizing-your-cluster.md)。

<!---HONumber=AcomDC_0921_2016-->