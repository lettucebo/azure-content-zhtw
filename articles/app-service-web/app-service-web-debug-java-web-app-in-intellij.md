<properties 
	pageTitle="在 IntelliJ 中對 Azure 上的 Java Web 應用程式進行偵錯 | Microsoft Azure" 
	description="本教學課程示範如何使用 Azure Toolkit for IntelliJ，對 Azure 上執行的 Java Web 應用程式進行偵錯。" 
	services="app-service\web" 
	documentationCenter="java" 
	authors="selvasingh" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="app-service-web" 
	ms.workload="web" 
	ms.tgt_pltfrm="na" 
	ms.devlang="Java" 
	ms.topic="article" 
	ms.date="09/20/2016" 
	ms.author="asirveda;robmcm"/>

# 在 IntelliJ 中對 Azure 上的 Java Web 應用程式進行偵錯

本教學課程示範如何使用[適用於 IntelliJ 的 Azure 工具組]，對 Azure 上執行的 Java Web 應用程式進行偵錯。為了簡單起見，您將針對本教學課程使用基本的 Java Server Page (JSP) 範例，但在 Azure 上進行偵錯時，步驟與 Java Servlet 類似。

當您完成本教學課程之後，在 IntelliJ 中對應用程式進行偵錯時，其看起來類似下圖：

![][01]
 
## 必要條件

* Java Developer Kit (JDK) 1.8 版或更新版本。
* IntelliJ 概念旗艦版。這可透過 <https://www.jetbrains.com/idea/download/index.html> 下載。
* Java 型 Web 伺服器或應用程式伺服器的散發套件，例如 Apache Tomcat 或 Jetty。
* Azure 訂用帳戶，可從 <https://azure.microsoft.com/free/> 或 <http://azure.microsoft.com/pricing/purchase-options/> 取得。
* 適用於 IntelliJ 的 Azure 工具組。如需詳細資訊，請參閱 [Installing the Azure Toolkit for IntelliJ (安裝適用於 IntelliJ 的 Azure 工具組)]。
* 動態 Web 專案已建立並部署到 Azure App Service；例如，請參閱[在 IntelliJ 中建立 Azure 的 Hello World Web 應用程式]。

## 對 Azure 上的 Java Web 應用程式進行偵錯

若要完成本節中的這些步驟，您可以使用現有的動態 Web 專案 (您已經將其部署為 Azure 上的 Java Web 應用程式)、下載[動態 Web 專案範例]，並依照[在 IntelliJ 中建立 Azure 的 Hello World Web 應用程式]的步驟，將其部署於 Azure 上。

1. 在 IntelliJ 中，開啟您已部署到 Azure 的 Java Web 應用程式專案。

1. 按一下 [執行] 功能表，然後按一下 [編輯組態...]。

    ![][02]

1. 當 [執行/偵錯組態] 對話方塊開啟時：

    1. 選取 [Azure Web 應用程式]。
    1. 按一下 [+] 以加入新的組態。
    1. 提供組態的**名稱**。
    1. 接受 Azure 工具組所建議的其餘預設值，然後按一下 [確定]。

        ![][03]

1. 在功能表右上方，選取您剛建立的 Azure Web 應用程式偵錯組態，然後按一下 [偵錯]

    ![][04]

1. 當系統顯示 [立即在遠端 Web 應用程式中啟用遠端偵錯？] 的提示時，按一下 [確定]。

1. 當系統顯示 [您的 Web 應用程式現在已準備好進行遠端偵錯] 的提示時，按一下 [確定]。

    ![][05]

1. 在功能表右上方，選取您剛建立的 Azure Web 應用程式偵錯組態，然後按一下 [偵錯]。

1. Windows 命令提示字元或 Unix 殼層隨即開啟並準備好必要的連接進行偵錯。您必須等到與遠端 Java Web 應用程式的連接成功之後，才能繼續。如果您使用 Windows，它將看起來類似下圖：

    ![][06]

1. 在 JSP 頁面中插入中斷點，然後在瀏覽器中開啟 Java Web 應用程式的 URL：

    1. 在 IntelliJ 中，開啟 [Azure 總管]。
    1. 瀏覽到 [Web Apps] 以及您要偵錯的 Java Web 應用程式。
    1. 以滑鼠右鍵按一下 Web 應用程式，然後按一下 [在瀏覽器中開啟]。
    1. IntelliJ 現在會進入偵錯模式。

## 後續步驟

如需如何搭配使用 Azure 與 Java 的詳細資訊，請參閱 [Azure Java 開發人員中心]。

如需建立 Azure Web Apps 的詳細資訊，請參閱 [Web 應用程式概觀]。

[AZURE.INCLUDE [app-service-web-try-app-service](../../includes/app-service-web-try-app-service.md)]

<!-- URL List -->

[Azure App Service]: http://go.microsoft.com/fwlink/?LinkId=529714
[適用於 IntelliJ 的 Azure 工具組]: ../azure-toolkit-for-intellij.md
[Installing the Azure Toolkit for IntelliJ (安裝適用於 IntelliJ 的 Azure 工具組)]: ../azure-toolkit-for-intellij-installation.md
[在 IntelliJ 中建立 Azure 的 Hello World Web 應用程式]: ./app-service-web-intellij-create-hello-world-web-app.md
[動態 Web 專案範例]: http://go.microsoft.com/fwlink/?LinkId=817337

[Azure Java 開發人員中心]: https://azure.microsoft.com/develop/java/
[Web 應用程式概觀]: ./app-service-web-overview.md

<!-- IMG List -->

[01]: ./media/app-service-web-debug-java-web-app-in-intellij/01-debug-java-web-app-in-intellij.png
[02]: ./media/app-service-web-debug-java-web-app-in-intellij/02-configure-intellij-remote-debug.png
[03]: ./media/app-service-web-debug-java-web-app-in-intellij/03-debug-configuration.png
[04]: ./media/app-service-web-debug-java-web-app-in-intellij/04-select-debug.png
[05]: ./media/app-service-web-debug-java-web-app-in-intellij/05-ready-for-remote-debugging.png
[06]: ./media/app-service-web-debug-java-web-app-in-intellij/06-windows-command-prompt-connection-successful-to-remote.png

<!---HONumber=AcomDC_0921_2016-->