<properties
 pageTitle="Azure IoT 中樞調整 | Microsoft Azure"
 description="描述如何調整 Azure IoT 中樞。"
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
 ms.date="09/19/2016"
 ms.author="elioda"/>

# 調整 IoT 中樞

Azure IoT 中樞最多可支援百萬個同時連線的裝置。如需詳細資訊，請參閱 [IoT 中樞價格][lnk-pricing]。每個 IoT 中樞單位允許一定數量的每日訊息。

為了適當調整您的解決方案，您必須考慮 IoT 中樞的特定使用方式。尤其要考慮以下類別的作業所需的尖峰輸送量：

* 裝置到雲端的訊息
* 雲端到裝置的訊息
* 身分識別登錄作業

除了此輸送量資訊之外，請參考 [IoT 中樞配額與節流][]並據以設計您的方案。

## 裝置到雲端及雲端到裝置訊息輸送量

若要評估每個單位的流量，最佳方式為調整 IoT 中樞解決方案的大小。

裝置到雲端訊息會遵循這些持續的輸送量指導方針。

| 層 | 持續的輸送量 | 持續的傳送速率 |
| ---- | -------------------- | ------------------- |
| S1 | 每個單位最多 1111 KB/分鐘<br/>(1.5 GB/天/單位) | 每個單位平均 278 個訊息/分鐘<br/>(400,000 個訊息/天/單位) |
| S2 | 每個單位最多 16 MB/分鐘<br/>(22.8 GB/天/單位) | 每個單位平均 4167 個訊息/分鐘<br/>(600 萬個訊息/天/單位) |
| S3 | 每個單位最多 814 MB/分鐘<br/>(1144.4 GB/天/單位) | 每個單位平均 208,333 個訊息/分鐘<br/>(3 億個訊息/天/單位) |

## 身分識別登錄作業輸送量

因為大部分的 IoT 中心識別登錄作業都與裝置佈建相關，所以不支援這些作業做為執行階段作業。

如需特定高載效能數據，請參閱 [IoT 中樞配額與節流][]。

## 分區化

雖然單一 IoT 中樞可以擴充到數百萬個裝置，但有時候您的解決方案需要單一 IoT 中樞無法保證的特定效能特性。在此情況下，建議您將裝置分割成多個 IoT 中樞。多個 IoT 中心可減緩資料傳輸量暴增，並取得所需的輸送量或作業速率。

## 後續步驟

若要進一步探索 IoT 中樞的功能，請參閱︰

- [開發人員指南][lnk-devguide]
- [使用範例 UI 探索裝置管理][lnk-dmui]
- [使用閘道 SDK 模擬裝置][lnk-gateway]
- [透過 Azure 入口網站管理 IoT 中樞][lnk-portal]

[lnk-pricing]: https://azure.microsoft.com/pricing/details/iot-hub
[IoT 中樞配額與節流]: iot-hub-devguide.md#throttling

[lnk-design]: iot-hub-guidance.md
[lnk-mqtt]: iot-hub-mqtt-support.md
[lnk-devices]: iot-hub-tested-configurations.md
[lnk-protocols]: iot-hub-protocol-gateway.md
[lnk-compare]: iot-hub-compare-event-hubs.md
[lnk-scaling]: iot-hub-scaling.md
[lnk-devguide]: iot-hub-devguide.md
[lnk-dmui]: iot-hub-device-management-ui-sample.md
[lnk-gateway]: iot-hub-linux-gateway-sdk-simulated-device.md
[lnk-portal]: iot-hub-manage-through-portal.md

<!---HONumber=AcomDC_0921_2016-->