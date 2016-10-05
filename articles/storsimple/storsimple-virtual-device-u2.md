<properties 
   pageTitle="StorSimple 虛擬裝置 Update 2 | Microsoft Azure"
   description="了解如何建立、部署和管理 Microsoft Azure 虛擬網路中的 StorSimple 虛擬裝置。(適用於 StorSimple Update 2)。"
   services="storsimple"
   documentationCenter=""
   authors="alkohli"
   manager="carmonm"
   editor="" />
<tags 
   ms.service="storsimple"
   ms.devlang="NA"
   ms.topic="hero-article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="09/23/2016"
   ms.author="alkohli" />

# 部署和管理 Azure 中的 StorSimple 虛擬裝置


##Overview
StorSimple 8000 系列虛擬裝置是 Microsoft Azure StorSimple 解決方案提供的另一項功能。StorSimple 虛擬裝置會在 Microsoft Azure 虛擬網路中的虛擬機器上執行，而您可以使用它來備份和複製主機上的資料。本教學課程說明如何在 Azure 中部署和管理虛擬裝置，適用於所有執行軟體版本 Update 2 和更低版本的虛擬裝置。


#### 虛擬裝置模型比較

StorSimple 虛擬裝置可以在兩種模型中使用，標準 8010 (前身為 1100) 和進階 8020 (於 Update 2 引進)。下表顯示兩個模型的比較。


| 裝置型號 | 8010<sup>1</sup> | 8020 |
|-----------------------|---------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------|
| **最大容量** | 30 TB | 64 TB |
| **Azure VM** | Standard\_A3 (4 核心、7 GB 記憶體) | Standard\_DS3 (4 核心、14 GB 記憶體) |
| **版本相容性** | 執行 Update 2 之前或更新版本的版本 | 執行 Update 2 或更新版本的版本 |
| **區域可用性** | 所有 Azure 區域 | 支援進階儲存體的 Azure 區域<br></br>如需區域的清單，請參閱 [8020 的支援區域](#supported-regions-for-8020) |
| **儲存體類型** | 將 Azure 標準儲存體使用於本機磁碟<br></br> 了解如何[建立標準儲存體帳戶]() | 將 Azure 進階儲存體使用於本機磁碟<sup>2</sup> <br></br>了解如何[建立進階儲存體帳戶](storage-premium-storage.md#create-and-use-a-premium-storage-account-for-a-virtual-machine-data-disk) |
| **工作負載指引** | 從備份的檔案的項目層級擷取 | 雲端開發和測試案例、低延遲、較高效能工作負載<br></br>災害復原的次要裝置 |
 
<sup>1</sup> *前身為 1100*。

<sup>2</sup> *8010 和 8020 會將 Azure 標準儲存體使用於雲端層。差異僅存在於裝置內的本機層*。

#### 8020 的支援區域

下表顯示目前支援 8020 的進階儲存體區域。當進階儲存體在更多區域可供使用時，這份清單將持續更新。

| S. 編號 | 目前支援的區域 |
|---------------------------------------------------------|--------------------------------|
| 1 | 美國中部 |
| 2 | 美國東部 |
| 3 | 美國東部 2 |
| 4 | 美國西部 |
| 5 | 北歐 |
| 6 | 西歐 |
| 7 | 東南亞 |
| 8 | 日本東部 |
| 9 | 日本西部 |
| 10 | 澳洲東部 |
| 11 | 澳大利亞東南部* |
| 12 | 東亞* |
| 13 | 美國中南部* |

*進階儲存體最近在這些地區啟動。

本文說明在 Azure 中部署 StorSimple 虛擬裝置的逐步程序。閱讀本文之後，您將能夠：

- 了解虛擬裝置與實體裝置的差異。

- 建立和設定虛擬裝置。

- 連接至虛擬裝置。

- 了解如何使用虛擬裝置。

本教學課程適用於所有執行 Update 2 和更新版本的 StorSimple 虛擬裝置。

## 虛擬裝置與實體裝置的差異

StorSimple 虛擬裝置是軟體形式的 StorSimple，在 Microsoft Azure 虛擬機器中的單一節點上執行。虛擬裝置支援無法使用實體裝置時的災害復原案例，並且適合用於從備份擷取的項目層級、內部部署嚴重損壞修復和雲端開發和測試案例。

#### 與實體裝置的差異

下表顯示 StorSimple 虛擬裝置和 StorSimple 實體裝置之間的一些主要差異。

| | 實體裝置 | 虛擬裝置 |
|-----------------------------|----------------------------------------------------------|-------------------------------------------------------------------------------------------|
| **位置** | 位於資料中心。 | 在 Azure 中執行。 |
| **網路介面** | 有六個網路介面：DATA 0 到 DATA 5。 | 只有一個網路介面：DATA 0。 |
| **註冊** | 在組態步驟中註冊。 | 註冊是個別的工作。 |
| **服務資料加密金鑰** | 在實體裝置上重新產生金鑰，然後以新的金鑰更新虛擬裝置。 | 無法從虛擬裝置重新產生。 |


## 虛擬裝置的必要條件

下列各節說明 StorSimple 虛擬裝置的設定必要條件。在部署虛擬裝置之前，請檢閱[使用虛擬裝置的安全性考量](storsimple-security.md#storsimple-virtual-device-security)。

#### Azure 需求

佈建虛擬裝置之前，您需要在 Azure 環境中做好下列準備：

- 針對虛擬裝置，[在 Azure 中設定虛擬網路](../virtual-network/virtual-networks-create-vnet-classic-portal.md)。如果使用進階儲存體，您必須在支援進階儲存體的 Azure 區域中建立虛擬網路。如需詳細資訊，請參閱[目前支援 8020 的區域](#supported-regions-for-8020)。
- 建議您使用 Azure 提供的預設 DNS 伺服器，而非指定您自己的 DNS 伺服器名稱。如果您的 DNS 伺服器名稱無效，或者如果 DNS 伺服器無法正確解析 IP 位址，則建立虛擬裝置將會失敗。
- 點對站及站對站都是選用的，但並非必要。如有需要，您可以針對更進階的案例設定這些選項。
- 您可以將 [Azure 虛擬機器](../virtual-machines/virtual-machines-linux-about.md) (主機伺服器) 建立在可使用由虛擬裝置公開之磁碟區的虛擬網路中。這些伺服器必須符合下列需求：
	- 是已安裝 iSCSI 啟動器軟體的 Windows 或 Linux VM。
	- 正在虛擬裝置所在的相同虛擬網路中執行。
	- 能夠透過虛擬裝置的內部 IP 位址連接到虛擬裝置的 iSCSI 目標。

- 確定您已在相同虛擬網路上設定 iSCSI 和雲端流量的支援。


#### StorSimple 需求

建立虛擬裝置之前，對 Azure StorSimple 服務進行下列更新：


- 針對要做為虛擬裝置之主機伺服器的 VM，新增[存取控制記錄](storsimple-manage-acrs.md)。

- 使用位於與虛擬裝置相同區域的[儲存體帳戶](storsimple-manage-storage-accounts.md#add-a-storage-account)。若儲存體帳戶位於不同區域，可能導致效能不佳。您可以搭配虛擬裝置使用標準或進階儲存體帳戶。如何建立[標準儲存體帳戶]()或[進階儲存體帳戶](storage-premium-storage.md#create-and-use-a-premium-storage-account-for-a-virtual-machine-data-disk)的詳細資訊

- 針對建立虛擬裝置使用與用於資料的儲存體帳戶不同的儲存體帳戶。使用相同的儲存體帳戶可能導致效能不佳。

開始之前，請確定您擁有下列資訊：

- 您的 Azure 傳統入口網站帳戶具有存取認證。

- 實體裝置中服務資料加密金鑰的複本。


## 建立和設定虛擬裝置

執行這些程序之前，請確定您符合[虛擬裝置的必要條件](#prerequisites-for-the-virtual-device)。

當您建立虛擬網路、設定 StorSimple Manager 服務，以及向該服務註冊實體 StorSimple 裝置之後，就可以使用下列步驟來建立和設定 StorSimple 虛擬裝置。

### 步驟 1：建立虛擬裝置

執行下列步驟來建立 StorSimple 虛擬裝置。

[AZURE.INCLUDE [建立虛擬裝置](../../includes/storsimple-create-virtual-device-u2.md)]

如果此步驟中的虛擬裝置建立失敗，您可能沒有網際網路的連線能力。如需詳細資訊，請在建立虛擬裝置時移至[針對網際網路連線失敗進行疑難排解](#troubleshoot-internet-connectivity-errors)。


### 步驟 2：設定和註冊虛擬裝置

開始此程序之前，請確定您擁有服務資料加密金鑰的複本。服務資料加密金鑰已在您設定第一個 StorSimple 裝置時建立，且系統已指示您將它儲存在安全的位置。如果您沒有服務資料加密金鑰的複本，就必須連絡 Microsoft 支援服務以尋求協助。

執行下列步驟來設定和註冊 StorSimple 虛擬裝置。[AZURE.INCLUDE [設定和註冊虛擬裝置](../../includes/storsimple-configure-register-virtual-device.md)]

### 步驟 3：(選擇性) 修改裝置組態設定

下一節說明如果您想要使用 CHAP、StorSimple Snapshot Manager 或變更裝置系統管理員密碼，StorSimple 虛擬裝置所需的裝置組態設定。

#### 設定 CHAP 起始端

此參數包含您的虛擬裝置 (目標) 預期從嘗試存取磁碟區的起始端 (伺服器) 收到的認證。在這個驗證過程中，起始端將提供 CHAP 使用者名稱和 CHAP 密碼，以便向您的裝置識別其身分。如需詳細步驟，請移至[設定裝置的 CHAP](storsimple-configure-chap.md#unidirectional-or-one-way-authentication)。

#### 設定 CHAP 目標

此參數包含具有 CHAP 功能的起始端要求相互或雙向驗證時，您的虛擬裝置所使用的認證。在這個驗證程序中，您的虛擬裝置將使用反向 CHAP 使用者名稱和反向 CHAP 密碼，向起始端識別其身分。請注意，CHAP 目標設定為全域設定。套用這些項目時，與虛擬裝置連接的所有磁碟區都將使用 CHAP 驗證。如需詳細步驟，請移至[設定裝置的 CHAP](storsimple-configure-chap.md#bidirectional-or-mutual-authentication)。

#### 設定 StorSimple Snapshot Manager 密碼

StorSimple Snapshot Manager 軟體位於您的 Windows 主機上，而且可讓系統管理員以本機和雲端快照的形式管理 StorSimple 裝置的備份。

>[AZURE.NOTE] 對虛擬裝置而言，您的 Windows 主機就是 Azure 虛擬機器。

當您在 StorSimple Snapshot Manager 中設定裝置時，系統將提示您提供 StorSimple 裝置 IP 位址和密碼來驗證您的儲存裝置。如需詳細步驟，請移至[設定 StorSimple Snapshot Manager 密碼](storsimple-change-passwords.md#change-the-storsimple-snapshot-manager-password)。

#### 變更裝置系統管理員密碼 

當您使用 Windows PowerShell 介面來存取虛擬裝置時，需要輸入裝置系統管理員密碼。為了確保資料的安全性，您必須變更此密碼，才能使用虛擬裝置。如需詳細步驟，請移至[設定裝置系統管理員密碼](storsimple-change-passwords.md#change-the-device-administrator-password)。

## 遠端連接至虛擬裝置
預設不會啟用透過 Windows PowerShell 介面遠端存取您的虛擬裝置。您需要先在虛擬裝置上啟用遠端管理，然後在將用來存取虛擬裝置的用戶端上啟用它。

遠端連接的兩個步驟程序詳述如下。

### 步驟 1：設定遠端管理

執行下列步驟來設定 StorSimple 虛擬裝置的遠端管理。

[AZURE.INCLUDE [透過 HTTP 設定虛擬裝置的遠端管理](../../includes/storsimple-configure-remote-management-http-device.md)]

### 步驟 2：遠端存取虛擬裝置

當您在 StorSimple 裝置設定頁面上啟用遠端管理之後，就可以使用 Windows PowerShell 遠端處理功能，從相同虛擬網路內部的另一部虛擬機器連接到虛擬裝置；例如，您可以從已設定且用來連接 iSCSI 的主機 VM 進行連接。在大部分的部署中，您應該都已開啟公用端點來存取您可用來存取虛擬裝置的主機 VM。

>[AZURE.WARNING] **為了加強安全性，強烈建議您在連接到端點時應使用 HTTPS，並且在完成 PowerShell 遠端工作階段之後刪除端點。**

您應遵循[遠端連接至 StorSimple 裝置](storsimple-remote-connect.md)中的程序，來設定虛擬裝置的遠端處理功能。

## 直接連接至虛擬裝置

您也可以直接連接至虛擬裝置。如果您要從虛擬網路或 Microsoft Azure 環境以外的另一部電腦直接連接到虛擬裝置，就必須依照下列程序中的說明來建立其他端點。

執行下列步驟，在虛擬裝置上建立公用端點。

[AZURE.INCLUDE [在虛擬裝置上建立公用端點](../../includes/storsimple-create-public-endpoints-virtual-device.md)]

建議您從相同虛擬網路內部的另一部虛擬機器進行連接，因為此做法可將虛擬網路上的公用端點數降到最低。使用此方法時，您只需透過遠端桌面工作階段連接到虛擬機器，然後依照您在區域網路上設定任何其他 Windows 用戶端的方式來設定該虛擬機器即可。您不需附加公用連接埠號碼，因為該連接埠將會是已知的。

## 使用 StorSimple 虛擬裝置

現在您已建立並設定 StorSimple 虛擬裝置，就已準備好開始使用它。您可以在虛擬裝置上使用磁碟區容器、磁碟區及備份原則，正如同您在實體 StorSimple 裝置 上所做的一樣。唯一的差異在於，您必須確定會從裝置清單中選取虛擬裝置。請參閱[使用 StorSimple Manager 服務來管理虛擬裝置](storsimple-manager-service-administration.md)，以取得虛擬裝置的各種管理工作的逐步程序。

下列各節將討論您在使用虛擬裝置時將遇到的一些差異。

### 維護 StorSimple 虛擬裝置

由於它是純軟體裝置，相較於實體裝置的維護，虛擬裝置的維護可說是最基本的。您有下列選擇：

- **軟體更新** - 您可以檢視上次更新軟體的日期，以及任何更新狀態訊息。如果想要檢查是否有新的更新，可以使用頁面底部的 [**掃描更新**] 按鈕來執行手動掃描。如果有可用更新，請按一下 [**安裝更新**] 加以安裝。因為虛擬裝置上只有單一介面，這表示在套用更新時將造成服務些微中斷。虛擬裝置將關閉並重新啟動 (如有必要)，以套用任何已發行的更新。如需逐步程序，請移至[更新您的裝置](storsimple-update-device.md#install-regular-updates-via-the-azure-classic-portal)。
- **支援套件** - 您可以建立並上傳支援套件，以協助 Microsoft 支援服務針對您的虛擬裝置問題進行疑難排解。如需逐步程序，請移至[建立和管理支援套件](storsimple-create-manage-support-package.md)。

### 虛擬裝置的儲存體帳戶

儲存體帳戶是建立來供 StorSimple Manager 服務、虛擬裝置和實體裝置使用。當您建立儲存體帳戶時，建議您在易記名稱中使用區域識別碼，有助於確保所有系統元件的區域都是一致的。在虛擬裝置中，所有元件都必須位於相同區域中，以防止發生效能問題。

如需逐步程序，請移至[新增儲存體帳戶](storsimple-manage-storage-accounts.md#add-a-storage-account)。

### 停用 StorSimple 虛擬裝置

停用虛擬裝置，將刪除佈建它時所建立的 VM 和資源。停用虛擬裝置之後，就無法將它還原為先前的狀態。停用虛擬裝置之前，請確實停止或刪除其依存的用戶端和主機。

停用虛擬裝置會導致下列動作：

- 移除虛擬裝置。

- 移除為虛擬裝置建立的 OS Disk 和資料磁碟。

- 保留在佈建期間建立的託管服務和虛擬網路。如果您不使用它們，就應該手動加以刪除。

- 保留為虛擬裝置建立的雲端快照。

如需逐步程序，請移至[停用及刪除 StorSimple 裝置](storsimple-deactivate-and-delete-device.md)。

只要虛擬裝置在 StorSimple Manager 服務頁面上顯示為已停用，您就能從 [裝置] 頁面的裝置清單中刪除該虛擬裝置。


### 啟動、停止和重新啟動虛擬裝置
不同於 StorSimple 實體裝置，在 StorSimple 虛擬裝置上並沒有開啟電源或關閉電源按鈕可供使用。但是，有時您可能需要停止並重新啟動虛擬裝置。例如，有些更新可能必須在 VM 重新啟動後才能完成更新程序。若要啟動、停止和重新啟動虛擬裝置，最簡單的方式是使用虛擬機器管理主控台。

當您檢視管理主控台時，虛擬裝置狀態是 [執行中]，因為它在建立之後會預設為啟動狀態。您隨時都能啟動、停止及重新啟動虛擬機器。

[AZURE.INCLUDE [停止和重新啟動虛擬裝置](../../includes/storsimple-stop-restart-virtual-device.md)]

### 重設為原廠預設值

如果您決定要重新設定全新的虛擬裝置，請直接停用並加以刪除，然後建立新的虛擬裝置。和重設實體裝置的情況一樣，新的虛擬裝置將不會安裝任何更新，因此，請確實檢查有無更新，然後再開始使用它。


## 容錯移轉至虛擬裝置

災害復原 (DR) 是針對其設計 StorSimple 虛擬裝置的重要案例之一。在此案例中，實體 StorSimple 裝置或整個資料中心可能無法使用。幸運的是，您可以使用虛擬裝置，在替代位置中還原作業。在 DR 期間，來源裝置的磁碟區容器會變更擁有權，並移轉到虛擬裝置。DR 的必要條件是必須已建立並設定虛擬裝置、磁碟區容器內的所有磁碟區都要離線，以及磁碟區容器應該具有相關聯的雲端快照。

>[AZURE.NOTE] 
>
> - 當使用虛擬裝置作為 DR 的次要裝置時，請記住，8010 有 30 TB 的標準儲存體，8020 有 64 TB 的進階儲存體。較高容量的 8020 虛擬裝置可能比較適合 DR 案例。
> - 您無法從執行 Update 2 的裝置容錯移轉或複製到執行 Update 1 之前版本軟體的裝置。但是，您可以將執行 Update 2 的裝置容錯移轉至執行 Update 1 (1.1 或 1.2) 的裝置。

如需逐步程序，請移至[容錯移轉至虛擬裝置](storsimple-device-failover-disaster-recovery.md#fail-over-to-a-storsimple-virtual-device)。
 

## 關閉或刪除虛擬裝置

如果您先前已設定並使用 StorSimple 虛擬裝置，但現在不想再因為使用而產生計算費用，您可以關閉虛擬裝置。關閉虛擬裝置，並不會刪除其作業系統或儲存體中的資料磁碟。這只會停止您的訂用帳戶所產生的費用，但作業系統和資料磁碟的儲存體費用仍會繼續計算。

如果您刪除或關閉虛擬裝置，StorSimple Manager 服務的 [裝置] 頁面上會將其顯示為 [離線]。如果您也想要刪除虛擬裝置所建立的備份，可以選擇停用或刪除裝置。如需詳細資訊，請參閱[停用及刪除 StorSimple 裝置](storsimple-deactivate-and-delete-device.md)。

[AZURE.INCLUDE [關閉虛擬裝置](../../includes/storsimple-shutdown-virtual-device.md)]

[AZURE.INCLUDE [刪除虛擬裝置](../../includes/storsimple-delete-virtual-device.md)]

   
## 針對網際網路連線錯誤進行疑難排解 

在虛擬裝置建立期間，如果沒無法連線到網際網路，則建立步驟將會失敗。若要針對是否因為網際網路連線而失敗進行疑難排解，請在 Azure 傳統入口網站中執行下列步驟︰

1. 在 Azure 中建立 Windows server 2012 虛擬機器。此虛擬機器應該使用與虛擬裝置使用的相同儲存體帳戶、VNet 和子網路。如果您在 Azure 中已經有使用相同儲存體帳戶、Vnet 和子網路的現有 Windows Server 主機，您也可以使用它來針對網際網路連線進行疑難排解。
2. 遠端登入在先前步驟中建立的虛擬機器。
3. 在虛擬機器內開啟命令視窗 (Win + R，然後輸入 `cmd`)。
4. 在出現提示時執行下列命令。

	`nslookup windows.net`

5. 如果 `nslookup` 失敗，則網際網路連線失敗會導致虛擬裝置無法註冊到 StorSimple Manager 服務。
6. 對虛擬網路進行必要的變更，確保虛擬裝置能夠存取 Azure 網站，例如 "windows.net"。

## 後續步驟

- 了解如何[使用 StorSimple Manager 服務管理虛擬裝置](storsimple-manager-service-administration.md)。
 
- 了解如何[從備份組還原 StorSimple 磁碟區](storsimple-restore-from-backup-set.md)。

<!---HONumber=AcomDC_0928_2016-->