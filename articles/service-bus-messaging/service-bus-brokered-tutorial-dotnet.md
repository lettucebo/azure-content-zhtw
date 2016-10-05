<properties 
    pageTitle="服務匯流排代理傳訊 .NET 教學課程 | Microsoft Azure"
    description="代理傳訊 .NET 教學課程。"
    services="service-bus-messaging"
    documentationCenter="na"
    authors="sethmanheim"
    manager="timlt"
    editor="" />
<tags 
    ms.service="service-bus-messaging"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="09/27/2016"
    ms.author="sethm" />

# 服務匯流排代理傳訊 .NET 教學課程

Azure 服務匯流排提供兩種全方位訊息解決方案 – 其中一個透過在雲端執行的集中式「轉送」服務，支援各種不同的傳輸通訊協定和 Web 服務標準，包括 SOAP、WS-* 和 REST。用戶端不需要直接連線到內部部署服務，也不需要知道服務所在的位置，而且內部部署服務不需要在防火牆上開啟任何輸入連接埠。

第二個訊息解決方案可提供「代理」傳訊功能。這些代理傳訊功能可以視為非同步的低耦合訊息功能，可使用服務匯流排傳訊基礎結構來支援發佈-訂閱、時脈解離及負載平衡案例。低耦合通訊有許多優點︰例如，用戶端和伺服器可視需要連接並以非同步方式執行其作業。

本教學課程主要為了提供佇列 (服務匯流排代理傳訊的其中一個核心元件) 的概觀及實作經驗。在您逐步完成本教學課程中的各個主題之後，您必須有應用程式可填入訊息清單、建立佇列，並將訊息傳送至該佇列。最後，應用程式會接收並顯示佇列中的訊息，然後清除其資源並結束。如需說明如何建置使用服務匯流排轉送之應用程式的對應教學課程，請參閱[服務匯流排轉送傳訊教學課程](../service-bus-relay/service-bus-relay-tutorial.md)。

## 簡介和必要條件

如果有一或多個競爭取用者，佇列會採取先進先出 (FIFO) 訊息傳遞機制。FIFO 表示通常預期由接收者依訊息加入佇列的時間順序來接收和處理訊息，而且每則訊息只能由一個訊息取用者接收和處理。使用佇列的主要優點是達成應用程式元件的「時脈解離」︰換句話說，產生者和取用者不需同時傳送和接收訊息，因為訊息會長期儲存在佇列中。相關的優點是「負載調節」，這可讓產生者和取用者以不同的速率傳送和接收訊息。

以下是您在開始本教學課程之前所應遵循的一些管理和先決步驟。第一步是建立服務命名空間，並取得共用存取簽章 (SAS) 金鑰。命名空間會為每個透過服務匯流排公開的應用程式提供應用程式界限。建立服務命名空間時，系統會自動產生 SAS 金鑰。服務命名空間與 SAS 金鑰的結合提供了一個認證，以供服務匯流排驗證對應用程式的存取權。

### 建立服務命名空間並取得 SAS 金鑰

第一步是建立服務命名空間，並取得[共用存取簽章](../service-bus/service-bus-sas-overview.md) (SAS) 金鑰。命名空間會為每個透過服務匯流排公開的應用程式提供應用程式界限。建立服務命名空間時，系統會自動產生 SAS 金鑰。服務命名空間與 SAS 金鑰的結合提供了一個認證，供服務匯流排驗證對應用程式的存取權。

[AZURE.INCLUDE [service-bus-create-namespace-portal](../../includes/service-bus-create-namespace-portal.md)]

下一步是建立 Visual Studio 專案並撰寫兩個協助程式函式，將以逗號分隔的訊息清單載入強類型的 [BrokeredMessage](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.aspx) .NET [List](https://msdn.microsoft.com/library/6sh2ey19.aspx) 物件。

### 建立 Visual Studio 專案

1. 以系統管理員身分開啟 Visual Studio：以滑鼠右鍵按一下 [開始] 功能表中的程式，然後按一下 [以系統管理員身分執行]。

1. 這會建立新的主控台應用程式專案。按一下 [檔案] 功能表，選取 [新增]，然後按一下 [專案]。在 [新增專案] 對話方塊中，按一下 [Visual C#] (如果 [Visual C#] 未出現，請查看 [其他語言] 下方)，按一下 [主控台應用程式] 範本，並將它命名為 **QueueSample**。使用預設 [位置]。按一下 [確定] 以建立專案。

1. 使用 NuGet 套件管理員將服務匯流排程式庫新增至您的專案︰
	1. 在 [方案總管] 中，以滑鼠右鍵按一下 **QueueSample** 專案，然後按一下 [管理 NuGet 套件]。
	2. 在 [管理 Nuget 套件] 對話方塊中，按一下 [瀏覽] 索引標籤，搜尋 [服務匯流排]，然後按一下 [安裝]。<br />
1. 在 [方案總管] 中，按兩下 Program.cs 檔案，以在 Visual Studio 編輯器中開啟它。將命名空間名稱從 `QueueSample` 的預設名稱變更為 `Microsoft.ServiceBus.Samples`。

	```
	Microsoft.ServiceBus.Samples
	{
	    ...
	```

1. 如下列程式碼所示，修改 `using` 陳述式。

	```
	using System;
	using System.Collections.Generic;
	using System.Data;
	using System.IO;
	using System.Threading;
	using System.Threading.Tasks;
	using Microsoft.ServiceBus.Messaging;
	```

1. 建立名為 Data.csv 的文字檔，並複製下列以逗號分隔的文字。

	```
	IssueID,IssueTitle,CustomerID,CategoryID,SupportPackage,Priority,Severity,Resolved
	1,Package lost,1,1,Basic,5,1,FALSE
	2,Package damaged,1,1,Basic,5,1,FALSE
	3,Product defective,1,2,Premium,5,2,FALSE
	4,Product damaged,2,2,Premium,5,2,FALSE
	5,Package lost,2,2,Basic,5,2,TRUE
	6,Package lost,3,2,Basic,5,2,FALSE
	7,Package damaged,3,7,Premium,5,3,FALSE
	8,Product defective,3,2,Premium,5,3,FALSE
	9,Product damaged,4,6,Premium,5,3,TRUE
	10,Package lost,4,8,Basic,5,3,FALSE
	11,Package damaged,5,4,Basic,5,4,FALSE
	12,Product defective,5,4,Basic,5,4,FALSE
	13,Package lost,6,8,Basic,5,4,FALSE
	14,Package damaged,6,7,Premium,5,5,FALSE
	15,Product defective,6,2,Premium,5,5,FALSE
	```

	儲存並關閉 Data.csv 檔案，然後記得您儲存它的位置。

1. 在 [方案總管] 中，以滑鼠右鍵按一下您的專案名稱 (在此例中為 **QueueSample**)、按一下 [新增]，然後按一下 [現有項目]。

1. 瀏覽至您在步驟 6 中建立的 Data.csv 檔案。按一下該檔案，然後按一下 [新增] 按鈕。確定已在檔案類型清單中選取 [所有檔案 (*.*)]。

### 建立可剖析訊息清單的方法

1. 在 `Main()` 方法前面的 `Program` 類別中，宣告兩個變數：其中一個變數的類型為 **DataTable**，以在 Data.csv 中包含訊息清單。另一個變數的類型應為 List 物件，其類型強制為 [BrokeredMessage](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.aspx)。後者是教學課程中的後續步驟將使用的代理訊息清單。

	```
	namespace Microsoft.ServiceBus.Samples
	{
	    class Program
	    {
	
	        private static DataTable issues;
	        private static List<BrokeredMessage> MessageList;
	```

1. 在 `Main()` 之外，定義 `ParseCSV()` 方法，以剖析 Data.csv 中的訊息清單，並將訊息載入 [DataTable](https://msdn.microsoft.com/library/azure/system.data.datatable.aspx) 資料表，如下所示。此方法會傳回 **DataTable** 物件。

	```
	static DataTable ParseCSVFile()
	{
	    DataTable tableIssues = new DataTable("Issues");
	    string path = @"..\..\data.csv";
	    try
	    {
	        using (StreamReader readFile = new StreamReader(path))
	        {
	            string line;
	            string[] row;
	
	            // create the columns
	            line = readFile.ReadLine();
	            foreach (string columnTitle in line.Split(','))
	            {
	                tableIssues.Columns.Add(columnTitle);
	            }
	
	            while ((line = readFile.ReadLine()) != null)
	            {
	                row = line.Split(',');
	                tableIssues.Rows.Add(row);
	            }
	        }
	    }
	    catch (Exception e)
	    {
	        Console.WriteLine("Error:" + e.ToString());
	    }
	
	    return tableIssues;
	}
	```

1. 在 `Main()` 方法中，新增可呼叫 `ParseCSVFile()` 方法的陳述式︰

	```
	public static void Main(string[] args)
	{
	
	    // Populate test data
	    issues = ParseCSVFile();
	
	}
	```

### 建立可載入訊息清單的方法

1. 在 `Main()` 之外，定義 `GenerateMessages()` 方法，以採用 `ParseCSVFile()` 所傳回的 **DataTable** 物件，並將此資料表載入代理訊息的強類型清單中。此方法會接著傳回 **List** 物件，如下列範例所示。

	```
	static List<BrokeredMessage> GenerateMessages(DataTable issues)
	{
	    // Instantiate the brokered list object
	    List<BrokeredMessage> result = new List<BrokeredMessage>();
	
	    // Iterate through the table and create a brokered message for each row
	    foreach (DataRow item in issues.Rows)
	    {
	        BrokeredMessage message = new BrokeredMessage();
	        foreach (DataColumn property in issues.Columns)
	        {
	            message.Properties.Add(property.ColumnName, item[property]);
	        }
	        result.Add(message);
	    }
	    return result;
	}
	```

1. 在 `Main()` 中，緊接在 `ParseCSVFile()` 呼叫後面，新增可呼叫 `GenerateMessages()` 方法的陳述式 (以 `ParseCSVFile()` 的傳回值作為引數)︰

	```
	public static void Main(string[] args)
	{
	
	    // Populate test data
	    issues = ParseCSVFile();
	    MessageList = GenerateMessages(issues);
	}
	```

### 取得使用者認證

1. 首先，建立三個全域字串變數來保留這些值。緊接在先前的變數宣告之後宣告這些變數；例如：

	```
	namespace Microsoft.ServiceBus.Samples
	{
	    public class Program
	    {
	
	        private static DataTable issues;
	        private static List<BrokeredMessage> MessageList; 

	        // Add these variables
			private static string ServiceNamespace;
	        private static string sasKeyName = "RootManageSharedAccessKey";
	        private static string sasKeyValue;
	        …
	```

1. 接下來，建立可接受和儲存服務命名空間和 SAS 金鑰的函式。在 `Main()` 之外新增此方法。例如：

	```
	static void CollectUserInput()
	{
	    // User service namespace
	    Console.Write("Please enter the namespace to use: ");
	    ServiceNamespace = Console.ReadLine();
	
	    // Issuer key
	    Console.Write("Enter the SAS key to use: ");
	    sasKeyValue = Console.ReadLine();
	}
	```

1. 在 `Main()` 中，緊接在 `GenerateMessages()` 呼叫後面，新增可呼叫 `CollectUserInput()` 方法的陳述式︰

	```
	public static void Main(string[] args)
	{
	
	    // Populate test data
	    issues = ParseCSVFile();
	    MessageList = GenerateMessages(issues);
	    
	    // Collect user input
	    CollectUserInput();
	}
	```

### 建立解決方案

從 Visual Studio 的 [建置] 功能表中，按一下 [建置方案] 或按 **Ctrl+Shift+B**，以確認您的工作到目前為止是否正確無誤。

## 建立管理認證

在此步驟中，您可定義將用於建立共用存取簽章 (SAS) 認證的管理作業，以便授權您的應用程式。

1. 為了清楚起見，本教學課程會將所有佇列作業放在不同的方法中。在 `Program` 類別中的 `Main()` 方法後面建立非同步 `Queue()` 方法。例如：
 
	```
	public static void Main(string[] args)
	{
	…
	}
	static async Task Queue()
	{
	}
	```

1. 下一步是使用 [TokenProvider](https://msdn.microsoft.com/library/azure/microsoft.servicebus.tokenprovider.aspx) 物件建立 SAS 認證。建立方法會採用在 `CollectUserInput()` 方法中取得的 SAS 金鑰名稱和值。將下列程式碼新增至 `Queue()` 方法：

	```
	static async Task Queue()
	{
	    // Create management credentials
	    TokenProvider credentials = TokenProvider.CreateSharedAccessSignatureTokenProvider(sasKeyName,sasKeyValue);
	}
	```

2. 建立新的命名空間管理物件，以包含在上一個步驟中取得之命名空間名稱和管理認證的 URI 作為引數。將此程式碼直接加在上一個步驟中新增的程式碼之後︰務必以您的服務命名空間名稱取代 `<yourNamespace>`：
	
	```
	NamespaceManager namespaceClient = new NamespaceManager(ServiceBusEnvironment.CreateServiceUri("sb", "<yourNamespace>", string.Empty), credentials);
	```

### 範例

此時，您的程式碼應會如下所示：

```
using System;
using System.Collections.Generic;
using System.Data;
using System.IO;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.ServiceBus.Messaging;

namespace Microsoft.ServiceBus.Samples
{
  public class Program
  {
    private static DataTable issues;
    private static List<BrokeredMessage> MessageList;
    private static string ServiceNamespace;
    private static string sasKeyName = "RootManageSharedAccessKey";
    private static string sasKeyValue;

    static void Main(string[] args)
    {
      // Populate test data
      issues = ParseCSVFile();
      MessageList = GenerateMessages(issues);

      // Collect user input
      CollectUserInput();

      // Add this call
      Task.WaitAll(Queue());
    }

    static async Task Queue()
    {
      // Create management credentials
      TokenProvider credentials = TokenProvider.CreateSharedAccessSignatureTokenProvider(sasKeyName, sasKeyValue);
      NamespaceManager namespaceClient = new NamespaceManager(ServiceBusEnvironment.CreateServiceUri("sb", "<yourNamespace>", string.Empty), credentials);
    }

    static DataTable ParseCSVFile()
    {
      DataTable tableIssues = new DataTable("Issues");
      string path = @"..\..\data.csv";
      try
      {
        using (StreamReader readFile = new StreamReader(path))
        {
          string line;
          string[] row;

          // create the columns
          line = readFile.ReadLine();
          foreach (string columnTitle in line.Split(','))
          {
            tableIssues.Columns.Add(columnTitle);
          }

          while ((line = readFile.ReadLine()) != null)
          {
            row = line.Split(',');
            tableIssues.Rows.Add(row);
          }
        }
      }
      catch (Exception e)
      {
        Console.WriteLine("Error:" + e.ToString());
      }

      return tableIssues;
    }

    static List<BrokeredMessage> GenerateMessages(DataTable issues)
    {
      // Instantiate the brokered list object
      List<BrokeredMessage> result = new List<BrokeredMessage>();

      // Iterate through the table and create a brokered message for each row
      foreach (DataRow item in issues.Rows)
      {
        BrokeredMessage message = new BrokeredMessage();
        foreach (DataColumn property in issues.Columns)
        {
          message.Properties.Add(property.ColumnName, item[property]);
        }
        result.Add(message);
      }
      return result;
    }

    static void CollectUserInput()
    {
      // User service namespace
      Console.Write("Please enter the service namespace to use: ");
      ServiceNamespace = Console.ReadLine();

      // Issuer key
      Console.Write("Please enter the issuer key to use: ");
      sasKeyValue = Console.ReadLine();
    }
  }
}
```

## 將訊息傳送到佇列

在此步驟中，您可以建立佇列，然後將代理訊息清單中包含的訊息傳送至該佇列。

### 建立佇列並將訊息傳送至佇列

1. 首先，建立佇列。例如，將它稱為 `myQueue`，並緊接在您在最後一個步驟的 `Queue()` 方法中新增的管理作業之後進行宣告：

	```
    QueueDescription myQueue;

    if (namespaceClient.QueueExists("IssueTrackingQueue"))
    {
        namespaceClient.DeleteQueue("IssueTrackingQueue");
    }

    myQueue = namespaceClient.CreateQueue("IssueTrackingQueue");
	```

1. 在 `Queue()` 方法中，建立傳訊 factory 物件，並以新建立的服務匯流排 URI 作為引數。將下列程式碼直接加在最後一個步驟中新增的管理作業之後。務必以您的服務命名空間名稱取代 `<yourNamespace>`：

	```
	MessagingFactory factory = MessagingFactory.Create(ServiceBusEnvironment.CreateServiceUri("sb", "<yourNamespace>", string.Empty), credentials);
	```

1. 接著，使用 [QueueClient](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.queueclient.aspx) 類別建立佇列物件。將下列程式碼直接加在您在最後一個步驟中新增的程式碼之後︰

	```
	QueueClient myQueueClient = factory.CreateQueueClient("IssueTrackingQueue");
	```

1. 接著，新增會在您先前建立的代理訊息清單中執行迴圈的程式碼，並將每則訊息傳送至佇列。將下列程式碼直接加在上一個步驟中的 `CreateQueueClient()` 陳述式之後︰
	
	```
	// Send messages
	Console.WriteLine("Now sending messages to the queue.");
	for (int count = 0; count < 6; count++)
	{
	    var issue = MessageList[count];
	    issue.Label = issue.Properties["IssueTitle"].ToString();
	    await myQueueClient.SendAsync(issue);
	    Console.WriteLine(string.Format("Message sent: {0}, {1}", issue.Label, issue.MessageId));
	}
	```

## 從佇列接收訊息

在此步驟中，您會從在上一個步驟中建立的佇列中取得訊息清單。

### 建立接收者並從佇列接收訊息

在 `Queue()` 方法中，逐一查看佇列並使用 [QueueClient.ReceiveAsync](https://msdn.microsoft.com/library/azure/dn130423.aspx) 方法接收訊息，並將每則訊息列印至主控台。將下列程式碼直接加在您在上一個步驟中新增的程式碼之後︰

```
Console.WriteLine("Now receiving messages from Queue.");
BrokeredMessage message;
while ((message = await myQueueClient.ReceiveAsync(new TimeSpan(hours: 0, minutes: 1, seconds: 5))) != null)
    {
        Console.WriteLine(string.Format("Message received: {0}, {1}, {2}", message.SequenceNumber, message.Label, message.MessageId));
        message.Complete();
	
        Console.WriteLine("Processing message (sleeping...)");
        Thread.Sleep(1000);
    }
```

請注意，`Thread.Sleep` 只用來模擬訊息處理，而您在真實的訊息應用程式中可能不需用到。

### 結束 Queue 方法並清除資源

緊接在先前的程式碼之後，新增下列程式碼來清除訊息工廠和佇列資源：

```
factory.Close();
myQueueClient.Close();
namespaceClient.DeleteQueue("IssueTrackingQueue");
```

### 呼叫 Queue 方法

最後一個步驟是新增可從 `Main()` 呼叫 `Queue()` 方法的陳述式。在 Main() 的結尾新增下列醒目提示的程式碼行：
	
```
public static void Main(string[] args)
{
    // Collect user input
    CollectUserInput();
	
    // Populate test data
    issues = ParseCSVFile();
    MessageList = GenerateMessages(issues);
	
    // Add this call
    Queue();
}
```

### 範例

下列程式碼包含完整的 **QueueSample** 應用程式。

```
using System;
using System.Collections.Generic;
using System.Data;
using System.IO;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.ServiceBus.Messaging;

namespace Microsoft.ServiceBus.Samples
{
    public class Program
    {
        private static DataTable issues;
        private static List<BrokeredMessage> MessageList;

        // Add these variables
        private static string ServiceNamespace;
        private static string sasKeyName = "RootManageSharedAccessKey";
        private static string sasKeyValue;

        static void Main(string[] args)
        {

            // Populate test data
            issues = ParseCSVFile();
            MessageList = GenerateMessages(issues);

            // Collect user input
            CollectUserInput();

            Queue();

        }

        static async Task Queue()
        {
            // Create management credentials
            TokenProvider credentials = TokenProvider.CreateSharedAccessSignatureTokenProvider(sasKeyName, sasKeyValue);
            NamespaceManager namespaceClient = new NamespaceManager(ServiceBusEnvironment.CreateServiceUri("sb", "<yourNamespace>", string.Empty), credentials);

            QueueDescription myQueue;

            if (namespaceClient.QueueExists("IssueTrackingQueue"))
            {
                namespaceClient.DeleteQueue("IssueTrackingQueue");
            }
            
            myQueue = namespaceClient.CreateQueue("IssueTrackingQueue");
            
            MessagingFactory factory = MessagingFactory.Create(ServiceBusEnvironment.CreateServiceUri("sb", "<yourNamespace>", string.Empty), credentials);

            QueueClient myQueueClient = factory.CreateQueueClient("IssueTrackingQueue");

            // Send messages
            Console.WriteLine("Now sending messages to the queue.");
            for (int count = 0; count < 6; count++)
            {
                var issue = MessageList[count];
                issue.Label = issue.Properties["IssueTitle"].ToString();
                await myQueueClient.SendAsync(issue);
                Console.WriteLine(string.Format("Message sent: {0}, {1}", issue.Label, issue.MessageId));
            }

            Console.WriteLine("Now receiving messages from Queue.");
            BrokeredMessage message;
            while ((message = await myQueueClient.ReceiveAsync(new TimeSpan(hours: 0, minutes: 1, seconds: 5))) != null)
            {
                Console.WriteLine(string.Format("Message received: {0}, {1}, {2}", message.SequenceNumber, message.Label, message.MessageId));
                message.Complete();

                Console.WriteLine("Processing message (sleeping...)");
                Thread.Sleep(1000);
            }

            factory.Close();
            myQueueClient.Close();
            namespaceClient.DeleteQueue("IssueTrackingQueue");


        }

        static void CollectUserInput()
        {
            // User service namespace
            Console.Write("Please enter the namespace to use: ");
            ServiceNamespace = Console.ReadLine();

            // Issuer key
            Console.Write("Enter the SAS key to use: ");
            sasKeyValue = Console.ReadLine();
        }

        static List<BrokeredMessage> GenerateMessages(DataTable issues)
        {
            // Instantiate the brokered list object
            List<BrokeredMessage> result = new List<BrokeredMessage>();

            // Iterate through the table and create a brokered message for each row
            foreach (DataRow item in issues.Rows)
            {
                BrokeredMessage message = new BrokeredMessage();
                foreach (DataColumn property in issues.Columns)
                {
                    message.Properties.Add(property.ColumnName, item[property]);
                }
                result.Add(message);
            }
            return result;
        }

        static DataTable ParseCSVFile()
        {
            DataTable tableIssues = new DataTable("Issues");
            string path = @"..\..\data.csv";
            try
            {
                using (StreamReader readFile = new StreamReader(path))
                {
                    string line;
                    string[] row;

                    // create the columns
                    line = readFile.ReadLine();
                    foreach (string columnTitle in line.Split(','))
                    {
                        tableIssues.Columns.Add(columnTitle);
                    }

                    while ((line = readFile.ReadLine()) != null)
                    {
                        row = line.Split(',');
                        tableIssues.Rows.Add(row);
                    }
                }
            }
            catch (Exception e)
            {
                Console.WriteLine("Error:" + e.ToString());
            }

            return tableIssues;
        }
    }
}
```

## 建置並執行 QueueSample 應用程式

既然您已完成上述步驟，您可以建置並執行 **QueueSample** 應用程式。

### 建置 QueueSample 應用程式

在 Visual Studio 中，按一下 [建置] 功能表中的 [建置方案]，或按 **Ctrl+Shift+B**。如果您遇到錯誤，請根據上一步結尾顯示的完整範例，確認您的程式碼正確無誤。

## 後續步驟

本教學課程示範了如何使用服務匯流排代理傳訊功能，建置服務匯流排用戶端應用程式和服務。如需使用服務匯流排[轉送](service-bus-messaging-overview.md#Relayed-messaging)的類似教學課程，請參閱[服務匯流排轉送傳訊教學課程](../service-bus-relay/service-bus-relay-tutorial.md)。

若要深入了解[服務匯流排](https://azure.microsoft.com/services/service-bus/)，請參閱下列主題。

- [服務匯流排傳訊概觀](service-bus-messaging-overview.md)
- [服務匯流排基本概念](../service-bus/service-bus-fundamentals-hybrid-solutions.md)
- [服務匯流排架構](../service-bus/service-bus-architecture.md)

<!---HONumber=AcomDC_0928_2016-->