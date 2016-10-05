<properties 
	pageTitle="教學課程：使用 Visual Studio 建立具有複製活動的管線 | Microsoft Azure" 
	description="在本教學課程中，您會使用 Visual Studio，建立具有複製活動的 Azure Data Factory 管線。" 
	services="data-factory" 
	documentationCenter="" 
	authors="spelluru" 
	manager="jhubbard" 
	editor="monicar"/>

<tags 
	ms.service="data-factory" 
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="get-started-article" 
	ms.date="08/01/2016" 
	ms.author="spelluru"/>

# 教學課程：使用 Visual Studio 建立具有複製活動的管線
> [AZURE.SELECTOR]
- [概觀和必要條件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
- [Azure 入口網站](data-factory-copy-activity-tutorial-using-azure-portal.md)
- [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md)
- [PowerShell](data-factory-copy-activity-tutorial-using-powershell.md)
- [REST API](data-factory-copy-activity-tutorial-using-rest-api.md)
- [.NET API](data-factory-copy-activity-tutorial-using-dotnet-api.md)
- [複製精靈](data-factory-copy-data-wizard-tutorial.md)

本教學課程示範如何使用 Visual Studio 建立和監視 Azure Data Factory。Data Factory 中的管線會使用複製活動將資料從 Azure Blob 複製到 Azure SQL Database。

以下是您會在本教學課程中執行的步驟：

1. 建立兩個連結的服務：**AzureStorageLinkedService1** 和 **AzureSqlinkedService1**。

	AzureStorageLinkedService1 連結 Azure 儲存體，而 AzureSqlLinkedService1 連結 Azure SQL 資料庫至 Data Factory：**ADFTutorialDataFactoryVS**。管線的輸入資料位於 Azure Blob 儲存體的 Blob 容器中，輸出資料則儲存在 Azure SQL Database 的資料表中。因此，您可以將這兩個資料存放區以連結服務的形式新增至 Data Factory。
2. 建立兩個資料集：**InputDataset** 和 **OutputDataset**，它們分別代表儲存在資料存放區的輸入/輸出資料。

	針對 InputDataset，您需要指定所含 Blob 具有來源資料的 Blob 容器。針對 OutputDataset，則需要指定可儲存輸出資料的 SQL 資料表。您也會指定其他屬性，例如結構、可用性和原則。
3. 在 ADFTutorialDataFactoryVS 中建立名為 **ADFTutorialPipeline** 的管線。

	管線有一個**複製活動**，會將輸入資料從 Azure Blob 複製到輸出 Azure SQL 資料表。複製活動會在 Azure Data Factory 中執行資料移動。此活動是由全域可用的服務所提供，可以使用安全、可靠及可調整的方式，在各種不同的資料存放區之間複製資料。如需複製活動的詳細資訊，請參閱[資料移動活動](data-factory-data-movement-activities.md)文章。
4. 建立名為 **VSTutorialFactory** 的 Data Factory。部署 Data Factory 和所有 Data Factory 實體 (連結的服務、資料表和管線)。

## 必要條件

1. 詳讀[教學課程概觀](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)一文並完成**必要**步驟。
2. 您必須是 **Azure 訂用帳戶的系統管理員**才能發佈 Data Factory 實體至 Azure Data Factory。
3. 您必須已在電腦上安裝下列項目：
	- Visual Studio 2013 或 Visual Studio 2015
	- 下載 Azure SDK for Visual Studio 2013 或 Visual Studio 2015。瀏覽至 [Azure 下載頁面](https://azure.microsoft.com/downloads/)，然後按一下 [.NET] 區段中的 [VS 2013] 或 [VS 2015]。
	- 下載適用於 Visual Studio 的最新 Azure Data Factory 外掛程式：[VS 2013](https://visualstudiogallery.msdn.microsoft.com/754d998c-8f92-4aa7-835b-e89c8c954aa5) 或 [VS 2015](https://visualstudiogallery.msdn.microsoft.com/371a4cf9-0093-40fa-b7dd-be3c74f49005)。您也可以執行下列步驟來更新外掛程式：在功能表中按一下 [工具] -> [擴充功能和更新] -> [線上] -> [Visual Studio 組件庫] -> [Microsoft Azure Data Factory Tools for Visual Studio] -> [更新]。

## 建立 Visual Studio 專案 
1. 啟動 **Visual Studio 2013**。按一下 [**檔案**]，指向 [**新增**]，然後按一下 [**專案**]。您應該會看到 [新增專案] 對話方塊。
2. 在 [新增專案] 對話方塊中，選取 **DataFactory** 範本，然後按一下 [空白 Data Factory 專案]。如果您沒有看到 DataFactory 範本，請關閉 Visual Studio、安裝 Azure SDK for Visual Studio 2013，並重新開啟 Visual Studio。

	![[新增專案] 對話方塊](./media/data-factory-copy-activity-tutorial-using-visual-studio/new-project-dialog.png)

3. 輸入專案的**名稱**、**位置**和**方案**的名稱，然後按一下 [確定]。

	![Solution Explorer](./media/data-factory-copy-activity-tutorial-using-visual-studio/solution-explorer.png)

## 建立連結服務
連結服務會將資料存放區或計算服務連結至 Azure Data Factory。如需複製活動支援的所有來源和接收，請參閱[支援的資料存放區](data-factory-data-movement-activities.md##supported-data-stores-and-formats)。如需 Data Factory 支援的計算服務清單，請參閱[計算連結服務](data-factory-compute-linked-services.md)。在本教學課程中，您不會使用任何計算服務。

在此步驟中，您會建立兩個連結服務：**AzureStorageLinkedService1** 和 **AzureSqlLinkedService1**。AzureStorageLinkedService1 連結服務會連結 Azure 儲存體帳戶，而 AzureSqlLinkedService 會將 Azure SQL 資料庫連結至 Data Factory：**ADFTutorialDataFactory**。

### 建立 Azure 儲存體連結服務

4. 以滑鼠右鍵按一下 [方案總管] 中的 [連結服務]，指向 [加入]，然後按一下 [新增項目]。
5. 在 [加入新項目] 對話方塊中，從清單選取 [Azure 儲存體連結服務]，然後按一下 [加入]。

	![新的連結服務](./media/data-factory-copy-activity-tutorial-using-visual-studio/new-linked-service-dialog.png)
 
3. 使用 Azure 儲存體帳戶的名稱及其金鑰來取代 `<accountname>` 和 `<accountkey>`。

	![Azure 儲存體連結服務](./media/data-factory-copy-activity-tutorial-using-visual-studio/azure-storage-linked-service.png)

4. 儲存 **AzureStorageLinkedService1.json** 檔案。

> 如需 JSON 屬性的詳細資訊，請參閱[在 Azure Blob 來回移動資料](data-factory-azure-blob-connector.md#azure-storage-linked-service)。

### 建立 Azure SQL 連結服務。

5. 再次以滑鼠右鍵按一下 [方案總管] 中的 [連結服務] 節點，指向 [加入]，然後按一下 [新增項目]。
6. 這次，請選取 [Azure SQL 連結服務]，然後按一下 [加入]。
7. 在 **AzureSqlLinkedService1.json 檔案**中，以您的 Azure SQL Server 名稱、資料庫名稱、使用者帳戶名稱和密碼取代 `<servername>`、`<databasename>`、`<username@servername>` 和 `<password>`。
8.  儲存 **AzureSqlLinkedService1.json** 檔案。

> [AZURE.NOTE]
如需 JSON 屬性的詳細資訊，請參閱[在 Azure SQL Database 來回移動資料](data-factory-azure-sql-connector.md#azure-sql-linked-service-properties)。

## 建立資料集
在上一個步驟中，您已建立連結服務 **AzureStorageLinkedService1** 和 **AzureSqlLinkedService1**，將 Azure 儲存體帳戶和 Azure SQL 資料庫連結至 Data Factory：**ADFTutorialDataFactory**。在此步驟中，您會定義兩個資料集 (**InputDataset** 和 **OutputDataset**)，它們分別代表 AzureStorageLinkedService1 和 AzureSqlLinkedService1 所參照資料存放區中所儲存的輸入/輸出資料。針對 InputDataset，您需要指定所含 Blob 具有來源資料的 Blob 容器。針對 OutputDataset，則需要指定可儲存輸出資料的 SQL 資料表。

### 建立輸入資料集
在此步驟中，您將在 **AzureStorageLinkedService1** 連結服務所代表的 Azure 儲存體中，建立指向 Blob 容器的 **InputDataset** 資料集。資料表是矩形的資料集，而且是目前唯一受支援的資料集類型。

9. 以滑鼠右鍵按一下 [方案總管] 中的 [資料表]，指向 [加入]，然後按一下 [新增項目]。
10. 在 [加入新項目] 對話方塊中，選取 [Azure Blob]，然後按一下 [加入]。
10. 將 JSON 文字取代為下列文字並儲存 **AzureBlobLocation1.json** 檔案。

		{
		  "name": "InputDataset",
		  "properties": {
		    "structure": [
		      {
		        "name": "FirstName",
		        "type": "String"
		      },
		      {
		        "name": "LastName",
		        "type": "String"
		      }
		    ],
		    "type": "AzureBlob",
		    "linkedServiceName": "AzureStorageLinkedService1",
		    "typeProperties": {
		      "folderPath": "adftutorial/",
		      "format": {
		        "type": "TextFormat",
		        "columnDelimiter": ","
		      }
		    },
		    "external": true,
		    "availability": {
		      "frequency": "Hour",
		      "interval": 1
		    }
		  }
		}

     請注意下列幾點：
	
	- 資料集 **type** 設為 **AzureBlob**。
	- **linkedServiceName** 設為 **AzureStorageLinkedService**。您已在步驟 2 中建立此連結服務。
	- **folderPath** 設為 **adftutorial** 容器。您也可以使用 **fileName** 屬性指定資料夾內 Blob 的名稱。由於您未指定 Blob 的名稱，容器中所有 Blob 的資料都會被視為輸入資料。
	- 格式 **type** 設為 **TextFormat**
	- 文字檔中有兩個欄位 (**FirstName** 和 **LastName**)，以逗號字元分隔 (**columnDelimiter**)
	- **availability** 設定為**每小時** (**frequency** 設定為**小時**，**interval** 設定為 **1**)。因此，Data Factory 會每小時都在您指定之 Blob 容器 (**adftutorial**) 的根資料夾中尋找輸入資料。
	
	如果您未指定**輸入**資料集的 **fileName**，則輸入資料夾 (**folderPath**) 中的所有檔案/Blob 都會被視為輸入。如果您在 JSON 中指定 fileName，則只有指定的檔案/Blob 會被視為輸入。
 
	如果您未指定輸出資料表的 **fileName**，**folderPath** 中產生的檔案會依照下列格式命名：Data.&lt;Guid&gt;.txt (範例：Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt)。

	若要根據 **SliceStart** 時間動態設定 **folderPath** 和 **fileName**，請使用 **partitionedBy** 屬性。在下列範例中，folderPath 使用 SliceStart (所處理配量的開始時間) 中的年、月和日，fileName 使用 SliceStart 中的小時。例如，如果配量產生於 2016-09-20T08:00:00，folderName 設定為 wikidatagateway/wikisampledataout/2016/09/20，而 fileName 設定為 08.csv。

			"folderPath": "wikidatagateway/wikisampledataout/{Year}/{Month}/{Day}",
	        "fileName": "{Hour}.csv",
	        "partitionedBy": 
	        [
	        	{ "name": "Year", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyy" } },
	            { "name": "Month", "value": { "type": "DateTime", "date": "SliceStart", "format": "MM" } }, 
	            { "name": "Day", "value": { "type": "DateTime", "date": "SliceStart", "format": "dd" } }, 
	            { "name": "Hour", "value": { "type": "DateTime", "date": "SliceStart", "format": "hh" } } 

> [AZURE.NOTE]
如需 JSON 屬性的詳細資訊，請參閱[在 Azure Blob 來回移動資料](data-factory-azure-blob-connector.md#azure-blob-dataset-type-properties)。

### 建立輸出資料集
在此步驟中，您會建立名為 **OutputDataset** 的輸出資料集。此資料集指向 Azure SQL Database 中 **AzureSqlLinkedService1** 所代表的 SQL 資料表。

11. 再次以滑鼠右鍵按一下 [**方案總管**] 中的 [**資料表**]，指向 [**新增**]，然後按一下 [**新增項目**]。
12. 在 [加入新項目] 對話方塊中，選取 [Azure SQL]，然後按一下 [加入]。
13. 將 JSON 文字取代成下列 JSON 並儲存 **AzureSqlTableLocation1.json** 檔案。

		{
		  "name": "OutputDataset",
		  "properties": {
		    "structure": [
		      {
		        "name": "FirstName",
		        "type": "String"
		      },
		      {
		        "name": "LastName",
		        "type": "String"
		      }
		    ],
		    "type": "AzureSqlTable",
		    "linkedServiceName": "AzureSqlLinkedService1",
		    "typeProperties": {
		      "tableName": "emp"
		    },
		    "availability": {
		      "frequency": "Hour",
		      "interval": 1
		    }
		  }
		}

     請注意下列幾點：
	
	- 資料集 **type** 設為 **AzureSQLTable**。
	- **linkedServiceName** 設為 **AzureSqlLinkedService** (您已在步驟 2 中建立此連結服務)。
	- **tablename** 設為 **emp**。
	- 資料庫的 emp 資料表中有三個資料行 – **ID**、**FirstName** 和 **LastName**。ID 是識別資料行，所以您只需在此指定 **FirstName** 和 **LastName**。
	- **availability** 設定為**每小時**，且 (**frequency** 設定為**小時**，**interval** 設定為 **1**)。Data Factory 服務會每隔一小時在 Azure SQL Database 的 **emp** 資料表中產生輸出資料配量。

> [AZURE.NOTE]
如需 JSON 屬性的詳細資訊，請參閱[在 Azure SQL Database 來回移動資料](data-factory-azure-sql-connector.md#azure-sql-linked-service-properties)。

## 建立管線 
您到目前為止已建立輸入/輸出連結服務和資料表。現在，您會使用 [複製活動] 建立管線，將資料從 Azure Blob 複製到 Azure SQL 資料庫。


1. 以滑鼠右鍵按一下 [方案總管] 中的 [管線]，指向 [加入]，然後按一下 [新增項目]。
15. 選取 [加入新項目] 對話方塊中的 [複製資料管線]，並按一下 [加入]。
16. 將 JSON 取代為下列 JSON 並儲存 **CopyActivity1.json** 檔案。
			
		{
		  "name": "ADFTutorialPipeline",
		  "properties": {
		    "description": "Copy data from a blob to Azure SQL table",
		    "activities": [
		      {
		        "name": "CopyFromBlobToSQL",
		        "type": "Copy",
		        "inputs": [
		          {
		            "name": "InputDataset"
		          }
		        ],
		        "outputs": [
		          {
		            "name": "OutputDataset"
		          }
		        ],
		        "typeProperties": {
		          "source": {
		            "type": "BlobSource"
		          },
		          "sink": {
		            "type": "SqlSink",
		            "writeBatchSize": 10000,
		            "writeBatchTimeout": "60:00:00"
		          }
		        },
		        "Policy": {
		          "concurrency": 1,
		          "executionPriorityOrder": "NewestFirst",
		          "style": "StartOfInterval",
		          "retry": 0,
		          "timeout": "01:00:00"
		        }
		      }
		    ],
		    "start": "2015-07-12T00:00:00Z",
		    "end": "2015-07-13T00:00:00Z",
		    "isPaused": false
		  }
		}

	請注意下列幾點：

	- 在活動區段中，只會有一個 **type** 設為 **Copy** 的活動。
	- 活動的輸入設定為 **InputDataset**，活動的輸出則設定為 **OutputDataset**。
	- 在 **transformation** 區段中，來源類型指定為 **BlobSource**，接收類型指定為 **SqlSink**。

	將 **start** 屬性的值替換為目前日期，並將 **end** 值替換為隔天的日期。在日期時間中，您只指定日期部分，並略過時間部分。例如，"2016-02-03"，這相當於 "2016-02-03T00:00:00Z"
	
	開始和結束日期時間都必須是 [ISO 格式](http://en.wikipedia.org/wiki/ISO_8601)。例如：2016-10-14T16:32:41Z。**end** 時間為選擇性項目，但在本教學課程中會用到。
	
	如果您未指定 **end** 屬性的值，則會以「**start + 48 小時**」計算。若要無限期地執行管線，請指定 **9999-09-09** 做為 **end** 屬性的值。
	
	在上述範例中，由於每小時即產生一個資料配量，共會有 24 個資料配量。

## 發佈/部署 Data Factory 實體
在此步驟中，您會發佈稍早建立的 Data Factory 實體 (連結的服務、資料集和管線)。您也可以指定要建立來保存這些實體的新 Data Factory 名稱。

18. 在 [方案總管] 中，以滑鼠右鍵按一下專案，再按一下 [發佈]。
19. 如果您看到 [登入您的 Microsoft 帳戶] 對話方塊，請輸入具有 Azure 訂用帳戶的帳戶認證，然後按一下 [登入]。
20. 您應該會看到下列對話方塊：

	![發佈對話方塊](./media/data-factory-copy-activity-tutorial-using-visual-studio/publish.png)
21. 在 [設定 Data Factory] 頁面中，執行下列步驟：
	1. 選取 [建立新的 Data Factory] 選項。
	2. 針對 [名稱] 輸入 **VSTutorialFactory**。
	
		> [AZURE.IMPORTANT]  
		Azure Data Factory 的名稱在全域必須是唯一的。如果您在發佈時收到與 Data Factory 名稱有關的錯誤，請變更 Data Factory 的名稱 (例如 yournameVSTutorialFactory) 並嘗試再發佈一次。請參閱 [Data Factory - 命名規則](data-factory-naming-rules.md)主題，以了解 Data Factory 成品的命名規則。
	3. 針對 [訂用帳戶] 欄位選取您的 Azure 訂用帳戶。
	 
		> [AZURE.IMPORTANT] 如果看不到任何訂用帳戶，請確定您是使用訂用帳戶的管理員或共同管理員的帳戶進行登入。
	4. 針對要建立的 Data Factory 選取 [資源群組]。
	5. 選取 Data Factory 的 [區域]。下拉式清單中只會顯示 Data Factory 服務支援的區域。
	6. 按 [下一步]，切換至 [發佈項目] 頁面。
	
		![設定 Data Factory 頁面](media/data-factory-copy-activity-tutorial-using-visual-studio/configure-data-factory-page.png)
23. 在 [發佈項目] 頁面上，確認所有 Data Factory 實體都已選取，並按 [下一步] 切換至 [摘要] 頁面。
	
	![發佈項目頁面](media/data-factory-copy-activity-tutorial-using-visual-studio/publish-items-page.png)
24. 檢閱摘要，然後按 [下一步] 開始部署程序，並檢視 [部署狀態]。

	![發佈摘要頁面](media/data-factory-copy-activity-tutorial-using-visual-studio/publish-summary-page.png)
25. 在 [部署狀態] 頁面上，您應該會看到部署程序的狀態。部署完成後按一下 [完成]。![部署狀態頁面](media/data-factory-copy-activity-tutorial-using-visual-studio/deployment-status.png) 請注意下列幾點︰

- 如果您收到錯誤：「此訂用帳戶未註冊為使用命名空間 Microsoft.DataFactory」，請執行下列其中一項，然後嘗試再次發佈︰

	- 在 Azure PowerShell 中，執行下列命令以註冊 Data Factory 提供者。
		
			Register-AzureRmResourceProvider -ProviderNamespace Microsoft.DataFactory
	
		您可以執行下列命令來確認已註冊 Data Factory 提供者。
	
			Get-AzureRmResourceProvider
	- 使用 Azure 訂用帳戶登入 [Azure 入口網站](https://portal.azure.com)並瀏覽至 [Data Factory] 刀鋒視窗 (或) 在 Azure 入口網站中建立 Data Factory。此動作會自動為您註冊提供者。
- 	Data Factory 的名稱未來可能會註冊為 DNS 名稱，因此會變成公開可見的名稱。

> [AZURE.IMPORTANT] 若要建立 Data Factory 執行個體，您必須是 Azure 訂用帳戶的管理員/共同管理員

## 摘要
在本教學課程中，您已建立要將資料從 Azure Blob 複製到 Azure SQL 資料庫的 Azure Data Factory。您已使用 Visual Studio 建立 Data Factory、連結服務、資料集和管線。以下是您在本教學課程中執行的高階步驟：

1.	建立 Azure **Data Factory**。
2.	建立**連結服務**：
	1. **Azure 儲存體**連結服務可連結保留輸入資料的 Azure 儲存體帳戶。
	2. **Azure SQL** 連結服務可連結保留輸出資料的 Azure SQL Database。
3.	建立可描述管線輸入資料和輸出資料的**資料集**。
4.	建立具有**複製活動**的**管線**，以 **BlobSource** 做為來源並以 **SqlSink** 做為接收器。


## 使用伺服器總管檢視 Data Factory

1. 在 **Visual Studio** 中，按一下功能表上的 [檢視]，然後按一下 [伺服器總管]。
2. 在 [伺服器總管] 視窗中，依序展開 **Azure** 和 **Data Factory**。如果您看到 [登入 Visual Studio]，請輸入和 Azure 訂用帳戶相關聯的**帳戶**，然後按一下 [繼續]。輸入**密碼**，然後按一下 [登入]。Visual Studio 會嘗試取得訂用帳戶中所有 Azure Data Factory 的相關資訊。您會在 [Data Factory 工作清單] 視窗中看到這項作業的狀態。![Server Explorer](./media/data-factory-copy-activity-tutorial-using-visual-studio/server-explorer.png)
3. 您可以在 Data Factory 上按一下滑鼠右鍵，並選取 [將 Data Factory 匯出至新的專案]，以便根據現有的 Data Factory 建立 Visual Studio 專案。![匯出 Data Factory 至 VS 專案](./media/data-factory-copy-activity-tutorial-using-visual-studio/export-data-factory-menu.png)

## 更新 Visual studio 的 Data Factory 工具
若要更新 Visual Studio 的 Azure Data Factory 工具，請執行下列步驟：

1. 按一下功能表上的 [工具]，然後選取 [擴充功能和更新]。
2. 選取左窗格中的 [更新]，然後選取 [Visual Studio 組件庫]。
4. 選取 [Visual Studio 的 Azure Data Factory 工具] 並按一下 [更新]。如果您看不到此項目，代表您已經有最新版本的工具。

如需如何使用 Azure 入口網站來監視您在本教學課程中建立的管線和資料集的指示，請參閱[監視資料集和管線](data-factory-copy-activity-tutorial-using-azure-portal.md#monitor-pipeline)。

## 另請參閱
| 主題 | 說明 |
| :---- | :---- |
| [資料移動活動](data-factory-data-movement-activities.md) | 本文提供您在本教學課程中使用的複製活動詳細資訊。 |
| [排程和執行](data-factory-scheduling-and-execution.md) | 本文說明 Azure Data Factory 應用程式模型的排程和執行層面。 |
| [管線](data-factory-create-pipelines.md) | 本文協助您了解 Azure Data Factory 中的管線和活動。 |
| [資料集](data-factory-create-datasets.md) | 本文協助您了解 Azure Data Factory 中的資料集。
| [使用監視應用程式來監視和管理管線](data-factory-monitor-manage-app.md) | 本文說明如何使用監視及管理應用程式，來監視、管理管線及進行偵錯。 

<!---HONumber=AcomDC_0928_2016-->