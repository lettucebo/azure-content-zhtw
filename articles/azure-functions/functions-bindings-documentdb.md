<properties
	pageTitle="Azure Functions DocumentDB 繫結 | Microsoft Azure"
	description="了解如何在 Azure Functions 中使用 Azure DocumentDB 繫結。"
	services="functions"
	documentationCenter="na"
	authors="christopheranderson"
	manager="erikre"
	editor=""
	tags=""
	keywords="azure functions, 函數, 事件處理, 動態運算, 無伺服器架構"/>

<tags
	ms.service="functions"
	ms.devlang="multiple"
	ms.topic="reference"
	ms.tgt_pltfrm="multiple"
	ms.workload="na"
	ms.date="08/22/2016"
	ms.author="chrande; glenga"/>

# Azure Functions DocumentDB 繫結

[AZURE.INCLUDE [functions-selector-bindings](../../includes/functions-selector-bindings.md)]

這篇文章說明如何在 Azure Functions 中為 Azure DocumentDB 繫結進行設定及撰寫程式碼。

[AZURE.INCLUDE [簡介](../../includes/functions-bindings-intro.md)]

## <a id="docdbinput"></a>Azure DocumentDB 輸入繫結

輸入繫結可從 DocumentDB 集合載入文件，並將它直接傳遞至您的繫結。您可以根據叫用該函式的觸發程序來判斷文件識別碼。在 C# 函式中，當函式成功結束時，會將記錄所做的任何變更自動傳回集合。

#### DocumentDB 輸入繫結的 function.json

「function.json」檔案提供下列屬性：

- `name`︰函式程式碼中用於文件的變數名稱。
- `type`︰必須設定為 "documentdb"。
- `databaseName`︰包含文件的資料庫。
- `collectionName`︰包含文件的集合。
- `id`︰要擷取之文件的識別碼。此屬性支援類似於 "{queueTrigger}" 的繫結，會將佇列訊息的字串值做為文件識別碼。
- `connection`︰此字串必須是針對您的 DocumentDB 帳戶設定為端點的「應用程式設定」。如果您是從 [整合] 索引標籤選擇您的帳戶，系統將會為您建立新的應用程式設定，其名稱會採用下列格式：yourAccount\_DOCUMENTDB。如果您需要手動建立應用程式設定，則實際的連接字串必須採用下列格式：AccountEndpoint=<Endpoint for your account>;AccountKey=<Your primary access key>;。
- `direction︰必須設為「"in"」。

範例「function.json」：
 
	{
	  "bindings": [
	    {
	      "name": "document",
	      "type": "documentdb",
	      "databaseName": "MyDatabase",
	      "collectionName": "MyCollection",
	      "id" : "{queueTrigger}",
	      "connection": "MyAccount_DOCUMENTDB",     
	      "direction": "in"
	    }
	  ],
	  "disabled": false
	}

#### C# 佇列觸發程序的 Azure DocumentDB 輸入程式碼範例
 
使用上述的 function.json 範例，DocumentDB 輸入繫結會擷取文件 (包含符合佇列訊息字串的識別碼)，並將它傳遞至 'document' 參數。如果找不到該文件，則 'document' 參數是 null。當函式結束時，文件隨後會以新的文字值更新。
 
	public static void Run(string myQueueItem, dynamic document)
	{   
	    document.text = "This has changed.";
	}

#### F# 佇列觸發程序的 Azure DocumentDB 輸入程式碼範例

使用上述的 function.json 範例，DocumentDB 輸入繫結會擷取文件 (包含符合佇列訊息字串的識別碼)，並將它傳遞至 'document' 參數。如果找不到該文件，則 'document' 參數是 null。當函式結束時，文件隨後會以新的文字值更新。

	open FSharp.Interop.Dynamic
	let Run(myQueueItem: string, document: obj) =
	    document?text <- "This has changed."

您會需要 `project.json` 檔案，以使用 NuGet 指定 `FSharp.Interop.Dynamic` 和 `Dynamitey` 封裝做為封裝相依性，例如︰

	{
	  "frameworks": {
	    "net46": {
	      "dependencies": {
	        "Dynamitey": "1.0.2",
	        "FSharp.Interop.Dynamic": "3.0.0"
	      }
	    }
	  }
	}

這會使用 NuGet 來擷取相依性，並會在指令碼中加以參考。

#### Node.js 佇列觸發程序的 Azure DocumentDB 輸入程式碼範例
 
使用上述範例 function.json 時，DocumentDB 輸入繫結會擷取識別碼與佇列訊息字串相符的文件，然後將它傳遞給 `documentIn` 繫結屬性。在 Node.js 函式中，更新的文件不會傳回至集合。不過，您可以將輸入繫結直接傳遞給名為 `documentOut` 的 DocumentDB 輸出繫結以支援更新。這個程式碼範例會更新輸入文件的 text 屬性，並將它設定為輸出文件。
 
	module.exports = function (context, input) {   
	    context.bindings.documentOut = context.bindings.documentIn;
	    context.bindings.documentOut.text = "This was updated!";
	    context.done();
	};

## <a id="docdboutput"></a>Azure DocumentDB 輸出繫結

您的函式可以使用 **Azure DocumentDB 文件**輸出繫結，將 JSON 文件寫入 Azure DocumentDB 資料庫中。如需有關 Azure DocumentDB 的詳細資訊，請檢閱 [DocumentDB 簡介](../documentdb/documentdb-introduction.md)和[開始使用教學課程](../documentdb/documentdb-get-started.md)。

#### DocumentDB 輸出繫結的 function.json

function.json 檔案提供下列屬性：

- `name`︰函式程式碼中用於新文件的變數名稱。
- `type`︰必須設為「"documentdb"」。
- `databaseName`︰包含其中將建立新文件之集合的資料庫。
- `collectionName`︰其中將建立新文件的集合。
- `createIfNotExists`︰這是一個布林值，用以指出當集合不存在時是否要建立集合。預設值為 *false*。因為新集合會使用保留的輸送量建立，其具有價格含意。如需詳細資訊，請瀏覽[定價頁面](https://azure.microsoft.com/pricing/details/documentdb/)。
- `connection`︰此字串必須是針對您的 DocumentDB 帳戶設定為端點的**應用程式設定**。如果您是從 [整合] 索引標籤選擇您的帳戶，系統將會為您建立新的應用程式設定，其名稱會採用下列格式：`yourAccount_DOCUMENTDB`。如果您需要手動建立應用程式設定，實際的連接字串必須採用下列格式：`AccountEndpoint=<Endpoint for your account>;AccountKey=<Your primary access key>;`。
- `direction`：必須設為「"out"」。
 
function.json 範例：

	{
	  "bindings": [
	    {
	      "name": "document",
	      "type": "documentdb",
	      "databaseName": "MyDatabase",
	      "collectionName": "MyCollection",
	      "createIfNotExists": false,
	      "connection": "MyAccount_DOCUMENTDB",
	      "direction": "out"
	    }
	  ],
	  "disabled": false
	}


#### Node.js 佇列觸發程序的 Azure DocumentDB 輸出程式碼範例

	module.exports = function (context, input) {
	   
	    context.bindings.document = {
	        text : "I'm running in a Node function! Data: '" + input + "'"
	    }   
	 
	    context.done();
	};

輸出文件︰

	{
	  "text": "I'm running in a Node function! Data: 'example queue data'",
	  "id": "01a817fe-f582-4839-b30c-fb32574ff13f"
	}
 

#### F# 佇列觸發程序的 Azure DocumentDB 輸出程式碼範例

	open FSharp.Interop.Dynamic
	let Run(myQueueItem: string, document: obj) =
	    document?text <- (sprintf "I'm running in an F# function! %s" myQueueItem)

#### C# 佇列觸發程序的 Azure DocumentDB 輸出程式碼範例


	using System;

	public static void Run(string myQueueItem, out object document, TraceWriter log)
	{
	    log.Info($"C# Queue trigger function processed: {myQueueItem}");
	   
	    document = new {
	        text = $"I'm running in a C# function! {myQueueItem}"
	    };
	}


#### Azure DocumentDB 輸出程式碼範例設定檔名稱

如果您想要在函式中設定文件名稱，只要設定 `id` 值即可。例如，如果正在將員工的 JSON 內容排入佇列中，如下所示︰

	{
	  "name" : "John Henry",
      "employeeId" : "123456",
	  "address" : "A town nearby"
	}

您可以在佇列觸發程序函式中使用下列 C# 程式碼︰
	
	#r "Newtonsoft.Json"
	
	using System;
	using Newtonsoft.Json;
	using Newtonsoft.Json.Linq;
	
	public static void Run(string myQueueItem, out object employeeDocument, TraceWriter log)
	{
	    log.Info($"C# Queue trigger function processed: {myQueueItem}");
	    
	    dynamic employee = JObject.Parse(myQueueItem);
	    
	    employeeDocument = new {
	        id = employee.name + "-" + employee.employeeId,
	        name = employee.name,
	        employeeId = employee.employeeId,
	        address = employee.address
	    };
	}

或對等的 F# 程式碼︰

	open FSharp.Interop.Dynamic
	open Newtonsoft.Json

	type Employee = {
	    id: string
	    name: string
	    employeeId: string
	    address: string
	}

	let Run(myQueueItem: string, employeeDocument: byref<obj>, log: TraceWriter) =
	    log.Info(sprintf "F# Queue trigger function processed: %s" myQueueItem)
	    let employee = JObject.Parse(myQueueItem)
	    employeeDocument <-
	        { id = sprintf "%s-%s" employee?name employee?employeeId
	          name = employee?name
	          employeeId = employee?id
	          address = employee?address }

範例輸出︰

	{
	  "id": "John Henry-123456",
	  "name": "John Henry",
	  "employeeId": "123456",
	  "address": "A town nearby"
	}

## 後續步驟

[AZURE.INCLUDE [後續步驟](../../includes/functions-bindings-next-steps.md)]

<!---HONumber=AcomDC_0921_2016-->