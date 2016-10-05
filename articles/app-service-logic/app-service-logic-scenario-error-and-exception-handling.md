<properties
    pageTitle="Logic Apps 中的記錄和錯誤處理 | Microsoft Azure"
    description="檢視 Logic Apps 之進階錯誤處理和記錄的現實生活使用案例"
    keywords=""
    services="logic-apps"
    authors="hedidin"
    manager=""
    editor=""
    documentationCenter=""/>

<tags
    ms.service="logic-apps"
    ms.workload="na"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="07/29/2016"
    ms.author="b-hoedid"/>

# Logic Apps 中的記錄和錯誤處理

本文說明如何擴充邏輯應用程式以提升對於例外狀況處理的支援。文中的內容取材自現實生活的使用案例，而這也是我們對於「Logic Apps 是否支援例外狀況和錯誤處理？」此一問題的回答。

>[AZURE.NOTE] Microsoft Azure App Service 目前的 Logic Apps 功能版本提供標準的動作回應範本。這包括內部驗證和 API 應用程式所傳回的錯誤回應。

## 使用案例和情節的概觀

下列案例是這篇文章的使用案例。知名的醫療保健組織找到了我們，他們想要開發 Azure 解決方案，以使用 Microsoft Dynamics CRM Online 建立病患入口網站。他們需要在 Dynamics CRM Online 病患入口網站和 Salesforce 之間傳送預約記錄。因此要求我們對所有病患記錄使用 [HL7 FHIR](http://www.hl7.org/implement/standards/fhir/) 標準。

此專案有兩大需求︰

 -  用來記錄從 Dynamics CRM Online 入口網站傳送過來之記錄的方法
 -  用來檢視工作流程中所發生之任何錯誤的方法


## 問題解決方式

>[AZURE.TIP] 您可以在[整合使用者群組](http://www.integrationusergroup.com/do-logic-apps-support-error-handling/ "Integration User Group")觀賞此專案的高階影片。

我們選擇以 [Azure DocumentDB](https://azure.microsoft.com/services/documentdb/ "Azure DocumentDB") 做為記錄檔和錯誤記錄的儲存機制 (DocumentDB 會將記錄當做文件)。Logic Apps 具有適用於所有回應的標準範本，因此我們不需要建立自訂結構描述。我們可以建立 API 應用程式來**插入**及**查詢**錯誤和記錄檔記錄。我們也可以為 API 應用程式中的每個項目定義結構描述。

另一個需求是要在特定日期之後清除記錄。DocumentDB 具有稱為[存留時間](https://azure.microsoft.com/blog/documentdb-now-supports-time-to-live-ttl/ "存留時間") (TTL) 的屬性，這可讓我們設定每一筆記錄或每一個集合的**存留時間**值。因此，我們不需要手動在 DocumentDB 中刪除記錄。

### 建立邏輯應用程式

第一個步驟是建立邏輯應用程式，並在設計工具中予以載入。在此範例中，我們會使用父子邏輯應用程式。假設我們已建立父項，而且將要建立一個子邏輯應用程式。

我們將會記錄來自 Dynamics CRM Online 的記錄，所以讓我們從最上層開始。我們需要使用要求觸發程序，因為父邏輯應用程式將會觸發這個子項。

> [AZURE.IMPORTANT] 為了完成本教學課程，您必須建立 DocumentDB 資料庫和兩個集合 (記錄和錯誤)。

### 邏輯應用程式觸發程序

我們將會使用如下列範例所示的要求觸發程序。

```` json
"triggers": {
        "request": {
          "type": "request",
          "kind": "http",
          "inputs": {
            "schema": {
              "properties": {
                "CRMid": {
                  "type": "string"
                },
                "recordType": {
                  "type": "string"
                },
                "salesforceID": {
                  "type": "string"
                },
                "update": {
                  "type": "boolean"
                }
              },
              "required": [
                "CRMid",
                "recordType",
                "salesforceID",
                "update"
              ],
              "type": "object"
            }
          }
        }
      },

````


### 步驟

我們必須記錄來自 Dynamics CRM Online 入口網站的病患記錄來源 (要求)。

1. 我們必須從 Dynamics CRM Online 取得新的預約記錄。來自 CRM 的觸發程序會提供我們 **CRM PatentId**、**記錄類型**、**新的或更新的記錄** (新增或更新布林值) 和 **SalesforceId**。**SalesforceId** 可以是 null，因為它只會用於更新。我們會使用 CRM **PatientID** 和**記錄類型**來取得 CRM 記錄。
1. 接下來，我們必須新增 DocumentDB API 應用程式 **InsertLogEntry** 作業，如下圖所示。


#### 插入記錄檔項目設計工具檢視

![插入記錄檔項目](./media/app-service-logic-scenario-error-and-exception-handling/lognewpatient.png)

#### 插入錯誤項目設計工具檢視
![插入記錄檔項目](./media/app-service-logic-scenario-error-and-exception-handling/insertlogentry.png)

#### 檢查建立記錄失敗

![條件](./media/app-service-logic-scenario-error-and-exception-handling/condition.png)


## 邏輯應用程式原始程式碼

>[AZURE.NOTE]  以下僅是範例。由於此教學課程是以目前在生產環境中的實作為基礎，**來源節點**的值可能不會顯示與安排預約相關的屬性。

### 記錄
下列邏輯應用程式的程式碼範例示範如何處理記錄。

#### 記錄檔項目
這是用來插入記錄檔項目的邏輯應用程式原始程式碼。

``` json
"InsertLogEntry": {
        "metadata": {
        "apiDefinitionUrl": "https://.../swagger/docs/v1",
        "swaggerSource": "website"
        },
        "type": "Http",
        "inputs": {
        "body": {
            "date": "@{outputs('Gets_NewPatientRecord')['headers']['Date']}",
            "operation": "New Patient",
            "patientId": "@{triggerBody()['CRMid']}",
            "providerId": "@{triggerBody()['providerID']}",
            "source": "@{outputs('Gets_NewPatientRecord')['headers']}"
        },
        "method": "post",
        "uri": "https://.../api/Log"
        },
        "runAfter":    {
            "Gets_NewPatientecord": ["Succeeded"]
        }
}
```

#### 記錄檔要求

這是張貼至 API 應用程式的記錄檔要求訊息。

``` json
    {
    "uri": "https://.../api/Log",
    "method": "post",
    "body": {
	    "date": "Fri, 10 Jun 2016 22:31:56 GMT",
	    "operation": "New Patient",
	    "patientId": "6b115f6d-a7ee-e511-80f5-3863bb2eb2d0",
	    "providerId": "",
	    "source": "{"Pragma":"no-cache","x-ms-request-id":"e750c9a9-bd48-44c4-bbba-1688b6f8a132","OData-Version":"4.0","Cache-Control":"no-cache","Date":"Fri, 10 Jun 2016 22:31:56 GMT","Set-Cookie":"ARRAffinity=785f4334b5e64d2db0b84edcc1b84f1bf37319679aefce206b51510e56fd9770;Path=/;Domain=127.0.0.1","Server":"Microsoft-IIS/8.0,Microsoft-HTTPAPI/2.0","X-AspNet-Version":"4.0.30319","X-Powered-By":"ASP.NET","Content-Length":"1935","Content-Type":"application/json; odata.metadata=minimal; odata.streaming=true","Expires":"-1"}"
    	}
    }

```


#### 記錄檔回應

這是來自 API 應用程式的記錄檔回應訊息。

``` json
{
    "statusCode": 200,
    "headers": {
	    "Pragma": "no-cache",
	    "Cache-Control": "no-cache",
	    "Date": "Fri, 10 Jun 2016 22:32:17 GMT",
	    "Server": "Microsoft-IIS/8.0",
	    "X-AspNet-Version": "4.0.30319",
	    "X-Powered-By": "ASP.NET",
	    "Content-Length": "964",
	    "Content-Type": "application/json; charset=utf-8",
	    "Expires": "-1"
    },
    "body": {
	    "ttl": 2592000,
	    "id": "6b115f6d-a7ee-e511-80f5-3863bb2eb2d0_1465597937",
	    "_rid": "XngRAOT6IQEHAAAAAAAAAA==",
	    "_self": "dbs/XngRAA==/colls/XngRAOT6IQE=/docs/XngRAOT6IQEHAAAAAAAAAA==/",
	    "_ts": 1465597936,
	    "_etag": ""0400fc2f-0000-0000-0000-575b3ff00000"",
	    "patientID": "6b115f6d-a7ee-e511-80f5-3863bb2eb2d0",
	    "timestamp": "2016-06-10T22:31:56Z",
	    "source": "{"Pragma":"no-cache","x-ms-request-id":"e750c9a9-bd48-44c4-bbba-1688b6f8a132","OData-Version":"4.0","Cache-Control":"no-cache","Date":"Fri, 10 Jun 2016 22:31:56 GMT","Set-Cookie":"ARRAffinity=785f4334b5e64d2db0b84edcc1b84f1bf37319679aefce206b51510e56fd9770;Path=/;Domain=127.0.0.1","Server":"Microsoft-IIS/8.0,Microsoft-HTTPAPI/2.0","X-AspNet-Version":"4.0.30319","X-Powered-By":"ASP.NET","Content-Length":"1935","Content-Type":"application/json; odata.metadata=minimal; odata.streaming=true","Expires":"-1"}",
	    "operation": "New Patient",
	    "salesforceId": "",
	    "expired": false
    }
}

```

現在讓我們看看錯誤處理步驟。


### 錯誤處理

下列 Logic Apps 的程式碼範例示範如何實作錯誤處理。

#### 建立錯誤記錄

這是用來建立錯誤記錄的 Logic Apps 原始程式碼。

``` json
"actions": {
    "CreateErrorRecord": {
        "metadata": {
        "apiDefinitionUrl": "https://.../swagger/docs/v1",
        "swaggerSource": "website"
        },
        "type": "Http",
        "inputs": {
        "body": {
            "action": "New_Patient",
            "isError": true,
            "crmId": "@{triggerBody()['CRMid']}",
            "patientID": "@{triggerBody()['CRMid']}",
            "message": "@{body('Create_NewPatientRecord')['message']}",
            "providerId": "@{triggerBody()['providerId']}",
            "severity": 4,
            "source": "@{actions('Create_NewPatientRecord')['inputs']['body']}",
            "statusCode": "@{int(outputs('Create_NewPatientRecord')['statusCode'])}",
            "salesforceId": "",
            "update": false
        },
        "method": "post",
        "uri": "https://.../api/CrMtoSfError"
        },
        "runAfter":
        {
            "Create_NewPatientRecord": ["Failed" ]
        }
    }
}  	       
```

#### 將錯誤插入至 DocumentDB--要求

``` json

{
    "uri": "https://.../api/CrMtoSfError",
    "method": "post",
    "body": {
        "action": "New_Patient",
        "isError": true,
        "crmId": "6b115f6d-a7ee-e511-80f5-3863bb2eb2d0",
        "patientId": "6b115f6d-a7ee-e511-80f5-3863bb2eb2d0",
        "message": "Salesforce failed to complete task: Message: duplicate value found: Account_ID_MED__c duplicates value on record with id: 001U000001c83gK",
        "providerId": "",
        "severity": 4,
        "salesforceId": "",
        "update": false,
        "source": "{"Account_Class_vod__c":"PRAC","Account_Status_MED__c":"I","CRM_HUB_ID__c":"6b115f6d-a7ee-e511-80f5-3863bb2eb2d0","Credentials_vod__c","DTC_ID_MED__c":"","Fax":"","FirstName":"A","Gender_vod__c":"","IMS_ID__c":"","LastName":"BAILEY","MasterID_mp__c":"","C_ID_MED__c":"851588","Middle_vod__c":"","NPI_vod__c":"","PDRP_MED__c":false,"PersonDoNotCall":false,"PersonEmail":"","PersonHasOptedOutOfEmail":false,"PersonHasOptedOutOfFax":false,"PersonMobilePhone":"","Phone":"","Practicing_Specialty__c":"FM - FAMILY MEDICINE","Primary_City__c":"","Primary_State__c":"","Primary_Street_Line2__c":"","Primary_Street__c":"","Primary_Zip__c":"","RecordTypeId":"012U0000000JaPWIA0","Request_Date__c":"2016-06-10T22:31:55.9647467Z","ONY_ID__c":"","Specialty_1_vod__c":"","Suffix_vod__c":"","Website":""}",
        "statusCode": "400"
    }
}
```

#### 將錯誤插入至 DocumentDB--回應


``` json
{
    "statusCode": 200,
    "headers": {
        "Pragma": "no-cache",
        "Cache-Control": "no-cache",
        "Date": "Fri, 10 Jun 2016 22:31:57 GMT",
        "Server": "Microsoft-IIS/8.0",
        "X-AspNet-Version": "4.0.30319",
        "X-Powered-By": "ASP.NET",
        "Content-Length": "1561",
        "Content-Type": "application/json; charset=utf-8",
        "Expires": "-1"
    },
    "body": {
        "id": "6b115f6d-a7ee-e511-80f5-3863bb2eb2d0-1465597917",
        "_rid": "sQx2APhVzAA8AAAAAAAAAA==",
        "_self": "dbs/sQx2AA==/colls/sQx2APhVzAA=/docs/sQx2APhVzAA8AAAAAAAAAA==/",
        "_ts": 1465597912,
        "_etag": ""0c00eaac-0000-0000-0000-575b3fdc0000"",
        "prescriberId": "6b115f6d-a7ee-e511-80f5-3863bb2eb2d0",
        "timestamp": "2016-06-10T22:31:57.3651027Z",
        "action": "New_Patient",
        "salesforceId": "",
        "update": false,
        "body": "CRM failed to complete task: Message: duplicate value found: CRM_HUB_ID__c duplicates value on record with id: 001U000001c83gK",
        "source": "{"Account_Class_vod__c":"PRAC","Account_Status_MED__c":"I","CRM_HUB_ID__c":"6b115f6d-a7ee-e511-80f5-3863bb2eb2d0","Credentials_vod__c":"DO - Degree level is DO","DTC_ID_MED__c":"","Fax":"","FirstName":"A","Gender_vod__c":"","IMS_ID__c":"","LastName":"BAILEY","MterID_mp__c":"","Medicis_ID_MED__c":"851588","Middle_vod__c":"","NPI_vod__c":"","PDRP_MED__c":false,"PersonDoNotCall":false,"PersonEmail":"","PersonHasOptedOutOfEmail":false,"PersonHasOptedOutOfFax":false,"PersonMobilePhone":"","Phone":"","Practicing_Specialty__c":"FM - FAMILY MEDICINE","Primary_City__c":"","Primary_State__c":"","Primary_Street_Line2__c":"","Primary_Street__c":"","Primary_Zip__c":"","RecordTypeId":"012U0000000JaPWIA0","Request_Date__c":"2016-06-10T22:31:55.9647467Z","XXXXXXX":"","Specialty_1_vod__c":"","Suffix_vod__c":"","Website":""}",
        "code": 400,
        "errors": null,
        "isError": true,
        "severity": 4,
        "notes": null,
        "resolved": 0
        }
}
```

#### Salesforce 錯誤回應

``` json
{
    "statusCode": 400,
    "headers": {
        "Pragma": "no-cache",
        "x-ms-request-id": "3e8e4884-288e-4633-972c-8271b2cc912c",
        "X-Content-Type-Options": "nosniff",
        "Cache-Control": "no-cache",
        "Date": "Fri, 10 Jun 2016 22:31:56 GMT",
        "Set-Cookie": "ARRAffinity=785f4334b5e64d2db0b84edcc1b84f1bf37319679aefce206b51510e56fd9770;Path=/;Domain=127.0.0.1",
        "Server": "Microsoft-IIS/8.0,Microsoft-HTTPAPI/2.0",
        "X-AspNet-Version": "4.0.30319",
        "X-Powered-By": "ASP.NET",
        "Content-Length": "205",
        "Content-Type": "application/json; charset=utf-8",
        "Expires": "-1"
    },
    "body": {
        "status": 400,
        "message": "Salesforce failed to complete task: Message: duplicate value found: Account_ID_MED__c duplicates value on record with id: 001U000001c83gK",
        "source": "Salesforce.Common",
        "errors": []
    }
}

```

### 將回應傳回父邏輯應用程式

取得回應之後，您可以將它傳遞回父邏輯應用程式。

#### 將成功回應傳回給父邏輯應用程式

``` json
"SuccessResponse": {
    "runAfter":
        {
            "UpdateNew_CRMPatientResponse": ["Succeeded"]
        },
    "inputs": {
        "body": {
            "status": "Success"
    },
    "headers": {
    "	Content-type": "application/json",
        "x-ms-date": "@utcnow()"
    },
    "statusCode": 200
    },
    "type": "Response"
}
```

#### 將錯誤回應傳回給父邏輯應用程式

``` json
"ErrorResponse": {
    "runAfter":
        {
            "Create_NewPatientRecord": ["Failed"]
        },
    "inputs": {
        "body": {
            "status": "BadRequest"
        },
        "headers": {
            "Content-type": "application/json",
            "x-ms-date": "@utcnow()"
        },
        "statusCode": 400
    },
    "type": "Response"
}

```


## DocumentDB 儲存機制和入口網站

我們的解決方案使用 [DocumentDB](https://azure.microsoft.com/services/documentdb) 加入了額外功能。

### 錯誤管理入口網站

若要檢視錯誤，您可以建立 MVC Web 應用程式，以顯示來自 DocumentDB 的錯誤記錄。目前的版本中包含了**清單**、**詳細資料**、**編輯**和**刪除**作業。

> [AZURE.NOTE] 編輯作業︰DocumentDB 會執行整個文件的取代工作。**清單**和**詳細資料**檢視所顯示的記錄僅是範例，而非實際的病患預約記錄。

以下是使用先前所述方法所建立之 MVC 應用程式詳細資料的範例。

#### 錯誤管理清單

![錯誤清單](./media/app-service-logic-scenario-error-and-exception-handling/errorlist.png)

#### 錯誤管理詳細資料檢視

![錯誤詳細資料](./media/app-service-logic-scenario-error-and-exception-handling/errordetails.png)

### 記錄檔管理入口網站

為了檢視記錄檔，我們還建立了 MVC Web 應用程式。以下是使用先前所述方法所建立之 MVC 應用程式詳細資料的範例。

#### 範例記錄檔詳細資料檢視

![記錄檔詳細資料檢視](./media/app-service-logic-scenario-error-and-exception-handling/samplelogdetail.png)

### API 應用程式詳細資料

#### Logic Apps 例外狀況管理 API

我們的開放原始碼 Logic Apps 例外狀況管理 API 應用程式提供了下列功能。

有兩個控制器：

- **ErrorController** 會在 DocumentDB 集合中插入錯誤記錄 (文件)。
- **LogController** 會在 DocumentDB 集合中插入記錄檔記錄 (文件)。

> [AZURE.TIP] 這兩個控制器都使用 `async Task<dynamic>` 作業。這可讓作業在執行階段解析，讓我們可以在作業的主體中建立 DocumentDB 結構描述。

DocumentDB 中的每個文件都必須具有唯一識別碼。我們將會使用 `PatientId`，並加入轉換為 Unix 時間戳記值 (雙精確度) 的時間戳記。我們會將此值截斷以移除小數值。

您可以 [從 GitHub](https://github.com/HEDIDIN/LogicAppsExceptionManagementApi/blob/master/Logic App Exception Management API/Controllers/ErrorController.cs) 檢視錯誤控制器 API 的原始程式碼。

我們使用下列語法從邏輯應用程式呼叫 API。

``` json
 "actions": {
        "CreateErrorRecord": {
          "metadata": {
            "apiDefinitionUrl": "https://.../swagger/docs/v1",
            "swaggerSource": "website"
          },
          "type": "Http",
          "inputs": {
            "body": {
              "action": "New_Patient",
              "isError": true,
              "crmId": "@{triggerBody()['CRMid']}",
              "prescriberId": "@{triggerBody()['CRMid']}",
              "message": "@{body('Create_NewPatientRecord')['message']}",
              "salesforceId": "@{triggerBody()['salesforceID']}",
              "severity": 4,
              "source": "@{actions('Create_NewPatientRecord')['inputs']['body']}",
              "statusCode": "@{int(outputs('Create_NewPatientRecord')['statusCode'])}",
              "update": false
            },
            "method": "post",
            "uri": "https://.../api/CrMtoSfError"
          },
          "runAfter": {
              "Create_NewPatientRecord": ["Failed"]
            }
        }
 }
```

上述程式碼範例的運算式會檢查「Create\_NewPatientRecord」狀態是否為 **Failed**。

## 摘要

- 您可以在邏輯應用程式中輕鬆地實作記錄和錯誤處理。
- 您可以使用 DocumentDB 做為記錄檔和錯誤記錄 (文件) 的儲存機制。
- 您可以使用 MVC 建立入口網站，以顯示記錄檔和錯誤記錄。

### 原始程式碼
Logic Apps 例外狀況管理 API 應用程式的原始程式碼可在此 [GitHub 儲存機制](https://github.com/HEDIDIN/LogicAppsExceptionManagementApi "邏輯應用程式例外狀況管理 API")取得。


## 後續步驟
- [檢視其他 Logic Apps 範例和案例](app-service-logic-examples-and-scenarios.md)
- [了解 Logic Apps 監視工具](app-service-logic-monitor-your-logic-apps.md)
- [建立邏輯應用程式的自動部署範本](app-service-logic-create-deploy-template.md)

<!---HONumber=AcomDC_0817_2016-->