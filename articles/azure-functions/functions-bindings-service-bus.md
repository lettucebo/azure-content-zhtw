<properties
	pageTitle="Azure Functions 服務匯流排觸發程序和繫結 | Microsoft Azure"
	description="瞭解如何在 Azure Functions 中使用「Azure 服務匯流排」觸發程序和繫結。"
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

# 佇列和主題的 Azure Functions 服務匯流排觸發程序和繫結

[AZURE.INCLUDE [functions-selector-bindings](../../includes/functions-selector-bindings.md)]

這篇文章說明如何在 Azure Functions 中為「Azure 服務匯流排」觸發程序和繫結進行設定及撰寫程式碼。

[AZURE.INCLUDE [簡介](../../includes/functions-bindings-intro.md)]

## <a id="sbtrigger"></a>服務匯流排的佇列或主題觸發程序

#### function.json

「function.json」檔案指定了下列屬性。

- `name`︰函式程式碼中用於佇列或主題 (或是佇列或主題訊息) 的變數名稱。
- `queueName`︰(僅適用於佇列觸發程序) 要輪詢的佇列名稱。
- `topicName`︰(僅適用於主題觸發程序) 要輪詢的主題名稱。
- `subscriptionName`︰(僅適用於主題觸發程序) 訂用帳戶名稱。
- `connection`︰包含「服務匯流排」連接字串的應用程式設定名稱。連接字串必須是用於服務匯流排命名空間，而不限於特定佇列或主題。如果連接字串不具有管理權限，請設定 `accessRights` 屬性。如果將 `connection` 留白，觸發程序或繫結將會使用函式應用程式的預設「服務匯流排」連接字串，此預設值是由 AzureWebJobsServiceBus 應用程式設定所指定。
- `accessRights`︰指定連接字串可用的存取權限。預設值為 `manage`。如果您使用的連接字串不提供管理權限，請設定為 `listen`。否則，Functions 執行階段可能會嘗試，但卻無法執行需要管理權限的作業。
- `type`︰必須設為「serviceBusTrigger」。
- `direction`：必須設為「in」。

「服務匯流排」佇列觸發程序的範例「function.json」：

```json
{
  "bindings": [
    {
      "queueName": "testqueue",
      "connection": "MyServiceBusConnection",
      "name": "myQueueItem",
      "type": "serviceBusTrigger",
      "direction": "in"
    }
  ],
  "disabled": false
}
```

#### 處理服務匯流排佇列訊息的 C# 程式碼範例

```csharp
public static void Run(string myQueueItem, TraceWriter log)
{
    log.Info($"C# ServiceBus queue trigger function processed message: {myQueueItem}");
}
```

#### 處理服務匯流排佇列訊息的 F# 程式碼範例

```fsharp
let Run(myQueueItem: string, log: TraceWriter) =
    log.Info(sprintf "F# ServiceBus queue trigger function processed message: %s" myQueueItem)
```

#### 處理服務匯流排佇列訊息的 Node.js 程式碼範例

```javascript
module.exports = function(context, myQueueItem) {
    context.log('Node.js ServiceBus queue trigger function processed message', myQueueItem);
    context.done();
};
```

#### 支援的類型

服務匯流排佇列訊息可以還原序列化為下列任何一種類型︰

* 物件 (來自 JSON)
* string
* 位元組陣列
* `BrokeredMessage` (C#)

#### <a id="sbpeeklock"></a>PeekLock 行為

Functions 執行階段會在 `PeekLock` 模式下接收訊息，並在函式順利完成時，於訊息上呼叫 `Complete`，或是在函式失敗時呼叫 `Abandon`。如果函數執行時間較 `PeekLock` 逾時還長，即會自動更新鎖定。

#### <a id="sbpoison"></a>有害訊息處理

服務匯流排會自行處理無法在 Azure Functions 組態或程式碼中控制或設定的有害訊息。

#### <a id="sbsinglethread"></a>單一執行緒

Functions 執行階段預設會並行處理多個佇列訊息。若要指示執行階段一次只處理一個佇列或主題訊息，請在「host.json」檔案中將 `serviceBus.maxConcurrrentCalls` 設定為 1。如需「host.json」檔案的相關資訊，請參閱開發人員參考文章中的[資料夾結構](functions-reference.md#folder-structure)和 WebJobs.Script 儲存機制 wiki 中的 [host.json](https://github.com/Azure/azure-webjobs-sdk-script/wiki/host.json)。

## <a id="sboutput"></a>服務匯流排的佇列或主題輸出繫結

#### function.json

「function.json」檔案指定了下列屬性。

- `name`︰函式程式碼中用於佇列或佇列訊息的變數名稱。
- `queueName`︰(僅適用於佇列觸發程序) 要輪詢的佇列名稱。
- `topicName`︰(僅適用於主題觸發程序) 要輪詢的主題名稱。
- `subscriptionName`︰(僅適用於主題觸發程序) 訂用帳戶名稱。
- `connection`︰與用於「服務匯流排」觸發程序的相同。
- `accessRights`︰指定連接字串可用的存取權限。預設值為 `manage`。如果您使用的連接字串不提供管理權限，請設定為 `send`。否則，Functions 執行階段可能會嘗試，但卻無法執行需要管理權限的作業，例如建立佇列。
- `type`︰必須設為「serviceBus」。
- `direction`：必須設為「out」。

使用計時器觸發程序來撰寫「服務匯流排」佇列訊息的「function.json」範例：

```JSON
{
  "bindings": [
    {
      "schedule": "0/15 * * * * *",
      "name": "myTimer",
      "runsOnStartup": true,
      "type": "timerTrigger",
      "direction": "in"
    },
    {
      "name": "outputSbQueue",
      "type": "serviceBus",
      "queueName": "testqueue",
      "connection": "MyServiceBusConnection",
      "direction": "out"
    }
  ],
  "disabled": false
}
``` 

#### 支援的類型

Azure Functions 可以從下列任何類型建立服務匯流排佇列訊息。

* 物件 (一律會建立 JSON 訊息，當函式結束時，如果此值為 Null，則會使用 Null 物件建立訊息)
* 字串 (當函式結束時，如果此值非 Null，則會建立訊息)
* 位元組陣列 (像字串般運作)
* `BrokeredMessage`(C#，運作方式與字串相同)

若要在 C# 函式中建立多個訊息，您可以使用 `ICollector<T>` 或 `IAsyncCollector<T>`。當您呼叫 `Add` 方法時，就會建立一則訊息。

#### 建立服務匯流排佇列訊息的 C# 程式碼範例

```csharp
public static void Run(TimerInfo myTimer, TraceWriter log, out string outputSbQueue)
{
	string message = $"Service Bus queue message created at: {DateTime.Now}";
    log.Info(message); 
    outputSbQueue = message;
}
```

```csharp
public static void Run(TimerInfo myTimer, TraceWriter log, ICollector<string> outputSbQueue)
{
	string message = $"Service Bus queue message created at: {DateTime.Now}";
    log.Info(message); 
    outputSbQueue.Add("1 " + message);
    outputSbQueue.Add("2 " + message);
}
```

#### 建立服務匯流排佇列訊息的 F# 程式碼範例

```fsharp
let Run(myTimer: TimerInfo, log: TraceWriter, outputSbQueue: byref<string>) =
    let message = sprintf "Service Bus queue message created at: %s" (DateTime.Now.ToString())
    log.Info(message)
    outputSbQueue = message
```

#### 建立服務匯流排佇列訊息的 Node.js 程式碼範例

```javascript
module.exports = function (context, myTimer) {
    var timeStamp = new Date().toISOString();
    
    if(myTimer.isPastDue)
    {
        context.log('Node.js is running late!');
    }
    var message = 'Service Bus queue message created at ' + timeStamp;
    context.log(message);   
    context.bindings.outputSbQueueMsg = message;
    context.done();
};
```

## 後續步驟

[AZURE.INCLUDE [後續步驟](../../includes/functions-bindings-next-steps.md)]

<!---HONumber=AcomDC_0921_2016-->