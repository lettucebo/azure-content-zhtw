<properties
pageTitle="在您的邏輯應用程式中新增 Twilio 連接器 | Microsoft Azure"
description="搭配 REST API 參數來使用 Twilio 連接器的概觀"
services=""    
documentationCenter=""     
authors="msftman"    
manager="erikre"    
editor=""
tags="connectors"/>

<tags
ms.service="logic-apps"
ms.devlang="na"
ms.topic="article"
ms.tgt_pltfrm="na"
ms.workload="integration"
ms.date="09/19/2016"
ms.author="mandia"/>

# 開始使用 Twilio 連接器

連線到 Twilio 來傳送及接收全域 SMS、多媒體及 IP 訊息。

>[AZURE.NOTE] 這一版的文章適用於邏輯應用程式 2015-08-01-preview 結構描述版本。

您可以利用 Twilio 來：

- 根據您從 Twilio 所取得的資料，來建置您的商務流程。
- 使用會取得訊息、列出訊息等等的動作。這些動作會收到回應，然後輸出能讓其他動作使用的資料。舉例來說，當您收到新的 Twilio 訊息時，您可以取得此訊息，並在服務匯流排工作流程中使用。

如要在邏輯應用程式中新增作業，請參閱[建立邏輯應用程式](../app-service-logic/app-service-logic-create-a-logic-app.md)。

## 觸發程序及動作
Twilio 連接器包含下列動作，但不包含觸發程序。

| 觸發程序 | 動作|
| --- | --- |
|None| <ul><li>取得訊息</li><li>列出訊息</li><li>傳送訊息</li></ul>|

所有連接器都支援 JSON 和 XML 格式的資料。

## 建立至 Twilio 的連線
當您將這個連接器新增到邏輯應用程式時，請輸入下列的 Twilio 值：

|屬性| 必要|說明|
| ---|---|---|
|帳戶識別碼|是|輸入您的 Twilio 帳戶識別碼|
|存取權杖|是|輸入您的 Twilio 存取權杖|

>[AZURE.INCLUDE [建立至 Twilio 連線的步驟](../../includes/connectors-create-api-twilio.md)]

如果您還沒有權杖，請參閱 [Twilio](https://www.twilio.com/docs/api/ip-messaging/guides/identity) 來建立存取權杖。


>[AZURE.TIP] 您可以在其他的邏輯應用程式中，使用這個相同的 Twilio 連線。

## Swagger REST API 參考
#### 本文件適用的版本：1.0

### 取得訊息
傳回由所提供訊息識別碼指定的單一訊息。```GET: /Messages/{MessageId}.json```

| 名稱| 資料類型|必要|位於|預設值|說明|
| ---|---|---|---|---|---|
|MessageId|string|yes|路徑|無|訊息識別碼|

### Response
|Name|說明|
|---|---|
|200|作業已順利完成|
|400|不正確的要求|
|404|找不到訊息|
|500|內部伺服器錯誤。發生未知錯誤|
|預設值|作業失敗。|


### 列出訊息
傳回與您帳戶相關聯的訊息清單。```GET: /Messages.json```

| 名稱| 資料類型|必要|位於|預設值|說明|
| ---|---|---|---|---|---|
|收件人|string|no|query|無|收件者的電話號碼|
|從|string|no|query|無|寄件者的電話號碼|
|DateSent|string|no|query|無|只顯示在這一天 (GMT 格式) 傳送的郵件 ，指定方式為 YYYY-MM-DD。例如：DateSent=2009-07-06。您也可以指定不相等的日期，例如 DateSent<=YYYY-MM-DD 的訊息就是在該日期午夜或午夜之前所傳送的訊息，而 DateSent>=YYYY-MM-DD 則是在該日期的午夜或午夜之後所傳送的訊息。|
|PageSize|integer|no|query|50|在每個清單頁面中所傳回的資源數目。預設值為 50。|
|頁面|integer|no|query|0|頁碼。預設值為 0。|

### Response
|Name|說明|
|---|---|
|200|作業已順利完成|
|400|不正確的要求|
|500|內部伺服器錯誤。發生未知錯誤|
|預設值|作業失敗。|



### 傳送訊息
將新的訊息傳送到某個行動電話號碼。```POST: /Messages.json```

| 名稱| 資料類型|必要|位於|預設值|說明|
| ---|---|---|---|---|---|
|sendMessageRequest| |yes|body|無|要傳送的訊息|

### Response
|Name|說明|
|---|---|
|200|作業已順利完成|
|400|不正確的要求|
|500|內部伺服器錯誤。發生未知錯誤|
|預設值|作業失敗。|


## 物件定義

#### SendMessageRequest：傳送訊息作業的要求模型

|屬性名稱 | 資料類型 | 必要|
|---|---|---|
|from|string|yes|
|to|string|yes|
|body|string|yes|
|media\_url|array|no|
|status\_callback|string|no|
|messaging\_service\_sid|string|no|
|application\_sid|string|no|
|max\_price|string|no|


#### Message：訊息的模型

|屬性名稱 | 資料類型 |必要|
|---|---|---|
|body|string|no|
|from|string|no|
|to|string|no|
|status|string|no|
|sid|string|no|
|account\_sid|string|no|
|api\_version|string|no|
|num\_segments|string|no|
|num\_media|string|no|
|date\_created|string|no|
|date\_sent|string|no|
|date\_updated|string|no|
|direction|string|no|
|error\_code|string|no|
|error\_message|string|no|
|price|string|no|
|price\_unit|string|no|
|uri|string|no|
|subresource\_uris|array|no|
|messaging\_service\_sid|string|no|

#### MessageList：列出訊息作業的回應模型

|屬性名稱 | 資料類型 |必要|
|---|---|---|
|messages|array|no|
|page|integer|no|
|page\_size|integer|no|
|num\_pages|integer|no|
|uri|string|no|
|first\_page\_uri|string|no|
|next\_page\_uri|string|no|
|total|integer|no|
|previous\_page\_uri|string|no|

#### IncomingPhoneNumberList：列出訊息作業的回應模型

|屬性名稱 | 資料類型 |必要|
|---|---|---|
|incoming\_phone\_numbers|array|no|
|page|integer|no|
|page\_size|integer|no|
|num\_pages|integer|no|
|uri|string|no|
|first\_page\_uri|string|no|
|next\_page\_uri|string|no|


#### AddIncomingPhoneNumberRequest：新增來電號碼作業的要求模型

|屬性名稱 | 資料類型 |必要|
|---|---|---|
|PhoneNumber|string|yes|
|AreaCode|string|no|
|FriendlyName|string|no|


#### IncomingPhoneNumber：來電號碼

|屬性名稱 | 資料類型 |必要|
|---|---|---|
|phone\_number|string|no|
|friendly\_name|string|no|
|sid|string|no|
|account\_sid|string|no|
|date\_created|string|no|
|date\_updated|string|no|
|capabilities|沒有定義|no|
|status\_callback|string|no|
|status\_callback\_method|string|no|
|api\_version|string|no|


#### Capabilities：電話號碼容量

|屬性名稱 | 資料類型 |必要|
|---|---|---|
|mms|布林值|no|
|sms|布林值|no|
|voice|布林值|no|

#### AvailablePhoneNumbers：可用的電話號碼

|屬性名稱 | 資料類型 |必要|
|---|---|---|
|phone\_number|string|no|
|friendly\_name|string|no|
|lata|string|no|
|緯度|string|no|
|經度|string|no|
|postal\_code|string|no|
|rate\_center|string|no|
|region|string|no|
|MMS|布林值|no|
|SMS|布林值|no|
|voice|布林值|no|


#### UsageRecords：使用量記錄類別

|屬性名稱 | 資料類型 |必要|
|---|---|---|
|category|string|no|
|usage|string|no|
|usage\_unit|string|no|
|說明|string|no|
|price|number|no|
|price\_unit|string|no|
|計數|string|no|
|count\_unit|string|no|
|start\_date|string|no|
|end\_date|string|no|


## 後續步驟
[建立邏輯應用程式](../app-service-logic/app-service-logic-create-a-logic-app.md)

<!---HONumber=AcomDC_0921_2016-->