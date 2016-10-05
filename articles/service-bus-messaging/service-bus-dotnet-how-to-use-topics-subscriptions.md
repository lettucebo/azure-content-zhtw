<properties
    pageTitle="搭配 .NET 使用服務匯流排主題 | Microsoft Azure"
    description="了解如何在 Azure 中搭配 .NET 使用服務匯流排主題及訂用帳戶。程式碼範例專為 .NET 應用程式撰寫。"
    services="service-bus-messaging"
    documentationCenter=".net"
    authors="sethmanheim"
    manager="timlt"
    editor=""/>

<tags
    ms.service="service-bus-messaging"
    ms.workload="na"
    ms.tgt_pltfrm="na"
    ms.devlang="dotnet"
    ms.topic="get-started-article"
    ms.date="09/16/2016"
    ms.author="sethm"/>

# 如何使用服務匯流排主題和訂用帳戶

[AZURE.INCLUDE [service-bus-selector-topics](../../includes/service-bus-selector-topics.md)]

本文說明如何使用服務匯流排主題和訂用帳戶。這些範例均以 C# 撰寫並使用 .NET API。所涵蓋的案例包括建立主題和訂用帳戶、建立訂用帳戶篩選器、傳送訊息至主題、接收訂用帳戶的訊息及刪除主題和訂用帳戶。如需主題和訂用帳戶的詳細資訊，請參閱[後續步驟](#next-steps)一節。

[AZURE.INCLUDE [create-account-note](../../includes/create-account-note.md)]

[AZURE.INCLUDE [howto-service-bus-topics](../../includes/howto-service-bus-topics.md)]

## 設定應用程式以使用服務匯流排

當您建立使用服務匯流排的應用程式時，您必須新增服務匯流排組件的參照，並包含對應的命名空間。執行這項工作最簡單的方式便是下載適當的 [NuGet](https://www.nuget.org) 封裝。

## 取得服務匯流排 NuGet 封裝

[服務匯流排 NuGet 套件](https://www.nuget.org/packages/WindowsAzure.ServiceBus)為取得服務匯流排 API，並設定具有所有必要服務匯流排相依性的應用程式的最容易方式。若要在專案中安裝服務匯流排 NuGet 封裝，請執行下列動作：

1.  在 [方案總管] 中，以滑鼠右鍵按一下 [喜好設定]，然後按一下 [Manage NuGet Packages]。
2.  搜尋「服務匯流排」並選取 [Microsoft Azure 服務匯流排] 項目。按一下 [安裝] 完成安裝作業，然後關閉下列對話方塊：

    ![][7]

您現在可以開始對服務匯流排撰寫程式碼。

## 建立服務匯流排連接字串

服務匯流排使用連接字串來儲存端點和認證。您可以將連接字串置於設定檔中，而非硬式編碼它：

- 使用 Azure 服務時，建議您使用 Azure 服務組態系統 (.csdef 和 .cscfg 檔案) 來儲存連接字串。
- 使用 Azure 網站或 Azure 虛擬機器時，建議您使用 .NET 組態系統 (例如 Web.config 檔案) 來儲存連接字串。

在這兩種情況下，您都可以使用 `CloudConfigurationManager.GetSetting` 方法擷取連接字串，如本文稍後所示範。

### 設定連接字串

服務組態機制可讓您從 [Azure 入口網站][]動態變更組態設定，而無需重新部署應用程式。例如，在您的服務定義 (**.csdef**) 檔案中新增 `Setting` 標籤，如下個範例所示。

```
<ServiceDefinition name="Azure1">
...
    <WebRole name="MyRole" vmsize="Small">
        <ConfigurationSettings>
            <Setting name="Microsoft.ServiceBus.ConnectionString" />
        </ConfigurationSettings>
    </WebRole>
...
</ServiceDefinition>
```

接著您可以在服務組態 (.cscfg) 檔案中指定值。

```
<ServiceConfiguration serviceName="Azure1">
...
    <Role name="MyRole">
        <ConfigurationSettings>
            <Setting name="Microsoft.ServiceBus.ConnectionString"
                     value="Endpoint=sb://yourServiceNamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=yourKey" />
        </ConfigurationSettings>
    </Role>
...
</ServiceConfiguration>
```

如先前所述，使用從入口網站擷取的共用存取簽章 (SAS) 金鑰名稱和金鑰值。

### 在使用 Azure 網站或 Azure 虛擬機器時設定連接字串

使用網站或虛擬機器時，建議您使用 .NET 組態系統 (例如，Web.config)。您可以使用 `<appSettings>` 元素儲存連接字串。

```
<configuration>
    <appSettings>
        <add key="Microsoft.ServiceBus.ConnectionString"
             value="Endpoint=sb://yourServiceNamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=yourKey" />
    </appSettings>
</configuration>
```

如先前所述，使用從 [Azure 入口網站][]擷取的 SAS 名稱和金鑰值。

## 建立主題

您可以透過 [NamespaceManager](https://msdn.microsoft.com/library/azure/microsoft.servicebus.namespacemanager.aspx) 類別，來執行服務匯流排主題和訂閱的管理作業。此類別提供建立、列舉及刪除主題的方法。

下列範例會使用 Azure `CloudConfigurationManager` 類別，以及包含服務匯流排命名空間基底位址的連接字串和具備管理此連接字串權限的適當 SAS 認證，來建構 `NamespaceManager` 物件。這個連接字串的格式如下：

```
Endpoint=sb://<yourNamespace>.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=<yourKey>
```

請使用下例，假設組態設定如上一節。

```
// Create the topic if it does not exist already.
string connectionString =
    CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

var namespaceManager =
    NamespaceManager.CreateFromConnectionString(connectionString);

if (!namespaceManager.TopicExists("TestTopic"))
{
    namespaceManager.CreateTopic("TestTopic");
}
```

[CreateTopic](https://msdn.microsoft.com/library/azure/microsoft.servicebus.namespacemanager.createtopic.aspx) 方法的超載可讓您設定主題的屬性 (例如，設定要套用至傳送至主題之訊息的預設存留時間 (TTL) 值)。您可以使用 [TopicDescription](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.topicdescription.aspx) 類別來套用這些設定。下列範例將說明如何建立大小上限為 5 GB、預設訊息存留時間為 1 分鐘，且名為 **TestTopic** 的主題。

```
// Configure Topic Settings.
TopicDescription td = new TopicDescription("TestTopic");
td.MaxSizeInMegabytes = 5120;
td.DefaultMessageTimeToLive = new TimeSpan(0, 1, 0);

// Create a new Topic with custom settings.
string connectionString =
    CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

var namespaceManager =
    NamespaceManager.CreateFromConnectionString(connectionString);

if (!namespaceManager.TopicExists("TestTopic"))
{
    namespaceManager.CreateTopic(td);
}
```

> [AZURE.NOTE] 您可以在 [NamespaceManager](https://msdn.microsoft.com/library/azure/microsoft.servicebus.namespacemanager.aspx) 物件上使用 [TopicExists](https://msdn.microsoft.com/library/azure/microsoft.servicebus.namespacemanager.topicexists.aspx) 方法，來檢查服務命名空間內是否已有指定名稱的主題存在。

## 建立訂用帳戶

您也可以使用 [NamespaceManager](https://msdn.microsoft.com/library/azure/microsoft.servicebus.namespacemanager.aspx) 類別來建立主題訂閱。為訂閱命名，且能包含選擇性篩選器，以用來限制傳遞至訂閱的虛擬佇列的訊息集合。

### 使用預設 (MatchAll) 篩選器建立訂用帳戶

如果在建立新的訂用帳戶時未指定篩選器，**MatchAll** 篩選器就會是預設使用的篩選器。使用 **MatchAll** 篩選器時，所有發佈至主題的訊息都會被置於訂用帳戶的虛擬佇列中。下列範例將建立名為 "AllMessages" 的訂閱，並使用預設的 **MatchAll** 篩選器。

```
string connectionString =
    CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

var namespaceManager =
    NamespaceManager.CreateFromConnectionString(connectionString);

if (!namespaceManager.SubscriptionExists("TestTopic", "AllMessages"))
{
    namespaceManager.CreateSubscription("TestTopic", "AllMessages");
}
```

### 使用篩選器建立訂用帳戶

您也可以設定篩選器，讓您指定傳送至主題的哪些訊息應出現在特定主題訂用帳戶中。

訂用帳戶所支援的最具彈性篩選器類型是實作 SQL92 子集的 [SqlFilter][] 類別。SQL 篩選器會對發佈至主題之訊息的屬性運作。如需可與 SQL 篩選器搭配使用的運算式詳細資訊，請參閱 [SqlFilter.SqlExpression][] 語法。

以下範例將建立名為 **HighMessages** 的訂用帳戶，其帶有只選取自訂 **MessageNumber** 屬性大於 3 之訊息的 [SqlFilter][] 物件。

```
// Create a "HighMessages" filtered subscription.
SqlFilter highMessagesFilter =
   new SqlFilter("MessageNumber > 3");

namespaceManager.CreateSubscription("TestTopic",
   "HighMessages",
   highMessagesFilter);
```

同樣地，下列範例將建立名為 **LowMessages** 並帶有只選取 **MessageNumber** 屬性小於或等於 3 之訊息的 [SqlFilter][] 訂用帳戶。

```
// Create a "LowMessages" filtered subscription.
SqlFilter lowMessagesFilter =
   new SqlFilter("MessageNumber <= 3");

namespaceManager.CreateSubscription("TestTopic",
   "LowMessages",
   lowMessagesFilter);
```

現在當訊息傳送到 `TestTopic` 時，一律會將該訊息傳遞到已訂閱 **AllMessages** 主題訂用帳戶的接收者，並選擇性地將它傳遞到已訂閱 **HighMessages** 和 **LowMessages** 主題訂用帳戶的接收者 (視訊息內容而定)。

## 傳送訊息至主題

若要傳送訊息至服務匯流排主題，應用程式會使用連接字串來建立 [TopicClient](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.topicclient.aspx) 物件。

下列程式碼將示範如何使用 [`CreateFromConnectionString`](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.topicclient.createfromconnectionstring.aspx) API 呼叫，為之前建立的 **TestTopic** 主題建立 [TopicClient](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.topicclient.aspx) 物件。

```
string connectionString =
    CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

TopicClient Client =
    TopicClient.CreateFromConnectionString(connectionString, "TestTopic");

Client.Send(new BrokeredMessage());
```

傳送至服務匯流排主題的訊息是 [BrokeredMessage](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.aspx) 類別的執行個體。[BrokeredMessage](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.aspx) 物件具有一組標準屬性 (例如 [Label](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.label.aspx) 和 [TimeToLive](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.timetolive.aspx))、一個用來保存自訂應用程式特定屬性的目錄，以及一堆任意的應用程式資料。應用程式可設定訊息內文，方法是將任何可序列化物件傳遞到 [BrokeredMessage](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.aspx) 物件的建構函式，接著系統便會使用適當的 **DataContractSerializer** 來序列化物件。也可以提供 **System.IO.Stream**。

下列範例將示範如何傳送五則測試訊息至上述程式碼範例中所取得的 **TestTopic** [TopicClient](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.topicclient.aspx) 物件。請注意，迴圈反覆運算上每個訊息的 [MessageNumber](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.properties.aspx) 屬性值會有變化 (這可判斷接收訊息的訂用帳戶為何)。

```
for (int i=0; i<5; i++)
{
  // Create message, passing a string message for the body.
  BrokeredMessage message = new BrokeredMessage("Test message " + i);

  // Set additional custom app-specific property.
  message.Properties["MessageNumber"] = i;

  // Send message to the topic.
  Client.Send(message);
}
```

服務匯流排主題支援的訊息大小上限：在[標準層](service-bus-premium-messaging.md)中為 256 KB 以及在[進階層](service-bus-premium-messaging.md)中為 1 MB。標頭 (包含標準和自訂應用程式屬性) 可以容納 64 KB 的大小上限。主題中所保存的訊息數目沒有限制，但主題所保存的訊息大小總計會有最高限制。此主題大小會在建立時定義，上限是 5 GB。如果啟用分割，上限會更高。如需詳細資訊，請參閱[分割傳訊實體](service-bus-partitioning.md)。

## 如何自訂用帳戶接收訊息

自訂用帳戶接收訊息的建議方式是使用 [SubscriptionClient](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.subscriptionclient.aspx) 物件。[SubscriptionClient](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.subscriptionclient.aspx) 物件可在兩種不同的模式下運作：[ReceiveAndDelete 和 PeekLock](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.receivemode.aspx)。

使用 **ReceiveAndDelete** 模式時，接收是一次性作業；也就是說，當服務匯流排在訂用帳戶中收到訊息的讀取要求時，它會將此訊息標示為已使用，並將它傳回應用程式。**ReceiveAndDelete** 模式是最簡單的模型，且最適合可容許在發生失敗時不處理訊息的應用程式案例。若要了解這一點，請考慮取用者發出接收要求，接著系統在處理此要求之前當機的案例。因為服務匯流排已將訊息標示為已取用，當應用程式重新啟動並開始重新取用訊息時，它將會遺漏當機前已取用的訊息。

在 **PeekLock** 模式 (此為預設模式) 中，接收程序會變成兩階段作業，因此可以支援無法容許遺漏訊息的應用程式。當服務匯流排收到要求時，它會尋找要取用的下一個訊息、將其鎖定以防止其他取用者接收此訊息，然後將它傳回應用程式。在應用程式完成處理訊息 (或可靠地儲存此訊息以供未來處理) 之後，它可透過呼叫所接收訊息上的 [Complete](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.complete.aspx)，來完成接收程序的第二個階段。當服務匯流排看到 **Complete** 呼叫時，它會將訊息標示為已取用，並將它從訂用帳戶中移除。

以下範例將示範如何使用預設的 **PeekLock** 模式來接收與處理訊息。若要指定其他 [ReceiveMode](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.receivemode.aspx) 值，您可以使用 [CreateFromConnectionString](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.subscriptionclient.createfromconnectionstring.aspx) 的另一個超載。這個範例會使用 [OnMessage](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.subscriptionclient.onmessage.aspx) 回呼在訊息到達 **HighMessages** 訂用帳戶時處理訊息。

```
string connectionString =
    CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

SubscriptionClient Client =
    SubscriptionClient.CreateFromConnectionString
            (connectionString, "TestTopic", "HighMessages");

// Configure the callback options.
OnMessageOptions options = new OnMessageOptions();
options.AutoComplete = false;
options.AutoRenewTimeout = TimeSpan.FromMinutes(1);

Client.OnMessage((message) =>
{
    try
    {
        // Process message from subscription.
        Console.WriteLine("\n**High Messages**");
        Console.WriteLine("Body: " + message.GetBody<string>());
        Console.WriteLine("MessageID: " + message.MessageId);
        Console.WriteLine("Message Number: " +
            message.Properties["MessageNumber"]);

        // Remove message from subscription.
        message.Complete();
    }
    catch (Exception)
    {
        // Indicates a problem, unlock message in subscription.
        message.Abandon();
    }
}, options);
```

此範例會使用 [OnMessageOptions](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.onmessageoptions.aspx) 物件設定 [OnMessage](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.subscriptionclient.onmessage.aspx) 回呼。[AutoComplete](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.onmessageoptions.autocomplete.aspx) 設為 **false**，以手動控制何時要在接收的訊息上呼叫 [Complete](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.complete.aspx)。[AutoRenewTimeout](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.onmessageoptions.autorenewtimeout.aspx) 設為 1 分鐘，這會導致用戶端在終止自動更新功能前，最多等候一分鐘的時間，而後用戶端會進行新的呼叫來檢查訊息。這個屬性值會減少用戶端無法擷取訊息所出現的收費呼叫次數。

## 如何處理應用程式當機與無法讀取的訊息

服務匯流排提供一種功能，可協助您從應用程式的錯誤或處理訊息的問題中順利復原。如果接收端應用程式因為某些原因無法處理訊息，它可以呼叫所接收訊息上的 [Abandon](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.abandon.aspx) 方法 (而不是 [Complete](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.complete.aspx) 方法)。這導致服務匯流排將訂閱中的訊息解除鎖定，讓此訊息可以被相同取用應用程式或其他取用應用程式重新接收。

與訂用帳戶內鎖定訊息相關的還有逾時，如果應用程式無法在鎖定逾時到期之前處理訊息 (例如，如果應用程式當機)，則服務匯流排會自動解除鎖定訊息，並讓訊息可以被重新接收。

如果應用程式在處理訊息之後但尚未發出 [Complete](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.complete.aspx) 要求時當機，則會在應用程式重新啟動時將訊息重新傳遞給該應用程式。這通常稱為*至少處理一次*，也就是說，每個訊息至少會被處理一次，但在特定狀況下，可能會重新傳遞相同訊息。如果案例無法容許重複處理，則應用程式開發人員應在其應用程式中加入其他邏輯，以處理重複的訊息傳遞。通常您可使用訊息的 [MessageId](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.messageid.aspx) 屬性來達到此目的，該屬性會在各個傳遞嘗試中會保持不變。

## 刪除主題和訂用帳戶

以下範例將示範如何從 **HowToSample** 服務命名空間中刪除 **TestTopic** 主題。

```
// Delete Topic.
namespaceManager.DeleteTopic("TestTopic");
```

刪除主題也將會刪除對主題註冊的任何訂用帳戶。您也可以個別刪除訂用帳戶。下列程式碼將示範如何將名為 **HighMessages** 的訂用帳戶從 **TestTopic** 主題中刪除。

```
namespaceManager.DeleteSubscription("TestTopic", "HighMessages");
```

## 後續步驟

了解基本的服務匯流排主題和訂用帳戶之後，請參考下列連結以取得更多資訊。

-   [佇列、主題和訂閱][]。
-   [主題篩選器範例][]
-   [SqlFilter][] 的 API 參考資料。
-   建立一個可行的應用程式，往返傳送或接收服務匯流排佇列的訊息：[服務匯流排代理傳訊 .NET 教學課程][]。
-   服務匯流排範例：從 [Azure 範例][]下載，或參閱[概觀](../service-bus/service-bus-samples.md)。

  [Azure 入口網站]: https://portal.azure.com

  [7]: ./media/service-bus-dotnet-how-to-use-topics-subscriptions/getting-started-multi-tier-13.png

  [佇列、主題和訂閱]: service-bus-queues-topics-subscriptions.md
  [主題篩選器範例]: https://github.com/Azure-Samples/azure-servicebus-messaging-samples/tree/master/TopicFilters
  [SqlFilter]: http://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.sqlfilter.aspx
  [SqlFilter.SqlExpression]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.sqlfilter.sqlexpression.aspx
  [服務匯流排代理傳訊 .NET 教學課程]: service-bus-brokered-tutorial-dotnet.md
  [Azure 範例]: https://code.msdn.microsoft.com/site/search?query=service%20bus&f%5B0%5D.Value=service%20bus&f%5B0%5D.Type=SearchText&ac=2

<!---HONumber=AcomDC_0928_2016-->