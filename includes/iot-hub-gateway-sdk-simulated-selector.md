> [AZURE.SELECTOR]
- [Linux](../articles/iot-hub/iot-hub-linux-gateway-sdk-simulated-device.md)
- [Windows](../articles/iot-hub/iot-hub-windows-gateway-sdk-simulated-device.md)

[Simulated Device Cloud Upload sample] \(Simulated Device Cloud Upload 範例) 的這個逐步解說示範如何使用 [Microsoft Azure IoT Gateway SDK (Microsoft Azure IoT 閘道 SDK)][lnk-sdk]，以將裝置到雲端遙測從模擬裝置傳送到 IoT 中樞。

本逐步解說涵蓋下列項目：

1. **架構**：Simulated Device Cloud Upload 範例的重要架構資訊。

2. **建置和執行**︰建置和執行範例所需的步驟。

## 架構

Simulated Device Cloud Upload 範例示範如何使用 SDK 建立閘道，以將遙測從模擬裝置傳送到 IoT 中樞。模擬裝置無法直接連接到 IoT 中樞，因為︰

- 裝置不會使用 IoT 中樞所了解的通訊協定。
- 裝置不夠聰明到記住 IoT 中樞指派給它們的身分識別。

閘道會使用下列方式來解決模擬裝置的這些問題︰

- 閘道了解模擬裝置所使用的通訊協定、從裝置接收裝置到雲端遙測，並使用集線器所了解的通訊協定將這些訊息轉送到 IoT 中樞。
- 閘道會代表模擬裝置儲存 IoT 中樞身分識別，並在模擬裝置將訊息傳送到 IoT 中樞時作為 Proxy。

下圖顯示範例的主要元件 (包含閘道模組)︰

![][1]


> [AZURE.NOTE] 模組不會彼此直接傳遞訊息。模組會將訊息發佈到內部訊息匯流排，而內部訊息匯流排會使用訂用帳戶機制將訊息傳遞到其他模組 (如下圖所示)。如需詳細資訊，請參閱[開始使用閘道 SDK][lnk-gw-getstarted]。

### 通訊協定擷取模組

此模組是透過閘道從裝置取得資料以及將資料傳遞到雲端的起點。在此範例中，模組會執行四項工作︰

1.  它會建立模擬溫度資料。
    
    注意︰如果您使用實際裝置，此模組會讀取那些實體裝置中的資料。

2.  它會將模擬溫度資料放到訊息的內容。

3.  並將具有假 MAC 位址的屬性加入包含模擬溫度資料的訊息。

4.  它讓訊息可供鏈結中的下一個模組使用。

> [AZURE.NOTE] 在原始程式碼中，上圖中的「通訊協定 X 擷取」模組稱為「模擬裝置」。

### MAC &lt;-&gt; IoT 中樞識別碼模組

此模組會掃描包含屬性的訊息，這個屬性包含模擬裝置的 MAC 位址 (由通訊協定擷取模組所加入)。如果模組發現這類屬性，會將另一個具有 IoT 中樞裝置金鑰的屬性加入訊息，然後讓訊息可供鏈結中的下一個模組使用。這是範例如何建立 IoT 中樞裝置身分識別與模擬裝置的關聯。設定模組時，開發人員會手動設定 MAC 位址與 IoT 中樞身分識別之間的對應。

> [AZURE.NOTE]  此範例使用 MAC 位址作為唯一裝置識別碼，並將它與 IoT 中樞裝置身分識別相互關聯。不過，您可以撰寫使用不同唯一識別碼的專屬模組。例如，您的裝置可能具有唯一序號或有其中內嵌唯一裝置名稱的遙測資料，而唯一裝置名稱可用來判斷 IoT 中樞裝置識別身分。

### IoT 中樞通訊模組

此模組採用具有前一個模組所指派之 IoT 中樞裝置識別身分的訊息，並使用 HTTPS 將訊息內容傳送到 IoT 中樞。HTTPS 是 IoT 中樞所了解的三種通訊協定中的其中一種。

此模組會開啟從閘道到 IoT 中樞的單一 HTTP 連接，而且透過該連接對來自所有模擬裝置的連接進行多工處理，而不是開啟每個模擬裝置的 IoT 中樞連接。這可讓單一閘道所連接的裝置 (模擬或其他) 多於為每個裝置開啟唯一連接時的可能裝置數目。

![][2]


<!-- Images -->
[1]: media/iot-hub-gateway-sdk-simulated-selector/image1.png
[2]: media/iot-hub-gateway-sdk-simulated-selector/image2.png

<!-- Links -->
[Simulated Device Cloud Upload sample]: https://github.com/Azure/azure-iot-gateway-sdk/blob/master/doc/sample_simulated_device_cloud_upload.md
[lnk-sdk]: https://github.com/Azure/azure-iot-gateway-sdk
[lnk-gw-getstarted]: ../articles/iot-hub/iot-hub-linux-gateway-sdk-get-started.md

