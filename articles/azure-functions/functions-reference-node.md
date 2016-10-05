<properties
	pageTitle="Azure Functions NodeJS 開發人員參考 | Microsoft Azure"
	description="了解如何使用 NodeJS 開發 Azure Functions。"
	services="functions"
	documentationCenter="na"
	authors="christopheranderson"
	manager="erikre"
	editor=""
	tags=""
	keywords="azure functions, 函式, 事件處理, webhook, 動態計算, 無伺服器架構"/>

<tags
	ms.service="functions"
	ms.devlang="nodejs"
	ms.topic="reference"
	ms.tgt_pltfrm="multiple"
	ms.workload="na"
	ms.date="05/13/2016"
	ms.author="chrande"/>

# Azure Functions NodeJS 開發人員參考

> [AZURE.SELECTOR]
- [C# 指令碼](../articles/azure-functions/functions-reference-csharp.md)
- [F# 指令碼](../articles/azure-functions/functions-reference-fsharp.md)
- [Node.js](../articles/azure-functions/functions-reference-node.md)

Azure Functions 的 Node/JavaScript 體驗能讓您輕鬆地匯出函式，系統會傳遞一個 `context` 物件給該函式以便與執行階段通訊，以及用來透過繫結接收和傳送資料。

本文假設您已經讀過 [Azure Functions 開發人員參考](functions-reference.md)。

## 匯出函數

所有 JavaScript 函式都必須透過 `module.exports` 匯出單一 `function`，如此執行階段才能找到函式並執行它。此函式一定要包含 `context` 物件。

```javascript
// You must include a context, but other arguments are optional
module.exports = function(context) {
    // Additional inputs can be accessed by the arguments property
    if(arguments.length === 4) {
        context.log('This function has 4 inputs');
    }
};
// or you can include additional inputs in your arguments
module.exports = function(context, myTrigger, myInput, myOtherInput) {
    // function logic goes here :)
};
```

`direction === "in"` 的繫結會和函式引數一起傳遞，這表示您可以使用 [`arguments`](https://msdn.microsoft.com/library/87dw3w1k.aspx) 以動態方式處理新的輸入 (例如，藉由使用 `arguments.length` 來反覆查看您的所有輸入)。如果您只有不含任何其他輸入的觸發程序，這項功能就很便利，因為您可以如預期般存取觸發程序資料，而不需要參考 `context` 物件。

引數一律會以在 function.json 中出現的順序傳遞至函式，即使您未在匯出陳述式中指定也一樣。例如，如果您有 `function(context, a, b)` 並將它變更為 `function(context, a)`，您仍然可以在函式程式碼中藉由參考 `arguments[3]` 來取得 `b` 的值。

所有繫結 (不論方向為何) 也都會傳遞到 `context` 物件 (如下所示)。

## context 物件

執行階段使用 `context` 物件來將資料傳遞至函式並從中傳出，而且可讓您與執行階段進行通訊。

內容物件一律為函式的第一個參數且應一律包含，因為它具有像是 `context.done` 和 `context.log` 的方法，而您必須要有這些方法才能正確地使用執行階段。您可以任意方式命名物件 (也就是 `ctx` 或 `c`)。

```javascript
// You must include a context, but other arguments are optional
module.exports = function(context) {
    // function logic goes here :)
};
```

## context.bindings

`context.bindings` 物件會收集所有輸入和輸出資料。資料會透過繫結的 `name` 屬性加入至 `context.bindings` 物件。例如，假設 function.json 中有下列繫結定義，您可以透過 `context.bindings.myInput` 存取佇列的內容。

```json
    {
        "type":"queue",
        "direction":"in",
        "name":"myInput"
        ...
    }
```

```javascript
// myInput contains the input data which may have properties such as "name"
var author = context.bindings.myInput.name;
// Similarly, you can set your output data
context.bindings.myOutput = { 
        some_text: 'hello world', 
        a_number: 1 };
```

## `context.done([err],[propertyBag])`

`context.done` 函式會通知執行階段您已完成執行。當您完成使用函數時呼叫它是很重要的。如果沒有這麼做，執行階段仍然永遠不會知道您的函數已完成。

`context.done` 函式可讓您將使用者定義的錯誤傳回執行階段，以及傳回屬性的屬性包，這些屬性將會覆寫 `context.bindings` 物件上的屬性。

```javascript
// Even though we set myOutput to have:
//  -> text: hello world, number: 123
context.bindings.myOutput = { text: 'hello world', number: 123 };
// If we pass an object to the done function...
context.done(null, { myOutput: { text: 'hello there, world', noNumber: true }});
// the done method will overwrite the myOutput binding to be: 
//  -> text: hello there, world, noNumber: true
```

## context.log(message)

`context.log` 方法可讓您輸出相互關聯在一起的記錄檔陳述式，以進行記錄。如果您使用 `console.log`，您的訊息將只會針對處理序層級記錄而顯示，但這樣並沒有什麼用處。

```javascript
/* You can use context.log to log output specific to this 
function. You can access your bindings via context.bindings */
context.log({hello: 'world'}); // logs: { 'hello': 'world' } 
```

`context.log` 方法與 Node [util.format 方法](https://nodejs.org/api/util.html#util_util_format_format)支援相同的參數格式。因此，比方說，像這樣的程式碼︰

```javascript
context.log('Node.js HTTP trigger function processed a request. RequestUri=' + req.originalUrl);
context.log('Request Headers = ' + JSON.stringify(req.headers));
```

可以撰寫成這樣︰

```javascript
context.log('Node.js HTTP trigger function processed a request. RequestUri=%s', req.originalUrl);
context.log('Request Headers = ', JSON.stringify(req.headers));
```

## HTTP 觸發程序︰context.req 和 context.res

在 HTTP 觸發程序的情況下，因為針對 HTTP 要求和回應物件使用 `req` 和 `res` 的模式相當常見，我們決定讓您能夠輕易地在內容物件中存取它們，而不會強迫您使用完整的 `context.bindings.name` 模式。

```javascript
// You can access your http request off of the context ...
if(context.req.body.emoji === ':pizza:') context.log('Yay!');
// and also set your http response
context.res = { status: 202, body: 'You successfully ordered more coffee!' };   
```

## Node 版本和封裝管理

Node 版本目前鎖定在 `5.9.1`。我們正在調查加入更多版本並允許設定的支援。

您可以在您的函式中加入封裝，做法是將 project.json 檔案上傳至函式應用程式檔案系統中的函式資料夾。如需檔案上傳指示，請參閱 [Azure Functions 開發人員參考主題](functions-reference.md#fileupdate)中的**如何更新函式應用程式檔案**一節。

您也可以在函式應用程式的 SCM (Kudu) 命令列介面中使用 `npm install`：

1. 瀏覽至 `https://<function_app_name>.scm.azurewebsites.net`。

2. 按一下 [偵錯主控台] > [CMD]。

3. 瀏覽至 `D:\home\site\wwwroot<function_name>`。

4. 執行 `npm install`。

一旦安裝您需要的封裝之後，就能以一般方式將它們匯入您的函式 (也就是透過 `require('packagename')`)

```javascript
// Import the underscore.js library
var _ = require('underscore');
var version = process.version; // version === 'v5.9.1'

module.exports = function(context) {
    // Using our imported underscore.js library
    var matched_names = _
        .where(context.bindings.myInput.names, {first: 'Carla'});
```

## 環境變數

若要取得環境變數或應用程式設定值，請使用 `process.env`，如下列程式碼範例所示：

```javascript
module.exports = function (context, myTimer) {
    var timeStamp = new Date().toISOString();
    
    context.log('Node.js timer trigger function ran!', timeStamp);   
    context.log(GetEnvironmentVariable("AzureWebJobsStorage"));
    context.log(GetEnvironmentVariable("WEBSITE_SITE_NAME"));
    
    context.done();
};

function GetEnvironmentVariable(name)
{
    return name + ": " + process.env[name];
}
```

## TypeScript/CoffeeScript 支援

目前仍沒有對於透過執行階段自動編譯 TypeScript/CoffeeScript 的任何支援，因此全都需要在部署時期的執行階段之外處理。

## 後續步驟

如需詳細資訊，請參閱下列資源：

* [Azure Functions 開發人員參考](functions-reference.md)
* [Azure Functions C# 開發人員參考](functions-reference-csharp.md)
* [Azure Functions F# 開發人員參考](functions-reference-fsharp.md)
* [Azure Functions 觸發程序和繫結](functions-triggers-bindings.md)

<!---HONumber=AcomDC_0921_2016-->