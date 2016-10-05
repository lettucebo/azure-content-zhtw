<properties 
	pageTitle="從 Azure SQL 資料倉儲來回移動資料 | Microsoft Azure" 
	description="了解如何使用 Azure Data Factory 從 Azure SQL 資料倉儲來回移動資料" 
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
	ms.topic="article" 
	ms.date="09/20/2016" 
	ms.author="spelluru"/>

# 使用 Azure Data Factory 從 Azure SQL 資料倉儲來回移動資料

本文將概述如何使用 Azure Data Factory 中的複製活動，在 Azure SQL 資料倉儲和另一個資料存放區之間往返移動資料。

您可以指定是否要在將資料載入 Azure SQL 資料倉儲時使用 PolyBase。我們建議您在將資料載入 Azure SQL 資料倉儲時使用 PolyBase 來達到最佳效能。如需詳細資訊，請參閱[使用 PolyBase 將資料載入 Azure SQL 資料倉儲](data-factory-azure-sql-data-warehouse-connector.md#use-polybase-to-load-data-into-azure-sql-data-warehouse)。


## 複製資料精靈
要建立將資料複製到 Azure SQL 資料倉儲，或複製 Azure SQL 資料倉儲資料的管線，最簡單的方法是使用複製資料精靈。如需使用複製資料精靈建立管線的快速逐步解說，請參閱[教學課程︰使用複製精靈建立管線](data-factory-copy-data-wizard-tutorial.md)。

以下範例提供可用來使用 [Azure 入口網站](data-factory-copy-activity-tutorial-using-azure-portal.md)或 [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) 或 [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md) 建立管線的範例 JSON 定義。它們會示範如何將資料複製到 Azure SQL 資料倉儲和 Azure Blob 儲存體，以及複製其中的資料。不過，您可以在 Azure Data Factory 中使用複製活動，從任何來源**直接**將資料複製到[這裡](data-factory-data-movement-activities.md#supported-data-stores)所說的任何接收器。


> [AZURE.NOTE] 
如需 Azure Data Factory 服務的概觀，請參閱 [Azure Data Factory 簡介](data-factory-introduction.md)。
> 
> 這篇文章提供 JSON 範例，但沒有提供如何建立資料處理站的逐步指示。請參閱[教學課程：將資料從 Azure Blob 複製到 Azure SQL Database](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)，以取得快速逐步解說，其中包含如何使用 Azure Data Factory 中複製活動的逐步指示。


## 範例：將資料從 Azure SQL 資料倉儲複製到 Azure Blob

此範例會定義下列 Data Factory 實體：

1. [AzureSqlDW](#azure-sql-data-warehouse-linked-service-properties) 類型的連結服務。
2. [AzureStorage](data-factory-azure-blob-connector.md#azure-storage-linked-service-properties) 類型的連結服務。
3. [AzureSqlDWTable](#azure-sql-data-warehouse-dataset-type-properties) 類型的輸入[資料集](data-factory-create-datasets.md)。
4. [AzureBlob](data-factory-azure-blob-connector.md#azure-blob-dataset-type-properties) 類型的輸出[資料集](data-factory-create-datasets.md)。
4. 具有使用 [SqlDWSource](#azure-sql-data-warehouse-copy-activity-type-properties) 和 [BlobSink](data-factory-azure-blob-connector.md#azure-blob-copy-activity-type-properties) 之複製活動的[管線](data-factory-create-pipelines.md)。

此範例會每小時將時間序列 (每小時、每日等等) 資料從 Azure SQL 資料倉儲資料庫中的資料表複製到 Blob。範例後面的各節會說明這些範例中使用的 JSON 屬性。

**Azure SQL 資料倉儲連結服務：**

	{
	  "name": "AzureSqlDWLinkedService",
	  "properties": {
	    "type": "AzureSqlDW",
	    "typeProperties": {
	      "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
	    }
	  }
	}

**Azure Blob 儲存體連結服務：**

	{
	  "name": "StorageLinkedService",
	  "properties": {
	    "type": "AzureStorage",
	    "typeProperties": {
	      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
	    }
	  }
	}

**Azure SQL 資料倉儲輸入資料集：**

此範例假設您已在 SQL Azure 資料倉儲中建立資料表 "MyTable"，其中包含時間序列資料的資料行 (名稱為 "timestampcolumn")。
 
設定 “external”: ”true” 會通知 Data Factory 服務：這是 Data Factory 外部的資料集而且不是由 Data Factory 中的活動所產生。

	{
	  "name": "AzureSqlDWInput",
	  "properties": {
	    "type": "AzureSqlDWTable",
	    "linkedServiceName": "AzureSqlDWLinkedService",
	    "typeProperties": {
	      "tableName": "MyTable"
	    },
	    "external": true,
	    "availability": {
	      "frequency": "Hour",
	      "interval": 1
	    },
	    "policy": {
	      "externalData": {
	        "retryInterval": "00:01:00",
	        "retryTimeout": "00:10:00",
	        "maximumRetry": 3
	      }
	    }
	  }
	}

**Azure Blob 輸出資料集：**

資料會每小時寫入至新的 Blob (頻率：小時，間隔：1)。根據正在處理之配量的開始時間，以動態方式評估 Blob 的資料夾路徑。資料夾路徑會使用開始時間的年、月、日和小時部分。

	{
	  "name": "AzureBlobOutput",
	  "properties": {
	    "type": "AzureBlob",
	    "linkedServiceName": "StorageLinkedService",
	    "typeProperties": {
	      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
	      "partitionedBy": [
	        {
	          "name": "Year",
	          "value": {
	            "type": "DateTime",
	            "date": "SliceStart",
	            "format": "yyyy"
	          }
	        },
	        {
	          "name": "Month",
	          "value": {
	            "type": "DateTime",
	            "date": "SliceStart",
	            "format": "MM"
	          }
	        },
	        {
	          "name": "Day",
	          "value": {
	            "type": "DateTime",
	            "date": "SliceStart",
	            "format": "dd"
	          }
	        },
	        {
	          "name": "Hour",
	          "value": {
	            "type": "DateTime",
	            "date": "SliceStart",
	            "format": "HH"
	          }
	        }
	      ],
	      "format": {
	        "type": "TextFormat",
	        "columnDelimiter": "\t",
	        "rowDelimiter": "\n"
	      }
	    },
	    "availability": {
	      "frequency": "Hour",
	      "interval": 1
	    }
	  }
	}


**具有複製活動的管線：**

此管線包含複製活動，該活動已設定為使用輸入和輸出資料集並排定為每小時執行。在管線 JSON 定義中，**source** 類型設為 **SqlDWSource**，而 **sink** 類型設為 **BlobSink**。針對 **SqlReaderQuery** 屬性指定的 SQL 查詢會選取過去一小時內要複製的資料。

	{  
	    "name":"SamplePipeline",
	    "properties":{  
	    "start":"2014-06-01T18:00:00",
	    "end":"2014-06-01T19:00:00",
	    "description":"pipeline for copy activity",
	    "activities":[  
	      {
	        "name": "AzureSQLDWtoBlob",
	        "description": "copy activity",
	        "type": "Copy",
	        "inputs": [
	          {
	            "name": "AzureSqlDWInput"
	          }
	        ],
	        "outputs": [
	          {
	            "name": "AzureBlobOutput"
	          }
	        ],
	        "typeProperties": {
	          "source": {
	            "type": "SqlDWSource",
	            "sqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
	          },
	          "sink": {
	            "type": "BlobSink"
	          }
	        },
	       "scheduler": {
	          "frequency": "Hour",
	          "interval": 1
	        },
	        "policy": {
	          "concurrency": 1,
	          "executionPriorityOrder": "OldestFirst",
	          "retry": 0,
	          "timeout": "01:00:00"
	        }
	      }
	     ]
	   }
	}

> [AZURE.NOTE] 在此範例中，已為 SqlDWSource 指定 **sqlReaderQuery**。複製活動會針對 Azure SQL 資料倉儲來源執行這項查詢以取得資料。
>  
> 或者，您可以藉由指定 **sqlReaderStoredProcedureName** 和 **storedProcedureParameters** (如果預存程序接受參數) 來指定預存程序。
>  
> 如果您未指定 sqlReaderQuery 或 sqlReaderStoredProcedureName，就會使用資料集 JSON 的結構區段中定義的資料行來建立一個查詢，以對 Azure SQL 資料倉儲執行 (從 mytable 選取 column1、column2)。如果資料集定義沒有結構，則會從資料表中選取所有資料行。

## 範例：將資料從 Azure Blob 複製到 Azure SQL 資料倉儲

此範例會定義下列 Data Factory 實體：

1.	[AzureSqlDW](#azure-sql-data-warehouse-linked-service-properties) 類型的連結服務。
2.	[AzureStorage](data-factory-azure-blob-connector.md#azure-storage-linked-service-properties) 類型的連結服務。
3.	[AzureBlob](data-factory-azure-blob-connector.md#azure-blob-dataset-type-properties) 類型的輸入[資料集](data-factory-create-datasets.md)。
4.	[AzureSqlDWTable](#azure-sql-data-warehouse-dataset-type-properties) 類型的輸出[資料集](data-factory-create-datasets.md)。
4.	具有使用 [BlobSource](data-factory-azure-blob-connector.md#azure-blob-copy-activity-type-properties) 和 [SqlDWSink](#azure-sql-data-warehouse-copy-activity-type-properties) 之複製活動的[管線](data-factory-create-pipelines.md)。


此範例會每小時將時間序列資料 (每小時、每日等等) 從 Azure Blob 複製到 Azure SQL 資料倉儲資料庫中的資料表。範例後面的各節會說明這些範例中使用的 JSON 屬性。

**Azure SQL 資料倉儲連結服務：**

	{
	  "name": "AzureSqlDWLinkedService",
	  "properties": {
	    "type": "AzureSqlDW",
	    "typeProperties": {
	      "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
	    }
	  }
	}

**Azure Blob 儲存體連結服務：**

	{
	  "name": "StorageLinkedService",
	  "properties": {
	    "type": "AzureStorage",
	    "typeProperties": {
	      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
	    }
	  }
	}

**Azure Blob 輸入資料集：**

每小時從新的 Blob 挑選資料 (頻率：小時，間隔：1)。根據正在處理之配量的開始時間，以動態方式評估 Blob 的資料夾路徑和檔案名稱。資料夾路徑使用開始時間的年、月、日部分，而檔案名稱使用開始時間的小時部分。“external”: “true” 設定會通知 Data Factory 服務：這是 Data Factory 外部的資料表而且不是由 Data Factory 中的活動所產生。

	{
	  "name": "AzureBlobInput",
	  "properties": {
	    "type": "AzureBlob",
	    "linkedServiceName": "StorageLinkedService",
	    "typeProperties": {
	      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}",
	      "fileName": "{Hour}.csv",
	      "partitionedBy": [
	        {
	          "name": "Year",
	          "value": {
	            "type": "DateTime",
	            "date": "SliceStart",
	            "format": "yyyy"
	          }
	        },
	        {
	          "name": "Month",
	          "value": {
	            "type": "DateTime",
	            "date": "SliceStart",
	            "format": "MM"
	          }
	        },
	        {
	          "name": "Day",
	          "value": {
	            "type": "DateTime",
	            "date": "SliceStart",
	            "format": "dd"
	          }
	        },
	        {
	          "name": "Hour",
	          "value": {
	            "type": "DateTime",
	            "date": "SliceStart",
	            "format": "HH"
	          }
	        }
	      ],
	      "format": {
	        "type": "TextFormat",
	        "columnDelimiter": ",",
	        "rowDelimiter": "\n"
	      }
	    },
	    "external": true,
	    "availability": {
	      "frequency": "Hour",
	      "interval": 1
	    },
	    "policy": {
	      "externalData": {
	        "retryInterval": "00:01:00",
	        "retryTimeout": "00:10:00",
	        "maximumRetry": 3
	      }
	    }
	  }
	}

**Azure SQL 資料倉儲輸出資料集：**

此範例會將資料複製到 Azure SQL 資料倉儲中名為 "MyTable" 的資料表。請在Azure SQL 資料倉儲中建立此資料表，其資料行的數目如您預期 Blob CSV 檔案要包含的數目。此資料表會每小時加入新的資料列。

	{
	  "name": "AzureSqlDWOutput",
	  "properties": {
	    "type": "AzureSqlDWTable",
	    "linkedServiceName": "AzureSqlDWLinkedService",
	    "typeProperties": {
	      "tableName": "MyOutputTable"
	    },
	    "availability": {
	      "frequency": "Hour",
	      "interval": 1
	    }
	  }
	}

**具有複製活動的管線**

此管線包含複製活動，該活動已設定為使用輸入和輸出資料集並排定為每小時執行。在管線 JSON 定義中，**source** 類型設為 **BlobSource**，而 **sink** 類型設為 **SqlDWSink**。

	{  
	    "name":"SamplePipeline",
	    "properties":{  
	    "start":"2014-06-01T18:00:00",
	    "end":"2014-06-01T19:00:00",
	    "description":"pipeline with copy activity",
	    "activities":[  
	      {
	        "name": "AzureBlobtoSQLDW",
	        "description": "Copy Activity",
	        "type": "Copy",
	        "inputs": [
	          {
	            "name": "AzureBlobInput"
	          }
	        ],
	        "outputs": [
	          {
	            "name": "AzureSqlDWOutput"
	          }
	        ],
	        "typeProperties": {
	          "source": {
	            "type": "BlobSource",
	            "blobColumnSeparators": ","
	          },
	          "sink": {
	            "type": "SqlDWSink"
	          }
	        },
	       "scheduler": {
	          "frequency": "Hour",
	          "interval": 1
	        },
	        "policy": {
	          "concurrency": 1,
	          "executionPriorityOrder": "OldestFirst",
	          "retry": 0,
	          "timeout": "01:00:00"
	        }
	      }
	      ]
	   }
	}

如需逐步解說，請參閱 Azure SQL 資料倉儲文件中的[使用 Azure Data Factory 載入資料](../sql-data-warehouse/sql-data-warehouse-get-started-load-with-azure-data-factory.md)一文。

## Azure SQL 資料倉儲連結服務屬性

下表提供 Azure SQL 資料倉儲連結服務專屬 JSON 元素的描述。

屬性 | 說明 | 必要
-------- | ----------- | --------
類型 | 類型屬性必須設為：**AzureSqlDW** | 是
**connectionString** | 針對 connectionString 屬性指定連線到 Azure SQL 資料倉儲執行個體所需的資訊。 | 是

> [AZURE.IMPORTANT] 設定 [Azure SQL Database 防火牆](https://msdn.microsoft.com/library/azure/ee621782.aspx#ConnectingFromAzure)和資料庫伺服器，以[允許 Azure 服務存取伺服器](https://msdn.microsoft.com/library/azure/ee621782.aspx#ConnectingFromAzure)。此外，如果您要從 Azure 外部 (包括從具有 Data Factory 閘道器的內部部署資料來源) 將資料複製到 Azure SQL 資料倉儲，則必須為傳送資料到 Azure SQL 資料倉儲的機器設定適當的 IP 位址範圍。

## Azure SQL 資料倉儲資料集類型屬性

如需定義資料集的區段和屬性完整清單，請參閱[建立資料集](data-factory-create-datasets.md)一文。資料集 JSON 的結構、可用性和原則等區段類似於所有的資料集類型 (SQL Azure、Azure Blob、Azure 資料表等)。

每個資料集類型的 typeProperties 區段都不同，可提供資料存放區中資料的位置相關資訊。**AzureSqlDWTable** 類型資料集的 **typeProperties** 區段具有下列屬性。

| 屬性 | 說明 | 必要 |
| -------- | ----------- | -------- |
| tableName | Azure SQL 資料倉儲資料庫中連結服務所參照的資料表名稱。 | 是 |

## Azure SQL 資料倉儲複製活動類型屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[建立管線](data-factory-create-pipelines.md)一文。屬性 (例如名稱、描述、輸入和輸出資料表，以及原則) 適用於所有類型的活動。

> [AZURE.NOTE]
複製活動只會採用一個輸入，而且只產生一個輸出。

另一方面，活動的 typeProperties 區段中可用的屬性會隨著每個活動類型而有所不同。就「複製活動」而言，這些屬性會根據來源和接收器的類型而有所不同。

### SqlDWSource

如果來源類型為 **SqlDWSource**，則 **typeProperties** 區段可使用下列屬性：

| 屬性 | 說明 | 允許的值 | 必要 |
| -------- | ----------- | -------------- | -------- |
| sqlReaderQuery | 使用自訂查詢來讀取資料。 | SQL 查詢字串。例如：select * from MyTable。 | 否 |
| sqlReaderStoredProcedureName | 從來源資料表讀取資料的預存程序名稱。 | 預存程序的名稱。 | 否 |
| storedProcedureParameters | 預存程序的參數。 | 名稱/值組。參數的名稱和大小寫必須符合預存程序參數的名稱和大小寫。 | 否 |

如果已為 SqlDWSource 指定 **sqlReaderQuery**，複製活動會針對 Azure SQL 資料倉儲來源執行這項查詢以取得資料。

或者，您可以藉由指定 **sqlReaderStoredProcedureName** 和 **storedProcedureParameters** (如果預存程序接受參數) 來指定預存程序。

如果您未指定 sqlReaderQuery 或 sqlReaderStoredProcedureName，就會使用資料集 JSON 的結構區段中定義的資料行來建立一個查詢，以對 Azure SQL 資料倉儲執行。範例：`select column1, column2 from mytable`.如果資料集定義沒有結構，則會從資料表中選取所有資料行。

#### SqlDWSource 範例

    "source": {
        "type": "SqlDWSource",
        "sqlReaderStoredProcedureName": "CopyTestSrcStoredProcedureWithParameters",
        "storedProcedureParameters": {
            "stringData": { "value": "str3" },
            "id": { "value": "$$Text.Format('{0:yyyy}', SliceStart)", "type": "Int"}
        }
    }

**預存程序定義：**

	CREATE PROCEDURE CopyTestSrcStoredProcedureWithParameters
	(
		@stringData varchar(20),
		@id int
	)
	AS
	SET NOCOUNT ON;
	BEGIN
	     select *
	     from dbo.UnitTestSrcTable
	     where dbo.UnitTestSrcTable.stringData != stringData
	    and dbo.UnitTestSrcTable.id != id
	END
	GO
 

### SqlDWSink
**SqlDWSink** 支援下列屬性：

| 屬性 | 說明 | 允許的值 | 必要 |
| -------- | ----------- | -------------- | -------- |
| writeBatchSize | 當緩衝區大小達到 writeBatchSize 時，將資料插入 SQL 資料表中 | 整數 (資料列數目) | 否 (預設值：10000) |
| writeBatchTimeout | 在逾時前等待批次插入作業完成的時間。 | 時間範圍<br/><br/> 範例：“00:30:00” (30 分鐘)。 | 否 | 
| sqlWriterCleanupScript | 指定要讓「複製活動」執行的查詢，以便清除特定分割的資料。如需詳細資訊，請參閱[可重複性](#repeatability-during-copy)一節。 | 查詢陳述式。 | 否 |
| allowPolyBase | 指出是否使用 PolyBase (適用的話) 將資料載入 Azure SQL 資料倉儲 (而不是使用 BULKINSERT 機制)。<br/><br/>目前只有 **Azure Blob** 資料集的**格式**會設定為 **TextFormat** 以做為來源資料集。<br/><br/>請參閱[使用 PolyBase 將資料載入 Azure SQL 資料倉儲](#use-polybase-to-load-data-into-azure-sql-data-warehouse)一節中的條件約束和詳細資料。 | True<br/>False (預設值) | 否 |  
| polyBaseSettings | 可以在 **allowPolybase** 屬性設定為 **true** 時指定的一組屬性。 | &nbsp; | 否 |  
| rejectValue | 指定在查詢失敗前可以拒絕的資料列數目或百分比。<br/><br/>在 [CREATE EXTERNAL TABLE (Transact-SQL)](https://msdn.microsoft.com/library/dn935021.aspx) 主題的**引數**一節中，深入了解 PolyBase 的拒絕選項。 | 0 (預設值)、1、2、… | 否 |  
| rejectType | 指定要將 rejectValue 選項指定為常值或百分比。 | 值 (預設值)、百分比 | 否 |   
| rejectSampleValue | 決定在 PolyBase 重新計算已拒絕的資料列百分比之前，所要擷取的資料列數目。 | 1、2、… | 是，如果 **rejectType** 是 **percentage** |  
| useTypeDefault | 指定當 PolyBase 從文字檔擷取資料時，如何處理分隔符號文字檔中的遺漏值。<br/><br/>從 [CREATE EXTERNAL FILE FORMAT (Transact-SQL)](https://msdn.microsoft.com/library/dn935026.aspx)的＜引數＞一節深入了解這個屬性。 | True/False (預設值為 False) | 否 | 


#### SqlDWSink 範例


    "sink": {
        "type": "SqlDWSink",
        "writeBatchSize": 1000000,
        "writeBatchTimeout": "00:05:00"
    }

## 使用 PolyBase 將資料載入 Azure SQL 資料倉儲
使用 **PolyBase** 是以高輸送量將大量資料載入 Azure SQL 資料倉儲的有效方法。使用 PolyBase 而不是預設的 BULKINSERT 機制，即可看到輸送量大幅提升。

將 **allowPolyBase** 屬性設定為 **true** (如下列範例所示)，以便 Azure Data Factory 使用 PolyBase，將資料複製到 Azure SQL 資料倉儲。當您將 allowPolyBase 設定為 true 時，您可以使用 **polyBaseSettings** 屬性群組來指定 PolyBase 特定屬性。如需您可搭配 polyBaseSettings 使用之屬性的詳細資訊，請參閱 [SqlDWSink](#SqlDWSink) 一節。


    "sink": {
        "type": "SqlDWSink",
		"allowPolyBase": true,
		"polyBaseSettings":
		{
			"rejectType": "percentage",
			"rejectValue": 10.0,
			"rejectSampleValue": 100,
			"useTypeDefault": true 
		}

    }

### 使用 PolyBase 直接複製
如果您的來源資料符合本節所述準則，您就可以使用 PolyBase，從來源資料存放區直接複製到 Azure SQL 資料倉儲。否則，您可以使用[使用 PolyBase 分段複製](#staged-copy-using-polybase)。

如果不符合需求，Azure Data Factory 會檢查設定，並自動切換回適用於資料移動的 BULKINSERT 機制。

1.	**來源連結服務**的類型為 **Azure 儲存體**，而且不會設定為使用 SAS (共用存取簽章) 驗證。如需詳細資訊，請參閱 [Azure 儲存體連結服務](data-factory-azure-blob-connector.md#azure-storage-linked-service)。
2. **輸入資料集**的類型為 **Azure Blob**，而類型屬性下方的格式類型為 **OrcFormat** 或 **TextFormat** 並具備下列組態：
	1. **rowDelimiter** 必須是 **\\n**。
	2. **nullValue** 設定為**空字串** ("")。
	3. **encodingName** 設定為 **utf-8**，這是**預設**值，所以不會設定為不同的值。
	4. 未指定 **escapeChar** 和 **quoteChar**。
	5. **Compression** 不是 **BZIP2**。
	 
			"typeProperties": {
				"folderPath": "<blobpath>",
				"format": {
					"type": "TextFormat",     
					"columnDelimiter": "<any delimiter>", 
					"rowDelimiter": "\n",       
					"nullValue": "",           
					"encodingName": "utf-8"    
				},
            	"compression": {  
                	"type": "GZip",  
	                "level": "Optimal"  
    	        }  
			},
3.	管線中複製活動的 **BlobSource** 之下沒有 **skipHeaderLineCount** 設定。
4.	管線中複製活動的 **SqlDWSink** 之下沒有 **sliceIdentifierColumnName** 設定。(PolyBase 保證所有資料都已更新，或在單一執行未更新任何項目。若要達到**重複性**，您可以使用 **sqlWriterCleanupScript**。
5.	目前沒有任何 **columnMapping** 使用於相關聯的複製活動。

### 使用 PolyBase 分段複製
當來源資料不符合上一節介紹的準則時，您可以啟用透過過渡暫存 Azure Blob 儲存體複製資料。在此情況下，Azure Data Factory 會執行資料轉換，以符合 PolyBase 的資料格式需求，然後使用 PolyBase 將資料載入到 SQL 資料倉儲。如需透過暫存 Azure Blob 複製資料通常如何運作的詳細資訊，請參閱[分段複製](data-factory-copy-activity-performance.md#staged-copy)。

> [AZURE.IMPORTANT] 如果您使用 PolyBase 和分段，將資料從內部部署資料存放區複製到 Azure SQL 資料倉儲，請在閘道電腦上安裝 JRE 8 (Java Runtime Environment)，以便用來將來源資料轉換為適當格式。64 位元閘道需要 64 位元 JRE，而 32 位元閘道需要 32 位元 JRE。從 [Java 下載位置](http://go.microsoft.com/fwlink/?LinkId=808605)下載適當的版本。

若要使用此功能，請建立 [Azure 儲存體連結服務](data-factory-azure-blob-connector.md#azure-storage-linked-service)，這是指具有過渡 Blob 儲存體的 Azure 儲存體帳戶，然後針對複製活動指定 **enableStaging** 和 **stagingSettings** 屬性，如下列程式碼所示：

	"activities":[  
	{
		"name": "Sample copy activity from SQL Server to SQL Data Warehouse via PolyBase",
		"type": "Copy",
		"inputs": [{ "name": "OnpremisesSQLServerInput" }],
		"outputs": [{ "name": "AzureSQLDWOutput" }],
		"typeProperties": {
			"source": {
				"type": "SqlSource",
			},
			"sink": {
				"type": "SqlDwSink",
				"allowPolyBase": true
			},
    		"enableStaging": true,
			"stagingSettings": {
				"linkedServiceName": "MyStagingBlob"
			}
		}
	}
	]


### 使用 PolyBase 時的最佳作法

#### 資料列大小限制
Polybase 不支援大於 32 KB的資料列大小。嘗試載入資料列大於 32 KB 的資料表會導致下列錯誤︰

	Type=System.Data.SqlClient.SqlException,Message=107093;Row size exceeds the defined Maximum DMS row size: [35328 bytes] is larger than the limit of [32768 bytes],Source=.Net SqlClient

如果您的來源資料包含大於 32 KB 的資料列，建議您將來源資料表垂直分割成數個小型資料表，而每個資料表的最大資料列大小均不超過限制。然後可以使用 PolyBase 載入較小的資料表而在 Azure SQL 資料倉儲中合併在一起。

#### Azure SQL 資料倉儲中的 tableName
下表提供的範例是關於如何針對各種結構描述和資料表名稱組合，在資料集 JSON 中指定 **tableName** 屬性。

| DB 結構描述 | 資料表名稱 | tableName JSON 屬性 |
| --------- | -----------| ----------------------- | 
| dbo | MyTable | MyTable 或 dbo.MyTable 或 [dbo].[MyTable] |
| dbo1 | MyTable | dbo1.MyTable 或 [dbo1].[MyTable] |
| dbo | My.Table | [My.Table] 或 [dbo].[My.Table] |
| dbo1 | My.Table | [dbo1].[My.Table] |

如果您看到下列錯誤，可能是您為 tableName 屬性指定的值有問題。請參閱資料表，以正確的方式指定 tableName JSON 屬性的值。

	Type=System.Data.SqlClient.SqlException,Message=Invalid object name 'stg.Account_test'.,Source=.Net SqlClient Data Provider

#### 包含預設值的資料行
Data Factory 中的 PolyBase 功能目前只接受與目標資料表中相同的資料行數目。假設您有內含四個資料行的資料表，且其中一個資料行已使用預設值進行定義。輸入資料應該仍會包含四個資料行。提供 3 個資料行的輸入資料集會產生類似下列訊息的錯誤︰

	All columns of the table must be specified in the INSERT BULK statement.

NULL 值是一種特殊形式的預設值。如果資料行可為 null，該資料行的輸入資料 (在 Blob 中) 可以是空的 (不能在輸入資料集中遺漏)。PolyBase 會在 Azure SQL 資料倉儲中為其插入 NULL。


[AZURE.INCLUDE [data-factory-type-repeatability-for-sql-sources](../../includes/data-factory-type-repeatability-for-sql-sources.md)]

[AZURE.INCLUDE [data-factory-structure-for-rectangualr-datasets](../../includes/data-factory-structure-for-rectangualr-datasets.md)]

### Azure SQL 資料倉儲的類型對應

如[資料移動活動](data-factory-data-movement-activities.md)一文所述，複製活動會使用下列 2 個步驟的方法，執行從來源類型轉換成接收類型的自動類型轉換：

1. 從原生來源類型轉換成 .NET 類型
2. 從 .NET 類型轉換成原生接收類型

將資料移到 Azure SQL、SQL Server、Sybase 或從這些位置移動資料時，會使用下列從 SQL 類型到 .NET 類型的對應，以及反向的對應。

此對應與 [ADO.NET 的 SQL Server 資料類型對應相同](https://msdn.microsoft.com/library/cc716729.aspx)。

| SQL Server Database Engine 類型 | .NET Framework 類型 |
| ------------------------------- | ------------------- |
| bigint | Int64 |
| binary | 位元組 |
| bit | Boolean |
| char | String、Char |
| 日期 | DateTime |
| Datetime | DateTime |
| datetime2 | DateTime |
| Datetimeoffset | DateTimeOffset |
| 十進位 | 十進位 |
| FILESTREAM 屬性 (varbinary(max)) | 位元組 |
| Float | 兩倍 |
| image | 位元組 | 
| int | Int32 | 
| money | 十進位 |
| nchar | String、Char |
| ntext | String、Char |
| numeric | 十進位 |
| nvarchar | String、Char |
| real | 單一 |
| rowversion | 位元組 |
| smalldatetime | DateTime |
| smallint | Int16 |
| smallmoney | 十進位 | 
| sql\_variant | 物件 * |
| 文字 | String、Char |
| 分析 | TimeSpan |
| timestamp | 位元組 |
| tinyint | 位元組 |
| uniqueidentifier | Guid |
| varbinary | 位元組 |
| varchar | String、Char |
| xml | Xml |



[AZURE.INCLUDE [data-factory-type-conversion-sample](../../includes/data-factory-type-conversion-sample.md)]

[AZURE.INCLUDE [data-factory-column-mapping](../../includes/data-factory-column-mapping.md)]

## 效能和微調  
請參閱[複製活動的效能及微調指南](data-factory-copy-activity-performance.md)一文，以了解在 Azure Data Factory 中會影響資料移動 (複製活動) 效能的重要因素，以及各種最佳化的方法。

<!---HONumber=AcomDC_0921_2016-->