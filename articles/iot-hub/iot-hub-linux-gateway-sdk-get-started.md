<properties
	pageTitle="開始使用 IoT 中樞閘道 SDK | Microsoft Azure"
	description="使用 Linux 的 Azure IoT 中樞閘道 SDK 逐步解說，說明您使用 Azure IoT 中樞閘道 SDK 時應該了解的重要概念。"
	services="iot-hub"
	documentationCenter=""
	authors="chipalost"
	manager="timlt"
	editor=""/>

<tags
     ms.service="iot-hub"
     ms.devlang="cpp"
     ms.topic="get-started-article"
     ms.tgt_pltfrm="na"
     ms.workload="na"
     ms.date="08/25/2016"
     ms.author="andbuc"/>


# IoT 閘道 SDK (Beta) - 開始使用 Linux

[AZURE.INCLUDE [iot-hub-gateway-sdk-getstarted-selector](../../includes/iot-hub-gateway-sdk-getstarted-selector.md)]

## 如何建置範例

開始之前，您必須[設定開發環境][lnk-setupdevbox]以在 Linux 上使用 SDK。

1. 開啟殼層。
2. 瀏覽至 **azure-iot-gateway-sdk** 儲存機制本機複本中的根資料夾。
3. 執行 **tools/build.sh** 指令碼。此指令碼使用 **cmake** 公用程式，在 **azure-iot-gateway-sdk** 儲存機制本機複本的根資料夾中建立稱為 **build** 的資料夾，以及產生 Makefile。此指令碼接著會建置方案，並執行測試。

> [AZURE.NOTE]  每次執行 **build.sh** 指令碼時，都會先在 **azure-iot-gateway-sdk** 儲存機制本機複本的根資料夾中刪除再重新建立 **build** 資料夾。

## 如何執行範例

1. **build.sh** 指令碼會在 **azure-iot-gateway-sdk** 儲存機制本機複本的 **build** 資料夾中產生其輸出。這包含在此範例中使用的兩個模組。

    build 指令碼會將 **liblogger\_hl.so** 放在 **build/modules/logger/** 資料夾中，並將 **libhello\_world\_hl.so** 放在 **build/modules/hello\_world/** 資料夾中。使用這些路徑作為「模組路徑」值 (如下面的 JSON 設定檔案中所示)。

2. **samples/hello\_world/src** 資料夾中的 **hello\_world\_lin.json** 檔案是適用於 Linux 且可用來執行範例的範例 JSON 設定檔案。下面顯示的範例 JSON 設定假設您將從 **azure-iot-gateway-sdk** 儲存機制本機複本的根資料夾執行範例。

3. 針對 **logger\_hl** 模組，在 **args** 區段中，將 **filename** 值設定為包含記錄資料的檔案名稱和路徑。

    這是適用於 Linux 且將 **log.txt** 寫入您執行範例之資料夾的 JSON 設定檔案範例。

    ```
    {
      "modules" :
      [ 
        {
          "module name" : "logger_hl",
          "module path" : "./build/modules/logger/liblogger_hl.so",
          "args" : 
          {
            "filename":"./log.txt"
          }
        },
        {
          "module name" : "hello_world",
          "module path" : "./build/modules/hello_world/libhello_world_hl.so",
          "args" : null
        }
      ],
      "links" :
      [
        {
          "source": "hello_world",
          "sink": "logger_hl"
        }
      ]
    }
    ```

3. 在您的殼層中，瀏覽至 **azure-iot-gateway-sdk** 資料夾。
4. 執行以下命令：
  
  ```
  ./build/samples/hello_world/hello_world_sample ./samples/hello_world/src/hello_world_lin.json
  ``` 

[AZURE.INCLUDE [iot-hub-gateway-sdk-getstarted-code](../../includes/iot-hub-gateway-sdk-getstarted-code.md)]

<!-- Links -->
[lnk-setupdevbox]: https://github.com/Azure/azure-iot-gateway-sdk/blob/master/doc/devbox_setup.md

<!---HONumber=AcomDC_0928_2016-->