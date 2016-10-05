<properties
	pageTitle="如何使用 Node.js 中的 Blob 儲存體 | Microsoft Azure"
	description="使用 Azure Blob 儲存體 (物件儲存體) 在雲端中儲存非結構化資料。"
	services="storage"
	documentationCenter="nodejs"
	authors="rmcmurray"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="storage"
	ms.workload="storage"
	ms.tgt_pltfrm="na"
	ms.devlang="nodejs"
	ms.topic="article"
    ms.date="08/11/2016"
	ms.author="micurd"/>



# 如何使用 Node.js 的 Blob 儲存體

[AZURE.INCLUDE [storage-selector-blob-include](../../includes/storage-selector-blob-include.md)]
<br/>
[AZURE.INCLUDE [storage-try-azure-tools-queues](../../includes/storage-try-azure-tools-blobs.md)]

## Overview

本文章示範如何使用 Blob 儲存體執行一般案例。這些範例透過 Node.js API 撰寫。涵蓋的案例包括如何上傳、列出、下載及刪除 blob。

[AZURE.INCLUDE [storage-blob-concepts-include](../../includes/storage-blob-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../../includes/storage-create-account-include.md)]

## 建立 Node.js 應用程式

如需如何建立 Node.js 應用程式的相關指示，請參閱[在 Azure App Service 中建立 Node.js Web 應用程式]、[使用 Windows PowerShell 建立 Node.js 應用程式並部署至 Azure 雲端服務]，或[使用 Web Matrix 建立 Node.js Web 應用程式並部署至 Azure]。

## 設定您的應用程式以存取儲存體

若要使用 Azure 儲存體，您需要 Azure Storage SDK for Node.js，這包含一組便利程式庫，能與儲存體 REST 服務通訊。

### 使用 Node Package Manager (NPM) 取得封裝

1.  使用命令列介面，例如 **PowerShell** (Windows)、[終端機]\(Mac) 或 **Bash** (Unix)，瀏覽到您建立範例應用程式的資料夾。

2.  在命令視窗中輸入 **npm install azure-storage**。此命令的輸出類似下列程式碼範例。

		azure-storage@0.5.0 node_modules\azure-storage
		+-- extend@1.2.1
		+-- xmlbuilder@0.4.3
		+-- mime@1.2.11
		+-- node-uuid@1.4.3
		+-- validator@3.22.2
		+-- underscore@1.4.4
		+-- readable-stream@1.0.33 (string_decoder@0.10.31, isarray@0.0.1, inherits@2.0.1, core-util-is@1.0.1)
		+-- xml2js@0.2.7 (sax@0.5.2)
		+-- request@2.57.0 (caseless@0.10.0, aws-sign2@0.5.0, forever-agent@0.6.1, stringstream@0.0.4, oauth-sign@0.8.0, tunnel-agent@0.4.1, isstream@0.1.2, json-stringify-safe@5.0.1, bl@0.9.4, combined-stream@1.0.5, qs@3.1.0, mime-types@2.0.14, form-data@0.2.0, http-signature@0.11.0, tough-cookie@2.0.0, hawk@2.3.1, har-validator@1.8.0)

3.  您可以手動執行 **ls** 命令，確認已建立 **node\_modules** 資料夾。在該資料夾中找出 **azure-storage** 封裝，當中包含存取儲存體所需的程式庫。

### 匯入封裝

使用記事本或其他文字編輯器，將以下內容新增至您要使用儲存體之應用程式的 **server.js** 檔案頂端：

    var azure = require('azure-storage');

## 設定 Azure 儲存體連接

Azure 模組會讀取環境變數 `AZURE_STORAGE_ACCOUNT` 及 `AZURE_STORAGE_ACCESS_KEY`，或讀取 `AZURE_STORAGE_CONNECTION_STRING` 以取得連接 Azure 儲存體帳戶所需的資訊。如果未設定這些環境變數，則呼叫 **createBlobService** 時必須指定帳戶資訊。

如需針對 Azure Web 應用程式在 [Azure 入口網站](https://portal.azure.com)中設定環境變數的範例，請參閱[使用 Azure 表格服務的 Node.js Web 應用程式]。

## 建立容器

**BlobService** 物件讓您能使用容器及 blob。下列程式碼會建立 **BlobService** 物件。將下列內容新增至接近 **server.js** 的頂端：

    var blobSvc = azure.createBlobService();

> [AZURE.NOTE] 使用 **createBlobServiceAnonymous** 並提供主機位址，可匿名存取 Blob。例如，使用 `var blobSvc = azure.createBlobServiceAnonymous('https://myblob.blob.core.windows.net/');`。

[AZURE.INCLUDE [storage-container-naming-rules-include](../../includes/storage-container-naming-rules-include.md)]

若要建立新的容器，請使用 **createContainerIfNotExists**。以下程式碼範例會建立一個名為 'mycontainer' 的新容器：

	blobSvc.createContainerIfNotExists('mycontainer', function(error, result, response){
	    if(!error){
	      // Container exists and is private
	    }
	});

如果是新建立的容器，`result.created` 為 true。如果容器已存在，則 `result.created` 為 false。`response` 包含作業的相關資訊，包括容器的 ETag 資訊。

### 容器安全性

依預設，新的容器屬私人性質，無法以匿名方式存取。若要將容器變成公開，以便以匿名方式進行存取，您可以將容器的存取等級設為 **blob** 或 **container**。

* **blob** - 允許對此容器內的 Blob 內容和中繼資料的匿名讀取存取，但不包含對容器中繼資料的匿名讀取存取，例如列出容器內的所有 Blob

* **container** - 允許對 Blob 內容和中繼資料及容器中繼資料的匿名讀取存取

下列程式碼範例示範將存取等級設為 **blob**：

	blobSvc.createContainerIfNotExists('mycontainer', {publicAccessLevel : 'blob'}, function(error, result, response){
	    if(!error){
	      // Container exists and allows
	      // anonymous read access to blob
	      // content and metadata within this container
	    }
	});

或者，您可以使用 **setContainerAcl** 指定存取等級，以修改容器的存取等級。下列程式碼範例會將存取等級變更為 container：

	blobSvc.setContainerAcl('mycontainer', null /* signedIdentifiers */, {publicAccessLevel : 'container'} /* publicAccessLevel*/, function(error, result, response){
	  if(!error){
	    // Container access level set to 'container'
	  }
	});

結果包含操作的相關資訊，包括容器的目前 **ETag**。

### 篩選器

您可以將選用的篩選作業套用到使用 **BlobService** 執行的作業。篩選作業可以包含記錄、自動重試等。篩選器是使用簽章實作方法的物件：

	function handle (requestOptions, next)

在對要求選項進行前處理之後，方法需要呼叫 "next" 並傳遞具有下列簽章的回呼：

	function (returnObject, finalCallback, next)

在此回呼中，以及處理 returnObject (來自對伺服器之要求的回應) 之後，回呼需要叫用 next (如果存在) 以繼續處理其他篩選，或是直接叫用 finalCallback 結束服務叫用。

Azure SDK for Node.js 包含了實作重試邏輯的兩個篩選器：**ExponentialRetryPolicyFilter** 和 **LinearRetryPolicyFilter**。以下會建立使用 **ExponentialRetryPolicyFilter** 的 **BlobService** 物件：

	var retryOperations = new azure.ExponentialRetryPolicyFilter();
	var blobSvc = azure.createBlobService().withFilter(retryOperations);

## 將 Blob 上傳至容器

有三種類型的 Blob：區塊 Blob、分頁 Blob 和附加 Blob。Block Blob 可讓您更有效率地上傳大型資料。附加 Blob 已針對附加作業最佳化。分頁 Blob 已針對讀/寫作業最佳化。如需詳細資訊，請參閱[了解區塊 Blob、附加 Blob 和分頁 Blob](http://msdn.microsoft.com/library/azure/ee691964.aspx)。

### 區塊 Blob

若要將資料上傳至區塊 Blob，請使用下列方法：

* **createBlockBlobFromLocalFile** - 建立新的區塊 Blob 並上傳檔案的內容

* **createBlockBlobFromStream** - 建立新的區塊 Blob 並上傳串流的內容

* **createBlockBlobFromText** - 建立新的區塊 Blob 並上傳字串的內容

* **createWriteStreamToBlockBlob** - 提供對區塊 Blob 的寫入串流

下列程式碼範例會將 **test.txt** 檔的內容上傳至 **myblob**。

	blobSvc.createBlockBlobFromLocalFile('mycontainer', 'myblob', 'test.txt', function(error, result, response){
	  if(!error){
	    // file uploaded
	  }
	});

這些方法傳回的 `result` 包含作業的相關資訊，例如 Blob 的 **ETag**。

### 附加 Blob

若要將資料上傳至附加 Blob，請使用下列方法：

* **createAppendBlobFromLocalFile** - 建立新的附加 Blob 並上傳檔案的內容

* **createAppendBlobFromStream** - 建立新的附加 Blob 並上傳串流的內容

* **createAppendBlobFromText** - 建立新的附加 Blob 並上傳字串的內容

* **createWriteStreamToNewAppendBlob** - 建立新的附加 Blob，然後提供串流來寫入它

下列程式碼範例會將 **test.txt** 檔案的內容上傳至 **myappendblob**。

	blobSvc.createAppendBlobFromLocalFile('mycontainer', 'myappendblob', 'test.txt', function(error, result, response){
	  if(!error){
	    // file uploaded
	  }
	});

若要將區塊附加到現有附加 Blob，請使用下列方法︰

* **appendFromLocalFile** - 將檔案的內容附加到現有附加 Blob

* **appendFromStream** - 將串流的內容附加到現有附加 Blob

* **appendFromText** - 將字串的內容附加到現有附加 Blob

* **appendBlockFromStream** - 將串流的內容附加到現有附加 Blob

* **appendBlockFromText** - 將字串的內容附加到現有附加 Blob

> [AZURE.NOTE] appendFromXXX API 會讓部分用戶端驗證立即失敗，以避免不必要的伺服器呼叫。appendBlockFromXXX 則否。

下列程式碼範例會將 **test.txt** 檔案的內容上傳至 **myappendblob**。

	blobSvc.appendFromText('mycontainer', 'myappendblob', 'text to be appended', function(error, result, response){
	  if(!error){
	    // text appended
	  }
	});


### 分頁 Blob

若要將資料上傳至分頁 Blob，請使用下列方法：

* **createPageBlob** - 建立特定長度的新分頁 Blob

* **createPageBlobFromLocalFile** - 建立新的分頁 Blob 並上傳檔案的內容

* **createPageBlobFromStream** - 建立新的分頁 Blob 並上傳串流的內容

* **createWriteStreamToExistingPageBlob** - 提供對現有分頁 Blob 的寫入串流

* **createWriteStreamToNewPageBlob** - 建立新的分頁 Blob，然後提供串流來寫入它

下列程式碼範例會將 **test.txt** 檔的內容上傳至 **mypageblob**。

	blobSvc.createPageBlobFromLocalFile('mycontainer', 'mypageblob', 'test.txt', function(error, result, response){
	  if(!error){
	    // file uploaded
	  }
	});

> [AZURE.NOTE] 分頁 Blob 由 512 位元組的「分頁」組成。如果上傳的資料大小不是 512 的倍數，則會接收到錯誤。

## 列出容器中的 Blob

若要列出容器中的 Blob，請使用 **listBlobsSegmented** 方法。若要傳回具有特定首碼的 Blob，請使用 **listBlobsSegmentedWithPrefix**。

	blobSvc.listBlobsSegmented('mycontainer', null, function(error, result, response){
	  if(!error){
	      // result.entries contains the entries
	      // If not all blobs were returned, result.continuationToken has the continuation token.
	  }
	});

`result` 包含 `entries` 集合，此集合是描述每個 Blob 的物件陣列。若無法傳回所有 Blob，`result` 也會提供 `continuationToken`，其可作為第二個參數來擷取更多項目。

## 下載 Blob

若要從 Blob 下載資料，請使用下列方法：

* **getBlobToLocalFile** - 將 Blob 內容寫入檔案

* **getBlobToStream** - 將 Blob 內容寫入串流

* **getBlobToText** - 將 Blob 內容寫入字串

* **createReadStream** - 提供串流來讀取 Blob

下列程式碼範例示範使用 **getBlobToStream** 來下載 **myblob** Blob 的內容，並使用串流存放到 **output.txt** 檔案：

	var fs = require('fs');
	blobSvc.getBlobToStream('mycontainer', 'myblob', fs.createWriteStream('output.txt'), function(error, result, response){
	  if(!error){
	    // blob retrieved
	  }
	});

`result` 包含 Blob 的相關資訊，包括 **ETag** 資訊。

## 刪除 Blob

最後，呼叫 **deleteBlob** 以刪除 blob。下列程式碼範例會刪除名為 **myblob** 的 Blob。

	blobSvc.deleteBlob(containerName, 'myblob', function(error, response){
	  if(!error){
		// Blob has been deleted
	  }
	});

## 並行存取

若要支援從多個用戶端或多個程序執行個體並行存取 Blob，您可以使用 **ETag** 或「租用」。

* **Etag** - 提供方法來偵測 Blob 或容器已被另一個程序修改過

* **租用** - 提供方法來取得在一段時間內對 Blob 的獨佔、可更新、寫入或刪除存取權

### ETag

若您需要允許多個用戶端或執行個體同時寫入區塊 Blob 或分頁 Blob，請使用 ETag。ETag 可讓您判斷容器或 Blob 自從您最初讀取或建立它之後是否已修改，這樣可讓您避免覆寫另一個用戶端或程序已認可的變更。

使用選用的 `options.accessConditions` 參數，可以設定 ETag 條件。只有在 Blob 已存在，且 `etagToMatch` 中包含 ETag 值時，下列程式碼範例才會上傳 **test.txt** 檔案。

	blobSvc.createBlockBlobFromLocalFile('mycontainer', 'myblob', 'test.txt', { accessConditions: { EtagMatch: etagToMatch} }, function(error, result, response){
	    if(!error){
	    // file uploaded
	  }
	});

使用 ETag 時，一般模式為：

1. 取得執行建立、列出或取得操作之後的 ETag。

2. 執行動作，檢查 ETag 值未被修改。

若值已被修改，這表示另一個用戶端或執行個體自從您取得 ETag 值之後已修改 Blob 或容器。

### 租用

使用 **acquireLease** 方法，並指定您要取得租用的 Blob 或容器，即可取得新的租用。例如，下列程式碼可取得 **myblob** 的租用。

	blobSvc.acquireLease('mycontainer', 'myblob', function(error, result, response){
	  if(!error) {
	    console.log('leaseId: ' + result.id);
	  }
	});

**myblob** 的後續作業必須提供 `options.leaseId` 參數。租用識別碼會從 **acquireLease** 作為 `result.id` 傳回。

> [AZURE.NOTE] 依預設，租用期間無限制。若要指定有限期間 (15 到 60 秒)，您可以提供 `options.leaseDuration` 參數。

若要移除租用，請使用 **releaseLease**。若要中止租用，但在原始期間到期之前不讓其他人取得新的租用，請使用 **breakLease**。

## 使用共用存取簽章

共用存取簽章 (SAS) 可安全地提供對 Blob 和容器的精確存取，而不必提供您的儲存體帳戶名稱或金鑰。共用存取簽章通常用來提供對資料的有限存取，例如允許行動應用程式存取 Blob。

> [AZURE.NOTE] 雖然您也可以用匿名方式存取 Blob，但共用存取簽章可讓您提供更受控制的存取，因為您必須產生 SAS。

信任的應用程式 (例如雲端型服務) 會使用 **BlobService** 的 **generateSharedAccessSignature** 來產生共用存取簽章，並提供它給不信任或不完全信任的應用程式 (例如行動應用程式)。共用存取簽章是使用原則來產生，該原則描述共用存取簽章有效期間的開始和結束日期，以及授與共用存取簽章持有者的存取等級。

下列程式碼範例會產生新的共用存取原則，讓共用存取簽章持有者對 **myblob** Blob 執行讀取操作，並於建立它之後的 100 分鐘過期。

	var startDate = new Date();
	var expiryDate = new Date(startDate);
	expiryDate.setMinutes(startDate.getMinutes() + 100);
	startDate.setMinutes(startDate.getMinutes() - 100);

	var sharedAccessPolicy = {
	  AccessPolicy: {
	    Permissions: azure.BlobUtilities.SharedAccessPermissions.READ,
	    Start: startDate,
	    Expiry: expiryDate
	  },
	};

	var blobSAS = blobSvc.generateSharedAccessSignature('mycontainer', 'myblob', sharedAccessPolicy);
	var host = blobSvc.host;

請注意，也必須提供主機資訊，因為共用存取簽章持有者嘗試存取容器時需要此資訊。

用戶端應用程式接著以 **BlobServiceWithSAS** 來使用共用存取簽章，對 Blob 執行操作。以下會取得 **myblob** 的相關資訊。

	var sharedBlobSvc = azure.createBlobServiceWithSas(host, blobSAS);
	sharedBlobSvc.getBlobProperties('mycontainer', 'myblob', function (error, result, response) {
	  if(!error) {
	    // retrieved info
	  }
	});

由於共用存取簽章是利用唯讀存取權所產生，如果嘗試修改 blob，就會傳回錯誤。

### 存取控制清單

您也可以使用存取控制清單 (ACL) 來設定 SAS 的存取原則。若您要允許用戶端存取容器，但對每個用戶端提供不同的存取原則，則這會很有用。

ACL 是使用存取原則陣列來實作，每個原則有相關聯的識別碼。下列程式碼範例定義兩個原則，其中一個用於 'user1'，另一個用於 'user2'：

	var sharedAccessPolicy = {
	  user1: {
	    Permissions: azure.BlobUtilities.SharedAccessPermissions.READ,
	    Start: startDate,
	    Expiry: expiryDate
	  },
	  user2: {
	    Permissions: azure.BlobUtilities.SharedAccessPermissions.WRITE,
	    Start: startDate,
	    Expiry: expiryDate
	  }
	};

下列程式碼範例會取得 **mycontainer** 的目前 ACL，然後使用 **setBlobAcl** 來加入新的原則。此方法允許：

	var extend = require('extend');
	blobSvc.getBlobAcl('mycontainer', function(error, result, response) {
	  if(!error){
	    var newSignedIdentifiers = extend(true, result.signedIdentifiers, sharedAccessPolicy);
	    blobSvc.setBlobAcl('mycontainer', newSignedIdentifiers, function(error, result, response){
	      if(!error){
	        // ACL set
	      }
	    });
	  }
	});

設定 ACL 之後，您可以根據原則的識別碼來建立共用存取簽章。下列程式碼範例會為 'user2' 建立新的共用存取簽章：

	blobSAS = blobSvc.generateSharedAccessSignature('mycontainer', { Id: 'user2' });

## 後續步驟

如需詳細資訊，請參閱下列資源。

-   [Azure Storage SDK for Node API 參考][]
-   [Azure 儲存體團隊部落格][]
-   GitHub 上的 [Azure Storage SDK for Node][] 儲存機制
-   [Node.js 開發人員中心](/develop/nodejs/)
-   [使用 AzCopy 命令列公用程式傳輸資料](storage-use-azcopy.md)

[Azure Storage SDK for Node]: https://github.com/Azure/azure-storage-node

[在 Azure App Service 中建立 Node.js Web 應用程式]: ../app-service-web/web-sites-nodejs-develop-deploy-mac.md
[Node.js Cloud Service with Storage]: ../cloud-services/storage-nodejs-use-table-storage-cloud-service-app.md
[使用 Azure 表格服務的 Node.js Web 應用程式]: ../app-service-web/storage-nodejs-use-table-storage-web-site.md
[使用 Web Matrix 建立 Node.js Web 應用程式並部署至 Azure]: ../app-service-web/web-sites-nodejs-use-webmatrix.md
[Using the REST API]: http://msdn.microsoft.com/library/azure/hh264518.aspx
[Azure Portal]: https://portal.azure.com
[使用 Windows PowerShell 建立 Node.js 應用程式並部署至 Azure 雲端服務]: ../cloud-services/cloud-services-nodejs-develop-deploy-app.md
[Azure 儲存體團隊部落格]: http://blogs.msdn.com/b/windowsazurestorage/
[Azure Storage SDK for Node API 參考]: http://dl.windowsazure.com/nodestoragedocs/index.html

<!---HONumber=AcomDC_0921_2016-->