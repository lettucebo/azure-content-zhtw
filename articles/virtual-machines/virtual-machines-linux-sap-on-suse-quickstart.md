<properties
   pageTitle="測試 Microsoft Azure SUSE Linux VM 上的 SAP NetWeaver | Microsoft Azure"
   description="在 Microsoft Azure SUSE Linux VM 上測試 SAP NetWeaver"
   services="virtual-machines-linux"
   documentationCenter=""
   authors="hermanndms"
   manager="timlt"
   editor=""
   tags="azure-resource-manager"
   keywords=""/>
<tags
   ms.service="virtual-machines-linux"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="vm-linux"
   ms.workload="infrastructure-services"
   ms.date="09/15/2016"
   ms.author="hermannd"/>

# 在 Microsoft Azure SUSE Linux VM 上執行 SAP NetWeaver


這篇文章描述在 Microsoft Azure SUSE Linux 虛擬機器 (VM) 上執行 SAP NetWeaver 時應考量的各種事項。自 2016 年 5 月 19 日起，在 Azure 的 SUSE Linux VM 上已正式支援 SAP NetWeaver。如需有關 Linux 版本、SAP 核心版本等等的所有詳細資料，請參閱 SAP 附註 1928533＜Azure 上的 SAP 應用程式︰支援的產品和 Azure VM 類型＞。如需 Linux VM 上 SAP 的進一步相關文件，請參閱︰[在 Linux 虛擬機器 (VM) 上使用 SAP](virtual-machines-linux-sap-get-started.md)。


下列資訊應該協助您避免一些潛在的問題。

## Azure 上用來執行 SAP 的 SUSE 映像

若要在 Azure 上執行 SAP NetWeaver，請只使用 SUSE Linux Enterprise Server SLES 12 (SPx) - 另請參閱 SAP 附註 1928533。您可以在 Azure Marketplace 找到特殊的 SUSE 映像 ("SLES 11 SP3 for SAP CAL")，但這不適用一般用途。請勿使用該映像，因為它是保留給 [SAP 雲端應用裝置程式庫](https://cal.sap.com/)解決方案使用的映像。

針對 Azure 上的所有新測試和安裝，您應該使用 Azure Resource Manager。若要尋找 SUSE SLES 映像和使用 Azure Powershell 或 Azure 命令列介面 (CLI) 的版本，請使用下列命令。然後可以使用輸出，例如，在 json 範本中定義作業系統映像，以部署新的 SUSE Linux VM。下列 PowerShell 命令適用於 Azure Powershell 版本 1.0.1 或更新版本。

* 尋找現有的發佈者，包括 SUSE：

   ```
   PS  : Get-AzureRmVMImagePublisher -Location "West Europe"  | where-object { $_.publishername -like "*US*"  }
   CLI : azure vm image list-publishers westeurope | grep "US"
   ```

* 從 SUSE 尋找現有的供應項目：

   ```
   PS  : Get-AzureRmVMImageOffer -Location "West Europe" -Publisher "SUSE"
   CLI : azure vm image list-offers westeurope SUSE
   ```

* 尋找 SUSE SLES 供應項目：

   ```
   PS  : Get-AzureRmVMImageSku -Location "West Europe" -Publisher "SUSE" -Offer "SLES"
   CLI : azure vm image list-skus westeurope SUSE SLES
   ```

* 尋找 SLES SKU 的特定版本：

   ```
   PS  : Get-AzureRmVMImage -Location "West Europe" -Publisher "SUSE" -Offer "SLES" -skus "12"
   CLI : azure vm image list westeurope SUSE SLES 12
   ```

## 在 SUSE VM 中安裝 WALinuxAgent

稱為 WALinuxAgent 的代理程式是 Azure Marketplace 中的 SLES 映像的一部分。如需手動安裝 (例如從內部部署上傳 SLES OS 虛擬磁碟 (VHD)) 的相關資訊，請參閱：

- [OpenSUSE](http://software.opensuse.org/package/WALinuxAgent)

- [Azure](virtual-machines-linux-endorsed-distros.md)

- [SUSE](https://www.suse.com/communities/blog/suse-linux-enterprise-server-configuration-for-windows-azure/)

## SAP「增強型監視」

SAP「增強型監視」是在 Azure 上執行 SAP 的必要先決條件。請查看 SAP 附註 2191498＜Linux 搭配 Azure 上的 SAP：增強型監視＞中的詳細資料。

## 將 Azure 資料磁碟連結至 Azure Linux VM

請絕對避免透過裝置識別碼將 Azure 資料磁碟掛接到 Azure Linux VM。相反地，請使用全域唯一識別碼 (UUID)。舉例來說，使用圖形工具來掛接 Azure 資料磁碟時應多加留意。請仔細檢查 /et/fstab 中的項目。

裝置識別碼的問題在於它可能會變更，然後 Azure VM 可能會在開機程序停止回應。若要解決這個問題，您可以在 /etc/fstab 中新增 nofail 參數。但請留意 nofail，因為應用程式可能會使用先前的掛接點，而且萬一外部 Azure 資料磁碟未在開機期間掛接，可能會寫入根目錄檔案系統。

透過 UUID 掛接的唯一例外狀況是連接 OS 磁碟以進行疑難排解，如下列章節所述。

## 針對無法再存取的 SUSE VM 進行疑難排解

有時候 Azure 上的 SUSE VM 會停在開機程序 (例如，發生與掛接磁碟相關的錯誤)。您可以使用 Azure 入口網站中的 Azure 虛擬機器 v2 開機診斷功能來確認問題。如需詳細資訊，請參閱[開機診斷](https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/)。

解決這個問題的其中一個方法，是從損毀的 VM 將 OS 磁碟連接到 Azure 上的另一部 SUSE VM。接下來是進行適當的變更，如編輯 /etc/fstab 或移除網路 udev 規則，如下一節所述。

請將一項重要事項納入考量。從相同的 Azure Marketplace 映像 (例如 SLES 11 SP4) 部署數個 SUSE VM，會導致 OS 磁碟一律將由相同的 UUID 掛接。因此，使用 UUID 從以相同 Azure Marketplace 映像部署的不同 VM 連接 OS 磁碟，將會導致兩個相同的 UUID。這會造成問題，並且可能讓原本進行疑難排解的 VM 實際上從連接和損毀的 OS 磁碟開機，而不是原始的磁碟。

避免這種情形的做法有二種：

* 針對進行疑難排解的 VM 使用不同的 Azure Marketplace 映像 (例如 SLES 11 SPx，而不是 SLES 12)。
* 不要使用 UUID 從另一部 VM 連接損毀的 OS 磁碟，而是使用其他項目

## 從內部部署將 SUSE VM 上傳至 Azure

如需從內部部署環境將 SUSE VM 上傳到 Azure 的步驟說明，請參閱[準備適用於 Azure 的 SLES 或 openSUSE 虛擬機器](virtual-machines-linux-create-upload-vhd-suse.md)。

如果您想要上傳 VM，但是希望結束時沒有取消佈建步驟 (例如，為了保留現有的 SAP 安裝及主機名稱)，請檢查下列項目：

* 確定 OS 磁碟是使用 UUID 掛接，而不是透過裝置識別碼。只在 /etc/fstab 變更 UUID 對作業系統磁碟而言並不夠。同時，不要忘了透過 Yast 或編輯 /boot/grub/menu.lst 來調整開機載入器。
* 如果您針對 SUSE OS 磁碟使用 VHDX 格式，為了上傳至 Azure 而將它轉換為 VHD，很有可能網路裝置已從 eth0 變更為 eth1。為避免稍後在 Azure 上開機時發生問題，請改回 eth0，如[修正複製之 SLES 11 VMware 中的 eth0](https://dartron.wordpress.com/2013/09/27/fixing-eth1-in-cloned-sles-11-vmware/) 所述。

除了文章中說明的以外，建議您一併移除下列項目：

   # sudo mv /lib/udev/rules.d/75-persistent-net-generator.rules /var/lib/waagent/

只要沒有多個 NIC，您還可以安裝 Azure Linux 代理程式 (waagent) 來避免潛在問題。

## 在 Azure 上部署 SUSE VM

您應該使用新 Azure Resource Manager 模型中的 JSON 範本檔案來建立新的 SUSE VM。建立 JSON 範本檔案後，就可以使用下列 CLI 命令做為 Powershell 的替代來部署 VM：

   ```
   azure group deployment create "<deployment name>" -g "<resource group name>" --template-file "<../../filename.json>"

   ```
如需有關 JSON 範本檔案的更多詳細資料，請參閱[編寫 Azure Resource Manager 範本](resource-group-authoring-templates.md)和 [Azure 快速入門範本](https://azure.microsoft.com/documentation/templates/)。

如需有關 CLI 和 Azure Resource Manager 的更多詳細資料，請參閱[搭配使用 Mac、Linux 和 Windows 適用的 Azure CLI 與 Azure Resource Manager](xplat-cli-azure-resource-manager.md)。

## SAP 授權與硬體金鑰

針對官方的 SAP-Azure 憑證，已經有新的機制可以計算 SAP 授權使用的 SAP 硬體金鑰。要使用此機制，必須調整 SAP 核心。Linux 先前的 SAP 核心版本不包括此程式碼變更。因此，在某些情況下 (例如 Azure VM 調整大小)，SAP 硬體金鑰已發生變更，而 SAP 授權已不再有效在最新的 SAP Linux 核心中已解決此問題。如需詳細資料，請查看 SAP 附註 1928533。

## SUSE sapconf 封裝 / tuned-adm

SUSE 提供稱為 "sapconf" 的封裝，這組封裝負責管理一組 SAP 特定設定。如需有關此封裝之用途、安裝方式及使用方式的更多詳細資料，請參閱[使用 sapconf 來準備要執行 SAP 系統的 SUSE Linux Enterprise Server](https://www.suse.com/communities/blog/using-sapconf-to-prepare-suse-linux-enterprise-server-to-run-sap-systems/) 和[何謂 sapconf 或如何準備要執行 SAP 系統的 SUSE Linux Enterprise Server？](http://scn.sap.com/community/linux/blog/2014/03/31/what-is-sapconf-or-how-to-prepare-a-suse-linux-enterprise-server-for-running-sap-systems)。

同時還提供一個新工具來取代 sapconf - tuned-adm。您可以在下列這兩個連結中找到關於這個工具的更多詳細資料。

您可以在[這裡](https://www.suse.com/documentation/sles-for-sap-12/book_s4s/data/sec_s4s_configure_sapconf.html)找到關於 tuned-adm 設定檔 sap-hana 的 SLES 文件

您可以在第 6.2 章中的[這裡](https://www.suse.com/documentation/sles-for-sap-12/pdfdoc/book_s4s/book_s4s.pdf)，找到如何利用 tuned-adm 針對 SAP 工作負載微調系統


## 分散式 SAP 安裝中的 NFS 共用

如果您擁有分散式安裝中 (例如，想要在個別的 VM 安裝資料庫和 SAP 應用程式伺服器)，可以透過網路檔案系統 (NFS) 共用 /sapmnt 目錄。如果為 /sapmnt 建立 NFS 共用之後安裝步驟發生問題，請檢查是否已為共用設定 "no\_root\_squash"。

## 邏輯磁碟區

在過去，如果使用者需要一個跨多個 Azure 資料磁碟的大型邏輯磁碟區 (例如用於 SAP 資料庫)，會建議使用 mdadm，因為 lvm 在 Azure 上尚未完全通過驗證。若要了解如何使用 mdadm 在 Azure 上設定 Linux RAID，請參閱[在 Linux 上設定軟體 RAID](virtual-machines-linux-configure-raid.md)。自 2016 年 5 月開始的同時起，lvm 在 Azure 上也已獲得完全支援，並可做為 mdadm 的替代方案。如需有關 Azure 上 lvm 的其他資訊，請參閱[在 Azure 中的 Linux VM 上設定 LVM](virtual-machines-linux-configure-lvm.md)。


## Azure SUSE 儲存機制

如果您在存取標準 Azure SUSE 儲存機制發生問題，可以使用一個簡單的命令來重設它。在 Azure 區域建立私用的 OS 映像，然後將該映像複製到不同的區域時，若您想依據這個私用的作業系統映像部署新的 VM 時，就可能會發生此問題。只要在 VM 中執行下列命令：

   ```
   service guestregister restart
   ```

## Gnome 桌面

如果您想要使用 Gnome 桌面，在單一 VM (包括 SAP GUI、瀏覽器及 SAP 管理主控台) 內安裝完整的 SAP 示範系統，請使用這裡的提示將它安裝在 Azure SLES 映像上：

   SLES 11：

   ```
   zypper in -t pattern gnome
   ```

   SLES 12：

   ```
   zypper in -t pattern gnome-basic
   ```

## 雲端中 Linux 上針對 Oracle 的 SAP 支援

在虛擬環境中，Oracle 對 Linux 的支援有所限制。雖然這不是 Azure 專屬的主題，不過仍請務必了解。SAP 不支援 SUSE 上的 Oracle 或類似 Azure 之公用雲端中的 Red Hat。若要討論這個主題，請直接連絡 Oracle。

<!---HONumber=AcomDC_0921_2016-->