<properties 
	pageTitle="從 Azure SQL Database 來回移動資料 | Microsoft Azure" 
	description="了解如何使用 Azure Data Factory 從 Azure SQL 資料庫來回移動資料。" 
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

# 使用 Azure Data Factory 從 Azure SQL Database 來回移動資料

本文概述如何使用 Azure Data Factory 中的「複製活動」，將資料從「Azure SQL Database」移到另一個資料存放區，或從另一個資料存放區移到「Azure SQL Database」。本文是根據[資料移動活動](data-factory-data-movement-activities.md)一文，該文呈現使用複製活動移動資料的一般概觀以及支援的資料存放區組合。

## 複製資料精靈
要建立將資料複製到 Azure SQL Database，或複製 Azure BSQL Database 資料的管線，最簡單的方法是使用複製資料精靈。如需使用複製資料精靈建立管線的快速逐步解說，請參閱[教學課程︰使用複製精靈建立管線](data-factory-copy-data-wizard-tutorial.md)。

以下範例提供可用來使用 [Azure 入口網站](data-factory-copy-activity-tutorial-using-azure-portal.md)或 [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) 或 [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md) 建立管線的範例 JSON 定義。它們會示範如何將資料複製到 Azure SQL Database 和 Azure Blob 儲存體，以及複製其中的資料。不過，您可以在 Azure Data Factory 中使用複製活動，從任何來源**直接**將資料複製到[這裡](data-factory-data-movement-activities.md#supported-data-stores)所說的任何接收器。

## 範例：將資料從 Azure SQL Database 複製到 Azure Blob

相同內容會定義下列 Data Factory 實體：

1. [AzureSqlDatabase](#azure-sql-linked-service-properties) 類型的連結服務。
2. [AzureStorage](data-factory-azure-blob-connector.md#azure-storage-linked-service-properties) 類型的連結服務。
3. [AzureSqlTable](#azure-sql-dataset-type-properties) 類型的輸入[資料集](data-factory-create-datasets.md)。
4. [AzureBlob](data-factory-azure-blob-connector.md#azure-blob-dataset-type-properties) 類型的輸出[資料集](data-factory-create-datasets.md)。
4. 具有使用 [SqlSource](#azure-sql-copy-activity-type-properties) 和 [BlobSink](data-factory-azure-blob-connector.md#azure-blob-copy-activity-type-properties) 之複製活動的[管線](data-factory-create-pipelines.md)。

此範例會每小時將時間序列資料 (每小時、每日等等) 從 Azure SQL Database 中的資料表複製到 Blob。範例後面的各節會說明這些範例中使用的 JSON 屬性。

**Azure SQL 連結服務**

	{
	  "name": "AzureSqlLinkedService",
	  "properties": {
	    "type": "AzureSqlDatabase",
	    "typeProperties": {
	      "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
	    }
	  }
	}

如需這個連結的服務所支援屬性的清單，請參閱 [Azure SQL 連結的服務](#azure-sql-linked-service-properties)一文。

**Azure Blob 儲存體連結服務**

	{
	  "name": "StorageLinkedService",
	  "properties": {
	    "type": "AzureStorage",
	    "typeProperties": {
	      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
	    }
	  }
	}

如需這個連結的服務所支援屬性的清單，請參閱 [Azure Blob](data-factory-azure-blob-connector.md#azure-storage-linked-service-properties) 一文。

**Azure SQL 輸入資料集**

此範例假設您已在 SQL Azure 中建立資料表 "MyTable"，其中包含時間序列資料的資料行 (名稱為 "timestampcolumn")。

設定 “external”: ”true” 會通知 Azure Data Factory 服務：這是 Data Factory 外部的資料集而且不是由 Data Factory 中的活動所產生。

	{
	  "name": "AzureSqlInput",
	  "properties": {
	    "type": "AzureSqlTable",
	    "linkedServiceName": "AzureSqlLinkedService",
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

如需這個資料集類型所支援屬性的清單，請參閱 [Azure SQL 資料集型別屬性](#azure-sql-dataset-type-properties)小節。

**Azure Blob 輸出資料集**

資料會每小時寫入至新的 Blob (頻率：小時，間隔：1)。根據正在處理之配量的開始時間，以動態方式評估 Blob 的資料夾路徑。資料夾路徑會使用開始時間的年、月、日和小時部分。

	{
	  "name": "AzureBlobOutput",
	  "properties": {
	    "type": "AzureBlob",
	    "linkedServiceName": "StorageLinkedService",
	    "typeProperties": {
	      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}/",
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

如需這個資料集類型所支援屬性的清單，請參閱 [Azure Blob 資料集型別屬性](data-factory-azure-blob-connector.md#azure-blob-dataset-type-properties)小節。

**具有複製活動的管線**

此管線包含複製活動，該活動已設定為使用輸入和輸出資料集並排定為每小時執行。在管線 JSON 定義中，**source** 類型設為 **SqlSource**，而 **sink** 類型設為 **BlobSink**。針對 **SqlReaderQuery** 屬性指定的 SQL 查詢會選取過去一小時內要複製的資料。

	{  
	    "name":"SamplePipeline",
	    "properties":{  
	    "start":"2014-06-01T18:00:00",
	    "end":"2014-06-01T19:00:00",
	    "description":"pipeline for copy activity",
	    "activities":[  
	      {
	        "name": "AzureSQLtoBlob",
	        "description": "copy activity",
	        "type": "Copy",
	        "inputs": [
	          {
	            "name": "AzureSQLInput"
	          }
	        ],
	        "outputs": [
	          {
	            "name": "AzureBlobOutput"
	          }
	        ],
	        "typeProperties": {
	          "source": {
	            "type": "SqlSource",
	            "SqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
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

在此範例中，已為 SqlSource 指定 **sqlReaderQuery**。複製活動會針對 Azure SQL Database 來源執行這項查詢以取得資料。或者，您可以藉由指定 **sqlReaderStoredProcedureName** 和 **storedProcedureParameters** (如果預存程序接受參數) 來指定預存程序。

如果您未指定 sqlReaderQuery 或 sqlReaderStoredProcedureName，就會使用資料集 JSON 的結構區段中定義的資料行來建立一個查詢，以對 Azure SQL Database 執行。例如：`select column1, column2 from mytable`。如果資料集定義沒有結構，則會從資料表中選取所有資料行。


如需 SqlSource 和 BlobSink 所支援屬性的清單，請參閱 [SQL 來源](#sqlsource)小節和 [BlobSink](data-factory-azure-blob-connector.md#azure-blob-copy-activity-type-properties)。


## 範例：從 Azure Blob 複製資料到 Azure SQL Database

此範例會定義下列 Data Factory 實體：

1.	[AzureSqlDatabase](data-factory-azure-sql-connector.md#azure-sql-linked-service-properties) 類型的連結服務。
2.	[AzureStorage](data-factory-azure-blob-connector.md#azure-storage-linked-service-properties) 類型的連結服務。
3.	[AzureBlob](data-factory-azure-blob-connector.md#azure-blob-dataset-type-properties) 類型的輸入[資料集](data-factory-create-datasets.md)。
4.	[AzureSqlTable](data-factory-azure-sql-connector.md#azure-sql-dataset-type-properties) 類型的輸出[資料集](data-factory-create-datasets.md)。
4.	具有使用 [BlobSource](data-factory-azure-blob-connector.md#azure-blob-copy-activity-type-properties) 和 [SqlSink](data-factory-azure-sql-connector.md#azure-sql-copy-activity-type-properties) 之複製活動的[管線](data-factory-create-pipelines.md)。

此範例會每小時將時間序列資料 (每小時、每日等等) 從 Azure Blob 複製到 Azure SQL Database 中的資料表。範例後面的各節會說明這些範例中使用的 JSON 屬性。


**Azure SQL 連結服務**
	
	{
	  "name": "AzureSqlLinkedService",
	  "properties": {
	    "type": "AzureSqlDatabase",
	    "typeProperties": {
	      "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
	    }
	  }
	}

如需這個連結的服務所支援屬性的清單，請參閱 [Azure SQL 連結的服務](#azure-sql-linked-service-properties)一文。

**Azure Blob 儲存體連結服務**

	{
	  "name": "StorageLinkedService",
	  "properties": {
	    "type": "AzureStorage",
	    "typeProperties": {
	      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
	    }
	  }
	}

如需這個連結的服務所支援屬性的清單，請參閱 [Azure Blob](data-factory-azure-blob-connector.md#azure-storage-linked-service-properties) 一文。

**Azure Blob 輸入資料集**

每小時從新的 Blob 挑選資料 (頻率：小時，間隔：1)。根據正在處理之配量的開始時間，以動態方式評估 Blob 的資料夾路徑和檔案名稱。資料夾路徑使用開始時間的年、月、日部分，而檔案名稱使用開始時間的小時部分。“external”: “true” 設定會通知 Data Factory 服務：這是 Data Factory 外部的資料表而且不是由 Data Factory 中的活動所產生。

	{
	  "name": "AzureBlobInput",
	  "properties": {
	    "type": "AzureBlob",
	    "linkedServiceName": "StorageLinkedService",
	    "typeProperties": {
	      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/",
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

如需這個資料集類型所支援屬性的清單，請參閱 [Azure Blob 資料集型別屬性](data-factory-azure-blob-connector.md#azure-blob-dataset-type-properties)小節。

**Azure SQL 輸出資料集**

此範例會將資料複製到 Azure SQL 中名為 "MyTable" 的資料表。請在Azure SQL 中建立此資料表，其資料行的數目如您預期 Blob CSV 檔案要包含的數目。此資料表會每小時加入新的資料列。

	{
	  "name": "AzureSqlOutput",
	  "properties": {
	    "type": "AzureSqlTable",
	    "linkedServiceName": "AzureSqlLinkedService",
	    "typeProperties": {
	      "tableName": "MyOutputTable"
	    },
	    "availability": {
	      "frequency": "Hour",
	      "interval": 1
	    }
	  }
	}

如需這個資料集類型所支援屬性的清單，請參閱 [Azure SQL 資料集型別屬性](#azure-sql-dataset-type-properties)小節。

**具有複製活動的管線**

此管線包含複製活動，該活動已設定為使用輸入和輸出資料集並排定為每小時執行。在管線 JSON 定義中，**source** 類型設為 **BlobSource**，而 **sink** 類型設為 **SqlSink**。

	{  
	    "name":"SamplePipeline",
	    "properties":{  
	    "start":"2014-06-01T18:00:00",
	    "end":"2014-06-01T19:00:00",
	    "description":"pipeline with copy activity",
	    "activities":[  
	      {
	        "name": "AzureBlobtoSQL",
	        "description": "Copy Activity",
	        "type": "Copy",
	        "inputs": [
	          {
	            "name": "AzureBlobInput"
	          }
	        ],
	        "outputs": [
	          {
	            "name": "AzureSqlOutput"
	          }
	        ],
	        "typeProperties": {
	          "source": {
	            "type": "BlobSource",
	            "blobColumnSeparators": ","
	          },
	          "sink": {
	            "type": "SqlSink"
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

如需 SqlSink 和 BlobSource 所支援屬性的清單，請參閱 [SQL 接收器](#sqlsink)小節和 [BlobSource](data-factory-azure-blob-connector.md#azure-blob-copy-activity-type-properties)。


## Azure SQL 連結服務屬性

下表提供 Azure SQL 連結服務專屬 JSON 元素的描述。

| 屬性 | 說明 | 必要 |
| -------- | ----------- | -------- |
| 類型 | type 屬性必須設為：AzureSqlDatabase | 是 |
| connectionString | 針對 connectionString 屬性指定連接到 Azure SQL Database 執行個體所需的資訊。 | 是 |

> [AZURE.NOTE] 設定 [Azure SQL Database 防火牆](https://msdn.microsoft.com/library/azure/ee621782.aspx#ConnectingFromAzure)和資料庫伺服器，以[允許 Azure 服務存取伺服器](https://msdn.microsoft.com/library/azure/ee621782.aspx#ConnectingFromAzure)。此外，如果您要從 Azure 外部 (包括從具有 Fata Factory 閘道器的內部部署資料來源) 將資料複製到 Azure SQL Database，請為傳送資料到 Azure SQL Database 的機器設定適當的 IP 位址範圍。

## Azure SQL 資料集類型屬性

如需定義資料集的區段和屬性完整清單，請參閱[建立資料集](data-factory-create-datasets.md)一文。資料集 JSON 的結構、可用性和原則等區段類似於所有的資料集類型 (SQL Azure、Azure Blob、Azure 資料表等)。

每個資料集類型的 typeProperties 區段都不同，可提供資料存放區中資料的位置相關資訊。**AzureSqlTable** 類型資料集的 **typeProperties** 區段具有下列屬性：

| 屬性 | 說明 | 必要 |
| -------- | ----------- | -------- |
| tableName | Azure SQL Database 執行個體中連結服務所參照的資料表名稱。 | 是 |

## Azure SQL 複製活動類型屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[建立管線](data-factory-create-pipelines.md)一文。屬性 (例如名稱、描述、輸入和輸出資料表，以及原則) 適用於所有類型的活動。

> [AZURE.NOTE] 複製活動只會採用一個輸入，而且只產生一個輸出。

另一方面，活動的 typeProperties 區段中可用的屬性會隨著每個活動類型而有所不同。就「複製活動」而言，這些屬性會根據來源和接收器的類型而有所不同。

### SqlSource

在複製活動中，如果來源的類型為 **SqlSource**，則 **typeProperties** 區段有下列可用屬性：

| 屬性 | 說明 | 允許的值 | 必要 |
| -------- | ----------- | -------------- | -------- |
| sqlReaderQuery | 使用自訂查詢來讀取資料。 | SQL 查詢字串。範例：`select * from MyTable`. | 否 |
| sqlReaderStoredProcedureName | 從來源資料表讀取資料的預存程序名稱。 | 預存程序的名稱。 | 否 |
| storedProcedureParameters | 預存程序的參數。 | 名稱/值組。參數的名稱和大小寫必須符合預存程序參數的名稱和大小寫。 | 否 |

如果已為 SqlSource 指定 **sqlReaderQuery**，複製活動會針對 Azure SQL Database 來源執行這項查詢以取得資料。或者，您可以藉由指定 **sqlReaderStoredProcedureName** 和 **storedProcedureParameters** (如果預存程序接受參數) 來指定預存程序。

如果您未指定 sqlReaderQuery 或 sqlReaderStoredProcedureName，就會使用資料集 JSON 的結構區段中定義的資料行來建立一個查詢 (`select column1, column2 from mytable`)，以對 Azure SQL Database 執行。如果資料集定義沒有結構，則會從資料表中選取所有資料行。

> [AZURE.NOTE] 當您使用 **sqlReaderStoredProcedureName** 時，仍必須為資料集 JSON 中的 **tableName** 屬性指定值。雖然目前尚未針對此資料表來進行驗證。

### SqlSource 範例

    "source": {
        "type": "SqlSource",
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


### SqlSink 

**SqlSink** 支援下列屬性：

| 屬性 | 說明 | 允許的值 | 必要 |
| -------- | ----------- | -------------- | -------- |
| writeBatchTimeout | 在逾時前等待批次插入作業完成的時間。 | 時間範圍<br/><br/> 範例：“00:30:00” (30 分鐘)。 | 否 | 
| writeBatchSize | 當緩衝區大小達到 writeBatchSize 時，將資料插入 SQL 資料表中 | 整數 (資料列數目)| 否 (預設值：10000)
| sqlWriterCleanupScript | 指定要讓「複製活動」執行的查詢，以便清除特定分割的資料。如需詳細資訊，請參閱[可重複性](#repeatability-during-copy)一節。 | 查詢陳述式。 | 否 |
| sliceIdentifierColumnName | 指定要讓「複製活動」以自動產生的分割識別碼填入的資料行名稱，這可在重新執行時用來清除特定分割的資料。如需詳細資訊，請參閱[可重複性](#repeatability-during-copy)一節。 | 資料類型為 binary(32) 之資料行的資料行名稱。 | 否 |
| sqlWriterStoredProcedureName | 將資料更新插入 (更新/插入) 目標資料表中的預存程序名稱。 | 預存程序的名稱。 | 否 |
| storedProcedureParameters | 預存程序的參數。 | 名稱/值組。參數的名稱和大小寫必須符合預存程序參數的名稱和大小寫。 | 否 | 
| sqlWriterTableType | 指定要在預存程序中使用的資料表類型名稱。複製活動可讓正在移動的資料可用於此資料表類型的暫存資料表。然後，預存程序程式碼可以合併正在複製的資料與現有的資料。 | 資料表類型名稱。 | 否 |

#### SqlSink 範例

    "sink": {
        "type": "SqlSink",
        "writeBatchSize": 1000000,
        "writeBatchTimeout": "00:05:00",
        "sqlWriterStoredProcedureName": "CopyTestStoredProcedureWithParameters",
        "sqlWriterTableType": "CopyTestTableType",
        "storedProcedureParameters": {
            "id": { "value": "1", "type": "Int" },
            "stringData": { "value": "str1" },
            "decimalData": { "value": "1", "type": "Decimal" }
        }
    }

## 目標資料庫中的身分識別資料行
本節提供的範例將示範，如何把資料從沒有身分識別資料行的來源資料表，複製到具有身分識別資料行的目的地資料表。

**來源資料表：**

	create table dbo.SourceTbl
	(
	       name varchar(100),
	       age int
	)

**目的地資料表：**

	create table dbo.TargetTbl
	(
	       id int identity(1,1),
	       name varchar(100),
	       age int
	)


請注意，目標資料表具有身分識別資料行。

**來源資料集 JSON 定義**

	{
	    "name": "SampleSource",
	    "properties": {
	        "published": false,
	        "type": " SqlServerTable",
	        "linkedServiceName": "TestIdentitySQL",
	        "typeProperties": {
	            "tableName": "SourceTbl"
	        },
	        "availability": {
	            "frequency": "Hour",
	            "interval": 1
	        },
	        "external": true,
	        "policy": {}
	    }
	}

**目的地資料集 JSON 定義**

	{
	    "name": "SampleTarget",
	    "properties": {
	        "structure": [
	            { "name": "name" },
	            { "name": "age" }
	        ],
	        "published": false,
	        "type": "AzureSqlTable",
	        "linkedServiceName": "TestIdentitySQLSource",
	        "typeProperties": {
	            "tableName": "TargetTbl"
	        },
	        "availability": {
	            "frequency": "Hour",
	            "interval": 1
	        },
	        "external": false,
	        "policy": {}
	    }	
	}


請注意，您的來源資料表與目標資料表的結構描述不同 (目標資料表有一個具有身分識別的額外資料行)。在此案例中，您必須在目標資料集定義中指定 **structure** 屬性，這不包含身分識別資料行。

[AZURE.INCLUDE [data-factory-type-repeatability-for-sql-sources](../../includes/data-factory-type-repeatability-for-sql-sources.md)]

[AZURE.INCLUDE [data-factory-sql-invoke-stored-procedure](../../includes/data-factory-sql-invoke-stored-procedure.md)]
 

[AZURE.INCLUDE [data-factory-structure-for-rectangualr-datasets](../../includes/data-factory-structure-for-rectangualr-datasets.md)]

### SQL Server 和 Azure SQL Database 的類型對應

如同[資料移動活動](data-factory-data-movement-activities.md)一文所述，複製活動會使用下列 2 個步驟的方法，執行自動類型轉換，將來源類型轉換成接收類型：

1. 從原生來源類型轉換成 .NET 類型
2. 從 .NET 類型轉換成原生接收類型

將資料移到 Azure SQL、SQL Server、Sybase 或從這些位置移動資料時，會使用下列從 SQL 類型到 .NET 類型的對應，以及反向的對應。

此對應與 ADO.NET 的 SQL Server 資料類型對應相同。

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