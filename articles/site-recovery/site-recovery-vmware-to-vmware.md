<properties
	pageTitle="將內部部署 VMware 虛擬機器或實體伺服器複寫至次要網站 | Microsoft Azure"
	description="使用這篇文章以使用 Azure Site Recovery，將 VMware VM 或 Windows/Linux 實體伺服器複寫至次要網站。"
	services="site-recovery"
	documentationCenter=""
	authors="nsoneji"
	manager="jwhit"
	editor=""/>

<tags
	ms.service="site-recovery"
	ms.workload="backup-recovery"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/16/2016"
	ms.author="nisoneji"/>


# 將內部部署 VMware 虛擬機器或實體伺服器複寫至次要網站


## 概觀

Azure Site Recovery 中的 InMage Scout 可提供內部部署 VMware 網站之間的即時複寫。InMage Scout 包含在 Azure Site Recovery 服務訂用帳戶中。


## 必要條件

**Azure 帳戶**：您將需要一個 [Microsoft Azure](https://azure.microsoft.com/) 帳戶。您可以從[免費試用](https://azure.microsoft.com/pricing/free-trial/)開始。[深入了解](https://azure.microsoft.com/pricing/details/site-recovery/) Site Recovery 價格。


## 步驟 1：建立保存庫

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 按一下 [資料服務] > [復原服務]，然後按一下 [Site Recovery 保存庫]。
3. 按一下 [新建] > [快速建立]。
4. 在 [**名稱**] 中，輸入保存庫的易記識別名稱。
5. 在 [**區域**] 中，選取保存庫的地理區域。若要查看支援的地區，請參閱 [Azure Site Recovery 定價](https://azure.microsoft.com/pricing/details/site-recovery/)。

檢查狀態列，以確認是否順利建立保存庫。保存庫在主要 [復原服務] 頁面上會列為 [使用中]。

## 步驟 2：設定保存庫並下載 InMage Scout 元件

1. 按一下 [建立保存庫]。
2. 在 [復原服務] 頁面中，按一下保存庫以開啟 [快速啟動] 頁面。
3. 在下拉式清單中，選取 [**兩個內部部署 VMWare 網站之間**]。
4. 下載 InMage Scout。所有必要元件的安裝程式檔案都在下載的 .zip 檔中。


## 步驟 3：安裝元件更新

深入了解最新的[更新](#updates)。您將會依下列順序在伺服器上安裝更新檔案：

1. RX 伺服器 (如果有的話)
2. 設定伺服器
3. 處理序伺服器
3. 主要目標伺服器
4. vContinuum 伺服器
5. 來源伺服器 (僅適用於 Windows Server)

安裝更新，如下所示：

1. 下載 [update](https://aka.ms/asr-scout-update3) .zip 檔案。此 ++zip 檔案包含下列檔案：

	-  RX_8.0.3.0_GA\_Update_3_6684045\_17Mar16.tar.gz
	-  CX\_Windows_8.0.3.0_GA\_Update_3_5048668\_16Mar16.exe
	-  UA\_Windows_8.0.3.0_GA\_Update_3_7101745\_04Apr16.exe
	-  UA\_RHEL6-64_8.0.3.0_GA\_Update_3_7101745\_04Apr16.zip
	-  vCon\_Windows_8.0.3.0_GA\_Update_3_6873369\_16Mar16.exe

2. 解壓縮 .zip 檔。
3. **RX 伺服器**：將 **RX_8.0.3.0_GA\_Update_3_6684045\_17Mar16.tar.gz** 複製到 RX 伺服器並將其解壓縮。在解壓縮的資料夾中執行 **/Install**。
4. **設定伺服器/處理序伺服器**：將 **CX\_Windows_8.0.3.0_GA\_Update_3_5048668\_16Mar16.exe** 複製到設定伺服器和處理序伺服器。連按兩下加以執行。
5. **Windows 主要目標伺服器**：若要更新整合代理程式，將 **UA\_Windows_8.0.3.0_GA\_Update_3_7101745\_04Apr16.exe** 複製到主要目標伺服器。連按兩下加以執行。請注意，整合代理程式也適用於來源伺服器。您也應該將其安裝在來源伺服器上，如本清單稍後所述。
6. **Linux 主要目標伺服器**：若要更新整合代理程式，將 **UA\_RHEL6-64_8.0.3.0_GA\_Update_3_7101745\_04Apr16.zip** 複製到主要目標伺服器並將其解壓縮。在解壓縮的資料夾中執行 **/Install**。
7. **vContinuum 伺服器**：將 **vCon\_Windows_8.0.3.0_GA\_Update_3_6873369\_16Mar16.exe** 複製到 vContinuum 伺服器。確定您已經關閉 vContinuum 精靈。連按兩下檔案加以執行。
8. **Windows 來源伺服器**：若要更新整合代理程式，將 **UA\_Windows_8.0.3.0_GA\_Update_3_7101745\_04Apr16.exe** 複製到來源伺服器。連按兩下加以執行。

## 步驟 4：設定複寫
1. 設定來源與目標 VMware 網站之間的複寫。
2. 如需指引，請使用隨產品下載的 InMage Scout 文件。或者，您可以存取下列文件：

	- [版本資訊](https://aka.ms/asr-scout-release-notes)
	- [相容性矩陣](https://aka.ms/asr-scout-cm)
	- [使用者指南](https://aka.ms/asr-scout-user-guide)
	- [RX 使用者指南](https://aka.ms/asr-scout-rx-user-guide)
	- [快速安裝指南](https://aka.ms/asr-scout-quick-install-guide)


## 更新

### Azure Site Recovery Scout 8.0.1 Update 3
Update 3 包含下列錯誤修正和增強功能：

- 設定伺服器和 RX 位於 Proxy 後方時無法向 Site Recovery 保存庫註冊。
- 健全狀況報告中未更新不符合復原點目標 (RPO) 的時數。
- 當 ESX 硬體詳細資料或網路詳細資料包含任何UTF-8 字元時，設定伺服器不會與 RX 同步處理。
- Windows Server 2008 R2 網域控制站無法在復原後啟動。
- 離線同步處理未如預期般運作。
- 在虛擬機器 (VM) 容錯移轉後，複寫配對刪除因為時間太長而在 CX UI 中停滯，且使用者無法完成容錯回復或繼續作業。
- 一致性作業完成的整體快照作業已進行最佳化，有助於減少應用程式中斷連接 SQL 用戶端等。
- 降低在 Windows 上建立快照所需的記憶體使用量，改善一致性工具 (VACP.exe) 的效能。
- 密碼超過 16 個字元時，推送安裝服務會當機。
- vContinuum 不會在認證變更時檢查和提示輸入新的 vCenter 認證。
- 在 Linux 上，主要目標快取管理員 (cachemgr) 並未從處理序伺服器下載檔案，導致發生複寫配對節流。
- 當所有節點上的實體容錯移轉叢集 (MSCS) 磁碟順序不同時，某些叢集磁碟區不會設定複寫。<br/>請注意，必須要重新保護叢集才能利用此修正程式。
- 在 RX 從 Scout 7.1 升級至 Scout 8.0.1 之後，SMTP 功能並未如預期般運作。
- 已在記錄檔中加入更多統計資料，以便復原作業追蹤完成此作業所需的時間。
- 已加入來源伺服器上 Linux 作業系統的支援：
	- Red Hat Enterprise Linux (RHEL) 6 Update 7
	- CentOS 6 Update 7
- CX 和 RX UI 現在可以針對進入點陣圖模式的配對顯示通知。
- RX 中已加入下列安全性修正：

**問題描述**|**實作程序**
---|---
透過竄改參數略過授權|對不適用使用者的存取權限制。
跨網站偽造要求|實作針對每一頁隨機產生的頁面權杖概念。<br/>有了這個，您會看見：<li> 相同使用者只有單一登入執行個體。</li><li> 頁面重新整理無法運作，而重新導向至儀表板。</li>
惡意檔案上傳|將檔案限制於特定副檔名。允許的副檔名︰7z、aiff、asf、avi、bmp、csv、doc、docx、fla、flv、gif、gz、gzip、jpeg、jpg、log、mid、mov、mp3、mp4、mpc、mpeg、mpg、ods、odt、pdf、png、ppt、pptx、pxd、qt、ram、rar、rm、rmi、rmvb、rtf、sdc、sitd、swf、sxc、sxw、tar、tgz、tif、tiff、txt、vsd、wav、wma、wmv、xls、xlsx、xml、zip。
持續性跨網站指令碼 | 加入輸入驗證。


>[AZURE.NOTE]
>
>-	所有的 Site Recovery 更新都是累計的。Update3 有 Update1 和 Update2 的所有修正。Update 3 可以直接套用於 8.0.1 GA。
>-	一旦在系統上套用設定伺服器和 RX 更新便無法回復。

### Azure Site Recovery Scout 8.0.1 Update 2 (Update 03Dec15)

Update 2 中的修正包括：

- **設定伺服器**：修正當設定伺服器登錄在 Site Recovery 時，會阻止 31 天免費計量功能正常運作的問題。
- **整合代理程式**：修正 Update 1 中導致 8.0 版升級至 8.0.1 版之後，更新未安裝在主要目標伺服器上的問題。


### Azure Site Recovery Scout 8.0.1 Update 1

Update 1 包含下列錯誤修正和新功能：

- 每個伺服器執行個體享有 31 天的免費保護。這可讓您測試功能或設定概念證明。
	- 從使用 Site Recovery Scout 第一次保護伺服器的時間開始計算的前 31 天，伺服器上的所有作業 (包括容錯移轉和容錯回復) 都是免費的。
	- 從第 32 天起，每個受保護的伺服器都會依照標準執行個體費率，向客戶擁有的網站收取 Azure Site Recovery 保護費用。
	- 在 Azure Site Recovery 保存庫的 [儀表板] 頁面上，隨時可取得目前計費的受保護伺服器數目。
- 加入對 vSphere 命令列介面 (vCLI) 5.5 Update 2 的支援。
- 已加入來源伺服器上 Linux 作業系統的支援：
	- RHEL 6 Update 6
	- RHEL 5 Update 11
	- CentOS 6 Update 6
	- CentOS 5 Update 11
- 用以處理下列問題的錯誤修正：
	- 設定伺服器或 RX 伺服器的保存庫註冊失敗。
	- 當叢集虛擬機器在恢復期間重新受到保護，叢集磁碟區就不會如預期般出現。
	- 當主要目標伺服器裝載於與內部部署實際執行虛擬機器不同的 ESXi 伺服器時，容錯回復會失敗。
	- 升級至 8.0.1 時，組態檔案權限會變更，進而影響保護和作業。
	- 不會如預期般強制執行重新同步臨界值，因而導致不一致的複寫行為。
	- RPO 設定並未正確地顯示在設定伺服器介面中。未壓縮的資料值未正確地顯示壓縮的值。
	-  未如預期在 vContinuum 精靈中刪除移除作業，而且未從設定伺服器介面中刪除複寫作業。
	-  在 vContinuum 精靈中，在 MSCS 虛擬機器的保護期間，按一下磁碟檢視中的 [詳細資料]，磁碟會自動取消選取。
	- 在實體到虛擬 (P2V) 案例期間，必要的 HP 服務 (例如 CIMnotify、CqMgHost) 不會移至虛擬機器復原中的 [手動]。這會導致額外的開機時間。
	- 主要目標伺服器上有超過 26 個磁碟時，Linux 虛擬機器的保護就會失敗。

## 後續步驟

若有任何問題，請造訪 [Azure Recovery Services 論壇](https://social.msdn.microsoft.com/forums/azure/home?forum=hypervrecovmgr) (英文)。

<!---HONumber=AcomDC_0921_2016-->