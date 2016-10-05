<properties
	pageTitle="移難排解 Azure 虛擬機器備份 | Microsoft Azure"
	description="Azure 虛擬機器備份與還原的疑難排解"
	services="backup"
	documentationCenter=""
	authors="trinadhk"
	manager="shreeshd"
	editor=""/>

<tags
	ms.service="backup"
	ms.workload="storage-backup-recovery"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/26/2016"
	ms.author="trinadhk;jimpark;"/>


# Azure 虛擬機器備份的疑難排解

> [AZURE.SELECTOR]
- [復原服務保存庫](backup-azure-vms-troubleshoot.md)
- [備份保存庫](backup-azure-vms-troubleshoot-classic.md)

您可以疑難排解將 Azure 備份使用於下表所列資訊時發生的錯誤。

## 探索

| 備份作業 | 錯誤詳細資料 | 因應措施 |
| -------- | -------- | -------|
| 探索 | 無法探索新的項目 - Microsoft Azure 備份發生內部錯誤。等候幾分鐘的時間，然後再次嘗試操作。 | 在 15 分鐘之後重試探索程式。
| 探索 | 無法探索新的項目 – 另一個探索作業正在進行中。請等待目前的探索作業完成。 | None |

## 註冊
| 備份作業 | 錯誤詳細資料 | 因應措施 |
| -------- | -------- | -------|
| 註冊 | 連接至虛擬機器的資料磁碟數目超過支援的限制 - 請卸離此虛擬機器上的某些資料磁碟，然後重試作業。Azure 備份最多支援 16 個資料磁碟連接至 Azure 虛擬機器進行備份。 | None |
| 註冊 | Microsoft Azure 備份遇到內部錯誤 - 等候幾分鐘的時間，然後再次嘗試操作。如果問題持續發生，請連絡 Microsoft 支援服務。 | 出現此錯誤的原因是進階 LRS 上有下列其中一個不支援的 VM 組態。<br> 進階儲存體 VM 可以使用復原服務保存庫來進行備份。[深入了解](backup-introduction-to-azure-backup.md/#back-up-and-restore-premium-storage-vms) |
| 註冊 | 因為安裝代理程式作業逾時而註冊失敗 | 請檢查是否支援虛擬機器的作業系統版本。 |
| 註冊 | 命令執行失敗 - 另一項作業正在此項目上進行。請等到前一項作業完成 | None |
| 註冊 | 不支援備份在進階儲存體中儲存虛擬硬碟的虛擬機器 | None |
| 註冊 | 虛擬機器代理程式不存在於虛擬機器上：請安裝所需的必要元件 VM 代理程式，並重新啟動作業。 | [深入了解](#vm-agent) VM 代理程式安裝，以及如何驗證 VM 代理程式安裝。 |

## 備份

| 備份作業 | 錯誤詳細資料 | 因應措施 |
| -------- | -------- | -------|
| 備份 | 無法與 VM 代理程式通訊來取得快照集狀態。快照集 VM 子工作已逾時。請參閱疑難排解指南中的解決之道。 | 如果 VM 代理程式發生問題，或以某種方式封鎖對 Azure 基礎結構的網路存取，則會擲回這個錯誤。深入了解如何[偵錯 VM 快照集問題](backup-azure-troubleshoot-vm-backup-fails-snapshot-timeout.md)。<br> 如果 VM 代理程式並未造成任何問題，請重新啟動 VM。有時，不正確的 VM 狀態可能會導致問題，重新啟動 VM 可清除此「錯誤狀態」 |
| 備份 | 備份因為內部錯誤而失敗 - 請在幾分鐘內重試此作業。如果問題持續發生，請連絡 Microsoft 支援服務。 | 請檢查存取 VM 儲存體時是否有暫時性的問題。請查看 [Azure Status (Azure 狀態)](https://azure.microsoft.com/status/)，了解該區域目前是否有與計算/儲存體/網路相關的問題。當備份後續問題緩和後，請重試一次。 |
| 備份 | 無法執行作業，因為 VM 不存在。 | 無法執行備份，因為已刪除針對備份設定的 VM。請移至受保護的項目檢視以停止進一步的備份、選取受保護的項目，然後按一下 [停止保護]。您可以選取 [保留備份資料] 選項來保留資料。您可以在稍後從 [已註冊的項目] 檢視按一下 [設定保護]，以繼續保護此虛擬機器|
| 備份 | 無法在選取的項目上安裝 Azure 復原服務延伸模組 - VM 代理程式是 Azure 復原服務延伸模組的必要條件。請安裝 Azure VM 代理程式並重新啟動註冊作業 | <ol> <li>檢查是否已正確安裝 VM 代理程式。<li>確定已正確設定 VM 組態中的旗標。</ol> [深入了解](#validating-vm-agent-installation) VM 代理程式安裝，以及如何驗證 VM 代理程式安裝。 |
| 備份 | 命令執行失敗 - 另一項作業目前正在此項目上進行。請等到前一項作業完成，然後重試 | VM 的現有備份或還原作業正在執行中，而當現有作業正在執行時，無法啟動新的作業。 |
| 備份 | 擴充功能安裝失敗，發生錯誤「COM+ 無法與 Microsoft Distributed Transaction Coordinator 通話」 | 這通常表示 COM+ 服務未執行。請連絡 Microsoft 支援服務來協助解決此問題。 |
| 備份 | 快照集作業失敗，發生 VSS 作業錯誤「這個磁碟機已由 BitLocker 磁碟機加密鎖定」。您必須至 [控制台] 解除鎖定這個磁碟機。 | 對 VM 上的所有磁碟機關閉 BitLocker，並觀察是否已解決 VSS 問題 |
| 備份 | 不支援備份在進階儲存體中儲存虛擬硬碟的虛擬機器 | None |
| 備份 | 找不到 Azure 虛擬機器。 | 當刪除主要 VM 但備份原則繼續尋找 VM 來執行備份時，就會發生這種情況。若要修正此錯誤：<ol><li>使用相同名稱和相同資源群組名稱 [雲端服務名稱] 重新建立虛擬機器，<br>(或) <li> 停用此 VM 的保護，以免觸發後續的備份。</ol> |
| 備份 | 虛擬機器代理程式不存在於虛擬機器上：請安裝所需的必要元件 VM 代理程式，並重新啟動作業。 | [深入了解](#vm-agent) VM 代理程式安裝，以及如何驗證 VM 代理程式安裝。 |

## 作業
| 作業 | 錯誤詳細資料 | 因應措施 |
| -------- | -------- | -------|
| 取消工作 | 不支援取消這種工作類型：請等待工作完成。 | None |
| 取消工作 | 作業不是處於可取消的狀態：請等待作業完成。<br> 或 <br> 所選取的作業不是處於可取消的狀態 - 請等待作業完成。| 工作極有可能幾乎要完成，請等待作業完成 |
| 取消工作 | 無法取消工作，因為它不在進行中：只支援取消進行中的工作。請嘗試取消進行中的工作。 | 這起因於暫時性狀態。請稍候再重試取消作業 |
| 取消工作 | 無法取消作業：請等待作業完成。 | None |


## 還原
| 作業 | 錯誤詳細資料 | 因應措施 |
| -------- | -------- | -------|
| 還原 | 還原失敗並發生雲端內部錯誤 | <ol><li>您嘗試還原的雲端服務已設定為 DNS 設定。您可以檢查 <br>$deployment = Get-AzureDeployment -ServiceName "ServiceName" -Slot "Production" Get-AzureDns -DnsSettings $deployment.DnsSettings<br>如果有設定位址，這表示已設定為 DNS 設定。<br> <li>您嘗試還原的雲端服務利用 ReservedIP 設定，且雲端服務中現有的 VM 皆處於停止狀態。<br>您可以使用下列 PowerShell Cmdlet 來檢查雲端服務是否有保留的 IP：<br>$deployment = Get-AzureDeployment -ServiceName "servicename" -Slot "Production" $dep.ReservedIPName <br><li>您正在嘗試還原相同雲端服務中具有以下特殊網路設定的虛擬機器。<br>- 負載平衡器設定下的虛擬機器 (內部與外部)<br>- 具有多個保留 IP 的虛擬機器<br>- 具有多個 NIC 的虛擬機器<br>請在 UI 中選取新的雲端服務，或參閱[還原考量](./backup-azure-restore-vms.md/#restoring-vms-with-special-network-configurations)以取得具有特殊網路設定之 VM 的相關資訊</ol> |
| 還原 | 所選取的 DNS 名稱已被使用：請指定不同的 DNS 名稱，然後再試一次。 | 此處的 DNS 名稱是指雲端服務名稱 (結尾通常是 .cloudapp.net)。這必須是唯一的。如果您遇到這個錯誤，您需要在還原期間選擇不同的 VM 名稱。<br><br> 只有 Azure 入口網站的使用者才會看到這個錯誤。透過 PowerShell 執行還原作業將會成功，因為它只會還原磁碟，並不會建立 VM。當您在磁碟還原作業之後明確建立 VM 時，將會遇到此錯誤。 |
| 還原 | 指定的虛擬網路組態不正確：請指定不同的虛擬網路組態，然後再試一次。 | None |
| 還原 | 指定的雲端服務使用保留的 IP，但不符合所要還原的虛擬機器組態：請指定另一個不使用保留 IP 的雲端服務，或選擇從另一個復原點還原。 | None |
| 還原 | 雲端服務已達到輸入端點數目限制：請指定不同的雲端服務或使用現有的端點來重試作業。 | None |
| 還原 | 備份保存庫和目標儲存體帳戶位於兩個不同區域：請確定還原作業中指定的儲存體帳戶，位於與備份保存庫相同的 Azure 區域。 | None |
| 還原 | 不支援還原作業所指定的儲存體帳戶：只支援具有本地備援或異地備援複寫設定的基本/標準儲存體帳戶。請選取支援的儲存體帳戶 | None |
| 還原 | 還原作業所指定的儲存體帳戶類型不在線上：請確定還原作業中指定的儲存體帳戶在線上 | 這可能是因為 Azure 儲存體的暫時性錯誤或運作中斷。請選擇另一個儲存體帳戶。 |
| 還原 | 已達到資源群組配額：請從 Azure 入口網站刪除一些資源群組，或連絡 Azure 支援以提高限制。 | None |
| 還原 | 選取的子網路不存在：請選取已存在的子網路 | None |


## 原則
| 作業 | 錯誤詳細資料 | 因應措施 |
| -------- | -------- | -------|
| 建立原則 | 無法建立原則：請減少保留選項以繼續設定原則。 | None |


## VM 代理程式

### 設定 VM 代理程式
一般而言，VM 代理程式已存在於從 Azure 資源庫建立的 VM 中。不過，從內部部署資料中心移轉的虛擬機器不會安裝 VM 代理程式。對於這類 VM，必須明確安裝 VM 代理程式。深入了解[在現有 VM 上安裝 VM 代理程式](http://blogs.msdn.com/b/mast/archive/2014/04/08/install-the-vm-agent-on-an-existing-azure-vm.aspx)。

若為 Windows VM：

- 下載並安裝[代理程式 MSI](http://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409)。您需要有系統管理員權限，才能完成安裝。
- [更新 VM 屬性](http://blogs.msdn.com/b/mast/archive/2014/04/08/install-the-vm-agent-on-an-existing-azure-vm.aspx)，表示已安裝代理程式。

如為 Linux VM：

- 從 GitHub 安裝最新版 [Linux 代理程式](https://github.com/Azure/WALinuxAgent)。
- [更新 VM 屬性](http://blogs.msdn.com/b/mast/archive/2014/04/08/install-the-vm-agent-on-an-existing-azure-vm.aspx)，表示已安裝代理程式。


### 更新 VM 代理程式
若為 Windows VM：

- 更新 VM 代理程式與重新安裝 [VM 代理程式二進位檔](http://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409)一樣簡單。不過，您需要確定在更新 VM 代理程式時，沒有任何執行中的備份作業。

如為 Linux VM：

- 請遵循[更新 Linux VM 代理程式](../virtual-machines/virtual-machines-linux-update-agent.md)上的指示進行。


### 驗證 VM 代理程式安裝
如何檢查 Windows VM 上的 VM 代理程式版本：

1. 登入 Azure 虛擬機器，然後瀏覽至 *C:\\WindowsAzure\\Packages* 資料夾。您應該會發現 WaAppAgent.exe 檔案已存在。
2. 在該檔案上按一下滑鼠右鍵，前往 [屬性]，然後選取 [詳細資料] 索引標籤。[產品版本] 欄位應為 2.6.1198.718 或更高版本

<!---HONumber=AcomDC_0921_2016-->