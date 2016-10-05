## 將訊息傳送至事件中心

在本節中，您將撰寫一個 Windows 主控台應用程式，以將事件傳送至事件中心。

1. 在 Visual Studio 中，使用**主控台應用程式**專案範本建立新的「Visual C# 桌面應用程式」專案。將專案命名為 **Sender**。

	![](./media/service-bus-event-hubs-getstarted-send-csharp/create-sender-csharp1.png)

2. 在 [方案總管] 中，以滑鼠右鍵按一下方案，然後按一下 [管理方案的 NuGet 封裝]。

3. 按一下 [瀏覽] 索引標籤，然後搜尋 `Microsoft Azure Service Bus`。確定已在 [版本] 方塊中指定專案名稱 (傳送者)。按一下 [安裝] 並接受使用條款。

	![](./media/service-bus-event-hubs-getstarted-send-csharp/create-sender-csharp2.png)

	Visual Studio 會下載、安裝並新增 [Azure 服務匯流排程式庫 NuGet 封裝](https://www.nuget.org/packages/WindowsAzure.ServiceBus)的參考。

4. 在 **Program.cs** 檔案開頭處新增下列 `using` 陳述式：

	```
	using System.Threading;
	using Microsoft.ServiceBus.Messaging;
	```

5. 將下列欄位加入至 **Program** 類別，並以您在上一節中建立的事件中樞名稱替代預留位置值，以及您先前儲存的命名空間層級連接字串。

	```
	static string eventHubName = "{Event Hub name}";
	static string connectionString = "{send connection string}";
	```

6. 將下列方法加入至 **Program** 類別：

	```
	static void SendingRandomMessages()
	{
	    var eventHubClient = EventHubClient.CreateFromConnectionString(connectionString, eventHubName);
	    while (true)
	    {
	        try
	        {
	            var message = Guid.NewGuid().ToString();
	            Console.WriteLine("{0} > Sending message: {1}", DateTime.Now, message);
	            eventHubClient.Send(new EventData(Encoding.UTF8.GetBytes(message)));
	        }
	        catch (Exception exception)
	        {
	            Console.ForegroundColor = ConsoleColor.Red;
	            Console.WriteLine("{0} > Exception: {1}", DateTime.Now, exception.Message);
	            Console.ResetColor();
	        }

	        Thread.Sleep(200);
	    }
	}
	```

	這個方法會以 200 毫秒的延遲時間持續將事件傳送至事件中心。

7. 最後，將下列幾行加入至 **Main** 方法：

	```
	Console.WriteLine("Press Ctrl-C to stop the sender process");
	Console.WriteLine("Press Enter to start now");
	Console.ReadLine();
	SendingRandomMessages();
	```

<!---HONumber=AcomDC_0921_2016-->