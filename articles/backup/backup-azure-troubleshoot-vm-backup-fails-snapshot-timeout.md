<properties
   pageTitle="Azure VM 備份失敗︰無法與 VM 代理程式通訊來取得快照集狀態 - 快照 VM 子工作已逾時 | Microsoft Azure"
   description="與無法與 VM 代理程式通訊來取得快照集狀態相關的 Azure VM 備份失敗的徵狀原因和解決方案。快照集 VM 子工作已逾時錯誤"
   services="backup"
   documentationCenter=""
   authors="genlin"
   manager="jwhit"
   editor=""/>

<tags
    ms.service="backup"
    ms.workload="storage-backup-recovery"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="07/14/2016"
    ms.author="jimpark; markgal;genli"/>

# Azure VM 備份失敗︰無法與 VM 代理程式通訊來取得快照集狀態 - 快照集 VM 子工作已逾時

## 摘要

註冊及排程備份 Azure 備份的 Azure 虛擬機器 (VM) 之後，Azure 服務會藉由與 VM 中的備份擴充功能通訊來取得時間點快照集，在排定的時間起始備份作業。有些情況可能會造成無法觸發快照集，因而導致備份失敗。這篇文章提供關於 Azure VM 備份失敗中，與快照集逾時錯誤相關之問題的疑難排解步驟。

[AZURE.INCLUDE [支援免責聲明](../../includes/support-disclaimer.md)]

## 徵狀

基礎結構即服務 (IaaS) 適用的 Microsoft Azure 備份 VM 失敗，並在 Azure 入口網站中的作業錯誤詳細資料中傳回下列錯誤訊息。

**無法與 VM 代理程式通訊來取得快照集狀態 - 快照集 VM 子工作已逾時。**

## 原因
此錯誤有四個常見的原因︰

- VM 沒有網際網路存取。
- VM 中安裝的 Microsoft Azure VM 代理程式已過時 (針對 Linux VM)。
- 備份擴充功能無法更新或載入。
- 無法擷取快照集狀態或無法取得快照集。

## 原因 1：VM 沒有網際網路存取
根據部署需求，VM 沒有網際網路存取，或是有既有限制，防止存取 Azure 基礎結構。

備份擴充功能需要連線到 Azure 公用 IP 位址，才能正確運作。這是因為它會將命令傳送至 Azure 儲存體端點 (HTTP URL) 來管理 VM 的快照集。如果擴充功能無法存取公用網際網路，則備份最終會失敗。

### 方案
在此案例中，使用下列方法之一來解決問題︰

- 將 Azure 資料中心的 IP 範圍列入允許清單
- 建立 HTTP 流量的行經路徑

### 將 Azure 資料中心的 IP 範圍列入允許清單

1. 取得要列入允許清單的 [Azure 資料中心 IP 清單](https://www.microsoft.com/download/details.aspx?id=41653)。
2. 使用 New-NetRoute Cmdlet 解除封鎖 IP。在提升權限的 PowerShell 視窗中 (以系統管理員身分執行)，於 Azure VM 執行這個 Cmdlet。
3. 若您有允許存取 IP 的規則，將規則新增網路安全性群組 (NSG)。

### 建立 HTTP 流量的行經路徑

1. 如果已有既定的網路限制 (例如，NSG)，請部署 HTTP Proxy 伺服器來路由傳送流量。
2. 如果您有網路安全性群組 (NSG)，請新增規則來允許從 HTTP Proxy 存取網際網路。

了解如何[設定 VM 備份的 HTTP Proxy](backup-azure-vms-prepare.md#using-an-http-proxy-for-vm-backups)。

## 原因 2︰VM 中安裝的 Microsoft Azure VM 代理程式已過時 (針對 Linux VM)

### 方案
針對 Linux VM，與代理程式或擴充功能相關的多數失敗是由於會影響舊 VM 代理程式的問題所造成。一般來說，針對此問題進行疑難排解的首要步驟如下所示︰

1. [安裝最新版的 Azure VM 代理程式](https://github.com/Azure/WALinuxAgent)。
2. 確定 Azure 代理程式在 VM 上執行。若要這樣做，請執行下列命令：```ps -e```

    如果此程序未執行，請使用下列命令來重新啟動它。

    針對 Ubuntu：```service walinuxagent start```

    針對其他散發套件︰```service waagent start
```

3. [設定自動重新啟動代理程式](https://github.com/Azure/WALinuxAgent/wiki/Known-Issues#mitigate_agent_crash)。

4. 執行新的測試備份。如果失敗持續發生，請從下列資料夾收集記錄，以進一步偵錯。

    我們需要來自客戶的 VM 中的下列記錄檔︰

    - /var/lib/waagent/*.xml
    - /var/log/waagent.log
    - /var/log/azure/*

如果我們需要 waagent 的詳細資訊記錄，請遵循下列步驟來啟用此功能︰

1. 在 /etc/waagent.conf 檔案中，找出下一行︰

    啟用詳細資訊記錄 (y | n)

2. 將 **Logs.Verbose** 值從 n 變更為 y。
3. 儲存變更，然後遵循本節前面的步驟，重新啟動 waagent。

## 原因 3︰備份擴充功能無法更新或載入
如果無法載入擴充功能，那麼，因為無法取得快照集，備份會失敗。

### 方案
針對 Windows 客體作業系統︰

1. 確認 iaasvmprovider 服務已啟用，而且啟動類型為自動。
2. 如果組態並非如此，請啟用服務以判斷下一次備份是否成功。

針對 Linux 客體︰

VMSnapshot Linux (備份所使用的擴充功能) 的最新版本是 1.0.91.0

如果還是無法更新或載入備份擴充功能，您可以透過解除安裝擴充功能來強制重新載入 VMSnapshot 擴充功能。下一次的備份嘗試將會重新載入擴充功能。

### 解除安裝擴充功能

1. 移至 [Azure 入口網站](https://portal.azure.com/)。
2. 找出具有備份問題的特定 VM。
3. 按一下 [設定]。
4. 按一下 [擴充功能]。
5. 按一下 [Vmsnapshot 擴充功能]。
6. 按一下 [解除安裝]。

這會導致在下一次備份期間重新安裝擴充功能。

## 原因 4︰無法擷取快照集狀態或無法取得快照集
VM 備份仰賴發給底層儲存體的快照命令。備份可能會失敗，因為它無法存取儲存體，或因為快照集工作執行中發生延遲。

### 方案
下列狀況可能導致快照集工作失敗：

| 原因 | 方案 |
| ----- | ----- |
| 已設定 Microsoft SQL Server 備份的 VM。根據預設，VM 備份會在 Windows VM 上執行 VSS 完整備份。在執行以 SQL Server 為基礎的伺服器並設定了 SQL Server 備份的 VM 上，快照集執行可能會發生延遲。 | 如果因為快照集問題發生備份失敗，請設定下列登錄機碼。<br><br>[HKEY\_LOCAL\_MACHINE\\SOFTWARE\\MICROSOFT\\BCDRAGENT] "USEVSSCOPYBACKUP"="TRUE" |
| 因為 RDP 中的 VM 關機，報告的 VM 狀態不正確。如果您關閉 RDP 中的虛擬機器，請檢查入口網站，以判斷是否正確反映該 VM 的狀態。 | 如果不存在，請在入口網站中使用 VM 儀表板中的「關閉」選項來關閉 VM。 |
| 相同的雲端服務中的許多 VM 都設定為同時備份。 | 最佳做法是向外分配相同雲端服務中的 VM，使其有不同的備份排程。 |
| VM 正在以高 CPU 或記憶體使用量執行。 | 如果 VM 正在以高 CPU 使用量 (超過 90%) 或高記憶體使用量執行，快照集工作會排入佇列並延遲，而最終會逾時。在此情況下，請嘗試隨選備份。 |
|VM 無法從 DHCP 取得主機/網狀架構位址。|必須在來賓內啟用 DHCP，IaaS VM 備份才能運作。如果 VM 無法從 DHCP 回應 245 取得主機/網狀架構位址，則無法下載或執行任何擴充功能。如果您需要靜態私人 IP 位址，您應該透過平台來進行設定。VM 內的 DHCP 選項應保持啟用。請參閱有關[設定靜態內部私人 IP 位址](../virtual-network/virtual-networks-reserved-private-ip.md)的詳細資訊。|

<!---HONumber=AcomDC_0921_2016-->