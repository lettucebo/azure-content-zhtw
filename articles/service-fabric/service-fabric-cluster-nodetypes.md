<properties
   pageTitle="Service Fabric 節點類型與 VM 調整集 |Microsoft Azure"
   description="說明 Service Fabric 節點類型如何與 VM 調整集產生關聯，以及如何遠端連線到 VM 調整集執行個體或叢集節點。"
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
   ms.date="09/09/2016"
   ms.author="chackdan"/>


# Service Fabric 節點類型與虛擬機器調整集之間的關聯性

虛擬機器調整集是一個 Azure 計算資源，可以用來將一組 VM 當做一個集合加以部署和管理。在 Service Fabric 叢集中定義的每個節點類型都會安裝為不同的 VM 擴展集。然後每個節點類型可以獨立相應增加或相應減少，可以開啟不同組的連接埠，並可以有不同的容量度量。

下列螢幕擷取畫面顯示具有 FrontEnd 和 BackEnd 兩個節點類型的叢集。每個節點類型各有五個節點。

![有兩個節點類型的叢集螢幕擷取畫面][NodeTypes]

## 將 VM 調整集執行個體對應至節點

如您在上面看到的，VM 調整集執行個體是從執行個體 0 開始，然後漸漸增加。編號反映在名稱中。例如，BackEnd_0 是 BackEnd VM 擴展集的執行個體 0。這個特定的 VM 擴展集有五個執行個體，名稱分別是 BackEnd_0、BackEnd_1、BackEnd_2、BackEnd_3、BackEnd_4。

當您相應增加 VM 調整集，就會建立的新執行個體。VM 調整集的名稱通常是 VM 調整集名稱 + 下一個執行個體編號。在我們的範例中是 BackEnd\_5。


## 將 VM 調整集負載平衡器對應至每個節點類型/VM 調整集

如果您已從入口網站或使用我們提供的範例 Resource Manager 範本部署您的叢集，您會得到一張資源群組下所有資源的清單，而且您會看到每個 VM 擴展集或節點類型的負載平衡器。

名稱類似 **LB-&lt;NodeType name&gt;**。例如，以下螢幕擷取畫面中的 LB-sfcluster4doc-0：


![資源][Resources]


## 遠端連線到 VM 調整集執行個體或叢集節點
在叢集中定義的每個節點類型都會安裝為不同的 VM 擴展集。這表示節點類型可以獨立相應增加或相應減少，且可由不同 VM SKU 組成。不同於單一執行個體的 VM，VM 調整集的執行個體不會收到自己的虛擬 IP 位址。所以當您要尋找可用來遠端連線至特定執行個體的 IP 位址和連接埠時，可能有點困難。

您可以依照以下步驟來找出它們。

### 步驟 1：找出該節點類型的虛擬 IP 位址，然後找出 RDP 的輸入 NAT 規則

若要獲得這項資訊，您必須取得 **Microsoft.Network/loadBalancers** 的資源定義中定義的輸入 NAT 規則值。

在入口網站中，瀏覽至 [負載平衡器] 刀鋒視窗的 [設定]。

![LBBlade][LBBlade]


在 [設定] 中，按一下 [輸入 NAT 規則]。這會給您可用來遠端連線至第一個 VM 調整集執行個體的 IP 位址和連接埠。在以下的螢幕擷取畫面中是 **104.42.106.156** 和 **3389**

![NATRules][NATRules]

### 步驟 2：找到可用來遠端連線至特定 VM 調整集執行個體/節點的連接埠

本文稍早討論了如何將 VM 調整集執行個體對應至節點。我們將使用它來找出確切的連接埠。

連接埠是以 VM 擴展集執行個體的遞增順序配置。因此，在 FrontEnd 節點類型的範例中，五個執行個體的每個連接埠分別如下。您現在需要對您的 VM 擴展集執行個體進行相同的對應。

|**VM 擴展集執行個體**|**連接埠**|
|-----------------------|--------------------------|
|FrontEnd\_0|3389|
|FrontEnd\_1|3390|
|FrontEnd\_2|3391|
|FrontEnd\_3|3392|
|FrontEnd\_4|3393|
|FrontEnd\_5|3394|


### 步驟 3：遠端連線到特定 VM 調整集執行個體

在以下螢幕擷取畫面中，使用「遠端桌面連接」來連線到 FrontEnd\_1：

![RDP][RDP]

## 如何變更 RDP 連接埠範圍值

### 叢集部署之前

當您使用 Resource Manager 範本設定叢集時，可以在 **inboundNatPools** 指定範圍。

移至 **Microsoft.Network/loadBalancers** 的資源定義。您會在下面找到 **inboundNatPools** 的描述。取代 *frontendPortRangeStart* 和 *frontendPortRangeEnd* 值。

![InboundNatPools][InboundNatPools]


### 叢集部署之後
這稍微複雜一點，而且可能會導致 VM 設定被回收。您現在必須使用 Azure PowerShell 設定新的值。請確定您的電腦已安裝 Azure PowerShell 1.0+ 或更新版本。如果您還沒安裝，強烈建議您依照[如何安裝和設定 Azure PowerShell](../powershell-install-configure.md) 一文中所述的步驟來進行。

登入您的 Azure 帳戶。如果這個 PowerShell 命令因為某些原因無法運作，您應該檢查 Azure PowerShell 是否已正確安裝。

```
Login-AzureRmAccount
```

執行下列命令取得負載平衡器的詳細資料，您會看到 **inboundNatPools** 的值以及描述：

```
Get-AzureRmResource -ResourceGroupName <RGname> -ResourceType Microsoft.Network/loadBalancers -ResourceName <load balancer name>
```

現在將 *frontendPortRangeStart* 和 *frontendPortRangeEnd* 設為您要的值。

```
$PropertiesObject = @{
	#Property = value;
}
Set-AzureRmResource -PropertyObject $PropertiesObject -ResourceGroupName <RG name> -ResourceType Microsoft.Network/loadBalancers -ResourceName <load Balancer name> -ApiVersion <use the API version that get returned> -Force
```


## 後續步驟

- [「到處部署」功能和與 Azure 受管理叢集比較的概觀](service-fabric-deploy-anywhere.md)
- [叢集安全性](service-fabric-cluster-security.md)
- [ Service Fabric SDK 和開始使用](service-fabric-get-started.md)


<!--Image references-->
[NodeTypes]: ./media/service-fabric-cluster-nodetypes/NodeTypes.png
[Resources]: ./media/service-fabric-cluster-nodetypes/Resources.png
[InboundNatPools]: ./media/service-fabric-cluster-nodetypes/InboundNatPools.png
[LBBlade]: ./media/service-fabric-cluster-nodetypes/LBBlade.png
[NATRules]: ./media/service-fabric-cluster-nodetypes/NATRules.png
[RDP]: ./media/service-fabric-cluster-nodetypes/RDP.png

<!---HONumber=AcomDC_0921_2016-->