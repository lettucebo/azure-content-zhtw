<properties
   pageTitle="建立和管理獨立 Azure Service Fabric 叢集 | Microsoft Azure"
   description="了解如何在執行 Windows Server (無論是在內部部署或任何雲端) 的任何電腦 (實體或虛擬) 上建立和管理 Azure Service Fabric 叢集。"
   services="service-fabric"
   documentationCenter=".net"
   authors="ChackDan"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="09/15/2016"
   ms.author="dkshir;chackdan"/>


# 建立和管理在 Windows Server 上執行的叢集

Azure Service Fabric 可允許在執行 Windows Server 的任何虛擬機器或電腦上建立 Service Fabric 叢集。這表示您能夠在包含一組互連式 Windows Server 電腦的任何環境中部署和執行 Service Fabric 應用程式，不論該環境是內部部署或是透過任何雲端提供者來提供。Service Fabric 會提供一個安裝封裝來建立稱為獨立 Windows Server 封裝的 Service Fabric 叢集。

本文將逐步引導您使用 Service Fabric 內部部署 (但可針對其他任何環境輕鬆地調整，例如其他雲端提供者) 的獨立封裝完成建立叢集的步驟。

>[AZURE.NOTE] 此獨立的 Windows Server 封裝可能包含一些目前為預覽版的功能，並不支援商業用途。若要查看預覽功能清單，請向下捲動到本文件的結尾。如果您想要立即下載一份 EULA，[按一下這裡](http://go.microsoft.com/fwlink/?LinkID=733084)。

<a id="downloadpackage"></a>
## 下載 Service Fabric 獨立封裝


[下載適用於 Windows Server 2012 R2 和更新版本的 Service Fabric 獨立封裝](http://go.microsoft.com/fwlink/?LinkId=730690)，名稱為「Microsoft.Azure.ServiceFabric.WindowsServer.&lt;version&gt;.zip」。

>[AZURE.NOTE] 如果您使用 Internet Explorer 或 Microsoft Edge 瀏覽器下載此封裝，必須將 [http://download.microsoft.com](http://download.microsoft.com) 網站加入內部網路中信任的網站，防止區域標籤在 zip 下載之後被寫入檔案

 ![TustedZone][TrustedZone]

在下載封裝中，您會找到下列檔案︰

|**檔案名稱**|**簡短說明**|
|-----------------------|--------------------------|
|MicrosoftAzureServiceFabric.cab|CAB 檔，其中包含部署到叢集中每部電腦的二進位檔。|
|ClusterConfig.Unsecure.DevCluster.json|叢集組態範例檔案，其中包含適用於不安全、三個節點、單一 VM/電腦部署叢集的設定，包括叢集中每個節點的相關資訊。 |
|ClusterConfig.Unsecure.MultiMachine.json|叢集組態範例檔案，其中包含適用於不安全、多個 VM/電腦叢集的設定，包括叢集中每個電腦的相關資訊。|
|ClusterConfig.Windows.DevCluster.json|叢集組態範例檔案，其中包含適用於安全、三個節點、單一 VM/電腦部署叢集的所有設定，包括叢集中每個節點的相關資訊。使用 [Windows 身分識別](https://msdn.microsoft.com/library/ff649396.aspx)保護叢集的安全。|
|ClusterConfig.Windows.MultiMachine.json|叢集組態範例檔案，其中包含適用於使用 Windows 安全性的安全、多個 VM/電腦叢集的所有設定，包括安全叢集中每個電腦的相關資訊。使用 [Windows 身分識別](https://msdn.microsoft.com/library/ff649396.aspx)保護叢集的安全。|
|ClusterConfig.x509.DevCluster.json|叢集組態範例檔案，其中包含適用於安全、三個節點、單一 VM/電腦部署叢集的所有設定，包括叢集中每個節點的相關資訊。使用 x509 憑證保護叢集的安全。|
|ClusterConfig.x509.MultiMachine.json|叢集組態範例檔案，其中包含適用於安全、多個 VM/電腦叢集的所有設定，包括安全叢集中每個節點的相關資訊。使用 x509 憑證保護叢集的安全。|
|EULA.txt|使用 Microsoft Azure Service Fabric 獨立 Windows Server 封裝的授權條款。如果您想要立即下載一份 EULA，[按一下這裡](http://go.microsoft.com/fwlink/?LinkID=733084)。|
|Readme.txt|指向版本資訊和基本安裝指示的連結。這是您在此頁面上找到的一部分指示。|
|CreateServiceFabricCluster.ps1|使用 ClusterConfig.json 檔案中的設定建立叢集的 PowerShell 指令碼。|
|RemoveServiceFabricCluster.ps1|使用 ClusterConfig.json 中的設定移除叢集的 PowerShell 指令碼。|
|AddNode.ps1|用於將節點加入至現有部署叢集的 PowerShell 指令碼。|
|RemoveNode.ps1|用於在現有部署叢集移除節點的 PowerShell 指令碼。|


## 規劃及準備叢集部署
建立叢集之前，請先執行下列步驟。

### 步驟 1︰規劃叢集基礎結構
您即將在您所擁有的電腦上建立 Service Fabric 叢集，因此您可以決定您希望叢集不受何種失敗的影響。例如，您是否需要提供給這些電腦的個別電源線或網際網路連線？ 此外，請考慮這些電腦的實體安全性。實體位置在哪裡？誰需要存取這些電腦？ 一旦您做出這些決定之後，就可以依據邏輯，將電腦對應到各種容錯網域 (向下捲動以了解詳細資訊)。生產叢集的基礎結構規劃比起測試叢集更為複雜。

<a id="preparemachines"></a>
### 步驟 2︰準備符合必要條件的電腦
您想要新增到叢集的每部電腦的必要條件︰

- 建議至少 4 GB 的記憶體
- 網路連線：請確定電腦位於一或多個安全的網路。
- Windows Server 2012 R2 或 Windows Server 2012 (您必須安裝 [KB2858668](https://support.microsoft.com/kb/2858668))。
- [.NET Framework 4.5.1 或更高版本](https://www.microsoft.com/download/details.aspx?id=40773)，完整安裝。
- [Windows PowerShell 3.0](https://msdn.microsoft.com/powershell/scripting/setup/installing-windows-powershell)。
- 部署和設定叢集的叢集系統管理員必須擁有每部電腦的[系統管理員權限](https://social.technet.microsoft.com/wiki/contents/articles/13436.windows-server-2012-how-to-add-an-account-to-a-local-administrator-group.aspx)。
- [RemoteRegistry 服務](https://technet.microsoft.com/library/cc754820)應該在所有電腦上執行。

### 步驟 3︰決定初始叢集大小
獨立 Service Fabric 叢集中的每個節點都已部署 Service Fabric 執行階段，並且是叢集的成員。在一般生產部署中，每個作業系統執行個體 (實體或虛擬) 都有一個節點。叢集大小取決於您的業務需求。不過，您必須至少有三個節點 (電腦/VM) 的叢集大小。基於開發目的，在一部指定的電腦上可以有多個節點。在生產環境中，對於每個實體或虛擬機器，Service Fabric 只支援一個節點。

### 步驟 4︰決定容錯網域和升級網域的數目
**容錯網域 (FD)** 是故障的實體單元，而且與資料中心內的實體基礎結構直接相關。容錯網域是由共用單一失敗點的硬體元件 (電腦、交換器、網路等) 所組成。雖然容錯網域和機架之間沒有 1:1 對應，但是大致上來說，可以將每個機架視為一個容錯網域。考慮叢集中的節點時，強烈建議至少在三個容錯網域之間散佈節點。

當您在「ClusterConfig.json」中指定 FD 時，可以選擇每個 FD 的名稱。Service Fabric 支援階層式 FD，因此，您可以在 FD 中反映您的基礎結構拓撲。例如，下列 FD 有效：

- "faultDomain": "fd:/Room1/Rack1/Machine1"
- "faultDomain": "fd:/FD1"
- "faultDomain": "fd:/Room1/Rack1/PDU1/M1"


**升級網域 (UD)** 是節點的邏輯單元。在 Service Fabric 協調式升級 (應用程式升級或叢集升級) 期間，會關閉 UD 中的所有節點來執行升級，而其他 UD 中的節點則仍然可用來提供要求。您對電腦所進行的韌體升級不會接受 UD，因此您必須一次在一部電腦上進行。

思考這些概念最簡單的方式就是將 FD 視為非計劃性故障的單元，並將 UD 視為計劃性維護的單元。

當您在「ClusterConfig.json」中指定 UD 時，可以選擇每個 UD 的名稱。例如，下列名稱有效：

- "upgradeDomain": "UD0"
- "upgradeDomain": "UD1A"
- "upgradeDomain": "DomainRed"
- "upgradeDomain": "Blue"

如需升級網域和容錯網域的詳細資訊，請閱讀[描述 Service Fabric 叢集](service-fabric-cluster-resource-manager-cluster-description.md)一文。

### 步驟 5︰下載適用於 Windows Server 的 Service Fabric 獨立封裝
[下載適用於 Windows Server 的 Service Fabric 獨立封裝](http://go.microsoft.com/fwlink/?LinkId=730690)，並將封裝解壓縮至不屬於叢集一部分的部署電腦，或解壓縮至將屬於叢集的其中一部電腦。

<a id="createcluster"></a>
## 建立叢集

完成以上規劃和準備一節中所述的步驟之後，您就可以開始建立您的叢集。

### 步驟 1︰修改叢集組態
「ClusterConfig.json」檔案中會描述叢集。如需此檔案中各個區段的詳細資訊，請閱讀[獨立 Windows 叢集的組態設定](service-fabric-cluster-manifest.md)一文。從您下載的封裝中開啟其中一個「ClusterConfig.json」檔案，然後修改下列設定︰

|**組態設定**|**說明**|
|-----------------------|--------------------------|
|**NodeTypes**|節點類型可讓您將叢集節點分成不同的群組。一個叢集至少必須有一個節點類型。群組中的所有節點都有下列共同的特性：<br> **Name** - 這是節點類型名稱。<br>**Endpoint Ports** - 這些是與這個節點類型相關聯的各種具名端點 (連接埠)。您可以使用任何您想要的連接埠號碼，只要該號碼未與此資訊清單中的其他任何號碼衝突，而且目前沒有任何其他在電腦/VM 上執行的應用程式在使用該號碼即可。<br> **Placement Properties** - 此節點類型的這些屬性是用來做為系統服務或您的服務的放置條件約束。這些屬性是使用者定義的索引鍵/值組，可針對指定節點提供額外的中繼資料。節點屬性的範例包括節點是否有硬碟機或圖形卡、其硬碟機的磁針數、核心，以及其他實體屬性。<br> **Capacities** - 節點容量會定義特定節點可以使用的特定資源名稱和數量。例如，節點可能會定義它具有名為 "MemoryInMb" 的度量容量，而且預設有 2048 MB 的可用記憶體。這些容量會在執行階段使用，以確保需要特定資源量的服務會放在有所需資源量可用的節點上。<br>**IsPrimary** - 如果您已定義多個 NodeType，請確定只有一個使用 *true* 值設定為主要，這是系統服務的執行所在位置。其他所有節點類型應該設定為值「false」|
|**Nodes**|這些是屬於叢集一部分的每個節點的詳細資料 (節點類型、節點名稱、IP 位址、節點的容錯網域和升級網域)。您想要建立叢集所在的電腦必須與其 IP 位址一起列在這裡。<br> 如果您為所有節點使用相同的 IP 位址，則會建立一整體叢集，您可以將此叢集用於測試之用。不要使用一整體叢集部署生產工作負載。|

### 步驟 2︰執行建立叢集指令碼
一旦您修改 JSON 文件中的叢集組態，並在其中加入所有節點資訊之後，請從封裝資料夾中執行叢集建立 *CreateServiceFabricCluster.ps1* PowerShell 指令碼，並傳入 JSON 組態檔的路徑及接受 EULA。

此指令碼可以在以系統管理員身分存取叢集組態檔中列為節點的所有電腦的任何電腦上執行。執行此指令碼所在的電腦不一定是叢集的一部分。

```
#Create an unsecured local development cluster

.\CreateServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.Unsecure.DevCluster.json -AcceptEULA true
```
```
#Create an unsecured multi-machine cluster

.\CreateServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.Unsecure.MultiMachine.json -AcceptEULA true
```

>[AZURE.NOTE] 您可以在用來執行 CreateServiceFabricCluster PowerShell 的 VM/電腦上本機使用部署記錄檔。部署記錄檔位於您執行 PowerShell 命令的資料夾下名為 "DeploymentTraces" 的子資料夾中。此外，若要查看 Service Fabric 是否已正確部署至電腦，您可以在 C:\\ProgramData 目錄中找到已安裝的檔案，並可在工作管理員中看到 FabricHost.exe 和 Fabric.exe 處理程序正在執行。

### 步驟 3：連接到叢集

請參閱[這份文件](service-fabric-connect-to-secure-cluster.md)了解連線到安全的叢集的指示。

若要連線到不安全的叢集，請執行下列 PowerShell 命令

```powershell

Connect-ServiceFabricCluster -ConnectionEndpoint <*IPAddressofaMachine*>:<Client connection end point port>

Connect-ServiceFabricCluster -ConnectionEndpoint 192.13.123.2345:19000

```
### 步驟 4：啟動 Service Fabric Explorer

現在，您可以使用 http://localhost:19080/Explorer/index.html 直接從其中一部電腦或是使用 http://<IPAddressofaMachine>:19080/Explorer/index.html 從遠端利用 Service Fabric Explorer 連接到叢集



## 在叢集中新增和移除節點

當您的商務需求改變時，您可以在獨立 Service Fabric 叢集中新增或移除節點。如需詳細步驟，請閱讀[在 Service Fabric 獨立叢集中新增或移除節點](service-fabric-cluster-windows-server-add-remove-nodes.md)。


## 移除叢集

若要移除叢集，請從封裝資料夾執行 *RemoveServiceFabricCluster.ps1* PowerShell 指令碼，然後傳入 JSON 組態檔的路徑以及選擇性指定要刪除的記錄檔位置。

此指令碼可以在以系統管理員身分存取叢集組態檔中列為節點的所有電腦的任何電腦上執行。執行此指令碼所在的電腦不一定是叢集的一部分

```
.\RemoveServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.Unsecure.MultiMachine.json   
```

## 此封裝包含的預覽功能

整個封裝目前皆為預覽版。

## 後續步驟
- [獨立 Windows 叢集的組態設定](service-fabric-cluster-manifest.md)
- [在獨立 Service Fabric 叢集中新增或移除節點](service-fabric-cluster-windows-server-add-remove-nodes.md)
- [建立具有執行 Windows 之 Azure VM 的獨立 Service Fabric 叢集](service-fabric-cluster-creation-with-windows-azure-vms.md)
- [使用 Windows 安全性保護 Windows 上的獨立叢集](service-fabric-windows-cluster-windows-security.md)
- [使用 X509 憑證保護 Windows 上的獨立叢集](service-fabric-windows-cluster-x509-security.md)


<!--Image references-->
[TrustedZone]: ./media/service-fabric-cluster-creation-for-windows-server/TrustedZone.png

<!---HONumber=AcomDC_0921_2016-->