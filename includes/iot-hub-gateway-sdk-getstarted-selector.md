> [AZURE.SELECTOR]
- [Linux](../articles/iot-hub/iot-hub-linux-gateway-sdk-get-started.md)
- [Windows](../articles/iot-hub/iot-hub-windows-gateway-sdk-get-started.md)

本文提供 [Hello World 範例程式碼][lnk-helloworld-sample]的詳細逐步解說，來說明 [Azure IoT 閘道 SDK][lnk-gateway-sdk] 架構的基本元件。這個範例會使用閘道 SDK，建置每五秒即將 "hello world" 訊息記錄到檔案的簡單閘道。

本逐步解說涵蓋下列項目：

- **概念**︰概念式概觀，說明撰寫使用閘道 SDK 所建立之任何閘道的元件。
- **Hello World 範例架構**︰描述概念套用到 Hello World 範例的方式，以及元件如何彼此搭配運作。
- **如何建置範例**︰建置範例所需的步驟。
- **如何執行範例**︰執行範例所需的步驟。
- **典型輸出**：執行範例時所預期的輸出範例。
- **程式碼片段**︰程式碼片段集合，顯示 Hello World 範例如何實作重要閘道元件。

## 閘道 SDK 概念

檢查範例程式碼或使用閘道 SDK 建立專屬現場閘道之前，您應該了解可加強 SDK 架構的重要概念。

### 模組

建立和組合「模組」，即可使用 Azure IoT 閘道 SDK 來建置閘道。模組使用「訊息」來彼此交換資料。模組會接收訊息、對其執行某個動作、選擇性地將其轉換為新的訊息，然後發佈它，以供其他模組處理。某些模組可能只會產生新的訊息，且永遠不會處理內送訊息。一連串的模組可建立資料處理管線，而每個模組都會執行該管線中某個點的資料轉換。

![][1]
 
SDK 包含下列項目：

- 可執行常見閘道函式的預先撰寫模組。
- 開發人員可用來撰寫自訂模組的介面。
- 部署和執行一組模組所需的基礎結構。

SDK 提供一個抽象層，可讓您建置要在各種作業系統和平台上執行的閘道。

![][2]

### 訊息

雖然考量到彼此傳遞訊息的模組是概念化閘道運作方式的便利方式，但是它不會精確地反映所發生的事情。模組會使用訊息代理程式來彼此通訊，它們會將訊息發佈到訊息代理程式 (匯流排、發佈訂閱或任何其他傳訊模式)，然後讓訊息代理程式將訊息路由傳送到與其連接的模組。

模組使用 **Broker\_Publish** 函式將訊息發佈到訊息代理程式。訊息代理程式會叫用回呼函式，以將訊息傳遞到模組。訊息包含一組索引鍵/值屬性以及傳遞為記憶體區塊的內容。

![][3]

### 訊息路由和篩選

有兩種方式可將訊息導向到正確的模組。可將一組連結傳遞給訊息代理程式，讓訊息代理程式知道每個模組的來源和接收，或是模組可以根據訊息的屬性進行篩選。模組只應對其為適用對象的訊息採取動作。連結和訊息篩選可有效地建立訊息管線。

## Hello World 範例架構

Hello World 範例說明上節中所述的概念。Hello World 範例會實作管線由兩個模組所構成的閘道︰

-	「Hello World」模組每五秒會建立一則訊息，並將其傳遞到 Logger 模組。
-	「Logger」模組會將所收到的訊息寫入檔案。

![][4]

如上節所述，Hello World 模組不會每五秒將訊息直接傳遞到 Logger 模組。而是每五秒將訊息發佈到訊息代理程式。

Logger 模組會接收來自訊息代理程式的訊息並對其採取動作，以將訊息的內容寫入檔案。

Logger 模組只會取用來自訊息代理程式的訊息，永遠不會對訊息代理程式發佈新訊息。

![][5]

上圖顯示 Hello World 範例的架構，以及實作[儲存機制][lnk-gateway-sdk]中範例不同部分之來源檔案的相對路徑。自行探索程式碼，或使用下面的程式碼片段作為指南。

<!-- Images -->
[1]: media/iot-hub-gateway-sdk-getstarted-selector/modules.png
[2]: media/iot-hub-gateway-sdk-getstarted-selector/modules_2.png
[3]: media/iot-hub-gateway-sdk-getstarted-selector/messages_1.png
[4]: media/iot-hub-gateway-sdk-getstarted-selector/high_level_architecture.png
[5]: media/iot-hub-gateway-sdk-getstarted-selector/detailed_architecture.png

<!-- Links -->
[lnk-helloworld-sample]: https://github.com/Azure/azure-iot-gateway-sdk/tree/master/samples/hello_world
[lnk-gateway-sdk]: https://github.com/Azure/azure-iot-gateway-sdk

<!---HONumber=AcomDC_0928_2016-->