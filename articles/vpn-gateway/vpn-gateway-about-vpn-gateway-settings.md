<properties 
   pageTitle="關於虛擬網路閘道的 VPN 閘道設定 |Microsoft Azure"
   description="了解 Azure 虛擬網路的 VPN 閘道設定。"
   services="vpn-gateway"
   documentationCenter="na"
   authors="cherylmc"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager,azure-service-management"/>
<tags 
   ms.service="vpn-gateway"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="09/21/2016"
   ms.author="cherylmc" />

# 關於 VPN 閘道設定

VPN 閘道連線解決方案仰賴多重資源的組態，以在虛擬網路和內部部署位置之間傳送網路流量。每個資源都包含可設定的設定。資源和設定的組合決定連線結果。

本文各節將討論與 **Resource Manager** 部署模型中的 VPN 閘道相關的資源和設定。您可能會發現使用連線拓撲圖會有助於檢視可用的組態。您可以在[關於 VPN 閘道](vpn-gateway-about-vpngateways.md)一文中找到每個連線解決方案的描述和拓撲圖。

## <a name="gwtype"></a>閘道類型

每個虛擬網路只能有一個各類型的虛擬網路閘道。建立虛擬網路閘道時，您必須確定組態的閘道類型是正確的。

-GatewayType 的可用值為：

- Vpn
- ExpressRoute

VPN 閘道需要 `-GatewayType` *Vpn*。

範例：

	New-AzureRmVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg `
	-Location 'West US' -IpConfigurations $gwipconfig -GatewayType Vpn `
	-VpnType RouteBased
 

## <a name="gwsku"></a>閘道 SKU


[AZURE.INCLUDE [vpn-gateway-gwsku-include](../../includes/vpn-gateway-gwsku-include.md)]

**在 Azure 入口網站中指定閘道 SKU**

如果您使用 Azure 入口網站來建立 Resource Manager 虛擬網路閘道，則預設會使用「標準 SKU」來設定虛擬網路閘道。目前，您無法在 Azure 入口網站中為 Resource Manager 部署模型指定其他 SKU。不過，在建立閘道之後，您可以使用 `Resize-AzureRmVirtualNetworkGateway` PowerShell Cmdlet 來升級到更強大的閘道 SKU (從 Basic/Standard 升級到 HighPerformance)。您也可以使用 PowerShell 將閘道 SKU 大小降級。

**使用 PowerShell 來指定閘道 SKU**


下列 PowerShell 範例將 `-GatewaySku` 指定為 *Standard*。

	New-AzureRmVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg `
	-Location 'West US' -IpConfigurations $gwipconfig -GatewaySku Standard `
	-GatewayType Vpn -VpnType RouteBased

**變更閘道 SKU**

您可以調整閘道 SKU 的大小。下列 PowerShell 範例顯示將閘道 SKU 的大小調整為 HighPerformance。

	$gw = Get-AzureRmVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg
	Resize-AzureRmVirtualNetworkGateway -VirtualNetworkGateway $gw -GatewaySku HighPerformance

<br>


###  <a name="aggthroughput"></a>依 SKU 和閘道類型列出的估計彙總輸送量


下表顯示閘道類型和預估的彙總輸送量。此資料表適用於資源管理員與傳統部署模型。

[AZURE.INCLUDE [vpn-gateway-table-gwtype-aggthroughput](../../includes/vpn-gateway-table-gwtype-aggtput-include.md)]


## <a name="connectiontype"></a>連線類型

在 Resource Manager 部署模型中，每個組態皆需要特定的虛擬網路閘道連線類型。`-ConnectionType` 的可用 Resource Manager PowerShell 值為：

- IPsec
- Vnet2Vnet
- ExpressRoute
- VPNClient

在下列 PowerShell 範例中，我們會建立需要 *IPsec* 連線類型的 S2S 連線。

	New-AzureRmVirtualNetworkGatewayConnection -Name localtovon -ResourceGroupName testrg `
	-Location 'West US' -VirtualNetworkGateway1 $gateway1 -LocalNetworkGateway2 $local `
	-ConnectionType IPsec -RoutingWeight 10 -SharedKey 'abc123'


## <a name="vpntype"></a>VPN 類型

當您為 VPN 閘道組態建立虛擬網路閘道時，必須指定 VPN 類型。您所選擇的 VPN 類型取決於您想要建立的連線拓撲。例如，P2S 連線需要 RouteBased VPN 類型。VPN 類型也取決於您將使用的硬體。S2S 組態需要 VPN 裝置。有些 VPN 裝置僅支援特定 VPN 類型。

您選取的 VPN 類型必須滿足您想建立的解決方案的所有連線需求。例如，如果您想為相同的虛擬網路建立 S2S VPN 閘道連線和 P2S VPN 閘道連線，您會使用 VPN 類型 *RouteBased*，因為 P2S 需要 RouteBased VPN 類型。您也必須確認您的 VPN 裝置支援 RouteBased VPN 連線。

一旦建立虛擬網路閘道，就無法變更 VPN 類型。您必須刪除虛擬網路閘道，然後再建立新的。有兩種 VPN 類型：

[AZURE.INCLUDE [vpn-gateway-vpntype](../../includes/vpn-gateway-vpntype-include.md)]


下列 PowerShell 範例將 `-VpnType` 指定為 *RouteBased*。在建立閘道時，您必須確定用於組態的 -VpnType 是正確的。

	New-AzureRmVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg `
	-Location 'West US' -IpConfigurations $gwipconfig `
	-GatewayType Vpn -VpnType RouteBased

##  <a name="requirements"></a>閘道需求

[AZURE.INCLUDE [vpn-gateway-table-requirements](../../includes/vpn-gateway-table-requirements-include.md)]


## <a name="gwsub"></a>閘道子網路

若要設定虛擬網路閘道，您必須先為 VNet 建立閘道子網路。此閘道子網路必須命名為 *GatewaySubnet* 才能正常運作。此名稱可讓 Azure 知道這個子網路應用於閘道。

閘道子網路的大小下限完全取決於您想要建立的組態。雖然可以建立像 /29 這麼小的閘道子網路，但建議您建立 /28 或更大 (/28、/27、/26 等) 的閘道子網路。

建立較大的閘道可防止違反閘道大小限制。例如，您可能已建立一個虛擬網路閘道，其中用於 S2S 連線的閘道子網路大小為 /29。您現在想要設定一個 S2S/ExpressRoute 並存組態。該組態要求閘道子網路大小至少為 /28。若要建立您的組態，您將必須修改閘道子網路以配合該連線的最低需求，亦即 /28。

下列 Resource Manager PowerShell 範例顯示名為 GatewaySubnet 的閘道子網路。您可以看到 CIDR 標記法指定 /27，這可提供足以供大多數現有組態使用的 IP 位址。

	Add-AzureRmVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -AddressPrefix 10.0.3.0/27

>[AZURE.IMPORTANT] 請確認閘道子網路並未套用「網路安全性群組」(NSG)，因為這可能導致連線失敗。


## <a name="lng"></a>區域網路閘道

建立 VPN 閘道組態時，區域網路閘道通常代表您的內部部署位置。在傳統部署模型中，區域網路閘道被稱為本機站台。

您需指定區域網路閘道的名稱 (即內部部署 VPN 裝置的公用 IP 位址)，並指定位於內部部署位置的位址首碼。Azure 會查看網路流量的目的地位址首碼、查閱您為區域網路閘道指定的組態，然後根據這些來路由傳送封包。您也可以針對使用 VPN 閘道連線的 VNet 對 VNet 組態指定區域網路閘道。

下列 PowerShell 範例會建立新的區域網路閘道︰

	New-AzureRmLocalNetworkGateway -Name LocalSite -ResourceGroupName testrg `
	-Location 'West US' -GatewayIpAddress '23.99.221.164' -AddressPrefix '10.5.51.0/24'

有時，您會需要修改區域網路閘道設定。例如，當您新增或修改位址範圍時，或 VPN 裝置的 IP 位址變更時。針對傳統 VNet，您可以在傳統入口網站的 [區域網路] 頁面上變更這些設定。針對 Resource Manager，請參閱[使用 PowerShell 修改區域網路閘道設定](vpn-gateway-modify-local-network-gateway.md)。

## <a name="resources"></a>REST API 和 PowerShell Cmdlet

如需將 REST API 和 PowerShell Cmdlet 使用於 VPN 閘道組態的其他技術資源和特定語法需求，請參閱下列頁面︰

|**傳統** | **資源管理員**|
|-----|----|
|[PowerShell](https://msdn.microsoft.com/library/mt270335.aspx)|[PowerShell](https://msdn.microsoft.com/library/mt163510.aspx)|
|[REST API](https://msdn.microsoft.com/library/jj154113.aspx)|[REST API](https://msdn.microsoft.com/library/mt163859.aspx)|


## 後續步驟

如需有關可用連線組態的詳細資訊，請參閱[關於 VPN 閘道](vpn-gateway-about-vpngateways.md)。







 

<!---HONumber=AcomDC_0921_2016-->