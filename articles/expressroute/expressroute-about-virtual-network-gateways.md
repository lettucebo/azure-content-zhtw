<properties 
   pageTitle="關於 ExpressRoute 虛擬網路閘道 | Microsoft Azure"
   description="了解 ExpressRoute 的虛擬網路閘道。"
   services="expressroute"
   documentationCenter="na"
   authors="cherylmc"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager, azure-service-management"/>
<tags 
   ms.service="expressroute"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="09/21/2016"
   ms.author="cherylmc" />

# 關於 ExpressRoute 的虛擬網路閘道


虛擬網路閘道可用來傳送 Azure 虛擬網路和內部部署位置之間的網路流量。當您設定 ExpressRoute 連線時，您必須建立及設定虛擬網路閘道和虛擬網路閘道連線。

當您建立虛擬網路閘道時，需要指定數個設定。其中一個必要設定會指定是否要對 ExpressRoute 或 VPN 閘道流量使用閘道。在 Resource Manager 部署模型中，此設定是「-GatewayType」。

如果網路流量是在專用私人連線上傳送，則使用的閘道類型為「ExpressRoute」。也稱為 ExpressRoute 閘道。如果網路流量是透過公用連線加密傳送，則使用的閘道類型為「Vpn」。稱之為 VPN 閘道。站對站、點對站和 VNet 對 VNet 連線都使用 VPN 閘道。

對於每種閘道類型，每個虛擬網路只能有一個虛擬網路閘道。例如，您可以有一個使用 -GatewayType Vpn 的虛擬網路閘道，以及一個使用 -GatewayType ExpressRoute 的虛擬網路閘道。本文著重於 ExpressRoute 虛擬網路閘道。

## <a name="gwsku"></a>閘道 SKU

[AZURE.INCLUDE [expressroute-gwsku-include](../../includes/expressroute-gwsku-include.md)]

如果您使用 Azure 入口網站來建立 Resource Manager 虛擬網路閘道，則預設會使用「標準 SKU」來設定虛擬網路閘道。目前，您無法在 Azure 入口網站中為 Resource Manager 部署模型指定其他 SKU。不過，在建立閘道之後，您可以使用 'Resize-AzureRmVirtualNetworkGateway' PowerShell Cmdlet 來升級到更強大的閘道 SKU (例如，從 Basic/Standard 升級到 HighPerformance)。

###  <a name="aggthroughput"></a>依閘道 SKU 列出的估計彙總輸送量


下表顯示閘道類型和預估的彙總輸送量。此資料表適用於資源管理員與傳統部署模型。

[AZURE.INCLUDE [expressroute-table-aggthroughput](../../includes/expressroute-table-aggtput-include.md)]


## <a name="resources"></a>REST API 和 PowerShell Cmdlet

如需將 REST API 和 PowerShell Cmdlet 使用於虛擬網路閘道組態時的其他技術資源和特定語法需求，請參閱下列頁面︰

|**傳統** | **資源管理員**|
|-----|----|
|[PowerShell](https://msdn.microsoft.com/library/mt270335.aspx)|[PowerShell](https://msdn.microsoft.com/library/mt163510.aspx)|
|[REST API](https://msdn.microsoft.com/library/jj154113.aspx)|[REST API](https://msdn.microsoft.com/library/mt163859.aspx)|


## 後續步驟

如需有關可用連線組態的詳細資訊，請參閱 [ExpressRoute 概觀](expressroute-introduction.md)。







 

<!---HONumber=AcomDC_0921_2016-->