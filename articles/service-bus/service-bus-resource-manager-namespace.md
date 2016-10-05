<properties
    pageTitle="使用 Resource Manager 範本建立服務匯流排命名空間 | Microsoft Azure"
    description="使用 Azure Resource Manager 範本來建立服務匯流排命名空間"
    services="service-bus"
    documentationCenter=".net"
    authors="sethmanheim"
    manager="timlt"
    editor=""/>

<tags
    ms.service="service-bus"
    ms.devlang="tbd"
    ms.topic="article"
    ms.tgt_pltfrm="dotnet"
    ms.workload="na"
    ms.date="04/15/2016"
    ms.author="sethm;shvija"/>

# 使用 Azure Resource Manager 範本來建立服務匯流排命名空間

本文說明如何使用 Azure Resource Manager 範本，建立**訊息**類型的服務匯流排命名空間與標準/基本 SKU。本文也會定義針對部署執行而指定的參數。您可以直接在自己的部署中使用此範本，或自訂此範本以符合您的需求。

如需建立範本的詳細資訊，請參閱[編寫 Azure Resource Manager 範本][]。

如需完整的範本，請參閱 GitHub 上的[服務匯流排命名空間範本][]。

>[AZURE.NOTE] 下列 Azure Resource Manager 範本可供下載和部署。
>
>-    [建立事件中樞命名空間與事件中樞和取用者群組](../event-hubs/event-hubs-resource-manager-namespace-event-hub.md)
>-    [建立服務匯流排命名空間與佇列](service-bus-resource-manager-namespace-queue.md)
>-    [建立服務匯流排命名空間與主題和訂用帳戶](service-bus-resource-manager-namespace-topic.md)
>-    [建立服務匯流排命名空間與佇列和授權規則](service-bus-resource-manager-namespace-auth-rule.md)
>
>若要檢查最新的範本，請造訪 [Azure 快速入門範本][]資源庫並搜尋服務匯流排。

## 您將部署什麼？

使用此範本時，您將部署具有[基本、標準或進階](https://azure.microsoft.com/pricing/details/service-bus/) SKU 的「服務匯流排」命名空間。

若要自動執行部署，請按一下下列按鈕：

[![部署至 Azure](./media/service-bus-resource-manager-namespace/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-servicebus-create-namespace%2Fazuredeploy.json)

## 參數

透過 Azure 資源管理員，您可以定義在部署範本時想要指定之值的參數。此範本有一個 `Parameters` 區段，內含所有參數值。您應該為會隨著要部署的專案或要部署到的環境而變化的值定義參數。請不要為永遠保持不變的值定義參數。每個參數值都可在範本中用來定義所部署的資源。

範本會定義下列參數。

### serviceBusNamespaceName

要建立的服務匯流排命名空間名稱。

```
"serviceBusNamespaceName": {
"type": "string",
"metadata": { 
    "description": "Name of the Service Bus namespace" 
    }
}
```

### serviceBusSKU

要建立的服務匯流排 [SKU](https://azure.microsoft.com/pricing/details/service-bus/) 名稱。

```
"serviceBusSku": { 
    "type": "string", 
    "allowedValues": [ 
        "Basic", 
        "Standard",
        "Premium" 
    ], 
    "defaultValue": "Standard", 
    "metadata": { 
        "description": "The messaging tier for service Bus namespace" 
    } 

```

此範本會定義此參數允許使用的值 (Basic、Standard 或 Premium)，並指派未指定任何值時的預設值 (Standard)。

如需服務匯流排價格的詳細資訊，請參閱[服務匯流排價格與計費][]。

### serviceBusApiVersion

範本的服務匯流排 API 版本。

```
"serviceBusApiVersion": { 
       "type": "string", 
       "defaultValue": "2015-08-01", 
       "metadata": { 
           "description": "Service Bus ApiVersion used by the template" 
       } 
```

## 要部署的資源

### 服務匯流排命名空間

建立**訊息**類型的標準服務匯流排命名空間。

```
"resources": [
    {
        "apiVersion": "[parameters('serviceBusApiVersion')]",
        "name": "[parameters('serviceBusNamespaceName')]",
        "type": "Microsoft.ServiceBus/Namespaces",
        "location": "[variables('location')]",
        "kind": "Messaging",
        "sku": {
            "name": "StandardSku",
            "tier": "Standard"
        },
        "properties": {
        }
    }
]
```

## 執行部署的命令

[AZURE.INCLUDE [app-service-deploy-commands](../../includes/app-service-deploy-commands.md)]

### PowerShell

```
New-AzureRmResourceGroupDeployment -ResourceGroupName <resource-group-name> -TemplateFile https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/101-servicebus-create-namespace/azuredeploy.json
```

### Azure CLI

```
azure config mode arm

azure group deployment create <my-resource-group> <my-deployment-name> --template-uri https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/101-servicebus-create-namespace/azuredeploy.json
```

## 後續步驟

現在您已使用 Azure Resource Manager 建立並部署資源，請檢視這些文件，了解如何管理這些資源︰

- [使用 Azure 自動化管理 Azure 服務匯流排](service-bus-automation-manage.md)
- [使用 PowerShell 管理服務匯流排](service-bus-powershell-how-to-provision.md)
- [使用服務匯流排總管管理服務匯流排資源](https://code.msdn.microsoft.com/Service-Bus-Explorer-f2abca5a)

  [編寫 Azure Resource Manager 範本]: ../resource-group-authoring-templates.md
  [服務匯流排命名空間範本]: https://github.com/Azure/azure-quickstart-templates/blob/master/101-servicebus-create-namespace/
  [Azure 快速入門範本]: https://azure.microsoft.com/documentation/templates/?term=service+bus
  [服務匯流排價格與計費]: https://azure.microsoft.com/documentation/articles/service-bus-pricing-billing/
  [Using Azure PowerShell with Azure Resource Manager]: ../powershell-azure-resource-manager.md
  [Using the Azure CLI for Mac, Linux, and Windows with Azure Resource Management]: ../xplat-cli-azure-resource-manager.md

<!---HONumber=AcomDC_0921_2016-->