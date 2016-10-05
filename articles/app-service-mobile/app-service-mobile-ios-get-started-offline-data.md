<properties
	pageTitle="啟用 Azure 行動應用程式的離線同步處理 (iOS)"
	description="了解如何在 iOS 應用程式中使用 App Service Mobile Apps 快取和同步離線資料"
	documentationCenter="ios"
	authors="krisragh"
	manager="dwrede"
	editor=""
	services="app-service\mobile"/>

<tags
	ms.service="app-service-mobile"
	ms.workload="mobile"
	ms.tgt_pltfrm="mobile-ios"
	ms.devlang="objective-c"
	ms.topic="article"
	ms.date="08/01/2016"
	ms.author="krisragh"/>

# 啟用 iOS 行動應用程式的離線同步處理

[AZURE.INCLUDE [app-service-mobile-selector-offline](../../includes/app-service-mobile-selector-offline.md)]

## Overview

本教學課程說明 iOS 之 Azure Mobile Apps 的離線同步功能。離線同步處理可讓使用者與行動應用程式進行互動 - 檢視、新增或修改資料 - 即使沒有網路連線也可行。變更會儲存在本機資料庫中︰裝置上線後，這些變更就會與遠端後端進行同步處理。

如果這是您第一次接觸 Azure Mobile Apps，請先完成[建立 iOS 應用程式]教學課程。如果您不要使用下載的快速入門伺服器專案，必須將資料存取擴充套件新增至您的專案。如需伺服器擴充套件的詳細資訊，請參閱[使用 Azure Mobile Apps 的 .NET 後端伺服器 SDK](app-service-mobile-dotnet-backend-how-to-use-server-sdk.md)。

若要深入了解離線同步處理功能，請參閱 [Azure Mobile Apps 中的離線資料同步處理]主題。

## <a name="review-sync"></a>檢閱用戶端同步程式碼

您在[建立 iOS 應用程式]教學課程中下載的用戶端專案，已經包含了使用本機核心資料式資料庫支援離線同步處理的程式碼。這一節是已包含在教學課程程式碼中的內容摘要。如需此功能的概念性概觀，請參閱 [Azure Mobile Apps中的離線資料同步]。

Azure Mobile Apps 的離線資料同步功能可讓使用者在無法存取網路時，仍可與本機資料庫互動。若要在您的應用程式中使用這些功能，您可初始化 `MSClient` 的同步處理內容以及參考本機存放區。接著，請透過 `MSSyncTable` 介面參考您的資料表。

1. 在 **QSTodoService.m** (Objective-C) 或 **ToDoTableViewController.swift** (Swift) 中，注意成員 `syncTable` 的類型為 `MSSyncTable`。離線同步處理會使用此同步處理資料表介面，而不是 `MSTable`。使用同步處理資料表時，所有作業都會移至本機存放區，且只有在執行明確的發送和提取作業時才會與遠端後端同步處理。

    若要取得同步處理資料表的參考，請在 `MSClient` 使用 `syncTableWithName` 方法。若要移除離線同步處理功能，請改用 `tableWithName`。

2. 必須先初始化本機存放區，才可以執行資料表作業。以下是相關的程式碼。
	
	**Objective-C**：
	
	在 `QSTodoService.init` 方法中：
	
	
	        MSCoreDataStore *store = [[MSCoreDataStore alloc] initWithManagedObjectContext:context];
	        self.client.syncContext = [[MSSyncContext alloc] initWithDelegate:nil dataSource:store callback:nil];
	
	
	**Swift**：
	
	在 `ToDoTableViewController.viewDidLoad` 方法中：
	
	
	        let client = MSClient(applicationURLString: "http:// ...") // URI of the Mobile App
	        let managedObjectContext = (UIApplication.sharedApplication().delegate as! AppDelegate).managedObjectContext!
	        self.store = MSCoreDataStore(managedObjectContext: managedObjectContext)
	        client.syncContext = MSSyncContext(delegate: nil, dataSource: self.store, callback: nil)
	

	這會建立一個本機存放區，其採用 Mobile Apps SDK 中提供的 `MSCoreDataStore` 介面。您可以實作 `MSSyncContextDataSource` 通訊協定，改為提供不同的本機存放區。
	
	此外，`MSSyncContext` 的第一個參數用來指定衝突處理常式。由於我們已傳遞 `nil`，所以我們會取得預設衝突處理常式，但該處理常式在任何衝突時都會失敗。
	
3. 現在，讓我們執行實際的同步處理作業，並從遠端後端服務取得資料。

	**Objective-C**：
	
	`syncData` 先推送新的變更，然後呼叫 `pullData` 從遠端後端取得資料。然而，`pullData` 方法會取得符合查詢的新資料︰
	
	
	        -(void)syncData:(QSCompletionBlock)completion
	        {
	            // push all changes in the sync context, then pull new data
	            [self.client.syncContext pushWithCompletion:^(NSError *error) {
	                [self logErrorIfNotNil:error];
	                [self pullData:completion];
	            }];
	        }
	
	        -(void)pullData:(QSCompletionBlock)completion
	        {
	            MSQuery *query = [self.syncTable query];
	
	            // Pulls data from the remote server into the local table.
	            // We're pulling all items and filtering in the view
	            // query ID is used for incremental sync
	            [self.syncTable pullWithQuery:query queryId:@"allTodoItems" completion:^(NSError *error) {
	                [self logErrorIfNotNil:error];
	
	                // Let the caller know that we have finished
	                if (completion != nil) {
	                    dispatch_async(dispatch_get_main_queue(), completion);
	                }
	            }];
	        }
        
        
      **Swift**：
        
        
		func onRefresh(sender: UIRefreshControl!) {
		    UIApplication.sharedApplication().networkActivityIndicatorVisible = true
		    
		    self.table!.pullWithQuery(self.table?.query(), queryId: "AllRecords") {
		        (error) -> Void in
		        
		        UIApplication.sharedApplication().networkActivityIndicatorVisible = false
		        
		        if error != nil {
		            // A real application would handle various errors like network conditions,
		            // server conflicts, etc via the MSSyncContextDelegate
		            print("Error: (error!.description)")
		            
		            // We will just discard our changes and keep the servers copy for simplicity
		            if let opErrors = error!.userInfo[MSErrorPushResultKey] as? Array<MSTableOperationError> {
		                for opError in opErrors {
		                    print("Attempted operation to item (opError.itemId)")
		                    if (opError.operation == .Insert || opError.operation == .Delete) {
		                        print("Insert/Delete, failed discarding changes")
		                        opError.cancelOperationAndDiscardItemWithCompletion(nil)
		                    } else {
		                        print("Update failed, reverting to server's copy")
		                        opError.cancelOperationAndUpdateItem(opError.serverItem!, completion: nil)
		                    }
		                }
		            }
		        }
		        self.refreshControl?.endRefreshing()
		    }
		} 
	
	
	在 OBJECTIVE-C 版本中，於 `syncData`，我們會先在同步處理內容上呼叫 `pushWithCompletion`。此方法是 `MSSyncContext` 的成員 (而非同步處理資料表本身)，因為它會將變更推送至所有資料表。只有以某種方式在本機上修改過的記錄 (透過 CUD 作業)，才會傳送至伺服器。接著會呼叫 `pullData` 協助程式，該程式會呼叫 `MSSyncTable.pullWithQuery` 來取出遠端資料並存放在本機資料庫中。
	
	在 Swift 版本中，則沒有呼叫 `pushWithCompletion`。這是因為推送作業並非絕對必要。如果同步處理內容中正在進行推送作業的資料表有任何變更擱置，則提取一律會先發出推送。不過，如果您有一個以上的同步處理資料表，最好能明確呼叫推送，以確保所有的相關資料表都能一致。
	
	在 Objective-C 與 Swift 版本中，`pullWithQuery` 方法可讓您指定查詢，以篩選您想要取出的記錄。在此範例中，查詢只會取出遠端 `TodoItem` 資料表中的所有記錄。
	
	`pullWithQuery` 的第二個參數是用於*增量同步處理* 的查詢識別碼。增量同步處理會使用記錄的 `UpdatedAt` 時間戳記 (在本機存放區中稱為 `updatedAt`)，僅取出自上次同步處理後修改的記錄。 對您應用程式中的每個邏輯查詢而言，查詢識別碼應該是唯一的描述性字串。若選擇不要增量同步處理，請傳遞 `nil` 做為查詢識別碼。請注意這可能是潛在效率不佳，因為它會取出每項提取作業的所有記錄。

5. 當我們修改或新增資料時、使用者執行重新整理動作及啟動時，OBJECTIVE-C 應用程式就會同步處理。當使用者執行重新整理動作及啟動時，Swift 應用程式就會同步處理。

因為每當資料修改 (Objective-C) 或應用程式啟動 (Objective-C & Swift) 時應用程式就會同步處理，故應用程式假設使用者在線上。在另一節中，我們將會更新應用程式，讓使用者即使是離線狀態也能進行編輯。

## <a name="review-core-data"></a>檢閱核心資料模型

在使用「核心資料離線」存放區時，您需要在資料模型中定義特定資料表和欄位。範例應用程式已經包含具有正確格式的資料模型。本這一節中，我們將逐步介紹這些資料表以及其使用方式。

- 開啟 **QSDataModel.xcdatamodeld**。已定義四個資料表--其中三個由 SDK 使用，而一個適用於 todo 項目本身：
      * MS\_TableOperations：用於追蹤需要與伺服器同步的項目
      * MS\_TableOperationErrors：用於追蹤在離線同步處理期間發生的任何錯誤
      * MS\_TableConfig：用於追蹤所有提取作業的最後一次同步處理作業的上次更新時間
      * TodoItem：用來儲存 todo 項目。系統資料行 **createdAt**、**updatedAt** 和 **version** 為可選擇性使用的系統屬性。

>[AZURE.NOTE] Azure Mobile Apps SDK 會保留開頭為「**``**」的資料行名稱。您不得在系統資料行以外的任何項目上使用此前置詞，否則會在使用遠端後端時修改您的資料行名稱。

- 使用離線同步功能時，您必須先定義系統資料表，如下所示。

    ### 系統資料表

    **MS\_TableOperations**

    ![][defining-core-data-tableoperations-entity]

    | 屬性 | 類型 |
    |----------- |   ------    |
    | id | 整數 64 |
    | itemId | String |
    | properties | 二進位資料 |
    | 資料表 | String |
    | tableKind | 整數 16 |

    <br>**MS\_TableOperationErrors**

    ![][defining-core-data-tableoperationerrors-entity]

    | 屬性 | 類型 |
    |----------- |   ------    |
    | id | String |
    | operationId | 整數 64 |
    | properties | 二進位資料 |
    | tableKind | 整數 16 |

    <br>**MS\_TableConfig**

    ![][defining-core-data-tableconfig-entity]

    | 屬性 | 類型 |
    |----------- |   ------    |
    | id | String |
    | 索引鍵 | String |
    | keyType | 整數 64 |
    | 資料表 | String |
    | value | String |

    ### 資料表

    **TodoItem**

    | 屬性 | 類型 | 注意 |
    |-----------   |  ------ | -------------------------------------------------------|
    | id | 字串 (標示為必要) | 遠端存放區中的主索引鍵 |
    | 完成 | Boolean | todo 項目欄位 |
    | 文字 | String | todo 項目欄位 |
    | 建立時間 | 日期 | (選擇性) 對應至 createdAt 系統屬性 |
    | 更新時間 | 日期 | (選擇性) 對應至 updatedAt 系統屬性 |
    | 版本 | String | (選擇性步驟) 用來偵測衝突，對應至版本 |


## <a name="setup-sync"></a>變更應用程式的同步處理行為

在本節中，您將修改應用程式，使其不會在應用程式啟動時，或插入及更新項目時同步處理，但只會在執行重新整理動作按鈕時同步處理。

**Objective-C**：

1. 在 **QSTodoListViewController.m** 中，變更 **viewDidLoad** 方法以移除在方法結尾的 `[self refresh]` 呼叫。現在，資料不會在應用程式啟動時與伺服器進行同步處理，但是成為本機存放區的內容。

2. 在 **QSTodoService.m** 中，修改 `addItem` 的定義，使其不會在插入項目後同步處理。移除 `self syncData` 區塊並以下列項目取代：

            if (completion != nil) {
                dispatch_async(dispatch_get_main_queue(), completion);
            }

3. 如上所述修改 `completeItem` 的定義 ；移除 `self syncData` 的區塊並以下列項目取代：

            if (completion != nil) {
                dispatch_async(dispatch_get_main_queue(), completion);
            }

**Swift**：

1. 在 **ToDoTableViewController.swift** 的 `viewDidLoad` 中，註解掉這兩行以停止在應用程式啟動時同步處理。在本文撰寫期間，當某人新增或完成項目時，Swift Todo 應用程式不會更新，只有在應用程式啟動時才會更新。

		self.refreshControl?.beginRefreshing()
		self.onRefresh(self.refreshControl)


## <a name="test-app"></a>測試應用程式

在本節中，將連接至無效的 URL，以模擬離線情況。當您新增資料項目時，這些項目會存放在本機核心資料存放區，但不會同步到行動後端。

1. 將 **QSTodoService.m** 中的行動應用程式 URL 變更為無效的 URL，然後再次執行該應用程式：

	**OBJECTIVE-C** 在 QSTodoService.m 中︰
	
        	self.client = [MSClient clientWithApplicationURLString:@"https://sitename.azurewebsites.net.fail"];
	
	**Swift** 在 ToDoTableViewController.swift 中︰

		let client = MSClient(applicationURLString: "https://sitename.azurewebsites.net.fail")

2. 新增一些新的 todo 項目。結束模擬器 (或強制關閉應用程式)，然後重新啟動。確認您的變更已保存下來。

3. 檢視遠端 TodoItem 資料表的內容：

    + 若為 Node.js 後端，請移至 [Azure 入口網站](https://portal.azure.com/)，在您的行動應用程式後端中按一下 [**簡單資料表**] > [**TodoItem**]，檢視 `TodoItem` 資料表的內容。
   	+ 若為 .NET 後端，請使用 SQL 工具 (例如 SQL Server Management Studio) 或 REST 用戶端 (例如 Fiddler 或 Postman) 檢視資料表內容。

    請確認新項目*尚未* 同步處理到伺服器：

4. 請將 **QSTodoService.m** 中的 URL 變更回正確的 URL，然後重新執行應用程式。將項目清單往下拉，執行重新整理動作。您會看到進度微調按鈕。

5. 再次檢視 TodoItem 資料。新的和變更的 TodoItems 現在應該會出現。

## 摘要

為了支援離線同步處理功能，我們使用了 `MSSyncTable` 介面，並對本機存放區初始化 `MSClient.syncContext`。在此案例中，本機存放區是以核心資料為基礎的資料庫。

使用核心資料本機存放區時，您必須使用[正確的系統屬性](#review-core-data)定義數個資料表。

正常情況下，在 Azure Mobile Apps 的 CRUD 作業執行時，應用程式會如同仍處於連線狀態，但所有的作業都會對本機存放區執行。

當我們要同步處理本機存放區與伺服器時，我們使用 `MSSyncTable.pullWithQuery` 方法。


## 其他資源

* [Azure Mobile Apps 中的離線資料同步處理]

* [雲端報導︰Azure Mobile Services 中的離線同步處理] \(注意︰影片位於 Mobile Services 上，但離線同步處理的運作方式類似在 Azure Mobile Apps 中的方式)

<!-- URLs. -->


[建立 iOS 應用程式]: ../app-service-mobile-ios-get-started.md
[Azure Mobile Apps 中的離線資料同步處理]: ../app-service-mobile-offline-data-sync.md
[Azure Mobile Apps中的離線資料同步]: ../app-service-mobile-offline-data-sync.md

[defining-core-data-tableoperationerrors-entity]: ./media/app-service-mobile-ios-get-started-offline-data/defining-core-data-tableoperationerrors-entity.png
[defining-core-data-tableoperations-entity]: ./media/app-service-mobile-ios-get-started-offline-data/defining-core-data-tableoperations-entity.png
[defining-core-data-tableconfig-entity]: ./media/app-service-mobile-ios-get-started-offline-data/defining-core-data-tableconfig-entity.png
[defining-core-data-todoitem-entity]: ./media/app-service-mobile-ios-get-started-offline-data/defining-core-data-todoitem-entity.png

[雲端報導︰Azure Mobile Services 中的離線同步處理]: http://channel9.msdn.com/Shows/Cloud+Cover/Episode-155-Offline-Storage-with-Donna-Malayeri
[Azure Friday: Offline-enabled apps in Azure Mobile Services]: http://azure.microsoft.com/documentation/videos/azure-mobile-services-offline-enabled-apps-with-donna-malayeri/

<!---HONumber=AcomDC_0810_2016------>