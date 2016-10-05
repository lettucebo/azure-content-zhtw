<properties 
   pageTitle="透過 .NET SDK 開始使用 Azure 資料湖分析 | Azure" 
   description="了解如何透過 .NET SDK 建立資料湖存放區帳戶、建立資料湖分析工作，以及提交以 U-SQL 撰寫的工作。" 
   services="data-lake-analytics" 
   documentationCenter="" 
   authors="edmacauley" 
   manager="jhubbard" 
   editor="cgronlun"/>
 
<tags
   ms.service="data-lake-analytics"
   ms.devlang="na"
   ms.topic="hero-article"
   ms.tgt_pltfrm="na"
   ms.workload="big-data" 
   ms.date="09/23/2016"
   ms.author="edmaca"/>

# 教學課程：透過 .NET SDK 開始使用 Azure 資料湖分析

[AZURE.INCLUDE [get-started-selector](../../includes/data-lake-analytics-selector-get-started.md)]


了解如何使用 Azure .NET SDK 將以 [U-SQL](data-lake-analytics-u-sql-get-started.md) 撰寫的作業提交到 Data Lake Analytics。如需有關資料湖分析的詳細資訊，請參閱 [Azure 資料湖分析概觀](data-lake-analytics-overview.md)。

在本教學課程中，您會開發 C# 主控台應用程式來提交 U-SQL 作業，以讀取定位鍵分隔值 (TSV) 檔案，並將該檔案轉換為逗號分隔值 (CSV) 檔案。若要使用其他支援的工具進行同一個教學課程，請按一下本文最上方的索引標籤。

##必要條件

開始進行本教學課程之前，您必須具備下列條件：

- **已安裝 Visual Studio 2015、Visual Studio 2013 更新 4，或具有 Visual C++ 的 Visual Studio 2012**。
- **Microsoft Azure SDK for .NET 2.5 版或更新版本**。使用 [Web Platform Installer](http://www.microsoft.com/web/downloads/platform.aspx) 來進行安裝。
- **Azure 資料湖分析帳戶**。請參閱[使用 Azure .NET SDK 管理 Data Lake Analytics](data-lake-analytics-manage-use-dotnet-sdk.md)。

##建立主控台應用程式

在本教學課程中，您將會處理一些搜尋記錄檔。搜尋記錄檔可以儲存在資料湖存放區或 Azure Blob 儲存體中。

您可以在公用 Azure Blob 容器中找到搜尋記錄檔範例。在應用程式中，您會將該檔案下載到您的工作站，然後再將其上傳到您 Data Lake Analytics 帳戶的預設 Data Lake Store 帳戶中。

**若要建立應用程式**

1. 開啟 Visual Studio。
2. 建立 C# 主控台應用程式。
3. 開啟 Nuget 封裝管理主控台，然後執行下列命令：

        Install-Package Microsoft.Azure.Management.DataLake.Analytics -Pre
        Install-Package Microsoft.Azure.Management.DataLake.Store -Pre
        Install-Package Microsoft.Azure.Management.DataLake.StoreUploader -Pre
        Install-Package Microsoft.Rest.ClientRuntime.Azure.Authentication -Pre
        Install-Package WindowsAzure.Storage

4. 將新檔案新增至名為 **SampleUSQLScript.txt** 的專案，然後貼上下列 U-SQL 指令碼。資料湖分析工作是以 U-SQL 語言撰寫。若要深入了解 U-SQL，請參閱[開始使用 U-SQL 語言](data-lake-analytics-u-sql-get-started.md)和 [U-SQL 語言參考](http://go.microsoft.com/fwlink/?LinkId=691348)。

        @searchlog =
            EXTRACT UserId          int,
                    Start           DateTime,
                    Region          string,
                    Query           string,
                    Duration        int?,
                    Urls            string,
                    ClickedUrls     string
            FROM "/Samples/Data/SearchLog.tsv"
            USING Extractors.Tsv();
        
        OUTPUT @searchlog   
            TO "/Output/SearchLog-from-Data-Lake.csv"
        USING Outputters.Csv();

	此 U-SQL 指令碼會使用 **Extractors.Tsv()** 讀取來源資料檔案，然後使用 **Outputters.Csv()** 建立 csv 檔案。
    
    在 C# 程式中，您必須準備 **/Samples/Data/SearchLog.tsv** 檔案及 **/Output/** 資料夾：
	
	使用儲存在預設資料湖帳戶中檔案的相對路徑，是比較容易的方法。您也可以使用絕對路徑。例如
    
        adl://<Data LakeStorageAccountName>.azuredatalakestore.net:443/Samples/Data/SearchLog.tsv
        
    您必須使用絕對路徑存取連結儲存體帳戶中的檔案。儲存在連結 Azure 儲存體帳戶中之檔案的語法是：
    
        wasb://<BlobContainerName>@<StorageAccountName>.blob.core.windows.net/Samples/Data/SearchLog.tsv

	>[AZURE.NOTE] 目前沒有任何關於使用 Azure Data Lake 服務的已知問題。如果範例應用程式已中斷或發生錯誤，您可能需要手動刪除指令碼所建立的 Data Lake Store 和 Data Lake Analytics 帳戶。如果您不熟悉入口網站，[使用 Azure 入口網站管理 Azure Data Lake Analytics](data-lake-analytics-manage-use-portal.md) 指南可協助您開始。
       
5. 在 Program.cs 中貼上下列程式碼：

		using System;
		using System.IO;
		using System.Collections.Generic;
		using System.Threading;
		using Microsoft.Rest;
		using Microsoft.Rest.Azure.Authentication;
		using Microsoft.Azure.Management.DataLake.Store;
		using Microsoft.Azure.Management.DataLake.StoreUploader;
		using Microsoft.Azure.Management.DataLake.Analytics;
		using Microsoft.Azure.Management.DataLake.Analytics.Models;
		using Microsoft.WindowsAzure.Storage.Blob;

		namespace SdkSample
		{
		  class Program
		  {
			private const string SUBSCRIPTIONID = "<Enter Your Azure Subscription ID>";
			private const string CLIENTID = "1950a258-227b-4e31-a9cf-717495945fc2";
			private const string DOMAINNAME = "common"; // Replace this string with the user's Azure Active Directory tenant ID or domain name, if needed.

			private static string _adlaAccountName = "<Enter an Existing Data Lake Analytics Account Name>";
			private static string _adlsAccountName = "<Enter the default Data Lake Store Account Name>";

			private static DataLakeAnalyticsAccountManagementClient _adlaClient;
			private static DataLakeStoreFileSystemManagementClient _adlsFileSystemClient;
			private static DataLakeAnalyticsJobManagementClient _adlaJobClient;
		
		    private static void Main(string[] args)
		    {
				string localFolderPath = @"c:\temp";

				// Connect to Azure
				var creds = AuthenticateAzure(DOMAINNAME, CLIENTID);

				SetupClients(creds, SUBSCRIPTIONID);

				// Transfer the source file from a public Azure Blob container to Data Lake Store.
				CloudBlockBlob blob = new CloudBlockBlob(new Uri("https://adltutorials.blob.core.windows.net/adls-sample-data/SearchLog.tsv"));
				blob.DownloadToFile(localFolderPath + "SearchLog.tsv", FileMode.Create); // from WASB
				UploadFile(localFolderPath + "SearchLog.tsv", "/Samples/Data/SearchLog.tsv"); // to ADLS
				WaitForNewline("Source data file prepared.", "Submitting a job.");


				// Submit the job
				Guid jobId = SubmitJobByPath(localFolderPath + "SampleUSQLScript.txt", "My First ADLA Job");
				WaitForNewline("Job submitted.", "Waiting for job completion.");

				// Wait for job completion
				WaitForJob(jobId);
				WaitForNewline("Job completed.", "Downloading job output.");

				// Download job output
				DownloadFile(@"/Output/SearchLog-from-Data-Lake.csv", localFolderPath + "SearchLog-from-Data-Lake.csv");
		
		      	WaitForNewline("Job output downloaded. You can now exit.");
		    }
		
			public static ServiceClientCredentials AuthenticateAzure(
				string domainName,
				string nativeClientAppCLIENTID)
			{
				// User login via interactive popup
				SynchronizationContext.SetSynchronizationContext(new SynchronizationContext());
				// Use the client ID of an existing AAD "Native Client" application.
				var activeDirectoryClientSettings = ActiveDirectoryClientSettings.UsePromptOnly(nativeClientAppCLIENTID, new Uri("urn:ietf:wg:oauth:2.0:oob"));
				return UserTokenProvider.LoginWithPromptAsync(domainName, activeDirectoryClientSettings).Result;
			}

			public static void SetupClients(ServiceClientCredentials tokenCreds, string subscriptionId)
			{
				_adlaClient = new DataLakeAnalyticsAccountManagementClient(tokenCreds);
				_adlaClient.SubscriptionId = subscriptionId;

				_adlaJobClient = new DataLakeAnalyticsJobManagementClient(tokenCreds);

				_adlsFileSystemClient = new DataLakeStoreFileSystemManagementClient(tokenCreds);
			}

			public static void UploadFile(string srcFilePath, string destFilePath, bool force = true)
			{
				var parameters = new UploadParameters(srcFilePath, destFilePath, _adlsAccountName, isOverwrite: force);
				var frontend = new DataLakeStoreFrontEndAdapter(_adlsAccountName, _adlsFileSystemClient);
				var uploader = new DataLakeStoreUploader(parameters, frontend);
				uploader.Execute();
			}

			public static void DownloadFile(string srcPath, string destPath)
			{
				var stream = _adlsFileSystemClient.FileSystem.Open(_adlsAccountName, srcPath);
				var fileStream = new FileStream(destPath, FileMode.Create);

				stream.CopyTo(fileStream);
				fileStream.Close();
				stream.Close();
			}

			// Helper function to show status and wait for user input
			public static void WaitForNewline(string reason, string nextAction = "")
			{
				Console.WriteLine(reason + "\r\nPress ENTER to continue...");

				Console.ReadLine();

				if (!String.IsNullOrWhiteSpace(nextAction))
					Console.WriteLine(nextAction);
			}


			// List all Data Lake Analytics accounts within the subscription
			public static List<DataLakeAnalyticsAccount> ListADLAAccounts()
			{
				var response = _adlaClient.Account.List();
				var accounts = new List<DataLakeAnalyticsAccount>(response);

				while (response.NextPageLink != null)
				{
					response = _adlaClient.Account.ListNext(response.NextPageLink);
					accounts.AddRange(response);
				}

				Console.WriteLine("You have %i Data Lake Analytics account(s).", accounts.Count);
				for (int i = 0; i < accounts.Count; i++)
				{
					Console.WriteLine(accounts[i].Name);
				}

				return accounts;
			}
			public static Guid SubmitJobByPath(string scriptPath, string jobName)
			{
				var script = File.ReadAllText(scriptPath);

				var jobId = Guid.NewGuid();
				var properties = new USqlJobProperties(script);
				var parameters = new JobInformation(jobName, JobType.USql, properties, priority: 1, degreeOfParallelism: 1, jobId: jobId);
				var jobInfo = _adlaJobClient.Job.Create(_adlaAccountName, jobId, parameters);

				return jobId;
			}

			public static JobResult WaitForJob(Guid jobId)
			{
				var jobInfo = _adlaJobClient.Job.Get(_adlaAccountName, jobId);
				while (jobInfo.State != JobState.Ended)
				{
					jobInfo = _adlaJobClient.Job.Get(_adlaAccountName, jobId);
				}
				return jobInfo.Result.Value;
			}
		  }
		}

6. 按 **F5** 鍵執行應用程式。輸出如下：

	![Azure Data Lake Analytics 作業 U-SQL .NET SDK 輸出](./media/data-lake-analytics-get-started-net-sdk/data-lake-analytics-dotnet-job-output.png)

7. 查看輸出檔案。預設路徑和檔案名稱是 c:\\Temp\\SearchLog-from-Data-Lake.csv。

## 另請參閱

- 若要使用其他工具檢視同一個教學課程，請按一下頁面最上方的索引標籤選取器。
- 若要了解更複雜的查詢，請參閱[使用 Azure 資料湖分析來分析網站記錄檔](data-lake-analytics-analyze-weblogs.md)。
- 若要開始開發 U-SQL 應用程式，請參閱[使用適用於 Visual Studio 的資料湖工具開發 U-SQL 指令碼](data-lake-analytics-data-lake-tools-get-started.md)。
- 若要了解 U-SQL，請參閱[開始使用 Azure Data Lake Analytics U-SQL 語言](data-lake-analytics-u-sql-get-started.md)和 [U-SQL 語言參考](http://go.microsoft.com/fwlink/?LinkId=691348)。
- 針對管理工作，請參閱[使用 Azure 入口網站管理 Azure Data Lake Analytics](data-lake-analytics-manage-use-portal.md)。
- 若要取得資料湖分析概觀，請參閱 [Azure 資料湖分析概觀](data-lake-analytics-overview.md)。

<!---HONumber=AcomDC_0928_2016-->