<properties
	pageTitle="在 Azure 入口網站使用 Azure Site Recovery 將 VMWare 虛擬機器和實體伺服器複寫至 Azure | Microsoft Azure"
	description="描述如何使用 Azure 入口網站部署 Azure Site Recovery，以協調內部部署 VMware 虛擬機器或 Windows/Linux 實體伺服器至 Azure 的複寫、容錯移轉和復原。"
	services="site-recovery"
	documentationCenter=""
	authors="rayne-wiselman"
	manager="jwhit"
	editor=""/>

<tags
	ms.service="site-recovery"
	ms.workload="backup-recovery"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/12/2016"
	ms.author="raynew"/>

# 使用 Azure 入口網站的 Azure Site Recovery 將 VMware 虛擬機器和實體機器複寫至 Azure

> [AZURE.SELECTOR]
- [Azure 入口網站](site-recovery-vmware-to-azure.md)
- [Azure 傳統型](site-recovery-vmware-to-azure-classic.md)
- [Azure 傳統 (舊版)](site-recovery-vmware-to-azure-classic-legacy.md)

歡迎使用 Azure Site Recovery！ 如果您想要在 Azure 入口網站中使用 Azure Site Recovery 將內部部署 VMware 虛擬機器或 Windows/Linux 實體伺服器複寫至 Azure，請使用這份文件。

> [AZURE.NOTE] Azure 建立和處理資源的[部署模型](../resource-manager-deployment-model.md)有二種：Azure Resource Manager (ARM) 和傳統。Azure 也有兩個入口網站 – 支援傳統部署模型的 Azure 傳統入口網站，以及支援兩種部署模型的 Azure 入口網站。

Azure 入口網站中的 Site Recovery 提供許多新功能︰

- Azure 備份和 Azure Site Recovery 服務會結合成單一復原服務保存庫，以便從單一位置設定和管理業務持續性和災害復原 (BCDR)。在統一儀表板中，您可以監視及管理您的內部部署網站與 Azure 公用雲端的作業。
- 具有與雲端解決方案提供者 (CSP) 程式一起佈建之 Azure 訂用帳戶的使用者，現在可以在 Azure 入口網站中管理 Site Recovery 作業。
- Azure 入口網站中的 Site Recovery 可以將機器複寫至 ARM 儲存體帳戶。容錯移轉時，Site Recovery 會在 Azure 中建立 ARM 架構的 VM。
- Site Recovery 會繼續支援複寫至傳統儲存體帳戶。容錯移轉時，Site Recovery 會使用傳統模型建立 VM。

閱讀本文之後，請將任何意見張貼在底部的 Disqus 註解中。請在 [Azure Recovery Services Forum (Azure 復原服務論壇)](https://social.msdn.microsoft.com/forums/azure/home?forum=hypervrecovmgr) 提出技術問題。

## 概觀

組織需要 BCDR 策略，以決定應用程式、工作負載和資料如何在規劃與未規劃停機期間維持運作，並儘速復原到正常運作的情況。BCDR 策略應保護商務資料安全且可復原，並確保發生災害時工作負載仍持續可用。

Site Recovery 是一項 Azure 服務，可藉由將內部部署實體伺服器和虛擬機器的複寫協調至雲端 (Azure) 或次要資料中心，協助您的 BCDR 策略。當您的主要位置發生故障時，您容錯移轉至次要位置，讓應用程式和工作負載保持可用。當它恢復正常作業時，容錯回復至您的主要位置。深入了解[什麼是 Azure Site Recovery？](site-recovery-overview.md)

本文提供將內部部署 VMware VM 和 Windows/Linux 實體伺服器複寫至 Azure 所需的全部資訊。包括架構概觀、規劃資訊，以及設定 Azure、內部部署伺服器、複寫設定和容量規劃的部署步驟。設定好基礎結構之後，您可以在您要保護的電腦上啟用複寫，並檢查容錯移轉是否可以運作。

## 商務優點

- Site Recovery 會為在 VMware VM 和實體伺服器上執行的商務工作負載和應用程式提供離站保護。
- 復原服務入口網站提供單一位置來設定、管理及監視複寫、容錯移轉和復原。
- Site Recovery 可以自動探索加入 vSphere 主機中的 VMware VM。
- 您可以輕鬆執行從內部部署基礎結構容錯移轉至 Azure，以及從 Azure 容錯回復 (還原) 至內部部署網站中的 VMware VM 伺服器。
- 您可以啟用多部 VM 並建立複寫群組，以便分層跨越多部機器的應用程式工作負載會同時複寫。在容錯移轉時，複寫群組中的所有機器都有損毀一致且應用程式一致的復原點。針對容錯移轉，您可以在復原方案中收集多部機器，以便與階層式應用程式工作負載一起容錯移轉。

## 支援的作業系統

### Windows (僅 64 位元)
- Windows Server 2008 R2 SP1+
- Windows Server 2012
- Windows Server 2012 R2

### Linux (僅 64 位元)
- Red Hat Enterprise Linux 6.7、7.1、7.2
- CentOS 6.5、6.6、6.7、7.0、7.1、7.2
- Oracle Enterprise Linux 6.4、6.5，執行 Red Hat 相容核心或 Unbreakable Enterprise Kernel 第 3 版 (UEK3)
- SUSE Linux Enterprise Server 11 SP3

## 案例架構

案例元件如下：

- **組態伺服器**：協調通訊和管理資料複寫與復原程序的內部部署機器。在此機器上，您將執行單一安裝程式檔案來安裝組態伺服器和下列額外元件︰
	- **處理序伺服器**：作為複寫閘道器。它會從受保護的來源機器接收複寫資料，以快取、壓縮和加密進行最佳化，然後將複寫資料傳送至 Azure 儲存體。它還會處理用來保護機器的行動服務的推入安裝，並執行 VMWare VM 的自動探索。組態伺服器上會安裝預設的處理序伺服器。您可以部署額外的獨立處理序伺服器來調整您的部署。
	- **主要目標伺服器**：在從 Azure 容錯回復期間，處理複寫資料。

- **行動服務**：此元件會部署在您想要複寫至 Azure 的每部機器上 (VMware VM 或實體伺服器)。它會擷取在電腦上寫入的資料，並將它們轉送到處理序伺服器。
- **Azure**：您不需要建立任何 Azure VM 來處理對 Azure 的複寫及容錯移轉。您需要 Azure 訂用帳戶、Azure 儲存體帳戶來儲存複寫的資料，以及 Azure 虛擬網路讓 Azure VM 在容錯移轉後連接到網路。儲存體帳戶和網路必須位於與復原服務保存庫相同的區域中。
- **容錯回復**︰當您準備好要在容錯移轉之後從 Azure 容錯回復到內部部署網站時，需要建立 Azure VM 作為暫時的處理序伺服器。容錯回復完成後，您可以刪除它。針對容錯回復，您還需要在內部部署網站與 Azure VM 所在 Azure 網路之間具有 VPN (或 Azure ExpressRoute) 連線。如果容錯回復流量繁重，則您可能也需要設定內部部署的專用主要目標伺服器機器。若是流量較輕，可以使用在組態伺服器上執行的預設主要目標伺服器。


下圖顯示這些元件如何互動。

![架構](./media/site-recovery-vmware-to-azure/v2a-architecture-henry.png)

**圖 1：VMware/實體到 Azure**

## Azure 必要條件

以下是您在 Azure 中部署此案例所需的項目。

**必要條件** | **詳細資料**
--- | ---
**Azure 帳戶**| 您將需要 [Microsoft Azure](http://azure.microsoft.com/) 帳戶。您可以從[免費試用](https://azure.microsoft.com/pricing/free-trial/)開始。[深入了解](https://azure.microsoft.com/pricing/details/site-recovery/) Site Recovery 價格。
**Azure 儲存體** | 複寫的資料會儲存在 Azure 儲存體，而在容錯移轉時會建立 Azure VM。<br/><br/>若要儲存資料，您需要在與復原服務保存庫相同的區域中有標準或進階儲存體帳戶。<br/><br/>您可以使用 LRS 或 GRS 儲存體帳戶。我們建議使用 GRS，以便在發生區域性停電或無法復原主要區域時，能夠恢復資料。[深入了解](../storage/storage-redundancy.md)。<br/><br/> [進階儲存體](../storage/storage-premium-storage.md)通常用於需要持續高 IO 效能和低度延遲的虛擬機器來裝載 IO 密集型工作負載。<br/><br/> 如果您想要使用進階儲存體帳戶來儲存複寫的資料，則也需要標準儲存體帳戶來儲存複寫記錄檔，而這類記錄檔會擷取內部部署資料的進行中變更。<br/><br/> 請注意，在 Azure 入口網站中建立的儲存體帳戶不能跨越資源群組移動。目前在印度中部與印度南部也不支援對進階儲存體帳戶提供保護。<br/><br/> [深入了解](../storage/storage-introduction.md) Azure 儲存體。
**Azure 網路** | 容錯移轉發生時，您需要 Azure VM 會連接的 Azure 虛擬網路。Azure 虛擬網路必須位於與復原服務保存庫相同的區域中。
**從 Azure 容錯回復** | 您將需要設定暫存處理序伺服器作為 Azure VM。您可以在準備好要容錯回復時建立，並在容錯回復完成之後刪除它。<br/><br/> 若要容錯回復，您將需要有從 Azure 網路到內部部署網站的 VPN 連線 (或 Azure ExpressRoute)。

## 組態伺服器必要條件

您將設定內部部署機器作為組態伺服器。

**必要條件** | **詳細資料**
--- | ---
**組態伺服器**| 您必須執行 Windows Server 2012 R2 的內部部署實體或虛擬機器。所有內部部署 Site Recovery 元件是安裝在此機器上。<br/><br/>針對 VMware VM 複寫，建議您部署伺服器作為高可用性的 VMware VM。如果您要複寫實體機器，則機器可以是實體伺服器。<br/><br/> 從 Azure 容錯回復到內部部署網站時一律是容錯回復到 VMware VM，無論您是容錯移轉 VM 還是實體伺服器。如果您不將組態伺服器部署為 VMware VM，則必須設定不同的主要目標伺服器作為 VMware VM，以接收容錯回復流量。<br/><br/>如果伺服器是 VMware VM，網路介面卡類型應該是 VMXNET3。如果您使用不同類型的網路介面卡，則必須在 vSphere 5.5 伺服器上安裝 [VMware 更新](https://kb.vmware.com/selfservice/microsites/search.do?cmd=displayKC&docType=kc&externalId=2110245&sliceId=1&docTypeID=DT_KB_1_1&dialogID=26228401&stateId=1)。<br/><br/>伺服器應該要有靜態 IP 位址。<br/><br/>伺服器不應該是網域控制站。<br/><br/>伺服器的主機名稱應該在 15 個字元以內。<br/><br/>應該只有英文版作業系統。<br/><br/> 您必須在組態伺服器上安裝 VMware vSphere PowerCLI 6.0。<br/><br/>組態伺服器需要網際網路存取。需要對外存取權，如下︰<br/><br/>安裝 Site Recovery 元件期間 HTTP 80 上的暫時存取權 (以便下載 MySQL)<br/><br/>HTTPS 443 上的持續對外存取權，用於管理複寫<br/><br/>HTTPS 9443 上的持續對外存取權，用於處理複寫流量 (可以修改此連接埠)<br/><br/>伺服器也需要下列 URL 的存取權，以便連線到 Azure：*.hypervrecoverymanager.windowsazure.com；*.accesscontrol.windows.net；*.backup.windowsazure.com；*.blob.core.windows.net；*.store.core.windows.net<br/><br/>如果您的伺服器上有以 IP 位址為基礎的防火牆規則，請檢查這些規則是否允許對 Azure 進行通訊。您將需要允許 [Azure 資料中心 IP 範圍](https://www.microsoft.com/download/confirmation.aspx?id=41653)和 HTTPS (443) 通訊協定。<br/><br/>請允許您訂用帳戶之 Azure 區域及美國西部的 IP 位址範圍。<br/><br/>允許此 URL 以進行 MySQL 下載：.http://cdn.mysql.com/archives/mysql-5.5/mysql-5.5.37-win32.msi


## VMware vCenter/vSphere 主機必要條件

**必要條件** | **詳細資料**
--- | ---
**vSphere**| 您需要一或多個 VMware vSphere Hypervisor。<br/><br/>Hypervisor 應該執行具有最新更新項目的 6.0、5.5 或 5.1 版 vSphere。<br/><br/>建議您讓 vSphere 主機和 vCenter 伺服器位於與處理序伺服器相同的網路 (除非您已設定專用的處理序伺服器，否則這會是組態伺服器所在的網路)。
**vCenter** | 建議您部署 VMware vCenter 伺服器來管理您的 vSphere 主機。它應該執行具有最新更新項目的 6.0 或 5.5 版 vCenter。<br/><br/>請注意，Site Recovery 不支援新的 vCenter 和 vSphere 6.0 功能，例如跨 vCenter vMotion、虛擬磁碟區和儲存體 DRS。Site Recovery 支援僅限於 5.5 版中可用的功能。


## 受保護的機器必要條件

**必要條件** | **詳細資料**
--- | ---
**內部部署 VMware VM** | 您想要保護的 VMware VM 應該安裝並執行 VMware 工具。<br/><br/> 您想要保護的機器應該要符合建立 Azure VM 的 [Azure 必要條件](site-recovery-best-practices.md#azure-virtual-machine-requirements)。<br/><br/>受保護機器上的個別磁碟容量不應超過 1023 GB。VM 可以有多達 64 個磁碟 (因此多達 64 TB)。<br/><br/>不支援使用加密的磁碟 (不論是根磁碟還是資料磁碟) 來保護 VM。

不支援共用磁碟客體叢集。<br/><br/>如果您想要啟用「應用程式一致性」，應將受保護 VM 上的「連接埠 20004」保持開啟。

不支援「整合可延伸韌體介面」(UEFI)/「可延伸韌體介面」(EFI) 開機。<br/><br/>機器名稱應包含 1 到 63 個字元 (字母、數字及連字號)。名稱必須以字母或數字開頭，並以字母或數字結尾。為機器啟用複寫之後，您就可以修改 Azure 名稱。<br/><br/>如果來源 VM 有 NIC 小組，在容錯移轉至 Azure 之後，它會轉換成單一 NIC。<br/><br/>如果受保護的 VM 具有 iSCSI 磁碟，則當 VM 容錯移轉至 Azure 時，Site Recovery 會將受保護的 VM iSCSI 磁碟轉換成 VHD 檔案。如果 Azure VM 可以觸及 iSCSI 目標，則它會連接到 iSCSI 目標，並且基本上查看兩個磁碟 – Azure VM 上的 VHD 磁碟和來源 iSCSI 磁碟。在此情況下，您必須將出現在 Azure VM 的 iSCSI 目標中斷連線。**Windows 機器 (實體或 VMware)** | 機器應該執行受支援的 64 位元作業系統：Windows Server 2012 R2、Windows Server 2012 或 Windows Server 2008 R2 (至少為 SP1)。<br/><br/> 作業系統應該安裝在 C:\\ 磁碟機上。OS 磁碟應該是 Windows 基本磁碟而非動態磁碟。資料磁碟可以是動態磁碟。<br/><br/>Site Recovery 支援具有 RDM 磁碟的 VM。在容錯回復期間，如果原始來源 VM 和 RDM 磁碟可用，則 Site Recovery 會重複使用 RDM 磁碟。如果它們都無法使用，在容錯回復期間，Site Recovery 會為每個磁碟建立新的 VMDK 檔案。**Linux 機器** | 您將需要受支援的 64 位元作業系統：Red Hat Enterprise Linux 6.7、7.1、7.2；Centos 6.5、6.6、6.7、7.0、7.1、7.2；執行 Red Hat 相容核心或 Unbreakable Enterprise Kernel 第 3 版 (UEK3) 的 Oracle Enterprise Linux 6.4、6.5；SUSE Linux Enterprise Server 11 SP3。<br/><br/>受保護機器上的 /etc/hosts 檔案應該包含將本機主機名稱對應至與所有網路介面卡關聯之 IP 位址的項目。<br/><br/>如果您想要在容錯移轉之後，使用「安全殼層」用戶端 (SSH) 來連線到執行 Linux 的 Azure 虛擬機器，請確定在受保護電腦上的「安全殼層」服務已設定為在系統開機時自動啟動，而且防火牆規則允許對其進行 SSH 連線。<br/><br/>主機名稱、掛接點、裝置名稱及 Linux 系統路徑和檔案名稱 (例如 /etc/、/usr) 應該僅使用英文。<br/><br/>只能針對具有下列儲存體的 Linux 機器啟用保護：檔案系統 (EXT3、ETX4、ReiserFS、XFS)；多重路徑軟體裝置對應工具 (multipath)；磁碟區管理員：(LVM2)。不支援使用 HP CCISS 控制站儲存體的實體伺服器。只有在 SUSE Linux Enterprise Server 11 SP3 上才支援 ReiserFS 檔案系統。<br/><br/>Site Recovery 支援具有 RDM 磁碟的 VM。在 Linux 的容錯回復期間，Site Recovery 不會重複使用 RDM 磁碟。取而代之的是，它會為每個對應的 RDM 磁碟建立新的 VMDK 檔案。<br/><br/>請確定您在 VMware 的 VM 組態參數中設定了 disk.enableUUID=true 設定。如果不存在，請建立該項目。這樣才能提供一致的 UUID 給 VMDK，使其可正確掛接。加入這項設定也可確保在容錯回復期間只將差異變更傳輸回到內部部署，而不是傳輸完整的複寫。**行動服務** | **Windows**︰若要將行動服務自動推送到執行 Windows 的 VM，您將需要提供系統管理員帳戶 (Windows 機器上的本機系統管理員)，如此處理序伺服器才能執行推送安裝。<br/><br/> **Linux**︰若要將行動服務自動推送到執行 Linux 的 VM，您將需要建立一個可供處理序伺服器用來執行推送安裝的帳戶。<br/><br/> 依預設會複寫機器上的所有磁碟。若要[將磁碟從複寫範圍中排除](#exclude-disks-from-replication)，必須先在機器上手動安裝行動服務，然後才啟用複寫。

## 準備部署

若要準備進行部署，您必須︰

1. [設定 Azure 網路](#set-up-an-azure-network)，Azure VM 會在容錯移轉之後建立時位於此網路中。此外，針對容錯回復，您需要設定從 Azure 網路到內部部署網站的 VPN 連線 (或 Azure ExpressRoute)。
2. 針對複寫的資料[設定 Azure 儲存體帳戶](#set-up-an-azure-storage-account)。
3. 在 vCenter 伺服器或 vSphere 主機上[準備帳戶](#prepare-an-account-for-automatic-discovery)，以便 Site Recovery 可以自動偵測新增的 VMware VM。
4. [準備組態伺服器](#prepare-the-configuration-server)以確保它可以存取所需的 URL，並安裝 vSphere PowerCLI 6.0。


### 設定 Azure 網路

- 此網路應與您將部署復原服務保存庫的網路位於相同的 Azure 區域。
- 視您想要針對已容錯移轉的 Azure VM 使用的資源模型而定，您將以 [ARM 模式](../virtual-network/virtual-networks-create-vnet-arm-pportal.md)或[傳統模式](../virtual-network/virtual-networks-create-vnet-classic-pportal.md)設定 Azure 網路。
- 若要從 Azure 容錯回復到內部部署的 VMware 網站，您需要從複寫的 Azure VM 所在 Azure 網路到組態伺服器所在內部部署網路的 VPN 連線 (或 Azure ExpressRoute 連線)。
- [深入了解](../vpn-gateway/vpn-gateway-site-to-site-create.md)針對 VPN 站台間連線支援的部署方法，以及如何[設定連線](../vpn-gateway/vpn-gateway-site-to-site-create.md#create-your-virtual-network)。
- 或者，您也可以設定 [Azure ExpressRoute](../expressroute/expressroute-introduction.md)。[深入了解](../expressroute/expressroute-howto-vnet-portal-classic.md)如何使用 ExpressRoute 來設定 Azure 網路。

> [AZURE.NOTE] [Migration of networks](../resource-group-move-resources.md) 對於用於部署 Site Recovery 的網路，不支援跨相同訂用帳戶內的資源群組或跨訂用帳戶。

### 設定 Azure 儲存體帳戶

- 您需要標準或進階 Azure 儲存體帳戶來保存複寫到 Azure 的資料。此帳戶必須位於與復原服務保存庫相同的區域中。視您想要針對已容錯移轉的 Azure VM 使用的資源模型而定，您將以 [ARM 模式](../storage/storage-create-storage-account.md)或[傳統模式](../storage/storage-create-storage-account-classic-portal.md)設定帳戶。
- 如果您將進階帳戶使用於複寫的資料，則必須建立其他標準帳戶來儲存複寫記錄檔，而這類記錄檔會擷取內部部署資料的進行中變更。

> [AZURE.NOTE] [Migration of storage accounts](../resource-group-move-resources.md) 對於用於部署 Site Recovery 的儲存體帳戶，不支援跨相同訂用帳戶內的資源群組或跨訂用帳戶。

### 準備帳戶以進行自動探索

Site Recovery 處理序伺服器可以自動探索 vSphere 主機或管理主機之 vCenter 伺服器上的 VMware VM。若要執行自動探索，Site Recovery 需要可以存取 VMware 伺服器的認證。如果您僅是複寫實體機器，則這個動作不相關。

1. 若要使用專用帳戶進行自動探索，請在 vCenter 層級建立具備[必要權限](#vmware-account-permissions)的角色 (例如 Azure\_Site\_Recovery)。
2. 在 vSphere 主機或 vCenter 伺服器上建立新的使用者，並將角色指派給該使用者。稍後您會讓 Site Recovery 知道這些認證，以便它能執行自動探索。

	>[AZURE.NOTE] 具有唯讀角色的 vCenter 使用者帳戶，可以執行容錯移轉，但不能關閉受保護的來源機器。如果您想要關閉這些機器，您將需要 [Azure\_Site\_Recovery](#vmware-account-permissions) 角色。如果您只是從 VMware 將 VM 移轉至 Azure，且不需要容錯回復，則唯讀角色已足夠。

### 準備組態伺服器

1.	請確定您用來做為組態伺服器的機器符合[必要條件](#configuration-server-prerequisites)。尤其是請確定機器已使用下列設定連接到網際網路︰

	- 允許存取這些 URL： *.hypervrecoverymanager.windowsazure.com；*.accesscontrol.windows.net；*.backup.windowsazure.com；*.blob.core.windows.net；*.store.core.windows.net
	- 允許存取 [http://cdn.mysql.com/archives/mysql-5.5/mysql-5.5.37-win32.msi](http://cdn.mysql.com/archives/mysql-5.5/mysql-5.5.37-win32.msi) 以下載 MySQL。
	- 允許使用 [Azure 資料中心 IP 範圍](https://www.microsoft.com/download/confirmation.aspx?id=41653)和 HTTPS (443) 通訊協定對 Azure 進行防火牆通訊。

2.	在組態伺服器上下載並安裝 [VMware vSphere PowerCLI 6.0](https://developercenter.vmware.com/tool/vsphere_powercli/6.0)。(目前不支援 PowerCLI 的其他版本，包括 6.0 版的 R 版本)。


## 建立復原服務保存庫

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 按一下 [新增] > [管理] > [備份和 Site Recovery (OMS)]。或者，您也可以按一下 [瀏覽] > [復原服務保存庫] > [加入]。

	![新增保存庫](./media/site-recovery-vmware-to-azure/new-vault3.png)

3. 在 [名稱] 中，指定保存庫的易記識別名稱。如果您有多個訂用帳戶，請選取其中一個。
4. [建立新的資源群組](../resource-group-template-deploy-portal.md)或選取現有的資源群組。指定 Azure 區域。機器將會複寫到此區域。請注意，用於 Site Recovery 的 Azure 儲存體和網路必須位於相同的區域。若要查看支援的地區，請參閱 [Azure Site Recovery 定價詳細資料](https://azure.microsoft.com/pricing/details/site-recovery/) (英文) 中的＜各地區上市情況＞。
4. 如果您想要從儀表板快速存取保存庫，請按一下 [釘選到儀表板]，然後按一下 [建立]。

	![新增保存庫](./media/site-recovery-vmware-to-azure/new-vault-settings.png)

新的保存庫會出現在 [儀表板] > [所有資源] 上，以及主要 [復原服務保存庫] 刀鋒視窗上。

## 開始使用

Site Recovery 提供的「快速入門」經驗，其設計是要協助您盡快開始運作。它會檢查必要條件，並引導您完成部署 Site Recovery 所需的步驟。

您會選取您要複寫的機器類型，以及您要複寫到的位置。您會設定基礎結構，包括內部部署伺服器、Azure 設定、複寫原則及容量規劃。您的基礎結構已準備就緒後，請啟用 VM 與實體伺服器的複寫。然後，您便可以執行特定機器的容錯移轉，或建立復原方案來容錯移轉多部機器。

選擇您要部署 Site Recovery 的方式，即可開始「快速入門」。視您的複寫需求而定，「快速入門」流程會稍有不同。


## 步驟 1︰選擇您的保護目標

選取您要複寫的項目以及您要複寫到的位置。

1. 在 [復原服務保存庫] 刀鋒視窗中選取您的保存庫，然後按一下 [設定]。
2. 在 [設定] > [快速入門] 中，按一下 [Site Recovery] > [步驟 1︰準備基礎結構] > [保護目標]。

	![選擇目標](./media/site-recovery-vmware-to-azure/choose-goals.png)

3. 在 [保護目標] 中選取 [至 Azure]，然後選取 [是，使用 VMware vSphere Hyperviso]。然後按一下 [確定]。

	![選擇目標](./media/site-recovery-vmware-to-azure/choose-goals2.png)


## 步驟 2︰設定來源環境

設定組態伺服器，並在復原服務保存庫中註冊它。如果您要複寫 VMware VM，請指定您用於自動探索的 VMware 帳戶。

1. 按一下 [步驟 1：準備基礎結構] > [來源]。在 [準備來源] 中，如果您沒有組態伺服器，請按一下 [+組態伺服器] 來新增一部伺服器。

	![設定來源](./media/site-recovery-vmware-to-azure/set-source1.png)

2. 在 [加入伺服器] 刀鋒視窗中，檢查 [組態伺服器] 是否出現在 [伺服器類型] 中。
3. 在您設定組態伺服器之前，請先確認[必要條件](#configuration-server-prerequisites)。尤其是要檢查機器是否可以存取必要的 URL。
4.	下載 Site Recovery 統一安裝的安裝檔案。
5.	下載保存庫註冊金鑰。您會在執行統一安裝時用到此金鑰。該金鑰在產生後會維持 5 天有效。

	![設定來源](./media/site-recovery-vmware-to-azure/set-source2.png)

6.	在您用來作為組態伺服器的機器上，執行統一安裝以安裝組態伺服器、處理序伺服器與主要目標伺服器。


### 執行 Site Recovery 統一安裝

1.	執行統一安裝的安裝檔案。
2.	在 [開始之前] 選取 [安裝組態伺服器和處理序伺服器]。

	![開始之前](./media/site-recovery-vmware-to-azure/combined-wiz1.png)

3. 在 [協力廠商軟體授權] 中，按一下 [我接受] 來下載並安裝 MySQL。

	![第三方=廠商軟體](./media/site-recovery-vmware-to-azure/combined-wiz105.PNG)

4. 在 [註冊] 中，瀏覽並選取您從保存庫下載的註冊金鑰。

	![註冊](./media/site-recovery-vmware-to-azure/combined-wiz3.png)

5. 在 [網際網路設定] 中，指定在組態伺服器上執行的 Provider 將如何透過網際網路連接到 Azure Site Recovery。

	- 如果您想要使用機器上目前設定的 Proxy 來連線，請選取 [以現有的 Proxy 設定連線]。
	- 如果您想要讓 Provider 直接連線，請選取 [不使用 Proxy 直接連線]。
	- 如果現有的 Proxy 需要驗證，或是您想要讓 Provider 使用自訂 Proxy 來連線，請選取 [以自訂 Proxy 設定連線]。
		- 如果您使用自訂 proxy，您必須指定位址、連接埠以及認證
		- 如果您使用 Proxy，您應該已經允許[必要條件](#configuration-server-prerequisites)中所述的 URL。

	![防火牆](./media/site-recovery-vmware-to-azure/combined-wiz4.png)

6. 在 [必要條件檢查] 中，安裝程式會執行檢查來確定可以執行安裝。如果出現有關「通用時間同步處理檢查」的警告，請確認系統時鐘上的時間 ([日期和時間] 設定) 與時區相同。

	![必要條件](./media/site-recovery-vmware-to-azure/combined-wiz5.png)

7. 在 [MySQL 組態] 中，建立認證來登入將要安裝的 MySQL 伺服器執行個體。

	![MySQL](./media/site-recovery-vmware-to-azure/combined-wiz6.png)

8. 在 [環境詳細資料] 中，選取您是否要複寫 VMware VM。如果是的話，安裝程式會檢查是否已安裝 PowerCLI 6.0。

	![MySQL](./media/site-recovery-vmware-to-azure/combined-wiz7.png)

9. 在 [安裝位置] 中，選取您要安裝二進位檔及儲存快取的位置。您可以選取至少有 5 GB 可用儲存體的磁碟機，但我們建議快取磁碟機至少有 600 GB 的可用空間。

	![安裝位置](./media/site-recovery-vmware-to-azure/combined-wiz8.png)

10. 在 [網路選取] 中，指定組態伺服器將用來傳送和接收複寫資料的接聽程式 (網路介面卡和 SSL 連接埠)。您可以修改預設連接埠 (9443)。除了此連接埠之外，網頁伺服器將會使用連接埠 443 來協調複寫作業。443 不應該用於接收複寫流量。


	![網路選擇](./media/site-recovery-vmware-to-azure/combined-wiz9.png)



11.  在 [摘要] 中檢閱資訊，然後按一下 [安裝]。安裝完成時，會產生複雜密碼。在您啟用複寫時會需要它，所以請將它複製並保存在安全的位置。

	![摘要](./media/site-recovery-vmware-to-azure/combined-wiz10.png)

12.  註冊完成後，伺服器會顯示在保存庫的 [設定] > [伺服器] 刀鋒視窗中。



#### 從命令列執行安裝程式

您可以從命令列設定組態伺服器︰

    UnifiedSetup.exe [/ServerMode <CS/PS>] [/InstallDrive <DriveLetter>] [/MySQLCredsFilePath <MySQL credentials file path>] [/VaultCredsFilePath <Vault credentials file path>] [/EnvType <VMWare/NonVMWare>] [/PSIP <IP address to be used for data transfer] [/CSIP <IP address of CS to be registered with>] [/PassphraseFilePath <Passphrase file path>]

當安裝完成時，若要完成註冊︰

1. 從 Windows [開始] 功能表啟動名為「Microsoft Azure 復原服務殼層」的應用程式。
2. 在開啟的命令視窗中，執行下列命令組來設定 Proxy 伺服器設定。

		PS C:\Windows\System32> $pwd = ConvertTo-SecureString -String ProxyUserPassword
		PS C:\Windows\System32> Set-OBMachineSetting -ProxyServer http://myproxyserver.domain.com -ProxyPort PortNumb – ProxyUserName domain\username -ProxyPassword $pwd
		PS C:\Windows\System32> net stop obengine.exe

參數：

- / ServerMode：必要。指定應該同時安裝組態和處理序伺服器，還是只安裝處理序伺服器。輸入值：CS、PS
- InstallLocation：必要。在其中安裝元件的資料夾。
- /MySQLCredsFilePath。必要。儲存 MySQL 伺服器認證的檔案路徑。檔案格式應該如下：
	- [MySQLCredentials]
	- MySQLRootPassword = "<Password>"
	- MySQLUserPassword = "<Password>"
- /VaultCredsFilePath。必要。保存庫認證檔的位置
- /EnvType。必要。安裝類型。值：VMware、NonVMware
- /PSIP 和 /CSIP。必要。處理序伺服器和組態伺服器 IP 位址。
- /PassphraseFilePath。必要。複雜密碼檔案的位置。
- /BypassProxy。選用。指定組態伺服器不使用 Proxy 連接至 Azure。
- /ProxySettingsFilePath。選用。Proxy 設定 (預設 Proxy 需要驗證或自訂的 Proxy)。檔案格式應該如下：
	- [ProxySettings]
	- ProxyAuthentication = "Yes/No"
	- Proxy IP = "IP Address>"
	- ProxyPort = "<連接埠>"
	- ProxyUserName="<使用者名稱>"
	- ProxyPassword="<密碼>"
- DataTransferSecurePort。選用。要用於複寫資料的連接埠號碼。
- SkipSpaceCheck。選用。略過快取的空間檢查。
- AcceptThirdpartyEULA。必要。旗標表示接受協力廠商使用者授權合約。
- ShowThirdpartyEULA。必要。顯示協力廠商使用者授權合約。如果提供作為輸入，則會忽略所有其他參數。

### 加入用於自動探索的 VMware 帳戶

 當您準備部署時，您應該已[建立 VMware 帳戶](#prepare-an-account-for-automatic-discovery)，讓 Site Recovery 可用來進行自動探索。加入此帳戶，如下所示︰

1. 開啟 **CSPSConfigtool.exe**。它會在桌面上的捷徑，位於 [安裝位置]\\home\\svsystems\\bin 資料夾。
2. 按一下 [管理帳戶] > [加入帳戶]。

	![新增帳戶](./media/site-recovery-vmware-to-azure/credentials1.png)

3. 在 [帳戶詳細資料] 中，新增將用來進行自動探索的帳戶。請注意，可能需要 15 分鐘以上，帳戶名稱才會出現在入口網站。若要立即更新，請按一下 [組態伺服器] > 伺服器名稱 > [重新整理伺服器]。

	![詳細資料](./media/site-recovery-vmware-to-azure/credentials2.png)

### 連線到 vSphere 主機與 vCenter 伺服器

如果您要複寫 VMware VM，請連線到 vCenter 伺服器和 vSphere 主機。

1. 驗證組態伺服器具有 vSphere 主機和 vCenter 伺服器的網路存取。
2. 按一下 [準備基礎結構] > [來源]。在 [準備來源] 中，選取組態伺服器，然後按一下 [+vCenter] 來新增 vSphere 主機或 vCenter 伺服器。
3. 在 [加入 vCenter] 中，指定 vSphere 主機或 vCenter 伺服器的易記名稱，並指定伺服器的 IP 位址或 FQDN。保留連接埠為 443，除非您的 VMware 伺服器設定為在不同連接埠上接聽要求。然後選取將用於連線至 VMware 伺服器的帳戶。按一下 [確定]。

	![VMware](./media/site-recovery-vmware-to-azure/vmware-server.png)

	>[AZURE.NOTE] 如果您要以沒有 vCenter 或主機伺服器系統管理員權限的帳戶加入 vCenter 伺服器或 vSphere 主機，則確定帳戶已啟用這些權限：資料中心、資料存放區、資料夾、主機、網路、資源、虛擬機器、vSphere 分散式交換器。此外，vCenter 伺服器需要儲存體檢視權限。

Site Recovery 會使用您指定的設定連線到 VMware 伺服器並探索 VM。

## 步驟 3︰設定目標環境

驗證您有儲存體帳戶可用於複寫，以及容錯移轉後 Azure VM 將與其連線的 Azure 網路。

1.	按一下 [準備基礎結構] > [目標]，然後選取您要使用的 Azure 訂用帳戶。
2.	指定您要在容錯移轉後用於 VM 的部署模型。
3.	Site Recovery 會檢查您是否有一或多個相容的 Azure 儲存體帳戶和網路。

	![目標](./media/site-recovery-vmware-to-azure/gs-target.png)

4.	如果您尚未建立儲存體帳戶而想要使用 ARM 建立一個帳戶，請按一下 [+儲存體帳戶] 以內嵌方式執行該作業。在 [建立儲存體帳戶] 刀鋒視窗中，指定帳戶名稱、類型、訂用帳戶和位置。此帳戶應位於與復原服務保存庫相同的區域。

	![儲存體](./media/site-recovery-vmware-to-azure/gs-createstorage.png)

	請注意：

	- 如果您想要使用傳統模型建立儲存體帳戶，請在 Azure 入口網站中執行該作業。[深入了解](../storage/storage-create-storage-account-classic-portal.md)
	- 如果您將進階儲存體帳戶使用於複寫的資料，則必須設定其他標準儲存體帳戶來儲存複寫記錄檔，而這類記錄檔會擷取內部部署資料的進行中變更。
	
	> [AZURE.NOTE] 目前在印度中部與印度南部不支援對進階儲存體帳戶提供保護。

4.	選取 Azure 網路。如果您尚未建立網路而想要使用 ARM 建立網路，請按一下 [+網路] 以內嵌方式執行該作業。在 [建立虛擬網路] 刀鋒視窗上，指定網路名稱、位址範圍、子網路詳細資料、訂用帳戶和位置。此網路應位於與復原服務保存庫相同的位置。

	![網路](./media/site-recovery-vmware-to-azure/gs-createnetwork.png)

	如果您想要使用傳統模型建立網路，請在 Azure 入口網站中執行該作業。[深入了解](../virtual-network/virtual-networks-create-vnet-classic-pportal.md)。

## 步驟 4：設定複寫設定

1. 若要建立新的複寫原則，請按一下 [準備基礎結構] > [複寫設定] > [+建立及關聯]。
2. 在 [建立及關聯原則] 中指定原則名稱。
3. 在 [RPO 臨界值] 中，指定 RPO 限制。連續複寫超過此限制時，會產生警示。
5. 在 [復原點保留] 中，針對每個復原點指定保留週期的長度 (以小時為單位)。受保護的機器可以復原到週期內的任意點。針對複寫到進階儲存體的機器支援最多 24 小時的保留。
6. 在 [應用程式一致快照頻率] 中，指定建立包含應用程式一致快照之復原點的頻率 (以分鐘為單位)。
7. 當您建立複寫原則時，依預設會針對容錯回復自動建立比對原則。例如，如果複寫原則是 **rep-policy**，容錯回復原則便會是 **rep-policy-failback**。在您起始容錯回復之前，不會使用此原則。
8. 按一下 [確定] 以建立原則。

	![複寫原則](./media/site-recovery-vmware-to-azure/gs-replication2.png)

9. 當您建立新的原則時，該原則會自動與組態伺服器產生關聯。按一下 [確定]。

	![複寫原則](./media/site-recovery-vmware-to-azure/gs-replication3.png)


## 步驟 5︰容量規劃

您現已設定您的基本基礎結構，您可以思考容量規劃並找出您是否需要額外的資源。

Site Recovery 會提供容量規劃工具，協助您為來源環境、Site Recovery 元件、網路和儲存體配置適當的資源。您可以在快速模式中執行規劃工具，以便根據 VM、磁碟和儲存體的平均數量進行估計，或在詳細模式中執行規劃工具，以輸入工作負載層級的數據。開始之前，您必須︰

- 收集有關複寫環境的資訊，包括 VM、每個 VM 的磁碟和每個磁碟的儲存體。
- 估計複寫資料的每日變更 (流失) 率。您可以使用 [vSphere Replication Capacity Planning Appliance (vSphere 容量規劃應用裝置)](https://labs.vmware.com/flings/vsphere-replication-capacity-planning-appliance) 來協助您執行這項操作。

1.	按一下 [下載] 來下載此工具並加以執行。閱讀工具隨附的[文章](site-recovery-capacity-planner.md)。
2.	當您完成時，在 [是否已完成容量規劃?] 中選取 [是]。

	![容量規劃](./media/site-recovery-vmware-to-azure/gs-capacity-planning.png)

下表擷取可協助您在這種情況下進行容量規劃的一些重點。


**元件** | **詳細資料**
--- | --- | ---
**複寫** | **每日變更率上限** - 受保護的機器只能使用一部處理序伺服器，而且單一處理序伺服器可處理的每日變更率最多為 2 TB。因此 2 TB 是針對受保護機器支援的每日資料變更率上限。<br/><br/> **最大輸送量** - 複寫的機器可以屬於 Azure 中的一個儲存體帳戶。標準儲存體帳戶每秒可處理最多 20000 個要求，建議您將來源機器的 IOPS 數保持為 20000。例如，如果您有一部具備 5 個磁碟的來源機器，並且在來源上的每個磁碟會產生 120 個 IOP (8K 大小)，則它會在 Azure 每個磁碟 IOPS 限制 500 之內。儲存體帳戶所需的數目 = 來源 IOP 總計/20000。
**組態伺服器** | 組態伺服器應該要能夠處理在受保護機器上執行之所有工作負載的每日變更率容量，因此需要足夠頻寬以持續地將資料複寫到 Azure 儲存體。<br/><br/> 我們建議的最佳做法是組態伺服器與您想要保護的機器位於相同網路與 LAN 區段上。它可以位於不同的網路，但是您想要保護的機器應該具有 L3 網路可見性。<br/><br/> 下表摘要說明組態伺服器的大小建議。
**處理序伺服器** | 組態伺服器上會安裝第一部處理序伺服器。您可以部署額外的處理序伺服器來調整您的環境。注意：<br/><br/>處理序伺服器會從受保護的機器接收複寫資料，然後以快取、壓縮及加密將資料最佳化之後，才傳送給 Azure。處理序伺服器機器應該要有足夠的資源來執行這些工作。<br/><br/> 處理序伺服器使用磁碟快取。我們建議每個快取磁碟有 600 GB 以上的空間，以處理發生網路瓶頸或中斷時儲存的資料變更。

### 組態伺服器的大小建議

**CPU** | **記憶體** | **快取磁碟大小** | **資料變更率** | **受保護的機器**
--- | --- | --- | --- | ---
8 個 vCPU (2 個通訊端 * 四核心 @ 2.5GHz) | 16 GB | 300 GB | 500 GB 或更少 | 複寫少於 100 部機器。
12 個 vCPU (2 個通訊端 * 六核心 @ 2.5GHz) | 18 GB | 600 GB | 500 GB 至 1 TB | 複寫 100-150 部機器。
16 個 vCPU (2 個通訊端 * 八核心 @ 2.5GHz) | 32 GB | 1 TB | 1 TB 至 2 TB | 複寫 150-200 部機器。
部署另一個處理序伺服器 | | | > 2 TB | 如果您要複寫 200 部以上的機器，或如果每日資料變更率超過 2 TB，部署額外的處理序伺服器。

其中：

- 每個來源機器已設定各 100 GB 的 3 個磁碟。
- 我們使用具有 RAID 10 的 8 個 10 K RPM 的 SAS 磁碟機的效能評定儲存體以進行快取磁碟度量。

### 處理序伺服器的大小建議

如果您要保護超過 200 部機器，或每日變更率大於 2 TB，您可以加入額外的處理序伺服器來處理複寫負載。若要擴充，您可以：

- 增加組態伺服器的數目。例如，您可以使用兩部組態伺服器保護最多 400 部機器。
- 加入額外的處理序伺服器並使用它們來處理流量，以取代 (或搭配) 組態伺服器。

下表描述案例，其中：

- 您不打算使用組態伺服器作為處理序伺服器。
- 您已設定額外的處理序伺服器。
- 您已設定受保護的虛擬機器，以使用額外的處理序伺服器。
- 每個受保護的來源機器已設定各 100 GB 的 3 個磁碟。

**組態伺服器** | **額外處理序伺服器**| **快取磁碟大小** | **資料變更率** | **受保護的機器**
--- | --- | --- | --- | ---
8 個 vCPU (2 個通訊端 * 四核心 @ 2.5GHz)，16 GB 記憶體 | 4 個 vCPU (2 個通訊端 * 雙核心 @ 2.5GHz)，8 GB 記憶體 | 300 GB | 250 GB 或更少 | 複寫 85 部或更少的機器。
8 個 vCPU (2 個通訊端 * 四核心 @ 2.5GHz)，16 GB 記憶體 | 8 個 vCPU (2 個通訊端 * 四核心 @ 2.5GHz)，12 GB 記憶體 | 600 GB | 250 GB 至 1 TB | 複寫 85-150 部機器。
12 個 vCPU (2 個通訊端 * 六核心 @ 2.5GHz)，18 GB 記憶體 | 12 個 vCPU (2 個通訊端 * 六核心 @ 2.5GHz)，24 GB 記憶體 | 1 TB | 1 TB 至 2 TB | 複寫 150-225 部機器。


您調整您的伺服器的方式取決於您的相應增加或相應放大模型的喜好設定。您部署幾個高階組態和處理序伺服器以相應增加，或使用較少的資源部署更多伺服器以相應放大。例如：如果您需要保護 220 部機器，您可以執行下列任一項：

- 設定具有 12 個 vCPU、18 GB 記憶體的組態伺服器、具有 12 個 vCPU、24 GB 記憶體的額外處理序伺服器，並將受保護的機器設定為僅使用額外的處理序伺服器。
- 或者，您也可以設定兩部組態伺服器 (2 x 8vCPU、16 GB RAM) 和兩部額外的處理序伺服器 (1 x 8vCPU 和 4vCPU x 1 以處理 135 + 85 (220) 部機器)，並將受保護的機器設定為僅使用額外的處理序伺服器。

[遵循這些指示](#deploy-additional-process-servers)以設定額外的處理序伺服器。

### 網路頻寬考量

您可以使用容量規劃工具來計算複寫 (初始複寫，而後是差異複寫) 所需的頻寬 。若要控制複寫所用的頻寬數量，您有幾個選項可用︰

- **節流頻寬**︰複寫至 Azure 的 VMware 流量會經過特定的處理序伺服器。您可在執行作為處理序伺服器的機器上進行頻寬節流。
- **影響頻寬**︰您可以使用幾個登錄機碼來影響用於複寫的頻寬：
	- **HKEY\_LOCAL\_MACHINE\\SOFTWARE\\Microsoft\\Windows Azure Backup\\UploadThreadsPerVM** 登錄值可指定用於磁碟資料傳輸 (初始或差異複寫) 的執行緒數目。較高的值可增加複寫所用的網路頻寬。
	- **HKEY\_LOCAL\_MACHINE\\SOFTWARE\\Microsoft\\Windows Azure Backup\\DownloadThreadsPerVM** 可指定在容錯回復期間用於資料傳輸的執行緒數目。

#### 節流頻寬

1. 在作為處理序伺服器的機器上開啟 Microsoft Azure 備份 MMC 嵌入式管理單元。根據預設，Microsoft Azure 備份的捷徑位於桌面上或在 C:\\Program Files\\Microsoft Azure Recovery Services Agent\\bin\\wabadmin 中。
2. 在嵌入式管理單元中，按一下 [變更屬性]。

	![節流頻寬](./media/site-recovery-vmware-to-azure/throttle1.png)

3. 在 [節流] 索引標籤上，選取 [啟用備份操作的網際網路頻寬使用節流設定]，然後設定工作和非工作時數的限制。有效範圍是每秒 512 Kbps 到 102 Mbps。

	![節流頻寬](./media/site-recovery-vmware-to-azure/throttle2.png)

您也可以使用 [Set-OBMachineSetting](https://technet.microsoft.com/library/hh770409.aspx) Cmdlet 來設定節流。以下是一個範例：

    $mon = [System.DayOfWeek]::Monday
    $tue = [System.DayOfWeek]::Tuesday
    Set-OBMachineSetting -WorkDay $mon, $tue -StartWorkHour "9:00:00" -EndWorkHour "18:00:00" -WorkHourBandwidth  (512*1024) -NonWorkHourBandwidth (2048*1024)

**Set-OBMachineSetting -NoThrottle** 表示不需要節流。


#### 影響網路頻寬

1. 在登錄中瀏覽至 **HKEY\_LOCAL\_MACHINE\\SOFTWARE\\Microsoft\\Windows Azure Backup\\Replication**。
	- 若要影響複製磁碟上的頻寬流量，請修改 **UploadThreadsPerVM** 的值，如果不存在則請建立機碼。
	- 若要影響從 Azure 容錯回復流量的頻寬，請修改 **DownloadThreadsPerVM** 的值。
2. 預設值為 4。在 “overprovisioned” 網路中，這些登錄機碼必須變更自其預設值。最大值為 32。監視流量，將此值最佳化。

## 步驟 6︰複寫應用程式

請確定您想要複寫的機器已準備好安裝行動服務，然後啟用複寫。

### 安裝行動服務

啟用虛擬機器和實體伺服器保護的第一個步驟是安裝行動服務。您可以使用下列幾種方式來執行這個動作：

- **處理序伺服器推送**︰當您在機器上啟用複寫時，請從處理序伺服器推送並安裝行動服務元件。請注意，如果機器已經執行最新版本的元件，便不會進行推送安裝。
- **企業推送**：使用您的企業推送處理程序 (例如 WSUS 或 System Center Configuration Manager) 來自動安裝元件。請先設定組態伺服器，再執行這項操作。
- **手動安裝**︰在您想要複寫的每一部機器上手動安裝元件。請先設定組態伺服器，再執行這項操作。


#### 準備在 Windows 機器上自動推入

以下是如何準備 Windows 機器，讓處理序伺服器可以自動安裝行動服務。

1.  建立可以由處理序伺服器用來存取機器的帳戶。帳戶應該具備系統管理員權限 (本機或網域)，並且僅用於推送安裝。

	>[AZURE.NOTE] 如果您未使用網域帳戶，您必須停用本機電腦上的遠端使用者存取控制。若要這樣做，請在登錄的 HKEY\_LOCAL\_MACHINE\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Policies\\System 下加入 DWORD 項目 LocalAccountTokenFilterPolicy，其值為 1。若要從 CLI 新增登錄項目，請輸入 **`REG ADD HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1`**。

2.  在您想要保護之機器的 Windows 防火牆上，選取 [允許應用程式或功能通過防火牆]。啟用 [檔案及印表機共用] 和 [Windows Management Instrumentation]。對於隸屬於網域的機器，您可以使用 GPO 設定防火牆設定。

	![防火牆設定](./media/site-recovery-vmware-to-azure/mobility1.png)

2. 加入您所建立的帳戶：

	- 開啟 **cspsconfigtool**。它會在桌面上的捷徑，位於 [安裝位置]\\home\\svsystems\\bin 資料夾。
	- 在 [管理帳戶] 索引標籤中，按一下 [新增帳戶]。
	- 加入您所建立的帳戶。加入帳戶之後，當您啟用機器的複寫時，必須提供認證。


#### 準備在 Linux 伺服器上自動推入

1.	確定您要保護的 Linux 機器受到支援，如[受保護的機器必要條件](#protected-machine-prerequisites)中所述。確定 Linux 機器和處理序伺服器之間有網路連線。

2.	建立可以由處理序伺服器用來存取機器的帳戶。帳戶應該是來源 Linux 伺服器上的根使用者，並且僅用於推送安裝。

	- 開啟 **cspsconfigtool**。它會在桌面上的捷徑，位於 [安裝位置]\\home\\svsystems\\bin 資料夾。
	- 在 [管理帳戶] 索引標籤中，按一下 [新增帳戶]。
	- 加入您所建立的帳戶。加入帳戶之後，當您啟用機器的複寫時，必須提供認證。

3.	檢查來源 Linux 伺服器上的 /etc/hosts 檔案包含將本機主機名稱對應到所有網路介面卡相關聯之 IP 位址的項目。
4.	在您想要複寫的機器上安裝最新的 openssh、openssh 伺服器、openssl 封裝。
5.	請確定 SSH 已啟用且正在連接埠 22 上執行。
6.	在 sshd\_config 檔案中啟用 SFTP 子系統與密碼驗證，如下所示：

	- 以 root 的身分登入。
	- 在 /etc/ssh/sshd\_config 檔案中，尋找以 **PasswordAuthentication** 為開頭的行。
	- 取消該行的註解並將值從 **no** 變更為 **yes**。
	- 尋找以 **Subsystem** 為開頭的行並取消其註解。

		![Linux](./media/site-recovery-vmware-to-azure/mobility2.png)


#### 手動安裝行動服務

安裝程式位於處理序伺服器上的 **C:\\Program Files (x86)\\Microsoft Azure Site Recovery\\home\\svsystems\\pushinstallsvc\\repository** 中。

來源作業系統 | 行動服務安裝檔案
--- | ---
Windows Server (僅限 64 位元) | Microsoft-ASR\_UA\_9.*.0.0_Windows_* release.exe
CentOS 6.4、6.5、6.6 (僅限 64 位元) | Microsoft-ASR\_UA\_9.*.0.0\_RHEL6-64\_*release.tar.gz
SUSE Linux Enterprise Server 11 SP3 (僅限 64 位元) | Microsoft-ASR\_UA\_9.*.0.0\_SLES11-SP3-64\_*release.tar.gz
Oracle Enterprise Linux 6.4、6.5 (僅限 64 位元) | Microsoft-ASR\_UA\_9.*.0.0\_OL6-64\_*release.tar.gz


#### 在 Windows 伺服器上手動安裝


1. 下載並執行相關安裝程式。
2. 在 [開始之前] 中，選取 [行動服務]。

	![行動服務](./media/site-recovery-vmware-to-azure/mobility3.png)

3. 在 [組態伺服器詳細資料] 中，指定組態伺服器的 IP 位址，以及您執行「統一安裝」時所產生的複雜密碼。您可以在組態伺服器上執行下列命令來取得複雜密碼：**<SiteRecoveryInstallationFolder>\\home\\sysystems\\bin\\genpassphrase.exe –v**。

	![行動服務](./media/site-recovery-vmware-to-azure/mobility6.png)

4. 在 [安裝位置] 中，保留預設設定，然後按 [下一步] 來開始安裝。
5. 在 [安裝進度] 中，監視安裝的進度，並在系統提示時重新啟動機器。安裝服務之後可能需要大約 15 分鐘，狀態才會更新在入口網站中。

您也可以從命令列安裝：

UnifiedAgent.exe [/Role <代理程式/主要目標>] [/InstallLocation <安裝目錄>] [/CSIP <用於註冊的 CS IP 位址>] [/PassphraseFilePath <複雜密碼檔案路徑>] [/LogFilePath <記錄檔路徑>]

其中：

- /Role：必要。指定是否應該要安裝行動服務。
- /InstallLocation：必要。指定安裝服務的位置。
- /PassphraseFilePath：必要。組態伺服器複雜密碼。
- /LogFilePath：必要。記錄安裝檔案位置。

#### 手動針對行動服務進行解除安裝

行動服務可以透過使用 [控制台] 中的 [新增或移除程式]，或是使用命令列來進行解除安裝。

使用命令列針對行動服務進行解除安裝的命令是

	MsiExec.exe /qn /x {275197FC-14FD-4560-A5EB-38217F80CBD1}


#### 在 Linux 伺服器上手動安裝：

1. 根據上表，將適當的 tar 封存檔複製到您要複寫的 Linux 機器。
2. 執行下列命令來開啟殼層程式，並將壓縮的 tar 封存檔解壓縮到本機路徑：`tar -xvzf Microsoft-ASR_UA_8.5.0.0*`
3. 在解壓縮 tar 封存檔內容的本機目錄中建立 passphrase.txt 檔案。若要執行這項操作，請在組態伺服器上從 C:\\ProgramData\\Microsoft Azure Site Recovery\\private\\connection.passphrase 複製複雜密碼，然後在殼層中執行 *`echo <passphrase> >passphrase.txt`* 以將它儲存在 passphrase.txt 中。
4. 執行 *`sudo ./install -t both -a host -R Agent -d /usr/local/ASR -i <IP address> -p <port> -s y -c https -P passphrase.txt`* 來安裝行動服務。
5. 指定組態伺服器的內部 IP 位址，並確定已選取連接埠 443。安裝服務之後可能需要大約 15 分鐘，狀態才會更新在入口網站中。

**您也可以從命令列安裝**：

1. 在組態伺服器上從 C:\\Program Files (x86)\\InMage Systems\\private\\connectio 複製複雜密碼，並且在組態伺服器上將它儲存為 "passphrase.txt"。然後執行以下命令。在我們的範例中，組態伺服器的 IP 位址是 104.40.75.37 且 HTTPS 連接埠應該是 443：

若要在實際執行伺服器上安裝：

    ./install -t both -a host -R Agent -d /usr/local/ASR -i 104.40.75.37 -p 443 -s y -c https -P passphrase.txt

若要在主要目標伺服器上安裝：


    ./install -t both -a host -R MasterTarget -d /usr/local/ASR -i 104.40.75.37 -p 443 -s y -c https -P passphrase.txt


### 啟用複寫

#### 開始之前

如果您要複寫 VMware 虛擬機器，請注意下列各項︰

- 系統會每隔 15 分鐘探索 VMware VM 一次，而且在探索之後，可能需要 15 分鐘以上，VMware VM 才會出現在入口網站中。同樣地，當您加入新的 vCenter 伺服器或 vSphere 主機時，探索可能需要 15 分鐘以上。
- 虛擬機器上的環境變更 (例如 VMware 工具安裝) 也可能需要 15 分鐘以上的時間，才會在入口網站更新。
- 您可以在 [組態伺服器] 刀鋒視窗上 vCenter 伺服器/vSphere 主機的 [上次連絡時間] 欄位中，查看 VMware VM 的上次探索時間。
- 若要新增要複寫的機器而不等候已排定的探索，請反白選取組態伺服器 (不要按它)，然後按一下 [重新整理] 按鈕。
- 當您啟用複寫時，如果機器已準備好，處理序伺服器會自動在其上安裝行動服務。

#### 從複寫排除磁碟

當您啟用複寫時，依預設會複寫機器上的所有磁碟。您可以從複寫排除磁碟。例如，您可能不想要複寫具有暫存資料的磁碟，或是每次機器或應用程式重新啟動時便重新整理的資料 (例如 pagefile.sys 或 SQL Server tempdb)。如果您想要排除磁碟，請注意︰

- 您只能排除已經安裝行動服務的磁碟。您將需要[手動安裝行動服務](#install-the-mobility-service-manually)，因為在啟用複寫後，只會透過推送機制來安裝行動服務。
- 只有基本磁碟可以從複寫排除。您無法排除 OS 或動態磁碟。
- 啟用複寫後，您無法加入或移除複寫的磁碟。如果您想要加入或排除磁碟，必須停用機器的保護，然後重新啟用它。
- 如果您排除應用程式運作所需的磁碟，在容錯移轉至 Azure 之後，您將必須在 Azure 中手動建立它，複寫的應用程式才能執行。或者，您可以將 Azure 自動化整合至復原計畫，在機器容錯移轉期間建立磁碟。
- 您以手動方式在 Azure 中建立的磁碟會容錯回復。例如，如果您容錯移轉三個磁碟，並直接在 Azure 中建立兩個磁碟，則五個磁碟全都將容錯回復。您無法從容錯回復排除手動建立的磁碟。

**立即啟用複寫，如下所示**︰

1. 按一下 [步驟 2: 複寫應用程式] > [來源]。第一次啟用複寫之後，請按一下保存庫中的 [+複寫]，以對其他機器啟用複寫。
2. 在 [來源] 刀鋒視窗 > [來源] 中，選取組態伺服器。
3. 在 [機器類型] 中，選取 [虛擬機器] 或 [實體機器]。
4. 在 [vCenter/vSphere Hypervisor] 中，選取管理 vSphere 主機的 vCenter 伺服器，或選取主機。如果您是複寫實體機器，則這個設定不相關。
5. 選取處理序伺服器。如果您尚未建立任何額外的處理序伺服器，這將是組態伺服器的名稱。然後按一下 [確定]。

	![啟用複寫](./media/site-recovery-vmware-to-azure/enable-replication2.png)

6. 在 [目標] 中，選取保存庫訂用帳戶，並在 [容錯移轉後部署模型] 中，選取您想要在容錯移轉後於 Azure 中使用的模型 (傳統或資源管理)。
7. 選取您要用來複寫資料的 Azure 儲存體帳戶。請注意：

	- 您可以選取進階或標準儲存體帳戶。如果選取進階帳戶，您需要針對進行中的複寫記錄檔，額外指定一個標準儲存體帳戶。帳戶必須位於與復原服務保存庫相同的區域中。
	- 如果您想使用與現有不同的儲存體帳戶，您可以[建立一個](#set-up-an-azure-storage-account)。若要使用 ARM 模型來建立儲存體帳戶，請按一下 [新建]。如果您想要使用傳統模型建立儲存體帳戶，請在 [Azure 入口網站](../storage/storage-create-storage-account-classic-portal.md)中執行該作業。

8. 選取 Azure VM 在容錯移轉後啟動時所要啟動的 Azure 網路和子網路。此網路必須位於與復原服務保存庫相同的區域中。選取 [立即設定選取的機器]，將網路設定套用至您選取要進行保護的所有機器。選取 [稍後設定] 以選取每部機器的 Azure 網路。如果您沒有網路，則必須[建立一個](#set-up-an-azure-network)。若要使用 ARM 模型建立網路，請按一下 [新建]。如果您想要使用傳統模型建立網路，請在 [Azure 入口網站](../virtual-network/virtual-networks-create-vnet-classic-pportal.md)中執行該作業。選取適用的子網路。然後按一下 [確定]。

	![啟用複寫](./media/site-recovery-vmware-to-azure/enable-replication3.png)

9. 在 [虛擬機器] > [選取虛擬機器] 中，按一下並選取您要複寫的每部機器。您只能選取可以啟用複寫的機器。然後按一下 [確定]。

	![啟用複寫](./media/site-recovery-vmware-to-azure/enable-replication5.png)

10. 在 [屬性] > [設定屬性] 中，選取處理序伺服器將用來在機器上自動安裝行動服務的帳戶。依預設會複寫所有磁碟。按一下 [所有磁碟]，然後將任何您不想要複寫的磁碟取消選取。然後按一下 [確定]。您可以稍後再設定其他屬性。

	![啟用複寫](./media/site-recovery-vmware-to-azure/enable-replication6.png)

11. 在 [複寫設定] > [設定複寫設定] 中，確認已選取正確的複寫原則。您可以在 [設定] > [複寫原則] > 原則名稱 > [編輯設定] 中修改複寫原則設定。套用到原則的變更將會套用到複寫和新的機器。

12. 如果您想要將機器聚集成一個複寫群組，請啟用 [多部 VM 一致性]，並指定群組的名稱。然後按一下 [確定]。請注意：

	- 複寫群組中的機器會一起複寫，並且在容錯移轉時會有共用的損毀一致和應用程式一致的復原點。
	- 我們建議您將 VM 與實體伺服器一起收集，讓它們鏡像您的工作負載。啟用多部 VM 一致性可能會影響工作負載的效能，應該只用於機器正在執行相同工作負載，且您需要一致性的情況。

	![啟用複寫](./media/site-recovery-vmware-to-azure/enable-replication7.png)

13. 按一下 [啟用複寫]。您可以在 [設定] > [作業] > [Site Recovery 作業] 中，追蹤 [啟用保護] 作業的進度。執行 [完成保護] 作業之後，機器即準備好進行容錯移轉。

> [AZURE.NOTE] 如果機器已準備好進行推送安裝，則當啟用保護時，將會安裝行動服務元件。在機器上安裝元件之後，保護工作會啟動並失敗。在失敗之後，您需要手動重新啟動每一部機器。重新啟動之後，保護作業再次開始，並且會發生初始複寫。

### 檢視及管理 VM 屬性

建議您確認來源機器的屬性。請記住，Azure VM 名稱應該符合 [Azure 虛擬機器需求](site-recovery-best-practices.md#azure-virtual-machine-requirements)。

1. 按一下 [設定] > [複寫的項目]，然後選取機器。[程式集] 刀鋒視窗會顯示機器設定與狀態的相關資訊。

2. 在 [屬性] 中，您可以檢視 VM 的複寫和容錯移轉資訊。

	![啟用複寫](./media/site-recovery-vmware-to-azure/test-failover2.png)

3. 在 [計算和網路] > [計算屬性] 中，您可以指定 Azure VM 名稱和目標大小。視需要修改名稱以符合 Azure 需求。您也可以檢視和加入目標網路、子網路的相關資訊，以及將指派給 Azure VM 的 IP 位址。請注意：

	- 您可以設定目標 IP 位址。如果您未提供地址，則容錯移轉的機器會使用 DHCP。如果您設定的位址在容錯移轉時無法使用，則容錯移轉會失敗。如果位址可用於測試容錯移轉網路，則相同的目標 IP 位址可用於測試容錯移轉。
	- 網路介面卡的數目會視您指定給目標虛擬機器的大小而有所不同，如下所示：

		- 如果來源電腦上的網路介面卡數目小於或等於針對目標機器大小所允許的介面卡數目，則目標將具備與來源相同的介面卡數目。
		- 如果來源虛擬機器的介面卡數目超過針對目標大小所允許的數目，則將使用目標大小的最大值。
		- 例如，如果來源機器具有兩張網路介面卡，而目標機器大小支援四張，則目標機器將會有兩張介面卡。如果來源機器具有兩張介面卡，但支援的目標大小僅支援一張，則目標機器將只會有一張介面卡。
	- 如果 VM 有多張網路介面卡，則全部會連接至相同的網路。

	![啟用複寫](./media/site-recovery-vmware-to-azure/test-failover4.png)

4. 在 [磁碟] 中，您可以看見 VM 上將要複寫的作業系統和資料磁碟。


## 步驟 7：測試部署

為測試部署，您可以針對單一虛擬機器執行測試容錯移轉，或執行包含一或多部虛擬機器的復原計劃。


### 準備容錯移轉

- 若要測試容錯移轉，建議您建立與您的 Azure 生產網路分隔的新 Azure 網路 (這是您在 Azure 中建立新網路時的預設行為)。[深入了解](site-recovery-failover.md#run-a-test-failover)如何執行測試容錯移轉。
- 若要在容錯移轉至 Azure 時獲得最佳效能，請在受保護的機器上安裝 Azure 代理程式。這可讓開機變快速，並協助進行疑難排解。安裝 [Linux](https://github.com/Azure/WALinuxAgent) 或 [Windows](http://go.microsoft.com/fwlink/?LinkID=394789) 代理程式。
- 若要完整測試您的部署，您需要有基礎結構，讓複寫的機器如預期般運作。如果您想要測試 Active Directory 和 DNS，您可以透過 DNS 建立虛擬機器做為網域控制站，並使用 Azure Site Recovery 將此虛擬機器複寫至 Azure。深入了解 [Active Directory 的測試容錯移轉考量](site-recovery-active-directory.md#considerations-for-test-failover)。
- 請確定組態伺服器正在執行。否則容錯移轉將會失敗。
- 如果您已從複寫排除磁碟，您可能需要在容錯移轉之後以手動方式在 Azure 中建立磁碟，讓應用程式能如預期般執行。
- 如果您想要執行非計劃性容錯移轉，而不是測試容錯移轉，請注意下列事項︰

	- 如果可能的話您應該在執行非計劃性容錯移轉之前關閉主要機器。這可確保您不需要同時執行來源和複本機器。如果您是複寫 VMware VM，則可以指定 Site Recovery 應該要盡全力關閉來源機器。根據主要網站的狀態，可能可以運作。如果您是複寫實體伺服器，則 Site Recovery 不提供此選項。
	- 當您執行非計劃性容錯移轉時，會停止從主要機器的資料複寫，讓任何資料差異不會在開始非計劃性容錯移轉之後傳送。此外，如果您在復原方案上執行非計劃性容錯移轉，它將會執行直到完成為止 (即使發生錯誤)。

### 準備在容錯移轉後連接到 Azure VM

如果您想要在容錯移轉後使用 RDP 連接到 Azure VM，請確定執行下列作業︰

**在容錯移轉前的內部部署機器上**：

- 針對透過網際網路的存取啟用 RDP，確定已針對 [公用] 新增 TCP 和 UDP 規則，並確定在 [Windows 防火牆] -> [允許的應用程式和功能] 中已針對所有設定檔允許 RDP。
- 針對透過站對站連線的存取在機器上啟用 RDP，並確定在 [Windows 防火牆] -> [允許的應用程式和功能] 中已針對 [網域] 和 [私人] 網路允許 RDP。
- 在內部部署機器上安裝 [Azure VM 代理程式](http://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409)。
- 在機器上[手動安裝行動服務](#install-the-mobility-service-manually)，而不使用處理序伺服器來自動推送服務。這是因為只有在機器已啟用複寫後，才會進行推送安裝。
- 確定作業系統的 SAN 原則已設為 OnlineAll。[深入了解](https://support.microsoft.com/kb/3031135)
- 在執行容錯移轉前，關閉 IPSec 服務。

**在容錯移轉後的 Azure VM 上**：

- 新增 RDP 通訊協定 (連接埠 3389) 的公用端點並指定用於登入的認證。
- 請確定沒有任何網域原則禁止您使用公用位址連接到虛擬機器。
- 嘗試連線。如果無法連線，請確認 VM 正在執行中。如需其他疑難排解提示，請閱讀這篇[文章](http://social.technet.microsoft.com/wiki/contents/articles/31666.troubleshooting-remote-desktop-connection-after-failover-using-asr.aspx)。


如果您想要在容錯移轉後使用安全殼層用戶端 (ssh) 存取執行 Linux 的 Azure VM，請執行下列作業︰

**在容錯移轉前的內部部署機器上**：

- 確定 Azure VM 上的安全殼層服務已設定為在系統開機時自動啟動。
- 請檢查防火牆規則是否允許 SSH 連線。

**在容錯移轉後的 Azure VM 上**：

- 已容錯移轉的 VM 上的網路安全性群組規則以及它所連接的 Azure 子網路必須允許 SSH 連接埠的連入連線。
- 應建立公用端點，以允許 SSH 連接埠 (預設為 TCP 通訊埠 22) 上的連入連線。
- 如果透過 VPN 連線 (Express Route 或站對站 VPN) 存取 VM，則用戶端可用來透過 SSH 直接連接到 VM。

**在容錯移轉後的 Azure Windows/Linux VM 上**：

如果您有與虛擬機器或機器所屬子網路相關聯的網路安全性群組，請確定網路安全性群組具有 輸出規則可允許 HTTP/HTTPS。也請確定虛擬機器要容錯移轉至的網路 DNS 設定正確。否則容錯移轉可能逾時，並發生錯誤 -「PreFailoverWorkflow 工作 WaitForScriptExecutionTask 逾時」。若要詳細了解這點，請參閱[監視和疑難排解指南](site-recovery-monitoring-and-troubleshooting.md#recovery)中的＜復原＞一節。

## 執行測試容錯移轉

1. 若要容錯移轉單一機器，請在 [設定] > [複寫的項目] 中，按一下 VM > [+測試容錯移轉] 圖示。

	![測試容錯移轉](./media/site-recovery-vmware-to-azure/test-failover1.png)

2. 若要容錯移轉復原方案，請在 [設定] > [復原方案] 中，以滑鼠右鍵按一下方案 > [測試容錯移轉]。若要建立復原方案，請[遵循這些指示](site-recovery-create-recovery-plans.md)。

3. 在 [測試容錯移轉] 中，選取 Azure VM 在容錯移轉之後要連接的 Azure 網路。
4. 按一下 [確定] 即可開始容錯移轉。您可以按一下 VM 以開啟其屬性，或在保存庫名稱 > [設定] > [作業] > [Site Recovery 作業] 中的 [測試容錯移轉] 作業上按一下，來追蹤進度。
5. 當容錯移轉到達 [完成測試] 狀態時，請執行下列作業：

	1. 在 Azure 入口網站中檢視複本虛擬機器。確認虛擬機器成功啟動。
	2. 如果您設定從內部部署網路存取虛擬機器，您可以初始化虛擬機器的「遠端桌面」連線。
	3. 按一下 [完成測試] 來完成它。

		![測試容錯移轉](./media/site-recovery-vmware-to-azure/test-failover6.png)


	4. 按一下 [記事] 記錄並儲存關於測試容錯移轉的任何觀察。
	5. 按一下 [測試容錯移轉已完成]，即可自動清除測試環境。此動作完成之後，測試容錯移轉將會顯示 [完成] 狀態。
	6.  在此階段，會刪除 Site Recovery 在測試容錯移轉期間自動建立的所有元素或 VM。不會刪除任何您為測試容錯移轉建立的其他元素。

	> [AZURE.NOTE] 如果測試容錯移轉持續兩週以上，系統即會強制完成該測試容錯移轉。


6. 容錯移轉完成之後，您應該也會看到複本 Azure 機器出現在 Azure 入口網站 > [虛擬機器] 中。您應該確定 VM 為適當的大小、已連接到適當的網路，而且正在執行中。
7. 如果您[已準備好容錯移轉後的連線](#prepare-to-connect-to-azure-vms-after-failover)，您應該能夠連接到 Azure VM。

## 監視您的部署

以下是監視 Site Recovery 部署的組態設定、狀態和健康狀態的方式︰

1. 按一下保存庫名稱來存取 [基本資訊] 儀表板。在此儀表板中，您可以看見 Site Recovery 作業、複寫狀態、復原方案、伺服器健康狀態和事件。您可以自訂 [基本資訊] 以顯示對您最有用的圖格和配置，包括其他 Site Recovery 和「備份」保存庫的狀態。
<br> 
![基本資訊](./media/site-recovery-vmware-to-azure/essentials.png)

2. 在 [健全狀況] 圖格中，您可以監視發生問題的站台伺服器 (VMM 或組態伺服器)，以及 Site Recovery 在過去 24 小時內引發的事件。
3. 您可以在 [複寫的項目]、[復原方案] 和 [Site Recovery 作業] 圖格中管理和監視複寫。您可以在 [設定] -> [作業] -> [Site Recovery 作業] 中向下鑽研作業。


## 部署額外處理序伺服器

如果您必須相應放大您的部署超過 200 部來源機器，或每日變換率總計超過 2 TB，您將需要額外的處理序伺服器來處理流量。

檢查[處理序伺服器的大小建議](#size-recommendations-for-the-process-server)，然後遵循這些指示來設定處理序伺服器。設定伺服器之後，您會移轉來源機器以便使用它。

### 安裝額外的處理序伺服器

1. 在 [設定] > [Site Recovery 伺服器] 中，按一下組態伺服器 > [處理序伺服器]。

	![新增處理序伺服器](./media/site-recovery-vmware-to-azure/migrate-ps1.png)

2. 在 [伺服器類型] 中，按一下 [處理序伺服器 (內部部署)]。

	![新增處理序伺服器](./media/site-recovery-vmware-to-azure/migrate-ps2.png)

3. 下載 Site Recovery 統一安裝檔案，然後執行它以安裝處理序伺服器，並在保存庫中註冊。
4. 在 [開始之前] 中，選取 [新增額外處理序伺服器以相應放大部署]。
5. 以您[設定](#step-2-set-up-the-source-environment)組態伺服器時的相同方式完成精靈。

	![新增處理序伺服器](./media/site-recovery-vmware-to-azure/add-ps1.png)

6. 在 [組態伺服器詳細資料] 中，指定組態伺服器的 IP 位址，以及複雜密碼。若要取得複雜密碼，請在組態伺服器上執行：**<SiteRecoveryInstallationFolder>\\home\\sysystems\\bin\\genpassphrase.exe –n**。

	![新增處理序伺服器](./media/site-recovery-vmware-to-azure/add-ps2.png)

### 移轉機器以使用新的處理序伺服器

1. 在 [設定] > [Site Recovery 伺服器] 中，按一下組態伺服器，然後展開 [處理序伺服器]。

	![更新處理序伺服器](./media/site-recovery-vmware-to-azure/migrate-ps2.png)

2. 以滑鼠右鍵按一下目前使用中的處理序伺服器，然後按一下 [切換]。

	![更新處理序伺服器](./media/site-recovery-vmware-to-azure/migrate-ps3.png)

3. 在 [選取目標處理序伺服器] 中，選取您要使用的新處理序伺服器，然後選取新處理序伺服器將處理的虛擬機器。按一下資訊圖示以取得伺服器的相關資訊。為了協助您進行負載的判斷，會顯示將每個選取的虛擬機器複寫到新的處理序伺服器所需的平均空間。按一下核取記號以開始複寫到新處理序伺服器。

## VMware 帳戶權限

處理序伺服器可以自動探索 vCenter 伺服器上的 VM。若要執行自動探索，您將需要[定義角色 (Azure\_Site\_Recovery)](#prepare-an-account-for-automatic-discovery)，以允許 Site Recovery 存取 VMware 伺服器。請注意，如果您只需要將 VMware 機器移轉至 Azure，而不需要從 Azure 容錯回復，您可以定義唯讀角色，這個角色已足夠。下表將摘要列出必要的角色權限。

**角色** | **詳細資料** | **權限**
--- | --- | ---
Azure\_Site\_Recovery 角色 | VMware VM 探索 |針對 v-Center 伺服器指派這些權限：<br/><br/>資料存放區 -> 配置空間、瀏覽資料存放區、低階檔案作業、移除檔案、更新虛擬機器檔案<br/><br/>網路 -> 網路指派<br/><br/>資源 -> 將虛擬機器指派給資源集區、移轉已關閉電源的虛擬機器、移轉已開啟電源的虛擬機器<br/><br/>工作 -> 建立工作、更新工作<br/><br/>虛擬機器 -> 組態<br/><br/>虛擬機器 -> 互動 -> 回答問題、裝置連線、設定 CD 媒體、設定軟碟機媒體、關閉電源、開啟電源、VMware 工具安裝<br/><br/>虛擬機器 -> 清查 -> 建立、註冊、取消註冊<br/><br/>虛擬機器 -> 佈建 -> 允許虛擬機器下載、允許虛擬機器檔案上傳<br/><br/>虛擬機器 -> 快照 -> 移除快照
vCenter 使用者角色 | VMware VM 探索/容錯移轉而不關閉來源 VM | 針對 v-Center 伺服器指派這些權限：<br/><br/>資料中心物件 –> 傳播至子物件、角色=唯讀 <br/><br/>由於是在資料中心層級指派使用者，因此使用者能夠存取資料中心內的所有物件。如果您想要限制存取權，請將具備 [傳播至子物件] 權限的 [沒有存取權] 角色指派給子物件 (vSphere 主機、資料存放區、VM 及網路)。
vCenter 使用者角色 | 容錯移轉和容錯回復 | 針對 v-Center 伺服器指派這些權限：<br/><br/>資料中心物件 – 傳播至子物件、角色=Azure\_Site\_Recovery<br/><br/>由於是在資料中心層級指派使用者，因此使用者能夠存取資料中心內的所有物件。如果您想要限制存取權，請將具備 [傳播至子物件] 權限的 [沒有存取權] 角色指派給子物件 (vSphere 主機、資料存放區、VM 及網路)。  
## 後續步驟

- [深入了解](site-recovery-failover.md)不同類型的容錯移轉。
- [深入了解容錯回復](site-recovery-failback-azure-to-vmware.md)，以便將在 Azure 中容錯移轉的機器回復到您的內部部署環境。

## 第三方廠商軟體注意事項和資訊

Do Not Translate or Localize

The software and firmware running in the Microsoft product or service is based on or incorporates material from the projects listed below (collectively, “Third Party Code”).Microsoft is the not original author of the Third Party Code.The original copyright notice and license, under which Microsoft received such Third Party Code, are set forth below.

The information in Section A is regarding Third Party Code components from the projects listed below.Such licenses and information are provided for informational purposes only.This Third Party Code is being relicensed to you by Microsoft under Microsoft's software licensing terms for the Microsoft product or service.

The information in Section B is regarding Third Party Code components that are being made available to you by Microsoft under the original licensing terms.

The complete file may be found on the [Microsoft Download Center](http://go.microsoft.com/fwlink/?LinkId=529428).Microsoft reserves all rights not expressly granted herein, whether by implication, estoppel or otherwise.

<!---HONumber=AcomDC_0921_2016-->