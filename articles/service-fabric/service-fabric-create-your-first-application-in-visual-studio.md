<properties
   pageTitle="在 Visual Studio 中建立第一個 Service Fabric 應用程式 | Microsoft Azure"
   description="使用 Visual Studio 建立、部署和偵錯 Service Fabric 應用程式"
   services="service-fabric"
   documentationCenter=".net"
   authors="rwike77"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotNet"
   ms.topic="hero-article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="09/28/2016"
   ms.author="ryanwi"/>


# 建立第一個 Azure Service Fabric 應用程式

> [AZURE.SELECTOR]
- [C Sharp](service-fabric-create-your-first-application-in-visual-studio.md)
- [Java](service-fabric-create-your-first-linux-application-with-java.md)

Service Fabric SDK 包含的 Visual Studio 增益集提供用來建立、部署和偵錯 Service Fabric 應用程式的範本和工具。本主題逐步解說在 Visual Studio 中建立第一個應用程式的程序。

## 必要條件

開始之前，請確定您已[設定開發環境](service-fabric-get-started.md)。

## 影片逐步解說

下列影片逐步解說本教學課程中的步驟：

>[AZURE.VIDEO creating-your-first-service-fabric-application-in-visual-studio]

## 建立應用程式

Service Fabric 應用程式可以包含一或多個服務，而每個服務在提供應用程式的功能時都有特定角色。使用 [新增專案] 精靈，您可以隨著第一個服務專案一起建立應用程式專案。您稍後可以新增更多服務。

1. 以系統管理員身分啟動 Visual Studio。

2. 按一下 [檔案] > [新增專案] > [雲端] > [Service Fabric 應用程式]。

3. 替應用程式命名，然後按一下 [確定]。

	![Visual Studio 中的新增專案對話方塊][1]

4. 在下一頁選擇 [具狀態]，做為要包含在應用程式中的第一個服務類型。予以命名，然後按一下 [確定]。

	![Visual Studio 中的新增服務對話方塊][2]

	>[AZURE.NOTE] 如需選項的詳細資訊，請參閱 [Service Fabric 程式設計模型概觀](service-fabric-choose-framework.md)。

	Visual Studio 會建立應用程式專案和具狀態服務專案，並在 [方案總管] 中加以顯示。

	![使用具狀態服務建立應用程式後的方案總管][3]

	應用程式專案未直接包含任何程式碼。它反而會參考一組服務專案。此外，它包含三種其他類型的內容：

	- **發佈設定檔**：用來管理不同環境的工具喜好設定。

	- **指令碼**：包含用來部署/升級應用程式的 PowerShell 指令碼。Visual Studio 會在幕後使用指令碼。該指令碼也可以直接在命令列叫用。

	- **應用程式定義**︰包含 *ApplicationPackageRoot* 下的應用程式資訊清單。相關聯的應用程式參數檔案位於 *ApplicationParameters* 下方，它們可以定義應用程式，讓您得以特別為指定的環境設定應用程式。

    如需服務專案的內容概觀，請參閱[開始使用 Reliable Services](service-fabric-reliable-services-quick-start.md)。

## 部署和偵錯應用程式

您現在有一個應用程式，您可以試著執行它。

1. 在 Visual Studio 中按 F5，部署應用程式以供偵錯。

	>[AZURE.NOTE] 第一次部署需要一些時間，因為 Visual Studio 正在建立本機叢集進行開發。本機叢集會只會在一部電腦上執行您將在多部電腦的叢集中建置的相同平台程式碼。叢集建立狀態會顯示在 Visual Studio 輸出視窗中。

	備妥叢集時，您將會收到來自 SDK 隨附的本機叢集系統匣管理員應用程式的通知。

	![本機叢集系統匣通知][4]

2. 啟動應用程式後，Visual Studio 會自動顯示 [診斷事件檢視器]，以便查看服務的追蹤輸出。

	![診斷事件檢視器][5]

	在具狀態服務範本的情況下，訊息只會顯示在 MyStatefulService.cs 的 `RunAsync` 方法中遞增的計數器值。

3. 展開其中一個事件以查看更多詳細資料，包括程式碼執行所在的節點。在此情況下是 _Node_2，雖然這在您的電腦上可能有所不同。

	![診斷事件檢視器詳細資訊][6]

	本機叢集包含五個裝載於單一電腦上的節點。它是模仿有五個節點的叢集，其節點位於不同的電腦上。讓我們取下本機叢集上的其中一個節點來模擬遺失機器，以及在相同的時間運用 Visual Studio 偵錯工具。

    >[AZURE.NOTE] 專案範本所發出的應用程式診斷事件會使用內含的 `ServiceEventSource` 類別。如需詳細資訊，請參閱[如何在本機監視及診斷服務](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md)。

4. 在您的服務專案中尋找衍生自 StatefulService 的類別 (例如 MyStatefulService)，以及在 `RunAsync` 方法的第一行上設定中斷點。

	![具狀態服務 RunAsync 方法的中斷點][7]

5. 以滑鼠右鍵按一下 [本機叢集管理員] 系統匣應用程式並選擇 [管理本機叢集]，以啟動 Service Fabric Explorer。

    ![從本機叢集管理員啟動 Service Fabric Explorer][systray-launch-sfx]

    Service Fabric Explorer 提供了叢集的視覺表示法，包括部署至叢集的應用程式集以及構成叢集的實體節點集合。若要了解 Service Fabric Explorer，請參閱[視覺化叢集](service-fabric-visualizing-your-cluster.md)。

6. 在左窗格中展開 [叢集] > [節點]，並尋找您的程式碼執行所在的節點。

7. 按一下 [動作] > [停用 (重新啟動)] 來模擬電腦重新啟動。(請注意，您也可以從左窗格之節點清單檢視中的內容功能表停用)。

	![在 Service Fabric Explorer 中停止節點][sfx-stop-node]

	您應該會在 Visual Studio 中短暫看見達到您的中斷點，因為您一個節點上所做的計算完美地容錯移轉至另一個節點。

8. 返回 [診斷事件檢視器] 並觀察訊息。請注意，計數器已繼續遞增，即使事件實際上來自不同的節點。

    ![容錯移轉之後的診斷事件檢視器][diagnostic-events-viewer-detail-post-failover]

## 切換叢集模式

根據預設，本機開發叢集已設定為以 5 個節點的叢集方式執行，這很適合用於偵錯跨多個節點部署的服務。不過，將應用程式部署到 5 個節點的開發叢集可能需要一些時間。如果您想要快速反覆執行程式碼變更，但不要在 5 個節點上執行您的應用程式，您可以將開發叢集切換到 1 個節點的模式。若要在具有一個節點的叢集上執行您的程式碼，請以滑鼠右鍵按一下系統匣中的 [本機叢集管理員] 並選取 [交換器叢集模式] -> [1 個節點]。

![切換叢集模式][switch-cluster-mode]

當您變更叢集模式時，開發叢集會重設，而且將移除在此叢集上佈建或執行的所有應用程式。

## 清除

  在我們做結論之前，請務必記得本機叢集非常真實。停止偵錯工具將會移除應用程式執行個體，並取消註冊應用程式類型。但叢集會繼續在背景中執行。您有數個選項可管理叢集：

  1. 若要關閉叢集，但保留應用程式資料及追蹤，請按一下系統匣應用程式中的 [停止本機叢集]。

  2. 若要完全刪除叢集，請按一下系統匣應用程式中的 [移除本機叢集]。請注意，此選項會導致下次您在 Visual Studio 中按 F5 鍵時發生其他緩慢部署。唯有您打算停用本機叢集一陣子或需要回收資源時，再刪除叢集。

## 後續步驟

- 了解如何[在 Azure 中建立叢集](service-fabric-cluster-creation-via-portal.md)或[在 Windows 上建立獨立叢集](service-fabric-cluster-creation-for-windows-server.md)。
- 嘗試使用 [Reliable Services](service-fabric-reliable-services-quick-start.md) 或 [Reliable Actors](service-fabric-reliable-actors-get-started.md) 程式設計模型來建立服務。
- 了解如何透過 [Web 服務前端](service-fabric-add-a-web-frontend.md)在網際網路公開服務。
- 逐步完成[實際操作實驗室](https://msdnshared.blob.core.windows.net/media/2016/07/SF-Lab-Part-I.docx)及建立無狀態服務、設定監視和健康情況報告，以及執行應用程式升級。

<!-- Image References -->

[1]: ./media/service-fabric-create-your-first-application-in-visual-studio/new-project-dialog.png
[2]: ./media/service-fabric-create-your-first-application-in-visual-studio/new-project-dialog-2.png
[3]: ./media/service-fabric-create-your-first-application-in-visual-studio/solution-explorer-stateful-service-template.png
[4]: ./media/service-fabric-create-your-first-application-in-visual-studio/local-cluster-manager-notification.png
[5]: ./media/service-fabric-create-your-first-application-in-visual-studio/diagnostic-events-viewer.png
[6]: ./media/service-fabric-create-your-first-application-in-visual-studio/diagnostic-events-viewer-detail.png
[7]: ./media/service-fabric-create-your-first-application-in-visual-studio/runasync-breakpoint.png
[sfx-stop-node]: ./media/service-fabric-create-your-first-application-in-visual-studio/sfe-deactivate-node.png
[systray-launch-sfx]: ./media/service-fabric-create-your-first-application-in-visual-studio/launch-sfx.png
[diagnostic-events-viewer-detail-post-failover]: ./media/service-fabric-create-your-first-application-in-visual-studio/diagnostic-events-viewer-detail-post-failover.png
[sfe-delete-application]: ./media/service-fabric-create-your-first-application-in-visual-studio/sfe-delete-application.png
[switch-cluster-mode]: ./media/service-fabric-create-your-first-application-in-visual-studio/switch-cluster-mode.png

<!---HONumber=AcomDC_0928_2016-->