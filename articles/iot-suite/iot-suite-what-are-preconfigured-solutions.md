<properties
 pageTitle="Azure IoT 預先設定解決方案 | Microsoft Azure"
 description="說明 Azure IoT 預先設定解決方案及其架構，且包含其他資源的連結。"
 services=""
 suite="iot-suite"
 documentationCenter=""
 authors="dominicbetts"
 manager="timlt"
 editor=""/>

<tags
 ms.service="iot-suite"
 ms.devlang="na"
 ms.topic="get-started-article"
 ms.tgt_pltfrm="na"
 ms.workload="na"
 ms.date="08/09/2016"
 ms.author="dobett"/>

# 什麼是 Azure IoT 套件預先設定的解決方案？

Azure IoT 套件預先設定的解決方案是常見 IoT 解決方案模式的實作，您可以使用您的訂用帳戶將這些模式部署到 Azure。您可以使用預先設定的解決方案：

- 做為您的 IoT 解決方案的起點。
- 深入了解 IoT 解決方案設計與開發中的常見模式。

每個預先設定解決方案都是使用模擬裝置建置的完整端對端實作，藉以產生遙測資料。

除了部署並執行 Azure 中的解決方案之外，您也可以下載完整的原始程式碼，然後自訂和擴充解決方案以符合您的特定 IoT 需求。

> [AZURE.NOTE] 若要部署其中一個預先設定解決方案，請造訪 [Microsoft Azure IoT 套件][lnk-azureiotsuite]。[開始使用 IoT 預先設定的解決方案][lnk-getstarted-preconfigured]一文提供如何部署並執行其中的一個解決方案的相關詳細資訊。

下表顯示這些解決方案與特定的 IoT 功能如何對應：

| 方案 | 資料擷取 | 裝置身分識別 | 命令和控制 | 規則和動作 | 預測性分析 |
|------------------------|-----|-----|-----|-----|-----|
| [遠端監視][lnk-getstarted-preconfigured] | 是 | 是 | 是 | 是 | - | 
| [預測性維護][lnk-predictive-maintenance] | 是 | 是 | 是 | 是 | 是 |

- *資料擷取*：雲端規模的資料輸入。
- *裝置身分識別*：管理每個連接裝置的唯一身分識別。
- *命令與控制*：從雲端傳送訊息給裝置，讓裝置執行一些動作。
- *規則和動作*：解決方案後端會使用規則來處理特定的裝置對雲端資料。
- *預測性分析*：解決方案後端會在裝置對雲端資料上套用分析，以預測何時應執行特定動作。例如，分析飛機引擎遙測，以判斷何時應維護引擎。

## 遠端監視預先設定解決方案概觀

在本文中，我們選擇討論遠端監視預先設定解決方案，因為它說明了其他解決方案共用的許多常見設計元素。

下圖說明遠端監視解決方案的主要元素。下列各節提供關於這些元素的詳細資訊。

![遠端監視預先設定解決方案架構][img-remote-monitoring-arch]

## 裝置

當您部署遠端監視預先設定的解決方案時，有四個模擬裝置會預先佈建於解決方案中，以便模擬冷卻裝置。這些模擬裝置具有可發出遙測資訊的內建溫度和溼度模型。這些模擬裝置都會包含在內，以說明透過解決方案的端對端資料流程，如果您是要使用解決方案做為自訂實作起點的後端開發人員，這些裝置也可以提供方便的遙測資料來源與傳送命令的目標。

當裝置第一次連線到遠端監視預先設定的解決方案中的 IoT 中樞時，傳送到 IoT 中樞的裝置資訊訊息會列舉裝置可以回應的命令清單。在遠端監視預先設定的解決方案中，命令如下：

- *偵測裝置*：裝置會以確認來回應此命令。這適合用於檢查裝置是否仍在作用中和接聽。
- *開始遙測*：指示裝置開始傳送遙測資料。
- *停止遙測*：指示裝置停止傳送遙測資料。
- *變更設定點溫度*：控制裝置傳送的模擬溫度遙測值。這適合用於測試後端邏輯。
- *診斷遙測*：控制裝置是否應傳送外部溫度做為遙測資料。
- *變更裝置狀態*：設定裝置所報告的裝置狀態中繼資料屬性。這適合用於測試後端邏輯。

您可以將更多模擬裝置新增至解決方案，以便發出相同的遙測資料並回應相同的命令。

## IoT 中樞

在此預先設定解決方案中，IoT 中樞執行個體會對應至典型 [IoT 解決方案架構][lnk-what-is-azure-iot]中的「雲端閘道器」。

IoT 中樞會從單一端點接收來自裝置的遙測資料。IoT 中樞也會維護裝置可從中擷取命令的裝置特定端點。

IoT 中樞可讓您透過服務端遙測讀取端點接收遙測資料。

## Azure 串流分析

預先設定解決方案使用三個 [Azure 串流分析][lnk-asa] \(ASA) 作業來篩選來自裝置的遙測串流：


- *DeviceInfo 作業* - 將資料輸出到會路由裝置註冊特定訊息的事件中樞，在裝置第一次連線或回應 [變更裝置狀態] 命令時傳送給解決方案裝置註冊 (DocumentDB 資料庫)。
- *遙測作業* - 將所有的原始遙測資訊傳送到 Azure Blob 儲存體進行冷儲存，並計算會顯示在解決方案儀表板中的遙測彙總。
- *規則作業* - 篩選遙測串流中超出任何規則臨界值的值，並將資料輸出到事件中樞。當規則引發時，解決方案入口網站儀表板檢視會將此事件顯示為警示歷程記錄資料表中的新資料列，並根據在解決方案入口網站中的 [規則和動作] 檢視上定義的設定來觸發動作。

在此預先設定解決方案中，ASA 作業會形成典型 [IoT 解決方案架構][lnk-what-is-azure-iot]中「IoT 解決方案後端」的一部分。

## 事件處理器

在此預先設定解決方案中，事件處理器會形成典型 [IoT 解決方案架構][lnk-what-is-azure-iot]中「IoT 解決方案後端」的一部分。

**DeviceInfo** 和**規則** ASA 作業會將它們的輸出傳送到事件中樞，以傳遞給其他後端服務。解決方案會使用在 [WebJob][lnk-web-job] 中執行的 [EventPocessorHost][lnk-event-processor] 執行個體，來從這些事件中樞讀取訊息。**EventProcessorHost** 會使用 **DeviceInfo** 資料來更新 DocumentDB 資料庫中的裝置資料，並使用**規則**資料來叫用邏輯應用程式集並更新解決方案入口網站中的警示顯示。

## 裝置身分識別登錄與 DocumentDB

每個 IoT 中樞都包括儲存裝置金鑰的[裝置身分識別登錄][lnk-identity-registry]。IoT 中樞會使用此資訊驗證裝置 - 裝置必須先登錄並擁有有效的金鑰，才能連線至中樞。

此解決方案會儲存與裝置有關的其他資訊，例如它們的狀態、它們支援的命令，以及其他中繼資料。解決方案會使用 DocumentDB 資料庫儲存此解決方案特定裝置資料，且解決方案入口網站會從此 DocumentDB 資料庫抓取資料，以供顯示及編輯。

解決方案也必須讓裝置身分識別登錄中的資訊與 DocumentDB 資料庫中的內容維持同步。**EventProcessorHost** 會使用來自 **DeviceInfo** 串流分析作業的資料來管理同步處理。

## 解決方案入口網站

![解決方案儀表板][img-dashboard]

解決方案入口網站是 Web 型 UI，部署至雲端做為預先設定解決方案的一部分。可以讓您：

- 檢視儀表板中的遙測和警示歷程記錄。
- 佈建新裝置。
- 管理和監視裝置。
- 將命令傳送至特定裝置。
- 管理規則和動作。

在此預先設定解決方案中，解決方案入口網站會形成典型 [IoT 解決方案架構][lnk-what-is-azure-iot]中「IoT 解決方案後端」及「處理和業務連線」的一部分。

## 後續步驟

如需 IoT 解決方案架構的詳細資訊，請參閱 [Microsoft Azure IoT 服務 ︰參考架構][lnk-refarch]。

您現已知道預先設定的解決方案是什麼，您可以部署「遠端監視」預先設定的解決方案來開始使用︰[開始使用預先設定的解決方案][lnk-getstarted-preconfigured]。

[img-remote-monitoring-arch]: ./media/iot-suite-what-are-preconfigured-solutions/remote-monitoring-arch1.png
[img-dashboard]: ./media/iot-suite-what-are-preconfigured-solutions/dashboard.png
[lnk-what-is-azure-iot]: iot-suite-what-is-azure-iot.md
[lnk-asa]: https://azure.microsoft.com/documentation/services/stream-analytics/
[lnk-event-processor]: ../event-hubs/event-hubs-programming-guide.md#event-processor-host
[lnk-web-job]: ../app-service-web/web-sites-create-web-jobs.md
[lnk-identity-registry]: ../iot-hub/iot-hub-devguide.md#device-identity-registry
[lnk-predictive-maintenance]: iot-suite-predictive-overview.md
[lnk-azureiotsuite]: https://www.azureiotsuite.com/
[lnk-refarch]: http://download.microsoft.com/download/A/4/D/A4DAD253-BC21-41D3-B9D9-87D2AE6F0719/Microsoft_Azure_IoT_Reference_Architecture.pdf
[lnk-getstarted-preconfigured]: iot-suite-getstarted-preconfigured-solutions.md

<!---HONumber=AcomDC_0810_2016-->