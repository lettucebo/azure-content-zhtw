<properties 
	pageTitle="Data Factory (資料整合服務) 簡介 | Microsoft Azure" 
	description="了解 Azure Data Factory 是什麼：一項雲端資料整合服務，用來協調以及自動移動和轉換資料。" 
	keywords="資料整合、雲端資料整合、azure data factory 是什麼"
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
	ms.date="09/22/2016" 
	ms.author="spelluru"/>

# Azure Data Factory 服務 (雲端中的資料整合服務) 簡介

## Azure 資料處理站是什麼？ 
Data Factory 是雲端式資料整合服務，可協調和自動化資料的**移動**及**轉換**作業。您可以使用 Data Factory 服務建立資料整合方案，以從各種資料存放區擷取資料、轉換/處理資料，並將結果資料發佈至資料存放區。

Data Factory 服務可讓您建立資料管線，以移動和轉換資料，然後依指定的排程 (每小時、每天、每週等) 執行管線。它也提供豐富的視覺效果來顯示資料管線之間的歷程和相依性，並可透過單一整合檢視監視您所有的資料管線，以輕鬆地找出問題並設定監視警示。

![圖表︰Data Factory 概觀 (資料整合服務)](./media/data-factory-introduction/what-is-azure-data-factory.png) **圖 1。** 從各種資料來源擷取資料，準備、轉換和分析資料，然後發佈立即可用的資料以供取用。

## 管線和活動
在 Data Factory 方案中，您可以建立一或多個資料**管線**。管線是活動的邏輯群組。可用來將活動群組成一個單位，共同執行任務。

**活動**可定義要對資料執行的動作。例如，您可以使用複製活動將資料從某個資料存放區複製到另一個資料存放區。同樣地，您可以使用在 Azure HDInsight 叢集上執行 Hive 查詢的 Hive 活動，來轉換或分析您的資料。Data Factory 支援兩種活動類型︰資料移動活動和資料轉換活動。
  
## 資料移動活動 
[AZURE.INCLUDE [data-factory-supported-data-stores](../../includes/data-factory-supported-data-stores.md)]

如需詳細資訊，請參閱[資料移動活動](data-factory-data-movement-activities.md)文章。

## 資料轉換活動
[AZURE.INCLUDE [data-factory-transformation-activities](../../includes/data-factory-transformation-activities.md)]

如需詳細資訊，請參閱[資料轉換活動](data-factory-data-transformation-activities.md)文章。

如果您需要將資料移入/移出「複製活動」不支援的資料存放區，或使用您自己的邏輯轉換資料，請建立**自訂 .NET 活動**。如需有關建立及使用自訂活動的詳細資料，請參閱[在 Azure Data Factory 管線中使用自訂活動](data-factory-use-custom-activities.md)。

## 連結的服務
連結的服務會定義 Data Factory 所需的資訊，以便連接到外部資源 (範例︰Azure 儲存體、內部部署 SQL Server、Azure HDInsight)。Data Factory 中的連結服務，有兩個用途：

- 用來代表**資料存放區**，其中包含 (但不限於) 內部部署 SQL Server、Oracle 資料庫、檔案共用或 Azure Blob 儲存體帳戶。如需支援的資料存放區清單，請參閱[資料移動活動](data-factory-data-movement-activities.md)一節。
- 用來代表可裝載活動執行的**計算資源**。例如，HDInsightHive 活動會在 HDInsight Hadoop 叢集上執行。如需支援的計算環境清單，請參閱[資料轉換活動](data-factory-data-transformation-activities.md)一節。

## 資料集 
連結服務會將資料存放區連結至 Azure Data Factory。資料集代表資料存放區內的資料結構。例如，Azure 儲存體連結的服務會提供可供 Data Factory 連線到 Azure 儲存體帳戶的連線資訊。Azure Blob 資料集會指定管線應從中讀取資料之 Azure Blob 儲存體中的 Blob 容器和資料夾。同樣地，Azure SQL 連結的服務會提供 Azure SQL Database 的連線資訊，而 Azure SQL 資料集則會指定包含資料的資料表。

## Data Factory 實體之間的關聯性
Data Factory 有幾個主要實體會共同運作，來定義輸入和輸出資料、處理事件以及執行指定之資料流程所需的排程和資源。

![圖表︰Data Factory 概觀 (雲端資料整合服務) - 重要概念](./media/data-factory-introduction/data-integration-service-key-concepts.png) **圖 2。** 資料集、活動、管線和連結服務之間的關聯性。

有了連結的服務、資料集、活動和管線這四個簡單的概念之後，您隨時可以開始使用！ 您可以[建置您的第一個管線](data-factory-build-your-first-pipeline.md)。

## 支援的區域
您目前可以在 [美國西部]、[美國東部] 和 [北歐] 區域建立 Data Factory。不過，Data Factory 可以存取其他 Azure 區域的資料存放區和計算資料，以在資料存放區之間移動資料或使用計算服務處理資料。

Azure Data Factory 本身不會儲存任何資料。它可讓您建立資料驅動的流程，以協調[所支援資料存放區](data-factory-data-movement-activities.md#supported-data-stores)之間的資料移動，以及使用[計算服務](data-factory-compute-linked-services.md)在其他區域或內部部署環境中處理資料。它也可讓您使用程式設計方式和 UI 機制[監視和管理工作流程](data-factory-monitor-manage-pipelines.md)。

即使 Azure Data Factory 只能在 [美國西部]、[美國東部] 和 [北歐] 區域使用，但 Data Factory 中支援資料移動的服務可在[全球](data-factory-data-movement-activities.md#global)好幾個區域中使用。如果資料存放區位於防火牆後面，則會改由內部部署環境中所安裝的[資料管理閘道](data-factory-move-data-between-onprem-and-cloud.md)移動資料。

如需範例，讓我們假設您的計算環境 (例如 Azure HDInsight 叢集和 Azure 機器學習服務) 即將用盡西歐區域的資源。您可以在北歐建立和利用 Azure Data Factory 執行個體，並用它排程您在西歐區域之計算環境上的工作。只要幾毫秒的時間，Data Factory 就能觸發計算環境上的作業，但執行計算環境上作業所需的時間則不會改變。

未來我們計畫讓 Azure 在每個地理位置都能支援 Azure Data Factory。
  
## 後續步驟
若要了解如何建置具有資料管線的 Data Factory，請遵循下列教學課程中的逐步指示。

教學課程 | 說明
-------- | -----------
[使用 Hadoop 叢集建置處理資料的資料管線](data-factory-build-your-first-pipeline.md) | 在本教學課程中，您會在 Azure HDInsight (Hadoop) 叢集上執行 Hive 指令碼，以建立您的第一個 Azure Data Factory 與用來**處理資料**的資料管線。 |
[建置資料管線以在兩個雲端資料存放區之間移動資料](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) | 在本教學課程中，您會建立具有管線的 Data Factory，以從 Blob 儲存體**移動資料**至 SQL Database。
[建置資料管線，以使用資料管理閘道在內部部署資料存放區與雲端資料存放區之間移動資料](data-factory-move-data-between-onprem-and-cloud.md) | 在本教學課程中，您會建置具有管線的 Data Factory，以從**內部部署** SQL Server 資料庫**移動資料**至 Azure Blob。在逐步解說中，您會在電腦上安裝及設定資料管理閘道。 

<!---HONumber=AcomDC_0928_2016-->