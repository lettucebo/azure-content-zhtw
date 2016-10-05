<properties
   pageTitle="復原服務保存庫常見問題集 | Microsoft Azure"
   description="此版本的常見問題集支援公開預覽版本的 Azure 備份服務。關於備份代理程式、備份和保留、復原、安全性，以及 Azure 備份解決方案其他常見問題的常見問題集答案。"
   services="backup"
   documentationCenter=""
   authors="markgalioto"
   manager="jwhit"
   editor=""
   keywords="備份解決方案；備份服務"/>

<tags
   ms.service="backup"
   ms.workload="storage-backup-recovery"
	 ms.tgt_pltfrm="na"
	 ms.devlang="na"
	 ms.topic="get-started-article"
	 ms.date="08/21/2016"
	 ms.author="trinadhk; markgal; jimpark;"/>

# 復原服務保存庫 - 常見問題集

> [AZURE.SELECTOR]
- [傳統模式的備份常見問題集](backup-azure-backup-faq.md)
- [Resource Manager 模式的備份常見問題集](backup-azure-backup-ibiza-faq.md)

本文提供復原服務保存庫特有資訊，並補充說明 [Azure 備份常見問題集](backup-azure-backup-faq.md)。Azure 備份常見問題集提供一組關於 Azure 備份服務的完整問答。

您可以在本文件或相關文件的 Disqus 一節中詢問有關 Azure 備份的問題。您也可以在[論壇](https://social.msdn.microsoft.com/forums/azure/home?forum=windowsazureonlinebackup)中張貼有關 Azure 備份服務的問題。

## 復原服務保存庫是以 Resource Manager 為基礎。備份保存庫 (傳統模式) 是否仍受支援？<br/>
是，仍然支援備份保存庫。在[傳統入口網站](https://manage.windowsazure.com)中建立備份保存庫。在 [Azure 入口網站](https://portal.azure.com)中建立復原服務保存庫。不過，強烈建議您建立復原服務保存庫，因為所有未來的增強功能僅可用於復原服務保存庫中。

## 是否可以將備份保存庫移轉至復原服務保存庫？<br/>
很遺憾，不行，目前無法將備份保存庫的內容移轉至復原服務保存庫。我們正著手新增這項功能，但公開預覽中並未提供。

## 復原服務保存庫是否支援傳統 VM 或以 Resource Manager 為基礎的 VM？<br/>
復原服務保存庫支援兩種模式。您可以將傳統入口網站中建立的 VM (傳統模式 VM) 或 Azure 入口網站中建立的 VM (以 Resource Manager 為基礎) 備份至復原服務保存庫。

## 我已在備份保存庫中備份傳統 VM。我現在想要將 VM 從傳統模式移轉至 Resource Manager 模式。如何才能在復原服務保存庫中備份它們？
當您將 VM 從傳統移轉至 Resource Manager 模式，備份保存庫中傳統 VM 的備份將不會自動移轉至復原服務保存庫。請遵循下列步驟進行 VM 備份的移轉︰

1. 在備份保存庫中，請移至 [受保護的項目] 索引標籤並選取 VM。按一下[停止保護](backup-azure-manage-vms-classic.md#stop-protecting-virtual-machines)。讓 [刪除相關聯的備份資料] 選項保持 [未核取] 狀態。
2. 將虛擬機器從傳統模式移轉至 Resource Manager 模式。確定虛擬機器對應的儲存體和網路也會移轉至 Resource Manager 模式。
3. 建立復原服務保存庫，並利用保存庫儀表板上的 [備份] 動作，在移轉的虛擬機器上設定備份。深入了解如何[在復原服務保存庫中啟用備份](backup-azure-vms-first-look-arm.md)

<!---HONumber=AcomDC_0928_2016-->