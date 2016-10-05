<properties
   pageTitle="在 StorSimple 裝置上安裝 Update 2 | Microsoft Azure"
   description="說明如何在您的 StorSimple 8000 系列裝置上安裝 StorSimple 8000 系列 Update 2。"
   services="storsimple"
   documentationCenter="NA"
   authors="alkohli"
   manager="carmonm"
   editor="" />
<tags
   ms.service="storsimple"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="TBD"
   ms.date="09/21/2016"
   ms.author="alkohli" />

# 在 StorSimple 裝置上安裝 Update 2

## Overview

本教學課程說明如何透過 Azure 傳統入口網站，在執行舊版軟體的 StorSimple 裝置上安裝 Update 2。本教學課程也涵蓋了當閘道器設定於 StorSimple 裝置之 DATA 0 以外的網路介面上，而您嘗試從 Update 1 以前的軟體版本更新時所需的步驟。

Update 2 包括裝置軟體更新、LSI 驅動程式更新和磁碟韌體更新。此裝置軟體和 LSI 更新為非干擾性更新，且可透過 Azure 傳統入口網站套用。磁碟韌體更新為干擾性更新，且只能透過裝置的 Windows PowerShell 介面套用。

> [AZURE.IMPORTANT]

> -  您可能不會立即看到 Update 2，因為我們會分階段推出更新。請在數天內再次掃描更新，因為很快就會提供 Update。
> - 安裝前會執行一組手動和自動預先檢查，以根據硬體狀態和網路連線來判斷裝置健全狀況。這些預先檢查只會在您從 Azure 傳統入口網站套用更新時執行。
> - 建議您透過 Azure 傳統入口網站安裝軟體和驅動程式更新。如果入口網站中的更新前閘道器檢查失敗，請移至裝置的 Windows PowerShell 介面安裝更新 (勿透過其他方式)。更新可能需要 4-7 小時才能安裝 (包括 Windows Updates)。維護模式更新必須透過裝置的 Windows PowerShell 介面安裝。由於維護模式更新是干擾性更新，它們將會導致裝置的停機時間。
> - 如果執行選擇性的 StorSimple Snapshot Manager，更新裝置之前，請先將您的 Snapshot Manager 版本升級至 Update 2。

[AZURE.INCLUDE [storsimple-preparing-for-update](../../includes/storsimple-preparing-for-updates.md)]

## 透過 Azure 傳統入口網站安裝 Update 2

請執行下列步驟，以將您的裝置更新至 [Update 2](storsimple-update2-release-notes.md)。


> [AZURE.NOTE]
Update 2 讓 Microsoft 從裝置提取其他診斷資訊。因此，當我們的作業小組識別有問題的裝置時，我們更有能力從裝置收集資訊並診斷問題。接受 Update 2，表示您允許我們提供此主動支援。

[AZURE.INCLUDE [storsimple-install-update2-via-portal](../../includes/storsimple-install-update2-via-portal.md)]

12. 驗證您的裝置正在執行 **StorSimple 8000 系列 Update 2 (6.3.9600.17673)**。[**上次更新日期**] 應該也已修改。您也會看到有可用的維護模式更新 (此訊息可能會在您安裝更新之後繼續顯示長達 24 小時)。

    維護模式更新為干擾性更新，會導致裝置產生停機時間，且只能透過您裝置的 Windows PowerShell 介面加以套用。在某些情況下，當您執行 Update 1.2 時，磁碟韌體可能已經是最新狀態，這種情況下，您不需要安裝任何維護模式更新。

13. 使用[下載 Hotfix](#to-download-hotfixes) 中列出的步驟，下載維護模式更新，以搜尋和下載 KB3121899，它會安裝磁碟韌體更新 (現在其他更新應該已安裝)。

13. 請遵循[安裝及驗證維護模式 Hotfix](#to-install-and-verify-maintenance-mode-hotfixes) 中列出的步驟安裝維護模式更新。


## 以 Hotfix 的方式安裝 Update 2

請在嘗試透過 Azure 傳統入口網站安裝更新，而無法通過閘道器檢查時執行此程序。當您指派閘道器給非 DATA 0 網路介面且您的裝置正在執行早於 Update 1 的軟體版本時，檢查才會失敗。

可以使用 Hotfix 方法升級的軟體版本為 Update 0.1、Update 0.2、Update 0.3、Update 1、Update 1.1 及 Update 1.2。Hotfix 方法涉及下列三個步驟：

- 從 Microsoft Update Catalog 下載 Hotfix。
- 安裝及驗證定期模式 Hotfix。
- 安裝及驗證維護模式 Hotfix。

若要將 Update 2 安裝為 Hotfix，您必須下載並安裝下列 Hotfix：

| 順序 | KB | 說明 | 更新類型 |
|--------|-----------|-------------------------|------------- |
| 1 | KB3121901 | 軟體更新 | 定期 |
| 2 | KB3121900 | LSI 驅動程式 | 定期 |
| 3 | KB3080728 | Storport 修正程式 </br> Windows Server 2012 R2 | 定期 |
| 4 | KB3090322 | Spaceport 修正程式 </br> Windows Server 2012 R2 | 定期 |
| 5 | KB3121899 | 磁碟韌體 | 維護 |


> [AZURE.IMPORTANT]
>
> - 如果您的裝置正在執行 Release (GA) 版本，請連絡 [Microsoft 支援服務](storsimple-contact-microsoft-support.md)以協助您進行更新。
> - 此程序僅需要執行一次，即可套用 Update 2。您可以使用 Azure 傳統入口網站套用後續的更新。
> - 每個 Hotfix 安裝可能需要約 20 分鐘才能完成。總計的安裝時間接近 2 小時。
> - 使用此程序套用更新之前，請確定兩個裝置控制器都在線上，而且所有硬體元件的狀況良好。

執行下列步驟，以 Hotfix 方式套用此更新。

[AZURE.INCLUDE [storsimple-install-update2-hotfix](../../includes/storsimple-install-update2-hotfix.md)]

[AZURE.INCLUDE [storsimple-install-troubleshooting](../../includes/storsimple-install-troubleshooting.md)]



## 後續步驟

深入了解 [Update 2 版](storsimple-update2-release-notes.md)。

<!---HONumber=AcomDC_0921_2016-->