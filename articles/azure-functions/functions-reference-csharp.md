<properties
	pageTitle="Azure Functions 開發人員參考 | Microsoft Azure"
	description="了解如何使用 C# 開發 Azure Functions。"
	services="functions"
	documentationCenter="na"
	authors="christopheranderson"
	manager="erikre"
	editor=""
	tags=""
	keywords="azure functions, 函式, 事件處理, webhook, 動態計算, 無伺服器架構"/>

<tags
	ms.service="functions"
	ms.devlang="dotnet"
	ms.topic="reference"
	ms.tgt_pltfrm="multiple"
	ms.workload="na"
	ms.date="05/13/2016"
	ms.author="chrande"/>

# Azure Functions C# 開發人員參考

> [AZURE.SELECTOR]
- [C# 指令碼](../articles/azure-functions/functions-reference-csharp.md)
- [F# 指令碼](../articles/azure-functions/functions-reference-fsharp.md)
- [Node.js](../articles/azure-functions/functions-reference-node.md)
 
Azure Functions 的 C# 體驗是以 Azure WebJobs SDK 為基礎。資料會透過方法引數流入您的 C# 函式。引數名稱會指定於 `function.json` 中，而且有預先定義的名稱可用來存取函式記錄器和取消權杖等項目。

本文假設您已經讀過 [Azure Functions 開發人員參考](functions-reference.md)。

## .csx 的運作方式

`.csx` 格式允許撰寫較少「重複使用」文字，只專注於撰寫 C# 函式。在 Azure Functions 中，您只要如往常般包含您所需的組件參考和命名空間，而不是在命名空間和類別中包裝所有項目，您可以只定義您的 `Run` 方法。如果您需要包含任何類別，例如若要定義 POCO 物件，您可以包含相同檔案內的類別。

## 繫結至引數

各種繫結會透過 function.json 組態中的 `name` 屬性繫結至 C# 函式。每個繫結都有自己支援的類型 (已依照繫結記載)；例如，Blob 觸發程序可以支援字串、POCO 或數個其他類型。您可以使用最符合您需求的類型。

```csharp
public static void Run(string myBlob, out MyClass myQueueItem)
{
    log.Verbose($"C# Blob trigger function processed: {myBlob}");
    myQueueItem = new MyClass() { Id = "myid" };
}

public class MyClass
{
    public string Id { get; set; }
}
```

## 記錄

若要在 C# 中將輸出記錄至串流記錄檔，您可以包含 `TraceWriter` 具類型引數。建議您將它命名為 `log`。建議您避免在 Azure Functions 中使用 `Console.Write`。

```csharp
public static void Run(string myBlob, TraceWriter log)
{
    log.Info($"C# Blob trigger function processed: {myBlob}");
}
```

## 非同步處理

若要讓函式變成非同步，請使用 `async` 關鍵字並傳回 `Task` 物件。

```csharp
public async static Task ProcessQueueMessageAsync(
        string blobName, 
        Stream blobInput,
        Stream blobOutput)
    {
        await blobInput.CopyToAsync(blobOutput, 4096, token);
    }
```

## 取消權杖

在某些情況下，您可能會有易受關閉影響的作業。雖然最好能撰寫可以處理當機的程式碼，但是當您想要處理順利關機要求時，您可以定義 [`CancellationToken`](https://msdn.microsoft.com/library/system.threading.cancellationtoken.aspx) 具類型引數。如果觸發主機關機，則會提供 `CancellationToken`。

```csharp
public async static Task ProcessQueueMessageAsyncCancellationToken(
        string blobName, 
        Stream blobInput,
        Stream blobOutput,
        CancellationToken token)
    {
        await blobInput.CopyToAsync(blobOutput, 4096, token);
    }
```

## 匯入命名空間

如果您需要匯入命名空間，可以如往常一樣利用 `using` 子句。

```csharp
using System.Net;
using System.Threading.Tasks;

public static Task<HttpResponseMessage> Run(HttpRequestMessage req, TraceWriter log)
```

下列命名空間會自動匯入，所以是選擇性的︰

* `System`
* `System.Collections.Generic`
* `System.IO`
* `System.Linq`
* `System.Net.Http`
* `System.Threading.Tasks`
* `Microsoft.Azure.WebJobs`
* `Microsoft.Azure.WebJobs.Host`。

## 參考外部組件

若為架構組件，請使用 `#r "AssemblyName"` 指示詞加入參考。

```csharp
#r "System.Web.Http"

using System.Net;
using System.Net.Http;
using System.Threading.Tasks;

public static Task<HttpResponseMessage> Run(HttpRequestMessage req, TraceWriter log)
```

Azure Functions 裝載環境會自動加入下列組件︰

* `mscorlib`,
* `System`
* `System.Core`
* `System.Xml`
* `System.Net.Http`
* `Microsoft.Azure.WebJobs`
* `Microsoft.Azure.WebJobs.Host`
* `Microsoft.Azure.WebJobs.Extensions`
* `System.Web.Http`
* `System.Net.Http.Formatting`。

此外，下列組件為特殊案例，可以使用簡單名稱來參考 (例如 `#r "AssemblyName"`)：

* `Newtonsoft.Json`
* `Microsoft.WindowsAzure.Storage`
* `Microsoft.ServiceBus`
* `Microsoft.AspNet.WebHooks.Receivers`
* `Microsoft.AspNEt.WebHooks.Common`。

如果您需要參考私用組件，可以將組件檔案上傳至相對於您函式的 `bin` 資料夾並使用檔案名稱 (例如 `#r "MyAssembly.dll"`) 來參考它。如需如何將檔案上傳至函數資料夾的資訊，請參閱以下的＜封裝管理＞小節。

## 封裝管理

若要在 C# 函式中使用 NuGet 封裝，請將 project.json 檔案上傳至函式應用程式檔案系統中的函式資料夾。以下是範例 project.json 檔案，該檔案會加入對 Microsoft.ProjectOxford.Face 1.1.0 版的參考：

```json
{
  "frameworks": {
    "net46":{
      "dependencies": {
        "Microsoft.ProjectOxford.Face": "1.1.0"
      }
    }
   }
}
```

只支援 .NET Framework 4.6，因此請確認您的 project.json 檔案會指定 `net46`，如下所示。

當您上傳 project.json 檔案時，執行階段會取得封裝並自動加入對封裝組件的參考。您不需要加入 `#r "AssemblyName"` 指示詞。只要將所需的 `using` 陳述式加入至 run.csx 檔案，即可使用 NuGet 封裝中定義的類型。


### 如何上傳 project.json 檔案

1. 首先，在 Azure 入口網站中開啟您的函式，以確定函式應用程式正在執行中。

	這也可供存取將要顯示封裝安裝輸出的串流記錄檔。

2. 若要上傳 project.json 檔案，請使用 [Azure Functions 開發人員參考主題](functions-reference.md#fileupdate)中**如何更新函式應用程式檔案**一節所述的其中一個方法。

3. 上傳 project.json 檔案之後，您會在函式的串流記錄檔中看到如下列範例所示的輸出：

```
2016-04-04T19:02:48.745 Restoring packages.
2016-04-04T19:02:48.745 Starting NuGet restore
2016-04-04T19:02:50.183 MSBuild auto-detection: using msbuild version '14.0' from 'D:\Program Files (x86)\MSBuild\14.0\bin'.
2016-04-04T19:02:50.261 Feeds used:
2016-04-04T19:02:50.261 C:\DWASFiles\Sites\facavalfunctest\LocalAppData\NuGet\Cache
2016-04-04T19:02:50.261 https://api.nuget.org/v3/index.json
2016-04-04T19:02:50.261 
2016-04-04T19:02:50.511 Restoring packages for D:\home\site\wwwroot\HttpTriggerCSharp1\Project.json...
2016-04-04T19:02:52.800 Installing Newtonsoft.Json 6.0.8.
2016-04-04T19:02:52.800 Installing Microsoft.ProjectOxford.Face 1.1.0.
2016-04-04T19:02:57.095 All packages are compatible with .NETFramework,Version=v4.6.
2016-04-04T19:02:57.189 
2016-04-04T19:02:57.189 
2016-04-04T19:02:57.455 Packages restored.
```

## 環境變數

若要取得環境變數或應用程式設定值，請使用 `System.Environment.GetEnvironmentVariable`，如下列程式碼範例所示：

```csharp
public static void Run(TimerInfo myTimer, TraceWriter log)
{
    log.Info($"C# Timer trigger function executed at: {DateTime.Now}");
    log.Info(GetEnvironmentVariable("AzureWebJobsStorage"));
    log.Info(GetEnvironmentVariable("WEBSITE_SITE_NAME"));
}

public static string GetEnvironmentVariable(string name)
{
    return name + ": " + 
        System.Environment.GetEnvironmentVariable(name, EnvironmentVariableTarget.Process);
}
```

## 重複使用 .csx 程式碼

您可以在您的 run.csx 檔案中使用其他 .csx 檔案中定義的類別和方法。若要這樣做，請在您的 run.csx 檔案中使用 `#load` 指示詞，如下列範例所示。

範例 run.csx：

```csharp
#load "mylogger.csx"

public static void Run(TimerInfo myTimer, TraceWriter log)
{
    log.Verbose($"Log by run.csx: {DateTime.Now}"); 
    MyLogger(log, $"Log by MyLogger: {DateTime.Now}");
}
```

範例 mylogger.csx：

```csharp
public static void MyLogger(TraceWriter log, string logtext)
{
    log.Verbose(logtext); 
}
```

您可以使用包含 `#load` 指示詞的相對路徑：

* `#load "mylogger.csx"` 會載入位於函式資料夾中的檔案。

* `#load "loadedfiles\mylogger.csx"` 會載入位於函式資料夾的資料夾中的檔案。

* `#load "..\shared\mylogger.csx"` 會載入位於與函式資料夾相同層級的資料夾中的檔案 (也就是在 wwwroot 的正下方)。
 
`#load` 指示詞只適用於 .csx (C# 指令碼) 檔案，不適用於 .cs 檔案。

## 後續步驟

如需詳細資訊，請參閱下列資源：

* [Azure Functions 開發人員參考](functions-reference.md)
* [Azure Functions NodeJS 開發人員參考](functions-reference-fsharp.md)
* [Azure Functions NodeJS 開發人員參考](functions-reference-node.md)
* [Azure Functions 觸發程序和繫結](functions-triggers-bindings.md)

<!---HONumber=AcomDC_0921_2016-->