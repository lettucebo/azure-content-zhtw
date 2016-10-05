<properties 
   pageTitle="StorSimple Manager 服務管理 | Microsoft Azure"
   description="了解如何使用 Azure 傳統入口網站中的 StorSimple Manager 服務管理 StorSimple 裝置。"
   services="storsimple"
   documentationCenter=""
   authors="alkohli"
   manager="carmonm"
   editor="" />
<tags 
   ms.service="storsimple"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="09/21/2016"
   ms.author="alkohli" />

# 使用 StorSimple Manager 服務管理 StorSimple 裝置

## 概觀

本文描述 StorSimple Manager 服務介面，包括如何與其連接、各種可用選項，以及可以透過此 UI 執行的特定工作流程連結。本指引適用於以下兩者；StorSimple 實體和虛擬裝置。

閱讀本文知後，您將了解：

- 和 StorSimple Manager 服務的連接
- 瀏覽 StorSimple Manager UI
- 透過 StorSimple Manager 服務管理 StorSimple 裝置


## 和 StorSimple Manager 服務的連接

StorSimple Manager 服務可在 Microsoft Azure 中執行，並且連接至多個 StorSimple 裝置。您可以使用在瀏覽器中執行的中央 Microsoft Azure 傳統入口網站來管理這些裝置。若要連接道 StorSimple Manager 服務，請執行下列動作。

#### 連接至此服務

1. 瀏覽至 [https://manage.windowsazure.com/](https://manage.windowsazure.com/)。

1. 使用您的 Microsoft 帳戶認證，登入 Microsoft Azure 傳統入口網站 (位於窗格右上角)。

1. 向下捲動左方瀏覽窗格以存取 StorSimple Manager 服務。


## 瀏覽 StorSimple Manager 服務 UI

StorSimple Manager 服務 UI 的瀏覽階層顯示於下表。

- **StorSimple Manager** 登陸頁面將帶您到適用於服務內所有裝置的 UI 服務層級頁面。

- [**裝置**] 頁面會將您導向適用於特定裝置的裝置層級 UI 頁面。

- [**磁碟區容器**] 頁面會將您導向顯示與裝置相關聯之所有磁碟區的磁碟區頁面。


#### StorSimple Manager 服務瀏覽階層

|登陸頁面|服務層級頁面|裝置層級頁面|裝置層級頁面|
|---|---|---|---|
|StorSimple Manager 服務|服務儀表板|裝置儀表板||
||裝置 →|監視|
||備份類別目錄|磁碟區容器 →|磁碟區|
||設定 (服務)|備份原則||
||作業|設定 (裝置)|
||Alerts|維護|

![提供的影片](./media/storsimple-manager-service-administration/Video_icon.png)**提供的影片**

若要觀看逐步引導您使用 StorSimple Manager 服務使用者介面的影片，請按一下[這裡](https://azure.microsoft.com/documentation/videos/storsimple-manager-service-overview/)。

## 使用 StorSimple Manager 服務管理 StorSimple 裝置

下表顯示所有一般管理工作和複雜工作流程 (可在 StorSimple Manager 服務 UI 內執行) 的摘要。這些工作會根據在其中啟動它們的 UI 頁面加以組織。

如需每個工作流程的詳細資訊，請按一下資料表中的適當程序。

#### StorSimple Manager 工作流程

|如果您想要執行此動作...|請移至此 UI 頁面...|使用此程序。|
|---|---|---|
|建立服務</br>刪除服務</br>取得服務註冊金鑰</br>重新產生服務註冊金鑰|StorSimple Manager 服務|[部署 StorSimple Manager 服務](storsimple-manage-service.md)
|變更服務資料加密金鑰</br>檢視作業記錄檔|StorSimple Manager 服務 → 儀表板|[使用 StorSimple Manager 服務儀表板](storsimple-service-dashboard.md)|
|停用裝置</br>刪除裝置|StorSimple Manager 服務 → 裝置|[停用或刪除裝置](storsimple-deactivate-and-delete-device.md)|
|了解災害復原和裝置容錯移轉</br>容錯移轉至實體裝置</br>容錯移轉至虛擬裝置</br>業務持續性災害復原 (BCDR)|StorSimple Manager 服務 → 裝置|[StorSimple 裝置的容錯移轉與災害復原](storsimple-device-failover-disaster-recovery.md)|
|列出磁碟區備份</br>選取備份組</br>刪除備份組|StorSimple Manager 服務 → 備份目錄|[管理備份](storsimple-manage-backup-catalog.md)|
|複製磁碟區|StorSimple Manager 服務 → 備份目錄|[複製磁碟區](storsimple-clone-volume.md)|
|還原備份組|StorSimple Manager 服務 → 備份目錄|[還原備份組](storsimple-restore-from-backup-set.md)|
|有關儲存體帳戶</br>加入儲存體帳戶</br>編輯儲存體帳戶</br>刪除儲存體帳戶</br>儲存體帳戶的金鑰輪替|StorSimple Manager 服務 → 設定|[管理儲存體帳戶](storsimple-manage-storage-accounts.md)|
|關於頻寬範本</br>新增頻寬範本</br>編輯頻寬範本</br>刪除頻寬範本</br>使用預設頻寬範本</br>建立在指定時間啟動的全天頻寬範本|StorSimple Manager 服務 → 設定|[管理頻寬範本](storsimple-manage-bandwidth-templates.md)|
|關於存取控制記錄</br>建立存取控制記錄</br>編輯存取控制記錄</br>刪除存取控制記錄|StorSimple Manager 服務 → 設定|[管理存取控制記錄](storsimple-manage-acrs.md)|
|檢視工作詳細資料</br>取消工作|StorSimple Manager 服務 → 工作|[管理工作](storsimple-manage-jobs.md)
|接收警示通知</br>管理警示</br>檢閱警示|StorSimple Manager 服務 → 警示|[檢視和管理 StorSimple 警示](storsimple-manage-alerts.md)
|檢視連接的啟動器</br>尋找裝置序號</br>尋找目標 IQN|StorSimple Manager 服務 → 裝置 → 儀表板|[使用 StorSimple Manager 裝置儀表板](storsimple-device-dashboard.md)|
|建立監視圖表|StorSimple Manager 服務 → 裝置 → 監視器|[監視您的 StorSimple 裝置](storsimple-monitor-device.md)|
|新增磁碟區容器</br>修改磁碟區容器</br>刪除磁碟區容器|StorSimple Manager 服務 → 裝置 → 磁碟區容器|[管理磁碟區容器](storsimple-manage-volume-containers.md)|
|新增磁碟區</br>修改磁碟區</br>讓磁碟區離線</br>刪除磁碟區</br>監視磁碟區|StorSimple Manager 服務 → 裝置 → 磁碟區容器 → 磁碟區|[管理磁碟區](storsimple-manage-volumes.md)|
|修改裝置設定</br>修改時間設定</br>修改 DNS.md 設定</br>設定網路介面|StorSimple Manager 服務 → 裝置 → 設定|[修改 StorSimple 裝置的裝置組態](storsimple-modify-device-config.md)|
|檢視 Web Proxy 設定|StorSimple Manager 服務 → 裝置 → 設定|[設定裝置的 Web Proxy](storsimple-configure-web-proxy.md)|
|修改裝置系統管理員密碼</br>修改 StorSimple Snapshot Manager 密碼|StorSimple Manager 服務 → 裝置 → 設定|[變更 StorSimple 密碼](storsimple-change-passwords.md)|
|設定遠端管理|StorSimple Manager 服務 → 裝置 → 設定|[遠端連接至 StorSimple 裝置](storsimple-remote-connect.md)|
|設定警示設定|StorSimple Manager 服務 → 裝置 → 設定|[檢視和管理 StorSimple 警示](storsimple-manage-alerts.md)|
|為 StorSimple 裝置設定 CHAP|StorSimple Manager 服務 → 裝置 → 設定|[為 StorSimple 裝置設定 CHAP](storsimple-configure-chap.md)|
|新增備份原則</br>新增或修改排程</br>刪除備份原則</br>手動備份</br>使用多個磁碟區和排程建立自訂備份原則|StorSimple Manager 服務 → 裝置 → 備份原則|[管理備份原則](storsimple-manage-backup-policies.md)|
|停止裝置控制器</br>重新啟動裝置控制器</br>關閉裝置控制器</br>將您的裝置重設為出廠預設值</br>(上述項目僅適用於內部部署裝置)|StorSimple Manager 服務 → 裝置 → 維護|[管理 StorSimple 裝置控制器](storsimple-manage-device-controller.md)|
|了解 StorSimple 硬體元件</br>監視硬體狀態</br>(上述項目僅適用於內部部署裝置)|StorSimple Manager 服務 → 裝置 → 維護|[監視硬體元件](storsimple-monitor-hardware-status.md)|
|建立支援封裝|StorSimple Manager 服務 → 裝置 → 維護|[建立及管理支援封裝](storsimple-create-manage-support-package.md)|
|安裝軟體更新|StorSimple Manager 服務 → 裝置 → 維護|[更新您的裝置](storsimple-update-device.md)|


##後續步驟
如果您遇到與 StorSimple 裝置的日常作業或其中任何硬體元件的任何問題，請參閱：

- [可運作裝置的疑難排解](storsimple-troubleshoot-operational-device.md)
- [使用 StorSimple 監視 LED 指示燈](storsimple-monitoring-indicators.md)

如果您不能解決問題，且您必須建立服務要求，請參閱：[連絡 Microsoft 支援服務](storsimple-contact-microsoft-support.md)。

<!---HONumber=AcomDC_0921_2016-->