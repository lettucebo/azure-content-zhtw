<properties
    pageTitle="如何使用 iOS 中的 Azure Blob 儲存體 | Microsoft Azure"
	description="使用 Azure Blob 儲存體 (物件儲存體) 在雲端中儲存非結構化資料。"
    services="storage"
    documentationCenter="ios"
    authors="micurd"
    manager="jahogg"
    editor="tysonn"/>

<tags
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="objective-c"
    ms.topic="article"
    ms.date="09/20/2016"
    ms.author="micurd;tamram"/>

# 如何使用 iOS 的 Blob 儲存體

[AZURE.INCLUDE [storage-selector-blob-include](../../includes/storage-selector-blob-include.md)] 
<br/> 
[AZURE.INCLUDE [storage-try-azure-tools-blobs](../../includes/storage-try-azure-tools-blobs.md)]

## Overview

本文將示範如何使用 Microsoft Azure Blob 儲存體執行一般案例。這些範例均以 Objective-C 撰寫，並使用 [Azure Storage Client Library for iOS](https://github.com/Azure/azure-storage-ios)。所涵蓋的案例包括「上傳」、「列出」、「下載」及「刪除」Blob。如需 Blob 的詳細資訊，請參閱[後續步驟](#next-steps)一節。您也可以下載[範例應用程式](https://github.com/Azure/azure-storage-ios/tree/master/BlobSample)，以快速查看在 iOS 應用程式中使用 Azure 儲存體的方法。

[AZURE.INCLUDE [storage-blob-concepts-include](../../includes/storage-blob-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../../includes/storage-create-account-include.md)]

## 將 Azure 儲存體 iOS 程式庫匯入您的應用程式中

您可以透過使用 [Azure 儲存體 CocoaPod](https://cocoapods.org/pods/AZSClient) 或匯入 **Framework** 檔，將 Azure 儲存體 iOS 程式庫匯入至您的應用程式。

## CocoaPod

1. 如果您還沒有這麼做，請開啟終端機視窗，並執行下列命令，在電腦上[安裝 CocoaPods](https://guides.cocoapods.org/using/getting-started.html#toc_3)

        sudo gem install cocoapods

2. 接下來，在專案目錄 (包含您 `.xcodeproj` 檔案的目錄)，建立稱為 `Podfile` 的檔案 (無副檔名)。將下列內容加入 `Podfile` 並儲存

        pod 'AZSClient'

3. 在終端機視窗中，巡覽至專案目錄並執行下列命令

        pod install

4. 如果您的 `.xcodeproj` 在 Xcode 中開啟，請先將它關閉。在您的專案目錄中開啟新建立的專案檔案 (副檔名為 `.xcworkspace`)。這是您從現在開始會使用的檔案。

## Framework
為了使用 Azure 儲存體 iOS 程式庫，您必須先建置架構檔案。

1. 首先，請下載或複製 [azure-storage-ios repo](https://github.com/azure/azure-storage-ios)。

2. 移至 *azure-storage-ios* -> *Lib* -> *Azure 儲存體用戶端程式庫*，然後在 Xcode 中開啟 `AZSClient.xcodeproj`。

3. 在 Xcode 左上方，將作用中配置從「Azure 儲存體用戶端程式庫」變更為「架構」。

4. 建置專案 (⌘ + B)。這會在您的桌面上建立 `AZSClient.framework` 檔案。

您可以透過下列方式，將 Framework 檔案匯入至您的應用程式：

1. 建立新的專案，或在 Xcode 中開啟現有的專案。

2. 按一下左側導覽列中的專案，然後按一下專案編輯器頂端的 [一般] 索引標籤。

3. 在 [連結的架構和程式庫] 區段下，按一下 [新增] 按鈕 (+)。

4. 按一下 [新增其他...]。導覽至您剛才建立的 `AZSClient.framework` 檔案，並加以新增。

5. 在 [連結的架構和程式庫] 區段下，再按一下 [新增] 按鈕 (+)。

6. 在已提供的程式庫清單中搜尋 `libxml2.2.dylib`，並將它新增至您的專案。

7. 按一下位於專案編輯器頂端的 [建置設定] 索引標籤。

8. 在 [搜尋路徑] 區段中，按兩下 [Framework 搜尋路徑]，並將路徑新增至您的 `AZSClient.framework` 檔案。

## 匯入陳述式
您必須在要叫用 Azure 儲存體 API 的檔案中納入下列匯入陳述式。

    // Include the following import statement to use blob APIs.
    #import <AZSClient/AZSClient.h>

[AZURE.INCLUDE [storage-mobile-authentication-guidance](../../includes/storage-mobile-authentication-guidance.md)]

## 非同步作業
> [AZURE.NOTE] 所有對服務執行要求的方法，都是非同步作業。在程式碼範例中，您會發現這些方法具有完成處理常式。完成處理常式內的程式碼會在要求完成**之後**執行。完成處理常式後面的程式碼會在進行要求**期間**執行。

## 建立容器
儲存體 Blob 中的每個 Blob 必須位於一個容器中。下列範例說明如何在您的儲存體帳戶中建立名為 *newcontainer* 的容器 (如果還不存在)。為您的容器選擇名稱時，請留意上面提到的命名規則。

    -(void)createContainer{
      NSError *accountCreationError;

      // Create a storage account object from a connection string.
      AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here" error:&accountCreationError];

      if(accountCreationError){
         NSLog(@"Error in creating account.");
      }

      // Create a blob service client object.
      AZSCloudBlobClient *blobClient = [account getBlobClient];

      // Create a local container object.
      AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"newcontainer"];

      // Create container in your Storage account if the container doesn't already exist
      [blobContainer createContainerIfNotExistsWithCompletionHandler:^(NSError *error, BOOL exists) {
         if (error){
             NSLog(@"Error in creating container.");
         }
      }];
    }

您可以查看 [Microsoft Azure 儲存體總管](http://storageexplorer.com)，並驗證 *newcontainer* 位於儲存體帳戶的容器清單中，以確認運作正常。

## 設定容器權限
依預設會針對 [私人] 存取設定容器的權限。不過，容器會提供幾個不同的容器存取選項：

- **私人**：只有帳戶擁有者可以讀取容器和 Blob 資料。

- **Blob**：您可以透過匿名要求讀取此容器內的 Blob 資料，但您無法使用容器資料。用戶端無法透過匿名要求列舉容器內的 Blob。

- **容器**：可以透過匿名要求讀取容器和 Blob 資料。用戶端可以透過匿名要求列舉容器內的 Blob，但無法列舉儲存體帳戶內的容器。

下列範例說明如何建立具有 [容器] 存取權限，而允許網際網路上的所有使用者進行公用、唯讀存取的容器：

    -(void)createContainerWithPublicAccess{
        NSError *accountCreationError;

        // Create a storage account object from a connection string.
        AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here" error:&accountCreationError];

        if(accountCreationError){
            NSLog(@"Error in creating account.");
        }

        // Create a blob service client object.
        AZSCloudBlobClient *blobClient = [account getBlobClient];

        // Create a local container object.
        AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"containerpublic"];

        // Create container in your Storage account if the container doesn't already exist
        [blobContainer createContainerIfNotExistsWithAccessType:AZSContainerPublicAccessTypeContainer requestOptions:nil operationContext:nil completionHandler:^(NSError *error, BOOL exists){
            if (error){
                NSLog(@"Error in creating container.");
            }
        }];
    }

## 將 Blob 上傳至容器
如 [Blob 服務概念](#blob-service-concepts)一節中所述，Blob 儲存體提供三種不同類型的 Blob：區塊 Blob、附加 Blob 和分頁 Blob。目前，Azure 儲存體 iOS 程式庫僅支援區塊 Blob。在大多數情況下，建議使用區塊 Blob 的類型。

下列範例說明如何從 NSString 上傳區塊 Blob。如果此容器中有相同名稱的 Blob 存在，此 Blob 的內容將會被覆寫。

    -(void)uploadBlobToContainer{
        NSError *accountCreationError;

        // Create a storage account object from a connection string.
        AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here" error:&accountCreationError];

        if(accountCreationError){
            NSLog(@"Error in creating account.");
        }

        // Create a blob service client object.
        AZSCloudBlobClient *blobClient = [account getBlobClient];

        // Create a local container object.
        AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"containerpublic"];

        [blobContainer createContainerIfNotExistsWithAccessType:AZSContainerPublicAccessTypeContainer requestOptions:nil operationContext:nil completionHandler:^(NSError *error, BOOL exists)
         {
             if (error){
                 NSLog(@"Error in creating container.");
             }
             else{
                 // Create a local blob object
                 AZSCloudBlockBlob *blockBlob = [blobContainer blockBlobReferenceFromName:@"sampleblob"];

                 // Upload blob to Storage
                 [blockBlob uploadFromText:@"This text will be uploaded to Blob Storage." completionHandler:^(NSError *error) {
                     if (error){
                         NSLog(@"Error in creating blob.");
                     }
                 }];
             }
         }];
    }

您可以查看 [Microsoft Azure 儲存體總管](http://storageexplorer.com)，並驗證容器 *containerpublic* 包含 Blob *sampleblob*，以確認運作正常。在此範例中我們使用公用容器，所以您也可以移至 Blob URI 來確認運作是否正常：

    https://nameofyourstorageaccount.blob.core.windows.net/containerpublic/sampleblob

除了從 NSString 上傳區塊 Blob 以外，NSData、NSInputStream 或本機檔案也有類似的方法。

## 列出容器中的 Blob
下列範例說明如何列出容器中的所有 Blob。執行此作業時，請留意下列參數：

- **continuationToken** - 接續權杖表示列出作業應從哪裡開始。如果未提供權杖，則會從頭列出 Blob。可以列出任何數目的 Blob，從零到設定的最大值皆可。即使此方法傳回零個結果，如果 `results.continuationToken` 不是 nil，則可能會有服務上的更多 Blob 未列出。
- **prefix** -您可以指定要用於 Blob 列出作業的前置詞。只有開頭為此前置詞的 Blob 才會列出。
- **useFlatBlobListing** - 如[命名與參考容器和 Blob](#naming-and-referencing-containers-and-blobs) 一節中所述，雖然 Blob 服務是一般儲存體配置，雖然您可以路徑資訊為 Blob 命名，以建立虛擬階層。不過目前不支援非一般列出；不久後將可支援。目前，此值應為 `YES`
- **blobListingDetails** - 您可以指定列出 Blob 時所要包含的項目
	- `AZSBlobListingDetailsNone`：僅列出已認可的 Blob，且不傳回 Blob 中繼資料。
	- `AZSBlobListingDetailsSnapshots`：列出已認可的 Blob 和 Blob 快照。
	- `AZSBlobListingDetailsMetadata`：為清單中傳回的每個 Blob 擷取 Blob 中繼資料。
	- `AZSBlobListingDetailsUncommittedBlobs`：列出已認可及未認可的 Blob。
	- `AZSBlobListingDetailsCopy`：在清單中包含複製屬性。
	- `AZSBlobListingDetailsAll`：列出所有可用的已認可 Blob、未認可的 Blob 和快照，並傳回這些 Blob 的所有中繼資料和複製狀態。
- **maxResults** - 要為此作業傳回的結果數目上限。使用 -1 則不會設定限制。
- **completionHandler** - 要執行的程式碼區塊與列出作業的結果。

在此範例中，將會在每次傳回接續權杖時使用 Helper 方法以遞迴方式呼叫列出 Blob 方法。

    -(void)listBlobsInContainer{
        NSError *accountCreationError;

        // Create a storage account object from a connection string.
        AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here" error:&accountCreationError];

        if(accountCreationError){
            NSLog(@"Error in creating account.");
        }

        // Create a blob service client object.
        AZSCloudBlobClient *blobClient = [account getBlobClient];

        // Create a local container object.
        AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"containerpublic"];

        //List all blobs in container
        [self listBlobsInContainerHelper:blobContainer continuationToken:nil prefix:nil blobListingDetails:AZSBlobListingDetailsAll maxResults:-1 completionHandler:^(NSError *error) {
            if (error != nil){
                NSLog(@"Error in creating container.");
            }
        }];
    }

    //List blobs helper method
    -(void)listBlobsInContainerHelper:(AZSCloudBlobContainer *)container continuationToken:(AZSContinuationToken *)continuationToken prefix:(NSString *)prefix blobListingDetails:(AZSBlobListingDetails)blobListingDetails maxResults:(NSUInteger)maxResults completionHandler:(void (^)(NSError *))completionHandler
    {
        [container listBlobsSegmentedWithContinuationToken:continuationToken prefix:prefix useFlatBlobListing:YES blobListingDetails:blobListingDetails maxResults:maxResults completionHandler:^(NSError *error, AZSBlobResultSegment *results) {
            if (error)
            {
                completionHandler(error);
            }
            else
            {
                for (int i = 0; i < results.blobs.count; i++) {
                    NSLog(@"%@",[(AZSCloudBlockBlob *)results.blobs[i] blobName]);
                }
                if (results.continuationToken)
                {
                    [self listBlobsInContainerHelper:container continuationToken:results.continuationToken prefix:prefix blobListingDetails:blobListingDetails maxResults:maxResults completionHandler:completionHandler];
                }
                else
                {
                    completionHandler(nil);
                }
            }
        }];
    }


## 下載 Blob

下列範例說明如何將 Blob 下載到 NSString 物件。

    -(void)downloadBlobToString{
        NSError *accountCreationError;

        // Create a storage account object from a connection string.
        AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here" error:&accountCreationError];

        if(accountCreationError){
            NSLog(@"Error in creating account.");
        }

        // Create a blob service client object.
        AZSCloudBlobClient *blobClient = [account getBlobClient];

        // Create a local container object.
        AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"containerpublic"];

        // Create a local blob object
        AZSCloudBlockBlob *blockBlob = [blobContainer blockBlobReferenceFromName:@"sampleblob"];

        // Download blob
        [blockBlob downloadToTextWithCompletionHandler:^(NSError *error, NSString *text) {
            if (error) {
                NSLog(@"Error in downloading blob");
            }
            else{
                NSLog(@"%@",text);
            }
        }];
    }

## 刪除 Blob

下列範例說明如何刪除 Blob。

    -(void)deleteBlob{
        NSError *accountCreationError;

        // Create a storage account object from a connection string.
        AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here" error:&accountCreationError];

        if(accountCreationError){
            NSLog(@"Error in creating account.");
        }

        // Create a blob service client object.
        AZSCloudBlobClient *blobClient = [account getBlobClient];

        // Create a local container object.
        AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"containerpublic"];

        // Create a local blob object
        AZSCloudBlockBlob *blockBlob = [blobContainer blockBlobReferenceFromName:@"sampleblob1"];

        // Delete blob
        [blockBlob deleteWithCompletionHandler:^(NSError *error) {
            if (error) {
                NSLog(@"Error in deleting blob.");
            }
        }];
    }

## 刪除 Blob 容器

下列範例說明如何刪除容器。

    -(void)deleteContainer{
      NSError *accountCreationError;

      // Create a storage account object from a connection string.
      AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here" error:&accountCreationError];

      if(accountCreationError){
         NSLog(@"Error in creating account.");
      }

      // Create a blob service client object.
      AZSCloudBlobClient *blobClient = [account getBlobClient];

      // Create a local container object.
      AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"containerpublic"];

      // Delete container
      [blobContainer deleteContainerIfExistsWithCompletionHandler:^(NSError *error, BOOL success) {
         if(error){
             NSLog(@"Error in deleting container");
         }
      }];
    }

## 後續步驟

您現在已經了解如何從 iOS 中使用 Blob 儲存體，請參考下列連結以深入了解 iOS 程式庫和儲存體服務。

- [Azure Storage Client Library for iOS](https://github.com/azure/azure-storage-ios)
- [Azure 儲存體 iOS 參考文件](http://azure.github.io/azure-storage-ios/)
- [Azure 儲存體服務 REST API](https://msdn.microsoft.com/library/azure/dd179355.aspx)
- [Azure 儲存體團隊部落格](http://blogs.msdn.com/b/windowsazurestorage)

如果您有關於此程式庫的問題，歡迎您貼文到我們的 [MSDN Azure 論壇](http://social.msdn.microsoft.com/Forums/windowsazure/home?forum=windowsazuredata)或[堆疊溢位](http://stackoverflow.com/questions/tagged/windows-azure-storage+or+windows-azure-storage+or+azure-storage-blobs+or+azure-storage-tables+or+azure-table-storage+or+windows-azure-queues+or+azure-storage-queues+or+azure-storage-emulator+or+azure-storage-files)。如果您有 Azure 儲存體功能方面的建議，請貼文到 [Azure 儲存體意見反應](https://feedback.azure.com/forums/217298-storage/)。

<!---HONumber=AcomDC_0921_2016-->