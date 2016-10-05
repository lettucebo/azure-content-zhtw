<properties 
	pageTitle="如何設定進階 Azure Redis 快取的虛擬網路支援 | Microsoft Azure" 
	description="了解如何建立和管理高階層 Azure Redis 快取執行個體的虛擬網路支援" 
	services="redis-cache" 
	documentationCenter="" 
	authors="steved0x" 
	manager="douge" 
	editor=""/>

<tags 
	ms.service="cache" 
	ms.workload="tbd" 
	ms.tgt_pltfrm="cache-redis" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="09/15/2016" 
	ms.author="sdanie"/>

# 如何設定高階 Azure Redis 快取的虛擬網路支援
Azure Redis 快取有不同的快取服務，在快取大小和功能 (包括新的進階層) 的選擇上提供了彈性。

Azure Redis 快取進階層功能包括叢集、永續性及虛擬網路 (VNet) 支援。VNet 是雲端中的私人網路。當 Azure Redis 快取執行個體是以 VNet 設定時，它不是公開定址，只能從 VNet 中的虛擬機器和應用程式存取。本文說明如何設定進階 Azure Redis 快取執行個體的虛擬網路支援。

>[AZURE.NOTE] Azure Redis 快取支援傳統和 ARM VNet。

如需其他進階快取功能的相關資訊，請參閱 [Azure Redis 快取進階層簡介](cache-premium-tier-intro.md)。

## 為何使用 VNet？
[Azure 虛擬網路 (VNet)](https://azure.microsoft.com/services/virtual-network/) 部署可為您的 Azure Redis 快取、子網路、存取控制原則，以及進一步限制存取 Azure Redis 快取的其他功能提供增強的安全性和隔離模式。

## 虛擬網路支援
虛擬網路 (VNet) 支援是在快取建立期間於 [新的 Redis 快取] 刀鋒視窗中設定的。

[AZURE.INCLUDE [redis-cache-create](../../includes/redis-cache-premium-create.md)]

一旦選取進階定價層之後，您就可以藉由選取與您的快取相同的訂用帳戶和位置中的 VNet，來設定 Azure Redis 快取 VNet 整合。若要使用新的 VNet，請先建立 VNet，方法是遵循[使用 Azure 入口網站建立虛擬網路](../virtual-network/virtual-networks-create-vnet-arm-pportal.md)或[使用 Azure 入口網站建立虛擬網路 (傳統)](../virtual-network/virtual-networks-create-vnet-classic-portal.md)，然後返回 [新的 Redis 快取] 刀鋒視窗來建立和設定進階快取。

若要為新的快取設定 VNet，按一下 [新的 Redis 快取] 刀鋒視窗上的 [虛擬網路]，然後從下拉式清單中選取想要的 VNet。

![虛擬網路][redis-cache-vnet]

從 [子網路] 下拉式清單中選取所需的子網路，然後指定所需的 [靜態 IP 位址]。如果您使用傳統的 VNet，[靜態 IP 位址] 欄位就是選擇性的，而且如果未指定任何位址，則會從選取的子網路選擇一個。

>[AZURE.IMPORTANT] 將 Azure Redis 快取部署到 ARM VNet 時，快取必須位於專用子網路中，其中只能包含 Azure Redis 快取執行個體，不含其他任何資源。如果嘗試將 Azure Redis 快取部署到含有其他資源的 ARM VNet 子網路，則部署會失敗。

![虛擬網路][redis-cache-vnet-ip]

>[AZURE.IMPORTANT] 子網路中的前四個位址是保留字，無法使用。如需詳細資訊，請參閱[在這些子網路內使用 IP 位址是否有任何限制？](../virtual-network/virtual-networks-faq.md#are-there-any-restrictions-on-using-ip-addresses-within-these-subnets)

建立快取之後，您可以從 [設定] 刀鋒視窗中按一下 [虛擬網路]，以檢視 VNet 的設定。

![虛擬網路][redis-cache-vnet-info]


若要在使用 VNet 時連接到 Azure Redis 快取執行個體，請在連接字串中指定您的快取主機名稱，如下列範例所示。

	private static Lazy<ConnectionMultiplexer> lazyConnection = new Lazy<ConnectionMultiplexer>(() =>
	{
	    return ConnectionMultiplexer.Connect("contoso5premium.redis.cache.windows.net,abortConnect=false,ssl=true,password=password");
	});
	
	public static ConnectionMultiplexer Connection
	{
	    get
	    {
	        return lazyConnection.Value;
	    }
	}

## Azure Redis 快取 VNet 常見問題集

下列清單包含 Azure Redis 快取調整常見問題的解答。

-	[Azure Redis 快取和 VNet 的某些常見錯誤設定有哪些？](#what-are-some-common-misconfiguration-issues-with-azure-redis-cache-and-vnets)
-	[可以搭配標準或基本快取使用 VNet 嗎？](#can-i-use-vnets-with-a-standard-or-basic-cache)
-	[為什麼無法在某些子網路中建立 Redis 快取，但其他的可以？](#why-does-creating-a-redis-cache-fail-in-some-subnets-but-not-others)
-	[將快取裝載於 VNET 時，所有快取功能都可以正常運作嗎？](#do-all-cache-features-work-when-hosting-a-cache-in-a-vnet)


## Azure Redis 快取和 VNet 的某些常見錯誤設定有哪些？

Azure Redis 快取裝載在 VNet 時，會使用下表中的連接埠。如果封鎖這些連接埠，快取可能無法正常運作。在 VNet 中使用 Azure Redis 快取時，將其中一或多個連接埠封鎖是最常見的錯誤組態問題。

| 連接埠 | 方向 | 傳輸通訊協定 | 目的 | 遠端 IP |
|-------------|------------------|--------------------|-----------------------------------------------------------------------------------|-------------------------------------|
| 80、443 | 輸出 | TCP | Azure 儲存體/PKI 上 Redis 的相依項目 (網際網路) | * |
| 53 | 輸出 | TCP/UDP | DNS 上 Redis 的相依項目 (網際網路/VNET) | * |
| 6379, 6380 | 輸入 | TCP | 對 Redis 的用戶端通訊，Azure 負載平衡 | VIRTUAL\_NETWORK, AZURE\_LOADBALANCER |
| 8443 | 輸入/輸出 | TCP | Redis 的實作詳細資料 | VIRTUAL\_NETWORK |
| 8500 | 輸入 | TCP/UDP | Azure 負載平衡 | AZURE\_LOADBALANCER |
| 10221-10231 | 輸入/輸出 | TCP | Redis 的實作詳細資料 (可以限制連至 VIRTUAL\_NETWORK 的遠端端點) | VIRTUAL\_NETWORK, AZURE\_LOADBALANCER |
| 13000-13999 | 輸入 | TCP | 對 Redis 叢集的用戶端通訊，Azure 負載平衡 | VIRTUAL\_NETWORK, AZURE\_LOADBALANCER |
| 15000-15999 | 輸入 | TCP | 對 Redis 叢集的用戶端通訊，Azure 負載平衡 | VIRTUAL\_NETWORK, AZURE\_LOADBALANCER |
| 16001 | 輸入 | TCP/UDP | Azure 負載平衡 | AZURE\_LOADBALANCER |
| 20226 | 輸入+輸出 | TCP | Redis 叢集的實作詳細資料 | VIRTUAL\_NETWORK |


在虛擬網路中，可能一開始就不符合 Azure Redis 快取的一些網路連線需求。Azure Redis 快取需要符合下列項目，才能在虛擬網路內使用時正確運作。

-  全球 Azure 儲存體端點的輸出網路連線。這包括位於與 Azure Redis 快取執行個體相同區域中的端點，以及位於**其他** Azure 區域的儲存體端點。Azure 儲存體端點在下列 DNS 網域之下解析：table.core.windows.net、blob.core.windows.net、queue.core.windows.net 和 file.core.windows.net。
-  ocsp.msocsp.com、mscrl.microsoft.com 和 crl.microsoft.com 的輸出網路連線。需要此連線才能支援 SSL 功能。
-  虛擬網路的 DNS 設定必須能夠解析前面幾點所提到的所有端點和網域。確定已針對虛擬網路設定及維護有效的 DNS 基礎結構，即可符合 DNS 需求。



### 可以搭配標準或基本快取使用 VNet 嗎？

VNet 僅適用於進階快取。

### 為什麼無法在某些子網路中建立 Redis 快取，但其他的可以？

如果您將 Azure Redis 快取部署到 ARM VNet，快取就必須位於不含任何其他資源類型的專用子網路中。如果嘗試將 Azure Redis 快取部署到含有其他資源的 ARM VNet 子網路，則部署會失敗。您必須先刪除子網路內的現有資源，然後才能建立新的 Redis 快取。

只要有足夠的可用 IP 位址，您就可以將多個類型的資源部署到傳統的 VNet。

### 將快取裝載於 VNET 時，所有快取功能都可以正常運作嗎？

如果您的快取是 VNET 的一部分，只有在 VNET 中的用戶端可以存取快取。因此，下列快取管理功能目前沒有作用。

-	Redis 主控台 - Redis 主控台使用的 redis cli.exe 用戶端裝載於不屬於您 VNET 的 VM 上，因此主控台無法連接到您的快取。


## 搭配 Azure Redis 快取使用 ExpressRoute

客戶可以將 [Azure ExpressRoute](https://azure.microsoft.com/services/expressroute/) 循環連接至虛擬網路基礎結構，因而將其內部部署網路延伸至 Azure。

新建立的 ExpressRoute 循環預設會通告允許輸出網際網路連線的預設路由。利用此組態，用戶端應用程式能連接到其他的 Azure 端點，包括 Azure Redis 快取。

不過，常見的客戶組態是定義其專屬預設路由 (0.0.0.0/0)，以強制輸出網際網路流量來替代透過內部部署方式流動。此流量流程一定會中斷與 Azure Redis 快取的連線，因為已在內部部署封鎖輸出流量，或者 NAT 至無法再使用各種 Azure 端點的一組無法辨識位址。

解決方法是在子網路上定義包含 Azure Redis 快取的一 (或多個) 使用者定義路由 (UDR)。UDR 會定義將使用的子網路特有路由，而非預設路由。

如果可能，建議使用下列設定：

- ExpressRoute 組態會通告 0.0.0.0/0 而且預設會使用強制通道將所有輸出流量傳送至內部部署。
- 已套用至包含 Azure Redis 快取之子網路的 UDR 會使用網際網路的下一個躍點類型定義 0.0.0.0/0 (本文後面會提供其範例)。

這些步驟的合併效果是子網路層級 UDR 會優先於 ExpressRoute 強制通道，因而確保來自 Azure Redis 快取的輸出網際網路存取。

儘管基於效能因素而導致使用 ExpressRoute 從內部部署應用程式連接到 Azure Redis 快取執行個體不能當成一般使用案例 (為了達到最佳效能，Azure Redis 快取用戶端應該位於 Azure Redis 快取相同的區域)，但在此案例中，輸出網路路徑無法通過內部公司 Proxy，也不能強制它在內部部署建立通道。這麼做會變更來自 Azure Redis 快取的輸出網路流量的有效 NAT 位址。變更 Azure Redis 快取執行環境之輸出網路流量的 NAT 位址會導致上述眾多端點的連接失敗。這會導致 Azure Redis 快取建立嘗試失敗。

**重要事項：**UDR 中定義的路由**必須**明確足以優先於 ExpressRoute 組態所通告的任何路由。下列範例使用廣泛 0.0.0.0/0 位址範圍，因此使用更明確的位址範圍，有可能會不小心由路由通告所覆寫。

**非常重要：** **未正確交叉通告從公用對等互連路徑至私人對等互連路徑之路由** 的 ExpressRoute 組態不支援 Azure Redis 快取。已設定公用對等互連的 ExpressRoute 組態，會收到來自 Microsoft 的一大組 Microsoft Azure IP 位址範圍的路由通告。如果這些位址範圍在私人對等互連路徑上不正確地交叉通告，最後的結果會是來自 Azure Redis 快取執行個體子網路的所有輸出網路封包，都會不正確地使用強制通道傳送至客戶的內部部署網路基礎結構。這個網路流量會中斷 Azure Redis 快取。此問題的解決方案是停止從公用對等互連路徑至私人對等互連路徑的交叉通告路由。

如需使用者定義路由的背景資訊，請參閱此[概觀](../virtual-network/virtual-networks-udr-overview.md)。

如需 ExpressRoute 的詳細資訊，請參閱 [ExpressRoute 技術概觀](../expressroute/expressroute-introduction.md)

## 後續步驟
了解如何使用更多進階快取功能。

-	[Azure Redis Cache 高階層簡介](cache-premium-tier-intro.md)





  
<!-- IMAGES -->

[redis-cache-vnet]: ./media/cache-how-to-premium-vnet/redis-cache-vnet.png

[redis-cache-vnet-ip]: ./media/cache-how-to-premium-vnet/redis-cache-vnet-ip.png

[redis-cache-vnet-info]: ./media/cache-how-to-premium-vnet/redis-cache-vnet-info.png

<!---HONumber=AcomDC_0921_2016-->