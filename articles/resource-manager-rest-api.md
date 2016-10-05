<properties
   pageTitle="資源管理員 REST API | Microsoft Azure"
   description="資源管理員 REST API 驗證和使用方式範例的概觀"
   services="azure-resource-manager"
   documentationCenter="na"
   authors="navalev"
   manager=""
   editor=""/>

<tags
   ms.service="azure-resource-manager"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="06/23/2016"
   ms.author="navale;tomfitz;"/>
   
# 資源管理員 REST API

> [AZURE.SELECTOR]
- [Azure PowerShell](powershell-azure-resource-manager.md)
- [Azure CLI](xplat-cli-azure-resource-manager.md)
- [入口網站](./azure-portal/resource-group-portal.md)
- [REST API](resource-manager-rest-api.md)

在每次對 Azure Resource Manager 的呼叫之後、每個部署的範本之後、每個設定的儲存體帳戶之後，都有一或數次對 Azure Resource Manager 之 Rest API 的呼叫。本主題專門討論這些 API 以及如何在完全不使用任何 SDK 的情況下呼叫它們。如果您想要完全控制對 Azure 的所有要求，或如果您偏好語言的 SDK 無法使用或不支援您要執行的作業，這可能非常有用。

本文並不會逐一介紹 Azure 中公開的每個 API，而會以某些 API 為例，說明如何繼續進行並連線到這些 API。如果您了解基本概念，您即可繼續進行並閱讀 [Azure Resource Manager REST API 參考](https://msdn.microsoft.com/library/azure/dn790568.aspx)，以尋找如何使用其餘 API 的詳細資訊。

## 驗證
ARM 的驗證由 Azure Active Directory (AD) 處理。為了連接到任何 API，您必須先向 Azure AD 進行驗證，以接收可以傳遞給每個要求的驗證權杖。由於我們要說明直接對 REST API 的純呼叫，所以我們也會假設當快顯畫面提示您輸入使用者名稱和密碼時，或者甚至是在雙因素驗證案例中使用的其他驗證機制中，您不想使用一般使用者名稱和密碼進行驗證。因此，我們將建立所謂的 Azure AD 應用程式和服務主體，以便用來進行登入。但請記住，Azure AD 支援數個驗證程序，而這些程序全都可以用來擷取後續 API 要求所需的驗證權杖。請依照建立 [Azure AD 應用程式和服務主體](./resource-group-create-service-principal-portal.md)的逐步指示。

### 產生存取權杖 
向外呼叫位於 login.microsoftonline.com 的 Azure AD，集合完成對 Azure AD 驗證。若要驗證，您必須具有下列資訊︰

* Azure AD 租用戶識別碼 (您用來登入的 Azure AD 名稱，通常與您的公司名稱相關，但不一定如此)
* 應用程式識別碼 (在 Azure AD 應用程式建立步驟期間取得)
* 密碼 (在建立 Azure AD 應用程式時選取）

在下面的 HTTP 要求中，確定以正確的值取代 "Azure AD Tenant ID"、"Application ID" 和 "Password"。

**一般 HTTP 要求︰**

```HTTP
POST /<Azure AD Tenant ID>/oauth2/token?api-version=1.0 HTTP/1.1 HTTP/1.1
Host: login.microsoftonline.com
Cache-Control: no-cache
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&resource=https%3A%2F%2Fmanagement.core.windows.net%2F&client_id=<Application ID>&client_secret=<Password>
```

... 將 (如果驗證成功) 導致類似下面的回應：

```json
{
  "token_type": "Bearer",
  "expires_in": "3600",
  "expires_on": "1448199959",
  "not_before": "1448196059",
  "resource": "https://management.core.windows.net/",
  "access_token": "eyJ0eXAiOiJKV1QiLCJhb...86U3JI_0InPUk_lZqWvKiEWsayA"
}
```
(已縮短上面回應中的 access\_token，以提高可讀性)

**使用 Bash 產生存取權杖：**

```console
curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "grant_type=client_credentials&resource=https://management.core.windows.net&client_id=<application id>&client_secret=<password you selected for authentication>" https://login.microsoftonline.com/<Azure AD Tenant ID>/oauth2/token?api-version=1.0
```

**使用 PowerShell 產生存取權杖：**

```powershell
Invoke-RestMethod -Uri https://login.microsoftonline.com/<Azure AD Tenant ID>/oauth2/token?api-version=1.0 -Method Post
 -Body @{"grant_type" = "client_credentials"; "resource" = "https://management.core.windows.net/"; "client_id" = "<application id>"; "client_secret" = "<password you selected for authentication>" }
```

此回應包含存取權杖、該權杖有效期間的相關資訊，以及您可以將該權杖用於哪項資源的相關資訊。您在先前的 HTTP 呼叫中收到的存取權杖必須針對所有要求傳遞至 ARM API，做為名稱為 "Authorization" 且具有 "Bearer YOUR\_ACCESS\_TOKEN" 值的標頭。請注意 "Bearer" 與您的存取權杖之間的間距。

您可以從上面的 HTTP 結果看到，權杖會在特定一段時間內保持有效狀態，而您應在這段期間內快取並重複使用該相同權杖。即使可以對 Azure AD 進行每個 API 呼叫的驗證，但是會很沒效率。

## 呼叫 ARM REST API

[Azure Resource Manager REST API 記載於此](https://msdn.microsoft.com/library/azure/dn790568.aspx)，而詳細記載每個 API 的使用方式則超出本教學課程的範圍。本文件只會使用幾個 API 來說明 API 的基本使用方式，之後我們會指引您參閱正式文件。

### 列出所有訂用帳戶

您可以執行的最簡單作業之一，就是列出您可以存取的可用訂用帳戶。在以下要求中，您可以看見如何傳入存取權杖做為標頭。

(以您實際的存取權杖取代 YOUR\_ACCESS\_TOKEN。)

```HTTP
GET /subscriptions?api-version=2015-01-01 HTTP/1.1
Host: management.azure.com
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
```

...因此，您將收到此服務主體可以存取的訂用帳戶清單

(為了便於閱讀，以下的訂用帳戶識別碼已縮短)

```json
{
  "value": [
    {
      "id": "/subscriptions/3a8555...555995",
      "subscriptionId": "3a8555...555995",
      "displayName": "Azure Subscription",
      "state": "Enabled",
      "subscriptionPolicies": {
        "locationPlacementId": "Internal_2014-09-01",
        "quotaId": "Internal_2014-09-01"
      }
    }
  ]
}
```

### 列出特定訂用帳戶中的所有資源群組

適用於 ARM API 的所有資源都放在資源群組中。我們將使用下列 HTTP GET 要求，向 ARM 查詢我們的訂用帳戶中現有的資源群組。請注意，這次訂用帳戶識別碼是如何當作 URL 的一部分而傳入。

(以您實際的存取權杖和訂用帳戶識別碼取代 YOUR\_ACCESS\_TOKEN 和 SUBSCRIPTION\_ID。)

```HTTP
GET /subscriptions/SUBSCRIPTION_ID/resourcegroups?api-version=2015-01-01 HTTP/1.1
Host: management.azure.com
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
```

您收到的回應將取決於您是否已定義任何資源群組，若已定義，則會多少個。

(為了便於閱讀，以下的訂用帳戶識別碼已縮短)

```json
{
    "value": [
        {
            "id": "/subscriptions/3a8555...555995/resourceGroups/myfirstresourcegroup",
            "name": "myfirstresourcegroup",
            "location": "eastus",
            "properties": {
                "provisioningState": "Succeeded"
            }
        },
        {
            "id": "/subscriptions/3a8555...555995/resourceGroups/mysecondresourcegroup",
            "name": "mysecondresourcegroup",
            "location": "northeurope",
            "tags": {
                "tagname1": "My first tag"
            },
            "properties": {
                "provisioningState": "Succeeded"
            }
        }
    ]
}
```

### 建立資源群組

到目前為止，我們只向 ARM API 查詢了資訊，我們現在可以開始建立一些資源，讓我們從最簡單的資源群組著手。下列 HTTP 要求會在您選擇的區域/位置建立新的資源群組並在其中新增一或多個標記 (下列範例實際上只新增一個標記）。

(以您實際的存取權杖、訂用帳戶識別碼和您要建立的資源群組名稱取代 YOUR\_ACCESS\_TOKEN、SUBSCRIPTION\_ID、RESOURCE\_GROUP\_NAME)

```HTTP
PUT /subscriptions/SUBSCRIPTION_ID/resourcegroups/RESOURCE_GROUP_NAME?api-version=2015-01-01 HTTP/1.1
Host: management.azure.com
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json

{
  "location": "northeurope",
  "tags": {
    "tagname1": "test-tag"
  }
}
```

如果成功，您會得到如下的回應

```json
{
  "id": "/subscriptions/3a8555...555995/resourceGroups/RESOURCE_GROUP_NAME",
  "name": "RESOURCE_GROUP_NAME",
  "location": "northeurope",
  "tags": {
    "tagname1": "test-tag"
  },
  "properties": {
    "provisioningState": "Succeeded"
  }
}
```

您已在 Azure 中成功建立資源群組。恭喜！

### 使用 ARM 範本將資源部署到資源群組

使用 ARM，您可以使用 ARM 範本部署您的資源。ARM 範本會定義數個資源與其相依性。在本節中，我們只是假設您熟悉 ARM 範本，我們只會示範如何進行 API 呼叫以便開始部署資源。在這裡可以找到 ARM 範本的詳細文件。

ARM 範本的部署與您呼叫其他 API 的方式沒有太大差異。其中一個重要層面就是範本部署需要相當長的時間 (視此範本的內容而定)，而 API 呼叫只會返回並由身為開發人員的您決定查詢部署狀態，以便了解何時完成部署。

在此範例中，我們將使用 [GitHub](https://github.com/Azure/azure-quickstart-templates) 上提供的公開 ARM 範本。我們即將使用的範本會將 Linux VM 部署到美國西部區域。即使此範本會在公開儲存機制 (如 GitHub) 中提供，您也可以選擇傳遞整個範本做為要求的一部分。請注意，我們將提供參數值做為要求的一部分，而這些參數值將使用於所用的範本內。

(將 SUBSCRIPTION\_ID、RESOURCE\_GROUP\_NAME、DEPLOYMENT\_NAME、YOUR\_ACCESS\_TOKEN、GLOBALY\_UNIQUE\_STORAGE\_ACCOUNT\_NAME、ADMIN\_USER\_NAME、ADMIN\_PASSWORD 和 DNS\_NAME\_FOR\_PUBLIC\_IP 取代為您的要求適用的值)

```HTTP
PUT /subscriptions/SUBSCRIPTION_ID/resourcegroups/RESOURCE_GROUP_NAME/providers/microsoft.resources/deployments/DEPLOYMENT_NAME?api-version=2015-01-01 HTTP/1.1
Host: management.azure.com
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json

{
  "properties": {
    "templateLink": {
      "uri": "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-simple-linux-vm/azuredeploy.json",
      "contentVersion": "1.0.0.0",
    },
    "mode": "Incremental",
    "parameters": {
        "newStorageAccountName": {
          "value": "GLOBALY_UNIQUE_STORAGE_ACCOUNT_NAME"
        },
        "adminUsername": {
          "value": "ADMIN_USER_NAME"
        },
        "adminPassword": {
          "value": "ADMIN_PASSWORD"
        },
        "dnsNameForPublicIP": {
          "value": "DNS_NAME_FOR_PUBLIC_IP"
        },
        "ubuntuOSVersion": {
          "value": "15.04"
        }
    }
  }
}
```

省略了此要求的較長 JSON 回應，以便改善這份文件的可讀性。回應將會包含您剛建立的樣板化部署相關資訊。

<!---HONumber=AcomDC_0921_2016-->