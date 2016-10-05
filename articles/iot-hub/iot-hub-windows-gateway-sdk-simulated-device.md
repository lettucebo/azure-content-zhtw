<properties
	pageTitle="使用閘道 SDK 模擬裝置 | Microsoft Azure"
	description="使用 Windows 的 Azure IoT 中樞閘道 SDK 逐步解說，說明如何使用 Azure IoT 中樞閘道 SDK 從模擬裝置傳送遙測。"
	services="iot-hub"
	documentationCenter=""
	authors="chipalost"
	manager="timlt"
	editor=""/>

<tags
     ms.service="iot-hub"
     ms.devlang="cpp"
     ms.topic="article"
     ms.tgt_pltfrm="na"
     ms.workload="na"
     ms.date="04/20/2016"
     ms.author="cstreet"/>


# IoT 閘道 SDK (Beta) – 搭配使用模擬裝置與 Windows 來傳送裝置到雲端訊息

[AZURE.INCLUDE [iot-hub-gateway-sdk-simulated-selector](../../includes/iot-hub-gateway-sdk-simulated-selector.md)]

## 建置並執行範例

開始之前，您必須：

- [設定開發環境][lnk-setupdevbox]以在 Windows 上使用 SDK。
- [建立 IoT 中樞][lnk-create-hub] \(於 Azure 訂用帳戶中) 時，您將需要中樞名稱才能完成此逐步解說。如果您還沒有 Azure 訂用帳戶，則可以取得[免費帳戶][lnk-free-trial]。
- 將兩個裝置加入 IoT 中樞中，並記下其識別碼和裝置金鑰。您可以使用[裝置總管或 iothub-explorer][lnk-explorer-tools] 工具將您的裝置加入您在上一個步驟中建立的 IoT 中樞，並擷取其金鑰。

建置範例：

1. 開啟 [VS2015 的開發人員命令提示字元] 命令提示字元。
2. 瀏覽至 **azure-iot-gateway-sdk** 儲存機制本機複本中的根資料夾。
3. 執行 **tools\\build.cmd** 指令碼。此指令碼會建立 Visual Studio 方案檔、建置方案，並執行測試。您可以在 **azure-iot-gateway-sdk** 儲存機制本機複本的 **build** 資料夾中尋找 Visual Studio 方案。

執行範例：

在文字編輯器中，開啟 **azure-iot-gateway-sdk** 儲存機制本機複本中的 **samples\\simulated\_device\_cloud\_upload\\src\\simulated\_device\_cloud\_upload\_win.json** 檔案。這個檔案設定範例閘道中的模組︰

- **IoTHub** 模組會連接至您的 IoT 中樞。您必須設定它，以將資料傳送至您的 IoT 中樞。特別的是，將 **IoTHubName** 值設定為 IoT 中樞的名稱，並將 **IoTHubSuffix** 的值設定為 **azure-devices.net**。
- **mapping** 模組會將您模擬裝置的 MAC 位址對應到您 IoT 中樞裝置識別碼。請確定 **deviceId** 值符合您加入 IoT 中樞之兩個裝置的識別碼，而且 **deviceKey** 值包含兩個裝置的金鑰。
- **BLE1** 和 **BLE2** 模組是模擬裝置。請注意，其 MAC 位址如何符合 **mapping** 模組中的 MAC 位址。
- **Logger** 模組會將閘道活動記錄到檔案。
- 下面顯示的「模組路徑」值假設您已將閘道 SDK 儲存機制複製到 **C:** 磁碟機的根目錄。如果您已將它下載到另一個位置，則需要據此調整「模組路徑」值。

```
{
    "modules" :
    [ 
        {
            "module name" : "IoTHub",
            "module path" : "C:\\azure-iot-gateway-sdk\\modules\\iothubhttp\\Debug\\iothubhttp_hl.dll",
            "args" : 
            {
                "IoTHubName" : "{Your IoT hub name}",
                "IoTHubSuffix" : "azure-devices.net"
            }
        },
        {
            "module name" : "mapping",
            "module path" : "C:\\azure-iot-gateway-sdk\\modules\\identitymap\\Debug\\identitymap_hl.dll",
            "args" : 
            [
                {
                    "macAddress" : "01-01-01-01-01-01",
                    "deviceId"   : "GW-ble1-demo",
                    "deviceKey"  : "{Device key}"
                },
                {
                    "macAddress" : "02-02-02-02-02-02",
                    "deviceId"   : "GW-ble2-demo",
                    "deviceKey"  : "{Device key}"
                }
            ]
        },
        {
            "module name":"BLE1",
            "module path" : "C:\\azure-iot-gateway-sdk\\modules\\ble_fake\\Debug\\ble_fake_hl.dll",
            "args":
            {
                "macAddress" : "01-01-01-01-01-01"
            }
        },
        {
            "module name":"BLE2",
            "module path" : "C:\\azure-iot-gateway-sdk\\modules\\ble_fake\\Debug\\ble_fake_hl.dll",
            "args":
            {
                "macAddress" : "02-02-02-02-02-02"
            }
        },
        {
            "module name":"Logger",
            "module path" : "C:\\azure-iot-gateway-sdk\\modules\\logger\\Debug\\logger_hl.dll",
            "args":
            {
                "filename":"C:\\azure-iot-gateway-sdk\\deviceCloudUploadGatewaylog.log"
            }
        }
    ]
}
```

儲存您對組態檔進行的任何變更。

執行範例：

1. 在命令提示字元中，瀏覽至 **azure-iot-gateway-sdk** 儲存機制本機複本中的根資料夾。
2. 執行以下命令：
  
    ```
    build\samples\simulated_device_cloud_upload\Debug\simulated_device_cloud_upload_sample.exe samples\simulated_device_cloud_upload\src\simulated_device_cloud_upload_win.json
    ```

3. 您可以使用[裝置總管或 iothub-explorer][lnk-explorer-tools] 工具監視 IoT 中樞接收自閘道的訊息。


## 後續步驟

如果您想要更進一步了解閘道 SDK 並實驗一些程式碼範例，請瀏覽下列開發人員教學課程和資源：

- [透過閘道 SDK 從實際裝置傳送裝置到雲端訊息][lnk-physical-device]
- [管理閘道裝置][lnk-manage-devices]
- [Azure IoT 閘道器 SDK][lnk-gateway-sdk]

若要進一步探索 IoT 中樞的功能，請參閱︰

- [設計您的解決方案][lnk-design]
- [開發人員指南][lnk-devguide]
- [使用範例 UI 探索裝置管理][lnk-dmui]
- [使用 Azure 入口網站管理 IoT 中樞][lnk-portal]

<!-- Links -->
[lnk-setupdevbox]: https://github.com/Azure/azure-iot-gateway-sdk/blob/master/doc/devbox_setup.md
[lnk-create-hub]: iot-hub-manage-through-portal.md
[lnk-free-trial]: https://azure.microsoft.com/pricing/free-trial/
[lnk-explorer-tools]: https://github.com/Azure/azure-iot-sdks/blob/master/doc/manage_iot_hub.md
[lnk-gateway-sdk]: https://github.com/Azure/azure-iot-gateway-sdk/

[lnk-physical-device]: iot-hub-gateway-sdk-physical-device.md
[lnk-manage-devices]: iot-hub-gateway-sdk-device-management.md

[lnk-design]: iot-hub-guidance.md
[lnk-devguide]: iot-hub-devguide.md
[lnk-dmui]: iot-hub-device-management-ui-sample.md
[lnk-portal]: iot-hub-manage-through-portal.md

<!---HONumber=AcomDC_0713_2016-->