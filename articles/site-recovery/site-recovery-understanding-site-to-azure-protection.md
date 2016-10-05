<properties
	pageTitle="了解如何使用 Azure Site Recovery 進行 Hyper-V 複寫 | Microsoft Azure" 
	description="使用本文來了解一些技術概念，協助您順利安裝、設定和管理 Azure Site Recovery。" 
	services="site-recovery" 
	documentationCenter="" 
	authors="anbacker" 
	manager="mkjain" 
	editor=""/>

<tags 
	ms.service="site-recovery" 
	ms.devlang="na"
	ms.topic="article"
	ms.tgt_pltfrm="na"
	ms.workload="storage-backup-recovery" 
	ms.date="09/12/2016" 
	ms.author="anbacker"/>
 

# 了解如何使用 Azure Site Recovery 進行 Hyper-V 複寫

本文說明的技術概念，可協助您透過 Azure Site Recovery，順利設定和管理 Azure 保護的 Hyper-V 網站或 VMM 網站。

## 了解元件

### 用於在內部部署和 Azure 之間複寫的 Hyper-V 網站或 VMM 網站部署。
 
在內部部署和 Azure 之間設定 DR 的一部分；Azure Site Recovery 提供者必須下載並安裝在 VMM 伺服器上，連同 Azure 復原服務代理程式，它必須安裝在每個 Hyper-V 主機上。

![用於在內部部署和 Azure 之間複寫的 VMM 網站部署。](media/site-recovery-understanding-site-to-azure-protection/image00.png)

Hyper-V 網站部署與 VMM 部署相同，唯一的差異在於，提供者和代理程式都安裝在 Hyper-V 主機本身。

## 了解工作流程

### 啟用保護
一旦您從入口網站或內部部署保護虛擬機器，名為「啟用保護」的 ASR 工作就會啟動，您也可以在 [工作] 索引標籤中監視此工作。

![疑難排解內部部署 Hyper-V 問題](media/site-recovery-understanding-site-to-azure-protection/image001.PNG)

「啟用保護」工作會在叫用 [CreateReplicationRelationship](https://msdn.microsoft.com/library/hh850036.aspx) 之前檢查必要條件，再使用保護期間設定的輸入，建立對 Azure 的複寫。「啟用保護」工作會叫用 [StartReplication](https://msdn.microsoft.com/library/hh850303.aspx)，將虛擬機器的虛擬磁碟傳送至 Azure，開始從內部部署進行初始複寫。

![疑難排解內部部署 Hyper-V 問題](media/site-recovery-understanding-site-to-azure-protection/IMAGE002.PNG)

### 完成保護
系統會在初始複寫觸發時擷取 [Hyper-V VM 快照](https://technet.microsoft.com/library/dd560637.aspx)。系統會逐一處理虛擬硬碟，直到所有磁碟都上傳至 Azure 為止。視磁碟大小和頻寬而定，這項作業通常要經過一段時間才能完成。請參閱[如何管理內部部署對 Azure 保護的網路頻寬使用量](https://support.microsoft.com/kb/3056159)，了解如何將網路使用量最佳化。當初始複寫「完成虛擬機器上的保護」工作後，會設定網路和複寫後設定。雖然初始複寫正在進行，但系統會如下方的「差異複寫」一節中所述追蹤磁碟的所有變更。初始複寫正在進行時，系統會使用額外的磁碟存放裝置放置快照和 HRL 檔案。初始複寫完成之後，Hyper-V VM 快照便會被刪除，使得初始複寫後的資料變更合併至父系磁碟。

![疑難排解內部部署 Hyper-V 問題](media/site-recovery-understanding-site-to-azure-protection/image03.png)

### 差異複寫
Hyper-V 複本複寫追蹤器是 Hyper-V 複本複寫引擎的一部分，會追蹤虛擬硬碟的變更，並記錄在 Hyper-V 複寫記錄檔 (*.hrl) 中。HRL 檔案會位於與相關聯磁碟的相同目錄中。每個設定用於複寫的磁碟都會有相關聯的 HRL 檔案。此記錄檔會在初始複寫完成後，傳送至客戶的儲存體帳戶。當記錄檔傳輸至 Azure 時，會在同一目錄內另一個記錄檔中追蹤主要複本中的變更。

如[監視虛擬機器的複寫健全狀況](./site-recovery-monitoring-and-troubleshooting.md#monitor-replication-health-for-virtual-machine)中所述，在 VM 檢視中可以監視在初始複寫或差異複寫期間的 VM 複寫健康情況。

### 重新同步處理 
當差異複寫失敗，且完整初始複寫會大量使用網路頻寬，或大幅增加完成完整初始複寫所需的時間時，系統就會標示虛擬機器以便重新同步處理。例如，當 HRL 檔案大小累積達總磁碟大小的 50% 時，系統就會標示虛擬機器以便重新同步處理。重新同步處理會計算來源和目標虛擬機器磁碟的總和檢查碼，並只傳送差異部分，藉此有效減少透過網路傳送的資料量。

重新同步處理完成之後，應會繼續進行一般差異複寫。重新同步處理可以在中斷後繼續 (例如網路中斷、VMMS 當機等)。

根據預設，系統會在非辦公期間設定「自動排程的重新同步處理」。如果需要以手動方式重新同步處理虛擬機器，請從入口網站中選取虛擬機器，然後按一下 [重新同步處理]。

![疑難排解內部部署 Hyper-V 問題](media/site-recovery-understanding-site-to-azure-protection/image04.png)

重新同步處理會使用固定區塊的區塊演算法，其中的來源和目標檔案會分成數個固定區塊；系統會產生每個區塊的總和檢查碼，然後進行比較，以判斷來源的哪些區塊必須套用至目標。

### 重試邏輯
發生複寫錯誤時，會啟動內建的重試邏輯。錯誤可分以下兩種類別。

| 類別 | 案例 |
|---------------------------|----------------------------------------------|
| 無法復原的錯誤 | 不會嘗試任何重試。虛擬機器複寫狀態會顯示為「重大」，而且需要管理員介入處理。範例包括<ul><li>中斷的 VHD 鏈結</li><li>複本虛擬機器處於無效狀態</li><li>網路驗證錯誤</li><li>授權錯誤</li><li>如果在獨立的 Hyper-V 伺服器中找不到虛擬機器</li></ul>|
| 可復原的錯誤 | 系統會使用指數方式的輪詢，從第一次嘗試開始增加重試間隔 (1、2、4、8、10 分鐘)，在每個複寫間隔中進行重試。如果錯誤持續發生，會每隔 30 分鐘重試一次。範例包括<ul><li>網路錯誤</li><li>磁碟空間太少</li><li>記憶體太少</li></ul>|

## 了解 Hyper-V 虛擬機器保護和復原的生命週期

![了解 Hyper-V 虛擬機器保護和復原檔案生命週期](media/site-recovery-understanding-site-to-azure-protection/image05.png)

## 其他參考資料

- [監視和疑難排解 VMware、VMM、Hyper-V 和實體網站的保護](./site-recovery-monitoring-and-troubleshooting.md)
- [連絡 Microsoft 支援](./site-recovery-monitoring-and-troubleshooting.md#reaching-out-for-microsoft-support)
- [常見的 ASR 錯誤及其解決方案](./site-recovery-monitoring-and-troubleshooting.md#common-asr-errors-and-their-resolutions)

<!---HONumber=AcomDC_0921_2016-->