<properties 
	pageTitle="從檔案系統來回移動資料 | Azure Data Factory" 
	description="了解如何使用 Azure Data Factory 從內部部署檔案系統來回移動資料。" 
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
	ms.date="09/01/2016" 
	ms.author="spelluru"/>

# 使用 Azure Data Factory 從內部部署檔案系統來回移動資料

本文概述如何使用資料處理站複製活動和內部部署檔案系統往來移動資料。如需可做為內部部署檔案系統之來源或接收的資料存放區清單，請參閱[支援的來源與接收](data-factory-data-movement-activities.md#supported-data-stores)。本文是根據[資料移動活動](data-factory-data-movement-activities.md)一文，該文呈現使用複製活動移動資料的一般概觀以及支援的資料存放區組合。

資料處理站支援透過資料管理閘道器連接至內部部署檔案系統。請參閱[在內部部署位置與雲端之間移動資料](data-factory-move-data-between-onprem-and-cloud.md)一文來了解資料管理閘道器和設定閘道器的逐步指示。

> [AZURE.NOTE] 
除了資料管理閘道器，不需要安裝其他二進位檔即可和內部部署檔案系統進行通訊。
> 
> 如需連接/閘道器相關問題的疑難排解秘訣，請參閱[針對閘道問題進行疑難排解](data-factory-data-management-gateway.md#troubleshoot-gateway-issues)。

## Linux 檔案共用 

執行下列兩個步驟來搭配使用 Linux 檔案共用和檔案伺服器連結服務：

- 在您的 Linux 伺服器上安裝 [Samba](https://www.samba.org/)。
- 在 Windows 伺服器上安裝和設定資料管理閘道器。不支援在 Linux 伺服器上安裝閘道器。
 
## 複製資料精靈
要建立將資料複製到內部部署檔案系統，或複製內部部署檔案系統資料的管線，最簡單的方法是使用複製資料精靈。如需使用複製資料精靈建立管線的快速逐步解說，請參閱[教學課程︰使用複製精靈建立管線](data-factory-copy-data-wizard-tutorial.md)。

以下範例提供可用來使用 [Azure 入口網站](data-factory-copy-activity-tutorial-using-azure-portal.md)或 [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) 或 [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md) 建立管線的範例 JSON 定義。它們會示範如何將資料複製到內部部署檔案系統和 Azure Blob 儲存體，以及複製其中的資料。不過，您可以在 Azure Data Factory 中使用複製活動，從任何來源**直接**將資料複製到[這裡](data-factory-data-movement-activities.md#supported-data-stores)所說的任何接收器。


## 範例：將資料從內部部署檔案系統複製到 Azure Blob

此範例示範如何將資料從內部部署檔案系統複製到 Azure Blob 儲存體。不過，您可以在 Azure Data Factory 中使用複製活動，**直接**將資料複製到[這裡](data-factory-data-movement-activities.md#supported-data-stores)所說的任何接收器。
 
此範例具有下列 Data Factory 實體：

- [OnPremisesFileServer](data-factory-onprem-file-system-connector.md#onpremisesfileserver-linked-service-properties) 類型的連結服務。
- [AzureStorage](data-factory-azure-blob-connector.md#azure-storage-linked-service-properties) 類型的連結服務。
- [FileShare](data-factory-onprem-file-system-connector.md#on-premises-file-system-dataset-type-properties) 類型的輸入[資料集](data-factory-create-datasets.md)。
- [AzureBlob](data-factory-azure-blob-connector.md#azure-blob-dataset-type-properties) 類型的輸出[資料集](data-factory-create-datasets.md)。
- 具有使用 [FileSystemSource](data-factory-onprem-file-system-connector.md#file-share-copy-activity-type-properties) 和 [BlobSink](data-factory-azure-blob-connector.md#azure-blob-copy-activity-type-properties) 之複製活動的[管線](data-factory-create-pipelines.md)。

下列範例每小時都會將時間序列資料從內部部署檔案系統複製到 Azure Blob。範例後面的各節會說明這些範例中使用的 JSON 屬性。

在第一個步驟中，根據[在內部部署位置與雲端之間移動資料](data-factory-move-data-between-onprem-and-cloud.md)一文中的指示，設定資料管理閘道器。

**內部部署檔案伺服器連結服務：**

	{
	  "Name": "OnPremisesFileServerLinkedService",
	  "properties": {
	    "type": "OnPremisesFileServer",
	    "typeProperties": {
	      "host": "\\\Contosogame-Asia.<region>.corp.<company>.com",
	      "userid": "Admin",
	      "password": "123456",
	      "gatewayName": "mygateway"
	    }
	  }
	}

我們建議使用 **encryptedCredential** 屬性，而不要使用 **userid** 和 **password** 屬性。如需此連結服務的詳細資訊，請參閱[檔案系統連結服務](#onpremisesfileserver-linked-service-properties)。

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

**內部部署檔案系統輸入資料集：**

每小時從新的檔案挑選資料。會根據配量的開始時間來決定 folderPath 和 fileName。

設定 "external": "true" 可讓 Data Factory 服務知道資料集是在 Data Factory 外部，而不是由 Data Factory 中的活動所產生。

	{
	  "name": "OnpremisesFileSystemInput",
	  "properties": {
	    "type": " FileShare",
	    "linkedServiceName": " OnPremisesFileServerLinkedService ",
	    "typeProperties": {
	      "folderPath": "mysharedfolder/yearno={Year}/monthno={Month}/dayno={Day}",
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
	      ]
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
	      "folderPath": "mycontainer/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
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

**複製活動：**

此管線包含複製活動，該活動已設定為使用輸入和輸出資料集並排定為每小時執行。在管線 JSON 定義中，**source** 類型設為 **FileSystemSource**，而 **sink** 類型設為 **BlobSink**。
	
	{  
	    "name":"SamplePipeline",
	    "properties":{  
	    "start":"2015-06-01T18:00:00",
	    "end":"2015-06-01T19:00:00",
	    "description":"Pipeline for copy activity",
	    "activities":[  
	      {
	        "name": "OnpremisesFileSystemtoBlob",
	        "description": "copy activity",
	        "type": "Copy",
	        "inputs": [
	          {
	            "name": "OnpremisesFileSystemInput"
	          }
	        ],
	        "outputs": [
	          {
	            "name": "AzureBlobOutput"
	          }
	        ],
	        "typeProperties": {
	          "source": {
	            "type": "FileSystemSource"
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

##範例：將資料從 Azure SQL 複製到內部部署檔案系統 

下列範例顯示︰

- AzureSqlDatabase 類型的連結服務。
- 類型 OnPremisesFileServer 的連結服務。
- AzureSqlTable 類型的輸入資料集。
- 類型 FileShare 的輸出資料集。
- 具有使用 SqlSource 和 FileSystemSink 之複製活動的管線。

此範例會每小時將時間序列資料從 Azure SQL 資料表複製到內部部署檔案系統。範例後面的各節會說明這些範例中使用的 JSON 屬性。

**Azure SQL 連結服務：**

	{
	  "name": "AzureSqlLinkedService",
	  "properties": {
	    "type": "AzureSqlDatabase",
	    "typeProperties": {
	      "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
	    }
	  }
	}

**內部部署檔案伺服器連結服務：**

	{
	  "Name": "OnPremisesFileServerLinkedService",
	  "properties": {
	    "type": "OnPremisesFileServer",
	    "typeProperties": {
	      "host": "\\\Contosogame-Asia.<region>.corp.<company>.com",
	      "userid": "Admin",
	      "password": "123456",
	      "gatewayName": "mygateway"
	    }
	  }
	}

我們建議使用 **encryptedCredential** 屬性，而不要使用 **userid** 和 **password** 屬性。如需此連結服務的詳細資訊，請參閱[檔案系統連結服務](#onpremisesfileserver-linked-service-properties)。

**Azure SQL 輸入資料集：**

此範例假設您已在 SQL Azure 中建立資料表 "MyTable"，其中包含時間序列資料的資料行 (名稱為 "timestampcolumn")。

設定 “external”: ”true” 可讓 Data Factory 服務知道資料集是在 Data Factory 外部，而不是由 Data Factory 中的活動所產生。

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

**內部部署檔案系統輸出資料集：**

資料會每小時複製到新的檔案。會根據配量的開始時間來決定 Blob 的 folderPath 和 fileName。

	{
	  "name": "OnpremisesFileSystemOutput",
	  "properties": {
	    "type": "FileShare",
	    "linkedServiceName": " OnPremisesFileServerLinkedService ",
	    "typeProperties": {
	      "folderPath": "mysharedfolder/yearno={Year}/monthno={Month}/dayno={Day}",
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
	      ]
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

**包含複製活動的管線：**此管線包含複製活動，該活動已設定為使用輸入和輸出資料集，並排定為每小時執行。在管線 JSON 定義中，**source** 類型設為 **SqlSource**，而 **sink** 類型設為 **FileSystemSink**。針對 **SqlReaderQuery** 屬性指定的 SQL 查詢會選取過去一小時內要複製的資料。

	
	{  
	    "name":"SamplePipeline",
	    "properties":{  
	    "start":"2015-06-01T18:00:00",
	    "end":"2015-06-01T20:00:00",
	    "description":"pipeline for copy activity",
	    "activities":[  
	      {
	        "name": "AzureSQLtoOnPremisesFile",
	        "description": "copy activity",
	        "type": "Copy",
	        "inputs": [
	          {
	            "name": "AzureSQLInput"
	          }
	        ],
	        "outputs": [
	          {
	            "name": "OnpremisesFileSystemOutput"
	          }
	        ],
	        "typeProperties": {
	          "source": {
	            "type": "SqlSource",
	            "SqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd}\\'', WindowStart, WindowEnd)"
	          },
	          "sink": {
	            "type": "FileSystemSink"
	          }
	        },
	       "scheduler": {
	          "frequency": "Hour",
	          "interval": 1
	        },
	        "policy": {
	          "concurrency": 1,
	          "executionPriorityOrder": "OldestFirst",
	          "retry": 3,
	          "timeout": "01:00:00"
	        }
	      }
	     ]
	   }
	}

## OnPremisesFileServer 連結服務屬性

您可以利用內部部署檔案伺服器連結服務將內部部署檔案系統連結至 Azure Data Factory。下表提供內部部署檔案伺服器連結服務專屬 JSON 元素的描述。

屬性 | 說明 | 必要
-------- | ----------- | --------
類型 | type 屬性應設為 **OnPremisesFileServer** | 是 
主機 | 想要複製之資料夾的根路徑。使用逸出字元 ‘ \\ ’ 當做字串中的特殊字元。如需範例，請參閱[範例連結服務和資料集定義](#sample-linked-service-and-dataset-definitions)。 | 是
userid | 指定具有伺服器存取權之使用者的識別碼 | 否 (如果您選擇 encryptedCredential)
password | 指定使用者的密碼 (userid) | 否 (如果您選擇 encryptedCredential) 
encryptedCredential | 指定可以透過執行 New-AzureRmDataFactoryEncryptValue Cmdlet 來取得的加密認證。 | 否 (如果您選擇以純文字指定使用者識別碼和密碼)
gatewayName | 資料處理站服務應該用來連接到內部部署檔案伺服器的閘道器名稱 | 是

如需為內部部署檔案系統資料來源設定認證的詳細資料，請參閱[設定認證和安全性](data-factory-move-data-between-onprem-and-cloud.md#set-credentials-and-security)。

### 範例連結服務和資料集定義 
案例 | 連結服務定義中的主機 | 資料集定義中的 folderPath
-------- | --------------------------------- | --------------------- |
資料管理閘道電腦上的本機資料夾︰<br/><br/>範例：D:\\* 或 D:\\folder\\subfolder\\* | D:\\\ (適用於閘道版本 2.0 和更新版本) <br/><br/> localhost (適用於版本 2.0 以下的閘道) | .\\\ 或 folder\\\subfolder (適用於閘道版本 2.0 和更新版本) <br/><br/>D:\\\ 或 D:\\\folder\\\subfolder (適用於版本 2.0 以下的閘道)
遠端共用資料夾︰<br/><br/>範例：\\\myserver\\share\\* 或 \\\myserver\\share\\folder\\subfolder\\* | \\\\\\myserver\\\share | .\\\ 或 folder\\\subfolder

若要尋找閘道器的**版本**︰
1. 在機器上啟動[資料管理閘道組態管理員](data-factory-data-management-gateway.md#data-management-gateway-configuration-manager)。
2. 切換至 [說明] 索引標籤。

> [AZURE.NOTE] 我們建議您[將閘道升級為版本 2.0 或更新版本](data-factory-data-management-gateway.md#update-data-management-gateway)以利用最新功能和修正。

**範例：使用純文字的使用者名稱和密碼**
	
	{
	  "Name": "OnPremisesFileServerLinkedService",
	  "properties": {
	    "type": "OnPremisesFileServer",
	    "typeProperties": {
	      "host": "\\\Contosogame-Asia",
	      "userid": "Admin",
	      "password": "123456",
	      "gatewayName": "mygateway"
	    }
	  }
	}
	
**範例：使用 encryptedcredential**

	{
	  "Name": " OnPremisesFileServerLinkedService ",
	  "properties": {
	    "type": "OnPremisesFileServer",
	    "typeProperties": {
	      "host": "D:\",
	      "encryptedCredential": "WFuIGlzIGRpc3Rpbmd1aXNoZWQsIG5vdCBvbmx5IGJ5xxxxxxxxxxxxxxxxx",
	      "gatewayName": "mygateway"
	    }
	  }
	}

## 內部部署檔案系統資料集類型屬性

如需定義資料集的區段和屬性完整清單，請參閱[建立資料集](data-factory-create-datasets.md)一文。資料集 JSON 的結構、可用性和原則等區段類似於所有的資料集類型。

每個資料集類型的 TypeProperties 區段都不同，可提供資料存放區中資料的位置、格式等相關資訊。**FileShare** 資料集類型的 typeProperties 區段具有下列屬性。

屬性 | 說明 | 必要
-------- | ----------- | --------
folderPath | 資料夾的子路徑。使用逸出字元 ‘ \\ ’ 當做字串中的特殊字元。如需範例，請參閱[範例連結服務和資料集定義](#sample-linked-service-and-dataset-definitions)。<br/><br/>您可以結合此屬性與 **partitionBy**，讓資料夾路徑以配量開始/結束日期時間為基礎。 | 是
fileName | 如果您想要資料表參考資料夾中的特定檔案，請指定 **folderPath** 中的檔案名稱。如果您未指定此屬性的任何值，資料表會指向資料夾中的所有檔案。<br/><br/>沒有為輸出資料集指定 fileName 時，所產生的檔案名稱會是下列格式：<br/><br/>Data.<Guid>.txt (範例：Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt | 否
partitionedBy | partitionedBy 可以用來指定時間序列資料的動態 folderPath 和 filename。例如，folderPath 可針對每小時的資料進行參數化。 | 否
格式 | 支援下列格式類型：**TextFormat**、**AvroFormat**、**JsonFormat** 及 **OrcFormat**。將格式下的 **type** 屬性設定為這些值其中之一。如需詳細資料，請參閱[指定 TextFormat](#specifying-textformat)、[指定 AvroFormat](#specifying-avroformat)、[指定 JsonFormat](#specifying-jsonformat) 及[指定 OrcFormat](#specifying-orcformat) 各節。如果您想要在以檔案為基礎的存放區之間依原樣複製檔案 (二進位複本)，您可以在輸入和輸出資料集定義中略過格式區段。 | 否
fileFilter | 指定要用來在 folderPath (而不是所有檔案) 中選取檔案子集的篩選器。<br/><br/>允許的值為：(多個字元) 和 ? (單一字元)。<br/><br/>範例 1："fileFilter": ".log"<br/>範例 2："fileFilter": 2014-1-?.txt"<br/><br/>**注意**：fileFilter 適用於輸入 FileShare 資料集 | 否
| compression | 指定此資料的壓縮類型和層級。支援的類型為：**GZip**、**Deflate** 及 **BZip2**，而支援的層級為：**最佳**和**最快**。**AvroFormat** 或 **OrcFormat** 格式的資料目前不支援壓縮設定。如需詳細資訊，請參閱[壓縮支援](#compression-support)一節。 | 否 |

> [AZURE.NOTE] 無法同時使用檔名和 fileFilter。

### 使用 partitionedBy 屬性

如上一節所述，您可以利用 partitionedBy 指定時間序列資料的動態 folderPath 和 filename。您可以使用 Data Factory 巨集和系統變數以及可指出給定資料配量的邏輯時間週期的 SliceStart、SliceEnd 來這麼做。

若要進一步了解時間序列資料集、排程和配量，請參閱[建立資料集](data-factory-create-datasets.md)、[排程和執行](data-factory-scheduling-and-execution.md)以及[建立管線](data-factory-create-pipelines.md)等文章。

#### 範例 1：

	"folderPath": "wikidatagateway/wikisampledataout/{Slice}",
	"partitionedBy": 
	[
	    { "name": "Slice", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyyMMddHH" } },
	],

在此範例中，{Slice} 會取代成 Data Factory 系統變數 SliceStart 的值 (使用指定的格式 (YYYYMMDDHH))。SliceStart 是指配量的開始時間。每個配量的 folderPath 都不同。例如：wikidatagateway/wikisampledataout/2014100103 或 wikidatagateway/wikisampledataout/2014100104。

#### 範例 2：

	"folderPath": "wikidatagateway/wikisampledataout/{Year}/{Month}/{Day}",
	"fileName": "{Hour}.csv",
	"partitionedBy": 
	 [
	    { "name": "Year", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyy" } },
	    { "name": "Month", "value": { "type": "DateTime", "date": "SliceStart", "format": "MM" } }, 
	    { "name": "Day", "value": { "type": "DateTime", "date": "SliceStart", "format": "dd" } }, 
	    { "name": "Hour", "value": { "type": "DateTime", "date": "SliceStart", "format": "hh" } } 
	],

在此範例中，SliceStart 的年、月、日和時間會擷取到 folderPath 和 fileName 屬性所使用的個別變數。

[AZURE.INCLUDE [data-factory-file-format](../../includes/data-factory-file-format.md)]  
[AZURE.INCLUDE [data-factory-compression](../../includes/data-factory-compression.md)]

## 檔案共用複製活動類型屬性

**FileSystemSource** 支援下列屬性：

| 屬性 | 說明 | 允許的值 | 必要 |
| -------- | ----------- | -------------- | -------- |
| 遞迴 | 表示是否從子資料夾，或只有從指定的資料夾，以遞迴方式讀取資料。 | True/False (預設值為 False)| 否 | 

**FileSystemSink** 支援下列屬性：

| 屬性 | 說明 | 允許的值 | 必要 |
| -------- | ----------- | -------------- | -------- |
| copyBehavior | 當來源為 BlobSource 或 FileSystem 時，定義複製行為。 | **PreserveHierarchy：**在目標資料夾中保留檔案階層，亦即，來源檔案至來源資料夾的相對路徑，與目標檔案至目標資料夾的相對路徑相同。<br/><br/>**FlattenHierarchy：**來源資料夾的所有檔案會建立在目標資料夾的第一層。目標檔案會使用自動產生的名稱來建立。<br/><br/>**MergeFiles：**會將來源資料夾中的所有檔案合併成一個檔案。如果已指定檔案/Blob 名稱，合併檔案名稱會是指定的名稱；否則，就會是自動產生的檔案名稱。 | 否 |

### 遞迴和 copyBehavior 範例
本節說明遞迴和 copyBehavior 值在不同組合的情況下，複製作業所產生的行為。

遞迴 | copyBehavior | 產生的行為
--------- | ------------ | --------
true | preserveHierarchy | 對於有下列結構的來源資料夾 Folder1：<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>目標資料夾 Folder1 是使用和來源資料夾相同的結構所建立<br/><br/>>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5.  
true | flattenHierarchy | 對於有下列結構的來源資料夾 Folder1：<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>目標 Folder1 是使用下列結構所建立：<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&File1 有自動產生的名稱<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2 有自動產生的名稱<br/>&nbsp;&nbsp;&nbsp;&nbsp;File3 有自動產生的名稱<br/>&nbsp;&nbsp;&nbsp;&nbsp;File4 有自動產生的名稱<br/>&nbsp;&nbsp;&nbsp;&nbsp;File5 有自動產生的名稱
true | mergeFiles | 對於有下列結構的來源資料夾 Folder1：<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>目標 Folder1 是使用下列結構所建立：<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1 + File2 + File3 + File4 + File 5 的內容會合併成一個有自動產生之檔案名稱的檔案<
false | preserveHierarchy | 對於有下列結構的來源資料夾 Folder1：<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>目標資料夾 Folder1 是使用下列結構所建立<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/><br/>系統不會挑選含有 File3、File4 和 File5 的 Subfolder1。
false | flattenHierarchy | 對於有下列結構的來源資料夾 Folder1：<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>目標資料夾 Folder1 是使用下列結構所建立<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1 有自動產生的名稱<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2 有自動產生的名稱<br/><br/>系統不會挑選含有 File3、File4 和 File5 的 Subfolder1。<
false | mergeFiles | 對於有下列結構的來源資料夾 Folder1：<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>目標資料夾 Folder1 是使用下列結構所建立<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1 + File2 的內容會合併成一個有自動產生之檔案名稱的檔案。File1 有自動產生的名稱<br/><br/>系統不會挑選含有 File3、File4 和 File5 的 Subfolder1。


[AZURE.INCLUDE [data-factory-structure-for-rectangualr-datasets](../../includes/data-factory-structure-for-rectangualr-datasets.md)]

[AZURE.INCLUDE [data-factory-column-mapping](../../includes/data-factory-column-mapping.md)]

## 效能和微調 
請參閱[複製活動的效能及微調指南](data-factory-copy-activity-performance.md)一文，以了解在 Azure Data Factory 中會影響資料移動 (複製活動) 效能的重要因素，以及各種最佳化的方法。






 

<!----HONumber=AcomDC_0907_2016-->