<properties
	pageTitle="教學課程︰DevOps 與 Azure 入口網站 | Microsoft Azure"
	description="了解 Azure 入口網站中的各種 DevOps 工作流程。"
	services="azure-portal"
	documentationCenter=""
	authors="mlearned"
	manager="douge"
	editor="mlearned"/>

<tags
	ms.service="multiple"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="hero-article"
	ms.date="06/05/2016"
	ms.author="mlearned"/>

# 教學課程︰DevOps 與 Azure 入口網站

Azure 平台充滿著彈性的 DevOps 工作流程。在本教學課程中，您將會了解如何運用 Azure 入口網站的功能，來開發、測試、部署、疑難排解、監視和管理執行中的應用程式。本教學課程著重於下列內容︰

1.  建立 Web 應用程式並啟用連續部署

2.  開發和測試應用程式

3.  監視和針對應用程式進行疑難排解

4.  一般應用程式管理工作

## 建立 Web 應用程式並啟用連續部署

請使用 [Azure App Service](https://azure.microsoft.com/services/app-service/) 來建立 Web 應用程式，以用於本教學課程的其餘部分。一開始，您將會進行從原始程式碼儲存機制到執行中 Azure 環境的連續部署。

1.  登入 Azure 入口網站

2.  選擇 [App Service] &gt; [新增] 圖示並輸入名稱，選擇訂用帳戶，然後建立新的資源群組來做為服務的容器。

    資源群組可讓您管理解決方案的各個層面，例如計費、部署和監視，而這一切全都可以透過 [Azure Resource Manager](https://azure.microsoft.com/documentation/articles/resource-group-overview/) 以單一群組的形式來進行。

	![image1][image1]


3.  幾分鐘後，您的 App Service 便會建立。請花幾分鐘的時間在入口網站中探索服務的各種功能表選項。

	![image2][image2]

4.  按一下 URL。請注意工具和儲存機制的可用選項變化。您也可以使用自己選擇的語言和架構，包括 .NET、Java 和 Ruby。

	![image3][image3]

3.  Azure 入口網站可讓連續部署變成只須執行幾個簡單步驟的輕鬆程序。在 Azure 入口網站中，從您剛才建立的 App Service 的圖示中選擇 [設定]。

    ![image4][image4]

    在右側開啟的刀鋒視窗中，捲動到 [發佈] 區段。

    ![image5][image5]

4.  接著，進行一些設定，讓應用程式能夠進行連續部署。按一下 [部署來源]，然後按一下 [選擇來源]。請注意儲存機制來源所具有的選項變化。

	![image6][image6]

1.  在此範例中，請選擇 [GitHub]。選擇性地選擇您所選擇的儲存機制，並設定授權認證。

    ![image7][image7]

2.  在授權給儲存機制之後，您接著可以選擇想要部署的專案和分支。下面列出幾個虛構的範例。

    ![image8][image8]

3.  在選擇專案和分支之後，按一下 [確定]。您應該會開始看到部署通知。

    ![image9][image9]

4.  瀏覽回到 Github，以查看為了整合原始檔控制儲存機制與 Azure 而建立的 Webhook。只須進行一些簡單的步驟，Azure 入口網站即可與 Github 整合。

    ![image10][image10]

5.  為了示範連續部署，請快速新增一些內容到儲存機制。如需簡單的範例，可將範例文字檔新增至 Github 儲存機制。您可以自由地使用 .NET、Ruby、Python 或其他某些類型的應用程式來搭配 App Service。請隨意新增文字檔、ASP.NET MVC、Java 或 Ruby 應用程式到您選擇的儲存機制。

    ![image11][image11]

6.  在對儲存機制認可變更之後，您會看到入口網站的通知區域中起始了新的部署。在對儲存機制認可之後，如果沒快速看到變更，請按一下 [同步處理]。

    ![image12][image12]

7.  此時，如果您嘗試並載入 App Service 頁面，可能會收到 403 錯誤。在此範例中，這是由於頁面 (例如 index.htm 或 default.html 之類的檔案) 沒有典型的預設文件設定。使用 Azure 入口網站中的工具，即可快速解決此問題。在 Azure 入口網站中，選擇 [設定] &gt; [應用程式設定]。

	 ![image13][image13]

8.  隨即會開啟 [應用程式設定] 的刀鋒視窗。輸入 “SamplePage.html” 頁面的名稱，然後按一下 [儲存]。花幾分鐘的時間探索其他設定。

	![Image14][image14]

9.  選擇性地重新整理瀏覽器 URL，以確保您會看到預期的變更。在本例中，還有一些簡單的文字現在正在填入頁面。儲存機制的每項額外變更都會導致進行新的自動部署。

    ![Image15][image15]

    透過 Azure 入口網站啟用連續部署是很輕鬆的體驗。您也可以建置更複雜的發行管線，並搭配現有原始檔控制和連續整合系統，使用其他許多技術來部署至 Azure，例如運用自動化建置和發行管理系統。

## 開發和測試應用程式

接下來，對程式碼基底進行一些變更，並快速部署這些變更。您也會針對 Web 應用程式設定一些效能測試。

1.  在 Azure 入口網站中，從瀏覽窗格選擇應用程式服務，並找到您的 App Service。

    ![Image16][image16]

2.  按一下 [工具]

    ![Image17][image17]

3.  注意 [工具] 底下的 [開發] 類別。這裡有幾個實用的工具，可供我們使用應用程式，但不需要離開 Azure 入口網站。按一下 [主控台]。

    ![Image18][image18]

4.  在 [主控台] 視窗中，您可以為您的應用程式發出即時命令。輸入 dir 命令，然後按 Enter 鍵。請注意，需要較高權限的命令不會運作。

    ![Image19][image19]

5.  回到 [開發] 類別，然後選擇 [Visual Studio Online]。注意：Visual Studio Online 現已更名為 Visual Studio Team Services。

    ![Image20][image20]

6.  開啟應用程式的瀏覽器中編輯體驗。

    ![Image21][image21]

7.  隨即會為應用程式安裝 Web 擴充功能。擴充功能可快速且輕鬆地將功能新增至 Azure 中的應用程式。請注意以下螢幕擷取畫面中可用的一些其他擴充功能類型。

    ![Image22][image22]

8.  在安裝 Visual Studio Online 擴充功能之後，按一下 [執行]。

    ![Image23][image23]

9.  隨即會開啟瀏覽器索引標籤，供您直接在瀏覽器中看到開發 IDE。請注意，下面的體驗是在 Chrome 中。

    ![Image24][image24]

10. 您可以執行數個活動，例如編輯檔案、新增檔案與資料夾，以及從即時網站下載內容。快速編輯 SamplePage.html 檔案。

    ![Image25][image25]

11. 幾分鐘後，便會自動儲存變更。如果您往回瀏覽至網頁，就可以看到變更。請記住，這類即時編輯很可能不適用於生產環境。不過，這些工具可以輕易地快速變更開發和測試環境。

    ![image26][image26]

    ![image27][image27]

12. 回到 [工具] 刀鋒視窗，然後按一下 [開發] 類別下的 [效能測試]。

    ![Image28][image28]

13. 您需要設定 Team Services 帳戶。如需詳細資訊，請參閱此處︰[建立 Team Services 帳戶](https://www.visualstudio.com/docs/setup-admin/team-services/sign-up-for-visual-studio-team-services)

14. 按一下 [新增] 以建立效能測試。

    ![Image29][image29]

    設定各種值，然後按一下對話方塊底部的 [執行測試] 以起始效能測試。

    ![Image30][image30]

	![Image31][image31]

1.  在測試開始執行之後，您可以監視狀態。

    ![Image32][image32]

    在測試完成時，按一下結果以顯示詳細資訊。

    ![Image33][image33]

2.  在此範例中，您建立了小型測試執行，所以可供分析的資料有限，但您可以查看各種度量，以及從這個檢視重新執行測試。Azure 入口網站可以讓建立、執行和分析 Web 效能測試變成簡單的程序。以下螢幕擷取畫面會顯示效能資料。

    ![Image34][image34]

    ![Image35][image35]

    ![Image36][image36]

## 監視和針對應用程式進行疑難排解

Azure 提供許多用來監視和針對執行中應用程式進行疑難排解的功能。

1.  在我們的 Web 應用程式的 Azure 入口網站中選擇 [工具]。

    ![Image37][image37]

2.  在 [疑難排解] 類別下，請注意各種使用工具來針對執行中應用程式潛在問題進行疑難排解的選項。您可以執行監視即時 HTTP 流量、啟用自我修復、檢視記錄檔等等的作業。

    ![Image38][image38]

3.  選擇 [網站度量]，快速檢視某些 HTTP 代碼。

    ![image39][image39]

4.  選擇 [診斷即服務]。選擇 [應用程式類型]，然後選擇 [執行]。

    ![image40][image40]

    隨即會開始收集。

    ![image41][image41]

5.  您可以選擇適當的記錄檔來診斷潛在的問題。您必須啟用記錄功能，才能查看所有可用的資料選項，例如 HTTP 記錄。

    ![image42][image42]

    按一下 [記憶體傾印] 檔案，即可下載並分析 DebugDiag 分析報告，以協助找出潛在的問題。

    ![image43][image43]

6.  若要檢視更多資料，您必須啟用其他記錄功能。在 Azure 入口網站中，瀏覽至 Web 應用程式並選擇 [設定]。

    ![image44][image44]

7.  向下捲動至 [功能] 類別，並選擇 [診斷記錄]。

  	 ![image45][image45]

8.  請注意記錄功能的各種選項。開啟 [Web 伺服器記錄]，並按一下 [儲存]。

    ![image46][image46]

9.  回到應用程式的工具區域，並選擇 [診斷即服務]，然後按一下 [執行] 以重新執行資料收集。

    ![image47][image47]

10. 在啟用 HTTP 記錄設定的情況下，您現在會看到針對 HTTP 記錄收集的資料。

    ![image48][image48]

11. 按一下 HTML 檔案記錄，就會產生豐富的瀏覽器型報告，以便進行進一步的調查。

    ![image49][image49]

12. 回到應用程式在 Azure 入口網站中的 [工具] 區段。捲動至 [工具] 區段，然後選擇 [處理序總管]。

    ![image50][image50]

13. 藉由選擇處理序總管，您即可檢視執行中處理序的詳細資料。請注意下面，您可以深入探索處理序，甚至是刪除處理序，全都透過 Azure 入口網站來進行。

    ![image51][image51]

    ![image52][image52]

14. 回到左側的 [設定] 刀鋒視窗。按一下 [新增支援要求]。

    ![image53][image53]

15. 在右側的刀鋒視窗中，您可以填寫問題的詳細資料、輸入連絡人資訊，甚至是上傳診斷資料。Azure 入口網站可讓使用 Microsoft 支援變成順暢的體驗。

    ![image54][image54]

    ![image55][image55]

    Azure 入口網站可協助提供強大且熟悉的工具體驗，來協助監視和針對執行中的應用程式進行疑難排解。您也可以透過執行各種工作，例如回收處理序、啟用和停用各種資料收集，甚至是與 Microsoft 專業支援服務整合，以快速地採取行動。

## 一般應用程式管理

在管理應用程式時，您經常必須執行各式各樣的活動，例如設定備份策略、實作和管理識別提供者，以及設定角色型存取控制。和其他 DevOps 體驗一樣，Azure 平台會將這些工作直接整合到入口網站。

1.  為了確保 Web 應用程式安全無虞，不會遺失資料，您必須設定備份。請瀏覽至 Web 應用程式的 [設定] 區域。

    ![image56][image56]

2.  在右側的刀鋒視窗中，向下捲動至 [功能] 類別。

	 ![image57][image57]

1.  選擇 [備份]；右側隨即會開啟刀鋒視窗。

    ![image58][image58]

2.  按一下 [設定]，在右側的刀鋒視窗中選擇儲存體帳戶。

    ![image59][image59]

3.  現在，建立並選擇儲存體容器以保存備份。按一下刀鋒視窗底部的 [建立]。然後選取容器。

    ![image60][image60]

4.  在選擇容器之後，即可設定排程，以及設定資料庫的備份。在此案例中，按一下 [儲存] 圖示。

     ![image61][image61]

5.  在儲存之後，往回捲動至位於左側的刀鋒視窗來找到 [備份]。按一下 [立即備份] 以備份應用程式。

     ![image62][image62]

6.  幾分鐘後，您便會看到建立了備份。請注意以下螢幕擷取畫面上的 [立即還原] 選項。

     ![image63][image63]

7.  按一下 [立即還原]，並檢查右側的刀鋒視窗選項。您可以選擇適當的備份，並視需要輕鬆地還原成先前的狀態。Azure 入口網站已幫助我們輕鬆地啟用應用程式的簡單災害復原策略。

     ![image64][image64]

8.  回到左側的 [設定] 刀鋒視窗，並選擇 [功能] 底下的 [驗證/授權]。

     ![image65][image65]

9.  在右側的刀鋒視窗中，選擇 [App Service 驗證]。請注意您可以針對熱門提供者設定的各種選項。

     ![image66][image66]

10. 選擇您所選擇的提供者，並注意該範圍的選項。您可以提供 [應用程式識別碼] 與 [應用程式密碼]，並快速啟用應用程式的 Facebook 驗證。Azure 入口網站可讓驗證做為應用程式的周全解決方案。

     ![image67][image67]

11. 回到 [設定] 刀鋒視窗，並選擇 [資源管理] 類別底下的 [使用者]。

     ![image68][image68]

12. 在右側的刀鋒視窗中，檢查各種用來新增角色和使用者的選項。Azure 入口網站可讓您輕鬆控制應用程式的 RBAC (角色型存取控制)。

     ![image69][image69]


## 摘要

本教學課程透過快速啟用 Web 應用程式的連續部署、執行各種開發和測試活動、監視和針對 Live App 進行疑難排解，以及最終管理金鑰策略 (例如災害復原、身分識別和角色型存取控制)，示範了 Azure 平台的部分功能。Azure 平台可讓這些 DevOps 工作流程獲得整合的體驗，而且您可以透過讓手邊的工作保持在環境中而能有效率地工作。


## 後續步驟 

* Azure Resource Manager 對於在 Azure 平台上啟用 DevOps 來說至關重要。若要深入了解，請造訪 [Azure Resource Manager 概觀](../resource-group-overview.md)。

* 若要深入了解 Azure App Service 部署，請瀏覽[將您的應用程式部署至 Azure App Service](../app-service-web/web-sites-deploy.md)


[image1]: ./media/tutorial-azureportal-devops/image1.png
[image2]: ./media/tutorial-azureportal-devops/image2.png
[image3]: ./media/tutorial-azureportal-devops/image3.png
[image4]: ./media/tutorial-azureportal-devops/image4.png
[image5]: ./media/tutorial-azureportal-devops/image5.png
[image6]: ./media/tutorial-azureportal-devops/image6.png
[image7]: ./media/tutorial-azureportal-devops/image7.png
[image8]: ./media/tutorial-azureportal-devops/image8.png
[image9]: ./media/tutorial-azureportal-devops/image9.png
[image10]: ./media/tutorial-azureportal-devops/image10.png
[image11]: ./media/tutorial-azureportal-devops/image11.png
[image12]: ./media/tutorial-azureportal-devops/image12.png
[image13]: ./media/tutorial-azureportal-devops/image13.png
[image14]: ./media/tutorial-azureportal-devops/image14.png
[image15]: ./media/tutorial-azureportal-devops/image15.png
[image16]: ./media/tutorial-azureportal-devops/image16.png
[image17]: ./media/tutorial-azureportal-devops/image17.png
[image18]: ./media/tutorial-azureportal-devops/image18.png
[image19]: ./media/tutorial-azureportal-devops/image19.png
[image20]: ./media/tutorial-azureportal-devops/image20.png
[image21]: ./media/tutorial-azureportal-devops/image21.png
[image22]: ./media/tutorial-azureportal-devops/image22.png
[image23]: ./media/tutorial-azureportal-devops/image23.png
[image24]: ./media/tutorial-azureportal-devops/image24.png
[image25]: ./media/tutorial-azureportal-devops/image25.png
[image26]: ./media/tutorial-azureportal-devops/image26.png
[image27]: ./media/tutorial-azureportal-devops/image27.png
[image28]: ./media/tutorial-azureportal-devops/image28.png
[image29]: ./media/tutorial-azureportal-devops/image29.png
[image30]: ./media/tutorial-azureportal-devops/image30.png
[image31]: ./media/tutorial-azureportal-devops/image31.png
[image32]: ./media/tutorial-azureportal-devops/image32.png
[image33]: ./media/tutorial-azureportal-devops/image33.png
[image34]: ./media/tutorial-azureportal-devops/image34.png
[image35]: ./media/tutorial-azureportal-devops/image35.png
[image36]: ./media/tutorial-azureportal-devops/image36.png
[image37]: ./media/tutorial-azureportal-devops/image37.png
[image38]: ./media/tutorial-azureportal-devops/image38.png
[image39]: ./media/tutorial-azureportal-devops/image39.png
[image40]: ./media/tutorial-azureportal-devops/image40.png
[image41]: ./media/tutorial-azureportal-devops/image41.png
[image42]: ./media/tutorial-azureportal-devops/image42.png
[image43]: ./media/tutorial-azureportal-devops/image43.png
[image44]: ./media/tutorial-azureportal-devops/image44.png
[image45]: ./media/tutorial-azureportal-devops/image45.png
[image46]: ./media/tutorial-azureportal-devops/image46.png
[image47]: ./media/tutorial-azureportal-devops/image47.png
[image48]: ./media/tutorial-azureportal-devops/image48.png
[image49]: ./media/tutorial-azureportal-devops/image49.png
[image50]: ./media/tutorial-azureportal-devops/image50.png
[image51]: ./media/tutorial-azureportal-devops/image51.png
[image52]: ./media/tutorial-azureportal-devops/image52.png
[image53]: ./media/tutorial-azureportal-devops/image53.png
[image54]: ./media/tutorial-azureportal-devops/image54.png
[image55]: ./media/tutorial-azureportal-devops/image55.png
[image56]: ./media/tutorial-azureportal-devops/image56.png
[image57]: ./media/tutorial-azureportal-devops/image57.png
[image58]: ./media/tutorial-azureportal-devops/image58.png
[image59]: ./media/tutorial-azureportal-devops/image59.png
[image60]: ./media/tutorial-azureportal-devops/image60.png
[image61]: ./media/tutorial-azureportal-devops/image61.png
[image62]: ./media/tutorial-azureportal-devops/image62.png
[image63]: ./media/tutorial-azureportal-devops/image63.png
[image64]: ./media/tutorial-azureportal-devops/image64.png
[image65]: ./media/tutorial-azureportal-devops/image65.png
[image66]: ./media/tutorial-azureportal-devops/image66.png
[image67]: ./media/tutorial-azureportal-devops/image67.png
[image68]: ./media/tutorial-azureportal-devops/image68.png
[image69]: ./media/tutorial-azureportal-devops/image69.png

<!---HONumber=AcomDC_0615_2016--->