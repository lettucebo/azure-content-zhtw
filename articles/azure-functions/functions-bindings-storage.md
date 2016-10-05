<properties
	pageTitle="Azure 儲存體的 Azure Functions 觸發程序和繫結 | Microsoft Azure"
	description="瞭解如何在 Azure Functions 中使用「Azure 儲存體」觸發程序和繫結。"
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
	ms.author="chrande"/>

# Azure 儲存體的 Azure Functions 觸發程序和繫結

[AZURE.INCLUDE [functions-selector-bindings](../../includes/functions-selector-bindings.md)]

這篇文章說明如何在 Azure Functions 中為「Azure 儲存體」觸發程序和繫結進行設定及撰寫程式碼。

[AZURE.INCLUDE [簡介](../../includes/functions-bindings-intro.md)]

## <a id="storagequeuetrigger"></a>Azure 儲存體佇列觸發程序

#### 儲存體佇列觸發程序的 function.json

「function.json」檔案指定了下列屬性。

- `name`︰函式程式碼中用於佇列或佇列訊息的變數名稱。
- `queueName`：要輪詢的佇列名稱。若要了解佇列命名規則，請參閱[為佇列和中繼資料命名](https://msdn.microsoft.com/library/dd179349.aspx)。
- `connection`︰包含儲存體連接字串的應用程式設定名稱。如果將 `connection` 留白，觸發程序將會使用函式應用程式的預設儲存體連接字串，此預設值是由 AzureWebJobsStorage 應用程式設定所指定。
- `type`︰必須設為「queueTrigger」。
- `direction`：必須設為「in」。

儲存體佇列觸發程序的範例「function.json」：

```json
{
    "disabled": false,
    "bindings": [
        {
            "name": "myQueueItem",
            "queueName": "myqueue-items",
            "connection":"",
            "type": "queueTrigger",
            "direction": "in"
        }
    ]
}
```

#### 佇列觸發程序支援的類型

佇列訊息可以還原序列化為下列任何一種類型︰

* 物件 (來自 JSON)
* String
* 位元組陣列
* `CloudQueueMessage` (C#)

#### 佇列觸發程序中繼資料

您可以使用這些變數名稱，在函式中取得佇列中繼資料︰

* expirationTime
* insertionTime
* nextVisibleTime
* id
* popReceipt
* dequeueCount
* queueTrigger (將佇列訊息文字做為字串擷取的另一種方式)

這個 C# 程式碼範例會擷取和記錄佇列中繼資料︰

```csharp
public static void Run(string myQueueItem, 
    DateTimeOffset expirationTime, 
    DateTimeOffset insertionTime, 
    DateTimeOffset nextVisibleTime,
    string queueTrigger,
    string id,
    string popReceipt,
    int dequeueCount,
    TraceWriter log)
{
    log.Info($"C# Queue trigger function processed: {myQueueItem}\n" +
        $"queueTrigger={queueTrigger}\n" +
        $"expirationTime={expirationTime}\n" +
        $"insertionTime={insertionTime}\n" +
        $"nextVisibleTime={nextVisibleTime}\n" +
        $"id={id}\n" +
        $"popReceipt={popReceipt}\n" + 
        $"dequeueCount={dequeueCount}");
}
```

#### 處理有害的佇列訊息

內容會導致函數失敗的訊息稱為「有害訊息」。當函數失敗時不會刪除佇列訊息，最後會再度挑選到該訊息，造成重複循環。SDK 可在有限的反覆次數之後自動中斷循環，或者您可以手動中斷循環。

SDK 將會呼叫函數最多 5 次以處理佇列訊息。如果第五次嘗試失敗，訊息便會移到有害佇列中。

有害佇列名為 *{originalqueuename}*-poison。您可以撰寫函數，透過記錄或傳送通知表示需要手動處理，來處理有害佇列中的訊息。

如果您想要手動處理有害訊息，您可以藉由檢查 `dequeueCount` 來取得訊息已被挑選來處理的次數。

## <a id="storagequeueoutput"></a>Azure 儲存體佇列輸出繫結

#### 儲存體佇列輸出繫結的 function.json

「function.json」檔案指定了下列屬性。

- `name`︰函式程式碼中用於佇列或佇列訊息的變數名稱。
- `queueName`：佇列的名稱。若要了解佇列命名規則，請參閱[為佇列和中繼資料命名](https://msdn.microsoft.com/library/dd179349.aspx)。
- `connection`︰包含儲存體連接字串的應用程式設定名稱。如果將 `connection` 留白，觸發程序將會使用函式應用程式的預設儲存體連接字串，此預設值是由 AzureWebJobsStorage 應用程式設定所指定。
- `type`：必須設為「queue」。
- `direction`：必須設為「out」。

儲存體佇列輸出繫結的範例「function.json」，此繫結會使用佇列觸發程序並寫入佇列訊息︰

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnection",
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "myQueue",
      "queueName": "samples-workitems-out",
      "connection": "MyStorageConnection",
      "type": "queue",
      "direction": "out"
    }
  ],
  "disabled": false
}
``` 

#### 佇列輸出繫結支援的類型

`queue` 繫結可將下列類型序列化到佇列訊息︰

* 物件 (在 C# 中為 `out T`，如果函式結束時，參數為 Null，就會使用 Null 物件來建立訊息)
* 字串 (在 C# 中為 `out string`，如果函式結束時，參數值不是 Null，就會建立佇列訊息)
* 位元組陣列 (在 C# 中為 `out byte[]`，運作方式與字串相同)
* `out CloudQueueMessage`(C#，運作方式與字串相同)

在 C# 中，您也可以繫結至 `ICollector<T>` 或 `IAsyncCollector<T>`，其中 `T` 是其中一個支援的類型。

#### 佇列輸出繫結程式碼範例

這個 C# 程式碼範例將為每個輸入佇列訊息，撰寫單一輸出佇列訊息。

```csharp
public static void Run(string myQueueItem, out string myOutputQueueItem, TraceWriter log)
{
    myOutputQueueItem = myQueueItem + "(next step)";
}
```

這個 C# 程式碼範例會使用 `ICollector<T>` (在非同步函式中會使用 `IAsyncCollector<T>`) 來撰寫多個訊息：

```csharp
public static void Run(string myQueueItem, ICollector<string> myQueue, TraceWriter log)
{
    myQueue.Add(myQueueItem + "(step 1)");
    myQueue.Add(myQueueItem + "(step 2)");
}
```

## <a id="storageblobtrigger"></a>Azure 儲存體 Blob 觸發程序

#### 儲存體 Blob 觸發程序的 function.json

「function.json」檔案指定了下列屬性。

- `name`︰函式程式碼中用於 Blob 的變數名稱。
- `path`：指定要監視之容器的路徑，此路徑也可以依選擇指定 Blob 名稱模式。
- `connection`︰包含儲存體連接字串的應用程式設定名稱。如果將 `connection` 留白，觸發程序將會使用函式應用程式的預設儲存體連接字串，此預設值是由 AzureWebJobsStorage 應用程式設定所指定。
- `type`︰必須設為「blobTrigger」。
- `direction`：必須設為「in」。

儲存體 Blob 觸發程序的範例「function.json」，此觸發程序會監看新增至 samples-workitems 容器的 Blob：

```json
{
    "disabled": false,
    "bindings": [
        {
            "name": "myBlob",
            "type": "blobTrigger",
            "direction": "in",
            "path": "samples-workitems",
            "connection":""
        }
    ]
}
```

#### Blob 觸發程序支援的類型

在 Node 或 C# 函式中，Blob 可以還原序列化為下列任何類型︰

* 物件 (來自 JSON)
* String

在 C# 函式中，您也可以繫結至下列任何類型︰

* `TextReader`
* `Stream`
* `ICloudBlob`
* `CloudBlockBlob`
* `CloudPageBlob`
* `CloudBlobContainer`
* `CloudBlobDirectory`
* `IEnumerable<CloudBlockBlob>`
* `IEnumerable<CloudPageBlob>`
* 透過 [ICloudBlobStreamBinder](../app-service-web/websites-dotnet-webjobs-sdk-storage-blobs-how-to.md#icbsb) 還原序列化的其他型別

#### Blob 觸發程序 C# 程式碼範例

這個 C# 程式碼範例會將每個新增的 Blob 內容記錄到受監視的容器。

```csharp
public static void Run(string myBlob, TraceWriter log)
{
    log.Info($"C# Blob trigger function processed: {myBlob}");
}
```

#### Blob 觸發程序名稱模式

您可以在 `path` 屬性中指定 Blob 名稱模式。例如：

```json
"path": "input/original-{name}",
```

此路徑會在「input」容器中尋找名為「original-Blob1.txt」的 Blob，且函式程式碼中的 `name` 變數值會是 `Blob1`。

另一個範例：

```json
"path": "input/{blobname}.{blobextension}",
```

此路徑也會尋找名為「original-Blob1.txt」的 Blob，而函式程式碼中的 `blobname` 和 `blobextension` 變數值會是「original-Blob1」和「txt」。

您可以使用固定值來為副檔名指定模式，從而限制觸發函式的 blob 類型。如果您將 `path` 設為「samples/{name}.png」，只有「samples」容器中的「.png」Blob 會觸發函式。

如果您需要針對名稱中包含大括號的 Blob 名稱指定名稱模式，請按兩下大括號。例如，如果您想要在 *images* 容器中尋找具備如下名稱的 Blob：

		{20140101}-soundfile.mp3

將以下內容用於 `path` 屬性︰

		images/{{20140101}}-{name}

在此範例中，`name` 變數值會是「soundfile.mp3」。

#### Blob 回條

Azure Functions 執行階段可確保不會針對一樣新或版本相同的 blob 多次呼叫 blob 觸發程序函式。它的運作方式是藉由維護 *Blob 回條*來判斷指定的 Blob 版本是否已處理過。

Blob 回條儲存於 AzureWebJobsStorage 連接字串所指定之 Azure 儲存體帳戶中名為 *azure-webjobs-hosts* 的容器中。Blob 回條具有下列資訊：

* 已為 Blob 呼叫的函式 ("{function app name}.Functions.{function name}"，例如："functionsf74b96f7.Functions.CopyBlob")
* 容器名稱
* Blob 類型 ("BlockBlob" 或 "PageBlob")
* Blob 名稱
* ETag (Blob 版本識別碼，例如："0x8D1DC6E70A277EF")

如果您想要強制重新處理某個 Blob，可以從 *azure-webjobs-hosts* 容器中手動刪除該 Blob 的 Blob 回條。

#### 處理有害的 blob

當 blob 觸發程序函式失敗時，SDK 會再次加以呼叫，以防失敗是因暫時性錯誤而造成。如果失敗是因為 Blob 的內容所造成，則此函數會在其每次嘗試處理該 Blob 時失敗。根據預設，SDK 最多會針對指定的 Blob 呼叫函數 5 次。如果第五次嘗試失敗，則 SDK 會在名為 *webjobs-blobtrigger-poison* 的佇列中新增一則訊息。

適用於有害 Blob 的佇列訊息是一個 JSON 物件，其中包含下列屬性：

* FunctionId (格式為 {function app name}.Functions.{function name})
* BlobType ("BlockBlob" 或 "PageBlob")
* ContainerName
* BlobName
* ETag (Blob 版本識別碼，例如："0x8D1DC6E70A277EF")

#### 大型容器的 Blob 輪詢

如果觸發程序正在監視的 blob 容器包含超過 10,000 個 blob，Functions 執行階段會掃描記錄檔以監看新增或變更的 blob。此程序的速度很慢；可能直到建立 Blob 之後數分鐘或更久，才會觸發函數。此外，[儲存體記錄檔建立在「竭盡所能」](https://msdn.microsoft.com/library/azure/hh343262.aspx)的基礎上；並不保證會擷取所有事件。在某些情況下可能會遺失記錄檔。如果您的應用程式無法接受大容器 blob 觸發程序的速度和可靠性限制，建議的方法是當您建立 blob 時建立佇列訊息，並使用佇列觸發程序而非 blob 觸發程序來處理 blob。
 
## <a id="storageblobbindings"></a>Azure 儲存體 Blob 輸入和輸出繫結

#### 儲存體 Blob 輸入或輸出繫結的 function.json

「function.json」檔案指定了下列屬性。

- `name`︰函式程式碼中用於 Blob 的變數名稱。
- `path`：指定要從中讀取 Blob 或將 Blob 寫入到其中的容器路徑，此路徑也可以依選擇指定 Blob 名稱模式。
- `connection`︰包含儲存體連接字串的應用程式設定名稱。如果將 `connection` 留白，繫結將會使用函式應用程式的預設儲存體連接字串，此預設值是由 AzureWebJobsStorage 應用程式設定所指定。
- `type`：必須設為「blob」。
- `direction`︰設定為「in」或「out」。

儲存體 Blob 輸入或輸出繫結的範例「function.json」，其中使用佇列觸發程序來複製 Blob：

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnection",
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "myInputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}",
      "connection": "MyStorageConnection",
      "direction": "in"
    },
    {
      "name": "myOutputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}-Copy",
      "connection": "MyStorageConnection",
      "direction": "out"
    }
  ],
  "disabled": false
}
``` 

#### Blob 輸入和輸出支援的類型

在 Node.js 或 C# 函式中，`blob` 繫結可將下列類型序列化或還原序列化︰

* 物件 (在 C# 中為 `out T`，適用於輸出 Blob︰如果函式結束時，參數值為 Null，就會將 Blob 建立成 Null 物件)
* 字串 (在 C# 中為 `out string`，適用於輸出 Blob：如果函式傳回時，字串參數不是 Null，則只會建立 Blob)

在 C# 函式中，您也可以繫結至下列類型︰

* `TextReader` (僅限輸入)
* `TextWriter` (僅限輸出)
* `Stream`
* `CloudBlobStream` (僅限輸出)
* `ICloudBlob`
* `CloudBlockBlob`
* `CloudPageBlob`

#### Blob 輸出 C# 程式碼範例

這個 C# 程式碼範例會複製名稱在佇列訊息中接收的 blob。

```csharp
public static void Run(string myQueueItem, string myInputBlob, out string myOutputBlob, TraceWriter log)
{
    log.Info($"C# Queue trigger function processed: {myQueueItem}");
    myOutputBlob = myInputBlob;
}
```

## <a id="storagetablesbindings"></a>Azure 儲存體資料表輸入和輸出繫結

#### 儲存體資料表的 function.json

「function.json」指定了下列屬性。

- `name`︰函式程式碼中用於資料表繫結的變數名稱。
- `tableName`：資料表的名稱。
- `partitionKey` 和 `rowKey`：搭配使用可讀取 C# 或 Node 函式中的單一實體，或在 Node 函式中寫入單一實體。
- `take`：針對 Node 函式中資料表輸入所讀取的資料列數目上限。
- `filter`：適用於 Node 函式中資料表輸入的 OData 篩選運算式。
- `connection`︰包含儲存體連接字串的應用程式設定名稱。如果將 `connection` 留白，繫結將會使用函式應用程式的預設儲存體連接字串，此預設值是由 AzureWebJobsStorage 應用程式設定所指定。
- `type`：必須設為「table」。
- `direction`︰設定為「in」或「out」。

以下的範例「function.json」會使用佇列觸發程序來讀取單一資料表列。JSON 提供硬式編碼的資料分割索引鍵值，並指定資料列索引鍵來自佇列訊息。

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnection",
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "personEntity",
      "type": "table",
      "tableName": "Person",
      "partitionKey": "Test",
      "rowKey": "{queueTrigger}",
      "connection": "MyStorageConnection",
      "direction": "in"
    }
  ],
  "disabled": false
}
```

#### 儲存體資料表輸入和輸出支援的類型

在 Node.js 或 C# 函式中，`table` 繫結可將物件序列化或還原序列化。物件會有 RowKey 和 PartitionKey 屬性。

在 C# 函式中，您也可以繫結至下列類型︰

* `T`，其中 `T` 會實作 `ITableEntity`
* `IQueryable<T>` (僅限輸入)
* `ICollector<T>` (僅限輸出)
* `IAsyncCollector<T>` (僅限輸出)

#### 儲存體資料表繫結案例

資料表繫結支援下列案例：

* 讀取 C# 或 Node 函式中的單一資料列。

	請設定 `partitionKey` 和 `rowKey`。此案例中不使用 `filter` 和 `take` 屬性。

* 讀取 C# 函式中的多個資料列。

	Functions 執行階段會提供一個繫結至資料表的 `IQueryable<T>` 物件。類型 `T` 必須衍生自 `TableEntity` 或實作 `ITableEntity`。此案例中不使用 `partitionKey`、`rowKey`、`filter` 和 `take`屬性；您可以使用 `IQueryable` 物件來執行任何所需的篩選。

* 讀取 Node 函式中的多個資料列。

	請設定 `filter` 和 `take` 屬性。請勿設定 `partitionKey` 或 `rowKey`。

* 在 C# 函式中寫入一或多個資料列。

	Functions 執行階段會提供一個繫結至資料表的 `ICollector<T>` 或 `IAsyncCollector<T>`，其中 `T` 指定您想要新增之實體的結構描述。一般而言，類型 `T` 會衍生自 `TableEntity` 或實作 `ITableEntity`，但不一定如此。此案例中不使用 `partitionKey`、`rowKey`、`filter` 和 `take` 屬性。

#### 儲存體資料表範例︰讀取 C# 或 Node 中的單一資料表實體

下列 C# 程式碼範例可與前述稍早所示的「function.json」檔案搭配運作，以讀取單一資料表實體。佇列訊息具有資料列金鑰值，並將資料表實體讀入「run.csx」檔案中定義的類型。類型包括 `PartitionKey` 和 `RowKey` 屬性，且不是衍生自 `TableEntity`。

```csharp
public static void Run(string myQueueItem, Person personEntity, TraceWriter log)
{
    log.Info($"C# Queue trigger function processed: {myQueueItem}");
    log.Info($"Name in Person entity: {personEntity.Name}");
}

public class Person
{
    public string PartitionKey { get; set; }
    public string RowKey { get; set; }
    public string Name { get; set; }
}
```

下列 F# 程式碼範例也使用上述「function.json」檔案以讀取單一資料表實體。

```fsharp
[<CLIMutable>]
type Person = {
  PartitionKey: string
  RowKey: string
  Name: string
}

let Run(myQueueItem: string, personEntity: Person) =
    log.Info(sprintf "F# Queue trigger function processed: %s" myQueueItem)
    log.Info(sprintf "Name in Person entity: %s" personEntity.Name)
```

下列 Node 程式碼範例也使用上述「function.json」檔案以讀取單一資料表實體。

```javascript
module.exports = function (context, myQueueItem) {
    context.log('Node.js queue trigger function processed work item', myQueueItem);
    context.log('Person entity name: ' + context.bindings.personEntity.Name);
    context.done();
};
```

#### 儲存體資料表範例︰讀取 C 中的多個資料表實體# 

下列「function.json」和 C# 程式碼範例會讀取佇列訊息中指定的資料分割金鑰的實體。

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnection",
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "tableBinding",
      "type": "table",
      "connection": "MyStorageConnection",
      "tableName": "Person",
      "direction": "in"
    }
  ],
  "disabled": false
}
```

C# 程式碼會新增對「Azure 儲存體 SDK」的參考，讓實體類型可以衍生自 `TableEntity`。

```csharp
#r "Microsoft.WindowsAzure.Storage"
using Microsoft.WindowsAzure.Storage.Table;

public static void Run(string myQueueItem, IQueryable<Person> tableBinding, TraceWriter log)
{
    log.Info($"C# Queue trigger function processed: {myQueueItem}");
    foreach (Person person in tableBinding.Where(p => p.PartitionKey == myQueueItem).ToList())
    {
        log.Info($"Name: {person.Name}");
    }
}

public class Person : TableEntity
{
    public string Name { get; set; }
}
``` 

#### 儲存體資料表範例︰在 C 中建立資料表實體# 

下列「function.json」和「run.csx」範例示範如何在 C# 中撰寫資料表實體。

```json
{
  "bindings": [
    {
      "name": "input",
      "type": "manualTrigger",
      "direction": "in"
    },
    {
      "tableName": "Person",
      "connection": "MyStorageConnection",
      "name": "tableBinding",
      "type": "table",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

```csharp
public static void Run(string input, ICollector<Person> tableBinding, TraceWriter log)
{
    for (int i = 1; i < 10; i++)
        {
            log.Info($"Adding Person entity {i}");
            tableBinding.Add(
                new Person() { 
                    PartitionKey = "Test", 
                    RowKey = i.ToString(), 
                    Name = "Name" + i.ToString() }
                );
        }

}

public class Person
{
    public string PartitionKey { get; set; }
    public string RowKey { get; set; }
    public string Name { get; set; }
}

```

#### 儲存體資料表範例︰在 F 中建立資料表實體#

下列「function.json」和「run.fsx」範例示範如何在 F# 中撰寫資料表實體。

```json
{
  "bindings": [
    {
      "name": "input",
      "type": "manualTrigger",
      "direction": "in"
    },
    {
      "tableName": "Person",
      "connection": "MyStorageConnection",
      "name": "tableBinding",
      "type": "table",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

```fsharp
[<CLIMutable>]
type Person = {
  PartitionKey: string
  RowKey: string
  Name: string
}

let Run(input: string, tableBinding: ICollector<Person>, log: TraceWriter) =
    for i = 1 to 10 do
        log.Info(sprintf "Adding Person entity %d" i)
        tableBinding.Add(
            { PartitionKey = "Test"
              RowKey = i.ToString()
              Name = "Name" + i.ToString() })
```

#### 儲存體資料表範例︰在 Node 中建立資料表實體

下列「function.json」和「run.csx」範例示範如何在 Node 中撰寫資料表實體。

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnection",
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "tableName": "Person",
      "partitionKey": "Test",
      "rowKey": "{queueTrigger}",
      "connection": "MyStorageConnection",
      "name": "personEntity",
      "type": "table",
      "direction": "out"
    }
  ],
  "disabled": true
}
```

```javascript
module.exports = function (context, myQueueItem) {
    context.log('Node.js queue trigger function processed work item', myQueueItem);
    context.bindings.personEntity = {"Name": "Name" + myQueueItem }
    context.done();
};
```

## 後續步驟

[AZURE.INCLUDE [後續步驟](../../includes/functions-bindings-next-steps.md)]

<!---HONumber=AcomDC_0921_2016-->