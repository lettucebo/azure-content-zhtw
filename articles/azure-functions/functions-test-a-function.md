<properties
   pageTitle="測試 Azure Functions | Microsoft Azure"
   description="使用 Postman、cURL、和 Node.js 來測試您的 Azure Functions。"
   services="functions"
   documentationCenter="na"
   authors="wesmc7777"
   manager="erikre"
   editor=""
   tags=""
   keywords="azure functions, 函數, 事件處理, webhook, 動態計算, 無伺服器架構, 測試"/>

<tags
   ms.service="functions"
   ms.devlang="multiple"
   ms.topic="article"
   ms.tgt_pltfrm="multiple"
   ms.workload="na"
   ms.date="08/19/2016"
   ms.author="wesmc"/>

# 測試 Azure Functions

## Overview

在本教學課程中，我們將逐步進行測試函數的不同方法。我們會定義會透過查詢字串參數或要求主體接受輸入的 http 觸發程序函數。預設 **HttpTrigger Node.js 函數**範本程式碼支援 `name` 查詢字串參數。我們也會新增程式碼來支援該參數，連同要求主體中使用者的 `address` 資訊。

## 建立用於測試的函數

本教學課程中的大多數時間，我們將使用建立新的函數時可以使用 **HttpTrigger Nodejs 函數**範本稍經修改的版本。如果您需要建立新函數的說明，可以檢閱[建立您的第一個 Azure 函數](functions-create-first-azure-function.md)。在 [Azure 入口網站]中建立測試函數時，只要選擇 **HttpTrigger Nodejs 函數**範本。

預設函式範本基本上是 hello world 函式，可從要求主體或查詢字串參數 `name=<your name>` 回應名稱。我們將更新程式碼，以讓您在要求主體中以 JSON 內容的形式提供名稱和位址。然後函數會在可使用時將這些內容回應給用戶端。

使用我們將用來測試的下列程式碼更新函數︰

	module.exports = function(context, req) {
	    context.log("Node.js HTTP trigger function processed a request. RequestUri=%s", req.originalUrl);
	    context.log("Request Headers = " + JSON.stringify(req.headers));    
	
	    if (req.query.name || (req.body && req.body.name)) {
	        if (typeof req.query.name != "undefined") {
	            context.log("Name was provided as a query string param..."); 
	            ProcessNewUserInformation(context, req.query.name);
	        }
	        else {
	            context.log("Processing user info from request body..."); 
	            ProcessNewUserInformation(context, req.body.name, req.body.address);
	        }
	    }
	    else {
	        context.res = {
	            status: 400,
	            body: "Please pass a name on the query string or in the request body"
	        };
	    }
	    context.done();
	};
	
	function ProcessNewUserInformation(context, name, address)
	{    
	    context.log("Processing User Information...");            
	    context.log("name = " + name);            
	    echoString = "Hello " + name;
	    
	    if (typeof address != "undefined")
	    {
	        echoString += "\n" + "The address you provided is " + address;
	        context.log("address = " + address);            
	    }
	    
	    context.res = {
	            // status: 200, /* Defaults to 200 */
	            body: echoString
	        };
	}


## 使用工具測試函數

### 使用 cURL 測試

測試軟體時往往只需要命令列即可協助偵錯您的應用程式，對於函數也是如此。

若要測試上述函數，請從入口網站複製**函數 URL**。它將具有下列格式：

    https://<Your Function App>.azurewebsites.net/api/<Your Function Name>?code=<your access code>
    
這是觸發程式函數的 URL，我們可以在命令列上使用 cURL 命令來測試它，以便對我們的函數取得 (`-G` 或 `--get`) 要求︰

    curl -G https://<Your Function App>.azurewebsites.net/api/<Your Function Name>?code=<your access code>
    
以上這個特定範例需要在 cURL 命令中將查詢字串參數傳遞做為資料 (`-d`)︰

    curl -G https://<Your Function App>.azurewebsites.net/api/<Your Function Name>?code=<your access code> -d name=<Enter a name here>
    
按 Enter 鍵，您會在命令列上看到函數的輸出。

![](./media/functions-test-a-function/curl-test.png)

在入口網站 [記錄] 視窗中，執行函數時會記錄類似下面的輸出︰

    2016-04-05T21:55:09  Welcome, you are now connected to log-streaming service.
    2016-04-05T21:55:30.738 Function started (Id=ae6955da-29db-401a-b706-482fcd1b8f7a)
    2016-04-05T21:55:30.738 Node.js HTTP trigger function processed a request. RequestUri=https://functionsExample.azurewebsites.net/api/HttpTriggerNodeJS1?code=XXXXXXX&name=Azure Functions
    2016-04-05T21:55:30.738 Function completed (Success, Id=ae6955da-29db-401a-b706-482fcd1b8f7a)

### 使用瀏覽器測試

不需要參數或只需要查詢字串參數的函數，即可以使用瀏覽器測試。

若要測試上述定義的函數，請從入口網站複製**函數 URL**。它將具有下列格式：

	https://<Your Function App>.azurewebsites.net/api/<Your Function Name>?code=<your access code>

如下所示，附加 `name` 查詢字串參數，對 `<Enter a name here>` 預留位置使用實際名稱。

	https://<Your Function App>.azurewebsites.net/api/<Your Function Name>?code=<your access code>&name=<Enter a name here>

將此 URL 貼入瀏覽器，應該會得到預似以下的回應。

![](./media/functions-test-a-function/browser-test.png)

在入口網站 [記錄] 視窗中，執行函數時會記錄類似下面的輸出︰

	2016-03-23T07:34:59  Welcome, you are now connected to log-streaming service.
	2016-03-23T07:35:09.195 Function started (Id=61a8c5a9-5e44-4da0-909d-91d293f20445)
	2016-03-23T07:35:10.338 Node.js HTTP trigger function processed a request. RequestUri=https://functionsExample.azurewebsites.net/api/WesmcHttpTriggerNodeJS1?code=XXXXXXXXXX==&name=Wes from a browser
	2016-03-23T07:35:10.338 Request Headers = {"cache-control":"max-age=0","connection":"Keep-Alive","accept":"text/html","accept-encoding":"gzip","accept-language":"zh-TW"}
	2016-03-23T07:35:10.338 Name was provided as a query string param.
	2016-03-23T07:35:10.338 Processing User Information...
	2016-03-23T07:35:10.369 Function completed (Success, Id=61a8c5a9-5e44-4da0-909d-91d293f20445)

### 使用 Postman 測試

用來測試您的多數函數所建議的工具是 Postman。若要安裝 Postman，請參閱[取得 Postman](https://www.getpostman.com/)。Postman 可對 HTTP 要求的許多其他屬性提供控制。

> [AZURE.TIP] 方便時請使用 REST 用戶端。以下是 Postman 的一些替代方案︰
> 
> * [Fiddler](http://www.telerik.com/fiddler)
> * [Paw](https://luckymarmot.com/paw)

若要在 Postman 中測試要求主體的函數︰

1. 從 Chrome 瀏覽器視窗左上角的**應用程式**按鈕啟動 Postman。
2. 複製您的 **函數 Url**，並將它貼到 Postman 中。它包含存取程式碼查詢字串參數。
3. 將 HTTP 方法變更為 **POST**。
4. 按一下 [主體] > [原始]，並新增類似以下的 JSON 要求主體︰

		{
		    "name" : "Wes testing with Postman",
		    "address" : "Seattle, W.A. 98101"
		}

5. 按一下 [傳送]。

下圖顯示在本教學課程中測試簡單的回應函數範例。

![](./media/functions-test-a-function/postman-test.png)

在入口網站 [記錄] 視窗中，執行函數時會記錄類似下面的輸出︰

	2016-03-23T08:04:51  Welcome, you are now connected to log-streaming service.
	2016-03-23T08:04:57.107 Function started (Id=dc5db8b1-6f1c-4117-b5c4-f6b602d538f7)
	2016-03-23T08:04:57.763 Node.js HTTP trigger function processed a request. RequestUri=https://functions841def78.azurewebsites.net/api/WesmcHttpTriggerNodeJS1?code=XXXXXXXXXX==
	2016-03-23T08:04:57.763 Request Headers = {"cache-control":"no-cache","connection":"Keep-Alive","accept":"*/*","accept-encoding":"gzip","accept-language":"zh-TW"}
	2016-03-23T08:04:57.763 Processing user info from request body...
	2016-03-23T08:04:57.763 Processing User Information...
	2016-03-23T08:04:57.763 name = Wes testing with Postman
	2016-03-23T08:04:57.763 address = Seattle, W.A. 98101
	2016-03-23T08:04:57.795 Function completed (Success, Id=dc5db8b1-6f1c-4117-b5c4-f6b602d538f7)
    
### 使用儲存體總管測試 blob 觸發程序

您可以使用 [Microsoft Azure 儲存體總管](http://storageexplorer.com/)來測試 blob 觸發程序函數。

1. 在您的 Functions 應用程式的 [Azure 入口網站]中，建立新的 C#、F# 或 Node blob 觸發程序函式。將要監視的路徑設定為您的 blob 容器名稱。例如：

		files

2. 按一下 **+** 按鈕以選取或建立您想要使用的儲存體帳戶。然後按一下 [**建立**]。

3. 建立含有下列內容的文字檔，並儲存該檔案：

		A text file for blob trigger function testing.

4. 執行 [Microsoft Azure 儲存體總管](http://storageexplorer.com/)，並連接要監視的儲存體帳戶中的 blob 容器。

5. 按一下 [上傳] 按鈕，然後上傳文字檔案。

	![](./media/functions-test-a-function/azure-storage-explorer-test.png)


	預設 blob 觸發程序的函數程式碼將會在記錄檔中報告 blob 的處理：

		2016-03-24T11:30:10  Welcome, you are now connected to log-streaming service.
		2016-03-24T11:30:34.472 Function started (Id=739ebc07-ff9e-4ec4-a444-e479cec2e460)
		2016-03-24T11:30:34.472 C# Blob trigger function processed: A text file for blob trigger function testing.
		2016-03-24T11:30:34.472 Function completed (Success, Id=739ebc07-ff9e-4ec4-a444-e479cec2e460)

## 在 Functions 內測試函數

### 使用 Functions 入口網站的執行按鈕測試

入口網站提供的**執行**按鈕可讓您執行一些有限的測試。您可以使用 [執行] 按鈕提供要求主體，但無法提供查詢字串參數或更新要求標頭。

藉由在 [要求主體] 欄位中新增類似以下的 JSON 字串，然後按一下 [執行] 按鈕，來測試我們稍早建立的 HTTP 觸發程序函數。

	{
		"name" : "Wes testing Run button",
		"address" : "USA"
	} 

在入口網站 [記錄] 視窗中，執行函數時會記錄類似下面的輸出︰

	2016-03-23T08:03:12  Welcome, you are now connected to log-streaming service.
	2016-03-23T08:03:17.357 Function started (Id=753a01b0-45a8-4125-a030-3ad543a89409)
	2016-03-23T08:03:18.697 Node.js HTTP trigger function processed a request. RequestUri=https://functions841def78.azurewebsites.net/api/wesmchttptriggernodejs1
	2016-03-23T08:03:18.697 Request Headers = {"connection":"Keep-Alive","accept":"*/*","accept-encoding":"gzip","accept-language":"zh-TW"}
	2016-03-23T08:03:18.697 Processing user info from request body...
	2016-03-23T08:03:18.697 Processing User Information...
	2016-03-23T08:03:18.697 name = Wes testing Run button
	2016-03-23T08:03:18.697 address = USA
	2016-03-23T08:03:18.744 Function completed (Success, Id=753a01b0-45a8-4125-a030-3ad543a89409)


### 使用計時器觸發程序測試

有些函數，實際上無法使用先前所述的工具進行測試。例如，將訊息放入 [Azure 佇列儲存體](../storage/storage-dotnet-how-to-use-queues.md)時執行的佇列觸發程序函數。您一律可以撰寫程式碼來將訊息放入佇列，以下提供在主控台專案中進行此動作的範例。不過，還有另一種方法可用來直接使用函數進行測試。

您可以使用設定了佇列輸出繫結的計時器觸發程序。該計時器觸發程序程式碼會接著將測試訊息寫入至佇列。本節將逐步解說範例。

如需使用 Azure Function 搭配使用繫結的深入資訊，請參閱 [Azure Functions 開發人員參考](functions-reference.md)。


#### 建立用於測試的佇列觸發程序

為了示範此方法，我們會先建立要對名為 `queue-newusers` 的佇列進行測試的佇列觸發程序函數。此函數會處理放入 Azure 佇列儲存體的新使用者的名稱和位址資訊。

> [AZURE.NOTE] 如果您使用不同的佇列名稱，請確定您使用的名稱符合[命名佇列和中繼資料](https://msdn.microsoft.com/library/dd179349.aspx)的規則。否則，您會得到的 HTTP 狀態碼 400︰錯誤的要求。

1. 在 Functions 應用程式的 [Azure 入口網站]，按一下 [新增函數] > [QueueTrigger - C#]。
2. 輸入要使用佇列函式監視的佇列名稱

		queue-newusers 

3. 按一下 **+** (新增) 按鈕來選取或建立您想要使用的儲存體帳戶。然後按一下 [**建立**]。
4. 將此入口網站瀏覽器視窗保持開啟，使得您可以監視預設的佇列函數範本程式碼的記錄項目。


#### 建立要將訊息放在佇列中的計時器觸發程序

1. 在新的瀏覽器視窗中開啟 [Azure 入口網站]，並瀏覽至您的 Function 應用程式。
2. 按一下 [新增函數] > [TimerTrigger - C#]。輸入 cron 運算式來設定計時器程式碼會執行測試您的佇列函數的頻率。然後按一下 [**建立**]。如果您想要每隔 30 秒執行測試，您可以使用下列 [CRON 運算式](https://wikipedia.org/wiki/Cron#CRON_expression)：

		*/30 * * * * *


2. 按一下新計時器觸發程序的 [整合] 索引標籤。
3. 在 [輸出] 下，按一下 [+ 新輸出] 按鈕。然後按一下 [佇列] 和 [選取] 按鈕。
4. 記下您要用於**佇列訊息物件**的名稱，您將在此計時器函數程式碼中使用該名稱。

		myQueue

4. 輸入會傳送訊息的目標佇列名稱︰

		queue-newusers 

3. 按一下 **+** (新增) 按鈕以選取您先前用於佇列觸發程序的儲存體帳戶。然後按一下 [儲存]。
4. 按一下計時器觸發程序的 [開發] 索引標籤。
5. 只要使用如上所示的相同佇列訊息物件名稱，您即可以對 C# 計時器函數使用下列程式碼。然後按一下 [儲存]

		using System;
		
		public static void Run(TimerInfo myTimer, out String myQueue, TraceWriter log)
		{
		    String newUser = 
		    "{"name":"User testing from C# timer function","address":"XYZ"}";
		
		    log.Verbose($"C# Timer trigger function executed at: {DateTime.Now}");   
		    log.Verbose($"{newUser}");   
		    
		    myQueue = newUser;
		}

目前，如果您已使用範例 cron 運算式，C# 計時器函數會每隔 30 秒執行。計時器函數的記錄檔會報告每次執行︰

	2016-03-24T10:27:02  Welcome, you are now connected to log-streaming service.
	2016-03-24T10:27:30.004 Function started (Id=04061790-974f-4043-b851-48bd4ac424d1)
	2016-03-24T10:27:30.004 C# Timer trigger function executed at: 3/24/2016 10:27:30 AM
	2016-03-24T10:27:30.004 {"name":"User testing from C# timer function","address":"XYZ"}
	2016-03-24T10:27:30.004 Function completed (Success, Id=04061790-974f-4043-b851-48bd4ac424d1)

在佇列函數的瀏覽器視窗中，您會看到正在處理的每個訊息︰

	2016-03-24T10:27:06  Welcome, you are now connected to log-streaming service.
	2016-03-24T10:27:30.607 Function started (Id=e304450c-ff48-44dc-ba2e-1df7209a9d22)
	2016-03-24T10:27:30.607 C# Queue trigger function processed: {"name":"User testing from C# timer function","address":"XYZ"}
	2016-03-24T10:27:30.607 Function completed (Success, Id=e304450c-ff48-44dc-ba2e-1df7209a9d22)
	
## 使用程式碼測試函數

### 使用 Node.js 程式碼測試 HTTP 觸發程序函數

您可以使用 Node.js 程式碼來執行 http 要求，以測試您的 Azure Function。

請務必設定：

- 要求選項中的 `host` 設為您的 Function 應用程式主機
- 在 `path` 中設定您的函數名稱。
- 在 `path` 中設定您的存取程式碼 (`<your code>`)。

程式碼範例：

	var http = require("http");
	
	var nameQueryString = "name=Wes%20Query%20String%20Test%20From%20Node.js";
	
	var nameBodyJSON = {
	    name : "Wes testing with Node.JS code",
	    address : "Dallas, T.X. 75201"
	};
	
	var bodyString = JSON.stringify(nameBodyJSON);
	
	var options = {
	  host: "functions841def78.azurewebsites.net",
	  //path: "/api/HttpTriggerNodeJS2?code=sc1wt62opn7k9buhrm8jpds4ikxvvj42m5ojdt0p91lz5jnhfr2c74ipoujyq26wab3wk5gkfbt9&" + nameQueryString,
	  path: "/api/HttpTriggerNodeJS2?code=sc1wt62opn7k9buhrm8jpds4ikxvvj42m5ojdt0p91lz5jnhfr2c74ipoujyq26wab3wk5gkfbt9",
	  method: "POST",
	  headers : {
	      "Content-Type":"application/json",
	      "Content-Length": Buffer.byteLength(bodyString)
	    }    
	};
	
	callback = function(response) {
	  var str = ""
	  response.on("data", function (chunk) {
	    str += chunk;
	  });
	
	  response.on("end", function () {
	    console.log(str);
	  });
	}
	
	var req = http.request(options, callback);
	console.log("*** Sending name and address in body ***");
	console.log(bodyString);
	req.end(bodyString);



輸出：

	C:\Users\Wesley\testing\Node.js>node testHttpTriggerExample.js
	*** Sending name and address in body ***
	{"name" : "Wes testing with Node.JS code","address" : "Dallas, T.X. 75201"}
	Hello Wes testing with Node.JS code
	The address you provided is Dallas, T.X. 75201
		
在入口網站 [記錄] 視窗中，執行函數時會記錄類似下面的輸出︰

	2016-03-23T08:08:55  Welcome, you are now connected to log-streaming service.
	2016-03-23T08:08:59.736 Function started (Id=607b891c-08a1-427f-910c-af64ae4f7f9c)
	2016-03-23T08:09:01.153 Node.js HTTP trigger function processed a request. RequestUri=http://functionsExample.azurewebsites.net/api/WesmcHttpTriggerNodeJS1/?code=XXXXXXXXXX==
	2016-03-23T08:09:01.153 Request Headers = {"connection":"Keep-Alive","host":"functionsExample.azurewebsites.net"}
	2016-03-23T08:09:01.153 Name not provided as query string param. Checking body...
	2016-03-23T08:09:01.153 Request Body Type = object
	2016-03-23T08:09:01.153 Request Body = [object Object]
	2016-03-23T08:09:01.153 Processing User Information...
	2016-03-23T08:09:01.215 Function completed (Success, Id=607b891c-08a1-427f-910c-af64ae4f7f9c)
    

### 使用 C# 程式碼測試佇列觸發程序函數 #

我們稍早提到，您可以透過使用程式碼在您的佇列中放置訊息來測試佇列觸發程序。下列範例程式碼中是基於[開始使用 Azure 佇列儲存體](../storage/storage-dotnet-how-to-use-queues.md)教學課程中提供的 C# 程式碼。從該連結也可以取得其他語言的程式碼。

若要在主控台應用程式中測試此程式碼，您必須︰

- [在 app.config 檔案中設定儲存體連接字串](../storage/storage-dotnet-how-to-use-queues.md#setup-a-storage-connection-string)。
- 在執行階段期間，此程式碼會接受新使用者名稱和位址做為命令列引數。傳遞 `name` 和 `address` 做為應用程式的參數。例如，`C:\myQueueConsoleApp\test.exe "Wes testing queues" "in a console app"`


範例 C# 程式碼︰

    static void Main(string[] args)
    {
        string name = null;
        string address = null;
        string queueName = "queue-newusers";
        string JSON = null;

        if (args.Length > 0)
        {
            name = args[0];
        }
        if (args.Length > 1)
        {
            address = args[1];
        }

        // Retrieve storage account from connection string
        CloudStorageAccount storageAccount = CloudStorageAccount.Parse(ConfigurationManager.AppSettings["StorageConnectionString"]);

        // Create the queue client
        CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

        // Retrieve a reference to a queue
        CloudQueue queue = queueClient.GetQueueReference(queueName);

        // Create the queue if it doesn't already exist
        queue.CreateIfNotExists();

        // Create a message and add it to the queue.
        if (name != null)
        {
            if (address != null)
                JSON = String.Format("{{"name":"{0}","address":"{1}"}}", name, address);
            else
                JSON = String.Format("{{"name":"{0}"}}", name);
        }

        Console.WriteLine("Adding message to " + queueName + "...");
        Console.WriteLine(JSON);

        CloudQueueMessage message = new CloudQueueMessage(JSON);
        queue.AddMessage(message);
    }

在佇列函數的瀏覽器視窗中，您會看到正在處理的每個訊息︰

	2016-03-24T10:27:06  Welcome, you are now connected to log-streaming service.
	2016-03-24T10:27:30.607 Function started (Id=e304450c-ff48-44dc-ba2e-1df7209a9d22)
	2016-03-24T10:27:30.607 C# Queue trigger function processed: {"name":"Wes testing queues","address":"in a console app"}
	2016-03-24T10:27:30.607 Function completed (Success, Id=e304450c-ff48-44dc-ba2e-1df7209a9d22)


<!-- URLs. -->

[Azure 入口網站]: https://portal.azure.com

<!---HONumber=AcomDC_0921_2016-->