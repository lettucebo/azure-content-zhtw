<properties
    pageTitle="開始使用服務匯流排佇列 | Microsoft Azure"
    description="如何撰寫用於服務匯流排訊息的 C# 主控台應用程式"
    services="service-bus-messaging"
    documentationCenter=".net"
    authors="jtaubensee"
    manager="timlt"
    editor=""/>

<tags
    ms.service="service-bus-messaging"
    ms.devlang="tbd"
    ms.topic="hero-article"
    ms.tgt_pltfrm="dotnet"
    ms.workload="na"
    ms.date="08/23/2016"
    ms.author="jotaub;sethm"/>

# 開始使用服務匯流排佇列

[AZURE.INCLUDE [service-bus-selector-queues](../../includes/service-bus-selector-queues.md)]

## 將會完成的工作

在本教學課程中，我們將完成下列工作︰

1. 使用 Azure 入口網站建立服務匯流排命名空間。

2. 使用 Azure 入口網站建立服務匯流排訊息佇列。

3. 撰寫主控台應用程式來傳送訊息。

4. 撰寫主控台應用程式來接收訊息。

## 必要條件

1. [Visual Studio 2013 或 Visual Studio 2015](http://www.visualstudio.com)。本教學課程中的範例使用 Visual Studio 2015。

2. Azure 訂用帳戶。

[AZURE.INCLUDE [create-account-note](../../includes/create-account-note.md)]

## 1\.使用 Azure 入口網站建立命名空間

如果您已經建立服務匯流排命名空間，請跳至[使用 Azure 入口網站建立佇列](#2-create-a-queue-using-the-azure-portal)一節。

[AZURE.INCLUDE [service-bus-create-namespace-portal](../../includes/service-bus-create-namespace-portal.md)]

## 2\.使用 Azure 入口網站建立佇列

如果您已經建立服務匯流排佇列，請跳至[將訊息傳送到佇列](#3-send-messages-to-the-queue)一節。

[AZURE.INCLUDE [service-bus-create-queue-portal](../../includes/service-bus-create-queue-portal.md)]

## 3\.將訊息傳送到佇列

為了將訊息傳送到佇列，我們會使用 Visual Studio 撰寫 C# 主控台應用程式。

### 建立主控台應用程式

1. 啟動 Visual Studio，並建立新的主控台應用程式。

### 新增服務匯流排 NuGet 封裝

1. 以滑鼠右鍵按一下新建立的專案，然後選取 [管理 NuGet 封裝]。

2. 按一下 [瀏覽] 索引標籤，然後搜尋「Microsoft Azure 服務匯流排」並選取 [Microsoft Azure 服務匯流排] 項目。按一下 [安裝] 完成安裝作業，然後關閉此對話方塊。

    ![選取 NuGet 封裝][nuget-pkg]

### 撰寫一些程式碼來將訊息傳送到佇列

1. 在 Program.cs 檔案開頭處新增以下 using 陳述式。

    ```
    using Microsoft.ServiceBus.Messaging;
    ```
    
2. 將下列程式碼新增至 `Main` 方法，將 **connectionString** 變數設定為建立命名空間時所取得的連接字串，以及將 **queueName** 設定為建立佇列時所使用的佇列名稱。

    ```
    var connectionString = "<Your connection string>";
    var queueName = "<Your queue name>";
  
    var client = QueueClient.CreateFromConnectionString(connectionString, queueName);
    var message = new BrokeredMessage("This is a test message!");
    client.Send(message);
    ```

    Program.cs 看起來應該會像下面這樣。

    ```
    using System;
    using Microsoft.ServiceBus.Messaging;

    namespace GettingStartedWithQueues
    {
        class Program
        {
            static void Main(string[] args)
            {
                var connectionString = "<Your connection string>";
                var queueName = "<Your queue name>";

                var client = QueueClient.CreateFromConnectionString(connectionString, queueName);
                var message = new BrokeredMessage("This is a test message!");

                client.Send(message);
            }
        }
    }
    ```
  
3. 執行程式，並檢查 Azure 入口網站。在命名空間 [概觀] 刀鋒視窗中按一下您的佇列名稱。請注意，[使用中訊息計數] 值現在應該是 1。
    
      ![訊息計數][queue-message]
    
## 4\.從佇列接收訊息

1. 建立新的主控台應用程式，並和上面的傳送應用程式類似，新增服務匯流排 NuGet 封裝的參考。

2. 在 Program.cs 檔案開頭處新增以下 `using` 陳述式。
  
    ```
    using Microsoft.ServiceBus.Messaging;
    ```
  
3. 將下列程式碼新增至 `Main` 方法，將 **connectionString** 變數設定為建立命名空間時所取得的連接字串，以及將 **queueName** 設定為建立佇列時所使用的佇列名稱。

    ```
    var connectionString = "";
    var queueName = "samplequeue";
  
    var client = QueueClient.CreateFromConnectionString(connectionString, queueName);
  
    client.OnMessage(message =>
    {
      Console.WriteLine(String.Format("Message body: {0}", message.GetBody<String>()));
      Console.WriteLine(String.Format("Message id: {0}", message.MessageId));
    });
  
    Console.ReadLine();
    ```

	Program.cs 檔案看起來應該會像下面這樣：

    ```
    using System;
    using Microsoft.ServiceBus.Messaging;
  
    namespace GettingStartedWithQueues
    {
      class Program
      {
        static void Main(string[] args)
        {
          var connectionString = "";
          var queueName = "samplequeue";
  
          var client = QueueClient.CreateFromConnectionString(connectionString, queueName);
  
          client.OnMessage(message =>
          {
            Console.WriteLine(String.Format("Message body: {0}", message.GetBody<String>()));
            Console.WriteLine(String.Format("Message id: {0}", message.MessageId));
          });
  
          Console.ReadLine();
        }
      }
    }
    ```
  
4. 執行程式，並檢查入口網站。請注意，**佇列長度**值現在應該是 0。

    ![佇列長度][queue-message-receive]
  
恭喜！ 您現已建立佇列、傳送訊息和接收訊息。

## 後續步驟

查看 [GitHub 儲存機制以及範例](https://github.com/Azure-Samples/azure-servicebus-messaging-samples)，其中會展示一些更進階的 Azure 服務匯流排訊息功能。

<!--Image references-->

[nuget-pkg]: ./media/service-bus-dotnet-get-started-with-queues/nuget-package.png
[queue-message]: ./media/service-bus-dotnet-get-started-with-queues/queue-message.png
[queue-message-receive]: ./media/service-bus-dotnet-get-started-with-queues/queue-message-receive.png


<!--Reference style links - using these makes the source content way more readable than using inline links-->

[github-samples]: https://github.com/Azure-Samples/azure-servicebus-messaging-samples

<!---HONumber=AcomDC_0928_2016-->