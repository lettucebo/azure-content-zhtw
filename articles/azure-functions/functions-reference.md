<properties
	pageTitle="Azure Functions 開發人員參考 | Microsoft Azure"
	description="了解通用於所有語言和繫結的 Azure Functions 概念和元件。"
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
	ms.date="05/13/2016"
	ms.author="chrande"/>

# Azure Functions 開發人員參考

不論您使用何種語言或繫結，Azure Functions 都共用一些核心技術概念和元件。閱讀指定語言或繫結特有的詳細資料之前，請務必詳閱這份適用於所有語言或繫結的概觀。

本文假設您已閱讀 [Azure Functions 概觀](functions-overview.md)，並熟悉 [WebJobs SDK 概念 (例如觸發程序、繫結和 JobHost 執行階段)](../app-service-web/websites-dotnet-webjobs-sdk.md)。Azure Functions 是以 WebJobs SDK 為基礎。


## 函式程式碼

「函數」 是 Azure Functions 中的主要概念。您使用選擇的語言撰寫函數的程式碼，並將程式碼檔案和組態檔儲存在相同的資料夾。組態為 JSON 格式，而檔案命名為 `function.json`。支援各種語言，而且每種語言的最適合最佳化經驗稍有不同。

`function.json` 檔案包含函式特有組態 (包括其繫結)。執行階段讀取這個檔案來判斷要關閉其觸發程序的事件、要在呼叫函數時包括的資料，以及在哪裏傳送從函數本身傳遞的資料。

```json
{
    "disabled":false,
    "bindings":[
        // ... bindings here
        {
            "type": "bindingType",
            "direction": "in",
            "name": "myParamName",
            // ... more depending on binding
        }
    ]
}
```

將 `disabled` 屬性設定為 `true`，即可防止執行階段執行函式。

`bindings` 屬性可讓您設定觸發程序和繫結。每個繫結都共用一些共用設定，以及特定類別的繫結特有的一些設定。每個繫結都需要下列設定︰

|屬性|值/類型|註解|
|---|-----|------|
|`type`|string|繫結類型。例如，`queueTrigger`。
|`direction`|'in'、'out'| 表示繫結用於將資料接收到函數，還是從函數傳送資料。
| `name` | string | 將作為函數中所繫結資料的名稱。在 C# 中，這將是引數名稱；在 JavaScript 中，這將是索引鍵/值清單中的索引鍵。

## 函式應用程式

函式應用程式是由一或多個由 Azure App Service 一起管理的個別函式所組成。函式應用程式中的所有函式會共用相同的定價方案、持續部署和執行階段版本。以多種語言撰寫的函式全都可以共用相同的函式應用程式。請將函式應用程式視為用來組織及集體管理函式的方式。

## 執行階段 (指令碼主機和 Web 主機)

執行階段 (或稱指令碼主機) 是基礎 WebJobs SDK 主機，可接聽事件、收集和傳送資料，最後執行您的程式碼。

為了簡化 HTTP 觸發程序，還可以使用 Web 主機，其設計是位在實際執行案例中指令碼主機的前面。這有助於隔離指令碼主機與 Web 主機所管理的前端流量。

## 資料夾結構

[AZURE.INCLUDE [functions-folder-structure](../../includes/functions-folder-structure.md)]

設定專案以將函式部署至 Azure App Service 中的函式應用程式時，您可以將此資料夾結構視為您的網站程式碼。您可以使用現有工具 (如持續整合和部署) 或自訂部署指令碼來執行部署時間封裝安裝或程式碼 Transpilation。

## <a id="fileupdate"></a> 如何更新函式應用程式檔案

Azure 入口網站內建的函式編輯器可讓您更新「function.json」檔案和函式的程式碼檔案。若要上傳或更新其他檔案，例如「package.json」或「project.json」或相依性，您必須使用其他部署方法。

函式應用程式是建置於 App Service 之上，因此[標準 Web 應用程式可用的部署選項](../app-service-web/web-sites-deploy.md)也可供函式應用程式使用。以下是一些您可以用來上傳或更新函式應用程式檔案的方法。

#### 使用 App Service 編輯器

1. 在 Azure Functions 入口網站中，按一下 [函式應用程式設定]。

2. 在 [進階設定] 區段中，按一下 [移至 App Service 設定]。

3. 在 [開發工具] 下方，按一下 [應用程式功能表導覽] 中的 [App Service 編輯器]。

4.  按一下 [執行]。

	App Service 編輯器載入之後，您將會在 wwwroot 下方看見 host.json 檔案和函式資料夾。

5. 開啟檔案加以編輯，或從您的開發電腦拖放檔案以上傳檔案。

#### 使用函式應用程式的 SCM (Kudu) 端點

1. 瀏覽至 `https://<function_app_name>.scm.azurewebsites.net`。

2. 按一下 [偵錯主控台] > [CMD]。

3. 瀏覽至 `D:\home\site\wwwroot` 更新 *host.json*，或瀏覽至 `D:\home\site\wwwroot<function_name>` 更新函式的檔案。

4. 將您要上傳的檔案拖放至檔案方格中適當資料夾。在檔案方格上您有兩個區域可以拖放檔案。對於「.zip」檔案，方塊顯示且具有標籤「拖曳到這裡以上傳和解壓縮」。 對於其他檔案類型，在檔案方格上但是在「解壓縮」方塊外拖放。

#### 使用 FTP

1. 請遵循[這裡](../app-service-web/web-sites-deploy.md#ftp)的指示設定 FTP。

2. 當您連線到函式應用程式網站時，請將更新的「host.json」檔案複製到 `/site/wwwroot`，或將函式檔案複製到 `/site/wwwroot/<function_name>`。

#### 使用持續部署

請依照 [Azure Functions 的持續部署](functions-continuous-deployment.md)主題中的指示進行。

## 平行執行

發生速度比單一執行緒函數執行階段快的多個觸發事件可以處理它們時，執行階段可能會平行多次叫用函數。如果函式應用程式使用 [動態服務方案](functions-scale.md#dynamic-service-plan)，函式應用程式可以自動相應放大。不論應用程式是在動態服務方案還是一般 [App Service 方案](../app-service/azure-web-sites-web-hosting-plans-in-depth-overview.md)上執行，函數應用程式的每個執行個體可能都會使用多個執行緒平行處理並行函數引動過程。每個函數應用程式執行個體中的並行函數引動過程數目上限，會根據函數應用程式的記憶體大小而不同。

## Azure Functions 脈衝  

脈衝是顯示您函數執行頻率以及成功和失敗的即時事件資料流。您也可以監視平均執行時間。我們將持續加入更多功能和自訂。您可以存取 [監視] 索引標籤的 [脈衝] 頁面。

## 儲存機制

Azure Functions 的程式碼是開放原始碼，儲存於 GitHub 儲存機制中︰

* [Azure Functions 執行階段](https://github.com/Azure/azure-webjobs-sdk-script/)
* [Azure Functions 入口網站](https://github.com/projectkudu/AzureFunctionsPortal)
* [Azure Functions 範本](https://github.com/Azure/azure-webjobs-sdk-templates/)
* [Azure WebJobs SDK](https://github.com/Azure/azure-webjobs-sdk/)
* [Azure WebJobs SDK 延伸模組](https://github.com/Azure/azure-webjobs-sdk-extensions/)

## 繫結

以下是所有已支援繫結的表格。

[AZURE.INCLUDE [動態計算](../../includes/functions-bindings.md)]

## 報告問題

[AZURE.INCLUDE [報告問題](../../includes/functions-reporting-issues.md)]

## 後續步驟

如需詳細資訊，請參閱下列資源：

* [Azure Functions C# 開發人員參考](functions-reference-csharp.md)
* [Azure Functions F# 開發人員參考](functions-reference-fsharp.md)
* [Azure Functions NodeJS 開發人員參考](functions-reference-node.md)
* [Azure Functions 觸發程序和繫結](functions-triggers-bindings.md)
* [Azure Functions︰Azure App Service 團隊部落格上的旅程](https://blogs.msdn.microsoft.com/appserviceteam/2016/04/27/azure-functions-the-journey/)。Azure Functions 的開發歷史。

<!---HONumber=AcomDC_0921_2016-->