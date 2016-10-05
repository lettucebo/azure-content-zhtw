<properties
   pageTitle="在 Azure VM 上手動安裝 SAP HANA 的快速入門指南 | Microsoft Azure"
   description="在 Azure VM 上手動安裝 SAP HANA 的快速入門指南"
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

# 在 Azure VM 上手動安裝單一執行個體 SAP HANA 的快速入門指南

## 簡介

本快速入門指南有助於藉由手動安裝 SAP NetWeaver 7.5 和 SAP HANA SP12，在 Azure VM 上設定單一執行個體 SAP HANA 原型/示範系統。本指南假設讀者已熟悉 Azure IaaS 基本概念，像是如何透過 Azure 入口網站或 Powershell/CLI 部署虛擬機器或虛擬網路，包括使用 json 範本的選項。此外還預期讀者已熟悉 SAP HANA、SAP NetWeaver 以及在內部部署進行安裝的方式。

讀者必須留意本文結尾的一般資訊一節中所提到的一般 SAP-Azure 文件。

由於非生產系統的限制，本指南不會涵蓋 HA、備份、DR、高效能或特殊安全性考量等主題。

範例設定是使用兩部虛擬機器來進行，以透過 Azure Resource Manager 模型完成分散式 SAP NetWeaver 安裝，因為 SAP-Linux-Azure 只在 Azure Resource Manager 上提供支援 (傳統模型上不提供支援)。在本文結尾的一般資訊一節中可以找到有關 Azure Resource Manager 的進一步資訊連結。

有兩個用於範例安裝的測試 VM：

* hana-appsrv (類型 DS3) 可裝載 NW 7.5 ASCS 執行個體 + PAS
* hana-dbsrv (類型 GS4) 可裝載 HANA SP12
* 這兩個 VM 屬於一個 Azure 虛擬網路 ( azure-hana-test-vnet )
* 這兩個案例中的 OS 均為 SLES 12 SP1


請留意以下事實：自 2016 年 7 月起，只有 Azure VM 類型 GS5 上的 OLAP (BW) 生產系統完全支援 SAP HANA。基於測試的目，若預期沒有官方 SAP 支援，則可以使用比較小的類型 (例如 GS4)。在 Azure 上一律要用於 SAP HANA 的是適用於 HANA 資料和記錄檔的 Azure 進階儲存體 - 請參閱底下的〈磁碟設定〉一節。關於 Azure 支援的 SAP 產品的進一步詳細資料，請查看本文結尾的一般資訊一節。


本指南說明在 Azure VM 上手動安裝 SAP HANA 的兩種方式：

* 在「資料庫執行個體」步驟中的分散式 NetWeaver 安裝過程中透過 SAP 軟體佈建管理員 (SWPM) 安裝 SAP HANA
* 使用 HANA 生命週期管理員工具 hdblcm 安裝 SAP HANA，然後安裝 NetWeaver

此外，也可能使用 SWPM 並且在一部 VM 中安裝所有元件 (SAP HANA、SAP 應用程式伺服器、ASCS 執行個體、SAP GUI)。本文並未說明此選項，但必須考量的項目相同。

開始安裝之前，應該閱讀以下檢查清單之後有關設定 Azure 測試 VM 的一節，以免在僅使用預設 Azure VM 組態時發生幾個基本錯誤。


## 透過 SAP SWPM 安裝 SAP HANA 的檢查清單

這是針對示範或原型設計目的手動安裝單一執行個體 SAP HANA (透過 SAP SWPM 進行分散式 SAP SWPM 7.5 安裝) 的相關主要項目的簡單檢查清單。整篇文章會更詳細說明個別項目並以螢幕擷取畫面的形式顯示這些項目︰

* 建立將包含兩部測試 VM 的 Azure 虛擬網路
* 透過 Azure Resource Manager 模型部署採用 OS SLES 12 SP1 的兩部 Azure VM
* 將兩個標準儲存體磁碟連接至應用程式伺服器 VM (例如 75GB 和 500GB)
* 將四個磁碟連接至 HANA DB 伺服器 VM- 2 個標準儲存體磁碟 (例如應用程式伺服器 VM) + 2 個進階儲存體磁碟 (例如 2x512GB)
* 視大小和/或輸送量需求而定，附加多個磁碟，並在 VM 內的 OS 層級上使用 lvm 或 mdadm 建立等量磁碟區
* 在連接的磁碟 / 邏輯磁碟區上建立 XFS 檔案系統
* 在 OS 層級裝載新的 XFS 檔案系統。使用一個檔案系統來保留所有 SAP 軟體，而另一個檔案系統用於 sapmnt 目錄且或許是備份。在 SAP HANA DB 伺服器上，於進階儲存體磁碟上掛接 XFS 檔案系統成為 /hana 和 /usr/sap。這全都是為了避免填滿 Linux Azure VM 上不會太大的檔案系統根目錄
* 在 /etc/hosts 中輸入測試 VM 的本機 ip 位址
* 在 /etc/fstab 中輸入 nofail 參數
* 根據HANA-SLES-12 SAP Note 設定核心參數 (請參閱下面核心參數一節中的詳細資料)
* 設定交換空間
* 如果想要 - 在測試 VM 上安裝圖形化桌面。否則使用遠端 sapinst 安裝
* 從 SAP Service Marketplace 下載 SAP 軟體
* 在應用程式伺服器 VM 上安裝 SAP ASCS 執行個體
* 透過 NFS 在測試 VM 之間共用 sapmnt 目錄 (應用程式伺服器 VM 為 NFS 伺服器)
* 在 DB 伺服器 VM 上透過 SWPM 安裝資料庫執行個體 (包括 HANA)
* 在應用程式伺服器 VM 上安裝 SAP
* 啟動 SAP MC 並透過 SAP GUI / HANA Studio 連接



## 透過 hdblcm 安裝 SAP HANA 的檢查清單

這是針對示範或原型設計目的手動安裝單一執行個體 SAP HANA (透過 SAP SWPM 進行分散式 SAP SWPM 7.5 安裝) 的相關主要項目的簡單檢查清單。整篇文章會更詳細說明個別項目並以螢幕擷取畫面的形式顯示這些項目︰

* 建立將包含兩部測試 VM 的 Azure 虛擬網路
* 透過 Azure Resource Manager 模型部署採用 OS SLES 12 SP1 的兩部 Azure VM
* 將兩個標準儲存體磁碟連接至應用程式伺服器 VM (例如 75GB 和 500GB)
* 將四個磁碟連接至 HANA DB 伺服器 VM- 2 個標準儲存體磁碟 (例如應用程式伺服器 VM) + 2 個進階儲存體磁碟 (例如 2x512GB)
* 視大小和/或輸送量需求而定，附加多個磁碟，並在 VM 內的 OS 層級上使用 lvm 或 mdadm 建立等量磁碟區
* 在連接的磁碟 / 邏輯磁碟區上建立 XFS 檔案系統
* 在 OS 層級裝載新的 XFS 檔案系統。使用一個檔案系統來保留所有 SAP 軟體，而另一個檔案系統用於 sapmnt 目錄且或許是備份。在 SAP HANA DB 伺服器上，於進階儲存體磁碟上掛接 XFS 檔案系統成為 /hana 和 /usr/sap。這全都是為了避免填滿 Linux Azure VM 上不會太大的檔案系統根目錄
* 在 /etc/hosts 中輸入測試 VM 的本機 ip 位址
* 在 /etc/fstab 中輸入 nofail 參數
* 根據HANA-SLES-12 SAP Note 設定核心參數 (請參閱下面核心參數一節中的詳細資料)
* 設定交換空間
* 如果想要 - 在測試 VM 上安裝圖形化桌面。否則使用遠端 sapinst 安裝
* 從 SAP Service Marketplace 下載 SAP 軟體
* 在 HANA DB Server VM 上建立具有群組識別碼 1001 的群組 "sapsys"
* 使用 hdblcm 在 DB 伺服器 VM 上安裝 SAP HANA
* 在應用程式伺服器 VM 上安裝 SAP ASCS 執行個體
* 透過 NFS 在測試 VM 之間共用 sapmnt 目錄 (應用程式伺服器 VM 為 NFS 伺服器)
* 在 DB 伺服器 VM 上透過 SWPM 安裝資料庫執行個體 (包括 HANA)
* 在應用程式伺服器 VM 上安裝 SAP
* 啟動 SAP MC 並透過 SAP GUI / HANA Studio 連接




## 準備 Azure VM 以便手動安裝 SAP HANA

這一章有關準備 Azure VM 以便手動安裝 SAP HANA，其中包含五個涵蓋下列主題的章節︰

* 磁碟設定
* 核心參數
* 檔案系統
* /etc/hosts
* /etc/fstab


### 磁碟設定

Azure 上 Linux VM 中的檔案系統根目錄大小受限。因此，必須將額外的磁碟空間附加至 VM 才能執行 SAP。在 SAP 應用程式伺服器 VM 使用於純原型/示範環境中的情況下，可以使用 Azure 標準儲存體磁碟。然而對 SAP HANA DB 資料和記錄檔而言 - Azure 進階儲存體磁碟應使用於非生產環境中。

有關如何將磁碟連接至 Linux VM 的一些詳細資訊，請參閱[這裡](virtual-machines-linux-add-disk.md)

如果是 Azure 磁碟快取 - 使用者必須對將用於儲存 HANA 交易記錄檔的磁碟使用「無」。若為 HANA 資料檔案，則可使用讀取快取。因為 HANA 是記憶體內部資料庫，所以視整體使用模式而定，Azure 磁碟層級上有多少讀取快取會改善效能 (例如啟動 HANA 並將磁碟中的資料讀入記憶體中)。

有關 Azure 進階儲存體的詳細資訊，請參閱[這裡](../storage/storage-premium-storage.md)

在[這裡](https://github.com/Azure/azure-quickstart-templates)可找到用於建立 VM 的範例 Json 範本。"101-vm-simple-linux" 會顯示基本範本的外觀，包括可加入 100GB 資料磁碟的儲存體區段。

[這篇文章](virtual-machines-linux-sap-on-suse-quickstart.md)包含有關如何透過 Powershell 或 CLI 尋找 SUSE 映像的一些資訊，以及透過 UUID 連接磁碟的重要性。


視系統大小和輸送量需求而定，可能需要附加多個磁碟 (而不是一個)，而稍後在這些 OS 層級上建立橫跨這些磁碟的等量磁碟區集合。這些就是使用者為何會跨多個 Azure 磁碟建立等量磁碟區集合的兩個層面︰

* 增加輸送量
* 需要大於 1 TB 的單一檔案系統，因為目前的 Azure 磁碟大小限制為 1TB (從 2016 年 7 月起)


在這裡可以找到有關用來設定串接的兩個主要工具的詳細資訊︰

[有關使用 mdadm 在 Azure VM 上設定 Linux 軟體 RAID 的文章](virtual-machines-linux-configure-raid.md)

[有關在 Linux Azure VM 上設定邏輯磁碟區管理員的文章](virtual-machines-linux-configure-lvm.md)



![](./media/virtual-machines-linux-sap-hana-get-started/image003.jpg)

在測試環境中，有兩個 Azure 標準儲存體磁碟已連接至 SAP 應用程式伺服器 VM。其中一個用來儲存所有可供安裝的 SAP 軟體 (例如 NetWeaver 7.5、SAP GUI、SAP HANA .... )，而另一個有足夠的空間可存放任何必要的資料 (例如備份、測試資料)，以及在屬於相同 SAP 環境的所有 VM 之間共用的 sapmnt 目錄 (例如 SAP 設定檔)。

![](./media/virtual-machines-linux-sap-hana-get-started/image004.jpg)

相對於應用程式伺服器 VM，SAP HANA 伺服器 VM 連接了四個磁碟。如同前面一樣，兩個磁碟用於保留 SAP 軟體 (使用者也可以透過 NFS 共用 SAP 軟體磁碟) 並提供足夠的空間進行備份。其他兩個磁碟是 Azure 進階儲存體磁碟，用以保留 SAP HANA 資料和記錄檔以及 /usr/sap 目錄。


### 核心參數


SAP HANA 需要不屬於標準 Azure 資源庫映像的特定 Linux 核心設定，並且必須以手動方式設定。沒有特別說明這些設定的 SAP Note。


SAP Note SAP HANA DB：SLES 12 / SLE for SAP Applications 12 的建議 OS 設定：[SAP Note 2205917](https://launchpad.support.sap.com/#/notes/2205917)

您可以在[這裡](https://www.suse.com/documentation/sles_for_sap/singlehtml/sles_for_sap_guide/sles_for_sap_guide.html#sec.s4s.configure.page-cache)的 6.1 Kernel: Page-Cache Limit 章節中，找到一個關於在 SLES 上執行 SAP HANA 的相關頁面快取的額外主題。

另外還有關於頁面快取限制的 SAP Note：[SAP Note 1557506](https://service.sap.com/sap/support/notes/1557506)

SLES 12 有一個新工具可取代舊的 sapconf 公用程式。這個新工具為 "tuned-adm"，而且有特別的 SAP HANA 設定檔可用。您可以在下列兩個連結中找到關於這個工具的更多詳細資料。

您可以在[這裡](https://www.suse.com/documentation/sles-for-sap-12/book_s4s/data/sec_s4s_configure_sapconf.html)找到關於 tuned-adm 設定檔 sap-hana 的 SLES 文件

您可以在[這裡](https://www.suse.com/documentation/sles-for-sap-12/pdfdoc/book_s4s/book_s4s.pdf)的 6.2 Tuning Systems for SAP Workloads with tuned-adm 章節中，找到有關 tuned-adm 設定檔 sap-hana 的 SLES 文件


![](./media/virtual-machines-linux-sap-hana-get-started/image005.jpg)

使用者可以在此看到 "tuned-adm" 如何根據必要的 SAP HANA 設定變更 transparent\_hugepage 和 numa\_balancing 值。


若要永久保留 SAP HANA 核心設定，使用者必須在 SLES 12 上使用 grub2。您可以在[這裡](https://www.suse.com/documentation/sled-12/book_sle_admin/data/sec_grub2_file_structure.html)找到關於 grub2 的進一步資訊。


![](./media/virtual-machines-linux-sap-hana-get-started/image006.jpg)

此螢幕擷取畫面顯示如何在組態檔中變更核心設定，然後再透過 grub2-mkconfig「編譯」


![](./media/virtual-machines-linux-sap-hana-get-started/image007.jpg)

另一個選項是透過 Yast 和開機載入器核心參數設定來變更這些設定。


### 檔案系統 

![](./media/virtual-machines-linux-sap-hana-get-started/image008.jpg)

使用者可以在此看到在 SAP 應用程式伺服器 VM 上建立於兩個連接的 Azure 標準儲存體磁碟之上的兩個檔案系統。這兩個檔案系統都屬 XFS 類型並且掛接至 /sapdata 和 /sapsoftware。

您不一定要採用這種方式。有不同的選項可建構磁碟空間。最重要的一點是避免檔案系統根目錄的空間不足。


![](./media/virtual-machines-linux-sap-hana-get-started/image009.jpg)

關於 SAP HANA DB VM，一定要知道在透過 sapinst (swpm) 且僅用簡易「標準」安裝選項的資料庫安裝期間，預設會在 /hana 和 /usr/sap 之下安裝項目。SAP HANA 記錄檔備份的預設設定位於 /usr/sap 之下。如同先前一樣，務必避免檔案系統根目錄的空間不足。因此，使用者應該在透過 swpm 安裝 SAP HANA 之前，先確定 /hana 和 /usr/sap 之下有足夠的可用空間。

SAP 的[這篇文章](http://help.sap.com/saphelp_hanaplatform/helpdata/en/4c/24d332a37b4a3caad3e634f9900a45/frameset.htm)說明 SAP HANA 的標準檔案系統配置


![](./media/virtual-machines-linux-sap-hana-get-started/image010.jpg)

在標準 SLES 12 Azure 資源庫映像上安裝 SAP NetWeaver 時，將會出現沒有交換空間的訊息。若要移除此訊息，您可以如本文件所述，透過 dd、mkswap 和 swapon 手動新增分頁檔案。只要在[這篇文章](https://www.suse.com/documentation/sled-12/book_sle_deployment/data/sec_yast2_i_y2_part_expert.html)中搜尋 "Adding a Swap File Manually"

另一個選項是透過 Linux VM 代理程式設定交換空間。您可以在[這裡](virtual-machines-linux-agent-user-guide.md)找到更多資訊。


### /etc/hosts

![](./media/virtual-machines-linux-sap-hana-get-started/image011.jpg)

開始安裝 SAP 前的另一個重要層面是在 /etc/hosts 檔案中包含 SAP VM 的主機名稱和 IP 位址。使用者應該在一個 Azure 虛擬網路內部署所有的 SAP VM，然後使用內部 IP 位址。

### /etc/fstab

![](./media/virtual-machines-linux-sap-hana-get-started/image000c.jpg)

在測試階段，將 nofail 參數新增到 fstab 竟然是個不錯的主意。如果磁碟發生錯誤，則 VM 仍會啟動，而不會懸置於開機程序但是，使用者必須留意在此情況下，可能無法使用額外的磁碟空間且處理程序可能會填滿檔案系統根目錄。為了避免 /hana 遺漏SAP HANA 完全不會啟動。


## 在 SLES 12 上安裝圖形化 Gnome 桌面

這一章包含兩節並涵蓋下列主題︰

* 在 SLES 12 上安裝 Gnome 桌面和 xrdp
* 在 SLES 12 上使用 Firefox 執行 Java 型 SAP MC

使用者也可以使用 Xterminal、VNC 等替代項目，但是從 2016 年 9 月起，本文僅說明 xrdp。

### 在 SLES 12 上安裝 Gnome 桌面和 xrdp

尤其對具有 Microsoft Windows 背景並想要在 SAP Linux VM 中直接使用圖形化桌面來執行 Firefox、Sapinst、SAP GUI、SAP MC 或 HANA Studio，而且或許會透過 RDP 從 Microsoft Windows 電腦連接到 VM 的使用者而言，有簡單的方式可達到此目的。雖然這可能不適合用於生產資料庫，卻適用於純原型/示範環境。以下是在 Azure SLES 12 VM 上安裝 Gnome 桌面的步驟：

經由下列命令安裝 Gnome 桌面 (例如在 putty 視窗中)︰

   zypper in -t pattern gnome-basic

然後安裝 xrdp 以允許透過 RDP 連接到 VM via RDP：

   zypper in xrdp

編輯 /etc/sysconfig/windowmanager 並將預設 Windows 管理員設定為 Gnome：

   DEFAULT\_WM="gnome"

執行 chkconfig 以確保 xrdp 在重新開機後自動啟動

  chkconfig -level 3 xrdp on

為了避免 RDP 連線發生問題，請嘗試重新啟動 (可能從 putty 視窗)：

  /etc/xrdp/xrdp.sh restart

為了避免上述的 xrdp restart 無法運作，請檢查是否有 .pid 檔案並加以移除：

  檢查 /var/run，尋找 xrdp.pid 並予以移除，然後再試一次重新啟動



### SAP MC


在如前一節所述安裝 Gnome 桌面之後，若要從在 Azure SLES 12 VM 中執行的 Firefox 啟動圖形化 Java 型 SAP MC，使用者會因為遺漏 Java 瀏覽器外掛程式而收到錯誤訊息。

啟動 SAP MC 的 URL 為 <server>:5<instance\_number>13

在[這裡](https://help.sap.com/saphelp_nwce10/helpdata/en/48/6b7c6178dc4f93e10000000a42189d/frameset.htm)可以找到更多詳細資料


![](./media/virtual-machines-linux-sap-hana-get-started/image013.jpg)

在上面的螢幕擷取畫面上，使用者可以看到遺漏 Java 瀏覽器外掛程式時所顯示的錯誤訊息。

![](./media/virtual-machines-linux-sap-hana-get-started/image014.jpg)

解決此問題的其中一個選項就是透過 Yast 安裝遺漏的外掛程式。

![](./media/virtual-machines-linux-sap-hana-get-started/image015.jpg)

重複 SAP MC URL，會在第一次要求啟動此外掛程式時出現一個小型對話方塊。


可能還會出現一個額外的問題，也就是有關遺漏檔案 (javafx.properties) 的錯誤訊息。這很可能與安裝 SAP GUI 7.4 所需的 Java 1.8 相關。

透過 Yast 所看到的 IBM Java 版本不包含此檔案。解決方法是從 Oracle 下載 Java。您可以在[這裡](https://scn.sap.com/thread/3908306)找到討論此特定問題的文章



## 在 NetWeaver 7.5 安裝過程中透過 SWPM 手動安裝 SAP HANA


下列幾個螢幕擷取畫面顯示透過 SWPM (sapinst) 安裝 SAP NetWeaver 7.5 和 SAP HANA SP12 的重要步驟。在 NW 7.5 安裝過程中，SWPM 也能夠將 HANA 資料庫安裝成單一執行個體。


![](./media/virtual-machines-linux-sap-hana-get-started/image012.jpg)

在範例測試環境中，只會安裝一部 ABAP 應用程式伺服器。[分散式系統] 選項用來在一個 Azure VM 中安裝 ASCS 執行個體和主要應用程式伺服器執行個體，以及在另一個 Azure VM 中將 SAP HANA 安裝為資料庫系統。


![](./media/virtual-machines-linux-sap-hana-get-started/image016.jpg)

ASCS 執行個體安裝於應用程式伺服器 VM 且在 SAP MC 中設定為「綠色」之後，必須與 SAP HANA DB 伺服器 VM 共用包含 SAP 設定檔目錄的 sapmnt 目錄。DB 安裝步驟需要存取此資訊。最好的方法是使用可透過 Yast 設定的 NFS。


![](./media/virtual-machines-linux-sap-hana-get-started/image017b.jpg)

在應用程式伺服器 VM 上，應透過 NFS 使用 "rw" 和 "no\_root\_squash" 選項來共用 sapmnt 目錄。預設值為 "ro" 和 "root\_squash"，這可能會導致安裝資料庫執行個體時發生問題。


![](./media/virtual-machines-linux-sap-hana-get-started/image018b.jpg)

在 SAP HANA DB 伺服器 VM 上，必須透過「NFS 用戶端」(例如藉助於 Yast) 來設定應用程式伺服器 VM 中的 sapmnt 共用。


![](./media/virtual-machines-linux-sap-hana-get-started/image019.jpg)

然後使用者必須登入 SAP HANA DB 伺服器 VM 並啟動 SWPM，才能完成分散式 NW 7.5 安裝的下一個步驟 - 「資料庫執行個體」。


![](./media/virtual-machines-linux-sap-hana-get-started/image035b.jpg)

關於 SAP HANA 安裝，在選取「標準」安裝之後，實際上不用輸入太多資料。除了安裝媒體的路徑以外，使用者必須輸入 DB SID、主機名稱、執行個體號碼和 DB 系統管理員密碼。

 
![](./media/virtual-machines-linux-sap-hana-get-started/image036b.jpg)

下一步是輸入 DBACOCKPIT 結構描述的密碼。



![](./media/virtual-machines-linux-sap-hana-get-started/image037b.jpg)

接著會出現 SAPABAP1 結構描述密碼問題。


![](./media/virtual-machines-linux-sap-hana-get-started/image023.jpg)

最後，在 DB 安裝程序的每個階段前面應該只有綠色勾選記號，而跳出的小型訊息方塊應該會顯示「執行 .....資料庫執行個體已完成。」

 
![](./media/virtual-machines-linux-sap-hana-get-started/image024.jpg)

安裝成功後，SAP MC 也應該將 DB 執行個體顯示為「綠色」並顯示完整的 SAP HANA 處理程序清單 (例如 hdbindexserver、hdbcompileserver)


![](./media/virtual-machines-linux-sap-hana-get-started/image025.jpg)

此螢幕擷取畫面會顯示 SWPM 在 HANA 安裝期間於 /hana/shared 之下所建立的部分檔案結構。沒有選項可以指定不同的路徑。因此，在透過 SWPM 安裝 SAP HANA 之前，一定要在 /hana 之下掛接額外的磁碟空間，以免檔案系統根目錄的空間不足。


![](./media/virtual-machines-linux-sap-hana-get-started/image026.jpg)

使用者可以在此看到如之前針對 /usr/sap 目錄所述的相同項目。


![](./media/virtual-machines-linux-sap-hana-get-started/image027b.jpg)

分散式 ABAP 安裝的最後一個步驟是「主要應用程式伺服器執行個體」


![](./media/virtual-machines-linux-sap-hana-get-started/image028b.jpg)

安裝 PAS 和 SAP GUI 之後，使用者可以透過 "dbacockpit" 交易來檢查 SAP HANA 安裝是否看起來一切正常。

 
![](./media/virtual-machines-linux-sap-hana-get-started/image038b.jpg)

在最後一個步驟中，使用者可以在 SAP 應用程式伺服器 VM 中安裝 SAP HANA Studio 並連接至在 DB 伺服器 VM 上執行的 SAP HANA 執行個體。




## 透過 HANA 生命週期管理員工具 hdblcm 手動安裝 SAP HANA


除了透過 SWPM 在分散式安裝過程中安裝 SAP HANA，也可能先使用 hdblcm 獨立安裝 HANA，然後再安裝 SAP NetWeaver 7.5 等。下列螢幕擷取畫面會顯示其運作方式。

以下是有關 HANA hdblcm 工具的三個資訊來源︰

[為您的工作選擇正確的 SAP HANA HDBLCM](https://help.sap.com/saphelp_hanaplatform/helpdata/en/68/5cff570bb745d48c0ab6d50123ca60/content.htm)

[SAP HANA 生命週期管理工具](http://saphanatutorial.com/sap-hana-lifecycle-management-tools/)

[SAP HANA 伺服器安裝與更新指南](http://help.sap.com/hana/SAP_HANA_Server_Installation_Guide_en.pdf)



![](./media/virtual-machines-linux-sap-hana-get-started/image030.jpg)

為了避免 <HANA SID>adm 使用者 (由 hdblcm 工具建立) 的預設群組識別碼設定稍後發生問題，使用者應該在使用 hdblcm 安裝 SAP HANA 之前，定義名為 "sapsys" 且群組識別碼為 1001 的新群組。




![](./media/virtual-machines-linux-sap-hana-get-started/image031.jpg)

第一次啟動 hdblcm 時，會出現簡單的 [開始] 功能表，使用者必須在其中選取第 1 項「安裝新系統」



![](./media/virtual-machines-linux-sap-hana-get-started/image032.jpg)

在此螢幕擷取畫面上，使用者可以看到之前輸入的所有重要選項。重要事項 - 針對 HANA 記錄檔和資料磁碟區命名的目錄以及安裝路徑 (在此範例中為 /hana/shared) 和 /usr/sap 不能是檔案系統根目錄的一部分，但是屬於連接到 VM 的 Azure 資料磁碟 (如 Azure VM 設定一節所述)。這會避免檔案系統根目錄可能空間不足的風險。使用者也可以看見 HANA 系統管理使用者的使用者識別碼為 1005，並屬於在安裝前定義的 sapsys 群組 (識別碼為 1001)。



![](./media/virtual-machines-linux-sap-hana-get-started/image033.jpg)

使用者可以在 /etc/passwd 中檢查 HANA <HANA SID>adm (在此範例中為 azdadm) 使用者詳細資料



![](./media/virtual-machines-linux-sap-hana-get-started/image034.jpg)

使用 hdblcm 安裝 SAP HANA 之後，即可顯示在 SAP HANA Studio 中。尚未提供 SAPABAP1 結構描述 (其中包括所有的 SAP NetWeaver 資料表)。



![](./media/virtual-machines-linux-sap-hana-get-started/image035b.jpg)

安裝 SAP HANA 之後，即可在其上安裝 SAP NetWeaver。在此範例中，這項作業是透過 SWPM 使用如上面對應章節中所述的「分散式安裝」來完成。透過 SWPM 安裝資料庫執行個體時，使用者只需輸入與 hdblcm 先前所用的相同資料 (例如主機名稱、HANA SID、執行個體號碼)。SWPM 會接著使用現有的 HANA 安裝，並加入其他結構描述。



![](./media/virtual-machines-linux-sap-hana-get-started/image036b.jpg)

這是 SWPM 安裝步驟的圖片，使用者必須輸入有關 DBACOCKPIT 結構描述的資料。


![](./media/virtual-machines-linux-sap-hana-get-started/image037b.jpg)

然後 SWPM 預期會輸入 SAPABAP1 結構描述的相關資料。


![](./media/virtual-machines-linux-sap-hana-get-started/image038b.jpg)

完成 SWPM 資料庫執行個體安裝後，即可在 HANA Studio 中看見 SAPABAP1 結構描述。



![](./media/virtual-machines-linux-sap-hana-get-started/image039b.jpg)

而最後在 SAP 應用程式伺服器和 SAP GUI 安裝之後，使用者應該能夠利用交易 "dbacockpit" 驗證 HANA DB 執行個體。




## 與 SAP Azure 認證、在 Azure 上執行 SAP HANA 和 SAP 軟體下載相關的一般資訊

* 有關在 Windows OS 處於傳統模式的 Azure 上執行 SAP 的一般 SAP Azure 文件︰[在 Azure 中的 Windows 虛擬機器上使用 SAP](virtual-machines-windows-classic-sap-get-started.md)

* 現有 SAP 範本由客戶的使用方式的相關資訊︰[適用於 SAP 的 Azure 快速入門範本](https://blogs.msdn.microsoft.com/saponsqlserver/2016/05/16/azure-quickstart-templates-for-sap/)

* 有關在 Linux OS 處於 Azure Resource Manager 模式的 Azure 上執行 SAP 的一般 SAP Azure 文件︰[ 虛擬機器 (VM) 上使用 SAP](virtual-machines-linux-sap-get-started.md)

* 已認證的 SAP HANA 硬體目錄，其中列出生產環境支援的 Azure VM 類型︰[已認證的 SAP HANA® 硬體目錄](https://global.sap.com/community/ebook/2014-09-02-hana-hardware/enEN/iaas.html)

* 如需關於虛擬機器大小 (尤其是對 Linux 工作負載) 的資訊：[Azure 中的虛擬機器大小](virtual-machines-linux-sizes.md)

* SAP Note，其中列出 Azure 上所有支援的 SAP 產品和針對 SAP 支援的 Azure VM 類型︰[SAP Note 1928533](https://launchpad.support.sap.com/#/notes/1928533/E)

* 有關對 Azure 上的 Linux VM 進行 SAP「增強型監視」的 SAP Note：[SAP Note 2191498](https://launchpad.support.sap.com/#/notes/2191498/E)

* 在 Azure「大型執行個體」上提供的 SAP HANA。請務必了解，這與在 Azure VM 上執行 SAP HANA 無關，而是關於在混合式環境 (在 Azure VM 中執行 SAP 應用程式伺服器，但在裸機伺服器上執行 SAP HANA) 中執行 SAP HANA︰[SAP Note 2316233](https://launchpad.support.sap.com/#/notes/2316233/E)

* 具有 Linux 上 SAPOSCOL 相關資訊的 SAP Note：[SAP Note 1102124](https://launchpad.support.sap.com/#/notes/1102124/E)

* Microsoft Azure 上的 SAP 主要監視度量：[SAP Note 2178632](https://launchpad.support.sap.com/#/notes/2178632/E)

* 有關 Azure Resource Manager 的資訊：[Azure Resource Manager 概觀](../resource-group-overview.md)

* 有關透過範本部署 Linux VM 的資訊：[使用 Azure Resource Manager 範本和 Azure CLI 部署和管理虛擬機器](virtual-machines-linux-cli-deploy-templates.md)

* Azure Resource Manager 與傳統之間的部署模型比較：[Azure Resource Manager 與傳統部署比較：了解資源的部署模型和狀態](../resource-manager-deployment-model.md)

* 從 SAP Service Marketplace 下載適用於 Linux/HANA 的 NetWeaver 7.5：![](./media/virtual-machines-linux-sap-hana-get-started/image001.jpg)

* 從 SAP Service Marketplace 下載 HANA SP12 平台版本︰![](./media/virtual-machines-linux-sap-hana-get-started/image002.jpg)

<!---HONumber=AcomDC_0921_2016-->