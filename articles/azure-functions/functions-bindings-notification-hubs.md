<properties
	pageTitle="Azure Functions 通知中樞繫結 | Microsoft Azure"
	description="了解如何在 Azure Functions 中使用「Azure 通知中樞」繫結。"
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
	ms.date="08/19/2016"
	ms.author="wesmc"/>

# Azure Functions 通知中樞輸出繫結

[AZURE.INCLUDE [functions-selector-bindings](../../includes/functions-selector-bindings.md)]

這篇文章說明如何在 Azure Functions 中為「Azure 通知中樞」繫結進行設定及撰寫程式碼。

[AZURE.INCLUDE [簡介](../../includes/functions-bindings-intro.md)]

您的函式可使用設定的 Azure 通知中樞與極少的程式碼行以傳送推播通知。不過，也必須為您要使用的平台通知服務 (PNS) 設定通知中樞。如需設定 Azure 通知中樞以及開發註冊來接收通知之用戶端應用程式的詳細資訊，請參閱[開始使用通知中樞](../notification-hubs/notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md)並按一下頂端的目標用戶端平台。

## Azure 通知中樞輸出繫結的 function.json

function.json 檔案提供下列屬性：

- `name`︰函式程式碼中用於通知中樞訊息的變數名稱。
- `type`︰必須設為「"notificationHub"」。
- `tagExpression`︰標籤運算式可讓您指定將通知傳遞到一組裝置，這些裝置已註冊要接收與標籤運算式相符的通知。如需詳細資訊，請參閱[路由與標籤運算式](../notification-hubs/notification-hubs-tags-segment-push-message.md)。
- `hubName`︰Azure 入口網站中通知中樞資源的名稱。
- `connection`︰此連接字串必須是針對通知中樞設為「DefaultFullSharedAccessSignature」值的**應用程式設定**連接字串。
- `direction`：必須設為「"out"」。
 
function.json 範例：

	{
	  "bindings": [
	    {
	      "name": "notification",
	      "type": "notificationHub",
	      "tagExpression": "",
	      "hubName": "my-notification-hub",
	      "connection": "MyHubConnectionString",
	      "direction": "out"
	    }
	  ],
	  "disabled": false
	}

## Azure 通知中樞連接字串設定

若要使用通知中樞輸出繫結，必須設定中樞的連接字串。若要這麼做，只要在 [整合] 索引標籤上選取您的通知中樞或建立一個新通知中樞即可。

您也可以將「DefaultFullSharedAccessSignature」新增至通知中樞，手動新增現有中樞的連接字串。此連接字串提供您的函式存取權限來傳送通知訊息。您可從 Azure 入口網站的通知中樞資源的主要刀鋒視窗，從 [金鑰] 按鈕存取「DefaultFullSharedAccessSignature」連接字串值。若要手動新增中樞的連接字串，請使用下列步驟︰

1. 在 Azure 入口網站的 [函式應用程式] 刀鋒視窗中，按一下 [函式應用程式設定] > [移至 App Service 設定]。

2. 在 [設定] 刀鋒視窗中，按一下 [應用程式設定]。

3. 向下捲動至 [連接字串] 區段，再為通知中樞新增「DefaultFullSharedAccessSignature」值的命名項目。將類型變更為 [自訂]。
4. 參考輸出繫結中的連接字串名稱。與上述範例中使用的 **MyHubConnectionString** 類似。

## Node.js 計時器觸發程序的 Azure 通知中樞程式碼範例 

這個範例會傳送包含 `location` 和 `message` 的[範本註冊](../notification-hubs/notification-hubs-templates-cross-platform-push-messages.md)通知。

	module.exports = function (context, myTimer) {
	    var timeStamp = new Date().toISOString();
	   
	    if(myTimer.isPastDue)
	    {
	        context.log('Node.js is running late!');
	    }
	    context.log('Node.js timer trigger function ran!', timeStamp);  
	    context.bindings.notification = {
	        location: "Redmond",
	        message: "Hello from Node!"
	    };
	    context.done();
	};

## F# 計時器觸發程序的 Azure 通知中樞程式碼範例

這個範例會傳送包含 `location` 和 `message` 的[範本註冊](../notification-hubs/notification-hubs-templates-cross-platform-push-messages.md)通知。

	let Run(myTimer: TimerInfo, notification: byref<IDictionary<string, string>>) =
	    notification = dict [("location", "Redmond"); ("message", "Hello from F#!")]

## C# 佇列觸發程序的 Azure 通知中樞程式碼範例

這個範例會傳送包含 `message` 的[範本註冊](../notification-hubs/notification-hubs-templates-cross-platform-push-messages.md)通知。


	using System;
	using System.Threading.Tasks;
	using System.Collections.Generic;
	 
	public static void Run(string myQueueItem,  out IDictionary<string, string> notification, TraceWriter log)
	{
	    log.Info($"C# Queue trigger function processed: {myQueueItem}");
        notification = GetTemplateProperties(myQueueItem);
	}
	 
	private static IDictionary<string, string> GetTemplateProperties(string message)
	{
	    Dictionary<string, string> templateProperties = new Dictionary<string, string>();
	    templateProperties["message"] = message;
	    return templateProperties;
	}

這個範例會使用有效的 JSON 字串，傳送包含 `message` 的[範本註冊](../notification-hubs/notification-hubs-templates-cross-platform-push-messages.md)通知。

	using System;
	 
	public static void Run(string myQueueItem,  out string notification, TraceWriter log)
	{
		log.Info($"C# Queue trigger function processed: {myQueueItem}");
		notification = "{"message":"Hello from C#. Processed a queue item!"}";
	}

## 使用通知類型的 Azure 通知中樞佇列觸發程序 C# 程式碼範例

這個範例示範如何使用 [Microsoft Azure 通知中樞程式庫](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/)中所定義的 `Notification` 類型。若要使用此類型和程式庫，您必須上傳函式應用程式的「project.json」檔案。project.json 檔案是看起來類似下列的 JSON 文字檔︰

	{
	  "frameworks": {
	    ".NETFramework,Version=v4.6": {
	      "dependencies": {
	        "Microsoft.Azure.NotificationHubs": "1.0.4"
	      }
	    }
	  }
	}

如需有關上傳 project.json 檔案的詳細資訊，請參閱[上傳 project.json 檔案](functions-reference.md#fileupdate)。

程式碼範例：

	using System;
	using System.Threading.Tasks;
	using Microsoft.Azure.NotificationHubs;
	 
	public static void Run(string myQueueItem,  out Notification notification, TraceWriter log)
	{
	   log.Info($"C# Queue trigger function processed: {myQueueItem}");
	   notification = GetTemplateNotification(myQueueItem);
	}
	private static TemplateNotification GetTemplateNotification(string message)
	{
	    Dictionary<string, string> templateProperties = new Dictionary<string, string>();
	    templateProperties["message"] = message;
	    return new TemplateNotification(templateProperties);
	}

## 後續步驟

[AZURE.INCLUDE [後續步驟](../../includes/functions-bindings-next-steps.md)]

<!---HONumber=AcomDC_0921_2016-->