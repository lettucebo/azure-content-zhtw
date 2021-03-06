<properties
	pageTitle="開始使用佇列儲存體和 Visual Studio 已連接服務 (ASP.NET 5) | Microsoft Azure"
	description="如何開始在 Visual Studio 的 ASP.NET 5 專案中使用 Azure 佇列儲存體"
	services="storage"
	documentationCenter=""
	authors="TomArcher"
	manager="douge"
	editor=""/>

<tags
	ms.service="storage"
	ms.workload="web"
	ms.tgt_pltfrm="vs-getting-started"
	ms.devlang="na"
	ms.topic="article"
	ms.date="07/18/2016"
	ms.author="tarcher"/>

# 開始使用佇列儲存體和 Visual Studio 已連接服務 (ASP.NET 5)

[AZURE.INCLUDE [storage-try-azure-tools-queues](../../includes/storage-try-azure-tools-queues.md)]

##概觀

本文說明如何在您使用 Visual Studio 的 [新增連接的服務] 對話方塊建立或參考了 ASP.NET 5 專案中的 Azure 儲存體帳戶之後，開始在 Visual Studio 使用 Azure 佇列儲存體。[新增連接的服務] 作業會安裝適當的 NuGet 套件，以存取專案中的 Azure 儲存體，並將儲存體帳戶的連接字串新增至您的專案設定檔。

Azure 佇列儲存體是一項儲存大量訊息的服務，全球任何地方都可利用 HTTP 或 HTTPS 並透過驗證的呼叫來存取這些訊息。單一佇列訊息的大小上限為 64 KB，而一個佇列可以包含數百萬個訊息，以儲存體帳戶的總容量為限。

若要開始，首先您必須在儲存體帳戶中建立 Azure 佇列。我們將會示範如何在程式碼中建立佇列。我們也將顯示如何執行基本的佇列作業，例如新增、修改、讀取和移除佇列訊息。這些範例均以 C# 程式碼撰寫，並使用 Azure Storage Client Library for .NET。如需 ASP.NET 的詳細資訊，請參閱 [ASP.NET](http://www.asp.net)。

**注意：**有一些 API會以非同步方式對 ASP.NET 5 中 Azure 儲存體執行呼叫。如需詳細資訊，請參閱[使用 Async 及 Await 進行非同步程式設計](http://msdn.microsoft.com/library/hh191443.aspx)。以下程式碼假設使用非同步程式設計方法。

- 如需以程式設計方式處理佇列的詳細資訊，請參閱[以 .NET 開始使用 Azure 佇列儲存體](storage-dotnet-how-to-use-queues.md)。
- 如需 Azure 儲存體的一般資訊，請參閱[儲存體文件](https://azure.microsoft.com/documentation/services/storage/)。
- 如需 Azure 雲端服務的一般資訊，請參閱[雲端服務文件](https://azure.microsoft.com/documentation/services/cloud-services/)。
- 若需要如何編寫 ASP.NET 應用程式的詳細資訊，請參閱 [ASP.NET](http://www.asp.net)。





##在程式碼中存取佇列

若要存取 ASP.NET 5 專案中的佇列，您需要將下列項目併入至存取 Azure 佇列儲存體的任何 C# 原始程式檔。

1. 請確定 C# 檔案頂端的命名空間宣告包含這些 **using** 陳述式。

		using Microsoft.Framework.Configuration;
		using Microsoft.WindowsAzure.Storage;
		using Microsoft.WindowsAzure.Storage.Queue;
		using System.Threading.Tasks;
		using LogLevel = Microsoft.Framework.Logging.LogLevel;

2. 取得 **CloudStorageAccount** 物件，其代表您的儲存體帳戶資訊。使用下列程式碼，從 Azure 服務組態取得您的儲存體連接字串和儲存體帳戶資訊。

		 CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
		   CloudConfigurationManager.GetSetting("<storage-account-name>_AzureStorageConnectionString"));

3. 取得 **CloudQueueClient** 物件以參考儲存體帳戶中的佇列物件。

	    // Create the CloudQueueClient object for the storage account.
    	CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

4. 取得 **CloudQueue** 物件以參考特定的佇列。

    	// Get a reference to the CloudQueue named "messageQueue"
	    CloudQueue messageQueue = queueClient.GetQueueReference("messageQueue");


**注意：**請在下列範例中的程式碼前面使用上述所有程式碼。

###在程式碼中建立佇列

若要在程式碼中建立 Azure 佇列，請加入 **CreateIfNotExistsAsync** 呼叫。

	// Create the CloudQueue if it does not exist.
	await messageQueue.CreateIfNotExistsAsync();

##將訊息新增至佇列

若要將訊息插入現有佇列中，請建立新的 **CloudQueueMessage** 物件，然後呼叫 **AddMessageAsync** 方法。

您可以從字串 (採用 UTF-8 格式) 或位元組陣列建立 **CloudQueueMessage** 物件。

以下是插入訊息 'Hello, World' 的範例。

	// Create a message and add it to the queue.
	CloudQueueMessage message = new CloudQueueMessage("Hello, World");
	await messageQueue.AddMessageAsync(message);

##讀取佇列中的訊息

您只要呼叫 **PeekMessageAsync** 方法，就能在佇列前查看訊息，而無須將其從佇列中移除。

	// Peek the next message in the queue. 
	CloudQueueMessage peekedMessage = await messageQueue.PeekMessageAsync();


##讀取並移除佇列中的訊息

您的程式碼可以使用兩個步驟將訊息從佇列中移除 (清除佇列)。
1. 呼叫 **GetMessageAsync**，以取得佇列中的下一則訊息。任何從此佇列讀取訊息的其他程式碼，將無法看到從 **GetMessageAsync** 傳回的訊息。依預設，此訊息會維持 30 秒的不可見狀態。
2.	若要完成從佇列中移除訊息，請呼叫 **DeleteMessageAsync**。

這個移除訊息的兩步驟程序可確保您的程式碼因為硬體或軟體故障而無法處理訊息時，另一個程式碼的執行個體可以取得相同訊息並再試一次。下列程式碼會在處理完訊息之後隨即呼叫 **DeleteMessageAsync**。

	// Get the next message in the queue.
	CloudQueueMessage retrievedMessage = await messageQueue.GetMessageAsync();

	// Process the message in less than 30 seconds.

    // Then delete the message.
	await messageQueue.DeleteMessageAsync(retrievedMessage);

## 運用清除佇列訊息的其他選項

自訂從佇列中擷取訊息的方法有兩種。首先，您可以取得一批訊息 (最多 32 個)。其次，您可以設定較長或較短的可見度逾時，讓您的程式碼有較長或較短的時間可以完全處理每個訊息。下列程式碼範例將使用 **GetMessages** 方法，在一次呼叫中取得 20 個訊息。接著它會使用 **foreach** 迴圈處理每個訊息。它也會將可見度逾時設定為每個訊息 5 分鐘。請注意，所有訊息皆會同時開始計時 5 分鐘，當呼叫 **GetMessages** 後的 5 分鐘一到，所有尚未刪除的訊息皆會重新出現。

    // Retrieve 20 messages at a time, keeping those messages invisible for 5 minutes, 
    //   delete each message after processing.

    foreach (CloudQueueMessage message in messageQueue.GetMessages(20, TimeSpan.FromMinutes(5)))
    {
        // Process all messages in less than 5 minutes, deleting each message after processing.
        queue.DeleteMessage(message);
    }

## 取得佇列長度

您可以取得佇列中的估計訊息數目。**FetchAttributes** 方法會要求佇列服務擷取佇列屬性，其中包含訊息計數。**ApproximateMethodCount** 屬性無須呼叫佇列服務，就會傳回 **FetchAttributes** 方法所擷取的最後一個值。

	// Fetch the queue attributes.
	messageQueue.FetchAttributes();

    // Retrieve the cached approximate message count.
    int? cachedMessageCount = messageQueue.ApproximateMessageCount;

	// Display the number of messages.
	Console.WriteLine("Number of messages in queue: " + cachedMessageCount);

## 搭配使用 Async-Await 模式和通用佇列 API

這個範例示範如何搭配使用 Async-Await 模式和通用佇列 API。此範例會呼叫每個特定方法的非同步版本。這可以透過每個方法的非同步 Postfix 看到。使用非同步方法時，async-await 模式會暫停本機執行，直到呼叫完成為止。這種行為可讓目前的執行緒執行其他工作，有助於避免發生效能瓶頸並提升應用程式的整體回應。如需如何在 .NET 中使用 Async-Await 模式的詳細資訊，請參閱 [Async 和 Await (C# 和 Visual Basic)](https://msdn.microsoft.com/library/hh191443.aspx)

    // Create a message to add to the queue.
    CloudQueueMessage cloudQueueMessage = new CloudQueueMessage("My message");

    // Async enqueue the message.
    await messageQueue.AddMessageAsync(cloudQueueMessage);
    Console.WriteLine("Message added");

    // Async dequeue the message.
    CloudQueueMessage retrievedMessage = await messageQueue.GetMessageAsync();
    Console.WriteLine("Retrieved message with content '{0}'", retrievedMessage.AsString);

    // Async delete the message.
    await messageQueue.DeleteMessageAsync(retrievedMessage);
    Console.WriteLine("Deleted message");
## 刪除佇列

若要刪除佇列及其內含的所有訊息，請在佇列物件上呼叫 **Delete** 方法。

    // Delete the queue.
    messageQueue.Delete();


##後續步驟

[AZURE.INCLUDE [vs-storage-dotnet-queues-next-steps](../../includes/vs-storage-dotnet-queues-next-steps.md)]

<!---HONumber=AcomDC_0727_2016-->