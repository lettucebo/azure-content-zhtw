<properties
	pageTitle="Azure Functions 事件中樞繫結 | Microsoft Azure"
	description="了解如何在 Azure Functions 中使用 Azure 事件中樞繫結。"
	services="functions"
	documentationCenter="na"
	authors="wesmc7777"
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
	ms.author="wesmc"/>

# Azure Functions 事件中樞繫結

[AZURE.INCLUDE [functions-selector-bindings](../../includes/functions-selector-bindings.md)]

本文說明如何針對 Azure Functions 設定 [Azure 事件中樞](../event-hubs/event-hubs-overview.md)繫結以及撰寫程式碼。Azure Functions 支援適用於 Azure 事件中樞的觸發程序和輸出繫結。

[AZURE.INCLUDE [簡介](../../includes/functions-bindings-intro.md)]


## Azure 事件中樞觸發程序繫結

Azure 事件中樞觸發程序可用來回應傳送至事件中樞事件資料流的事件。您必須具有事件中樞的讀取權限，才能設定觸發程序繫結。

#### 適用於事件中樞觸發程序繫結的 function.json

適用於 Azure 事件中樞觸發程序的「function.json」檔案會指定下列屬性：

- `type`︰必須設定為「eventHubTrigger」。
- `name`︰用於事件中樞訊息之函式程式碼中的變數名稱。
- `direction`：必須設為 in。
- `path`：事件中樞的名稱。
- `connection`：應用程式設定的名稱，包含事件中樞所在之命名空間的連接字串。按一下命名空間的 [連接資訊] 按鈕 (而不是事件中樞本身)，來複製此連接字串。此連接字串至少必須具備讀取權限，才能啟動觸發程序。

		{
		  "bindings": [
		    {
		      "type": "eventHubTrigger",
		      "name": "myEventHubMessage",
		      "direction": "in",
		      "path": "MyEventHub",
		      "connection": "myEventHubReadConnectionString"
		    }
		  ],
		  "disabled": false
		}

#### Azure 事件中樞觸發程序 C# 範例
 
使用上述範例 function.json，利用下列 C# 函數程式碼來記錄事件訊息的主體：
 
	using System;
	
	public static void Run(string myEventHubMessage, TraceWriter log)
	{
	    log.Info($"C# Event Hub trigger function processed a message: {myEventHubMessage}");
	}

#### Azure 事件中樞觸發程序 F# 範例

使用上述範例 function.json，利用下列 F# 函式程式碼來記錄事件訊息的主體：

	let Run(myEventHubMessage: string, log: TraceWriter) =
	    log.Info(sprintf "F# eventhub trigger function processed work item: %s" myEventHubMessage)

#### Azure 事件中樞觸發程序 Node.js 範例
 
使用上述範例 function.json，利用下列 Node.js 函數程式碼來記錄事件訊息的主體：
 
	module.exports = function (context, myEventHubMessage) {
	    context.log('Node.js eventhub trigger function processed work item', myEventHubMessage);	
	    context.done();
	};


## Azure 事件中樞輸出繫結

Azure 事件中樞輸出繫結可用來將事件寫入事件中樞事件資料流。您必須具備事件中樞的傳送權限，才能將事件寫入其中。

#### 適用於事件中樞輸出繫結的 function.json

適用於 Azure 事件中樞輸出繫結的「function.json」檔案會指定下列屬性：

- `type`：必須設定為「eventHub」。
- `name`︰用於事件中樞訊息之函式程式碼中的變數名稱。
- `path`：事件中樞的名稱。
- `connection`：應用程式設定的名稱，包含事件中樞所在之命名空間的連接字串。按一下命名空間的 [連接資訊] 按鈕 (而不是事件中樞本身)，來複製此連接字串。此連接字串必須具有傳送權限，才能將訊息傳送至事件中樞資料流。
- `direction`：必須設為「out」。

	    {
	      "type": "eventHub",
	      "name": "outputEventHubMessage",
	      "path": "myeventhub",
	      "connection": "MyEventHubSend",
	      "direction": "out"
	    }


#### 適用於輸出繫結的 Azure 事件中樞 C# 程式碼範例
 
下列 C# 範例函式程式碼示範如何將事件寫入事件中樞事件資料流。這個範例代表上述套用到 C# 計時器觸發程序的事件中樞輸出繫結。
 
	using System;
	
	public static void Run(TimerInfo myTimer, out string outputEventHubMessage, TraceWriter log)
	{
	    String msg = $"TimerTriggerCSharp1 executed at: {DateTime.Now}";
	
	    log.Verbose(msg);   
	    
	    outputEventHubMessage = msg;
	}

#### 適用於輸出繫結的 Azure 事件中樞 F# 程式碼範例

下列 F# 範例函式程式碼示範如何將事件寫入事件中樞事件資料流。這個範例代表上述套用到 C# 計時器觸發程序的事件中樞輸出繫結。

	let Run(myTimer: TimerInfo, outputEventHubMessage: byref<string>, log: TraceWriter) =
	    let msg = sprintf "TimerTriggerFSharp1 executed at: %s" DateTime.Now.ToString()
	    log.Verbose(msg);
	    outputEventHubMessage <- msg;

#### 適用於輸出繫結的 Azure 事件中樞 Node.js 程式碼範例
 
下列 Node.js 範例函數程式碼示範如何將事件寫入事件中樞事件資料流。這個範例代表上述套用到 Node.js 計時器觸發程序的事件中樞輸出繫結。
 
	module.exports = function (context, myTimer) {
	    var timeStamp = new Date().toISOString();
	    
	    if(myTimer.isPastDue)
	    {
	        context.log('TimerTriggerNodeJS1 is running late!');
	    }

	    context.log('TimerTriggerNodeJS1 function ran!', timeStamp);   
	    
	    context.bindings.outputEventHubMessage = "TimerTriggerNodeJS1 ran at : " + timeStamp;
	
	    context.done();
	};

## 後續步驟

[AZURE.INCLUDE [後續步驟](../../includes/functions-bindings-next-steps.md)]

<!---HONumber=AcomDC_0921_2016-->