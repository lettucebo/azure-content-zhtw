<properties
	pageTitle="從 Azure Blob 儲存體來回移動資料 | Microsoft Azure"
	description="從 Azure Blob 儲存體來回移動資料"
	services="machine-learning,storage"
	documentationCenter=""
	authors="bradsev"
	manager="jhubbard"
	editor="cgronlun" />

<tags
	ms.service="machine-learning"
	ms.workload="data-services"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/14/2016"
	ms.author="bradsev;sachouks" />

# 從 Azure Blob 儲存體來回移動資料

[AZURE.INCLUDE [cap-ingest-data-selector](../../includes/cap-ingest-data-selector.md)]

以下是有關從 Azure Blob 儲存體來回移動資料所使用之技術的指引連結：

[AZURE.INCLUDE [blob-storage-tool-selector](../../includes/machine-learning-blob-storage-tool-selector.md)]
 
哪一種方法最適合會取決於您的案例。[在 Azure 機器學習中的進階分析案例](machine-learning-data-science-plan-sample-scenarios.md)文章可協助您判斷進階分析程序中各種資料科學工作流程所需的資源。

> [AZURE.NOTE] 如需 Azure Blob 儲存體的完整介紹，請參閱 [Azure Blob 基本概念](../storage/storage-dotnet-how-to-use-blobs.md)和 [Azure Blob 服務](https://msdn.microsoft.com/library/azure/dd179376.aspx)。

或者，您可以使用 [Azure Data Factory](https://azure.microsoft.com/services/data-factory/) 來︰

- 建立和排程管線，以從 Azure Blob 儲存體下載資料，
- 將其傳遞至已發佈的 Azure Machine Learning Web 服務，
- 接收預測性分析結果，並
- 將結果上傳至儲存體。

如需詳細資訊，請參閱[使用 Azure Data Factory 和 Azure Machine Learning 來建立預測管線](../data-factory/data-factory-azure-ml-batch-execution-activity.md)。

## 必要條件

本文件假設您擁有 Azure 訂用帳戶、儲存體帳戶和該帳戶的對應儲存體金鑰。上傳/下載資料之前，您必須知道 Azure 儲存體帳戶名稱和帳戶金鑰。

- 若要設定 Azure 訂用帳戶，請參閱[免費試用一個月](https://azure.microsoft.com/pricing/free-trial/)。
- 如需建立儲存體帳戶的指示，以及取得帳戶和金鑰的資訊，請參閱[關於 Azure 儲存體帳戶](../storage/storage-create-storage-account.md)。

<!---HONumber=AcomDC_0921_2016-->