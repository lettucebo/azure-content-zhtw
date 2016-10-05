<properties
	pageTitle="在 Azure App Service 中開始使用 Node.js Web 應用程式"
	description="了解如何將 Node.js 應用程式部署到 Azure App Service 中的 Web 應用程式。"
	services="app-service\web"
	documentationCenter="nodejs"
	authors="cephalin"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="app-service-web"
	ms.workload="web"
	ms.tgt_pltfrm="na"
	ms.devlang="nodejs"
	ms.topic="get-started-article"
	ms.date="07/01/2016"
	ms.author="cephalin"/>

# 在 Azure App Service 中開始使用 Node.js Web 應用程式

[AZURE.INCLUDE [索引標籤](../../includes/app-service-web-get-started-nav-tabs.md)]

本教學課程示範如何建立簡單的 [Node.js] 應用程式，並透過 cmd.exe 或 bash 之類的命令列環境將其部署至 [Azure App Service]。本教學課程中的指示可運用在任何能夠執行 Node.js 應用程式的作業系統上。

<a name="prereq"></a>
## 必要條件

- [Node.js]
- [Bower]
- [Yeoman]
- [Git]
- [Azure CLI]
- Microsoft Azure 帳戶。如果您沒有這類帳戶，可以[申請免費試用]，或是[啟用自己的 Visual Studio 訂閱者權益]。

## 建立和部署簡單的 Node.js Web 應用程式

1. 開啟您選擇的命令列終端機，並安裝[適用於 Yeoman 的 Express 產生器]。

        npm install -g generator-express

2. `CD` 至工作目錄，並使用下列語法產生快速應用程式︰

        yo express
        
    在系統提示時選擇下列選項︰

    `? Would you like to create a new directory for your project?` **Yes** `? Enter directory name` **{appname}** `? Select a version to install:` **MVC** `? Select a view engine to use:` **Jade** `? Select a css preprocessor to use (Sass Requires Ruby):` **None** `? Select a database to use:` **None** `? Select a build tool to use:` **Grunt**

3. `CD` 至新應用程式的根目錄，並加以啟動以確保它會在您的開發環境中執行︰

        npm start

    在瀏覽器中瀏覽至 <http://localhost:3000> 以確定您可以看到 Express 首頁。一旦您確認應用程式正確執行，請使用 `Ctrl-C` 來停止它。
    
1. 變更為 ASM 模式並登入 Azure (您需要 [Azure CLI](#prereq))：

        azure config mode asm
        azure login

    依照提示，在瀏覽器中繼續使用具有 Azure 訂用帳戶的 Microsoft 帳戶進行登入。

2. 確定您仍在應用程式的根目錄中，然後使用下一個命令，以唯一的應用程式名稱在 Azure 中建立 App Service 應用程式資源。例如：http://{appname}.azurewebsites.net

        azure site create --git {appname}

    依照提示來選取要部署的目標 Azure 區域。如果您從未針對 Azure 訂用帳戶設定 Git/FTP 部署認證，則系統也會提示您加以建立。

3. 從應用程式根目錄開啟 ./config/config.js 檔案，並將生產連接埠變更為 `process.env.port`；`config` 物件中的 `production` 屬性看起來應該會像下列範例：

        production: {
            root: rootPath,
            app: {
                name: 'express1'
            },
            port: process.env.port,
        }

    這可讓您的 Node.js 應用程式在 iisnode 所接聽的預設連接埠上回應 Web 要求。
    
4. 開啟 ./package.json，並新增 `engines` 屬性以[指定所需的 Node.js 版本](#version)。

        "engines": {
            "node": "6.6.0"
        }, 

4. 儲存變更，然後使用 git 將您的應用程式部署至 Azure：

        git add .
        git commit -m "{your commit message}"
        git push azure master

    Express 產生器已提供 .gitignore 檔案，因此您的 `git push` 不會取用頻寬來嘗試上傳 node\_modules/ 目錄。

5. 最後，在瀏覽器中啟動即時 Azure 應用程式︰

        azure site browse

    您現在應該會看到 Node.js Web 應用程式在 Azure App Service 中即時執行。
    
    ![瀏覽至已部署的應用程式的範例。][deployed-express-app]

## 更新您的 Node.js Web 應用程式

若要更新在 App Service 中執行的 Node.js Web 應用程式，只需和您最初部署 Web 應用程式時一樣執行 `git add`、`git commit` 和 `git push`。
     
## App Service 如何部署您的 Node.js 應用程式

Azure App Service 使用 [iisnode] 來執行 Node.js 應用程式。Azure CLI 和 Kudu 引擎 (Git 部署) 會一同合作，讓您在透過命令列開發和部署 Node.js 應用程式時獲得流暢的體驗。

- `azure site create --git` 會辨識 server.js 或 app.js 的常見 Node.js 模式，並在根目錄中建立 iisnode.yml。您可以使用這個檔案來自訂 iisnode。
- 在 `git push azure master` 中，Kudu 會自動執行下列部署工作︰

    - 如果 package.json 位於儲存機制根目錄中，請執行 `npm install --production`。
    - 為 iisnode 產生指向 package.json 中的啟動指令碼的 Web.config (例如 server.js 或 app.js)。
    - 自訂 Web.config 以讓應用程式準備好使用節點偵測器進行偵錯。
    
## 使用 Node.js 架構

如果您使用熱門的 Node.js 架構 (例如 [Sails.js][SAILSJS] 或 [MEAN.js][MEANJS]) 來開發應用程式，您可以將這些應用程式部署到 App Service。熱門的 Node.js 架構有其特定的行為模式，而且其封裝相依性會不斷更新。不過，App Service 可提供 stdout 和 stderr 記錄給您，讓您確實了解應用程式發生了什麼事並據以變更。如需詳細資訊，請參閱[從 iisnode 取得 stdout 和 stderr 記錄](#iisnodelog)。

下列教學課程將為您示範如何在 App Service 中使用特定架構：

- [將 Sails.js Web 應用程式部署至 Azure App Service]
- [在 Azure App Service 中使用 Socket.IO 建立 Node.js 聊天應用程式]
- [如何搭配使用 io.js 和 Azure App Service Web Apps]

<a name="version"></a>
## 使用特定的 Node.js 引擎

和平常在 package.json 中的方式一樣，您可以在一般工作流程中告知 App Service 使用特定的 Node.js 引擎。例如：

    "engines": {
        "node": "6.6.0"
    }, 

Kudu 部署引擎會依下列順序決定要使用哪個 Node.js 引擎︰

- 首先，查看 iisnode.yml 以確認是否已指定 `nodeProcessCommandLine`。如果是，則使用它。
- 接下來，查看 package.json 以確認是否已在 `engines` 物件中指定 `"node": "..."`。如果是，則使用它。
- 依預設選擇預設的 Node.js 版本。

>[AZURE.NOTE] 建議您明確定義想要的 Node.js 引擎。預設的 Node.js 版本可以變更，但因為預設的 Node.js 版本不適用於您的 Azure Web 應用程式，您可能會在應用程式中收到錯誤。

<a name="iisnodelog"></a>
## 從 iisnode 取得 stdout 和 stderr 記錄

若要讀取 iisnode 記錄，請遵循下列步驟。

> [AZURE.NOTE] 在完成這些步驟之後，可能要等到發生錯誤時才會有記錄檔。

1. 開啟 Azure CLI 提供的 iisnode.yml 檔案。

2. 設定下列兩個參數︰

        loggingEnabled: true
        logDirectory: iisnode
    
    它們會一起告訴 App Service 中的 iisnode 將其 stdout 和 stderror 輸出放在 D:\\home\\site\\wwwroot*iisnode* 目錄中。

3. 儲存變更，然後使用下列 Git 命令將變更推送至 Azure：

        git add .
        git commit -m "{your commit message}"
        git push azure master
   
    iisnode 現在已設定好。接下來的步驟會示範如何存取這些記錄。
     
4. 在瀏覽器中存取應用程式的 Kudu 偵錯主控台，其位於︰

        https://{appname}.scm.azurewebsites.net/DebugConsole 

    此 URL 不同於 Web 應用程式 URL，因為前者在 DNS 名稱中加入了「.scm.」。如果您未在 URL 中加入此項目，就會收到 404 錯誤。

5. 瀏覽至 D:\\home\\site\\wwwroot\\iisnode

    ![瀏覽至 iisnode 記錄檔的位置。][iislog-kudu-console-find]

6. 按一下您想讀取之記錄的 [編輯] 圖示。如果您想要的話，也可以按一下 [下載] 或 [刪除]。

    ![開啟 iisnode 記錄檔。][iislog-kudu-console-open]

    現在您可以查看記錄以協助您偵錯 App Service 部署。
    
    ![檢查 iisnode 記錄檔。][iislog-kudu-console-read]

## 使用節點偵測器偵錯應用程式

如果您使用節點偵測器來偵錯 Node.js 應用程式，您可以將它用於您的即時 App Service 應用程式。節點偵測器會預先安裝在 App Service 的 iisnode 安裝中。如果您透過 Git 部署，則從 Kudu 自動產生的 Web.config 已包含啟用節點偵測器所需的所有組態。

若要啟用節點偵測器，請遵循下列步驟︰

1. 開啟位於儲存機制根目錄中的 iisnode.yml，並指定下列參數︰

        debuggingEnabled: true
        debuggerExtensionDll: iisnode-inspector.dll

3. 儲存變更，然後使用下列 Git 命令將變更推送至 Azure：

        git add .
        git commit -m "{your commit message}"
        git push azure master
   
4. 現在，只需要在 URL 中加上 /debug 以瀏覽至 package.json 中的啟動指令碼所指定的應用程式啟動檔案。例如，

        http://{appname}.azurewebsites.net/server.js/debug
    
    或者，
    
        http://{appname}.azurewebsites.net/app.js/debug

## 其他資源

- [在 Azure 應用程式中指定 Node.js 版本](../nodejs-specify-node-version-azure-apps.md)
- [Azure 上 Node.js 應用程式的最佳作法和疑難排解指南](app-service-web-nodejs-best-practices-and-troubleshoot-guide.md)
- [如何在 Azure App Service 中偵錯 Node.js Web 應用程式](web-sites-nodejs-debug.md)
- [使用 Node.js 模組與 Azure 應用程式搭配](../nodejs-use-node-modules-azure-apps.md)
- [Azure App Service Web Apps：Node.js](http://blogs.msdn.com/b/silverlining/archive/2012/06/14/windows-azure-websites-node-js.aspx)
- [Node.js 開發人員中心](/develop/nodejs/)
- [在 Azure App Service 中開始使用 Web 應用程式](app-service-web-get-started.md)
- [探索神秘無比的 Kudu 偵錯主控台]

<!-- URL List -->

[Azure CLI]: ../xplat-cli-install.md
[Azure App Service]: ../app-service/app-service-value-prop-what-is.md
[啟用自己的 Visual Studio 訂閱者權益]: http://go.microsoft.com/fwlink/?LinkId=623901
[Bower]: http://bower.io/
[在 Azure App Service 中使用 Socket.IO 建立 Node.js 聊天應用程式]: ./web-sites-nodejs-chat-app-socketio.md
[將 Sails.js Web 應用程式部署至 Azure App Service]: ./app-service-web-nodejs-sails.md
[探索神秘無比的 Kudu 偵錯主控台]: /documentation/videos/super-secret-kudu-debug-console-for-azure-web-sites/
[適用於 Yeoman 的 Express 產生器]: https://github.com/petecoop/generator-express
[Git]: http://www.git-scm.com/downloads
[如何搭配使用 io.js 和 Azure App Service Web Apps]: ./web-sites-nodejs-iojs.md
[iisnode]: https://github.com/tjanczuk/iisnode/wiki
[MEANJS]: http://meanjs.org/
[Node.js]: http://nodejs.org
[SAILSJS]: http://sailsjs.org/
[申請免費試用]: http://go.microsoft.com/fwlink/?LinkId=623901
[web app]: ./app-service-web-overview.md
[Yeoman]: http://yeoman.io/

<!-- IMG List -->

[deployed-express-app]: ./media/app-service-web-nodejs-get-started/deployed-express-app.png
[iislog-kudu-console-find]: ./media/app-service-web-nodejs-get-started/iislog-kudu-console-navigate.png
[iislog-kudu-console-open]: ./media/app-service-web-nodejs-get-started/iislog-kudu-console-open.png
[iislog-kudu-console-read]: ./media/app-service-web-nodejs-get-started/iislog-kudu-console-read.png

<!---HONumber=AcomDC_0928_2016-->