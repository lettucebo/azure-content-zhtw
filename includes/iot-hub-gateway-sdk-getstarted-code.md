## 典型輸出

下列範例是 Hello World 範例寫入記錄檔中的輸出。已加入新行和 Tab 字元以利閱讀︰

```
[{
	"time": "Mon Apr 11 13:48:07 2016",
	"content": "Log started"
}, {
	"time": "Mon Apr 11 13:48:48 2016",
	"properties": {
		"helloWorld": "from Azure IoT Gateway SDK simple sample!"
	},
	"content": "aGVsbG8gd29ybGQ="
}, {
	"time": "Mon Apr 11 13:48:55 2016",
	"properties": {
		"helloWorld": "from Azure IoT Gateway SDK simple sample!"
	},
	"content": "aGVsbG8gd29ybGQ="
}, {
	"time": "Mon Apr 11 13:49:01 2016",
	"properties": {
		"helloWorld": "from Azure IoT Gateway SDK simple sample!"
	},
	"content": "aGVsbG8gd29ybGQ="
}, {
	"time": "Mon Apr 11 13:49:04 2016",
	"content": "Log stopped"
}]
```

## 程式碼片段

本節探討 Hello World 範例中程式碼的一些重要部分。

### 建立閘道

開發人員必須撰寫「閘道程序」。此程式會建立內部基礎結構 (訊息代理程式)、載入模組，以及設定所有項目才能正確運作。SDK 提供 **Gateway\_Create\_From\_JSON** 函式可讓您從 JSON 檔案啟動閘道。若要使用 **Gateway\_Create\_From\_JSON** 函式，您必須將它傳遞到 JSON 檔案的路徑，而 JSON 檔案指定要載入的模組。

在 [main.c][lnk-main-c] 檔案中，您可以找到 Hello World 範例中閘道程序的程式碼。下列程式碼片段顯示精簡版本的閘道程序程式碼，以利閱讀。此程式會建立閘道，然後先等待使用者按下 **ENTER** 鍵，再終止閘道。

```
int main(int argc, char** argv)
{
    GATEWAY_HANDLE gateway;
    if ((gateway = Gateway_Create_From_JSON(argv[1])) == NULL)
    {
        printf("failed to create the gateway from JSON\n");
    }
    else
    {
        printf("gateway successfully created from JSON\n");
        printf("gateway shall run until ENTER is pressed\n");
        (void)getchar();
        Gateway_LL_Destroy(gateway);
    }
	return 0;
}
```

JSON 設定檔案包含要載入之模組的清單。每個模組都必須指定：

- **module\_name**：模組的唯一名稱。
- **module\_path**︰包含模組之程式庫的路徑。在 Linux 上，這是 .so 檔案，在 Windows 上，這是 .dll 檔案。
- **args**：模組所需的任何組態資訊。

JSON 檔案也包含將會傳遞給訊息代理程式之模組之間的連結。連結有兩個屬性︰
- **來源**︰來自 `modules` 區段的模組名稱，或 "*"。
- **接收**︰來自 `modules` 區段的模組名稱

每個連結都會定義訊息路由和方向。來自模組 `source` 的訊息會傳遞給模組 `sink`。`source` 可能會設定為 "*"，代表來自任何模組的訊息都會由 `sink` 接收。

下列範例示範用來在 Linux 上設定 Hello World 範例的 JSON 設定檔案。模組 `hello_world` 所產生的每個訊息都會由模組 `logger` 取用。模組是否需要引數取決於模組的設計。在此範例中，Logger 模組所採用的引數是輸出檔的路徑，而 Hello World 模組不採用任何引數︰

```
{
    "modules" :
    [ 
        {
            "module name" : "logger",
            "module path" : "./modules/logger/liblogger_hl.so",
            "args" : {"filename":"log.txt"}
        },
        {
            "module name" : "hello_world",
            "module path" : "./modules/hello_world/libhello_world_hl.so",
			"args" : null
        }
    ],
    "links" :
    [
        {
            "source" : "hello_world",
            "sink" : "logger"
        }
    ]
}
```

### Hello World 模組訊息發佈

您可以在 ['hello\_world.c'][lnk-helloworld-c] 檔案中找到 "hello world" 模組所使用的程式碼來發佈訊息。下列程式碼片段所示範的修改過版本已移除其他註解和一些錯誤處理程式碼，以利閱讀︰

```
int helloWorldThread(void *param)
{
    // create data structures used in function.
    HELLOWORLD_HANDLE_DATA* handleData = param;
    MESSAGE_CONFIG msgConfig;
    MAP_HANDLE propertiesMap = Map_Create(NULL);
    
    // add a property named "helloWorld" with a value of "from Azure IoT
    // Gateway SDK simple sample!" to a set of message properties that
    // will be appended to the message before publishing it. 
    Map_AddOrUpdate(propertiesMap, "helloWorld", "from Azure IoT Gateway SDK simple sample!")

    // set the content for the message
    msgConfig.size = strlen(HELLOWORLD_MESSAGE);
    msgConfig.source = HELLOWORLD_MESSAGE;

    // set the properties for the message
    msgConfig.sourceProperties = propertiesMap;
    
    // create a message based on the msgConfig structure
    MESSAGE_HANDLE helloWorldMessage = Message_Create(&msgConfig);

    while (1)
    {
        if (handleData->stopThread)
        {
            (void)Unlock(handleData->lockHandle);
            break; /*gets out of the thread*/
        }
        else
        {
            // publish the message to the broker
            (void)Broker_Publish(handleData->brokerHandle, helloWorldMessage);
            (void)Unlock(handleData->lockHandle);
        }

        (void)ThreadAPI_Sleep(5000); /*every 5 seconds*/
    }

    Message_Destroy(helloWorldMessage);

    return 0;
}
```

### Hello World 模組訊息處理

Hello World 模組永遠不需要處理其他模組發佈至訊息代理程式的任何訊息。這樣會將 Hello World 模組中的訊息回呼實作設為無作業函式。

```
static void HelloWorld_Receive(MODULE_HANDLE moduleHandle, MESSAGE_HANDLE messageHandle)
{
    /* No action, HelloWorld is not interested in any messages. */
}
```

### Logger 模組訊息發佈和處理

Logger 模組會接收來自訊息代理程式的訊息，並將它們寫入檔案。此模組永遠不會發佈任何訊息。因此，Logger 模組的程式碼永遠不會呼叫 **Broker\_Publish** 函式。

[logger.c][lnk-logger-c] 檔案中的 **Logger\_Recieve** 函式是訊息代理程式所叫用的回呼，以將訊息傳遞到 Logger 模組。下列程式碼片段所示範的修改過版本已移除其他註解和一些錯誤處理程式碼，以利閱讀︰

```
static void Logger_Receive(MODULE_HANDLE moduleHandle, MESSAGE_HANDLE messageHandle)
{

    time_t temp = time(NULL);
    struct tm* t = localtime(&temp);
    char timetemp[80] = { 0 };

    // Get the message properties from the message
    CONSTMAP_HANDLE originalProperties = Message_GetProperties(messageHandle); 
    MAP_HANDLE propertiesAsMap = ConstMap_CloneWriteable(originalProperties);

    // Convert the collection of properties into a JSON string
    STRING_HANDLE jsonProperties = Map_ToJSON(propertiesAsMap);

    //  base64 encode the message content
    const CONSTBUFFER * content = Message_GetContent(messageHandle);
    STRING_HANDLE contentAsJSON = Base64_Encode_Bytes(content->buffer, content->size);

    // Start the construction of the final string to be logged by adding
    // the timestamp
    STRING_HANDLE jsonToBeAppended = STRING_construct(",{"time":"");
    STRING_concat(jsonToBeAppended, timetemp);

    // Add the message properties
    STRING_concat(jsonToBeAppended, "","properties":"); 
    STRING_concat_with_STRING(jsonToBeAppended, jsonProperties);

    // Add the content
    STRING_concat(jsonToBeAppended, ","content":"");
    STRING_concat_with_STRING(jsonToBeAppended, contentAsJSON);
    STRING_concat(jsonToBeAppended, ""}]");

    // Write the formatted string
    LOGGER_HANDLE_DATA *handleData = (LOGGER_HANDLE_DATA *)moduleHandle;
    addJSONString(handleData->fout, STRING_c_str(jsonToBeAppended);
}
```

## 後續步驟

若要了解如何使用閘道 SDK，請參閱下列項目︰

- [IoT 閘道 SDK – 搭配使用模擬裝置與 Linux 來傳送裝置到雲端訊息][lnk-gateway-simulated]。
- GitHub 上的 [Azure IoT 閘道 SDK][lnk-gateway-sdk]。

<!-- Links -->
[lnk-main-c]: https://github.com/Azure/azure-iot-gateway-sdk/blob/master/samples/hello_world/src/main.c
[lnk-helloworld-c]: https://github.com/Azure/azure-iot-gateway-sdk/blob/master/modules/hello_world/src/hello_world.c
[lnk-logger-c]: https://github.com/Azure/azure-iot-gateway-sdk/blob/master/modules/logger/src/logger.c
[lnk-gateway-sdk]: https://github.com/Azure/azure-iot-gateway-sdk/
[lnk-gateway-simulated]: ../articles/iot-hub/iot-hub-linux-gateway-sdk-simulated-device.md

<!---HONumber=AcomDC_0928_2016-->