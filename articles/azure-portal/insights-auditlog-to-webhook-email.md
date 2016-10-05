<properties
	pageTitle="針對 Azure 活動記錄警示設定 Webhook | Microsoft Azure"
	description="了解如何使用活動記錄警示來呼叫 Webhook。"
	authors="kamathashwin"
	manager=""
	editor=""
	services="monitoring-and-diagnostics"
	documentationCenter="monitoring-and-diagnostics"/>

<tags
	ms.service="monitoring-and-diagnostics"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/15/2016"
	ms.author="ashwink"/>

# 針對 Azure 活動記錄警示設定 Webhook

Webhook 可讓您將 Azure 警示通知路由到其他系統以進行後處理或自訂動作。您可以針對警示使用 Webhook，以將警示路由到會傳送簡訊、記錄錯誤、透過聊天/傳訊服務通知小組，或進行任意數量之其他動作的服務。本文說明如何針對 Azure 活動記錄警示設定 Webhook，並說明 HTTP POST 對 Webhook 之承載的樣貌。如需有關 Azure 度量警示之設定和結構描述的資訊，[請改為參閱本頁](./insights-webhooks-alerts.md)。您也可以將活動記錄警示設定為在啟動時傳送電子郵件。

>[AZURE.NOTE] 這項功能目前僅供預覽，因此品質和效能會有變化。

您可以使用 [Azure PowerShell Cmdlet](./insights-powershell-samples.md#create-alert-rules)、[跨平台 CLI](./insights-cli-samples.md#work-with-alerts) 或 [Insights REST API](https://msdn.microsoft.com/library/azure/dn933805.aspx) 設定活動記錄警示。

## 驗證 Webhook
Webhook 可使用下列其中一種方法來進行驗證︰

1. **權杖型授權** - 所儲存的 Webhook URI 具有權杖識別碼，例如 `https://mysamplealert/webcallback?tokenid=sometokenid&someparameter=somevalue`
2.	**基本授權** - 所儲存的 Webhook URI 具有使用者名稱和密碼，例如 `https://userid:password@mysamplealert/webcallback?someparamater=somevalue&foo=bar`

## 承載結構描述
POST 作業對於所有以活動記錄為基礎的警示會包含下列 JSON 承載和結構描述。此結構描述類似於度量型警示所使用的結構描述。

```
{
        "status": "Activated",
        "context": {
                "resourceProviderName": "Microsoft.Web",
                "event": {
                        "$type": "Microsoft.WindowsAzure.Management.Monitoring.Automation.Notifications.GenericNotifications.Datacontracts.InstanceEventContext, Microsoft.WindowsAzure.Management.Mon.Automation",
                        "authorization": {
                                "action": "Microsoft.Web/sites/start/action",
                                "scope": "/subscriptions/s1/resourcegroups/rg1/providers/Microsoft.Web/sites/leoalerttest"
                        },
                        "eventDataId": "327caaca-08d7-41b1-86d8-27d0a7adb92d",
                        "category": "Administrative",
                        "caller": "myname@mycompany.com",
                        "httpRequest": {
                                "clientRequestId": "f58cead8-c9ed-43af-8710-55e64def208d",
                                "clientIpAddress": "104.43.166.155",
                                "method": "POST"
                        },
                        "status": "Succeeded",
                        "subStatus": "OK",
                        "level": "Informational",
                        "correlationId": "4a40beaa-6a63-4d92-85c4-923a25abb590",
                        "eventDescription": "",
                        "operationName": "Microsoft.Web/sites/start/action",
                        "operationId": "4a40beaa-6a63-4d92-85c4-923a25abb590",
                        "properties": {
                                "$type": "Microsoft.WindowsAzure.Management.Common.Storage.CasePreservedDictionary, Microsoft.WindowsAzure.Management.Common.Storage",
                                "statusCode": "OK",
                                "serviceRequestId": "f7716681-496a-4f5c-8d14-d564bcf54714"
                        }
                },
                "timestamp": "Friday, March 11, 2016 9:13:23 PM",
                "id": "/subscriptions/s1/resourceGroups/rg1/providers/microsoft.insights/alertrules/alertonevent2",
                "name": "alertonevent2",
                "description": "test alert on event start",
                "conditionType": "Event",
                "subscriptionId": "s1",
                "resourceId": "/subscriptions/s1/resourcegroups/rg1/providers/Microsoft.Web/sites/leoalerttest",
                "resourceGroupName": "rg1"
        },
        "properties": {
                "key1": "value1",
                "key2": "value2"
        }
}
```

|元素名稱|	說明|
|---|---|
|status |用於度量警示。針對活動記錄警示，一律設為「啟動」。|
|context|事件的內容。|
|resourceProviderName|受影響資源的資源提供者。|
|conditionType |一律為「事件」。|
|名稱 |警示規則的名稱。|
|id |警示的資源識別碼。|
|說明|	警示建立期間所設定的警示說明。|
|subscriptionId |Azure 訂用帳戶識別碼。|
|timestamp|	處理要求的 Azure 服務產生事件的時間。|
|resourceId |受影響資源的資源識別碼。|
|resourceGroupName|受影響資源的資源群組的名稱|
|屬性 |一組包含事件相關詳細資料的 `<Key, Value>` 配對 (也就是 `Dictionary<String, String>`)。|
|事件|包含事件相關中繼資料的元素。|
|授權|事件的 RBAC 屬性。這些屬性通常包括「動作」、「角色」和「範圍」。|
|category | 事件的類別。支援值包括︰管理、警示、安全性、ServiceHealth、建議。|
|呼叫者|已執行作業的使用者的電子郵件地址，根據可用性的 UPN 宣告或 SPN 宣告。特定系統呼叫可為 Null。|
|correlationId|	通常是字串格式的 GUID。具有屬於同一個較大動作的 correlationId 的事件，且通常會共用 correlationId。|
|eventDescription |事件的靜態文字描述。|
|eventDataId|事件的唯一識別碼。|
|eventSource |產生事件的 Azure 服務或基礎結構的名稱。|
|httpRequest|	通常包括 “clientRequestId”、“clientIpAddress” 和 “method” (HTTP 方法，例如 PUT)。|
|層級|下列其中一個值：「重大」、「錯誤」、「警告」、「資訊」和「詳細資訊」。|
|operationId|通常是對應至單一作業的事件之間共用的 GUID。|
|operationName|作業名稱。|
|屬性 |事件的屬性。|
|status|字串。作業的狀態。常見的值包括︰「已啟動」、「進行中」、「成功」、「失敗」、「使用中」、「已解決」。|
|子狀態|	通常包含對應 REST 呼叫的 HTTP 狀態碼。它也可以包含其他描述子狀態的字串。常見子狀態的值包括：確定 (HTTP 狀態碼︰200)，已建立 (HTTP 狀態碼︰201)、接受 (HTTP 狀態碼︰202)、沒有內容 (HTTP 狀態碼︰204)、不正確的要求 (HTTP 狀態碼︰400)、找不到 (HTTP 狀態碼︰404)，衝突 (HTTP 狀態碼︰409)、內部伺服器錯誤 (HTTP 狀態碼︰500)、服務無法使用 (HTTP 狀態碼︰503)、閘道逾時 (HTTP 狀態碼︰504)|

## 後續步驟
- [深入了解活動記錄](./monitoring-overview-activity-logs.md)
- [對 Azure 警示執行 Azure 自動化指令碼 (Runbook)](http://go.microsoft.com/fwlink/?LinkId=627081)
- [使用邏輯應用程式透過 Twilio 從 Azure 警示傳送簡訊](https://github.com/Azure/azure-quickstart-templates/tree/master/201-alert-to-text-message-with-logic-app)。此範例適用於度量警示，但經過修改後即可用於活動記錄警示。
- [使用邏輯應用程式從 Azure 警示傳送 Slack 訊息](https://github.com/Azure/azure-quickstart-templates/tree/master/201-alert-to-slack-with-logic-app)。此範例適用於度量警示，但經過修改後即可用於活動記錄警示。
- [使用邏輯應用程式從 Azure 警示傳送訊息到 Azure 佇列](https://github.com/Azure/azure-quickstart-templates/tree/master/201-alert-to-queue-with-logic-app)。此範例適用於度量警示，但經過修改後即可用於活動記錄警示。

<!---HONumber=AcomDC_0921_2016-->