<properties
	pageTitle="針對 Azure 度量警示設定 Webhook | Microsoft Azure"
	description="重設 Azure 警示的路徑到其他非 Azure 系統。"
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

# 針對 Azure 度量警示設定 Webhook

Webhook 可讓您將 Azure 警示通知路由到其他系統以進行後處理或自訂動作。您可以針對警示使用 Webhook，以將警示路由到會傳送簡訊、記錄錯誤、透過聊天/傳訊服務通知小組，或進行任意數量之其他動作的服務。本文說明如何針對 Azure 度量警示設定 Webhook，並說明 HTTP POST 對 Webhook 之承載的樣貌。如需有關 Azure 活動記錄警示 (事件警示) 之設定和結構描述的資訊，[請改為參閱本頁](./insights-auditlog-to-webhook-email.md)。

Azure 警示會將警示內容以 JSON 格式 (定義如下的結構描述) HTTP POST 到您在建立警示時提供的 Webhook URI。此 URI 必須是有效的 HTTP 或 HTTPS 端點。當警示啟動時，Azure 會針對每個要求張貼一個項目。

## 透過入口網站設定 Webhook

您可以在[入口網站](https://portal.azure.com/)的 [建立/更新警示] 畫面中新增或更新 Webhook URI。

![新增警示規則](./media/insights-webhooks-alerts/Alertwebhook.png)

您也可以使用 [Azure PowerShell Cmdlet](./insights-powershell-samples.md#create-alert-rules)、[跨平台 CLI](./insights-cli-samples.md#work-with-alerts) 或 [Insights REST API](https://msdn.microsoft.com/library/azure/dn933805.aspx) 設定警示以張貼至 Webhook URI。

## 驗證 Webhook

Webhook 可使用下列其中一種方法來進行驗證︰

1. **權杖型授權** - 所儲存的 Webhook URI 具有權杖識別碼，例如 `https://mysamplealert/webcallback?tokenid=sometokenid&someparameter=somevalue`
2.	**基本授權** - 所儲存的 Webhook URI 具有使用者名稱和密碼，例如 `https://userid:password@mysamplealert/webcallback?someparamater=somevalue&foo=bar`

## 承載結構描述

POST 作業對於所有以計量為基礎的警示會包含下列 JSON 承載和結構描述。

```
{
"status": "Activated",
"context": {
            "timestamp": "2015-08-14T22:26:41.9975398Z",
            "id": "/subscriptions/s1/resourceGroups/useast/providers/microsoft.insights/alertrules/ruleName1",
            "name": "ruleName1",
            "description": "some description",
            "conditionType": "Metric",
            "condition": {
                        "metricName": "Requests",
                        "metricUnit": "Count",
                        "metricValue": "10",
                        "threshold": "10",
                        "windowSize": "15",
                        "timeAggregation": "Average",
                        "operator": "GreaterThanOrEqual"
                },
            "subscriptionId": "s1",
            "resourceGroupName": "useast",                                
            "resourceName": "mysite1",
            "resourceType": "microsoft.foo/sites",
            "resourceId": "/subscriptions/s1/resourceGroups/useast/providers/microsoft.foo/sites/mysite1",
            "resourceRegion": "centralus",
            "portalLink": "https://portal.azure.com/#resource/subscriptions/s1/resourceGroups/useast/providers/microsoft.foo/sites/mysite1"
},
"properties": {
              "key1": "value1",
              "key2": "value2"
              }
}
```


| 欄位 | 強制 | 一組固定值 | 注意事項 |
| :-------------| :-------------   | :-------------   | :-------------   |
|status|Y|“Activated”、“Resolved”|以您設定的條件為基礎的警示狀態。|
|context| Y | | 警示內容。|
|timestamp| Y | | 警示觸發的時間。|
|id | Y | | 每個警示規則都有唯一的識別碼。|
|名稱 |Y | | 警示名稱。|
|說明 |Y | |警示的描述。|
|conditionType |Y |“Metric”、“Event” |支援兩種類型的警示。一種根據度量條件，另一種根據活動記錄中的事件。使用此值來檢查警示是根據度量或事件。|
|condition |Y | | 根據 conditionType 所要檢查的特定欄位。|
|metricName |用於計量警示 | |定義規則所監視的計量名稱。|
|metricUnit |用於計量警示 |"Bytes"、"BytesPerSecond"、"Count"、"CountPerSecond"、"Percent"、"Seconds"|	 計量允許的單位。[允許的值在此列出](https://msdn.microsoft.com/library/microsoft.azure.insights.models.unit.aspx)。|
|metricValue |用於計量警示 | |造成警示的計量實際值。|
|threshold |用於計量警示 | |會啟動警示的臨界值。|
|windowSize |用於計量警示 | |根據 threshold 用來監視警示活動的時間長度。必須介於 5 分鐘到 1 天之間。ISO 8601 持續時間格式。|
|timeAggregation |用於計量警示 |"Average"、"Last"、"Maximum"、"Minimum"、"None"、"Total" |	收集的資料應如何隨著時間結合。預設值為 Average。[允許的值在此列出](https://msdn.microsoft.com/library/microsoft.azure.insights.models.aggregationtype.aspx)。|
|operator |用於計量警示 | |用來比較目前之度量資料與設定之臨界值的運算子。|
|subscriptionId |Y | |Azure 訂用帳戶識別碼。|
|resourceGroupName |Y | |受影響資源的資源群組的名稱。|
|resourceName |Y | |受影響資源的資源名稱。|
|resourceType |Y | |受影響資源的資源類型。|
|resourceId |Y | |受影響資源的資源識別碼。|
|resourceRegion |Y | |受影響資源的區域或位置。|
|portalLink |Y | |入口網站資源摘要頁面的直接連結。|
|屬性 |N |選用 |一組包含事件相關詳細資料的 `<Key, Value>` 配對 (也就是 `Dictionary<String, String>`)。properties 欄位是選擇性的。在自訂 UI 或邏輯應用程式的工作流程中，使用者可以輸入可透過承載傳遞的索引鍵/值。另一種將自訂屬性傳回給 Webhook 的替代方式是透過 Webhook URI 本身 (做為查詢參數)|


>[AZURE.NOTE] 屬性欄位只能使用 [Insights REST API](https://msdn.microsoft.com/library/azure/dn933805.aspx) 來設定。

## 後續步驟

- 請觀賞[使用 PagerDuty 整合 Azure 警示](http://go.microsoft.com/fwlink/?LinkId=627080)視訊，以深入了解 Azure 警示和 Webhook
- [對 Azure 警示執行 Azure 自動化指令碼 (Runbook)](http://go.microsoft.com/fwlink/?LinkId=627081)
- [使用邏輯應用程式透過 Twilio 從 Azure 警示傳送簡訊](https://github.com/Azure/azure-quickstart-templates/tree/master/201-alert-to-text-message-with-logic-app)
- [使用邏輯應用程式從 Azure 警示傳送 Slack 訊息](https://github.com/Azure/azure-quickstart-templates/tree/master/201-alert-to-slack-with-logic-app)
- [使用邏輯應用程式從 Azure 警示傳送訊息到 Azure 佇列](https://github.com/Azure/azure-quickstart-templates/tree/master/201-alert-to-queue-with-logic-app)

<!---HONumber=AcomDC_0921_2016-->