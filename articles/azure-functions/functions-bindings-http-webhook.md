<properties
	pageTitle="Azure Functions HTTP 和 Webhook 繫結 | Microsoft Azure"
	description="了解如何在 Azure Functions 中使用 HTTP 和 Webhook 觸發程序與繫結。"
	services="functions"
	documentationCenter="na"
	authors="christopheranderson"
	manager="erikre"
	editor=""
	tags=""
	keywords="azure functions, 函式, 事件處理, webhook, 動態計算, 無伺服器架構"/>

<tags
	ms.service="functions"
	ms.devlang="multiple"
	ms.topic="reference"
	ms.tgt_pltfrm="multiple"
	ms.workload="na"
	ms.date="08/22/2016"
	ms.author="chrande"/>

# Azure Functions HTTP 和 Webhook 繫結

[AZURE.INCLUDE [functions-selector-bindings](../../includes/functions-selector-bindings.md)]

這篇文章說明如何在 Azure Functions 中為 HTTP 和 Webhook 觸發程序與繫結進行設定及撰寫程式碼。

[AZURE.INCLUDE [簡介](../../includes/functions-bindings-intro.md)]

## HTTP 和 Webhook 繫結的 function.json

「function.json」檔案會提供要求和回應的相關內容。

HTTP 要求的屬性︰

- `name`︰函式程式碼中用於要求物件 (或是在 Node.js 函式的情況下，則是要求本文) 的變數名稱。
- `type`︰必須設定為「httpTrigger」。
- `direction`：必須設為「in」。
- `webHookType`︰就 WebHook 觸發程序而言，有效值為「github」、「slack」及「genericJson」。對於非 WebHook 的 HTTP 觸發程序，會將這個屬性設為空字串。如需有關 Webhook 的詳細資訊，請參閱接下來的 [WebHook 觸發程序](#webhook-triggers)一節。
- `authLevel`︰不適用於 WebHook 觸發程序。設定為 [function] 要求 API 金鑰、[anonymous] 以拖放 API 金鑰需求，或 [admin] 要求主要 API 金鑰。如需詳細資訊，請參閱 [API 金鑰](#apikeys)。

HTTP 回應的屬性︰

- `name`︰函式程式碼中用於回應物件的變數名稱。
- `type`：必須設定為「http」。
- `direction`：必須設為「out」。
 
範例「function.json」：

```json
{
  "bindings": [
    {
      "webHookType": "",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "authLevel": "function"
    },
    {
      "name": "res",
      "type": "http",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

## WebHook 觸發程序

WebHook 觸發程序是一種 HTTP 觸發程序，具有下列專為 Webhook 而設計的功能︰

* 對於特定 WebHook 提供者 (目前支援 GitHub 和 Slack)，Functions 執行階段會驗證提供者的簽章。
* 對於 Node.js 函式，Functions 執行階段提供要求主體而非要求物件。C# 函式無需特殊的處理方式，因為您只要指定參數類型即可控制所提供的內容。如果指定 `HttpRequestMessage`，就會取得要求物件。如果指定 POCO 類型，Functions 執行階段會嘗試剖析要求主體中的 JSON 物件以填入物件屬性。
* 若要觸發 WebHook 函式，HTTP 要求必須包含 API 金鑰。對於非 WebHook HTTP 觸發程序，這項需求是選擇性的。

如需有關如何設定 GitHub WebHook 的資訊，請參閱 [GitHub 開發人員 - 建立 Webhook](http://go.microsoft.com/fwlink/?LinkID=761099&clcid=0x409)。

## 用來觸發函數的 URL

若要觸發函數，您需傳送 HTTP 要求給 URL，此要求是函數應用程式 URL 與函數名稱的組合：

```
 https://{function app name}.azurewebsites.net/api/{function name} 
```

## API 金鑰

根據預設，API 金鑰必須包含在 HTTP 要求之中，才能觸發 HTTP 或 WebHook 函式。金鑰可以包含在名為 `code` 的查詢字串變數中，或包含在 `x-functions-key` HTTP 標頭。對於非 WebHook 函式，您可以在 function.json 檔案中將 `authLevel` 屬性設為 [anonymous]，以將 API 金鑰指定為非必要。

您可以在函式應用程式的檔案系統中，於 D:\\home\\data\\Functions\\secrets 中找到 API 金鑰值。主要金鑰和函式金鑰均設定於 host.json 檔案之中，如此範例所示。

```json
{
  "masterKey": "K6P2VxK6P2VxK6P2VxmuefWzd4ljqeOOZWpgDdHW269P2hb7OSJbDg==",
  "functionKey": "OBmXvc2K6P2VxK6P2VxK6P2VxVvCdB89gChyHbzwTS/YYGWWndAbmA=="
}
```

host.json 的函式鍵可用來觸發任何函式，但不會觸發已停用的函式。主要金鑰可用來觸發任何函式，並且會觸發已停用的函式。您可以將 `authLevel` 屬性設為 [admin]，設定函式來要求主要金鑰。

如果 secrets 資料夾包含與函式同名的 JSON 檔案，也可使用該檔案中的 `key` 屬性來觸發函式，且此金鑰只會使用其參考的函式。例如，名為 `HttpTrigger` 之函式的 API 金鑰應在 secrets 資料夾的 HttpTrigger.json 中加以指定。下列是一個範例：

```json
{
  "key":"0t04nmo37hmoir2rwk16skyb9xsug32pdo75oce9r4kg9zfrn93wn4cx0sxo4af0kdcz69a4i"
}
```

> [AZURE.NOTE] 設定 WebHook 觸發程序時，請不要與 WebHook 提供者共用主要金鑰。使用只適用於處理 WebHook 的函式。主要金鑰可用來觸發任何函式，即使已停用的函式亦然。

## HTTP 觸發程序函式的 C# 程式碼範例 

程式碼範例會尋找 `name` 參數，其位於查詢字串或 HTTP 要求的主體。

```csharp
using System.Net;
using System.Threading.Tasks;

public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, TraceWriter log)
{
    log.Info($"C# HTTP trigger function processed a request. RequestUri={req.RequestUri}");

    // parse query parameter
    string name = req.GetQueryNameValuePairs()
        .FirstOrDefault(q => string.Compare(q.Key, "name", true) == 0)
        .Value;

    // Get request body
    dynamic data = await req.Content.ReadAsAsync<object>();

    // Set name to query string or body data
    name = name ?? data?.name;

    return name == null
        ? req.CreateResponse(HttpStatusCode.BadRequest, "Please pass a name on the query string or in the request body")
        : req.CreateResponse(HttpStatusCode.OK, "Hello " + name);
}
```

## HTTP 觸發程序函式的 F# 程式碼範例

程式碼範例會尋找 `name` 參數，其位於查詢字串或 HTTP 要求的主體。

```fsharp
open System.Net
open System.Net.Http
open FSharp.Interop.Dynamic

let Run(req: HttpRequestMessage) =
    async {
        let q =
            req.GetQueryNameValuePairs()
                |> Seq.tryFind (fun kv -> kv.Key = "name")
        match q with
        | Some kv ->
            return req.CreateResponse(HttpStatusCode.OK, "Hello " + kv.Value)
        | None ->
            let! data = Async.AwaitTask(req.Content.ReadAsAsync<obj>())
            try
                return req.CreateResponse(HttpStatusCode.OK, "Hello " + data?name)
            with e ->
                return req.CreateErrorResponse(HttpStatusCode.BadRequest, "Please pass a name on the query string or in the request body")
    } |> Async.StartAsTask
```

您會需要 `project.json` 檔案，以使用 NuGet 來參考 `FSharp.Interop.Dynamic` 和 `Dynamitey` 組件，例如︰

```json
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
```

這會使用 NuGet 來擷取相依性，並會在指令碼中加以參考。

## HTTP 觸發程序函式的 Node.js 程式碼範例 

此程式碼範例會尋找 `name` 參數，其位於查詢字串或 HTTP 要求的主體。

```javascript
module.exports = function(context, req) {
    context.log('Node.js HTTP trigger function processed a request. RequestUri=%s', req.originalUrl);

    if (req.query.name || (req.body && req.body.name)) {
        context.res = {
            // status: 200, /* Defaults to 200 */
            body: "Hello " + (req.query.name || req.body.name)
        };
    }
    else {
        context.res = {
            status: 400,
            body: "Please pass a name on the query string or in the request body"
        };
    }
    context.done();
};
```

## GitHub WebHook 函式的 C# 程式碼範例 

此程式碼範例會記錄 GitHub 問題註解。

```csharp
#r "Newtonsoft.Json"

using System;
using System.Net;
using System.Threading.Tasks;
using Newtonsoft.Json;

public static async Task<object> Run(HttpRequestMessage req, TraceWriter log)
{
    string jsonContent = await req.Content.ReadAsStringAsync();
    dynamic data = JsonConvert.DeserializeObject(jsonContent);

    log.Info($"WebHook was triggered! Comment: {data.comment.body}");

    return req.CreateResponse(HttpStatusCode.OK, new {
        body = $"New GitHub comment: {data.comment.body}"
    });
}
```

## GitHub WebHook 函式的 F# 程式碼範例

此程式碼範例會記錄 GitHub 問題註解。

```fsharp
open System.Net
open System.Net.Http
open FSharp.Interop.Dynamic
open Newtonsoft.Json

type Response = {
    body: string
}

let Run(req: HttpRequestMessage, log: TraceWriter) =
    async {
        let! content = req.Content.ReadAsStringAsync() |> Async.AwaitTask
        let data = content |> JsonConvert.DeserializeObject
        log.Info(sprintf "GitHub WebHook triggered! %s" data?comment?body)
        return req.CreateResponse(
            HttpStatusCode.OK,
            { body = sprintf "New GitHub comment: %s" data?comment?body })
    } |> Async.StartAsTask
```

## GitHub WebHook 函式的 Node.js 程式碼範例 

此程式碼範例會記錄 GitHub 問題註解。

```javascript
module.exports = function (context, data) {
    context.log('GitHub WebHook triggered!', data.comment.body);
    context.res = { body: 'New GitHub comment: ' + data.comment.body };
    context.done();
};
```

## 後續步驟

[AZURE.INCLUDE [後續步驟](../../includes/functions-bindings-next-steps.md)]

<!---HONumber=AcomDC_0921_2016-->