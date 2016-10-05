<properties
	pageTitle="進階儲存體：Azure 虛擬機器工作負載適用的高效能儲存體 | Microsoft Azure"
	description="「進階儲存體」可針對在「Azure 虛擬機器」上執行且需要大量 I/O 的工作負載，提供高效能、低延遲的磁碟支援。Azure DS 系列、DSv2 系列和 GS 系列 VM 支援進階儲存體。"
	services="storage"
	documentationCenter=""
	authors="aungoo-msft"
	manager="tadb"
	editor="tysonn"/>

<tags
	ms.service="storage"
	ms.workload="storage"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/19/2016"
	ms.author="aungoo;robinsh"/>


# 進階儲存體： Azure 虛擬機器工作負載適用的高效能儲存體

## Overview

針對執行時需要大量 I/O 之工作負載的虛擬機器，「Azure 進階儲存體」可提供高效能、低延遲的磁碟支援。使用「進階儲存體」的虛擬機器 (VM) 會將資料儲存在固態硬碟 (SSD) 上。您可以將應用程式的 VM 磁碟移轉到「Azure 進階儲存體」，以利用這些磁碟的速度和效能。

Azure VM 支援連接數個「進階儲存體」磁碟，讓您應用程式的每一 VM 最多可擁有 64 TB 的儲存體。使用「進階儲存體」時，您應用程式的每一 VM 可達到 80,000 IOPS (每秒輸入/輸出作業)，而每一 VM 的每秒磁碟輸送量為 2000 MB，且讀取作業的延遲極低。

Azure 使用進階儲存體，提供您將要求較高的企業應用程式 (例如 Dynamics AX、Dynamics CRM、Exchange Server、SharePoint Farms 與 SAP Business Suite) 真正提升並轉移至雲端的能力。您可以在進階儲存體上執行強調效能、需要持續高效能和低延遲的資料庫工作負載，例如 SQL Server、Oracle、MongoDB、MySQL、Redis。

>[AZURE.NOTE] 建議您將任何需要高 IOPS 的虛擬機器磁碟移轉到「Azure 進階儲存體」，以發揮應用程式最佳效能。如果您的磁碟不需要高 IOPS，您可以在「標準儲存體」中維護它來限制成本，這會將虛擬機器磁碟資料儲存在「硬碟機 (HDD)」上而非 SSD 上。

若要開始使用 Azure Premium 儲存體，請造訪[免費開始](https://azure.microsoft.com/pricing/free-trial/)頁面。如需有關將現有虛擬機器移轉到「進階儲存體」的資訊，請參閱＜移轉到 Azure 進階儲存體＞[](storage-migration-to-premium-storage.md)。

>[AZURE.NOTE] 目前有些區域不支援進階儲存體。您可以在[依區域的 Azure 服務](https://azure.microsoft.com/regions/#services)中找到可用區域清單。

## 進階儲存體功能

**進階儲存體磁碟**：「Azure 進階儲存體」支援可連接到「進階儲存體」支援之 Azure VM (DS、DSv2、GS 或 Fs 系列) 的 VM 磁碟。使用進階儲存體時，您可以選擇三種磁碟大小 (也就是 P10 (128GiB)、P20 (512GiB) 及 P30 (1024GiB))，每種都有自己的效能規格。視您的應用程式需求而定，您可以將一或多個這類磁碟連接到「進階儲存體」支援的 VM。在下一節的[進階儲存體延展性和效能目標](#premium-storage-scalability-and-performance-targets)中，我們會更詳細地說明規格。

進階儲存體 Blob：進階儲存體可支援 Azure 分頁 Blob，其用於保存適用於 Azure 虛擬機器 (VM) 的永續性磁碟。進階儲存體目前不支援 Azure 區塊 Blob、Azure 附加 Blob、Azure 檔案、Azure 資料表或 Azure 佇列。放在進階儲存體帳戶中的任何其他物件都會是分頁 Blob，並且貼齊其中一個支援的佈建大小。Heance 進階儲存體帳戶不是用於儲存小型 Blob。

**進階儲存體帳戶**︰若要開始使用進階儲存體，您必須建立進階儲存體帳戶。如果您想要使用 [Azure 入口網站](https://portal.azure.com)，則可以指定「進階」效能層和「本地備援儲存體 (LRS)」作為複寫選項來建立進階儲存體帳戶。使用[儲存體 REST API](http://msdn.microsoft.com//library/azure/dd179355.aspx) 2014-02-14 版或更新版本將類型指定為 “Premium\_LRS”、[服務管理 REST API](http://msdn.microsoft.com/library/azure/ee460799.aspx) 2014-10-01 版或更新版本 (傳統部署)、[Azure 儲存體資源提供者 REST API 參考](http://msdn.microsoft.com/library/azure/mt163683.aspx) (Resource Manager 部署) 以及 [Azure PowerShell](../powershell-install-configure.md) 0.8.10 版或更新版本，也可以建立進階儲存體帳戶。在下一節的[進階儲存體延展性和效能目標](#premium-storage-scalability-and-performance-targets)中，深入了解進階儲存體帳戶限制。

**進階本地備援儲存體**：進階儲存體帳戶僅支援本地備援儲存體 (LRS) 作為複寫選項，並在單一區域內會保留三份資料。如需使用進階儲存體時關於異地複寫的考量，請參閱本文的[快照與複製 Blob](#snapshots-and-copy-blob) 一節。

Azure 使用儲存體帳戶做為作業系統 (OS) 和資料磁碟的容器。當您建立 Azure DS、DSv2、GS 或 Fs VM 並選取「Azure 進階儲存體」帳戶時，您的作業系統和資料磁碟會儲存在該儲存體帳戶中。

有兩種方式可使用 Premium 磁碟儲存體：
- 首先，建立新的進階儲存體帳戶。接著，在建立新的 DS、DSv2、GS 或 Fs VM 時，在儲存體組態設定中選取進階儲存體帳戶。或者，
- 在建立新的 DS、DSv2、GS 或 Fs VM 時，在儲存體組態設定中建立新的進階儲存體帳戶，或讓 Azure 入口網站建立預設的進階儲存體帳戶。

如需逐步指示，請參閱本文稍後的[快速啟動](#quick-start)一節。

>[AZURE.NOTE] Premium 儲存體帳戶無法對應到自訂網域名稱。

## 進階儲存體支援的 VM

「進階儲存體」支援 DS 系列、DSv2 系列、GS 系列及 Fs 系列的 Azure 虛擬機器 (VM)。您可以將「標準」和「進階」儲存體磁碟與「進階儲存體」支援的 VM 搭配使用。但是不能將「進階儲存體」磁碟與非「進階儲存體」相容的 VM 系列搭配使用。

如需可用 Azure VM 類型和 Windows VM 大小的資訊，請參閱 [Windows VM 大小](../virtual-machines/virtual-machines-windows-sizes.md)。如需可用 VM 類型和 Linux VM 大小的資訊，請參閱 [Linux VM 大小](../virtual-machines/virtual-machines-linux-sizes.md)。

以下是 DS、DSv2、GS 及 Fs 系列 VM 的一些功能：

**雲端服務**：可以將 DS 系列 VM 加入至僅包含 DS 系列 VM 的雲端服務。請勿將 DS 系列 VM 加入至包含非 DS 系列 VM 的現有雲端服務。您可以將現有的 VHD 移轉到僅執行 DS 系列 VM 的新雲端服務。如果您想要為裝載 DS 系列 VM 的新雲端服務保留相同的虛擬 IP 位址 (VIP)，請使用[保留的 IP 位址](../virtual-network/virtual-networks-instance-level-public-ip.md)。可將 GS 系列 VM 加入至僅執行 G 系列 VM 的現有雲端服務。

**作業系統磁碟**：您可以將「進階儲存體」支援的 Azure 虛擬機器設定成使用裝載在「標準儲存體」帳戶或「進階儲存體」帳戶上的作業系統 (OS) 磁碟。我們建議使用進階儲存體型的 OS 磁碟以獲得最佳體驗。

**資料磁碟**：您可以在相同的「進階儲存體」支援 VM 中同時使用「進階」和「標準」儲存體磁碟。使用「進階儲存體」時，您可以佈建「進階儲存體」支援的 VM，然後將數個持續性資料磁碟連接到此 VM。如有需要，您可以跨磁碟等量磁碟區以增加磁碟區的容量和效能。

> [AZURE.NOTE] 如果您使用[儲存空間](http://technet.microsoft.com/library/hh831739.aspx)等量 Premium 儲存體資料磁碟，應該為所使用的每個磁碟，以一個資料行進行設定。否則，等量磁碟區的整體效能可能會因為磁碟流量分配不平均而比預期的效能還低。根據預設，伺服器管理員使用者介面 (UI) 可讓您設定最多 8 個磁碟的資料行。但是，如果您有 8 個以上的磁碟，您就必須使用 PowerShell 來建立磁碟區，並且手動指定資料行數目。否則，即使您擁有更多磁碟，伺服器管理員 UI 還是會繼續使用 8 個資料行。例如，如果您在單一等量磁碟區組有 32 個磁碟，您應該指定 32 個資料行。您可以使用 [New-VirtualDisk](http://technet.microsoft.com/library/hh848643.aspx) PowerShell Cmdlet 的 *NumberOfColumns* 參數，指定虛擬磁碟所使用的資料行數目。如需詳細資訊，請參閱[儲存體空間概觀](http://technet.microsoft.com/library/hh831739.aspx)和[儲存體空間常見問題集](http://social.technet.microsoft.com/wiki/contents/articles/11382.storage-spaces-frequently-asked-questions-faq.aspx)。

**快取**：「進階儲存體」支援的 VM 具有獨特的快取功能，可讓您享有超越基礎「進階儲存體」磁碟效能的高層級輸送量和延遲表現。您可以在進階儲存體磁碟上將磁碟快取原則設定為 [唯讀]、[讀寫] 或 [無]。所有進階資料磁碟的預設磁碟快取原則都是 [唯讀]，而作業系統磁碟的磁碟快取原則則是 [讀寫]。使用正確的組態設定，以達到應用程式的最佳效能。例如，對於讀取繁重或唯讀資料磁碟 (如 SQL Server 資料檔)，將磁碟快取原則設定為 [唯讀]。例如，對於寫入繁重或唯寫資料磁碟 (如 SQL Server 記錄檔)，將磁碟快取原則設定為 [無]。在[使用進階儲存體設計高效能](storage-premium-storage-performance.md)中，深入了解如何最佳化進階儲存體。

**分析**︰若要分析使用進階儲存體帳戶磁碟的 VM 效能，您可以在 Azure 入口網站中啟用 Azure VM 診斷。如需詳細資料，請參閱[使用 Azure 診斷擴充功能監視 Microsoft Azure 虛擬機器](https://azure.microsoft.com/blog/2014/09/02/windows-azure-virtual-machine-monitoring-with-wad-extension/)。若要查看磁碟效能，請使用作業系統工具，例如適用於 Windows VM 的 [Windows 效能監視器](https://technet.microsoft.com/library/cc749249.aspx)和適用於 Linux VM 的 [IOSTAT](http://linux.die.net/man/1/iostat)。

**VM 規模限制和效能**︰每個「進階儲存體」支援的 VM 大小在 IOPS、頻寬及每個 VM 可連接的磁碟數目上都有規模限制和效能規格。使用進階儲存體磁碟搭配「進階儲存體」支援的 VM 時，請確定 VM 上有足夠的 IOPS 和頻寬可用來運送磁碟流量。例如，STANDARD\_DS1 VM 每秒專用頻寬有 32 MB 的 Premium 儲存體磁碟流量。P10 進階儲存體磁碟可以提供每秒 100 MB 的頻寬。如果 P10 進階儲存體磁碟連接到此 VM，則最高只能達到每秒 32 MB，而無法達到 P10 磁碟可提供的每秒 100 MB。

目前在 DS 系列上最大的 VM 是 STANDARD\_DS14，它在所有磁碟最多可以提供每秒 512 MB。GS 系列上最大的 VM 是 STANDARD\_GS5，它在所有磁碟最多可以提供每秒 2000 MB。請注意，這些限制僅適用於磁碟流量，不包含快取命中數和網路流量。VM 網路流量有不同的頻寬，與 Premium 儲存體磁碟專用的頻寬不同。

如需有關「進階儲存體」支援之 VM 的最新 IOPS 和輸送量 (頻寬) 上限資訊，請參閱 [Windows VM 大小](../virtual-machines/virtual-machines-windows-sizes.md)或 [Linux VM 大小](../virtual-machines/virtual-machines-linux-sizes.md)。

若要了解進階儲存體磁碟及其 IOPS 與輸送量限制，請參閱本文的[進階儲存體延展性和效能目標](#premium-storage-scalability-and-performance-targets)一節中的表格。

## 進階儲存體延展性和效能目標

在這一節中，我們將說明在使用進階儲存體時必須考慮的延展性和效能目標。

### 進階儲存體帳戶限制

進階儲存體帳戶有下列延展性目標：

<table border="1" cellspacing="0" cellpadding="5" style="border: 1px solid #000000;">
<tbody>
<tr>
	<td><strong>總帳戶容量</strong></td>
	<td><strong>本地備援儲存體帳戶總頻寬</strong></td>
</tr>
<tr>
	<td>
	<ul>
       <li type=round>磁碟容量：35 TB</li>
       <li type=round>快照容量：10 TB</li>
    </ul>
	</td>
	<td>每秒最多 50 Gbps (輸入 + 輸出)</td>
</tr>
</tbody>
</table>

- 輸入是指傳送至某個儲存體帳戶的所有資料 (要求)。
- 輸出是指從某個儲存體帳戶接收的所有資料 (回應)。

如需詳細資訊，請參閱 [Azure 儲存體延展性和效能目標](storage-scalability-targets.md)。

如果您的應用程式需求超出單一儲存體帳戶的延展性目標，請建置使用多個儲存體帳戶的應用程式，並將資料分散到那些儲存體帳戶中。例如，如果要將 51 TB 的磁碟連結到多個 VM，請將它們分散到兩個儲存體帳戶，因為單一進階儲存體帳戶的限制是 35 TB。請務必確認單一 Premium 儲存體帳戶的佈建磁碟不要超過 35 TB。

### 進階儲存體磁碟限制

當您為某個 Premium 儲存體帳戶佈建磁碟時，其每秒的輸入/輸出作業 (IOPS) 和輸送量 (頻寬) 取決於磁碟大小。目前有三種類型的 Premium 儲存體磁碟：P10、P20 及 P30。每種各有特定的 IOPS 和輸送量限制，如下表所示：

<table border="1" cellspacing="0" cellpadding="5" style="border: 1px solid #000000;">
<tbody>
<tr>
	<td><strong>Premium 儲存體磁碟類型</strong></td>
	<td><strong>P10</strong></td>
	<td><strong>P20</strong></td>
	<td><strong>P30</strong></td>
</tr>
<tr>
	<td><strong>磁碟大小</strong></td>
	<td>128 GB</td>
	<td>512 GB</td>
	<td>1024 GB (1 TB)</td>
</tr>
<tr>
	<td><strong>每一磁碟的 IOPS</strong></td>
	<td>500</td>
	<td>2300</td>
	<td>5000</td>
</tr>
<tr>
	<td><strong>每一磁碟的輸送量</strong></td>
	<td>每秒 100 MB </td>
	<td>每秒 150 MB </td>
	<td>每秒 200 MB </td>
</tr>
</tbody>
</table>

> [AZURE.NOTE] 請確定您的 VM 上有足夠的頻寬可用來運送磁碟流量，如本文稍早[進階儲存體支援的 VM](#ds-dsv2-and-gs-series-vms) 一節所述。否則，您的磁碟輸送量和 IOPS 將會根據 VM 限制而不是上表所述的磁碟限制而受限於較低的值。

以下是有關進階儲存體延展性和效能目標，您必須知道的一些重要事項︰

- **佈建的容量和效能**︰當您佈建進階儲存體磁碟時，不同於標準儲存體的是，您可獲得該磁碟的容量、IOPS 和輸送量保證。例如，如果您建立 P30 磁碟，Azure 會為該磁碟佈建 1024 GB 儲存體容量、5000 IOPS 和每秒 200 MB 的輸送量。您的應用程式可以使用全部或部分的容量和效能。

- **磁碟大小**：Azure 會將磁碟大小對應 (無條件進位) 至表格中指定之最接近的 [進階儲存體磁碟] 選項。例如，大小為 100 GB 的磁碟會分類為 P10 選項，每秒最多可執行 500 個 IO 單位，每秒輸送量可達 100 MB。同樣地，大小為 400 GB 的磁碟會分類為 P20 選項，每秒最多可執行 2300 個 IO 單位，每秒輸送量可達 150 MB。

	> [AZURE.NOTE] 您可以輕易增加現有磁碟的大小。例如，如果您想要將 30 GB 大小的磁碟增加到 128 GB 或 1 TB。或者，如果您想要將 P20 磁碟轉換為 P30 磁碟，因為您需要更多容量或更多的 IOPS 和輸送量。您可以使用 "Update-AzureDisk" PowerShell 命令搭配 "-ResizedSizeInGB" 屬性來擴充磁碟。若要執行這個動作，必須先從 VM 卸離磁碟或停止 VM。

- **IO 大小**：輸入/輸出 (I/O) 單位大小為 256 KB。如果要傳送的資料少於 256 KB，會視為單一 I/O 單位。較大的 I/O 大小則會視為大小是 256 KB 的多個 I/O。例如，1100 KB 的 I/O 會視為五個 I/O 單位。

- **輸送量**：輸送量限制包含寫入至磁碟，以及從磁碟而不是從快取的讀取。例如，P10 磁碟有每秒 100 MB 的每一磁碟輸送量。P10 磁碟的有效輸送量範例如下：
<table border="1" cellspacing="0" cellpadding="5" style="border: 1px solid #000000;">
<tbody>
<tr>
<td><strong>每個 P10 磁碟的輸送量上限</strong></td>
<td><strong>從磁碟的非快取讀取</strong></td>
<td><strong>對磁碟的非快取寫入</strong></td>
</tr>
<tr>
<td>每秒 100 MB</td>
<td>每秒 100 MB</td>
<td>0</td>
</tr>
<tr>
<td>每秒 100 MB</td>
<td>0</td>
<td>每秒 100 MB</td>
</tr>
<tr>
<td>每秒 100 MB </td>
<td>每秒 60 MB </td>
<td>每秒 40 MB </td>
</tr>
</tbody>
</table>

- **快取命中數**：快取命中數不會受到磁碟配置 IOPS/輸送量的限制。例如，當您在「進階儲存體」支援的 VM 上使用具有 ReadOnly 快取設定的資料磁碟時，從快取提供的「讀取數」並不受「進階儲存體」磁碟限制約束。因此，如果工作負載以讀取為主，可以從磁碟獲得極高的輸送量。請注意，快取會根據 VM 大小，受到 VM 層級個別 IOPS / 輸送量的限制。DS 系列 VM 大約有 4000 IOPS，快取與本機 SSD IO 是每個核心 33 MB/秒。GS 系列 VM 的限制為 5000 IOPS，而快取與本機 SSD IO 是每個核心 50 MB/秒。

## 節流
如果您的應用程式 IOPS 或輸送量超過進階儲存體磁碟的配置限制，或如果您在 VM 上所有磁碟的磁碟總流量超過 VM 可用的磁碟頻寬限制，您可能會看到節流。若要避免節流，建議您根據佈建之磁碟的延展性和效能目標，以及根據 VM 可用的磁碟頻寬，來限制磁碟的擱置 I/O 要求數。

當您的應用程式設計為避免節流情況時，可以達到最低延遲。但是，如果磁碟的擱置 I/O 要求數過小，您的應用程式便無法達到磁碟最大的 IOPS 和輸送量層級。

下列範例示範如何計算節流層級。所有計算都是以 256 KB 的 I/O 單位大小為基礎：

### 範例 1：
在 P10 磁碟上，您的應用程式一秒內有 495 個 16 KB 大小的 I/O 單位。這些會被視為每秒 495 個 I/O 單位 (IOPS)。如果您在該秒內嘗試 2 MB 的 I/O，I/O 單位的總數會等於 495 + 8。這是因為當 I/O 單位大小是 256 KB 時，2 MB 的 I/O 會產生 2048 KB / 256 KB = 8 個 I/O 單位。因為 495 + 8 的總和超出磁碟 500 個 IOPS 的限制，因此會發生節流情況。

### 範例 2：
在 P10 磁碟上，您的應用程式有 400 個 256 KB 大小的 I/O 單位。耗用的總頻寬是 (400 * 256) / 1024 = 100 MB/秒。P10 磁碟的輸送量限制為每秒 100 MB。如果您的應用程式嘗試在該秒內執行更多 I/O 便會發生節流情況，因為會超出配置的限制。

### 範例 3：
DS4 VM 連接了兩個 P30 磁碟。每個 P30 磁碟有每秒 200 MB 的輸送量。不過，DS4 VM 有每秒 256 MB 的磁碟總頻寬容量。因此，此 DS4 VM 上連接的磁碟無法同時達到最大的輸送量。若要解決這個問題，您可以在一個磁碟上維持每秒 200 MB 的流量，在另一個磁碟上維持每秒 56 MB 的流量。如果您的磁碟流量總和超過每秒 256 MB，磁碟流量就會發生節流。

>[AZURE.NOTE] 如果磁碟流量大多包含小型 I/O，您的應用程式在達到輸送量限制前很可能會先達到 IOPS 限制。但是，如果磁碟流量大多包含大型 I/O，您的應用程式可能會達到輸送量限制，而非 IOPS 限制。您可以使用最佳的 I/O 大小和限制磁碟的擱置 I/O 要求數，以最大化應用程式 IOPS 和輸送量容量。

若要了解如何使用進階儲存體設計高效能，請參閱[使用進階儲存體設計高效能](storage-premium-storage-performance.md)一文。

## 快照與複製 Blob
您可以使用 Standard 儲存體時建立快照的方式來為 Premium 儲存體建立快照。因為進階儲存體僅支援本地備援儲存體 (LRS) 作為複寫選項，所以建議您建立快照集，然後將那些快照集複製到異地備援標準儲存體帳戶。如需詳細資訊，請參閱 [Azure 儲存體備援選項](storage-redundancy.md)。

如果磁碟已連結至 VM，在備份磁碟的分頁 Blob 上不允許某些 API 作業。例如，只要磁碟連接至 VM，您就無法在該 Blob 上執行[複製 Blob](http://msdn.microsoft.com/library/azure/dd894037.aspx) 作業。您必須先使用[快照 Blob](http://msdn.microsoft.com/library/azure/ee691971.aspx) REST API 方法建立該 Blob 的快照，然後對該快照執行[複製 Blob](http://msdn.microsoft.com/library/azure/dd894037.aspx) 以複製連接的磁碟。或者，您可以中斷連結磁碟，然後在基礎 Blob 上執行任何必要的作業。

下列限制適用於進階儲存體 Blob 快照︰
<table border="1" cellspacing="0" cellpadding="5" style="border: 1px solid #000000;">
<tbody>
<tr>
	<td><strong>進階儲存體限制</strong></td>
	<td><strong>值</strong></td>
</tr>
<tr>
	<td>每個 Blob 的快照數目上限</td>
	<td>100</td>
</tr>
<tr>
	<td>快照的儲存體帳戶容量 (只包含快照中的資料，不包含基底 Blob 中的資料)。</td>
	<td>10 TB</td>
</tr>
<tr>
	<td>連續快照之間的時間下限</td>
	<td>10 分鐘</td>
</tr>
</tbody>
</table>

若要維護快照集的異地備援副本，您可以使用 AzCopy 或「複製 Blob」，將進階儲存體帳戶中的快照複製到異地備援的標準儲存體帳戶。如需詳細資訊，請參閱[使用 AzCopy 命令列公用程式傳輸資料](storage-use-azcopy.md)和[複製 Blob](http://msdn.microsoft.com/library/azure/dd894037.aspx)。

如需對 Premium 儲存體帳戶中的分頁 Blob 執行 REST 作業的詳細資訊，請參閱 MSDN Library 中的[在 Azure 儲存體帳戶使用 Blob 服務作業](http://go.microsoft.com/fwlink/?LinkId=521969)。

## 在 Premium 儲存體使用 Linux VM
請參閱下方重要的指示了解如何在 Premium 儲存體上設定 Linux VM：

- 對於快取設定為 "ReadOnly" 或 "None" 的所有 Premium 儲存體磁碟，您在掛接檔案系統時必須停用 "barrier" (阻礙)，才能達到 Premium 儲存體的延展性目標。此案例中您不需要阻礙，因為這些快取設定的 Premium 儲存體磁碟寫入都是持久的。寫入要求成功完成時，資料就已寫入永久性的存放區。請根據您的檔案系統，使用下列方法停用 "barrier" (阻礙)：
	- 如果您使用 **reiserFS**，請使用掛接選項 "barrier=none" 停用阻礙 (若要啟用阻礙，請使用 "barrier=flush")
	- 如果您使用 **ext3/ext4**，請使用掛接選項 "barrier=0" 停用阻礙 (若要啟用阻礙，請使用 "barrier=1")
	- 如果您使用 **XFS**，請使用掛接選項 "nobarrier" 停用阻礙 (若要啟用阻礙，請使用 "barrier" 選項)

- 對於快取設定為 "ReadWrite" 的 Premium 儲存體磁碟，則應該啟用阻礙以持續寫入。
- 對於要在 VM 重新開機後保存的磁碟機標籤，您必須以參考磁碟的 UUID 更新 /etc/fstab。另請參閱[如何將資料磁碟連接至 Linux 虛擬機器](../virtual-machines/virtual-machines-linux-classic-attach-disk.md)

以下是我們驗證能使用 Premium 儲存體的 Linux 散發套件。我們建議您升級 VM 到至少其中一個版本 (或更新版本)，以便獲得 Premium 儲存體較佳的效能和穩定性。此外，部分版本需要最新的 LIS (適用於 Microsoft Azure 的 Linux Integration Services v4.0)。請依照下面提供的連結進行下載及安裝。當我們完成其他驗證後，將繼續在清單中新增更多映像。請注意，我們的驗證顯示效能依映像而有所不同，而且也取決於工作負載特性和映像上的設定。不同的映像已針對不同種類的工作負載進行調整。
<table border="1" cellspacing="0" cellpadding="5" style="border: 1px solid #000000;">
<tbody>
<tr>
	<td><strong>配送映像</strong></td>
	<td><strong>版本</strong></td>
	<td><strong>支援的核心</strong></td>
	<td><strong>詳細資料</strong></td>
</tr>
<tr>
	<td rowspan="2"><strong>Ubuntu</strong></td>
	<td>12.04</td>
	<td>3.2.0-75.110+</td>
	<td>Ubuntu-12_04_5-LTS-amd64-server-20150119-zh-TW-30GB</td>
</tr>
<tr>
	<td>14.04+</td>
	<td>3.13.0-44.73+</td>
	<td>Ubuntu-14_04_1-LTS-amd64-server-20150123-zh-TW-30GB</td>
</tr>
<tr>
	<td><strong>Debian</strong></td>
	<td>7.x、8.x</td>
	<td>3.16.7-ckt4-1+</td>
    <td> </td>
</tr>
<tr>
	<td rowspan="2"><strong>SUSE</strong></td>
	<td>SLES 12</td>
	<td>3.12.36-38.1+</td>
	<td>suse-sles-12-priority-v20150213<br>suse-sles-12-v20150213</td>
</tr>
<tr>
	<td>SLES 11 SP4</td>
    <td>3.0.101-0.63.1+</td>
    <td> </td>
</tr>
<tr>
	<td><strong>CoreOS</strong></td>
	<td>584.0.0+</td>
	<td>3.18.4+</td>
	<td>CoreOS 584.0.0</td>
</tr>
<tr>
	<td rowspan="2"><strong>CentOS</strong></td>
	<td>6.5、6.6、6.7、7.0</td>
	<td></td>
	<td>
		<a href="http://go.microsoft.com/fwlink/?LinkID=403033&clcid=0x409"> LIS4 (必要) </a> <br/>
		*請參閱下方注意事項*
	</td>
</tr>
<tr>
	<td>7.1+</td>
	<td>3.10.0-229.1.2.el7+</td>
	<td>
		<a href="http://go.microsoft.com/fwlink/?LinkID=403033&clcid=0x409"> LIS4 (建議使用) </a> <br/>
		*請參閱下方注意事項*
	</td>
</tr>
<tr>
	<td><strong>RHEL</strong></td>
	<td>6.8+、7.2+</td>
	<td> </td>
	<td></td>
</tr>
<tr>
	<td rowspan="3"><strong>Oracle</strong></td>
    <td>6.8+、7.2+</td>
    <td> </td>
    <td> UEK4 或 RHCK </td>

</tr>
<tr>
	<td>7.0-7.1</td>
	<td> </td>
	<td>UEK4 或 RHCK 搭配 <a href="http://go.microsoft.com/fwlink/?LinkID=403033&clcid=0x409">LIS 4.1+</a></td>
</tr>
<tr>
	<td>6.4-6.7</td>
	<td></td>
	<td>UEK4 或 RHCK 搭配 <a href="http://go.microsoft.com/fwlink/?LinkID=403033&clcid=0x409">LIS 4.1+</a></td>
</tr>
</tbody>
</table>


### Openlogic CentOS 的 LIS 驅動程式

執行 OpenLogic CentOS VM 的客戶應該執行下列命令以安裝最新的驅動程式：

	sudo rpm -e hypervkvpd  ## (may return error if not installed, that's OK)
	sudo yum install microsoft-hyper-v

您需要重新開機才能啟動驅動程式。

## 價格和計費
使用 Premium 儲存體時，需考量下列計費資訊：
- 進階儲存體磁碟/Blob 大小
- 進階儲存體快照
- 輸出資料傳輸

**進階儲存體磁碟/Blob 大小**：進階儲存體磁碟/Blob 的計費依據是磁碟/Blob 的佈建大小。Azure 會將佈建大小 (無條件進位) 對應至[使用進階儲存體時的延展性和效能目標](#premium-storage-scalability-and-performance-targets)一節的表格中最接近的進階儲存體磁碟選項。儲存在進階儲存體帳戶中的所有物件都會對應至其中一個支援的佈建大小，並據此計費。Heance 會避免使用進階儲存體帳戶來儲存小型 Blob。任何已佈建的磁碟/Blob 都是依每月的進階儲存體優惠價格以每小時的方式計費。例如，如果您在佈建完 P10 磁碟的 20 小時後刪除它，則會以 20 小時計算 P10 解決方案的費用。這與寫入磁碟的實際資料量或使用的 IOPS/輸送量無關。

**進階儲存體快照**：進階儲存體上的快照會因為使用的額外容量而產生費用。如需有關快照的資訊，請參閱[建立 Blob 的快照](http://msdn.microsoft.com/library/azure/hh488361.aspx)。

**輸出資料傳輸**：[輸出資料傳輸](https://azure.microsoft.com/pricing/details/data-transfers/) (Azure 資料中心送出的資料) 會產生頻寬使用量費用。

如需有關「進階儲存體」與「進階儲存體」支援之 VM 的定價詳細資訊，請參閱：

- [Azure 儲存體定價](https://azure.microsoft.com/pricing/details/storage/)
- [虛擬機器定價](https://azure.microsoft.com/pricing/details/virtual-machines/)

## 備份
使用進階儲存體的虛擬機器可使用 Azure 備份進行備份。[其他詳細資訊](../backup/backup-azure-vms-first-look-arm.md)。

## 快速啟動

## 為虛擬機器資料磁碟建立和使用 Premium 儲存體帳戶

在這一節中，我們將使用 Azure 入口網站、Azure PowerShell 和 Azure CLI 示範下列案例︰

- 如何建立進階儲存體帳戶。
- 如何在使用進階儲存體時，建立虛擬機器並將資料磁碟連接到虛擬機器。
- 如何變更已連接到虛擬機器的資料磁碟的磁碟快取原則：

### 透過 Azure 入口網站使用進階儲存體來建立 Azure 虛擬機器

#### I.在 Azure 入口網站中建立進階儲存體帳戶

本節說明如何使用 Azure 入口網站來建立進階儲存體帳戶。

1.	登入 [Azure 入口網站](https://portal.azure.com)。如果您還沒有訂用帳戶，請參考[免費試用](https://azure.microsoft.com/pricing/free-trial/)方案。

2. 在 [中樞] 功能表上，選取 [新增] -> [資料+儲存體] -> [儲存體帳戶]。

3. 輸入儲存體帳戶的名稱。

	> [AZURE.NOTE] 儲存體帳戶名稱必須介於 3 到 24 個字元的長度，而且只能包含數字和小寫字母。
	>  
	> 儲存體帳戶名稱必須在 Azure 中是獨一無二的。Azure 入口網站會指出您選取的儲存體帳戶名稱是否已在使用中。

4. 指定所要使用的部署模型：[Resource Manager] 或 [傳統]。[資源管理員] 是建議的部署模型。如需詳細資訊，請參閱[了解資源管理員部署和傳統部署](../resource-manager-deployment-model.md)。

5. 將儲存體帳戶的效能層指定為 [進階]。

6. **本地備援儲存體 (LRS)** 是進階儲存體唯一可用的複寫選項。如需 Azure 儲存體複寫選項的詳細資訊，請參閱下方的 [Azure 儲存體複寫](storage-redundancy.md)。

7. 選取您要在其中建立新儲存體帳戶的訂用帳戶。

8. 指定新的資源群組，或選取現有的資源群組。如需資源群組的詳細資訊，請參閱 [Azure Resource Manager 概觀](../resource-group-overview.md)。

9. 選取儲存體帳戶的地理位置。您可以參考[依區域提供的 Azure 服務](https://azure.microsoft.com/regions/#services)，來確認進階儲存體是否可在選取的位置使用。

10. 按一下 [建立] 建立儲存體帳戶。

#### II.透過 Azure 入口網站建立 Azure 虛擬機器

您必須建立「進階儲存體」支援的 VM，才能使用「進階儲存體」。請依照[在 Azure 入口網站中建立 Windows 虛擬機器](../virtual-machines/virtual-machines-windows-hero-tutorial.md)中的步驟來建立新的 DS、DSv2、GS 或 Fs 虛擬機器。

#### III.透過 Azure 入口網站連接進階儲存體資料磁碟

1. 在 Azure 入口網站中尋找新的或現有的 DS、DSv2、GS 或 Fs VM。
2. 在 VM 的 [所有設定] 中，移至 [磁碟]，然後按一下 [連接新項目]。
3. 輸入資料磁碟的名稱，然後將 [類型] 選取為 [進階]。選取所需的 [大小] 和 [主機快取] 設定。

	![進階磁碟][Image1]

請參閱[如何在 Azure 入口網站中連接資料磁碟](../virtual-machines/virtual-machines-windows-attach-disk-portal.md)中的詳細步驟。

#### IV.透過 Azure 入口網站變更磁碟快取原則

1. 在 Azure 入口網站中尋找新的或現有的 DS、DSv2、GS 或 Fs VM。
2. 在 VM 的 [所有設定] 中，移至 [磁碟]，然後按一下您要變更的磁碟。
3. 將 [主機快取] 選項變更為所要的值：[無]、[唯讀] 或 [讀寫]

>[AZURE.WARNING] 變更 Azure 磁碟的快取設定會將目標磁碟中斷連接再重新連接。如果它是作業系統磁碟，則會重新啟動 VM。在變更磁碟快取設定之前，請先將可能受此中斷情況影響的所有應用程式/服務停止。

### 透過 Azure PwerShell 使用 Premium 儲存體建立 Azure 虛擬機器

#### I.在 Azure PowerShell 中建立進階儲存體帳戶

這個 PowerShell 範例示範如何建立新的 Premium 儲存體帳戶並將使用該帳戶的資料磁碟連接至新的 Azure 虛擬機器。

1. 依照[如何安裝和設定 Azure PowerShell](../powershell-install-configure.md) 中提供的步驟設定您的 PowerShell 環境。
2. 啟動 PowerShell 主控台，連接至您的訂閱，並在主控台視窗中執行下列 PowerShell Cmdlet。如此 PowerShell 陳述式所示，當您建立「進階儲存體」帳戶時，必須將 **Type** 參數指定為 **Premium\_LRS**。

		New-AzureStorageAccount -StorageAccountName "yourpremiumaccount" -Location "West US" -Type "Premium_LRS"

#### II.透過 Azure PowerShell 建立 Azure 虛擬機器

接著建立新的 DS 系列 VM，並在主控台視窗中執行下列 PowerShell Cmdlet 以指定您要使用進階儲存體。您可以使用相同的步驟建立 GS 系列 VM。在命令中指定適當的 VM 大小。例如針對 Standard\_GS2︰

    	$storageAccount = "yourpremiumaccount"
    	$adminName = "youradmin"
    	$adminPassword = "yourpassword"
    	$vmName ="yourVM"
    	$location = "West US"
    	$imageName = "a699494373c04fc0bc8f2bb1389d6106__Windows-Server-2012-R2-201409.01-en.us-127GB.vhd"
    	$vmSize ="Standard_DS2"
    	$OSDiskPath = "https://" + $storageAccount + ".blob.core.windows.net/vhds/" + $vmName + "_OS_PIO.vhd"
    	$vm = New-AzureVMConfig -Name $vmName -ImageName $imageName -InstanceSize $vmSize -MediaLocation $OSDiskPath
    	Add-AzureProvisioningConfig -Windows -VM $vm -AdminUsername $adminName -Password $adminPassword
    	New-AzureVM -ServiceName $vmName -VMs $VM -Location $location

#### III.透過 Azure PowerShell 連接進階儲存體資料磁碟

如果您希望 VM 有更多磁碟空間，請在虛擬機器建立後，於主控台視窗中執行下列 PowerShell Cmdlet，以將新資料磁碟連接至現有的「進階儲存體」支援 VM：

    	$storageAccount = "yourpremiumaccount"
    	$vmName ="yourVM"
    	$vm = Get-AzureVM -ServiceName $vmName -Name $vmName
    	$LunNo = 1
    	$path = "http://" + $storageAccount + ".blob.core.windows.net/vhds/" + "myDataDisk_" + $LunNo + "_PIO.vhd"
    	$label = "Disk " + $LunNo
    	Add-AzureDataDisk -CreateNew -MediaLocation $path -DiskSizeInGB 128 -DiskLabel $label -LUN $LunNo -HostCaching ReadOnly -VM $vm | Update-AzureVm

#### IV.透過 Azure PowerShell 變更磁碟快取原則

若要更新磁碟快取原則，請記下所連接資料磁碟的 LUN 編號。執行下列命令，將連接於 LUN 編號 2 的資料磁碟更新為 [唯讀]。

		Get-AzureVM "myservice" -name "MyVM" | Set-AzureDataDisk -LUN 2 -HostCaching ReadOnly | Update-AzureVM

>[AZURE.WARNING] 變更 Azure 磁碟的快取設定會將目標磁碟中斷連接再重新連接。如果它是作業系統磁碟，則會重新啟動 VM。在變更磁碟快取設定之前，請先將可能受此中斷情況影響的所有應用程式/服務停止。

### 透過 Azure 命令列介面使用 Premium 儲存體建立 Azure 虛擬機器

[Azure 命令列介面](../xplat-cli-install.md) (Azure CLI) 提供一組可與 Azure 平台搭配運作的開放原始碼跨平台命令。下列範例示範如何使用 Azure CLI (版本 0.8.14 和更新版本) 建立進階儲存體帳戶、新的虛擬機器，並從進階儲存體帳戶連接新的資料磁碟。

#### I.透過 Azure CLI 建立進階儲存體帳戶

````
azure storage account create "premiumtestaccount" -l "west us" --type PLRS
````

#### II.透過 Azure CLI 建立 DS 系列的虛擬機器

	azure vm create -z "Standard_DS2" -l "west us" -e 22 "premium-test-vm"
		"b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-14_10-amd64-server-20150202-zh-TW-30GB" -u "myusername" -p "passwd@123"

顯示虛擬機器的相關資訊

	azure vm show premium-test-vm

#### III.透過 Azure CLI 連接新的進階資料磁碟

	azure vm disk attach-new premium-test-vm 20 https://premiumstorageaccount.blob.core.windows.net/vhd-store/data1.vhd

顯示新資料磁碟的相關資訊

	azure vm disk show premium-test-vm-premium-test-vm-0-201502210429470316

#### IV.變更磁碟快取原則

若要使用 Azure CLI 變更其中一個磁碟上的快取原則，請執行下列命令：

		$ azure vm disk attach -h ReadOnly <VM-Name> <Disk-Name>

請注意，快取原則選項可以是 ReadOnly、None 或 ReadWrite。如需其他選項，請執行下列命令參閱說明：

		azure vm disk attach --help

>[AZURE.WARNING] 變更 Azure 磁碟的快取設定會將目標磁碟中斷連接再重新連接。如果它是作業系統磁碟，則會重新啟動 VM。在變更磁碟快取設定之前，請先將可能受此中斷情況影響的所有應用程式/服務停止。

## 常見問題集

1. **我是否可以將進階和標準資料磁碟都連接到「進階儲存體」支援的 VM？**

	是。您可將進階和標準資料磁碟都連接到「進階儲存體」支援的系列 VM。

2. **我是否可以將進階和標準資料磁碟都連接到 D、Dv2、G 或 F 系列 VM？**

	否。您只能將標準資料磁碟連接到所有不屬於「進階儲存體」支援系列的 VM。

3. **如果我從現有的 VHD (大小 為 80 GB) 建立進階資料磁碟，需要多少費用？**

	從 80 GB VHD 建立的進階資料磁碟會被視為下一個可用的進階磁碟大小 (P10 磁碟)。我們將根據 P10 磁碟價格收費。

4. **使用進階儲存體時是否有任何交易成本？**

	每個磁碟大小都有固定成本，其隨著特定數量的 IOPS 和輸送量佈建。其他成本包括輸出頻寬和快照容量 (如果適用的話)。如需詳細資料，請參閱 [Azure 儲存體定價](https://azure.microsoft.com/pricing/details/storage/)。

5. **我可以將「進階儲存體」支援之系列 VM 的開機診斷儲存在哪裡？**

	請建立標準儲存體帳戶來儲存「進階儲存體」支援之系列 VM 的開機診斷。

6. **我可以從磁碟快取取得多少 IOPS 和輸送量？**

	DS 系列的快取和本機 SSD 合併限制是每個核心 4000 IOPS，以及每個核心每秒 33 MB。GS 系列提供每個核心 5000 IOPS，以及每個核心每秒 50 MB。

7. **「進階儲存體」支援之系列 VM 中的本機 SSD 是什麼？**

	本機 SSD 是「進階儲存體」支援之系列 VM 隨附的暫時儲存體。暫時儲存體不需額外的成本。建議您不要使用此暫時儲存體或本機 SSD 來儲存應用程式資料，因為它不會保存在 Azure Blob 儲存體中。

8. **是否可以將標準儲存體帳戶轉換成進階儲存體帳戶？**

	否。不可能將標準儲存體帳戶轉換成進階儲存體帳戶 (反之亦然)。您必須以所需的類型建立新的儲存體帳戶，並將資料複製到新的儲存體帳戶 (如果適用的話)。

9. **如何將 D 系列 VM 轉換成 DS 系列 VM？**

	請參閱[移轉到 Azure 進階儲存體](storage-migration-to-premium-storage.md)移轉指南，以將您的工作負載從使用標準儲存體帳戶的 D 系列 VM 移到使用「進階儲存體」帳戶的 DS 系列 VM。

## 後續步驟

如需 Azure 進階儲存體的詳細資訊，請參閱下列文章。

### 使用 Azure 進階儲存體進行設計和實作

- [使用進階儲存體設計高效能](storage-premium-storage-performance.md)
- [在 Azure 儲存體帳戶使用 Blob 服務作業](http://go.microsoft.com/fwlink/?LinkId=521969)

### 作業指引

- [移轉到 Azure 進階儲存體](storage-migration-to-premium-storage.md)

### 部落格文章

- [Azure 進階儲存體正式推出](https://azure.microsoft.com/blog/azure-premium-storage-now-generally-available-2/)
- [宣佈 GS 系列︰將進階儲存體支援加入至公用雲端中的最大 VM](https://azure.microsoft.com/blog/azure-has-the-most-powerful-vms-in-the-public-cloud/)

[Image1]: ./media/storage-premium-storage/Azure_attach_premium_disk.png

<!---HONumber=AcomDC_0921_2016-->