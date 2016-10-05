<properties
   pageTitle="ExpressRoute 常見問題集"
   description="ExpressRoute 常見問題集包含支援的 Azure 服務、費用、資料和連線、SLA、提供者和位置、頻寬等資訊及其他技術詳細資料。"
   documentationCenter="na"
   services="expressroute"
   authors="cherylmc"
   manager="carmonm"
   editor=""/>
<tags
   ms.service="expressroute"
   ms.devlang="na"
   ms.topic="article" 
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="09/19/2016"
   ms.author="cherylmc"/>

# ExpressRoute 常見問題集


## 什麼是 ExpressRoute？
ExpressRoute 這項 Azure 服務可讓您在 Microsoft 資料中心和內部部署或共置設備中的基礎結構之間建立私人連線。ExpressRoute 連線不會經過公用網際網路，相較於網際網路一般連線，它提供了更高的安全性、可靠性、速度以及更低的延遲。

### 使用 ExpressRoute 和私人網路連線的優勢有哪些？
ExpressRoute 連線不會經過公用網際網路，相較於網際網路一般連線，它提供了更高的安全性、可靠性、速度以及更低且一致的延遲。在某些情況下，使用 ExpressRoute 連線在內部部署裝置和 Azure 之間傳輸資料，也可以產生重大的成本效益。

### ExpressRoute 支援哪些 Microsoft 雲端服務？
ExpressRoute 目前支援大多數的 Microsoft Azure 服務，包括 Office 365。搜尋近期是否有公開上市的更新。

### 哪裡可以使用此服務？
請參閱以下頁面，以取得服務位置和可用性：[ExpressRoute 合作夥伴和位置](expressroute-locations.md)。

### 如果我和其中一個 ExpressRoute 載波合作夥伴沒有合作關係，我要如何使用 ExpressRoute 來與 Microsoft 連線？
您可以在區域載波和陸地乙太網路連線中選取其中一個支援的 Exchange 提供者位置。您接著可以在提供者位置上與 Microsoft 對等互連。檢查 [ExpressRoute 合作夥伴和位置](expressroute-locations.md)的最後一部分，以查看您的服務提供者是否存在於任何 Exchange 位置中。您接著可以訂購一個 ExpressRoute 電路，透過服務提供者連線至 Azure。

### ExpressRoute 需要多少錢？
如需價格資訊，請查看[價格詳細資訊](https://azure.microsoft.com/pricing/details/expressroute/)。

### 如果我支付指定頻寬的 ExpressRoute 電路，則我是否必須從網路服務提供者購買相同速度的 VPN 連線？
否。您可以從服務提供者購買任何速度的 VPN 連線。不過，您與 Azure 的連線將會受限於您所購買的 ExpressRoute 電路頻寬。

### 如果我支付指定頻寬的 ExpressRoute 電路，我是否能夠在需要時突增到較高的速度？
是。ExpressRoute 電路已設定支援您可以突增兩倍的所採購頻寬限制，且無需另外付費。請向您的服務提供者查詢是否支援此功能。

### 我是否可以在虛擬網路和其他 Azure 服務中同時使用相同私人網路連線？
是。設定 ExpressRoute 電路之後，您可以同時存取虛擬網路內部的服務和其他 Azure 服務。您將透過私人對等互連路徑連線至虛擬網路，並透過公用對等互連路徑連線至其他服務。

### ExpressRoute 是否提供服務等級協定 (SLA)？
如需詳細資訊，請參閱 [ExpressRoute SLA 頁面](https://azure.microsoft.com/support/legal/sla/)。

## 支援的服務
ExpressRoute 支援大多數的 Azure 雲端服務。

- 私人對等互連路徑支援與虛擬網路中所部署虛擬機器和雲端服務的連線能力。
- 公用對等互連路徑支援 Azure 網站。
- 公用對等互連路徑支援 IoT 中樞。
- Microsoft 對等互連路徑支援 Office 365。
- 您可透過公用對等互連路徑存取所有其他服務。例外狀況如下。

	**不支援下列服務：**

	- CDN
	- Visual Studio Team Services 負載測試
	- 多因素驗證
	- 流量管理員

## 資料與連線

### 我可以使用 ExpressRoute 傳輸的資料量是否有所限制？
我們不會設定資料傳輸量的限制。如需頻寬費率的相關資訊，請參閱[價格詳細資訊](https://azure.microsoft.com/pricing/details/expressroute/)。

### ExpressRoute 支援的連線速度為何？
支援的頻寬優惠：

|50 Mbps、100 Mbps、200 Mbps、500 Mbps、1Gbps、2 Gbps、5 Gbps、10Gbps|

### 可以使用的服務提供者有哪些？
如需服務提供者和位置清單，請參閱 [ExpressRoute 合作夥伴和位置](expressroute-locations.md)。

## 技術詳細資訊

### 將我的內部部署位置連線至 Azure 會有哪些技術需求？
請參閱 [ExpressRoute 必要條件頁面](expressroute-prerequisites.md)以取得相關需求。

### 連線至 ExpressRoute 是多餘的嗎？
是。為提供高可用性，每個 Express Route 電路都設有交叉連線的備援配對。

### 如果我的其中一個 ExpressRoute 連結失敗，連線是否就會中斷？
如果其中一個交叉連線失敗，您的連線就會中斷。您可以使用備援連線來支援網路的負載。為取得失敗恢復，您可以在其他對等互連位置額外建立多個電路。

### <a name="onep2plink"></a>如果我不要在雲端交換中共置，而我的服務提供者提供點對點連線，我需要在內部部署網路與 Microsoft 之間訂購兩個實體連線嗎？ 
不需要，如果您的服務提供者可以透過實體連線建立兩個乙太網路的虛擬電路，您就只需要一個實體連線。實體連線 (例如光纖) 的終點在實體層 (L1) 裝置 (請見下圖)。兩個乙太網路虛擬電路都會標記不同的 VLAN ID，一個供主要電路使用，一個供次要電路使用。這些 VLAN ID 位於外部 802.1Q 乙太網路標頭中。內部 802.1Q 乙太網路標頭 (不顯示) 會對應至特定的 [ExpressRoute 路由網域](expressroute-circuit-peerings.md)。

![](./media/expressroute-faqs/expressroute-p2p-ref-arch.png)


### 我可以使用 ExpressRoute 來擴充其中一個至 Azure 的 VLAN 嗎？
否。我們不支援至 Azure 的第 2 層連線擴充程式。

### 我的訂用帳戶是否可以有多個 ExpressRoute 電路？
是。您的訂用帳戶可以有多個 ExpressRoute 電路。預設的專用電路數目限制已設為 10。如需提高限制，您可以洽詢 Microsoft 支援。

### 我可以使用其他服務提供者的 ExpressRoute 電路嗎？
是。您可以使用許多服務提供者的 ExpressRoute 電路。每個 ExpressRoute 電路只會與一個服務提供者產生關聯。

### 我要如何將虛擬網路連線至 ExpressRoute 電路
基本步驟說明如下。

- 您必須建立 ExpressRoute 電路，並要求服務提供者將它啟用。
- 您或提供者必須設定 BGP 對等互連。
- 您必須將虛擬網路連結至 ExpressRoute 電路。

如需詳細資訊，請參閱[ExpressRoute 工作流程電路佈建和電路狀態](expressroute-workflows.md)。

### 我的 ExpressRoute 電路是否有連線範圍？
是。[ExpressRoute 合作夥伴和位置](expressroute-locations.md)頁面提供 ExpressRoute 電路的連線範圍概觀。ExpressRoute 電路的連線能力僅限於單一地緣政治區域。透過啟用 ExpressRoute 進階功能，可將連線能力擴充到不同地緣政治區域。

### 我可以將多個虛擬網路連結至 ExpressRoute 電路嗎？
是。您最多可以將 10 個虛擬網路連結至 ExpressRoute 電路。

### 我有多個含有虛擬網路的 Azure 訂用帳戶。我可以將各個訂用帳戶的虛擬網路連線到單一 ExpressRoute 電路嗎？
是。您最多可以向 10 個其他 Azure 訂用帳戶授權使用單一 ExpressRoute 電路。您可透過啟用 ExpressRoute 進階功能來提高此限制。

如需詳細資訊，請參閱[在多個訂用帳戶中共用 ExpressRoute 電路](expressroute-howto-linkvnet-arm.md)。

### 連線到相同電路的虛擬網路會相互隔離嗎？
否。從路由的觀點來看，所有連結至相同 ExpressRoute 電路的虛擬網路會屬於相同的路由網域，且不會相互隔離。如果需要路由隔離，您必須建立個別的 ExpressRoute 電路。

### 我可以將一個虛擬網路連結至多個 ExpressRoute 電路嗎？
是。您最多可以將單一虛擬網路連結至 4 個 ExpressRoute 電路。它們必須透過 4 個不同的 [ExpressRoute 位置](expressroute-locations.md)來排序。

### 我可以從連線至 ExpressRoute 電路的虛擬網路來存取網際網路嗎？
是。如果您尚未透過 BGP 工作階段通告預設路由 (0.0.0.0/0) 或網際網路路由首碼，您將能夠從連結至 ExpressRoute 電路的虛擬網路連線到網際網路。

### 我可以封鎖連線至 ExpressRoute 電路之虛擬網路的網際網路連線嗎？
是。您可以通告預設路由 (0.0.0.0/0) 以封鎖所有部署於虛擬網路內之虛擬機器的網際網路連線，並將所有流量透過 ExpressRoute 電路路由出去。請注意，如果您通告預設路由，我們會將流量透過公用對等互連 (例如 Azure 儲存體和 SQL DB) 強制流向提供的服務，再回到您的內部部署。您將必須設定路由器，以將流量透過公用對等互連路徑或透過網際網路傳回 Azure。

### 連結至相同 ExpressRoute 電路的虛擬網路是否可以互通訊息？
是。在連線到相同 ExpressRoute 電路的虛擬網路中部署的虛擬機器可以互相通訊。

### 我是否可以將虛擬網路的站對站連線與 ExpressRoute 搭配使用?
是。ExpressRoute 可與站對站 VPN 並存。

### 我可以將虛擬網路從站對站/點對站組態移至使用 ExpressRoute 嗎？
是。您必須在您的虛擬網路內建立 ExpressRoute 閘道。會有與此程序相關的短暫停機時間。

### 我該採取什麼動作才能透過 ExpressRoute 連線至 Azure 儲存體？
您必須建立 ExpressRoute 電路，並設定公用對等互連的路由。

### 是否有可通告的路由數限制？
是。我們接受私人對等互連最多使用 4000 個路由首碼；公用對等互連和 Microsoft 對等互連各自最多使用 200 個路由首碼。如果您啟用 ExpressRoute 進階功能，您可以針對私人對等互連將此值提高至 10,000 個路由。

### 我可以透過 BGP 工作階段通告的 IP 範圍有無限制？
我們不接受公用和 Microsoft 對等互連 BGP 工作階段中的私人首碼 (RFC1918)。

### 如果超過 BGP 限制，該怎麼辦？
BGP 工作階段將會被刪除。當首碼計數降至限制以下時，系統便會重設 BGP 工作階段。

### 什麼是 ExpressRoute BGP 保留時間？ 可調整此保留時間嗎？
保留時間為 180。會每隔 60 秒傳送保持運作訊息。這些是 Microsoft 端固定的設定，無法變更。

### 在將預設路由 (0.0.0.0/0) 通告至虛擬網路之後，我無法啟動在 Azure VM 上執行的 Windows。我該怎麼辦？
下列步驟將協助 Azure 識別啟動需求：

1. 為您的 ExpressRoute 電路建立公用對等互連。
2. 執行 DNS 查詢，並尋找 **kms.core.windows.net** 的 IP 位址
3. 然後執行下列其中一個項目，以便金鑰管理服務識別啟動需求是來自 Azure，並接受要求。
	- 在您的內部部署網路上，將目的地為 IP 位址 (在步驟 2 中取得) 的流量透過公用對等互連路由回到 Azure。
	- 要求您的 NSP 提供者將流量透過公用對等互連夾帶回 Azure。

### 我可以變更 ExpressRoute 電路的頻寬嗎？
是。您可以增加 ExpressRoute 電路的頻寬，而無需將它拆解。您必須追蹤連線提供者的後續情形，以確保他們更新網路內的流速以支援增加頻寬。不過，妳無法降低 ExpressRoute 電路的頻寬。必須降低頻寬係指拆解和重新建立 ExpressRoute 電路。

### 我如何變更 ExpressRoute 電路的頻寬？
您可以使用更新的專用電路 API 和 PowerShell cmdlet，來更新 ExpressRoute 電路的頻寬。

## ExpressRoute Premium

### 什麼是 ExpressRoute Premium？
ExpressRoute Premium 是下面所列功能的集合。

 - 將私用對等互連的路由表限制，從 4000 個路由提高到 10,000 個路由。
 - 提高可連線到 ExpressRoute 電路的 VNet 數目 (預設值為 10)。如需詳細資訊，請參閱下表。
 - 透過 Microsoft 核心網路的全球連線。您現在能夠將某一個地緣政治區域中的 VNet 與另一個區域中的 ExpressRoute 電路連結。**範例：**您可以將在歐洲西部建立的 VNet created 連結至在矽谷建立的 ExpressRoute 電路。
 - 可連線至 Office 365 服務和 CRM Online。

### 如果已啟用 ExpressRoute Premium，我可以將多少個 VNet 連結至 ExpressRoute 電路？
下表顯示 ExpressRoute 限制和每個 ExpressRoute 電路的 VNet 數目。


[AZURE.INCLUDE [expressroute-limits](../../includes/expressroute-limits.md)]


### 我要如何啟用 ExpressRoute Premium？
當啟用功能時便可啟用 ExpressRoute Premium 功能，並可透過更新電路狀態將它關閉。您可以在建立電路時啟用 ExpressRoute Premium，或呼叫更新專用電路 API / PowerShell cmdlet 以啟用 ExpressRoute Premium。

### 我要如何停用 ExpressRoute Premium？
您可以呼叫更新專用電路 API / PowerShell cmdlet 以停用 ExpressRoute Premium。您必須確認已調整連線需求以達到預設限制，才能停用 ExpressRoute Premium。如果您的使用率擴充超越預設限制，我們將捨棄要求並停用 ExpressRoute Premium。

### 我可以從進階功能集中挑選和選擇所需功能嗎？
否。您無法挑選所需功能。當您開啟 ExpressRoute Premium 時，我們便會將所有功能啟用。

### ExpressRoute Premium 需要多少錢？
如需成本相關資訊，請參閱[價格詳細資訊](https://azure.microsoft.com/pricing/details/expressroute/)。

### 除了標準的 ExpressRoute 費用以外，我是否仍需支付 ExpressRoute Premium？
是。除了 ExpressRoute 電路費用和連線提供者所需費用以外，還需另行支付 ExpressRoute Premium 費用。

## ExpressRoute 與 Office 365 服務和 CRM Online

[AZURE.INCLUDE [expressroute-office365-include](../../includes/expressroute-office365-include.md)]

### 我要如何建立可連線到 Office 365 服務和 CRM Online 的ExpressRoute 電路？

1. 請檢閱 [ExpressRoute 必要條件頁面](expressroute-prerequisites.md)，以確定您符合需求。
2. 請檢閱 [ExpressRoute 合作夥伴和位置](expressroute-locations.md)中的服務提供者和位置清單，以確定符合您的連線需求。
3. 透過檢閱 [Office 365 的網路規劃和效能調整](http://aka.ms/tune/)來計劃您的容量需求。
4. 遵循以下工作流程中所列的步驟來設定連線：[ExpressRoute 工作流程線路佈建和線路狀態](expressroute-workflows.md)。

>[AZURE.IMPORTANT] 設定與 Office 365 服務和 CRM Online 的連線時，請確定您已啟用 ExpressRoute 進階附加元件。

### 我是否需要啟用 Azure 公用對等以連接至 Office 365 服務和 CRM Online？
否，您只需要啟用 Microsoft 對等互連。將會透過 Microsoft 對等互連傳送至 Azure AD 的驗證流量。

### 我的現有 ExpressRoute 電路是否支援與 Office 365 服務和 CRM Online 連線？
是。您可以設定現有 ExpressRoute 電路以支援與 Office 365 服務的連線。請確保您有足夠的容量可以連線到 Office 365 服務，並確定您已啟用進階附加元件。[Office 365 的網路規劃和效能調整](http://aka.ms/tune/)將協助您計劃連線需求。另請參閱[建立和修改 ExpressRoute 電路](expressroute-howto-circuit-classic.md)。

### 透過 ExpressRoute 連線可以存取哪些 Office 365 服務？

請參閱 [Office 365 URL 與 IP 位址範圍](http://aka.ms/o365endpoints)頁面，以取得透過 ExpressRoute 支援的最新服務清單。

### 適用於 Office 365 服務和 CRM Online 的 ExpressRoute 費用是多少？
必須啟用進階附加元件，才能使用 Office 365 服務和 CRM Online。[價格詳細資料頁面](https://azure.microsoft.com/pricing/details/expressroute/)提供 ExpressRoute 費用的詳細資料。

### 哪些區域支援 ExpressRoute for Office 365？
如需支援 ExpressRoute 的合作夥伴和位置清單詳細資訊，請參閱 [ExpressRoute 合作夥伴和位置](expressroute-locations.md)。

### 即使 ExpressRoute 已設定供我的組織使用，我是否可以透過網際網路存取 Office 365？
是。即使 ExpressRoute 已設定供我的組織使用，您仍可以透過網際網路存取 Office 365 服務端點。如果所處位置已設定透過 ExpressRoute 連線到 Office 365 服務，您將會透過 ExpressRoute 進行連線。

### 可以透過 ExpressRoute 連線存取 Dynamics AX Online 嗎？
不，不支援此做法。

<!---HONumber=AcomDC_0921_2016-->