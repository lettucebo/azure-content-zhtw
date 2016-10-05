<properties
	pageTitle="使用增量快照備份和復原 Azure 虛擬機器 | Microsoft Azure"
	description="使用增量快照，建立備份和復原 Azure 虛擬機器磁碟的自訂解決方案。"
	services="storage"
	documentationCenter="na"
	authors="aungoo-msft"
	manager=""
	editor="tysonn"/>

<tags
	ms.service="storage"
	ms.workload="storage"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/20/2016"
	ms.author="tamram;aungoo"/>


# 使用增量快照集備份 Azure 虛擬機器磁碟

## Overview

Azure 儲存體提供拍攝 Blob 快照的功能。快照會擷取該時間點的 Blob 狀態。在本文中，我們將說明如何使用快照維護虛擬機器磁碟備份的案例。當您選擇不使用 Azure 的備份和復原服務，但是想要為虛擬機器磁碟建立自訂的備份策略時，您可以使用這個方法。

Azure 虛擬機器磁碟在 Azure 儲存體中會儲存為分頁 Blob。本文中我們所討論的是虛擬機器磁碟的備份策略，因此，我們指的是分頁 Blob 內容中的快照。若要深入了解快照，請參閱[建立 Blob 的快照](https://msdn.microsoft.com/library/azure/hh488361.aspx)。

## 什麼是快照？

Blob 快照是在某個時間點擷取的 Blob 唯讀版本。一旦建立快照集後，即可加以讀取、複製或刪除，但不能修改。快照集提供在某個時間點備份 Blob 的方法。在 REST 2015-04-05 版之前，您可以複製完整快照集。使用 REST 2015-07-08 版或更新版本時，您也可以複製增量快照集。

## 完整快照複製

快照可以當做 Blob，複製到另一個儲存體帳戶，以保留基底 Blob 的備份。您也可以透過其基底 Blob 複製快照，就像將 Blob 還原至舊版一樣。將快照集從某個儲存體帳戶複製到另一個儲存體帳戶時，它將會佔用與基底分頁 Blob 相同的空間。因此，將整個快照從某個儲存體帳戶複製到另一個儲存體帳戶時很慢，而且也會耗用目標儲存體帳戶中的大量空間。

>[AZURE.NOTE] 如果您將基底 Blob 複製到另一個目的地，則不會一起複製 Blob 的快照。同樣地，如果您以複本覆寫基底 Blob，與基底 Blob 相關聯的快照則不會受到影響，而且會讓基底 Blob 名稱保持不變。

### 使用快照集備份磁碟

您可以拍攝磁碟或分頁 Blob 的定期快照，並使用[複製 Blob](https://msdn.microsoft.com/library/azure/dd894037.aspx) 作業或 [AzCopy](storage-use-azcopy.md) 之類的工具，將其複製到另一個儲存體帳戶，做為虛擬機器磁碟的備份策略。您可以將快照複製到不同名稱的目的地分頁 Blob。所產生的目的地分頁 Blob 是一個可寫入的分頁 Blob，而不是快照。在本文稍後，我們將說明使用快照建立虛擬機器磁碟備份的步驟。

### 使用快照還原磁碟

到了將磁碟還原至之前在其中一個備份快照中擷取的穩定版本時，您可以將快照複製到基底分頁 Blob 之上。在快照升級至基底分頁 Blob 之後，快照會保留，但您能使用可同時讀取和寫入的複本來覆寫其來源。在本文稍後，我們將說明從快照還原先前磁碟版本的步驟。

### 實作完整快照複製

您可以執行下列命令，以實作完整快照複製：

-   首先，使用[快照 Blob](https://msdn.microsoft.com/library/azure/ee691971.aspx) 作業建立基底 Blob 的快照。
-   接著，使用[複製 Blob](https://msdn.microsoft.com/library/azure/dd894037.aspx)，將快照複製到目標儲存體帳戶。
-   重複此程序來維護您基底 Blob 的備份複本。

## 增量快照複製

[GetPageRanges](https://msdn.microsoft.com/library/azure/ee691973.aspx) API 中的新功能提供更好的方式來備份分頁 Blob 或磁碟的快照集。此 API 會傳回基底 Blob 和快照之間變更的清單。這可減少備份帳戶上使用的儲存空間量。此 API 支援進階儲存體以及標準儲存體的分頁 Blob。您現在可以使用此 API，為 Azure VM 建置更快速且有效率的備份解決方案。這將適用於 REST 2015-07-08 版和更新版本。

「增量快照複製」可讓您將以下兩者之間的差異，從一個儲存體帳戶複製到另一個儲存體帳戶：

-   基底 Blob 及其快照，或
-   基底 Blob 的任何兩個快照

前提是符合下列條件，

- Blob 是在 2016 年 1 月 1 日或之後建立。
- 在兩個快照集之間，Blob 不會覆寫為 [PutPage](https://msdn.microsoft.com/library/azure/ee691975.aspx) 或[複製 Blob](https://msdn.microsoft.com/library/azure/dd894037.aspx)。


**注意**︰此功能適用於進階和標準 Azure 分頁 Blob。

當您有使用快照的自訂備份策略時，將快照從一個儲存體帳戶複製到另一個儲存體帳戶可能會非常慢，而且會耗用大量的儲存空間。您可以將連續快照集之間的差異寫入備份分頁 Blob，而不是將整個快照集複製到備份儲存體帳戶。如此一來，將大量減少複製的時間和儲存備份的空間。

### 實作增量快照複製

您可以執行下列命令，以實作增量快照複製：

-   使用[快照集 Blob](https://msdn.microsoft.com/library/azure/ee691971.aspx) 建立基底 Blob 的快照集。
-   使用[複製 Blob](https://msdn.microsoft.com/library/azure/dd894037.aspx)，將快照複製到目標備份儲存體帳戶。這會是備份分頁 Blob。建立此備份分頁 Blob 的快照集，並將其儲存在備份帳戶中。
-   使用快照集 Blob 建立基底 Blob 的另一個快照集。
-   使用 [GetPageRanges](https://msdn.microsoft.com/library/azure/ee691973.aspx)，取得基底 Blob 的第一個與第二個快照集之間的差異。使用新的參數 **prevsnapshot** 指定您要用來取得差異的快照集的 DateTime 值。當此參數存在時，REST 回應將只包含在目標快照集與先前快照集之間變更的分頁 (包括清除分頁)。
-   使用 [PutPage](https://msdn.microsoft.com/library/azure/ee691975.aspx) 將這些變更套用至備份分頁 Blob。
-   最後，建立備份分頁 Blob 的快照集，並將其儲存在備份儲存體帳戶中。

在下一節中，我們將詳細說明如何使用增量快照複製維護磁碟的備份

## 案例

在本節中，我們將使用快照說明涉及虛擬機器磁碟的自訂備份策略的案例。

請考慮使用連接進階儲存體 P30 磁碟的 DS 系列 Azure VM。稱為 *mypremiumdisk* 的 P30 磁碟會儲存在稱為 *mypremiumaccount* 的進階儲存體帳戶中。稱為 *mybackupstdaccount* 的標準儲存體帳戶將用於儲存 *mypremiumdisk* 的備份。我們想要每 12 小時保留一個 *mypremiumdisk* 的快照集。

若要了解如何建立儲存體帳戶和磁碟，請參閱[關於 Azure 儲存體帳戶](storage-create-storage-account.md)。

若要了解如何備份 Azure VM，請參閱[規劃 Azure VM 備份](../backup/backup-azure-vms-introduction.md)。

## 使用增量快照維護磁碟備份的步驟

以下所述的步驟將建立 *mypremiumdisk* 的快照集，並維護 *mybackupstdaccount* 中的備份。備份將會是稱為 *mybackupstdpageblob* 的標準分頁 Blob。備份分頁 Blob 一律會反映與 *mypremiumdisk* 的最新快照集相同的狀態。

1.  首先，建立進階儲存體磁碟的備份分頁 Blob。若要這樣做，請建立稱為 *mypremiumdisk\_ss1* 的 *mypremiumdisk* 快照集。
2.  將此快照集複製到 mybackupstdaccount，以作為稱為 *mybackupstdpageblob* 的分頁 Blob。
3.  使用[快照集 Blob](https://msdn.microsoft.com/library/azure/ee691971.aspx) 建立稱為 *mybackupstdpageblob\_ss1* 的 *mybackupstdpageblob* 快照集，並將其儲存在 *mybackupstdaccount* 中。
4.  在備份時間範圍期間，建立 *mypremiumdisk* 的快照集 (即 *mypremiumdisk\_ss2*)，並將其儲存在 *mypremiumaccount* 中。
5.  搭配使用 *mypremiumdisk\_ss2* 上的 [GetPageRanges](https://msdn.microsoft.com/library/azure/ee691973.aspx) 與設為 *mypremiumdisk\_ss1* 時間戳記的 **prevsnapshot** 參數，以取得兩個快照集 (*mypremiumdisk\_ss2* 與 *mypremiumdisk\_ss1*) 之間的增量變更。將這些增量變更寫入 *mybackupstdaccount* 中的備份分頁 Blob *mybackupstdpageblob*。如果增量變更中有已刪除的範圍，則必須從備份分頁 Blob 中清除這些範圍。使用 [PutPage](https://msdn.microsoft.com/library/azure/ee691975.aspx) 將增量變更寫入備份分頁 Blob。
6.  建立稱為 *mybackupstdpageblob\_ss2* 的備份分頁 Blob *mybackupstdpageblob* 的快照集。從進階儲存體帳戶刪除先前的快照集 *mypremiumdisk\_ss1*。
7.  在每個備份時間範圍期間重複步驟 4-6。如此一來，您可以維護標準儲存體帳戶中的 *mypremiumdisk* 備份。

![使用增量快照集備份磁碟](./media/storage-incremental-snapshots/storage-incremental-snapshots-1.png)

## 從快照還原磁碟的步驟

以下所述的步驟會將進階磁碟 *mypremiumdisk* 從備份儲存體帳戶 *mybackupstdaccount* 還原至先前的快照集。

1.  識別您要將進階磁碟還原到哪個時間點。讓我們假設這是儲存在備份儲存體帳戶 *mybackupstdaccount* 中的快照集 *mybackupstdpageblob\_ss2*。
2.  在 mybackupstdaccount 中，將快照集 *mybackupstdpageblob\_ss2* 升級為新的備份基底分頁 Blob *mybackupstdpageblobrestored*。
3.  建立稱為 *mybackupstdpageblobrestored\_ss1* 的這個已還原備份分頁 Blob 的快照集。
4.  將已還原分頁 Blob *mybackupstdpageblobrestored* 從 *mybackupstdaccount* 複製到 *mypremiumaccount*，作為新的進階磁碟 *mypremiumdiskrestored*。
5.  建立稱為 *mypremiumdiskrestored\_ss1* 的 *mypremiumdiskrestored* 快照集，以進行未來的增量備份。
6.  將 DS 系列 VM 指向已還原的磁碟 *mypremiumdiskrestored*，並從 VM 卸離舊的 *mypremiumdisk*。
7.  使用 *mybackupstdpageblobrestored* 作為備份分頁 Blob，為已還原的磁碟 *mypremiumdiskrestored* 開始上一節中所述的備份程序。

![從快照還原磁碟](./media/storage-incremental-snapshots/storage-incremental-snapshots-2.png)

## 後續步驟

深入了解建立 Blob 的快照集以及使用下列連結來規劃 VM 備份基礎結構。

- [建立 Blob 的快照集](https://msdn.microsoft.com/library/azure/hh488361.aspx)
- [規劃 VM 備份基礎結構](../backup/backup-azure-vms-introduction.md)

<!---HONumber=AcomDC_0921_2016-->