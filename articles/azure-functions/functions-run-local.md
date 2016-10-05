<properties
	pageTitle="開發並在本機執行 Azure functions | Microsoft Azure"
	description="了解在 Azure App Service 中執行之前，如何編碼並測試 Visual Studio 中的 Azure Functions。"
	services="functions"
	documentationCenter="na"
	authors="tdykstra"
	manager="erikre"
	editor=""/>

<tags
	ms.service="functions"
	ms.workload="na"
	ms.tgt_pltfrm="multiple"
	ms.devlang="multiple"
	ms.topic="article"
	ms.date="08/22/2016"
	ms.author="glenga"/>

# 如何在 Visual Studio 中編碼和測試 Azure Functions

## Overview

這篇文章說明如何透過下載 [WebJobs.Script](https://github.com/Azure/azure-webjobs-sdk-script/) GitHub 儲存機制在本機執行 [Azure Functions](functions-overview.md)，並執行其所包含的 Visual Studio 解決方案。

Azure Functions 的執行階段為 WebJobs.Script 開放原始碼專案的實作。此專案會依次組建於 [Azure WebJobs SDK](../app-service-web/websites-dotnet-webjobs-sdk.md) 上，且這兩個架構都可以在本機執行。不過，由於 WebJobs SDK 會使用儲存體模擬器不支援的儲存體帳戶功能，因此您需要連線至 Azure 儲存體帳戶。

您可以輕鬆地在 Azure 入口網站中對函式進行編碼和測試，但有時候適合在 Azure 中執行之前先在本機處理這些函式。例如，Azure Functions 所支援的部份語言更容易在 Visual Studio 中撰寫程式碼，這是由於它提供 [IntelliSense](https://msdn.microsoft.com/library/hcw1s69b.aspx)。雖然您可以從遠端偵錯函式，但在本機偵錯可能更快速且更輕鬆。當您在本機執行時，可以進行偵錯，並在函式的程式碼及 WebJobs 指令碼主機程式碼中設定中斷點。

>[AZURE.NOTE] Azure Functions 目前僅供預覽，而包含工具的整體體驗仍在快速開發中。這篇文章中所描述的程序並未反映最後的本機開發經驗，我們希望您能夠[提供意見反應](https://feedback.azure.com/forums/355860-azure-functions)。

## 必要條件

### 具有函式應用程式的 Azure 帳戶

這篇文章假設您已在入口網站使用 [Azure Functions](functions-overview.md) 並已熟悉 Azure Functions 概念，例如[觸發程序、繫結和 JobHost](functions-reference.md)。

當您在本機執行函式時，會在主控台視窗獲得部分輸出，但您也會想要使用由即時函式應用程式所裝載的儀表板來檢視函式引動過程和記錄檔。

### Visual Studio 2015 (含最新的 Azure SDK for .NET)

若您沒有 Visual Studio 2015，或您沒有最新的 Azure SDK，[請下載 Azure SDK for Visual Studio 2015](http://go.microsoft.com/fwlink/?linkid=518003)。如果您沒有 Visual Studio 2015，它會自動與 SDK 一起安裝。

### 條件式的必要條件

只有當您計劃執行的函式會用到時，才會需要某些 Azure 資源和軟體安裝，例如︰

* Azure 資源
	* 服務匯流排
	* 簡單的資料表
	* DocumentDB
	* 事件中樞
	* 通知中樞

* 編譯器和指令碼引擎
	* F#
	* BASH
	* Python
	* PHP

如需關於這些需求，包括您需要對這些環境變數進行設定的詳細資訊，請參閱 [WebJobs.Script 儲存機制的 Wiki 頁面](https://github.com/Azure/azure-webjobs-sdk-script/wiki/home)

如果您的目的是要參與 WebJobs.SDK 專案，則需要所有條件式的必要條件，才能執行完整的測試。

## 若要在本機執行

1. [複製](https://github.com/Azure/azure-webjobs-sdk-script/)或[下載](https://github.com/Azure/azure-webjobs-sdk-script/archive/master.zip) Webjobs.Script 儲存機制。

2. 設定儲存體連接字串的環境變數。

	* AzureWebJobsStorage
	* AzureWebJobsDashboard

	您可以針對函式應用程式，從 App Service 應用程式設定入口網站刀鋒視窗中取得這些值。

	a.在「函式應用程式」刀鋒視窗中，按一下 [函式應用程式設定]。

	![按一下 [函式應用程式設定]](./media/functions-run-local/clickfuncappsettings.png)
 
	b.在「函式應用程式設定」刀鋒視窗中，按一下 [移至 App Service 設定]。

	![按一下 [App Service 設定]](./media/functions-run-local/clickappsvcsettings.png)
 
	c.在「設定」刀鋒視窗中，按一下 [應用程式設定]。

	![按一下 [應用程式設定]](./media/functions-run-local/clickappsettings.png)
 
	d.在「應用程式設定」刀鋒視窗中，向下捲動至 [應用程式設定] 區段，並尋找WebJobs SDK 設定。

	![WebJobs 設定](./media/functions-run-local/wjsettings.png)

	e.將環境變數設定為和 `AzureWebJobsStorage` 應用程式設定相同的名稱和值。

	f.對 `AzureWebJobsDashboard` 應用程式設定進行相同步驟。

2. 建立名為 AzureWebJobsServiceBus 的環境變數，並將它設定為服務匯流排連接字串。

	服務匯流排繫結需要這個環境變數，我們建議您即便不使用服務匯流排繫結，仍應進行設定。在某些情況下，無論是否使用繫結，如果未設定服務匯流排連接字串，您可能會看到例外狀況。

3. 請確定已設定您所需的其他任何環境變數。(請參閱先前的[條件式的必要條件](#conditional-prerequisites)一節)。

4. 啟動 Visual Studio，然後再開啟 WebJobs.Script 方案。

6. 設定啟動專案。如果您想要執行使用 HTTP 或 WebHook 觸發程序的函式，請選擇 [WebJobs.Script.WebHost]；否則，請選擇 [WebJobs.Script.Host]。

4. 如果您的啟動專案為 WebJobs.Script.Host：

	a.在「方案總管」中，以滑鼠右鍵按一下 [WebJobs.Script.Host] 方案，然後按一下 [屬性]。

	b.在「專案屬性」視窗的 [偵錯] 索引標籤中，將 [命令列引數] 設定為 `..\..\..\..\sample`。

	![命令列引數](./media/functions-run-local/cmdlineargs.png)

	這是儲存機制中「範例」資料夾的相對路徑。「範例」資料夾包含 host.json 檔案，其中包含通用設定，以及每個範例函式的資料夾。

	若要開始使用，使用所提供的「範例」資料夾是最容易的方式。稍後您可以將自己的函式加入「範例」 資料夾，或任何使用包含 host.json 和函式資料夾的資料夾。

5. 如果您的啟動專案為 WebJobs.Script.WebHost：

	a.將 AzureWebJobsScriptRoot 環境變數設定至 `sample` 資料夾的完整路徑 。

	b.重新啟動 Visual Studio 來挑選新的環境變數值。

	如需有關如何執行 HTTP 觸發程序函式的其他資訊，請參閱 [API 金鑰](#api-keys)一節。

5. 開啟 sample\\host.json 檔案，並新增 `functions` 屬性來指定您想要執行的函式。

	例如：下列的 JSON 會讓 WebJobs SDK JobHost 只尋找兩個函式。

		{
		  "functions": [ "TimerTrigger-CSharp", "QueueTrigger-CSharp"],
		  "id": "5a709861cab44e68bfed5d2c2fe7fc0c"
		}

	當您使用的是自己的資料夾，而不是「範例」資料夾，在其中只會包含您想要執行的函式。然後您可以省略 host.json 中的 `functions` 屬性。
 
6. 建置並執行解決方案。

	主控台視窗顯示 JobHost 只會尋找在 `host.json` 檔案中指定的函式。

		Found the following functions:
		Host.Functions.QueueTrigger-CSharp
		Host.Functions.TimerTrigger-CSharp
		Job host started

	如果您正在開始 WebHost 專案，您會得到一個空白瀏覽器頁面，因為在專案的基底 URL 並無內容可提供。如需有關用於 HTTP 觸發程序函數之 URL 的資訊，請參閱 [API 金鑰](#apikeys)一節。

## 檢視函式輸出

請移至函式應用程式的儀表板，查看函式引動過程，並為其記錄輸出。

儀表板位於下列 URL：

	https://{function app name}.scm.azurewebsites.net/azurejobs/#/functions

「函數」頁面會顯示已執行函數的清單，以及函數叫用清單。

![引動過程詳細資料](./media/functions-run-local/invocationdetail.png)

按一下某個叫用可查看「叫用詳細資料」頁面，當中會指出函數觸發時間、大約執行時間，以及成功完成狀態。按一下 [切換輸出] 按鈕，即可查看函數程式碼所寫入的記錄檔。

![引動過程詳細資料](./media/functions-run-local/invocationdetail.png)

## <a id="apikeys"></a>HTTP 觸發程序的 API 金鑰

若要執行 HTTP 或 WebHook 函數，您將需要 API 金鑰，除非您將 `"authLevel": "anonymous"` 納入 *function.json* 檔案中。

例如，如果 API 金鑰為 `12345`，當 WebJobs.Script.WebHost 專案正在執行時，您可以使用下列 URL 來觸發 *HttpTrigger* 函數。

	http://localhost:28549/api/httptrigger?code=12345

(或者，您也可以將 API 金鑰放在 `x-functions-key` HTTP 標頭中。)

API 金鑰儲存在 WebJobs.Script.WebHost 專案之 [App\_Data/secrets](https://github.com/Azure/azure-webjobs-sdk-script/tree/master/src/WebJobs.Script.WebHost/App_Data/secrets) 資料夾的 `.json` 檔案中。

### 適用於所有 HTTP 和 WebHook 函式的 API 金鑰

*App\_Data/secrets* 資料夾中的 *host.json* 檔案有兩組金鑰︰

```json
{
  "masterKey": "hyexydhln844f2mb7hyexydhln844f2mb7",
  "functionKey": "7hyexydhn844f2mb7hyexydhln844f2mb7"
}
```

`functionKey` 屬性儲存一組金鑰，若未針對該特定函數定義任何覆寫的情況下，該組金鑰可用於任何 HTTP 或 WebHook 函數。這項功能可讓您無需為您所建立的每個函式定義新 API 金鑰。

`masterKey` 屬性儲存一組金鑰，該組金鑰在某些測試案例中相當有用︰

* 如果您以主要金鑰呼叫 WebHook 函數，WebJobs SDK 就會略過 WebHook 提供者的簽章驗證。

* 如果您以主要金鑰呼叫 HTTP 或 WebHook 函數，則即使函數在 *function.json* 檔案中已停用，仍然會觸發該函數。這是在 Azure 入口網站中用來讓 [執行] 按鈕即使在函數已停用的情況下，仍然可以運作。
 
### 套用於個別函式的 API 金鑰

名稱為 *{function name}.json* 的檔案包含特定函數的 API 金鑰。例如，下列在 *App\_Data/secrets/HttpTrigger.json* 中的範例 JSON 內容會設定 `HttpTrigger` 函數的 API 金鑰。

```json
{
  "key": "844f2mdhn844f2mb7hyexydhln844f2mb7"
}
```

## 在函數中使用 NuGet 套件參考  

基於目前處理 NuGet 參考的方式，請確定您在主機處於執行狀態時，會「存取」*project.json* 檔案。主機會監看是否發生檔案修改，並在偵測到變更時，起始還原作業。此外，*NuGet.exe* (建議使用 3.3.0) 必須位於您的路徑中，或是您必須設定一個名為 AzureWebJobs\_NuGetPath 且含有 *NuGet.exe* 路徑的環境變數。

## 疑難排解

在執行 Visual Studio 時，不會自動挑選已完成變更的環境變數。如果您在啟動 Visual Studio 後新增或變更環境變數，請關閉 Visual Studio，然後重新啟動 Visual Studio，以確保其可挑出目前的值。

偵錯時，您可以透過在 [例外狀況設定] 視窗中 (按 CTRL-ALT-E 來開啟視窗) 選取 [通用語言執行平台例外狀況]，取得例外狀況的更多相關資訊。

還有另一種您可以在偵錯時取得更多例外狀況資訊的方法，就是在指令碼裝載的主迴圈 `catch` 區塊中設定中斷點。您可以在 WebJobs.Script 專案中、*Host/ScriptHostManager.cs* 中以及 `RunAndBlock` 方法中找到此區塊。

## 後續步驟

如需詳細資訊，請參閱下列資源：

* [Azure Functions 開發人員參考](functions-reference.md)
* [Azure Functions C# 開發人員參考](functions-reference-csharp.md)
* [Azure Functions F# 開發人員參考](functions-reference-fsharp.md)
* [Azure Functions NodeJS 開發人員參考](functions-reference-node.md)
* [Azure Functions 觸發程序和繫結](functions-triggers-bindings.md)

<!---HONumber=AcomDC_0921_2016-->