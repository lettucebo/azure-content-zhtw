<properties
 pageTitle="IoT 中樞 HA 和 DR | Microsoft Azure"
 description="描述可使用災害復原功能協助建立高可用性 IoT 解決方案的功能。"
 services="iot-hub"
 documentationCenter=""
 authors="fsautomata"
 manager="timlt"
 editor=""/>

<tags
 ms.service="iot-hub"
 ms.devlang="na"
 ms.topic="article"
 ms.tgt_pltfrm="na"
 ms.workload="na"
 ms.date="02/03/2016"
 ms.author="elioda"/>

# IoT 中樞高可用性和災害復原

作為 Azure 服務，IoT 中心提供高可用性 (HA)，在 Azure 區域層級使用備援，且解決方案不需任何額外的工作。此外，Azure 會提供一些可協助建立解決方案的功能，必要時也提供災害復原 (DR) 功能或跨區域的可用性。若要提供全域、跨區域的高可用性給裝置或使用者，必須設計及準備解決方案以善用這些 DR 功能。[Azure 業務持續性技術指引](../resiliency/resiliency-technical-guidance.md)一文描述業務持續性和 DR 的 Azure 內建功能。[Azure 應用程式的災害復原與高可用性][]一文針對 Azure 應用程式的策略提供架構指引以達到 HA 和 DR。

## Azure IoT 中樞 DR
除了內部區域 HA，IoT 中樞會實作不需要使用者介入的災害復原容錯移轉機制。IoT 中樞 DR 會自行起動，其復原時間目標 (RTO) 為 2 到 26 小時，以及下列復原點目標 (RPO)。

| 功能 | RPO |
| ------------- | --- |
| 登錄和通訊作業的服務可用性 | 可能的 CName 遺失 |
| 裝置身分識別登錄中的身分識別資料 | 0 到 5 分鐘的資料遺失 |
| 裝置到雲端的訊息 | 所有未讀取的訊息都會遺失 |
| 作業監視訊息 | 所有未讀取的訊息都會遺失 |
| 雲端到裝置的訊息 | 0 到 5 分鐘的資料遺失 |
| 雲端到裝置的意見反應佇列 | 所有未讀取的訊息都會遺失 |

## 使用 IoT 中心的區域容錯移轉

IoT 解決方案中部署拓撲的完整處理已超出本文的範圍，但是為了高可用性和災害復原，我們要考慮*區域容錯移轉*部署模型。

在區域容錯移轉模型中，解決方案後端主要是在一個資料中心位置執行，但其他 IoT 中樞和後端將會部署在其他資料中心位置以進行容錯移轉，以防主要資料中心中的 IoT 中樞發生中斷，或者從裝置到主要資料中心的網路連線因某種理由而中斷。每當無法連接主要閘道器時，裝置會使用次要服務端點。使用跨區域容錯移轉功能時，可改善解決方案的可用性，勝過單一區域的高可用性。

在較高層級，為了使用 IoT 中樞實作區域容錯移轉模型，您需要下列內容。

* **次要 IoT 中樞和裝置路由邏輯**：萬一主要區域的服務中斷，裝置必須開始與您的次要地區連接。由於大部分服務狀態感知的本質，解決方案的系統管理員通常會觸發區域間的容錯移轉程序。要讓新端點與裝置通訊，同時保有程序的控制權，最佳方式是讓它們定期檢查*指引*服務是否有目前作用中的端點。這裡的指引服務可以是簡單的 Web 應用程式，其可藉由使用 DNS 重新導向技術複寫並保持連接 (例如使用 [Azure 流量管理員][])。
* **身分識別登錄複寫** - 為了成為可用狀態，次要 IoT 中樞必須包含能夠連接到解決方案的所有裝置身分識別。解決方案應該保留裝置身分識別的異地複寫備份，並在切換裝置的作用中端點之前將其上傳至次要 IoT 中樞。IoT 中樞的裝置身分識別匯出功能在此內容中非常有用。如需詳細資訊，請參閱 [IoT 中樞開發人員指南 - 身分識別登錄][]。
* **合併邏輯** - 當主要區域再次可供使用時，所有在次要站台中建立的狀態和資料都必須移轉回主要區域。這大多與裝置身分識別和應用程式中繼資料相關，必須與主要 IoT 中樞合併，也可能要與主要區域中的其他應用程式專屬存放區合併。若要簡化這個步驟，通常建議使用等冪作業。這樣可以將副作用降到最低，不只包括來自最終一致的事件分佈的副作用，也包括來自事件的重複項目或失序傳遞的副作用。此外，應用程式邏輯應該設計為能夠容忍潛在的不一致或「稍微」過期的狀態。這是因為系統需要額外的時間來根據復原點目標 (RPO)「療癒」。

## 後續步驟

遵循下列連結以深入了解 Azure IoT 中樞：

- [開始使用 IoT 中心 (教學課程)][lnk-get-started]
- [何謂 Azure IoT 中心？][]

[Azure resiliency technical guidance]: ../resiliency/resiliency-technical-guidance.md
[Azure 應用程式的災害復原與高可用性]: ../resiliency/resiliency-disaster-recovery-high-availability-azure-applications.md
[Failsafe: Guidance for Resilient Cloud Architectures]: https://msdn.microsoft.com/library/azure/jj853352.aspx
[Azure 流量管理員]: https://azure.microsoft.com/documentation/services/traffic-manager/
[IoT 中樞開發人員指南 - 身分識別登錄]: iot-hub-devguide.md#identityregistry

[lnk-get-started]: iot-hub-csharp-csharp-getstarted.md
[何謂 Azure IoT 中心？]: iot-hub-what-is-iot-hub.md

<!---HONumber=AcomDC_0921_2016-->