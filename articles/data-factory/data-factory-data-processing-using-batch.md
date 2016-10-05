<properties
    pageTitle="使用 Data Factory 和 Batch 處理大型資料集 | Microsoft Azure"
    description="說明如何使用 Azure Batch 的平行處理功能處理 Azure Data Factory 管線中的大量資料。"
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
    ms.date="07/18/2016"
    ms.author="spelluru"/>

# 使用 Data Factory 和 Batch 處理大型資料集
本文說明範例解決方案的架構，此解決方案能以自動且排程的方式移動和處理大型資料集。本文也提供如何使用 Azure Data Factory 和 Azure Batch 實作解決方案的端對端逐步解說。

這篇文章比我們的一般文章還長，因為它包含整個範例解決方案的逐步解說。如果您不熟悉 Batch 和 Data Factory，您可以了解這兩項服務以及它們如何搭配運作。如果您已對這兩項服務有所了解，而且要設計/架構解決方案，則可以將焦點純粹放在文章內的[架構章節](#architecture-of-sample-solution)，如果您正在開發原型或解決方案，或許也會想試試[逐步解說](#implementation-of-sample-solution)中的逐步指示。歡迎您對此內容以及您如何使用它提出看法。

讓我們先看看 Data Factory 和 Batch 服務如何有助於處理雲端中的大型資料集。

## 為何使用 Azure Batch？
Azure Batch 可讓您在雲端有效率地執行大規模的平行和高效能運算 (HPC) 應用程式。它是一項平台服務，可排程要在一組受管理的虛擬機器上執行的計算密集型工作，而且可以調整計算資源以符合工作的需求。

在使用 Batch 服務時，您可以定義用來大規模平行執行應用程式的 Azure 計算資源。您可以依需要或依排程的工作來執行，而不需要手動建立、設定和管理 HPC 叢集、個別的虛擬機器、虛擬網路或複雜的作業和工作排程基礎結構。

如果您不熟悉 Azure Batch，請參閱下列文章，因為其內容有助於了解本文所述解決方案的架構/實作。

- [Azure Batch 的基本概念](../batch/batch-technical-overview.md)
- [Batch 功能概觀](../batch/batch-api-basics.md)

(選擇性) 若要深入了解 Azure Batch，請參閱 [Azure Batch 的學習路徑](https://azure.microsoft.com/documentation/learning-paths/batch/)。

## 為何使用 Azure Data Factory？
Data Factory 是雲端架構資料整合服務，用來協調以及自動移動和轉換資料。透過 Data Factory 服務，您可以建立受管理的資料管線，將資料從內部部署和雲端的資料存放區移動至集中式資料存放區 (例如︰Azure Blob 儲存體)，以及使用服務 (例如 Azure HDInsight 和 Azure Machine Learning) 處理/轉換資料。您也可以利用排程方式 (每小時、每天、每週等) 排定執行資料管線，以及快速地監視和管理資料管線以找出問題並採取行動。

如果您不熟悉 Azure Data Factory，請參閱下列文章，因為其內容有助於了解本文所述解決方案的架構/實作。

- [Azure Data Factory 簡介](data-factory-introduction.md)
- [建置第一個資料管線](data-factory-build-your-first-pipeline.md)

(選擇性) 若要深入了解 Azure Data Factory，請參閱 [Azure Data Factory 的學習路徑](https://azure.microsoft.com/documentation/learning-paths/data-factory/)。

## Data Factory 和 Batch 一起使用
Data Factory 中有內建的活動，例如複製活動，其可將資料從來源資料存放區複製/移動到目的地資料存放區，以及 Hive 活動，其可在 Azure 上使用 Hadoop 叢集 (HDInsight) 處理資料。如需支援的轉換活動清單，請參閱[資料轉換活動](data-factory-data-transformation-activities.md)。

它也可讓您建立自訂的 .NET 活動來使用您自己的邏輯移動或處理資料，以及在 Azure HDInsight 叢集上或 Azure Bach VM 集區上執行這些活動。當您使用 Azure Batch 時，您可以將集區設定為根據您提供的公式自動調整大小 (根據工作負載新增或移除 VM)。

## 範例解決方案架構
雖然本文所描述的架構是針對簡單的解決方案，但它也和複雜案例相關，例如依金融服務、映像處理和轉譯以及基因分析進行的風險模型建立。

此圖表將說明 1) Data Factory 如何協調資料移動和處理，以及 2) Azure Batch 如何以平行方式處理資料。請下載並列印圖表以便參考 (11 x 17 英吋或 A3 大小)：[使用 Azure Batch 和 Data Factory 的 HPC 和資料協調](http://go.microsoft.com/fwlink/?LinkId=717686)。

[![大規模資料處理圖表](./media/data-factory-data-processing-using-batch/image1.png)](http://go.microsoft.com/fwlink/?LinkId=717686)

這些是程序中的基本步驟。此解決方案包含用來建置端對端解決方案的程式碼和說明。

1.  **以計算節點的集區 (VM) 設定 Azure Batch**。您可以指定節點數目和每個節點的大小。

2.  **建立 Azure Data Factory 執行個體**，並為其設定代表 Azure Blob 儲存體、Azure Batch 計算服務、輸入/輸出資料和工作流程/管線的實體，並建立具有會移動和轉換資料之活動的工作流程/管線。

3.   **在 Data Factory 管線建立自訂 .NET 活動**。活動是將在 Azure Batch 集區上執行的使用者程式碼。

4.  **將大量的輸入資料儲存為 Azure 儲存體中的 blob**。資料會分成邏輯配量 (通常依時間來分)。

5.  **Data Factory 將要以平行方式處理的資料複製到次要位置**。

6.  **Data Factory 使用 Batch 所配置的集區執行自訂活動**。Data Factory 可以同時執行多個活動。每個活動各處理某個配量的資料。結果會儲存在 Azure 儲存體中。

7.  **Data Factory 會將最終結果移至第三個位置**，以透過應用程式加以散發，或由其他工具做進一步的處理。

## 範例解決方案的實作
此範例解決方案是刻意簡化的，旨在為您示範如何搭配使用 Data Factory 和 Batch 來處理資料集。此解決方案純粹計算某個搜尋詞彙 (“Microsoft”) 在以時間序列組織的輸入檔案中出現的次數。它會將此計數輸出至輸出檔案。

**時間**：如果您熟悉 Azure、Data Factory 和 Batch 的基本概念，並且已符合下列先決條件，我們預估此解決方案將需要 1-2 小時來完成。

### 必要條件

#### Azure 訂閱
如果您沒有 Azure 訂用帳戶，則只需要幾分鐘的時間就可以建立免費試用帳戶。請參閱[免費試用](https://azure.microsoft.com/pricing/free-trial/)。

#### Azure 儲存體帳戶
在本教學課程中，您將使用 Azure 儲存體帳戶來儲存資料。如果您沒有 Azure 儲存體帳戶，請參閱[建立儲存體帳戶](../storage/storage-create-storage-account.md#create-a-storage-account)。範例解決方案會使用 Blob 儲存體。

#### Azure Batch 帳戶
使用 [Azure 入口網站](http://manage.windowsazure.com/)建立 Azure Batch 帳戶。請參閱[建立和管理 Azure Batch 帳戶](../batch/batch-account-create-portal.md)。請記下 Azure Batch 帳戶名稱和帳戶金鑰。您也可以使用 [New-AzureRmBatchAccount](https://msdn.microsoft.com/library/mt603749.aspx) Cmdlet 建立 Azure Batch 帳戶。如需使用此 Cmdlet 的詳細指示，請參閱[開始使用 Azure Batch PowerShell Cmdlet](../batch/batch-powershell-cmdlets-get-started.md)。

範例解決方案會使用 Azure Batch (透過 Azure Data Factory 管線間接使用)，以平行方式處理計算節點集區上的資料；該集區是受管理的虛擬機器集合。

#### Azure Batch 虛擬機器 (VM) 集區
建立至少有 2 個計算節點的 **Azure Batch 集區**。

1.  在 [Azure 入口網站](https://portal.azure.com)中，按一下左側功能標中的 [瀏覽]，然後按一下 [Batch 帳戶]。
2. 選取您的 Azure Batch 帳戶，以開啟 [Batch 帳戶] 刀鋒視窗。
3. 按一下 [集區] 圖格。
4. 在 [集區] 刀鋒視窗中，按一下工具列上的 [新增] 按鈕以新增集區。
	1. 輸入集區的識別碼 (**集區識別碼**)。請注意**集區的識別碼**；您將在建立 Data Factory 解決方案時需要它。
	2. 指定作業系統系列設定的 **Windows Server 2012 R2**。
	3. 選取**節點定價層**。
	4. 輸入 **2** 做為 [目標專用] 設定的值。
	5. 輸入 **2** 做為 [每個節點的工作上限] 設定的值。
	6. 按一下 [確定] 以建立集區。
 	
#### Azure 儲存體總管   
[Azure 儲存體總管 6 (工具)](https://azurestorageexplorer.codeplex.com/) 或 [CloudXplorer](http://clumsyleaf.com/products/cloudxplorer) (來自 ClumsyLeaf 軟體)。這些 GUI 工具可用來檢查及更改 Azure 儲存體專案中的資料，包括雲端架構應用程式的記錄檔。

1.  使用私用存取 (沒有匿名存取) 建立名為 **mycontainer** 的容器

2.  如果您使用 **CloudXplorer**，請建立具有下列結構的資料夾和子資料夾：

	![](./media/data-factory-data-processing-using-batch/image3.png)

	**Inputfolder** 和 **outputfolder** 是 **mycontainer** 中的最上層資料夾，且 **inputfolder** 包含具有日期時間戳記 (YYYY-MM-DD-HH) 的子資料夾。

	如果您使用 **Azure 儲存體總管**，在下一個步驟中，您必須上傳具有下列名稱的檔案：inputfolder/2015-11-16-00/file.txt、inputfolder/2015-11-16-01/file.txt、依此類推。這會自動建立資料夾。

3.  在您的電腦上建立內容中含有關鍵字 **Microsoft** 的文字檔 **file.txt** 。例如：“test custom activity Microsoft test custom activity Microsoft”。

4.  將檔案上傳至 Azure Blob 儲存體中的下列輸入資料夾。

	![](./media/data-factory-data-processing-using-batch/image4.png)

	如果您使用 **Azure 儲存體總管**，請將檔案 **file.txt** 上傳至 **mycontainer**。在工具列上按一下 [複製]，以建立 blob 的複本。在 [複製 Blob] 對話方塊中，將**目的地 Blob 名稱**變更為 **inputfolder/2015-11-16-00/file.txt。** 重複此步驟，以建立 inputfolder/2015-11-16-01/file.txt、inputfolder/2015-11-16-02/file.txt、inputfolder/2015-11-16-03/file.txt、inputfolder/2015-11-16-04/file.txt，依此類推。這會自動建立資料夾。

3.  建立另一個名為 **customactivitycontainer** 的容器。您會將自訂活動 zip 檔案上傳至此容器。

#### Visual Studio
安裝 Microsoft Visual Studio 2012 或更新版本，以建立要在 Data Factory 解決方案中使用的自訂 Batch 活動。

### 建立解決方案的高階步驟

1.  建立包含資料處理邏輯的自訂活動。
2.  建立使用自訂活動的 Azure Data Factory：

詳細步驟位於下列各節。

### 建立自訂活動

Data Factory 自訂活動是此範例解決方案的核心。範例解決方案會使用 Azure Batch 執行自訂活動。如需開發自訂活動，並在 Azure Data Factory 管線中加以使用的基本資訊，請參閱[在 Azure Data Factory 管線中使用自訂活動](data-factory-use-custom-activities.md)。

若要建立您可以在 Azure Data Factory 管線中使用的 .NET 自訂活動，您必須利用實作 **IDotNetActivity** 介面的類別建立 **.NET 類別庫**專案。這個介面只有一個方法：**執行**。以下是該方法的簽章：

	public IDictionary<string, string> Execute(
	            IEnumerable<LinkedService> linkedServices,
	            IEnumerable<Dataset> datasets,
	            Activity activity,
	            IActivityLogger logger)

此方法有幾個您必須了解的關鍵元件。

-   此方法會採用四個參數：

    1.  **linkedServices**。這是將輸入/輸出資料來源 (例如：Azure Blob 儲存體) 連結到 Data Factory 的連結服務列舉清單。在此範例中，只有一個用於輸入和輸出的 Azure 儲存體類型連結服務。

    2.  **資料集**。這是資料集的可列舉清單。您可以使用這個參數取得輸入和輸出資料集定義的位置和結構描述。

    3.  **活動**。這個參數代表目前的計算實體 - 在此情況下為 Azure Batch 服務。

    4.  **記錄器**。記錄器可讓您撰寫會呈現為管線的「使用者」記錄檔的偵錯註解。

-   此方法會傳回未來可用來將自訂活動鏈結在一起的字典。尚未實作這項功能，因此只會從方法傳回空的字典。

#### 程序：建立自訂活動

1.  在 Visual Studio 中建立 .NET 類別庫專案。

    1.  啟動 **Visual Studio 2012**/**2013/2015**。

    2.  按一下 [**檔案**]，指向 [**新增**]，然後按一下 [**專案**]。

    3.  展開 [範本]，然後選取 [Visual C#]。在此逐步解說中，您使用 C# 中，但您可以使用任何 .NET 語言來開發自訂活動。

    4.  從右邊的專案類型清單中選取 [類別庫]。

    5.  針對 [名稱] 輸入 **MyDotNetActivity**。

    6.  在 [位置] 中選取 **C:\\ADF**。建立資料夾 **ADF** (如果不存在)。

    7.  按一下 [確定] 以建立專案。

2.  按一下 [工具]，指向 [NuGet 封裝管理員]，然後按一下 [封裝管理員主控台]。

3.  在 [封裝管理員主控台] 中，執行下列命令以匯入 **Microsoft.Azure.Management.DataFactories**。

			Install-Package Microsoft.Azure.Management.DataFactories

4.  將 **Azure 儲存體** NuGet 封裝匯入專案中。您將使用 Blob 儲存體 API，因此您會需要此封裝。

		Install-Package Azure.Storage

5.  將下列 **using** 指示詞加入至專案中的原始程式檔。

		using System.IO;
		using System.Globalization;
		using System.Diagnostics;
		using System.Linq;

		using Microsoft.Azure.Management.DataFactories.Models;
		using Microsoft.Azure.Management.DataFactories.Runtime;

		using Microsoft.WindowsAzure.Storage;
		using Microsoft.WindowsAzure.Storage.Blob;

6.  將 **namespace** 的名稱變更為 **MyDotNetActivityNS**。

		namespace MyDotNetActivityNS

7.  將類別的名稱變更為 **MyDotNetActivity**，並從 **IDotNetActivity** 介面延伸它，如下所示。

		public class MyDotNetActivity : IDotNetActivity

8.  對 **MyDotNetActivity** 類別實作 (加入) **IDotNetActivity** 介面的 **Execute** 方法，並將下列範例程式碼複製到方法。請參閱 [Execute 方法](#execute-method)一節，以了解此方法中使用的邏輯。

		/// <summary>
        /// Execute method is the only method of IDotNetActivity interface you must implement.
        /// In this sample, the method invokes the Calculate method to perform the core logic.  
		/// </summary>
        public IDictionary<string, string> Execute(
            IEnumerable<LinkedService> linkedServices,
            IEnumerable<Dataset> datasets,
            Activity activity,
            IActivityLogger logger)
        {

            // declare types for input and output data stores
            AzureStorageLinkedService inputLinkedService;

            Dataset inputDataset = datasets.Single(dataset => dataset.Name == activity.Inputs.Single().Name);
	
            foreach (LinkedService ls in linkedServices)
                logger.Write("linkedService.Name {0}", ls.Name);

            // using First method instead of Single since we are using the same
            // Azure Storage linked service for input and output.
            inputLinkedService = linkedServices.First(
                linkedService =>
                linkedService.Name ==
                inputDataset.Properties.LinkedServiceName).Properties.TypeProperties
                as AzureStorageLinkedService;

            string connectionString = inputLinkedService.ConnectionString; // To create an input storage client.
            string folderPath = GetFolderPath(inputDataset);
            string output = string.Empty; // for use later.

            // create storage client for input. Pass the connection string.
            CloudStorageAccount inputStorageAccount = CloudStorageAccount.Parse(connectionString);
            CloudBlobClient inputClient = inputStorageAccount.CreateCloudBlobClient();

            // initialize the continuation token before using it in the do-while loop.
            BlobContinuationToken continuationToken = null;
            do
            {   // get the list of input blobs from the input storage client object.
                BlobResultSegment blobList = inputClient.ListBlobsSegmented(folderPath,
                                         true,
                                         BlobListingDetails.Metadata,
                                         null,
                                         continuationToken,
                                         null,
                                         null);

                // Calculate method returns the number of occurrences of
                // the search term (“Microsoft”) in each blob associated
        		// with the data slice.
        		//
        	    // definition of the method is shown in the next step.
                output = Calculate(blobList, logger, folderPath, ref continuationToken, "Microsoft");

            } while (continuationToken != null);

            // get the output dataset using the name of the dataset matched to a name in the Activity output collection.
            Dataset outputDataset = datasets.Single(dataset => dataset.Name == activity.Outputs.Single().Name);

            folderPath = GetFolderPath(outputDataset);

            logger.Write("Writing blob to the folder: {0}", folderPath);

            // create a storage object for the output blob.
            CloudStorageAccount outputStorageAccount = CloudStorageAccount.Parse(connectionString);
            // write the name of the file.
            Uri outputBlobUri = new Uri(outputStorageAccount.BlobEndpoint, folderPath + "/" + GetFileName(outputDataset));

            logger.Write("output blob URI: {0}", outputBlobUri.ToString());
            // create a new blob and upload the output text.
            CloudBlockBlob outputBlob = new CloudBlockBlob(outputBlobUri, outputStorageAccount.Credentials);
            logger.Write("Writing {0} to the output blob", output);
            outputBlob.UploadText(output);

			// The dictionary can be used to chain custom activities together in the future.
			// This feature is not implemented yet, so just return an empty dictionary.
            return new Dictionary<string, string>();
        }


9.  將下列協助程式方法加入至類別。這些方法可用 **Execute** 方法來叫用。最重要的是，**Calculate** 方法會隔離逐一查看每個 blob 的程式碼。

        /// <summary>
        /// Gets the folderPath value from the input/output dataset.
		/// </summary>
		private static string GetFolderPath(Dataset dataArtifact)
		{
		    if (dataArtifact == null || dataArtifact.Properties == null)
		    {
		        return null;
		    }

		    AzureBlobDataset blobDataset = dataArtifact.Properties.TypeProperties as AzureBlobDataset;
		    if (blobDataset == null)
		    {
		        return null;
		    }

		    return blobDataset.FolderPath;
		}

		/// <summary>
		/// Gets the fileName value from the input/output dataset.
		/// </summary>

		private static string GetFileName(Dataset dataArtifact)
		{
		    if (dataArtifact == null || dataArtifact.Properties == null)
		    {
		        return null;
		    }

		    AzureBlobDataset blobDataset = dataArtifact.Properties.TypeProperties as AzureBlobDataset;
		    if (blobDataset == null)
		    {
		        return null;
		    }

		    return blobDataset.FileName;
		}

		/// <summary>
		/// Iterates through each blob (file) in the folder, counts the number of instances of search term in the file,
		/// and prepares the output text that will be written to the output blob.
		/// </summary>

		public static string Calculate(BlobResultSegment Bresult, IActivityLogger logger, string folderPath, ref BlobContinuationToken token, string searchTerm)
		{
		    string output = string.Empty;
		    logger.Write("number of blobs found: {0}", Bresult.Results.Count<IListBlobItem>());
		    foreach (IListBlobItem listBlobItem in Bresult.Results)
		    {
		        CloudBlockBlob inputBlob = listBlobItem as CloudBlockBlob;
		        if ((inputBlob != null) && (inputBlob.Name.IndexOf("$$$.$$$") == -1))
		        {
		            string blobText = inputBlob.DownloadText(Encoding.ASCII, null, null, null);
		            logger.Write("input blob text: {0}", blobText);
		            string[] source = blobText.Split(new char[] { '.', '?', '!', ' ', ';', ':', ',' }, StringSplitOptions.RemoveEmptyEntries);
		            var matchQuery = from word in source
		                             where word.ToLowerInvariant() == searchTerm.ToLowerInvariant()
		                             select word;
		            int wordCount = matchQuery.Count();
		            output += string.Format("{0} occurrences(s) of the search term "{1}" were found in the file {2}.\r\n", wordCount, searchTerm, inputBlob.Name);
		        }
		    }
		    return output;
		}


	**GetFolderPath** 方法會將路徑傳回資料集所指向的資料夾，而 **GetFileName** 方法會傳回資料集指向的 blob/檔案名稱。

	    "name": "InputDataset",
	    "properties": {
	        "type": "AzureBlob",
	        "linkedServiceName": "StorageLinkedService",
	        "typeProperties": {
	            "fileName": "file.txt",
	            "folderPath": "mycontainer/inputfolder/{Year}-{Month}-{Day}-{Hour}",


	**Calculate** 方法會在輸入檔案 (資料夾中的 blob) 計算關鍵字 **Microsoft** 的執行個體數目。搜尋詞彙 ("Microsoft") 已在程式碼中硬式編碼。

10.  編譯專案。按一下功能表中的 [建置]，然後按一下 [建置方案]。

11.  啟動 [Windows 檔案總管]，瀏覽至 **bin\\debug** 或 **bin\\release** 資料夾，視建置類型而定。

12.  建立 zip 檔案 **MyDotNetActivity.zip**，檔案中包含 **\\bin\\Debug** 資料夾中的所有二進位檔。您可能會想加入 MyDotNetActivity.**pdb** 檔案，讓您可以取得額外的詳細資訊，例如在失敗時，原始程式碼中引起問題的程式碼行號。

	![](./media/data-factory-data-processing-using-batch/image5.png)

13.  將 **MyDotNetActivity.zip** 當做 Blob 上傳至 Blob 容器：Azure Blob 儲存體中的 **customactivitycontainer**，由 **ADFTutorialDataFactory** 中的 **StorageLinkedService** 連結服務使用。如果 Blob 容器 **customactivitycontainer** 不存在，請自行建立。

#### 執行方法

本節提供 Execute 方法中與程式碼相關的詳細資料和注意事項。

1.	逐一查看輸入集合的成員可在 [Microsoft.WindowsAzure.Storage.Blob](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.storage.blob.aspx) 命名空間中找到。逐一查看 blob 集合需要使用 **BlobContinuationToken** 類別。基本上，您必須搭配使用 do-while 迴圈和權杖做為結束迴圈的機制。如需詳細資訊，請參閱[如何從 .NET 使用 Blob 儲存體](../storage/storage-dotnet-how-to-use-blobs.md)。基本迴圈如下所示：

		// Initialize the continuation token.
		BlobContinuationToken continuationToken = null;
		do
		{
		// Get the list of input blobs from the input storage client object.
		BlobResultSegment blobList = inputClient.ListBlobsSegmented(folderPath,
		    					true,
		                                  BlobListingDetails.Metadata,
		                                  null,
		                                  continuationToken,
		                                  null,
		                                  null);
		// Return a string derived from parsing each blob.
		    output = Calculate(blobList, logger, folderPath, ref continuationToken, "Microsoft");

		} while (continuationToken != null);

	請參閱 [ListBlobsSegmented](https://msdn.microsoft.com/library/jj717596.aspx) 方法的文件以了解詳細資料。

2.	以邏輯方式逐一查看 blob 集的程式碼會在 do-while 迴圈中執行。在 **Execute** 方法中，執行 do-while 迴圈會將 blob 清單傳遞至名為 **Calculate** 的方法。此方法會傳回名為 **output** 的字串變數，也就是在區段中逐一查看所有 blob 的結果。

	它會在傳遞給 **Calculate** 方法的 blob 中，傳回搜尋詞彙 (**Microsoft**) 的出現次數。

		output += string.Format("{0} occurrences of the search term "{1}" were found in the file {2}.\r\n", wordCount, searchTerm, inputBlob.Name);

3.	一旦 **Calculate** 方法完成此工作，它必須寫入至新的 blob。因此對於每個處理過的 blob 集，都可以利用結果撰寫新的 blob。若要寫入新的 blob，請先找到輸出資料集。

		// Get the output dataset using the name of the dataset matched to a name in the Activity output collection.
		Dataset outputDataset = datasets.Single(dataset => dataset.Name == activity.Outputs.Single().Name);

4.	程式碼也會呼叫 helper 方法：**GetFolderPath** 來擷取資料夾路徑 (儲存體容器名稱)。

		folderPath = GetFolderPath(outputDataset);

	**GetFolderPath** 會將 DataSet 物件轉換成 AzureBlobDataSet，其具有一個名為 FolderPath 的屬性。

		AzureBlobDataset blobDataset = dataArtifact.Properties.TypeProperties as AzureBlobDataset;

		return blobDataset.FolderPath;

5.	程式碼會呼叫 **GetFileName** 方法來擷取檔案名稱 (blob 名稱)。程式碼取得資料夾路徑的方式類似上述程式碼。

		AzureBlobDataset blobDataset = dataArtifact.Properties.TypeProperties as AzureBlobDataset;

		return blobDataset.FileName;

6.	藉由建立新的 URI 物件寫入檔案的名稱。URI 建構函式使用 **BlobEndpoint** 屬性傳回容器名稱。新增資料夾路徑和檔案名稱以建構輸出 blob URI。

		// Write the name of the file.
		Uri outputBlobUri = new Uri(outputStorageAccount.BlobEndpoint, folderPath + "/" + GetFileName(outputDataset));

7.	已寫入檔案名稱，現在您可以從 **Calculate** 方法將輸出字串寫入新的 blob：

		// Create a new blob and upload the output text.
		CloudBlockBlob outputBlob = new CloudBlockBlob(outputBlobUri, outputStorageAccount.Credentials);
		logger.Write("Writing {0} to the output blob", output);
		outputBlob.UploadText(output);


### 建立 Data Factory

在 [建立自訂活動][](#create-the-custom-activity) 區段中，您建立自訂活動，並將包含二進位檔和 PDB 檔案的 zip 檔案上傳到 Azure blob 容器。在本節中，您將透過使用**自訂活動**的**管線**建立 Azure **Data Factory**。

自訂活動的輸入資料集代表 blob 儲存體中輸入資料夾 (mycontainer\\inputfolder) 的 blob (檔案)。活動的輸出資料集代表 blob 儲存體中輸出資料夾 (mycontainer\\outputfolder) 的輸出 blob。

您會將一或多個檔案放置在輸入資料夾中：

	mycontainer -> inputfolder
		2015-11-16-00
		2015-11-16-01
		2015-11-16-02
		2015-11-16-03
		2015-11-16-04

例如，將含有下列內容的一個檔案 (file.txt) 放入每個資料夾中。

	test custom activity Microsoft test custom activity Microsoft

即使資料夾有 2 個以上的檔案，每個輸入資料夾還是會對應至 Azure Data Factory 中的配量。管線處理每個配量時，自訂活動會為該配量逐一查看輸入資料夾中的所有 blob。

您會看到五個具有相同內容的輸出檔案。例如，處理 2015-11-16-00 資料夾中的檔案所產生的輸出檔案會包含下列內容：

	2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-00/file.txt.

如果您將多個具有相同內容的檔案 (file.txt、file2.txt、file3.txt) 放置到輸入資料夾中，您將會在輸出檔案中看見下列內容。每個資料夾 (2015-11-16-00 等) 分別對應至此範例中的配量，即使資料夾有多個輸入檔案亦然。

	2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-00/file.txt.
	2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-00/file2.txt.
	2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-00/file3.txt.


請注意，輸出檔案現在會有三行，與配量相關聯的資料夾 (2015-11-16-00) 中的每個輸入檔案 (blob) 各一行。

每個活動執行都會建立一個工作。在此範例中，管線中只有一個活動。由管線處理配量時，自訂活動即會在 Azure Batch 上執行，以處理配量。由於有 5 個配量 (每個配量可以有多個 blob 或檔案)，因此在 Azure Batch 中會建立 5 個工作。工作在 Batch 上執行時，它實際上就是執行中的自訂活動。

下列逐步解說將提供其他詳細資料。

#### 步驟 1：建立 Data Factory

1.  登入 [Azure 入口網站](https://portal.azure.com/)之後，請執行下列動作：

    1.  按一下左側功能表上的 [新增]。

    2.  按一下 [新增] 刀鋒視窗中的 [資料 + 分析]。

    3.  按一下 [資料分析] 刀鋒視窗上的 [Data Factory]。

2.  在 [新增 Data Factory] 刀鋒視窗中，輸入 **CustomActivityFactory** 做為 [名稱]。Azure Data Factory 的名稱在全域必須是唯一的。如果您收到錯誤：**Data Factory 名稱 “CustomActivityFactory” 無法使用**，請變更 Data Factory 名稱 (例如 **yournameCustomActivityFactory**)，然後試著重新建立。

3.  按一下 [資源群組名稱]，並選取現有的資源群組，或建立一個新群組。

4.  請確認您使用的是要在其中建立 Data Factory 的正確訂用帳戶和區域。

5.  按一下 [新增 Data Factory] 刀鋒視窗上的 [建立]。

6.  您會看到 Data Factory 建立在 Azure 入口網站的 [儀表板] 中。

7.  在 Data Factory 成功建立後，您會看到 Data Factory 頁面，顯示 Data Factory 的內容。

 ![](./media/data-factory-data-processing-using-batch/image6.png)

#### 步驟 2：建立連結服務

連結服務會將資料存放區或計算服務連結至 Azure Data Factory。在此步驟中，您會將您的 **Azure 儲存體**帳戶和 **Azure Batch** 帳戶連結到您的 Data Factory。

#### 建立 Azure 儲存體連結服務

1.  按一下 **CustomActivityFactory** 的 **DATA FACTORY** 刀鋒視窗上的 [作者和部署] 磚。這會啟動 Data Factory 編輯器。

2.  在命令列上按一下 [新增資料存放區]，然後選擇 [Azure 儲存體]。 在編輯器中，您應該會看到用來建立 Azure 儲存體連結服務的 JSON 指令碼。

    ![](./media/data-factory-data-processing-using-batch/image7.png)

3.  使用您的 Azure 儲存體帳戶名稱取代帳戶名稱，並使用 Azure 儲存體帳戶的存取金鑰取代帳戶金鑰。若要了解如何取得儲存體存取金鑰，請參閱[檢視、複製和重新產生儲存體存取金鑰](../storage/storage-create-storage-account.md#view-copy-and-regenerate-storage-access-keys)

4.  按一下命令列的 [部署]，部署連結服務。

    ![](./media/data-factory-data-processing-using-batch/image8.png)

#### 建立 Azure Batch 連結服務

在此步驟中，您將為您的 **Azure Batch** 帳戶建立用來執行 Data Factory 自訂活動的連結服務。

1.  在命令列上按一下 [新增計算]，然後選擇 [Azure Batch]。 您應該會在編輯器中看到用來建立 Azure Batch 連結服務的 JSON 指令碼。

2.  在 JSON 指令碼中：

    1.  使用您的 Azure Batch 帳戶名稱來取代**帳戶名稱**。

    2.  使用 Azure Batch 帳戶的存取金鑰來取代**存取金鑰**。

    3.  針對 **poolName** 屬性輸入集區識別碼。 您可以針對此屬性指定集區名稱或集區識別碼。

    4.  針對 **batchUri** JSON 屬性，輸入 Batch URI。
    
		> [AZURE.IMPORTANT] \[Azure Batch 帳戶刀鋒視窗] 中的 **URL** 格式如下：\<accountname\>.\<region\>.batch.azure.com。針對 JSON 中的 **batchUri** 屬性，您必須從該 URL 中**移除 "accountname"**。範例："batchUri": "https://eastus.batch.azure.com"。

        ![](./media/data-factory-data-processing-using-batch/image9.png)

		針對 **poolName** 屬性，您也可以指定該集區的 ID，而非集區名稱。

		> [AZURE.NOTE] 與支援 HDInsight 的情況不同，Data Factory 服務不支援 Azure Batch 的隨選選項。您只能使用 Azure Data Factory 中自己的 Azure Batch 集區。

    5.  針對 **linkedServiceName** 屬性，指定 **StorageLinkedService**。您已在前述步驟中建立此連結服務。此儲存體會做為檔案和記錄檔的預備區域。

3.  按一下命令列的 [部署]，部署連結服務。

#### 步驟 3：建立資料集

在此步驟中，您將建立資料集來代表輸入和輸出資料。

#### 建立輸入資料集

1.  在 Data Factory 的**編輯器**中，按一下工具列上的 [**新增資料集**] 按鈕，然後從下拉式功能表中選取 [**Azure Blob 儲存體**]。

2.  使用下列 JSON 程式碼片段取代右窗格中的 JSON：

		{
		    "name": "InputDataset",
		    "properties": {
		        "type": "AzureBlob",
		        "linkedServiceName": "AzureStorageLinkedService",
		        "typeProperties": {
		            "folderPath": "mycontainer/inputfolder/{Year}-{Month}-{Day}-{Hour}",
		            "format": {
		                "type": "TextFormat"
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
		        },
		        "external": true,
		        "policy": {}
		    }
		}


	 您稍後將在本逐步解說建立管線，開始時間為：2015-11-16T00:00:00Z，而結束時間為：2015-11-16T05:00:00Z。排程為**每小時**產生，將會有 5 個輸入/輸出配量 (在 **00**:00:00 -> **05**:00:00 之間)。

	 輸入資料集的**頻率**和**間隔**設定為**小時**和 **1**，這表示輸入配量會每小時提供一次。

	 以下是每個配量的開始時間，由上述 JSON 程式碼片段中的 **SliceStart** 系統變數代表。

	| **配量** | **開始時間** |
	|-----------|-------------------------|
	| 1 | 2015-11-16T**00**:00:00 |
	| 2 | 2015-11-16T**01**:00:00 |
	| 3 | 2015-11-16T**02**:00:00 |
	| 4 | 2015-11-16T**03**:00:00 |
	| 5 | 2015-11-16T**04**:00:00 |

	 **FolderPath** 是使用配量開始時間 (**SliceStart**) 的年、月、日和小時部分來計算的。因此，輸入資料夾對應至配量的方式如下。

	| **配量** | **開始時間** | **輸入資料夾** |
	|-----------|-------------------------|-------------------|
	| 1 | 2015-11-16T**00**:00:00 | 2015-11-16-**00** |
	| 2 | 2015-11-16T**01**:00:00 | 2015-11-16-**01** |
	| 3 | 2015-11-16T**02**:00:00 | 2015-11-16-**02** |
	| 4 | 2015-11-16T**03**:00:00 | 2015-11-16-**03** |
	| 5 | 2015-11-16T**04**:00:00 | 2015-11-16-**04** |

3.  按一下工具列上的 [部署]，以建立並部署 **InputDataset** 資料表。

#### 建立輸出資料集

在此步驟中，您將建立屬於 AzureBlob 類型的另一個資料集，來代表輸出資料。

1.  在 Data Factory 的**編輯器**中，按一下工具列上的 [**新增資料集**] 按鈕，然後從下拉式功能表中選取 [**Azure Blob 儲存體**]。

2.  使用下列 JSON 程式碼片段取代右窗格中的 JSON：

		{
		    "name": "OutputDataset",
		    "properties": {
		        "type": "AzureBlob",
		        "linkedServiceName": "AzureStorageLinkedService",
		        "typeProperties": {
		            "fileName": "{slice}.txt",
		            "folderPath": "mycontainer/outputfolder",
		            "partitionedBy": [
		                {
		                    "name": "slice",
		                    "value": {
		                        "type": "DateTime",
		                        "date": "SliceStart",
		                        "format": "yyyy-MM-dd-HH"
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


 	為每個輸入配量產生輸出 blob/檔案。以下是為每個配量命名輸出檔案的方式。所有的輸出檔案會產生於一個輸出資料夾中：**mycontainer\\outputfolder**。


	| **配量** | **開始時間** | **輸出檔案** |
	|-----------|-------------------------|-----------------------|
	| 1 | 2015-11-16T**00**:00:00 | 2015-11-16-**00.txt** |
	| 2 | 2015-11-16T**01**:00:00 | 2015-11-16-**01.txt** |
	| 3 | 2015-11-16T**02**:00:00 | 2015-11-16-**02.txt** |
	| 4 | 2015-11-16T**03**:00:00 | 2015-11-16-**03.txt** |
	| 5 | 2015-11-16T**04**:00:00 | 2015-11-16-**04.txt** |

	 請記得，輸入資料夾 (例如：2015-11-16-00) 中的所有檔案，都是開始時間為 2015-11-16-00 之配量的一部分。處理此配量時，自訂活動會掃描每個檔案，並利用搜尋詞彙 (“Microsoft”) 的出現次數在輸出檔案中產生資料行。如果資料夾 2015-11-16-00 中有三個檔案，則輸出檔案中將會有三行：2015-11-16-00.txt。

3.  按一下工具列上的 [部署]，以建立並部署 **OutputDataset**。

#### 步驟 4：建立並執行使用自訂活動的管線

在此步驟中，您將建立具有一個活動的管線，也就是您先前建立的自訂活動。

> [AZURE.IMPORTANT] 如果尚未將 **file.txt** 上傳至 blob 容器中的輸入資料夾，請先執行此動作，再建立管線。在管線 JSON 中，**IsPaused** 屬性會設定為 false，使管線會在**開始**日期到達後立即執行。

1.  在 Data Factory 編輯器中，按一下工具列上的 [**新增管線**]。如果看不到此命令，請按一下 [...] \(省略符號) 就可看到。

2.  使用下列 JSON 指令碼取代右窗格中的 JSON。

		{
			"name": "PipelineCustom",
			"properties": {
				"description": "Use custom activity",
				"activities": [
					{
						"type": "DotNetActivity",
						"typeProperties": {
							"assemblyName": "MyDotNetActivity.dll",
							"entryPoint": "MyDotNetActivityNS.MyDotNetActivity",
							"packageLinkedService": "AzureStorageLinkedService",
							"packageFile": "customactivitycontainer/MyDotNetActivity.zip"
						},
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
						"policy": {
							"timeout": "00:30:00",
							"concurrency": 5,
							"retry": 3
						},
						"scheduler": {
							"frequency": "Hour",
							"interval": 1
						},
						"name": "MyDotNetActivity",
						"linkedServiceName": "AzureBatchLinkedService"
					}
				],
				"start": "2015-11-16T00:00:00Z",
				"end": "2015-11-16T05:00:00Z",
				"isPaused": false
		   }
		}

	請注意：

	-   管線中只有一個活動，且其類型為：**DotNetActivity**。

	-   **AssemblyName** 會設定為 DLL 的名稱：**MyDotNetActivity.dll**。

	-   **EntryPoint** 設定為 **MyDotNetActivityNS.MyDotNetActivity**。在您的程式碼中，它基本上是 <namespace>.<classname>。

	-   **PackageLinkedService** 設為 **StorageLinkedService**，會指向包含自訂活動 zip 檔案的 Blob 儲存體。如果您將不同的 Azure 儲存體帳戶用於輸入/輸出檔案和自訂活動 zip 檔案，您必須建立另一個 Azure 儲存體連結服務。本文假設您使用相同的 Azure 儲存體帳戶。

	-   **PackageFile** 設定為 **customactivitycontainer/MyDotNetActivity.zip**。其格式為：<containerforthezip>/<nameofthezip.zip>。

	-   自訂活動會採用 **InputDataset** 做為輸入和 **OutputDataset** 做為輸出。

	-   自訂活動的 **linkedServiceName** 屬性會指向 **AzureBatchLinkedService**，這會告知 Azure Data Factory 自訂活動必須在 Azure Batch 上執行。

	-   **並行**設定很重要。如果您使用預設值 1，則即使您在 Azure Batch 集區中有 2 個或更多計算節點，配量仍會逐一進行處理。因此，您將無法使用 Azure Batch 的平行處理功能。如果您將**並行**設定為較高的值 (例如 2)，這表示可以同時處理 2 個配量 (對應於 Azure Batch 中的 2 個工作)，在此情況下，將會同時使用 Azure Batch 集區中的兩個 VM。因此，請適當設定並行屬性。

	-   根據預設，無論何時，一個工作 (配量) 都只會在一個 VM 上執行。這是因為 Azure Batch 集區的 [每個 VM 的工作數上限] 預設為 1。根據必要條件，您在建立集區時將此屬性設定為 2，因此可以同時在 VM 上執行兩個 Data Factory 配量。


	-   **isPaused** 屬性預設為 false。在此範例中，管線會立即執行，因為配量已在過去開始。您可以將此屬性設為 true，以暫停管線，並將其設回 false，以重新啟動。

	-   **開始**時間和**結束**時間相差 5 小時，而配量會每小時產生，因此管線會產生 5 個配量。

3.  按一下命令列上的 [部署]，部署管線。

#### 步驟 5：測試管線

在此步驟中，您會將檔案放置在輸入資料夾中，以測試管線。首先，我們以每一個輸入資料夾含有一個檔案的方式來測試管線。

1.  在 Azure 入口網站的 [Data Factory] 刀鋒視窗中，按一下 [圖表]。

    ![](./media/data-factory-data-processing-using-batch/image10.png)

2.  在圖表檢視中，按兩下輸入資料集：**InputDataset**。

    ![](./media/data-factory-data-processing-using-batch/image11.png)

3.  您應會看到 **InputDataset** 刀鋒視窗中已包含所有的 5 個配量。請留意每個配量的 [配量開始時間] 和 [配量結束時間]。

    ![](./media/data-factory-data-processing-using-batch/image12.png)

4.  現在在 [圖表檢視] 中，按一下 **OutputDataset**。

5.  如果已產生 5 個輸出配量，您應該會看到它們都處於就緒狀態。

    ![](./media/data-factory-data-processing-using-batch/image13.png)

6.  使用 Azure 入口網站檢視與**配量**相關聯的**工作**，並查看每個配量在哪個 VM 上執行。請參閱 [Data Factory 和 Batch 整合](#data-factory-and-batch-integration)一節，以取得詳細資料。

7.  在您的 Azure Blob 儲存體中，您應會在 **mycontainer** 的 **outputfolder** 中看見輸出檔案。

    ![](./media/data-factory-data-processing-using-batch/image15.png)

    您應會看見五個輸出檔案，每個輸入配量各一個。每個輸出檔案應會有如下的內容：

    	2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-00/file.txt.

    下圖說明 Data Factory 配量如何對應至 Azure Batch 中的工作。在此範例中，一個配量只有一個執行。

    ![](./media/data-factory-data-processing-using-batch/image16.png)

8.  現在，我們要以一個資料夾包含多個檔案的方式來試試。使用與資料夾 **2015-11-06-01** 中的 file.txt 相同的內容，建立檔案 **file2.txt**、**file3.txt**、**file4.txt** 和 **file5.txt**。

9.  在輸出資料夾中，**刪除**輸出檔案：**2015-11-16-01.txt**。

10. 現在，在 **OutputDataset** 刀鋒視窗中，以滑鼠右鍵按一下 [配量開始時間] 設定為 **11/16/2015 01:00:00 AM** 的配量，然後按一下 [執行]，以重新執行/重新處理配量。現在，配量會有 5 個檔案，而不是 1 個檔案。

    ![](./media/data-factory-data-processing-using-batch/image17.png)

11. 在配量執行完成，且其狀態為 [就緒] 之後，在您 Blob 儲存體之 **mycontainer** 的 **outputfolder** 中驗證此配量的輸出檔案 (**2015-11-16-01.txt**) 中的內容。配量的每個檔案應該都有一行。

		2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-01/file.txt.
		2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-01/file2.txt.
		2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-01/file3.txt.
		2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-01/file4.txt.
		2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-01/file5.txt.


> [AZURE.NOTE] 如果您未先刪除輸出檔案 2015-11-16-01.txt 即以 5 個輸入檔案來嘗試，您將會看到先前的配量執行有一行，而目前的配量執行有五行。根據預設，內容會附加至已存在的輸出檔案。

#### Data Factory 和 Batch 整合
Data Factory 服務會在 Azure Batch 中建立作業，其名為：**adf-poolname:job-xxx**。

![Azure Data Factory - Batch 作業](media/data-factory-data-processing-using-batch/data-factory-batch-jobs.png)

配量的每個活動執行都會在作業中建立一個工作。如果有 10 個配量就緒可供處理，作業中會建立 10 個工作。如果您在集區中有多個計算結點，您可以同時執行多個配量。如果每個計算結點的最大工作設為 > 1，您也可以在相同的計算中執行多個配量。

在此範例中會有 5 個配量，因此 Azure Batch 中有 5 個工作。在 Azure Data Factory 中的管線 JSON 中將**並行**設定為 **5**，並且在具有 **2** 個 VM 的 Azure Batch 集區中將 [每個 VM 的工作數上限] 設定為 **2**，工作會執行得非常快 (請檢查工作的開始和結束時間)。

使用入口網站來檢視與**配量**相關聯的 Batch 作業及其工作，並查看每個配量在哪個 VM 上執行。

![Azure Data Factory - Batch 作業工作](media/data-factory-data-processing-using-batch/data-factory-batch-job-tasks.png)

### 偵錯管線

偵錯包含一些基本技術：

1.  如果輸入配量不是設定為 [就緒]，請確認輸入資料夾結構正確，且 file.txt 存在於輸入資料夾中。

    ![](./media/data-factory-data-processing-using-batch/image3.png)

2.  在自訂活動的 **Execute** 方法中，使用可協助您疑難排解問題的 **IActivityLogger** 物件記錄資訊。記錄的訊息會顯示在 user\_0.log 檔案中。

    在 [OutputDataset] 刀鋒視窗中，按一下配量，以查看該配量的 [資料配量] 刀鋒視窗。您會看到該配量的**活動執行**。您會看到一個為該配量執行的活動。如果您按一下命令列中的 [執行]，您可以為相同的配量啟動另一個活動執行。

    當您按一下活動執行，您會看到包含記錄檔清單的 [活動執行詳細資料] 刀鋒視窗。您會在 **user\_0.log** 檔案中看到記錄的訊息。發生錯誤時，您會看到三個活動執行，因為管線/活動 JSON 中的重試計數設定為 3。當您按一下活動執行，您會看到您可以檢閱的記錄檔來疑難排解錯誤。

    ![](./media/data-factory-data-processing-using-batch/image18.png)

    在記錄檔清單中，按一下 [user-0.log]。在右窗格中的是使用 **IActivityLogger.Write** 方法的結果。

    ![](./media/data-factory-data-processing-using-batch/image19.png)

    您也應該檢查 system-0.log 是否有任何系統錯誤訊息和例外狀況。

		Trace\_T\_D\_12/6/2015 1:43:35 AM\_T\_D\_\_T\_D\_Verbose\_T\_D\_0\_T\_D\_Loading assembly file MyDotNetActivity...

		Trace\_T\_D\_12/6/2015 1:43:35 AM\_T\_D\_\_T\_D\_Verbose\_T\_D\_0\_T\_D\_Creating an instance of MyDotNetActivityNS.MyDotNetActivity from assembly file MyDotNetActivity...

		Trace\_T\_D\_12/6/2015 1:43:35 AM\_T\_D\_\_T\_D\_Verbose\_T\_D\_0\_T\_D\_Executing Module

		Trace\_T\_D\_12/6/2015 1:43:38 AM\_T\_D\_\_T\_D\_Information\_T\_D\_0\_T\_D\_Activity e3817da0-d843-4c5c-85c6-40ba7424dce2 finished successfully

3.  在 zip 檔案中包含 **PDB** 檔案，錯誤詳細資料才會在錯誤發生時包含**呼叫堆疊**等資訊。

4.  自訂活動之 zip 檔案中的所有檔案都必須位於**最上層**且不包含任何子資料夾。

    ![](./media/data-factory-data-processing-using-batch/image20.png)

5.  確認 **assemblyName** (MyDotNetActivity.dll)、**entryPoint**(MyDotNetActivityNS.MyDotNetActivity)、**packageFile** (customactivitycontainer/MyDotNetActivity.zip) 和 **packageLinkedService** (應指向包含 zip 檔案的 Azure Blob 儲存體) 都設為正確的值。

6.  如果您修正錯誤，並想要重新處理配量，請以滑鼠右鍵按一下 [OutputDataset] 刀鋒視窗中的配量，然後按一下 [執行]。

    ![](./media/data-factory-data-processing-using-batch/image21.png)

    **注意：** 您會在 Azure Blob 儲存體中看到一個**容器**，名為：**adfjobs**。此容器並不會自動刪除，但您可在完成解決方案的測試後安全地加以刪除。同樣地，Data Factory 解決方案也會建立 Azure Batch **作業**，名為：**adf-<pool ID/name>:job-0000000001**。您可以在完成解決方案的測試之後刪除此作業 (如果您要的話)。
7. 自訂活動不會使用來自您套件的 **app.config** 檔案，因此如果您的程式碼會從組態檔讀取任何連接字串，在執行階段將沒有作用。最佳做法是使用 Azure Batch 將所有密碼存放 **Azure KeyVault** 中、使用以憑證為基礎的服務主體來保護 keyvault，然後將憑證發佈至 Azure Batch 集區。接著，.NET 自訂活動便可以在執行階段從 KeyVault 存取密碼。這是一般解決方案，而且可以擴展至任何類型的密碼，不僅限於連接字串。

	此外，也有較簡單的因應措施 (但並非最佳做法)︰您可以建立一個帶有連接字串設定的新**Azure SQL 連結服務**、建立一個使用該連結服務的資料集，然後將該資料集以虛擬輸入資料集的形式鏈結至自訂 .NET 活動。接著，您便可以在自訂活動程式碼中存取連結服務的連接字串，並且這在執行階段應該能夠發揮作用。

#### 擴充範例

您可以擴充此範例，以深入了解 Azure Data Factory 和 Azure Batch 的功能。例如，若要處理不同時間範圍的配量，請執行下列動作：

1.  在 **inputfolder** 中新增下列子資料夾，然後將輸入檔案放入這些資料夾中：2015-11-16-05、2015-11-16-06、201-11-16-07、2011-11-16-08、2015-11-16-09。將管線的結束時間從 2015-11-16T05:00:00Z 變更為 2015-11-16T10:00:00Z。在 [圖表檢視] 中，按兩下 **InputDataset**，並確認輸入配量已就緒。按兩下 **OuptutDataset**，以查看輸出配量的狀態。如果它們都處於 [就緒] 狀態，請在 outputfolder 中查看輸出檔案。

2.  增加或減少**並行**設定，以了解它對您解決方案的效能有何影響，尤其是在 Azure Batch 上執行的處理。(請參閱「步驟 4：建立並執行管線」，以進一步了解**並行**設定。)

3.  建立 [每個 VM 的工作數上限] 較低/較高的集區。更新 Data Factory 解決方案中的 Azure Batch 連結服務，以使用您所建立的新集區。(請參閱「步驟 4：建立並執行管線」，以進一步了解 [每個 VM 的工作數上限] 設定。)

4.  建立具有**自動調整**功能的 Azure Batch 集區。自動調整 Azure Batch 集區中的計算節點就是動態調整應用程式所使用的處理能力。例如，您可以用 0 專用 VM 和依據暫止工作數目自動調整的公式，建立 Azure Batch 集區︰
 
	每個暫止工作一次一個 VM (例如︰5 個暫止工作 -> 5 個 VM)：

		pendingTaskSampleVector=$PendingTasks.GetSample(600 * TimeInterval_Second);
		$TargetDedicated = max(pendingTaskSampleVector);

	無論暫止工作的數目為何，一次最多一個 VM︰

		pendingTaskSampleVector=$PendingTasks.GetSample(600 * TimeInterval_Second);
		$TargetDedicated = (max(pendingTaskSampleVector)>0)?1:0;

	如需詳細資訊，請參閱[自動調整 Azure Batch 集區中的計算節點](../batch/batch-automatic-scaling.md)。

	如果集區使用預設的 [autoScaleEvaluationInterval](https://msdn.microsoft.com/library/azure/dn820173.aspx)，Batch 服務在執行自訂活動之前，可能需要 15-30 分鐘的時間準備 VM。如果集區使用不同的 autoScaleEvaluationInterval，Batch 服務可能需要 autoScaleEvaluationInterval + 10 分鐘。
	 
5. 在範例解決方案中，**Execute** 方法會叫用可處理輸入資料配量以產生輸出資料配量的 **Calculate** 方法。您可以自行撰寫方法來處理輸入資料，然後呼叫您自己的方法，而取代 Execute 方法中的 Calculate 方法呼叫。

 


### 後續步驟：取用資料

處理資料之後，您可以使用 **Microsoft Power BI** 之類的線上工具來取用資料。以下連結可協助您了解 Power BI，以及如何在 Azure 中加以使用：

-   [在 Power BI 中探索資料集](https://powerbi.microsoft.com/zh-TW/documentation/powerbi-service-get-data/)

-   [開始使用 Power BI Desktop](https://powerbi.microsoft.com/zh-TW/documentation/powerbi-desktop-getting-started/)

-   [重新整理 Power BI 中的資料](https://powerbi.microsoft.com/zh-TW/documentation/powerbi-refresh-data/)

-   [Azure 和 Power BI - 基本概觀](https://powerbi.microsoft.com/zh-TW/documentation/powerbi-azure-and-power-bi/)

## 參考

-   [Azure Data Factory](https://azure.microsoft.com/documentation/services/data-factory/)

    -   [Azure Data Factory 服務簡介](data-factory-introduction.md)

    -   [開始使用 Azure Data Factory](data-factory-build-your-first-pipeline.md)

    -   [在 Azure 資料處理站管線中使用自訂活動](data-factory-use-custom-activities.md)

-   [Azure Batch](https://azure.microsoft.com/documentation/services/batch/)

    -   [Azure Batch 的基本概念](../batch/batch-technical-overview.md)

    -   [Azure Batch 功能概觀](../batch/batch-api-basics.md)

    -   [在 Azure 入口網站中建立和管理 Azure Batch 帳戶](../batch/batch-account-create-portal.md)

    -   [開始使用 Azure Batch 程式庫 .NET](../batch/batch-dotnet-get-started.md)


[batch-explorer]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/BatchExplorer
[batch-explorer-walkthrough]: http://blogs.technet.com/b/windowshpc/archive/2015/01/20/azure-batch-explorer-sample-walkthrough.aspx

<!---HONumber=AcomDC_0727_2016-->