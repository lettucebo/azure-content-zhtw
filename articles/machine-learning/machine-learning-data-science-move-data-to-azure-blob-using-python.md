<properties
	pageTitle="使用 Python 從 Azure Blob 儲存體來回移動資料 | Microsoft Azure"
	description="使用 Python 從 Azure Blob 儲存體來回移動資料"
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

# 使用 Python 從 Azure Blob 儲存體來回移動資料

本主題說明如何使用 Python API 列出、上傳及下載 Blob。使用 Azure SDK 中提供的 Python API，您可以

- 建立容器
- 將 Blob 上傳至容器
- 下載 Blob
- 列出容器中的 Blob
- 刪除 Blob

如需使用 Python API 的詳細資訊，請參閱[如何從 Python 使用 Blob 儲存體服務](../storage/storage-python-how-to-use-blob-storage.md)。

以下是有關從 Azure Blob 儲存體來回移動資料所使用之技術的指引連結：

[AZURE.INCLUDE [blob-storage-tool-selector](../../includes/machine-learning-blob-storage-tool-selector.md)]


> [AZURE.NOTE] 如果您是使用以 [Azure 中的資料科學虛擬機器](machine-learning-data-science-virtual-machines.md)提供的指令碼所設定的 VM，則 AzCopy 已經安裝在 VM 上。

> [AZURE.NOTE] 如需 Azure Blob 儲存體的完整介紹，請參閱 [Azure Blob 基本概念](../storage/storage-dotnet-how-to-use-blobs.md)和 [Azure Blob 服務](https://msdn.microsoft.com/library/azure/dd179376.aspx)。


## 必要條件

本文件假設您擁有 Azure 訂用帳戶、儲存體帳戶和該帳戶的對應儲存體金鑰。上傳/下載資料之前，您必須知道 Azure 儲存體帳戶名稱和帳戶金鑰。

- 若要設定 Azure 訂用帳戶，請參閱[免費試用一個月](https://azure.microsoft.com/pricing/free-trial/)。

- 如需建立儲存體帳戶以及取得帳戶和金鑰資訊的指示，請參閱[關於 Azure 儲存體帳戶](../storage/storage-create-storage-account.md)。


## 將資料上傳至 Blob

將下列程式碼片段新增至您希望以程式設計方式存取 Azure 儲存體的任何 Python 程式碼頂端附近：

	from azure.storage.blob import BlobService

**BlobService** 物件讓您能使用容器及 blob。下列程式碼會使用儲存體帳戶名稱和帳戶金鑰來建立 BlobService 物件。使用您真正的帳戶和金鑰來取代帳戶名稱和帳戶金鑰。

	blob_service = BlobService(account_name="<your_account_name>", account_key="<your_account_key>")

使用下列方法，將資料上傳至 Blob：

1. put\_block\_blob\_from\_path (從指定路徑上傳檔案的內容)
2. put\_block\_blob\_from\_file (從任何已經開啟的檔案/資料流上傳內容)
3. put\_block\_blob\_from\_bytes (上傳位元組陣列)
4. put\_block\_blob\_from\_text (使用指定的編碼方式上傳指定的文字值)

下列程式碼範例會將本機檔案上傳至容器：

	blob_service.put_block_blob_from_path("<your_container_name>", "<your_blob_name>", "<your_local_file_name>")

下列程式碼範例會將本機目錄中的所有檔案 (不含目錄) 上傳至 Blob 儲存體：

	from azure.storage.blob import BlobService
	from os import listdir
	from os.path import isfile, join

	# Set parameters here
	ACCOUNT_NAME = "<your_account_name>"
	ACCOUNT_KEY = "<your_account_key>"
	CONTAINER_NAME = "<your_container_name>"
	LOCAL_DIRECT = "<your_local_directory>"		

	blob_service = BlobService(account_name=ACCOUNT_NAME, account_key=ACCOUNT_KEY)
	# find all files in the LOCAL_DIRECT (excluding directory)
	local_file_list = [f for f in listdir(LOCAL_DIRECT) if isfile(join(LOCAL_DIRECT, f))]

	file_num = len(local_file_list)
	for i in range(file_num):
	    local_file = join(LOCAL_DIRECT, local_file_list[i])
	    blob_name = local_file_list[i]
	    try:
	        blob_service.put_block_blob_from_path(CONTAINER_NAME, blob_name, local_file)
	    except:
	        print "something wrong happened when uploading the data %s"%blob_name


## 從 Blob 下載資料

使用下列方法，從 Blob 下載資料：
1. get\_blob\_to\_path
2. get\_blob\_to\_file
3. get\_blob\_to\_bytes
4. get\_blob\_to\_text

這些方法可以在資料大小超過 64 MB 時執行必要的區塊化動作。

下列程式碼範例會將容器中 Blob 的內容下載至本機檔案：

	blob_service.get_blob_to_path("<your_container_name>", "<your_blob_name>", "<your_local_file_name>")

下列程式碼範例會從容器下載所有 Blob。它會使用 list\_blobs 來取得容器中可用的 Blob 清單，並將它們下載至本機目錄。

	from azure.storage.blob import BlobService
	from os.path import join

	# Set parameters here
	ACCOUNT_NAME = "<your_account_name>"
	ACCOUNT_KEY = "<your_account_key>"
	CONTAINER_NAME = "<your_container_name>"
	LOCAL_DIRECT = "<your_local_directory>"		

	blob_service = BlobService(account_name=ACCOUNT_NAME, account_key=ACCOUNT_KEY)

	# List all blobs and download them one by one
	blobs = blob_service.list_blobs(CONTAINER_NAME)
	for blob in blobs:
	    local_file = join(LOCAL_DIRECT, blob.name)
	    try:
	        blob_service.get_blob_to_path(CONTAINER_NAME, blob.name, local_file)
	    except:
	        print "something wrong happened when downloading the data %s"%blob.name

<!---HONumber=AcomDC_0921_2016-->