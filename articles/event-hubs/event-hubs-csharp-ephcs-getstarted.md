<properties
	pageTitle="開始使用以 C# 撰寫的事件中樞 | Microsoft Azure"
	description="遵循此教學課程，以開始使用具備 C# 的 Azure 事件中樞並使用 EventProcessorHost。"
	services="event-hubs"
	documentationCenter=""
	authors="jtaubensee"
	manager="timlt"
	editor=""/>

<tags
	ms.service="event-hubs"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="hero-article"
	ms.date="09/02/2016"
	ms.author="jotaub;sethm"/>

# 開始使用事件中心

[AZURE.INCLUDE [service-bus-selector-get-started](../../includes/service-bus-selector-get-started.md)]

## 簡介

「事件中樞」是一種服務，可處理來自連接裝置和應用程式的大量事件資料 (遙測)。收集資料至「事件中樞」之後，可以使用存放裝置叢集來儲存資料，或使用即時分析提供者進行轉換。此大規模事件收集和處理功能是新型應用程式架構 (包括物聯網 (IoT)) 的重要元件。

本教學課程示範如何使用 Azure 傳統入口網站來建立事件中樞。另外也會示範如何使用以 C# 撰寫的主控台應用程式將訊息收集到「事件中樞」，以及如何使用 C# [事件處理器主機][]程式庫平行擷取訊息。

若要完成此教學課程，您需要下列項目：

+ [Microsoft Visual Studio](http://visualstudio.com)

+ 使用中的 Azure 帳戶。如果您沒有帳戶，只需要幾分鐘的時間就可以建立免費帳戶。如需詳細資訊，請參閱 [Azure 免費試用](https://azure.microsoft.com/free/)。

[AZURE.INCLUDE [event-hubs-create-event-hub](../../includes/event-hubs-create-event-hub.md)]

[AZURE.INCLUDE [service-bus-event-hubs-get-started-send-csharp](../../includes/service-bus-event-hubs-get-started-send-csharp.md)]

[AZURE.INCLUDE [service-bus-event-hubs-get-started-receive-ephcs](../../includes/service-bus-event-hubs-get-started-receive-ephcs.md)]

## 執行應用程式

現在您已經準備好執行應用程式。

1. 從 Visual Studio 內開啟您稍早建立的 **Receiver** 專案。

2. 在 **Receiver** 方案上按一下滑鼠右鍵，按一下 [新增]，然後按一下 [現有專案]。
 
3. 找到現有的 Sender.csproj 檔案，然後按兩下該檔案以將它新增到方案。
 
4. 再次於 **Receiver** 方案上按一下滑鼠右鍵，然後按一下 [屬性]。**Receiver** 屬性頁面隨即出現。

5. 按一下 [啟始專案]，然後按一下 [多個啟始專案] 按鈕。將 **Receiver** 和 **Sender** 專案的 [動作] 方塊設定為 [啟動]。

	![][19]

6. 按一下 [專案相依性]。在 [專案] 方塊中，按一下 [寄件者]。在 [取決於] 方塊中，確定已核取 **Receiver**。

	![][20]

7. 按一下 [確定] 以關閉 [屬性] 對話方塊。

1.	按 F5 以從 Visual Studio 內部執行 **Receiver** 專案，然後等它啟動所有資料分割的接收器。

	![][21]

2.	**Sender** 專案將會自動執行。按下主控台視窗中的 **Enter** 鍵，並查看出現在接收器視窗中的事件。

	![][22]

在 **Sender** 視窗中按 **Ctrl + C** 以結束 Sender 應用程式，然後在 Receiver 視窗中按 **Enter** 以關閉該應用程式。

## 後續步驟

您已經建置工作應用程式，可建立「事件中樞」和傳送及接收資料，接下來可進行下列案例：

- [使用事件中樞的完整範例應用程式][]。
- [使用「事件中樞」相應放大事件處理][]範例。
- 使用服務匯流排佇列的[佇列訊息解決方案][]。
- [事件中心概觀][]

<!-- Images. -->
[19]: ./media/event-hubs-csharp-ephcs-getstarted/create-eh-proj1.png
[20]: ./media/event-hubs-csharp-ephcs-getstarted/create-eh-proj2.png
[21]: ./media/event-hubs-csharp-ephcs-getstarted/run-csharp-ephcs1.png
[22]: ./media/event-hubs-csharp-ephcs-getstarted/run-csharp-ephcs2.png

<!-- Links -->
[Azure classic portal]: https://manage.windowsazure.com/
[事件處理器主機]: https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost
[事件中心概觀]: event-hubs-overview.md
[使用事件中樞的完整範例應用程式]: https://code.msdn.microsoft.com/Service-Bus-Event-Hub-286fd097
[使用「事件中樞」相應放大事件處理]: https://code.msdn.microsoft.com/Service-Bus-Event-Hub-45f43fc3
[佇列訊息解決方案]: ../service-bus/service-bus-dotnet-multi-tier-app-using-service-bus-queues.md
 

<!---HONumber=AcomDC_0928_2016-->