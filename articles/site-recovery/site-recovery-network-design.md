<properties
	pageTitle="設計用於災害復原的網路基礎結構 | Microsoft Azure"
	description="這篇文章討論 Azure Site Recovery 的網路設計考量"
	services="site-recovery"
	documentationCenter=""
	authors="prateek9us"
	manager="jwhit"
	editor=""/>

<tags
	ms.service="site-recovery"
	ms.devlang="na"
	ms.topic="article"
	ms.tgt_pltfrm="na"
	ms.workload="storage-backup-recovery"
	ms.date="09/19/2016"
	ms.author="pratshar"/>

#  設計用於災害復原的網路基礎結構

這篇文章的對象是 IT 專業人員，他們負責架構設計、實作和支援業務持續性和災害復原 (BCDR) 的基礎結構，而且想要利用 Microsoft Azure Site Recovery (ASR) 來支援並增強其 BCDR 服務。本白皮書將討論 System Center Virtual Machine Manager 伺服器部署的實際考量、自動縮放子網路與子網路容錯移轉的優缺點比較，以及如何架構 Microsoft Azure 中虛擬網站的災害復原。

## Overview

[Azure Site Recovery (ASR)](https://azure.microsoft.com/services/site-recovery/) 是 Microsoft Azure 服務，可協調虛擬化應用程式的保護和復原，以達成業務持續性和災害復原 (BCDR) 之目的。這份文件的目的是引導讀者完成設計網路的程序，著重於使用 Site Recovery 複寫虛擬機器 (VM) 時如何架構災害復原網站上的 IP 範圍和子網路。

此外，本文將說明 Site Recovery 如何架構及實作多網站的虛擬資料中心，以在測試或損毀時支援 BCDR 服務。

在每個人都期待 24 小時連線不中斷的世界裡，保持基礎結構和應用程式開啟並執行的重要性更甚於以往。業務持續性和災害復原 (BCDR) 的目的是要還原失敗的元件，讓組織可以快速恢復正常營運。制定災害復原策略來應變不太可能發生的嚴重破壞事件非常具有挑戰性。這是因為預測未來的固有難度，特別是與未必會發生的事有關，再加上提供適當的措施來防止未知災難的成本很高。

BCDR 規劃的關鍵在於必須為災害復原計畫定義復原時間目標 (RTO) 和復原點目標 (RPO)。當使用 Azure Site Recovery 的客戶資料中心發生災害時，客戶可以在資料遺失最少 (低 RPO) 的情況下，迅速 (低 RTO) 讓其位於次要資料中心或 Microsoft Azure 的複寫虛擬機器上線。

ASR 讓容錯移轉成真，第一步是將指定的虛擬機器從主要資料中心複製至次要資料中心或 Azure (視案例而定)，然後定期更新複本。規劃基礎結構時，應將網路設計視為潛在瓶頸，可能會讓您無法達成公司的 RTO 和 RPO 目標。

當系統管理員打算要部署災害復原解決方案時，腦中的其中一個重要問題是，如何在容錯移轉完成後與虛擬機器連線。ASR 允許系統管理員選擇在容錯移轉之後虛擬機器連接的網路。如果主要網站是由 VMM 伺服器管理，則可以使用「網路對應」來達成。如需詳細資訊，請參閱[準備網路對應](site-recovery-network-mapping.md)。

設計復原網站的網路時，系統管理員有兩種選擇：

- 在復原網站的網路使用不同的 IP 位址範圍。在此案例中，虛擬機器在容錯移轉之後會收到新的 IP 位址，系統管理員必須進行 DNS 更新。在[這裡](site-recovery-vmm-to-vmm.md#test-your-deployment)深入了解如何進行 DNS 更新
- 在復原網站的網路使用相同的 IP 位址範圍。在某些案例中，即使容錯移轉之後，系統管理員偏好保留他們的 IP 位址給主要網站。在正常案例中，系統管理員必須更新路由以指出 IP 位址的新位置。但是，有些案例在主要和復原網站之間有部署延伸的 VLAN，保留虛擬機器的 IP 位址會變成一個不錯的選擇。保留相同 IP 位址可簡化復原程序，因為拿掉了容錯移轉後的所有網路相關步驟。


當系統管理員打算要部署災害復原解決方案時，腦中的其中一個重要問題是，如何在容錯移轉完成後與應用程式連線。現代應用程式幾乎一律在某種程度上依賴網路，因此實際將某個網站中的服務移至另一個網站，是一種網站挑戰。在災害復原解決方案中，解決這個問題有兩種主要方法。第一種方法是要維持固定 IP 位址。儘管移動服務和主控伺服器開始於不同的實體位置，但是應用程式會將 IP 位址組態連同它們帶至新位置。第二種方法則是在轉換為復原網站期間完全變更 IP 位址。每一種方法有數個實作變化，摘要說明如下。

設計復原網站的網路時，系統管理員有兩種選擇：

## 選項 1：保留 IP 位址 

從災害復原程序的觀點來看，使用固定的 IP 位址似乎是最簡單的實作方法，但它有許多潛在的挑戰，實際上反而是最不常用的方法。Azure Site Recovery 可讓您保留所有案例中的 IP 位址。決定要保留 IP 之前，應該詳加考量它將加諸於容錯移轉功能的約束。讓我們看一下可協助您決定是否保留的 IP 位址的因素。可以透過兩種方式來達成，使用延伸的子網路，或執行完整的子網路容錯移轉。

### 延伸的子網路

若採取此做法，可在主要位置和 DR 位置同時使用子網路。簡單地說，這表示您可以將伺服器和其 IP (第 3 層) 組態移至第二個網站，而且網路會自動將流量路由傳送至新位置。從伺服器的觀點來看，這是微不足道的處理，但它有幾項挑戰：

- 從第 2 層 (資料連結層) 觀點來看，它將需要可以管理延伸的 VLAN 的網路設備，不過這不成問題，因為它現在已普遍使用。第二個且更困難的問題是，透過延伸 VLAN，潛在的容錯網域會延伸到兩個網站，基本上成為單一失敗點。雖然不太可能，過去曾經發生過廣播風暴但無法隔離。關於這個問題我們得到各種意見，看過許多成功的實作，也遇過「我們永遠不會實作這種技術」的人。
- 如果您使用 Microsoft Azure 做為 DR 網站，就不可能實作延伸的子網路。


### 子網路容錯移轉

可以實作子網路容錯移轉，以獲得延伸子網路解決方案的上述優點，而無需真的跨多個網站延伸子網路。若採取此做法，任何給定的子網路會出現在網站 1 或網站 2 中，但永遠不會同時出現在這兩個網站。為了在發生容錯移轉時維持 IP 位址空間，可以程式設計方式排列路由器基礎結構，將子網路從某個網站移到另一個網站。在容錯移轉案例中，子網路會和與其相關聯的受保護 VM 一起移動。這種方法的主要缺點是，如果失敗，您必須移動整個子網路，這可能沒問題，但是可能會影響容錯移轉的細微性考量。

讓我們看看虛構企業 Contoso 如何能夠將其 VM 複寫到復原位置，同時又能夠容錯移轉整個子網路。首先，我們將看看 Contoso 如何能夠管理它的子網路，同時又在兩個內部部署位置之間複寫 VM，然後，我們將討論當 [Azure 做為災害復原網站](#failover-to-azure)時，子網路容錯移轉如何運作。

#### 容錯移轉至次要內部部署網站

讓我們看另一個案例，我們將保留每個 VM 的 IP，並一同容錯移轉完整的子網路。主要站台有在子網路 192.168.1.0/24 中執行的應用程式。當容錯移轉發生時，屬於此子網路中的所有虛擬機器將會容錯移轉至復原網站，並保留其 IP 位址必須適當地修改路由，以反映所有屬於子網路 192.168.1.0/24 的虛擬機器現在都已移至復原網站的事實。

下圖中，主要網站與復原網站、第三個網站與主要網站、以及第三個網站與復原網站之間的路由都必須適當地修改。

下圖顯示容錯移轉之前的子網路。子網路 192.168.0.1/24 在容錯移轉之前於主要網站上為作用中，在容錯移轉之後於復原網站變成作用中

![容錯移轉之前](./media/site-recovery-network-design/network-design2.png)

容錯移轉之前


下圖顯示容錯移轉之後的網路和子網路。
	
![容錯移轉之後](./media/site-recovery-network-design/network-design3.png)

容錯移轉之後

如果您的次要網站為內部部署且您使用 VMM 伺服器管理它，則當啟用特定虛擬機器的保護時，ASR 會根據以下工作流程配置網路資源：

- ASR 會從相關網路上針對每個 System Center VMM 執行個體定義的靜態 IP 位址集區，配置虛擬機器上每個網路介面的 IP 位址。
- 如果系統管理員為復原網站上的網路定義的 IP 位址集區，與主要網站上網路的 IP 位址集區相同，則 ASR 在配置複本虛擬機器的 IP 位址時，會配置與主要虛擬機器相同的 IP 位址。IP 會保留在 VMM 中，但不是會設定為容錯移轉 IP。容錯移轉 IP 會在容錯移轉之前設定。

![保留 IP 位址](./media/site-recovery-network-design/network-design4.png)
	
圖 5

圖 5 顯示複本虛擬機器的容錯移轉 TCP/IP 設定 (在 Hyper-V 主控台上)。這些設定會在虛擬機器啟動之前、容錯移轉之後填入。

如果找不到相同的 IP，ASR 會配置已定義的 IP 位址集區中一些其他可用的 IP 位址。

在啟用 VM 保護之後，您可以使用下列範例指令碼，來確認已配置給虛擬機器的 IP。相同的 IP 會設定為容錯移轉 IP，並在容錯移轉時指派給 VM：

    	$vm = Get-SCVirtualMachine -Name <VM_NAME>
		$na = $vm[0].VirtualNetworkAdapters>
		$ip = Get-SCIPAddress -GrantToObjectID $na[0].id
		$ip.address  

>[AZURE.NOTE] 在虛擬機器使用 DHCP 的案例中，IP 位址的管理完全是在 ASR 控制範圍之外。系統管理員必須確保負責`提供復原網站上 IP 位址的 DHCP 伺服器，可以從與主要網站相同的範圍中提供位址。

#### 容錯移轉至 Azure

Azure Site Recovery (ASR) 能讓 Microsoft Azure 做為虛擬機器的災害復原網站。在此情況下，您必須多處理一個限制。

讓我們看看這個案例，虛構公司 Woodgrove Bank 的內部部署基礎結構裝載了企業營運系統應用程式，其行動裝置應用程式則裝載在 Azure 之上。Azure 中的 Woodgrove Bank VM 以及內部部署伺服器之間的連線，由站對站 (S2S) 虛擬私人網路 (VPN) 提供。S2S VPN 使得 Azure 中的 Woodgrove Bank 的虛擬網路，可被視為 Woodgrove Bank 內部部署網路的擴充。這項通訊由 Woodgrove Bank 邊緣和 Azure 虛擬網路之間的 S2S VPN 實現。現在，Woodgrove 想要使用 ASR 將其資料中心內執行的工作負載複寫到 Azure。此做法符合 Woodgrove 的需求，既擁有經濟實惠的 DR 選項，又能夠將資料儲存在公用雲端環境。Woodgrove 必須處理相依於硬式編碼 IP 位址的應用程式和組態，因此在容錯移轉至 Azure 之後，它們需要保留其應用程式的 IP 位址。

Woodgrove 決定將來自 IP 位址範圍 (172.16.1.0/24, 172.16.2.0/24) 的 IP 位址指派給其在 Azure 中執行的資源。

為了讓 Woodgrove 能夠將其虛擬機器複寫至 Azure 並同時保留 IP 位址，需要建立 Azure 虛擬網路。此網路應該是內部部署網路的擴充，這樣應用程式才可以順暢地從內部部署網站容錯移轉至 Azure。Azure 可讓您將站對站以及點對站 VPN 連線能力加入在 Azure 中建立的虛擬網路。當設定站對站連線時，只在 IP 位址範圍不同於內部部署 IP 位址範圍時，Azure 網路才可讓您將流量路由傳送到內部部署位置 (Azure 將它稱為本機網路)，因為 Azure 不支援延伸子網路。這表示，如果您有子網路 192.168.1.0/24 內部部署，則您無法在 Azure 網路中加入本機網路 192.168.1.0/24。這是預期行為，因為 Azure 不知道子網路中沒有任何作用中的 VM，也不知道子網路是針對 DR 用途而建立的。若要能夠從 Azure 網路正確地路由傳送網路流量，網路和本機網路中的子網路不得發生衝突。

![子網路容錯移轉之前](./media/site-recovery-network-design/network-design7.png)

容錯移轉之前

為了協助 Woodgrove 滿足他們的業務需求，我們必須實作下列工作流程：

- 建立額外的網路，我們叫它「復原網路」，容錯移轉的虛擬機器將在這裡建立。
- 為了確保 VM 的 IP 會在容錯移轉之後保留，請移至於 VM 屬性下方的 [設定] 索引標籤，指定 VM 在內部部署使用的相同 IP，然後按一下 [儲存]。當 VM 容錯移轉時，Azure Site Recovery 將指派前述 IP 給虛擬機器。

![網路屬性](./media/site-recovery-network-design/network-design8.png)

一旦觸發容錯移轉，並在復原網路中以想要的 IP 位址建立虛擬機器後，可以使用 [Vnet 對 Vnet 連接](../vpn-gateway/virtual-networks-configure-vnet-to-vnet-connection.md)建立與此網路的連接：如果需要，此動作可以編寫指令碼。如同我們在關於子網路容錯移轉的前一節所討論，如果容錯移轉至 Azure，路由也須適當地修改，以反映該 192.168.1.0/24 現在已移至 Azure。

![子網路容錯移轉之後](./media/site-recovery-network-design/network-design9.png)

容錯移轉之後

如果您沒有如上圖所示的「Azure 網路」。您可以在容錯移轉之後，在「主要網站」與「復原網路」之間建立站對站 VPN 連接。


## 選項 2：變更 IP 位址

根據我們所見，這種方法似乎最普遍。採取的做法是變更容錯移轉中每個 VM 的 IP 位址。此方法的缺點是連入網路需要「學習」原本在 IPx 的應用程式現在在 IPy。即使 IPx 和 IPy 是邏輯名稱，在整個網路中，DNS 項目通常需要變更或排清，而且網路資料表中的快取項目必須更新或排清，因此，停機時間取決於 DNS 基礎結構如何設定。在內部網路應用程式的情況下使用低 TTL 值，以及對網際網路架構應用程式使用 [Azure 流量管理員與 ASR](http://azure.microsoft.com/blog/2015/03/03/reduce-rto-by-using-azure-traffic-manager-with-azure-site-recovery/)，即可減輕這些問題。

### 變更 IP 位址 - 圖

讓我們看一下在主要和復原網站使用不同 IP 的案例。在以下範例中，我們有第三個網站，可以從中存取主要或復原網站上裝載的應用程式。

![不同的 IP - 容錯移轉之前](./media/site-recovery-network-design/network-design10.png)

圖 11

在 圖 11 中，有些應用程式裝載在主要網站上的子網路 192.168.1.0/24 中，而且它們已設定為在容錯移轉後出現在子網路 172.16.1.0/24 的復原網站上。已適當地設定 VPN 連線/網路路由，讓所有的三個站台可以互相存取。
 
如圖 12 所示，在容錯移轉一或多個應用程式之後，它們會在復原子網路中還原。在此情況下，我們不受同時容錯移轉整個子網路的限制。不需要進行任何變更來重新設定 VPN 或網路路由。容錯移轉和某些 DNS 更新會確定應用程式可供存取。如果 DNS 設定為允許動態更新，則虛擬機器一旦在容錯移轉之後啟動，就會使用新的 IP 自行註冊。

![不同的 IP - 容錯移轉之後](./media/site-recovery-network-design/network-design11.png)

圖 12

在容錯移轉之後，複本虛擬機器具有的 IP 位址可能與主要虛擬機器的 IP 位址不同。虛擬機器在啟動後將更新他們正在使用的 DNS 伺服器。在整個網路中，DNS 項目通常需要變更或排清，而且網路資料表中的快取項目必須更新或排清，因此這些狀態變更發生時面臨停機時間，不是罕見的情況。減輕這個問題的方法有：

- 對內部網路應用程式使用低 TTL 值。
- 對網際網路架構應用程式使用 Azure 流量管理員與 ASR。
- 在您的復原計劃內使用下列指令碼來更新 DNS 伺服器，以確保及時更新 (如果已設定動態 DNS 登錄，不需要指令碼)

		string]$Zone,
		[string]$name,
		[string]$IP
		)
		$Record = Get-DnsServerResourceRecord -ZoneName $zone -Name $name
		$newrecord = $record.clone()
		$newrecord.RecordData[0].IPv4Address  =  $IP
		Set-DnsServerResourceRecord -zonename $zone -OldInputObject $record -NewInputObject $Newrecord


### 變更 IP 位址 - DR 到 Azure

[ Microsoft Azure 做為災害復原網站的網路基礎結構設定](http://azure.microsoft.com/blog/2014/09/04/networking-infrastructure-setup-for-microsoft-azure-as-a-disaster-recovery-site/)部落格文章說明如何在不一定要保留 IP 位址時，設定必要的 Azure 網路基礎結構。文章一開始描述應用程式，接著探討如何在內部部署及 Azure 設定網路，最後說明如何執行測試容錯移轉和計劃容錯移轉。



## 後續步驟

[了解](site-recovery-network-mapping.md)當 VMM 伺服器用來管理主要網站時，「網站復原」如何對應來源和目標網路。

<!---HONumber=AcomDC_0921_2016-->