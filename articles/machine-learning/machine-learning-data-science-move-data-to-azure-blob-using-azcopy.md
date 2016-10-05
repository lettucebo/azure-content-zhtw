<properties
	pageTitle="使用 AzCopy 從 Azure Blob 儲存體來回移動資料 | Microsoft Azure"
	description="使用 AzCopy 從 Azure Blob 儲存體來回移動資料"
	services="machine-learning,storage"
	documentationCenter=""
	authors="bradsev"
	manager="jhubbard"
	editor="cgronlun" />

<tags
	ms.service="machine-learning"
	ms.workload="data-services"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/14/2016"
	ms.author="bradsev" />

# 使用 AzCopy 從 Azure Blob 儲存體來回移動資料

AzCopy 是個命令列公用程式，專為上傳、下載，以及將資料複製到和複製出 Microsoft Azure Blob、檔案和表格儲存體所設計。

如需安裝 AzCopy 的指示，和以 Azure 平台加以執行的其他資訊，請參閱[開始使用 AzCopy 命令列公用程式](../storage/storage-use-azcopy.md)。

以下是有關從 Azure Blob 儲存體來回移動資料所使用之技術的指引連結：

[AZURE.INCLUDE [blob-storage-tool-selector](../../includes/machine-learning-blob-storage-tool-selector.md)]


> [AZURE.NOTE] 如果您是使用以 [Azure 中的資料科學虛擬機器](machine-learning-data-science-virtual-machines.md)提供的指令碼所設定的 VM，則 AzCopy 已經安裝在 VM 上。

> [AZURE.NOTE] 如需 Azure Blob 儲存體的完整介紹，請參閱 [Azure Blob 基本概念](../storage/storage-dotnet-how-to-use-blobs.md)和 [Azure Blob 服務](https://msdn.microsoft.com/library/azure/dd179376.aspx)。


## 必要條件

本文件假設您擁有 Azure 訂用帳戶、儲存體帳戶和該帳戶的對應儲存體金鑰。上傳/下載資料之前，您必須知道 Azure 儲存體帳戶名稱和帳戶金鑰。

- 若要設定 Azure 訂用帳戶，請參閱[免費試用一個月](https://azure.microsoft.com/pricing/free-trial/)。

- 如需建立儲存體帳戶以及取得帳戶和金鑰資訊的指示，請參閱[關於 Azure 儲存體帳戶](../storage/storage-create-storage-account.md)。


## 執行 AzCopy 命令

若要執行 AzCopy 命令，請開啟命令視窗並瀏覽至電腦上的 AzCopy 安裝目錄，也就是 AzCopy.exe 可執行檔的所在位置。

AzCopy 命令的基本語法是：

	AzCopy /Source:<source> /Dest:<destination> [Options]

>[AZURE.NOTE] 您可以將 AzCopy 安裝位置新增到系統路徑，然後從任何目錄執行命令。根據預設，AzCopy 會安裝到 *%ProgramFiles(x86)%\\Microsoft SDKs\\Azure\\AzCopy* 或 *%ProgramFiles%\\Microsoft SDKs\\Azure\\AzCopy*。

## 將檔案上傳至 Azure Blob

若要上傳檔案，請使用下列命令：

	# Upload from local file system
	AzCopy /Source:<your_local_directory> /Dest: https://<your_account_name>.blob.core.windows.net/<your_container_name> /DestKey:<your_account_key> /S


## 從 Azure Blob 下載檔案

若要從 Azure Blob 下載檔案，請使用下列命令：

	# Downloading blobs to local file system
	AzCopy /Source:https://<your_account_name>.blob.core.windows.net/<your_container_name>/<your_sub_directory_at_blob>  /Dest:<your_local_directory> /SourceKey:<your_account_key> /Pattern:<file_pattern> /S


## 在 Azure 容器之間傳輸 Blob

若要在 Azure 容器之間傳輸 Blob，請使用下列命令：

	# Transferring blobs between Azure containers
	AzCopy /Source:https://<your_account_name1>.blob.core.windows.net/<your_container_name1>/<your_sub_directory_at_blob1> /Dest:https://<your_account_name2>.blob.core.windows.net/<your_container_name2>/<your_sub_directory_at_blob2> /SourceKey:<your_account_key1> /DestKey:<your_account_key2> /Pattern:<file_pattern> /S

	<your_account_name>: your storage account name
	<your_account_key>: your storage account key
	<your_container_name>: your container name
	<your_sub_directory_at_blob>: the sub directory in the container
	<your_local_directory>: directory of local file system where files to be uploaded from or the directory of local file system files to be downloaded to
	<file_pattern>: pattern of file names to be transferred. The standard wildcards are supported


## 使用 AzCopy 的秘訣

> [AZURE.TIP]   
> 1. **上傳**檔案時，/S 將以遞迴方式上傳檔案。如果沒有這個參數，則不會上傳子目錄中的檔案。
> 2. **下載**檔案時，/S 將以遞迴方式搜尋容器，直到下載了指定目錄及其子目錄中的所有檔案，或指定目錄及其子目錄中所有符合指定模式的所有檔案為止。
> 3.  您無法使用 /Source 參數來指定要下載的**特定 Blob 檔案**。若要下載特定檔案，請使用 /Pattern 參數指定要下載的 Blob 檔案名稱。**/S** 參數可用來讓 AzCopy 以遞迴方式尋找檔案名稱模式。若未提供模式參數，AzCopy 會下載該目錄中的所有檔案。

<!---HONumber=AcomDC_0921_2016-->