<properties
	pageTitle="使用 Site Recovery 在內部部署 Hyper-V 虛擬機器與 Azure (沒有 VMM) 之間複寫 | Microsoft Azure"
	description="本文說明如何使用 Azure Site Recovery，將不在 VMM 雲端中管理的 Hyper-V 虛擬機器複寫至 Azure。"
	services="site-recovery"
	documentationCenter=""
	authors="rayne-wiselman"
	manager="jwhit"
	editor=""/>

<tags
	ms.service="site-recovery"
	ms.devlang="na"
	ms.topic="article"
	ms.tgt_pltfrm="na"
	ms.workload="storage-backup-recovery"
	ms.date="09/19/2016"
	ms.author="raynew"/>


# 使用 Azure Site Recovery 在內部部署 Hyper-V 虛擬機器與 Azure (沒有 VMM) 之間複寫

> [AZURE.SELECTOR]
- [Azure 入口網站](site-recovery-hyper-v-site-to-azure.md)
- [PowerShell - ARM](site-recovery-deploy-with-powershell-resource-manager.md)
- [傳統入口網站](site-recovery-hyper-v-site-to-azure-classic.md)

閱讀本文，以了解當 Hyper-V 主機不在 System Center Virtual Machine Manager (VMM) 雲端中受管理時，如何部署 Site Recovery 來複寫 Hyper-V 虛擬機器至 Azure。

本文將摘要說明部署的必要條件，可協助您設定複寫設定，並啟用虛擬機器的保護。最後它會測試容錯移轉，藉此確定一切如預期般運作。

在閱讀本文之後，請在下方或 [Azure Recovery Services Forum (Azure 復原服務論壇)](https://social.msdn.microsoft.com/forums/azure/home?forum=hypervrecovmgr) 中張貼任何意見或問題。


## 概觀

組織需要商務持續性和災害復原 (BCDR) 策略，決定應用程式、工作負載和資料如何在規劃與未規劃停機期間維持運作，並儘速復原到正常運作的情況。BCDR 策略的解決方案重點集中在保護商務資料安全且可復原，以及當發生災害時工作負載持續可用。

Site Recovery 是一項 Azure 服務，可藉由將內部部署實體伺服器和虛擬機器的複寫協調至雲端 (Azure) 或次要資料中心，協助您的 BCDR 策略。當您的主要位置發生故障時，您容錯移轉至次要網站，讓應用程式和工作負載保持可用。當它恢復正常作業時，容錯回復至您的主要位置。

Site Recovery 可以用在許多案例中，並可保護許多工作負載。深入了解[什麼是 Azure Site Recovery？](site-recovery-overview.md)


## Azure 必要條件

- 您將需要 [Microsoft Azure](https://azure.microsoft.com/) 帳戶。您可以從[免費試用](https://azure.microsoft.com/pricing/free-trial/)開始。
- 您需要 Azure 儲存體帳戶來儲存複寫的資料。此帳戶必須啟用異地複寫。它應該與 Azure Site Recovery 保存庫位於相同的區域，且和同一個訂用帳戶產生關聯。[深入了解 Azure 儲存體](../storage/storage-introduction.md)。請注意，我們不支援使用[新的 Azure 入口網站](../storage/storage-create-storage-account.md)來跨資源群組移動所建立的儲存體帳戶。
- 您將需要 Azure 虛擬網路，如此一來，當您從主要網站容錯移轉時，Azure 虛擬機器就會連接至網路。

## Hyper-V 的必要條件

- 在來源內部部署網站中，您將需要一部或多部執行已安裝 Hyper-V 角色之 Windows Server 2012 R2 的伺服器。此伺服器應該：
- 包含一或多部虛擬機器。
- 直接或透過 Proxy 連接到網際網路。
- 執行 KB [2961977](https://support.microsoft.com/zh-TW/kb/2961977 "KB2961977") 中所述的修正。

## 虛擬機器先決條件

您想要保護的虛擬機器必須符合 [Azure 虛擬機器需求](site-recovery-best-practices.md#azure-virtual-machine-requirements)。

## 提供者和代理程式的必要條件

在 Azure Site Recovery 部署期間，您將在每部 Hyper-V 伺服器上，安裝 Azure Site Recovery Provider 和 Azure 復原服務代理程式。請注意：

- 建議您一律執行最新版本的提供者和代理程式。這些程式可以在 Site Recovery 入口網站中取得。
- 保存庫中的所有 Hyper-V 伺服器應該有相同版本的提供者和代理程式。
- 伺服器上執行的提供者會透過網際網路連接到 Site Recovery。您不需要使用 Poxy 就能選擇執行這個動作，方法是使用目前設定於 Hyper-V 伺服器上的 Poxy 設定，或使用您在提供者安裝期間所設定的自訂 Poxy 設定。您必須確定您想要使用的 Proxy 伺服器可以存取這些 URL 以連接到 Azure：
	- *.hypervrecoverymanager.windowsazure.com
	- *.accesscontrol.windows.net
	- *.backup.windowsazure.com
	- *.blob.core.windows.net
	- *.store.core.windows.net
	
- 此外，允許 [Azure 資料中心 IP 範圍](https://www.microsoft.com/download/details.aspx?id=41653)中所述的 IP 位址和 HTTPS (443) 通訊協定。您必須具有打算使用以及美國西部之 Azure 區域的允許名單 IP 範圍。


下圖顯示 Site Recovery 為協調流程與複寫所使用的不同通訊通道與連接埠

![B2A 拓樸](./media/site-recovery-hyper-v-site-to-azure-classic/b2a-topology.png)


## 步驟 1：建立保存庫

1. 登入[管理入口網站](https://portal.azure.com)。


2. 展開 [資料服務] > [復原服務]，然後按一下 [Site Recovery 保存庫]。


3. 按一下 [新建] > [快速建立]。

4. 在 [**名稱**] 中，輸入保存庫的易記識別名稱。

5. 在 [地區] 中，選取保存庫的地理區域。若要查看支援的區域，請參閱 [Azure Site Recovery 定價詳細資料](https://azure.microsoft.com/pricing/details/site-recovery/)中的＜各地區上市情況＞。

6. 按一下 [建立保存庫]。

	![新增保存庫](./media/site-recovery-hyper-v-site-to-azure-classic/vault.png)

檢查狀態列，以確認是否順利建立保存庫。保存庫在主要復原服務頁面上會列為 [使用中]。


## 步驟 2：建立 Hyper-V 站台

1. 在 [復原服務] 頁面中，按一下保存庫以開啟 [快速入門] 頁面。您也可以使用圖示隨時開啟 [快速啟動]。

	![快速啟動](./media/site-recovery-hyper-v-site-to-azure-classic/quick-start-icon.png)

2. 在下拉式清單中，選取 [在內部部署 Hyper-V 站台與 Azure 之間]。

	![Hyper-V 站台案例](./media/site-recovery-hyper-v-site-to-azure-classic/select-scenario.png)

3. 在 [建立 Hyper-V 站台] 中，按一下 [建立 Hyper-V 站台]。指定站台名稱並儲存。

	![Hyper-V 站台](./media/site-recovery-hyper-v-site-to-azure-classic/create-site.png)


## 步驟 3：安裝提供者和代理程式
在具有您想要保護的 VM 的每部 Hyper-V 伺服器上，安裝提供者和代理程式。

如果您正在 Hyper-V 叢集上進行安裝，請在容錯移轉叢集中的每個節點上執行步驟 5-11 。在註冊所有節點並啟用保護之後，虛擬機器將會受到保護，即使它們在叢集中的節點間進行移轉也一樣。

1. 在 [準備 Hyper-V 伺服器] 中，按一下 [下載註冊金鑰] 檔案。
2. 在 [下載註冊金鑰] 頁面上，按一下站台旁邊的 [下載]。將金鑰下載到 Hyper-V 伺服器容易存取的安全位置。該金鑰在產生後會維持 5 天有效。

	![註冊金鑰](./media/site-recovery-hyper-v-site-to-azure-classic/download-key.png)

4. 按一下 [下載提供者] 來取得最新版本。
5. 在您想要於保存庫中註冊的每一部 Hyper-V 伺服器上執行該檔案。該檔案會安裝兩個元件：
	- **Azure Site Recovery Provider** - 處理 Hyper-V 伺服器與 Azure Site Recovery 入口網站之間的通訊和協調流程。
	- **Azure 復原服務代理程式** - 處理在來源 Hyper-V 伺服器上執行之虛擬機器與 Azure 儲存體之間的資料傳輸。
6. 在 [Microsoft Update] 中，您可以選擇加入以取得更新。啟用這項設定時，會根據您的 Microsoft Update 原則來安裝提供者和代理程式更新。

	![Microsoft Update](./media/site-recovery-hyper-v-site-to-azure-classic/provider1.png)

7. 在 [安裝] 中，指定要將提供者和代理程式安裝到 Hyper-V 伺服器上的哪個位置。

	![安裝位置](./media/site-recovery-hyper-v-site-to-azure-classic/provider2.png)

8. 安裝完成之後，請繼續設定以在保存庫中註冊伺服器。

9. 在 [保存庫設定] 頁面上，按一下 [瀏覽] 來選取金鑰檔案。指定 Azure Site Recovery 訂用帳戶、保存庫名稱，以及 Hyper-V 伺服器所屬的 Hyper-V 網站。

	![伺服器註冊](./media/site-recovery-hyper-v-site-to-azure-classic/provider8.PNG)

10. 在 [網際網路連線] 頁面上，您將指定提供者連接到 Azure Site Recovery 的方式。選取 [Use default system proxy settings]，以使用在伺服器上設定的預設網際網路連線設定。如果未指定值，將使用預設設定。

	![網際網路設定](./media/site-recovery-hyper-v-site-to-azure-classic/provider7.PNG)

11. 註冊作業會開始在保存庫中註冊伺服器。

	![伺服器註冊](./media/site-recovery-hyper-v-site-to-azure-classic/provider15.PNG)

11. 註冊完成之後，Azure Site Recovery 便會抓取來自 Hyper-V 伺服器的中繼資料，而該伺服器會顯示在保存庫中 [伺服器] 頁面的 [Hyper-V 網站] 索引標籤上 。


### 從命令列安裝提供者

或者，您可以從命令列安裝 Azure Site Recovery 提供者。如果您想要在執行 Windows Server Core 2012 R2 的電腦上安裝提供者，您應該使用這個方法。從命令列執行，如下所示：

1. 將提供者安裝檔案和註冊金鑰下載至資料夾。例如 C:\\ASR。
2. 以系統管理員身分開啟 [命令提示字元] 並輸入：

    	C:\Windows\System32> CD C:\ASR
    	C:\ASR> AzureSiteRecoveryProvider.exe /x:. /q

3. 然後執行下列命令以安裝提供者：

		C:\ASR> setupdr.exe /i

4. 執行下列命令以完成註冊：

    	CD C:\Program Files\Microsoft Azure Site Recovery Provider
    	C:\Program Files\Microsoft Azure Site Recovery Provider> DRConfigurator.exe /r  /Friendlyname <friendly name of the server> /Credentials <path of the credentials file> /EncryptionEnabled <full file name to save the encryption certificate>         

參數包括：

- **/Credentials**：指定您下載的註冊金鑰的位置。
- **/FriendlyName**：指定名稱以識別 Hyper-V 主機伺服器。這個名稱會出現在入口網站中。
- **/EncryptionEnabled**：選擇性。指定是否要在 Azure 中加密複本虛擬機器 (靜態加密)。
- **/proxyAddress**；**/proxyport**；**/proxyUsername**；**/proxyPassword**：選擇性。如果您想要使用自訂 proxy，或您現有的 proxy 需要驗證，請指定 proxy 參數。


## 步驟 4：建立 Azure 儲存體帳戶 

1. 在 [準備資源] 中選取 [建立儲存體帳戶]，以建立 Azure 儲存體帳戶 (如果您沒有儲存體帳戶)。此帳戶應啟用異地複寫。它應該與 Azure Site Recovery 保存庫位於相同的區域，並與同一個訂用帳戶建立關聯。

	![建立儲存體帳戶](./media/site-recovery-hyper-v-site-to-azure-classic/create-resources.png)

>[AZURE.NOTE] 1.我們不支援使用[新的 Azure 入口網站](../storage/storage-create-storage-account.md)來跨資源群組移動所建立的儲存體帳戶。2.對於用於部署 Site Recovery 的儲存體帳戶，不支援跨相同訂用帳戶內的資源群組或跨訂用帳戶[移轉儲存體帳戶](../resource-group-move-resources.md)。

## 步驟 5：建立和設定保護群組

保護群組是虛擬機器的邏輯群組，您想要使用相同的保護設定來保護這類群組。您只要將保護設定套用至保護群組，這些設定就會套用至您新增至該群組中的所有虛擬機器。

1. 在 [建立和設定保護群組] 中，按一下 [建立保護群組]。如果有任何必要條件不符，系統就會發出訊息，您可以按一下 [檢視詳細資料] 來取得詳細資訊。

2. 在 [保護群組] 索引標籤中，新增保護群組。指定名稱、來源 Hyper-V 網站、目標 **Azure**、您的 Azure Site Recovery 訂用帳戶名稱，以及 Azure 儲存體帳戶。

	![保護群組](./media/site-recovery-hyper-v-site-to-azure-classic/protection-group.png)


2. 在 [複寫設定] 中，設定 [複製頻率]，來指定應在來源和目標之間同步處理資料差異的頻率。您可以設定為 30 秒、5 分鐘或 15 分鐘。
3. 在 [保留復原點] 中，指定復原歷程記錄應該儲存多少個小時。
4. 在 [應用程式一致快照的頻率] 中，您可以指定是否要建立快照，使用「磁碟區陰影複製服務」(VSS) 來確保建立快照時，應用程式是處於一致狀態。預設不會建立這些快照。請確定已將此值設定為小於您設定的其他復原點數目。只有當虛擬機器正在執行 Windows 作業系統時，才支援此功能。
5. 在 [初始複寫開始時間] 中，指定何時應將保護群組中虛擬機器的初始複寫傳送到 Azure。

	![保護群組](./media/site-recovery-hyper-v-site-to-azure-classic/protection-group2.png)


## 步驟 6：啟用虛擬機器保護


將虛擬機器新增到保護群組，為其啟用保護。

>[AZURE.NOTE] 不支援保護使用靜態 IP 位址執行 Linux 的 VM。

1. 在保護群組的 [機器] 索引標籤上，按一下 [將虛擬機器加入保護群組以啟用保護]。
2. 在 [啟用虛擬機器保護] 頁面上，選取要保護的虛擬機器。

	![啟用虛擬機器保護](./media/site-recovery-hyper-v-site-to-azure-classic/add-vm.png)

	「啟用保護」工作隨即開始。您可以在 [工作] 索引標籤上追蹤進度。執行「完成保護」工作之後，虛擬機器即準備好進行容錯移轉。
3. 設定保護之後，您可以：

	- 在 [受保護的項目] > [保護群組] > *protectiongroup\_name* > [虛擬機器] 中檢視虛擬機器。您可以在 [屬性] 索引標籤中向下切入到機器詳細資料。
	- 在 [受保護的項目] > [保護群組] > *protectiongroup\_name* > [虛擬機器] *virtual\_machine\_name* > [設定] 中，設定虛擬機器的容錯移轉屬性。您可以設定：
		- **名稱**：Azure 中的虛擬機器名稱。
		- **大小**：要容錯移轉之虛擬機器的目標大小。

		![設定虛擬機器屬性](./media/site-recovery-hyper-v-site-to-azure-classic/vm-properties.png)
	- 在 [受保護的項目]* > *[保護群組]* > *protectiongroup\_name * > *[虛擬機器]* > *virtual\_machine\_name* > *[設定]* 中，設定其他虛擬機器設定，包括：

		- **網路介面卡**：網路介面卡的數目取決於您針對目標虛擬機器所指定的大小。查看[虛擬機器大小規格](../virtual-machines/virtual-machines-linux-sizes.md#size-tables)，了解虛擬機器大小所支援的 NIC 數目。


			在修改虛擬機器的大小並儲存設定之後，當您下次開啟 [設定] 頁面時，網路介面卡的數量將會改變。目標虛擬機器的網路介面卡數目，是來源虛擬機器上的網路介面卡數目下限，以及所選虛擬機器大小支援的網路介面卡數目上限。其說明如下：


			- 如果來源電腦上的網路介面卡數目小於或等於針對目標機器大小所允許的介面卡數目，則目標將具備與來源相同的介面卡數目。
			- 如果來源虛擬機器的介面卡數目超過針對目標大小所允許的數目，則將使用目標大小的最大值。
			- 例如，如果來源機器具有兩張網路介面卡，而目標機器大小支援四張，則目標機器將會有兩張介面卡。如果來源機器具有兩張介面卡，但支援的目標大小僅支援一張，則目標機器將只會有一張介面卡。
		- **Azure 網路**：指定應將虛擬機器容錯移轉到其中的目標網路。如果虛擬機器具有多張網路介面卡，則所有的介面卡都應該連接到同一個 Azure 網路。
		- **子網路**：針對虛擬機器上的每張網路介面卡，請選取 Azure 網路中的子網路，機器在容錯移轉之後應會連接到該子網路。
		- **目標 IP 位址**：如果來源虛擬機器的網路介面卡已設定為使用靜態 IP 位址，則您可以指定目標虛擬機器的 IP 位址，以確保機器在容錯移轉後會有相同的 IP 位址。如果您未指定 IP 位址，則系統將在容錯移轉期間指派任何可用的位址。如果您指定了使用中的位址，則容錯移轉將會失敗。
		
        > [AZURE.NOTE] [Migration of networks](../resource-group-move-resources.md) 對於用於部署 Site Recovery 的網路，不支援跨相同訂用帳戶內的資源群組或跨訂用帳戶。

		![設定虛擬機器屬性](./media/site-recovery-hyper-v-site-to-azure-classic/multiple-nic.png)




## 步驟 7：建立復原計畫

為測試部署，您可以針對單一虛擬機器執行測試容錯移轉，或執行包含一或多部虛擬機器的復原計劃。[深入了解](site-recovery-create-recovery-plans.md)如何建立復原計劃。

## 步驟 8：測試部署

有兩種方式可以測試容錯移轉至 Azure。

- **在沒有 Azure 網路的情況下測試容錯移轉**—這種測試容錯移轉可以檢查虛擬機器是否正確地出現在 Azure 中。在容錯移轉之後，虛擬機器不會連線到任何 Azure 網路。
- **利用 Azure 網路測試容錯移轉**—這種測試容錯移轉可以檢查整個復寫環境是否如預期般出現並進行容錯移轉，虛擬機器會連線到只訂的目標 Azure 網路。請注意，針對測試容錯移轉，可根據複本虛擬機器的子網路得知測試虛擬機器的子網路。這和一般的複寫不同，一般複寫的複本虛擬機器子網路是根據來源虛擬機器的子網路得知。

如果您想要執行測試容錯轉移，卻不想指定 Azure 網路，您不需要作任何準備。

若要以目標 Azure 網路執行測試容錯移轉，您必須建立與您的 Azure 正式作業網路 (當您在 Azure 中建立新網路時的預設行為) 分隔的新的 Azure 網路。如需詳細資訊，請參閱[執行測試容錯移轉](site-recovery-failover.md#run-a-test-failover)。


若要完整測試您的複寫和網路部署，您必須針對複寫虛擬機器設定基礎結構，以如預期般運作。其中一個方式是將虛擬機器設定為具有 DNS 的網域控制站，並且使用 Site Recovery 將其複寫至 Azure，以藉由執行測試容錯移轉在測試網路中建立它。[深入了解](site-recovery-active-directory.md#considerations-for-test-failover) Active Directory 的測試容錯移轉考量。

執行測試容錯移轉，如下所示：

>[AZURE.NOTE] 若要在執行容錯移轉至 Azure 時獲得最佳效能，請確保您已經在受保護的機器上安裝 Azure 代理程式。這有助於更快速開機，也有助於診斷發生的問題。您可以在[這裡](https://github.com/Azure/WALinuxAgent)找到 Linux 代理程式，而 Windows 代理程式則可在[這裡](http://go.microsoft.com/fwlink/?LinkID=394789)找到

1. 在 [復原計畫] 索引標籤上，選取計畫，然後按一下 [測試容錯移轉]。
2. 在 [確認測試容錯移轉] 頁面上，選取 [無] 或特定的 Azure 網路。請注意，如果選取 [無]，測試容錯移轉將會檢查虛擬機器是否會正確複寫到 Azure，但並不會檢查您的複寫網路組態。

	![測試容錯移轉](./media/site-recovery-hyper-v-site-to-azure-classic/test-nonetwork.png)

3. 在 [工作] 索引標籤上，您可以追蹤容錯移轉進度。您應該可以在 Azure 入口網站中看到該虛擬機器測試複本。如果您設定從內部部署網路存取虛擬機器，您可以初始化虛擬機器的「遠端桌面」連線。
4. 當容錯移轉到達 [完成測試] 階段時，請按一下 [完成測試] 以完成測試容錯移轉。您可以向下切入到 [工作] 索引標籤，來追蹤容錯移轉進度和狀態，並執行任何所需的動作。
5. 在容錯移轉之後，您將可以在 Azure 入口網站中看到該虛擬機器測試複本。如果您設定從內部部署網路存取虛擬機器，您可以初始化虛擬機器的「遠端桌面」連線。

	1. 確認虛擬機器成功啟動。
    2. 如果您在容錯移轉之後要使用遠端桌面連接到 Azure 中的虛擬機器，請先在虛擬機器上啟用遠端桌面連線，再執行測試容錯移轉。您也必須在虛擬機器上新增 RDP 端點。您可以利用 [Azure 自動化 Runbook](site-recovery-runbook-automation.md) 來執行這個動作。
    3. 容錯移轉之後，如果您使用公用 IP 位址，利用 [遠端桌面] 連接到 Azure 中的虛擬機器，請確定您沒有任何網域原則會阻止您使用公用位址連接到虛擬機器。

6. 測試完成之後，請執行下列動作：

	- 按一下 [測試容錯移轉完成]。清除測試環境，以自動關閉電源及刪除測試虛擬機器。
	- 按一下 [記事] 記錄並儲存關於測試容錯移轉的任何觀察。
7. 當容錯移轉到達 [完成測試] 階段時，請依下列方式完成驗證：
	1. 在 Azure 入口網站中檢視複本虛擬機器。確認虛擬機器成功啟動。
	2. 如果您設定從內部部署網路存取虛擬機器，您可以初始化虛擬機器的「遠端桌面」連線。
	3. 按一下 [完成測試] 來完成它。
	4. 按一下 [記事] 記錄並儲存關於測試容錯移轉的任何觀察。
	5.  按一下 [測試容錯移轉完成]。清除測試環境，以自動關閉電源及刪除測試虛擬機器。

## 後續步驟

在您的部署設定完成並開始執行之後，[深入了解](site-recovery-failover.md)容錯移轉。

<!---HONumber=AcomDC_0921_2016-->