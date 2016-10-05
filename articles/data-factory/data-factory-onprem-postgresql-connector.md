<properties 
	pageTitle="從 PostgreSQL 移動資料 | Azure Data Factory" 
	description="了解如何使用 Azure Data Factory 從 PostgreSQL 資料庫移動資料。" 
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

# 使用 Azure Data Factory 從 PostgreSQL 移動資料

本文將概述如何使用 Azure 資料處理站中的複製活動將資料從 PostgreSQL 移動到另一個資料存放區。本文是根據[資料移動活動](data-factory-data-movement-activities.md)一文，該文呈現使用複製活動移動資料的一般概觀以及支援的資料存放區組合。

Data Factory 服務支援使用資料管理閘道器連接至內部部署 PostgreSQL 來源。請參閱[在內部部署位置與雲端之間移動資料](data-factory-move-data-between-onprem-and-cloud.md)一文來了解資料管理閘道和設定閘道的逐步指示。

> [AZURE.NOTE]
即使 PostgreSQL 資料庫裝載於 Azure IaaS VM 中，也必須要有閘道。您可以將閘道安裝在與資料存放區相同或相異的 IaaS VM 上，只要閘道可以連線到資料庫即可。

資料處理站只支援將資料從 PostgreSQL 移至其他資料存放區，而不支援從其他資料存放區移至 PostgreSQL。

## 安裝 

若要讓資料管理閘道器連接至 PostgreSQL 資料庫，您必須在與資料管理閘道器相同的系統上安裝 [PostgreSQL 的 Ngpsql 資料提供者](http://go.microsoft.com/fwlink/?linkid=282716)。

> [AZURE.NOTE] 如需連接/閘道器相關問題的疑難排解秘訣，請參閱[針對閘道問題進行疑難排解](data-factory-data-management-gateway.md#troubleshoot-gateway-issues)。

## 複製資料精靈
若要建立管線以將資料從 PostgreSQL 資料庫複製到任何支援的接收資料存放區，最簡單的方式是使用複製資料精靈。如需使用複製資料精靈建立管線的快速逐步解說，請參閱[教學課程︰使用複製精靈建立管線](data-factory-copy-data-wizard-tutorial.md)。

下列範例提供您使用 [Azure 入口網站](data-factory-copy-activity-tutorial-using-azure-portal.md)、[Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) 或 [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md) 來建立管線時，可使用的範例 JSON 定義。這些範例示範如何將資料從 PostgreSQL 資料庫複製到 Azure Blob 儲存體。不過，您可以在 Azure Data Factory 中使用複製活動，將資料複製到[這裡](data-factory-data-movement-activities.md#supported-data-stores)所說的任何接收器。

## 範例：從 PostgreSQL 複製資料到 Azure Blob

此範例示範如何將資料從 PostgreSQL 資料庫複製到 Azure Blob 儲存體。不過，您可以在 Azure Data Factory 中使用複製活動，**直接**將資料複製到[這裡](data-factory-data-movement-activities.md#supported-data-stores)所說的任何接收器。
 
此範例具有下列 Data Factory 實體：

1.	[OnPremisesPostgreSql](data-factory-onprem-postgresql-connector.md#postgresql-linked-service-properties) 類型的連結服務。
2.	[AzureStorage](data-factory-azure-blob-connector.md#azure-storage-linked-service-properties) 類型的連結服務。
3.	[RelationalTable](data-factory-onprem-postgresql-connector.md#postgresql-dataset-type-properties) 類型的輸入[資料集](data-factory-create-datasets.md)。
4.	[AzureBlob](data-factory-azure-blob-connector.md#azure-blob-dataset-type-properties) 類型的輸出[資料集](data-factory-create-datasets.md)。
4.	具有使用 [RelationalSource](data-factory-onprem-postgresql-connector.md#postgresql-copy-activity-type-properties) 和 [BlobSink](data-factory-azure-blob-connector.md#azure-blob-copy-activity-type-properties) 之複製活動的[管線](data-factory-create-pipelines.md)。

此範例會每個小時將資料從 PostgreSQL 資料庫中的查詢結果複製到 Blob。範例後面的各節會說明這些範例中使用的 JSON 屬性。

第一步是設定資料管理閘道。如需相關指示，請參閱[在內部部署位置和雲端之間移動資料](data-factory-move-data-between-onprem-and-cloud.md)。

**PostgreSQL 連結服務：**

	{
	    "name": "OnPremPostgreSqlLinkedService",
	    "properties": {
	        "type": "OnPremisesPostgreSql",
	        "typeProperties": {
	            "server": "<server>",
	            "database": "<database>",
	            "schema": "<schema>",
	            "authenticationType": "<authentication type>",
	            "username": "<username>",
	            "password": "<password>",
	            "gatewayName": "<gatewayName>"
	        }
	    }
	}

**Azure Blob 儲存體連結服務：**
		
	{
	  "name": "AzureStorageLinkedService",
	  "properties": {
	    "type": "AzureStorage",
	    "typeProperties": {
	      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<AccountName>;AccountKey=<AccountKey>"
	    }
	  }
	}

**PostgreSQL 輸入資料集：**

此範例假設您已在 PostgreSQL 中建立資料表 "MyTable"，其中包含時間序列資料的資料行 (名稱為 "timestamp")。

設定 “external”: true 會通知 Data Factory 服務：這是 Data Factory 外部的資料集而且不是由 Data Factory 中的活動所產生。

	{
	    "name": "PostgreSqlDataSet",
	    "properties": {
	        "type": "RelationalTable",
	        "linkedServiceName": "OnPremPostgreSqlLinkedService",
	        "typeProperties": {},
	        "availability": {
	            "frequency": "Hour",
	            "interval": 1
	        },
			"external": true,
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

資料會每小時寫入至新的 Blob (頻率：小時，間隔：1)。根據正在處理之配量的開始時間，以動態方式評估 Blob 的資料夾路徑和檔案名稱。資料夾路徑會使用開始時間的年、月、日和小時部分。

	{
	    "name": "AzureBlobPostgreSqlDataSet",
	    "properties": {
	        "type": "AzureBlob",
	        "linkedServiceName": "AzureStorageLinkedService",
	        "typeProperties": {
	            "folderPath": "mycontainer/postgresql/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
	            "format": {
	                "type": "TextFormat",
	                "rowDelimiter": "\n",
	                "columnDelimiter": "\t"
	            },
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
	        "availability": {
	            "frequency": "Hour",
	            "interval": 1
	        }
	    }
	}


**複製活動：**

此管線包含複製活動，該活動已設定為使用輸入和輸出資料集並排定為每小時執行。在管線 JSON 定義中，**source** 類型設為 **RelationalSource**，而 **sink** 類型設為 **BlobSink**。針對 **query** 屬性指定的 SQL 查詢會從 PostgreSQL Database 中的 public.usstates 資料表選取資料。
	
	{
	    "name": "CopyPostgreSqlToBlob",
	    "properties": {
	        "description": "pipeline for copy activity",
	        "activities": [
	            {
	                "type": "Copy",
	                "typeProperties": {
	                    "source": {
	                        "type": "RelationalSource",
	                        "query": "select * from "public"."usstates""
	                    },
	                    "sink": {
	                        "type": "BlobSink"
	                    }
	                },
	                "inputs": [
	                    {
	                        "name": "PostgreSqlDataSet"
	                    }
	                ],
	                "outputs": [
	                    {
	                        "name": "AzureBlobPostgreSqlDataSet"
	                    }
	                ],
	                "policy": {
	                    "timeout": "01:00:00",
	                    "concurrency": 1
	                },
	                "scheduler": {
	                    "frequency": "Hour",
	                    "interval": 1
	                },
	                "name": "PostgreSqlToBlob"
	            }
	        ],
	        "start": "2014-06-01T18:00:00Z",
	        "end": "2014-06-01T19:00:00Z"
	    }
	}


## PostgreSQL 連結服務屬性

下表提供 PostgreSQL 連結服務專屬 JSON 元素的描述。

屬性 | 說明 | 必要
-------- | ----------- | --------
類型 | 類型屬性必須設為：**OnPremisesPostgreSql** | 是
伺服器 | PostgreSQL 伺服器的名稱。 | 是 
資料庫 | PostgreSQL 資料庫的名稱。 | 是 
結構描述 | 在資料庫中的結構描述名稱。結構描述名稱會區分大小寫。 | 否 
authenticationType | 用來連接到 PostgreSQL 資料庫的驗證類型。可能的值為：匿名、基本和 Windows。 | 是 
username | 如果您使用基本或 Windows 驗證，請指定使用者名稱。 | 否 
password | 指定您為使用者名稱所指定之使用者帳戶的密碼。 | 否 
gatewayName | Data Factory 服務應該用來連接到內部部署 PostgreSQL 資料庫的閘道器名稱。 | 是 

如需關於設定內部部署 PostgreSQL 資料來源認證的詳細資料，請參閱[設定認證和安全性](data-factory-move-data-between-onprem-and-cloud.md#set-credentials-and-security)。

## PostgreSQL 資料集類型屬性

如需定義資料集的區段和屬性完整清單，請參閱[建立資料集](data-factory-create-datasets.md)一文。資料集 JSON 的結構、可用性和原則等區段類似於所有的資料集類型 (SQL Azure、Azure Blob、Azure 資料表等)。

每個資料集類型的 typeProperties 區段都不同，可提供資料存放區中資料的位置相關資訊。**RelationalTable** 資料集類型的 typeProperties 區段 (包含 PostgreSQL 資料集) 具有下列屬性：

屬性 | 說明 | 必要
-------- | ----------- | --------
tableName | PostgreSQL 資料庫執行個體中連結服務所參照的資料表名稱。tableName 會區分大小寫。 | 否 (如果已指定 **RelationalSource** 的 **query**) 

## PostgreSQL 複製活動類型屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[建立管線](data-factory-create-pipelines.md)一文。屬性 (例如名稱、描述、輸入和輸出資料表，以及原則) 適用於所有類型的活動。

另一方面，活動的 typeProperties 區段中可用的屬性會隨著每個活動類型而有所不同。就「複製活動」而言，這些屬性會根據來源和接收器的類型而有所不同。

如果來源類型為 **RelationalSource** (包含 PostgreSQL)，則 typeProperties 區段可使用下列屬性：

屬性 | 說明 | 允許的值 | 必要
-------- | ----------- | -------------- | --------
query | 使用自訂查詢來讀取資料。 | SQL 查詢字串。例如："query": "select * from "MySchema"."MyTable""。 | 否 (如果已指定 **dataset** 的 **tableName**)

> [AZURE.NOTE] 結構描述和資料表名稱會區分大小寫，而且必須在查詢中以 "" (雙引號) 括住。

**範例：**

 "query": "select * from "MySchema"."MyTable""

[AZURE.INCLUDE [data-factory-structure-for-rectangualr-datasets](../../includes/data-factory-structure-for-rectangualr-datasets.md)]

## PostgreSQL 的類型對應

如同[資料移動活動](data-factory-data-movement-activities.md)一文所述，複製活動會使用下列 2 個步驟的方法，執行自動類型轉換，將來源類型轉換成接收類型：

1. 從原生來源類型轉換成 .NET 類型
1. 從 .NET 類型轉換成原生接收類型

將資料移到 PostgreSQL 時，下列對應會用於從 PostgreSQL 類型到 .NET 類型。

PostgreSQL 資料庫類型 |	PostgresSQL 別名 | .NET Framework 類型
------------------------ | -------------------- | ---------------------
abstime | | Datetime
bigint | int8 | Int64
bigserial | serial8 | Int64
位元 [ (n) ] | | 位元組、字串
位元不同 [ (n) ] | varbit | 位元組、字串
布林值 | 布林 | Boolean
方塊 | | 位元組、字串
bytea | | 位元組、字串
字元 [ (n) ] | char [ (n) ] | String
字元不同 [ (n) ] | varchar [ (n) ] | String
cid | | String
cidr | | String
圓形 | | 位元組、字串
日期 | | Datetime
daterange | | String
雙精度| float8 | 兩倍
inet | | 位元組、字串
intarry | | String
int4range | | String
int8range | | String
integer | int, int4 | Int32
間隔 [ 欄位 ] [ (p) ] | | Timespan
json | | String
jsonb | | 位元組
線條 | | 位元組、字串
lseg | | 位元組、字串
macaddr | | 位元組、字串
money | | 十進位
數值 [ (p, s) ] | 十進位 [ (p, s) ] | 十進位
numrange | | String
oid | | Int32
路徑 | | 位元組、字串
pg\_lsn | | Int64
點 | | 位元組、字串
多邊形 | | 位元組、字串
real | float4 | 單一
smallint | int2 | Int16
smallserial | serial2 | Int16
serial | serial4 | Int32
文字 | | String


[AZURE.INCLUDE [data-factory-column-mapping](../../includes/data-factory-column-mapping.md)]

[AZURE.INCLUDE [data-factory-type-repeatability-for-relational-sources](../../includes/data-factory-type-repeatability-for-relational-sources.md)]

## 效能和微調  
請參閱[複製活動的效能及微調指南](data-factory-copy-activity-performance.md)一文，以了解在 Azure Data Factory 中會影響資料移動 (複製活動) 效能的重要因素，以及各種最佳化的方法。

<!---HONumber=AcomDC_0921_2016-->