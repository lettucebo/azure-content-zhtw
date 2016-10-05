<properties 
	pageTitle="在 Azure Data Factory 中建立資料集 | Microsoft Azure" 
	description="了解如何透過使用位移和 anchorDateTime 等屬性的範例，在 Azure Data Factory 中建立資料集。"
    keywords="建立資料集, 資料集範例, 位移範例"
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
	ms.date="09/13/2016" 
	ms.author="spelluru"/>

# Azure Data Factory 中的資料集
本文說明 Azure Data Factory 中的資料集，並包含位移、anchorDateTime 和位移/樣式資料庫等範例。

當您建立資料集時，您會對想要處理的資料建立指標。資料會在活動中處理 (輸入/輸出)，而活動則包含在管線中。輸入資料集表示管線中的活動輸入，而輸出資料集表示活動的輸出。

資料集可識別資料表、檔案、資料夾和文件等各種資料存放區中的資料。建立資料集之後，您可以將其與管線中的活動搭配使用。例如，資料集可以是複製活動或 HDInsightHive 活動的輸入/輸出資料集。Azure 入口網站可提供您所有管線和資料的輸入和輸出的視覺化版面配置。您只需要看一眼，就可以看出管線在所有來源的所有關聯性和相依性，因此能永遠知道資料的來源和目的地。

在 Azure Data Factory 中，您可以使用管線中的複製活動從資料集取得資料。

> [AZURE.NOTE] 如果您不熟悉 Azure Data Factory，請參閱 [Azure Data Factory 簡介](data-factory-introduction.md)以取得 Azure Data Factory 服務的概觀。如需建立第一個 Data Factory 的教學課程，請參閱[建置第一個 Data Factory](data-factory-build-your-first-pipeline.md)。這兩篇文章提供您進一步了解這篇文章所需的背景資訊。

## 定義資料集
Azure Data Factory 中的資料集定義如下：


	{
	    "name": "<name of dataset>",
	    "properties": {
	        "type": "<type of dataset: AzureBlob, AzureSql etc...>",
			"external": <boolean flag to indicate external data. only for input datasets>,
	        "linkedServiceName": "<Name of the linked service that refers to a data store.>",
	        "structure": [
	            {
	                "name": "<Name of the column>",
	                "type": "<Name of the type>"
	            }
	        ],
	        "typeProperties": {
	            "<type specific property>": "<value>",
				"<type specific property 2>": "<value 2>",
	        },
	        "availability": {
	            "frequency": "<Specifies the time unit for data slice production. Supported frequency: Minute, Hour, Day, Week, Month>",
	            "interval": "<Specifies the interval within the defined frequency. For example, frequency set to 'Hour' and interval set to 1 indicates that new data slices should be produced hourly>"
	        },
	       "policy": 
	        {      
	        }
	    }
	}

下表描述上述 JSON 的屬性：

| 屬性 | 說明 | 必要 | 預設值 |
| -------- | ----------- | -------- | ------- |
| 名稱 | 資料集的名稱。請參閱 [Azure Data Factory - 命名規則](data-factory-naming-rules.md)，以了解命名規則。 | 是 | NA |
| 類型 | 資料集的類型。指定 Azure Data Factory 支援的其中一個類型 (例如︰AzureBlob、AzureSqlTable)。<br/><br/>請參閱[資料集類型](#Type)以取得詳細資訊。 | 是 | NA |
| structure | 資料集的結構描述<br/><br/>如需詳細資訊，請參閱[資料集結構](#Structure)一節。 | 編號 | NA |
| typeProperties | 對應至所選類型的屬性。請參閱[資料集類型](#Type)一節，以取得支援的類型及其屬性的詳細資訊。 | 是 | NA |
| external | 用來指定資料集是否由 Data Factory 管線明確產生的布林值旗標。 | 否 | false | 
| availability | 定義處理時間範圍或資料集生產的切割模型。<br/><br/>如需詳細資訊，請參閱[資料集可用性](#Availability)一節。<br/><br/>如需資料集切割模型的詳細資訊，請參閱[排程和執行](data-factory-scheduling-and-execution.md)一文。 | 是 | NA
| 原則 | 定義資料集配量必須符合的準則或條件。<br/><br/>如需詳細資訊，請參閱[資料集原則](#Policy)一節。 | 否 | NA |

## 資料集範例
在下列範例中，資料集表示 **Azure SQL Database** 中名為 **MyTable** 的資料表。

	{
	    "name": "DatasetSample",
	    "properties": {
	        "type": "AzureSqlTable",
	        "linkedServiceName": "AzureSqlLinkedService",
	        "typeProperties": 
	        {
	            "tableName": "MyTable"
	        },
	        "availability": 
	        {
	            "frequency": "Day",
	            "interval": 1
	        }
	    }
	}

請注意下列幾點：

- 類型設為 AzureSqlTable。
- tableName 類型屬性 (針對 AzureSqlTable 類型) 設定為 MyTable。
- linkedServiceName 代表 AzureSqlDatabase 類型的連結服務。請參閱下列連結服務的定義。
- 可用性頻率設為 [天]，間隔設為 1，表示每天產生配量。

AzureSqlLinkedService 定義如下︰

	{
	    "name": "AzureSqlLinkedService",
	    "properties": {
	        "type": "AzureSqlDatabase",
	        "description": "",
	        "typeProperties": {
	            "connectionString": "Data Source=tcp:<servername>.database.windows.net,1433;Initial Catalog=<databasename>;User ID=<username>@<servername>;Password=<password>;Integrated Security=False;Encrypt=True;Connect Timeout=30"
	        }
	    }
	}

在上述 JSON 中：

- 類型設為 AzureSqlDatabase
- connectionString 類型屬性會指定連接至 Azure SQL Database 的資訊。


如您所見，連結服務會定義如何連線到 Azure SQL Database。資料集會定義哪個資料表可做為管線中活動的輸入/輸出。您的[管線](data-factory-create-pipelines.md) JSON 中的 [活動] 區段會指定使用資料集做為輸入或輸出資料集。


> [AZURE.IMPORTANT] 除非資料集是由 Azure Data Factory 產生，否則應該標示為**外部**。此設定通常會套用至管線中第一個活動的輸入。

## <a name="Type"></a> 資料集類型
支援的資料來源和資料集類型會對應。如需資料集的類型和組態資訊，請參閱[資料移動活動](data-factory-data-movement-activities.md#supported-data-stores)一文中參照的主題。例如，如果您使用 Azure SQL Database 中的資料，請在支援的資料存放區清單中按一下 Azure SQL Database 以查看詳細資訊。

## <a name="Structure"></a>資料集結構
**Structure** 區段會定義資料集的結構描述。它包含資料行的名稱和資料類型的集合。在下列範例中，資料集有三個資料行 slicetimestamp、projectname 和 pageviews，它們的類型分別是：String、String 和 Decimal。

	structure:  
	[ 
	    { "name": "slicetimestamp", "type": "String"},
	    { "name": "projectname", "type": "String"},
	    { "name": "pageviews", "type": "Decimal"}
	]

## <a name="Availability"></a> 資料集可用性
資料集中的 **availability** 區段定義處理時間 (每小時、每天、每週等) 或資料集的切割模型。如需資料集切割和相依性模型的詳細資訊，請參閱[排程和執行](data-factory-scheduling-and-execution.md)。

下列 availability 區段指定輸出資料集是每小時產生 (或) 輸入資料集是每小時可用：

	"availability":	
	{	
		"frequency": "Hour",		
		"interval": 1	
	}

下表描述您可以在 [可用性] 區段中使用的屬性：

| 屬性 | 說明 | 必要 | 預設值 |
| -------- | ----------- | -------- | ------- |
| frequency | 指定資料集配量生產的時間單位。<br/><br/>**支援的頻率**：分鐘、小時、日、週、月 | 是 | NA |
| interval | 指定頻率的倍數<br/><br/>「頻率 x 間隔」可決定產生配量的頻率。<br/><br/>如果您需要每小時切割資料集，請將 **Frequency** 設為 **Hour**，將 **interval** 設為 **1**。<br/><br/>**注意：**如果您將 Frequency 指定為 Minute，建議您將間隔設為不小於 15 | 是 | NA |
| style | 指定是否應該在間隔開始/結束時產生配量。<ul><li>StartOfInterval</li><li>EndOfInterval</li></ul><br/><br/>如果 Frequency 設為 Month 且 style 設為 EndOfInterval，則會在月份的最後一天產生配量。如果 style 設為 StartOfInterval，則會在月份的第一天產生配量。<br/><br/>如果 Frequency 設為 Day 且 style 設為 EndOfInterval，則會在一天的最後一個小時產生配量。<br/><br/>如果 Frequency 設為 Hour 且 style 設為 EndOfInterval，則會在一小時結束時產生配量。例如，若為下午 1 – 2 點期間的配量，此配量會在下午 2 點產生。 | 否 | EndOfInterval |
| anchorDateTime | 定義排程器用來計算資料集配量界限的時間絕對位置。<br/><br/>**注意：**如果 AnchorDateTime 有比頻率更細微的日期部分，則會忽略更細微的部分。<br/><br/>例如，如果 **interval** 為**每小時** (frequency: hour 且 interval: 1) 而且 **AnchorDateTime** 包含**分鐘和秒鐘**，則會忽略 AnchorDateTime 的**分鐘和秒鐘**部分。 | 否 | 01/01/0001 |
| Offset | 所有資料集配量的開始和結束移位所依據的時間範圍。<br/><br/>**注意︰**如果已指定 anchorDateTime 和 offset，結果會是合併的移位。 | 否 | NA |

### 位移範例

於上午 6 點 (而非預設的午夜) 開始的每日配量。

	"availability":
	{
		"frequency": "Day",
		"interval": 1,
		"offset": "06:00:00"
	}

**頻率**設為**天**，**間隔**設為**1** (一天一次)︰如果您想要在上午 6 點而非預設時間上午 12 點產生配量。請記住，此時間是 UTC 時間。

## anchorDateTime 範例

**範例：**於 2007-04-19T08:00:00 開始的 23 小時資料集配量

	"availability":	
	{	
		"frequency": "Hour",		
		"interval": 23,	
		"anchorDateTime":"2007-04-19T08:00:00"	
	}

## 位移/樣式範例

如果您需要在特定日期和時間 (假設在每月 3 日上午 8:00) 執行以每月為基礎的資料集，請使用**位移**標籤，以設定其應該要執行的日期和時間。

	{
	  "name": "MyDataset",
	  "properties": {
	    "type": "AzureSqlTable",
	    "linkedServiceName": "AzureSqlLinkedService",
	    "typeProperties": {
	      "tableName": "MyTable"
	    },
	    "availability": {
	      "frequency": "Month",
	      "interval": 1,
	      "offset": "3.08:10:00",
	      "style": "StartOfInterval"
	    }
	  }
	}


## <a name="Policy"></a>資料集原則

資料集中的 **policy** 區段定義資料集配量必須符合的準則或條件。

### 驗證原則

| 原則名稱 | 說明 | 適用於 | 必要 | 預設值 |
| ----------- | ----------- | ---------- | -------- | ------- |
| minimumSizeMB | 驗證 **Azure Blob** 中的資料是否符合最小的大小需求 (以 MB 為單位)。 | Azure Blob | 否 | NA |
|minimumRows | 驗證 **Azure SQL Database** 或 **Azure 資料表**中的資料是否包含最小的資料列數。 | <ul><li>Azure SQL Database</li><li>Azure 資料表</li></ul> | 否 | NA

#### 範例

**minimumSizeMB：**

	"policy":
	
	{
	    "validation":
	    {
	        "minimumSizeMB": 10.0
	    }
	}

**minimumRows**

	"policy":
	{
		"validation":
		{
			"minimumRows": 100
		}
	}

### 外部資料集

外部資料集是並非由 Data Factory 中的執行中管線所產生的資料集。如果資料集標示為**外部**，則可定義 **ExternalData** 原則來影響資料集配量可用性的行為。

除非資料集是由 Azure Data Factory 產生，否則應該標示為**外部**。除非會使用活動或管線鏈結，否則此設定通常會套用到管線中第一個活動的輸入。

| 名稱 | 說明 | 必要 | 預設值 |
| ---- | ----------- | -------- | -------------- |
| dataDelay | 延遲指定配量之外部資料可用性檢查的時間。例如，如果資料應該每小時可用，則可以使用 dataDelay 來延遲「查看外部資料可用，且對應的配量已就緒」的檢查。<br/><br/>僅適用於目前的時間。例如，如果現在的時間是下午 1:00，且這個值是 10 分鐘，驗證會在下午 1:10 啟動。<br/><br/>此設定不會影響沒有任何延遲時過去配量 (「配量結束時間 + dataDelay < 現在時間」的配量) 的處理。<br/><br/>大於 23:59 小時的時間必須使用 day.hours:minutes:seconds 格式來指定。例如，若要指定 24 小時，請不要使用 24:00:00；請改用 1.00:00:00。如果您使用 24:00:00，這會視同 24 天 (24.00:00:00)。如為 1 天又 4 小時，請指定 1:04:00:00。 | 否 | 0 |
| retryInterval | 失敗與下一步重試嘗試之間的等待時間。適用於當前的時間；如果先前嘗試失敗，我們會在上次嘗試之後等待這麼久。<br/><br/>如果現在是下午 1:00，我們會開始第一次嘗試。如果完成第一次驗證檢查的持續時間是 1 分鐘且作業失敗，則下一次重試會在 1:00 + 1 分鐘 (持續時間) + 1 分鐘 (重試間隔) = 1:02pm。<br/><br/>若是過去的配量，則不會有任何延遲。重試會立即發生。 | 否 | 00:01:00 (1 分鐘) | 
| retryTimeout | 每次重試嘗試的逾時。<br/><br/>如果此屬性設為 10 分鐘，則必須在 10 分鐘內完成驗證。如果要花超過 10 分鐘來執行驗證，則重試會逾時。<br/><br/>如果所有驗證嘗試都逾時，則配量會標示為 TimedOut。 | 否 | 00:10:00 (10 分鐘) |
| maximumRetry | 檢查外部資料可用性的次數。允許的最大值為 10。 | 否 | 3 | 

## 範圍資料集
您可以使用 **datasets** 屬性來建立範圍設定為管線的資料集。這些資料集只能由此管線中的活動使用，不能由其他管線中的活動使用。下列範例會定義具有兩個資料集 (InputDataset-rdc 和 OutputDataset-rdc) 的管線，以在管線內使用：

> [AZURE.IMPORTANT] 只有一次性管線 (**pipelineMode** 設為 **OneTime**) 支援範圍資料集。如需詳細資訊，請參閱[一次性管線](data-factory-scheduling-and-execution.md#onetime-pipeline)。

	{
	    "name": "CopyPipeline-rdc",
	    "properties": {
	        "activities": [
	            {
	                "type": "Copy",
	                "typeProperties": {
	                    "source": {
	                        "type": "BlobSource",
	                        "recursive": false
	                    },
	                    "sink": {
	                        "type": "BlobSink",
	                        "writeBatchSize": 0,
	                        "writeBatchTimeout": "00:00:00"
	                    }
	                },
	                "inputs": [
	                    {
	                        "name": "InputDataset-rdc"
	                    }
	                ],
	                "outputs": [
	                    {
	                        "name": "OutputDataset-rdc"
	                    }
	                ],
	                "scheduler": {
	                    "frequency": "Day",
	                    "interval": 1,
	                    "style": "StartOfInterval"
	                },
	                "name": "CopyActivity-0"
	            }
	        ],
	        "start": "2016-02-28T00:00:00Z",
	        "end": "2016-02-28T00:00:00Z",
	        "isPaused": false,
	        "pipelineMode": "OneTime",
	        "expirationTime": "15.00:00:00",
	        "datasets": [
	            {
	                "name": "InputDataset-rdc",
	                "properties": {
	                    "type": "AzureBlob",
	                    "linkedServiceName": "InputLinkedService-rdc",
	                    "typeProperties": {
	                        "fileName": "emp.txt",
	                        "folderPath": "adftutorial/input",
	                        "format": {
	                            "type": "TextFormat",
	                            "rowDelimiter": "\n",
	                            "columnDelimiter": ","
	                        }
	                    },
	                    "availability": {
	                        "frequency": "Day",
	                        "interval": 1
	                    },
	                    "external": true,
	                    "policy": {}
	                }
	            },
	            {
	                "name": "OutputDataset-rdc",
	                "properties": {
	                    "type": "AzureBlob",
	                    "linkedServiceName": "OutputLinkedService-rdc",
	                    "typeProperties": {
	                        "fileName": "emp.txt",
	                        "folderPath": "adftutorial/output",
	                        "format": {
	                            "type": "TextFormat",
	                            "rowDelimiter": "\n",
	                            "columnDelimiter": ","
	                        }
	                    },
	                    "availability": {
	                        "frequency": "Day",
	                        "interval": 1
	                    },
	                    "external": false,
	                    "policy": {}
	                }
	            }
	        ]
	    }
	}

<!---HONumber=AcomDC_0921_2016-->