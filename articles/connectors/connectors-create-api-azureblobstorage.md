<properties
    pageTitle="在您的 Logic Apps 中新增 Azure Blob 儲存體連接器 | Microsoft Azure"
    description="搭配 REST API 參數來使用 Azure Blob 儲存體連接器的概觀"
    services=""
    documentationCenter="" 
    authors="MandiOhlinger"
    manager="erikre"
    editor=""
    tags="connectors"/>

<tags
   ms.service="logic-apps"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="integration" 
   ms.date="07/25/2016"
   ms.author="mandia"/>

# 開始使用 Azure Blob 儲存體連接器
Azure Blob 儲存體是用來儲存大量非結構化資料的服務。請在 Azure Blob 儲存體中執行上傳、更新、取得和刪除 Blob 等各種動作。

您可以利用 Azure Blob 儲存體來：

- 上傳新專案或取得最近更新的檔案以建置工作流程。
- 使用動作來取得檔案中繼資料、刪除檔案、複製檔案和進行其他作業。例如，當 Azure 網站中的工具更新時 (觸發程序)，則更新 Blob 儲存體中的檔案 (動作)。

本主題說明如何在邏輯應用程式中使用 Blob 儲存體連接器，並且也列出動作。

>[AZURE.NOTE] 這個版本的文章適用於 Logic Apps 公開上市版本 (GA)。

若要深入瞭解 Logic Apps，請參閱[什麼是邏輯應用程式](../app-service-logic/app-service-logic-what-are-logic-apps.md)以及[建立邏輯應用程式](../app-service-logic/app-service-logic-create-a-logic-app.md)。

## 連線至 Azure Blob 儲存體

您必須先建立與服務的「連線」，才能透過邏輯應用程式存取任何服務。連線可讓邏輯應用程式與另一個服務連線。例如，若要連線至儲存體帳戶，您得先建立 Blob 儲存體連線。若要建立連線，請輸入平常用來存取所連線服務的認證。因此，請在 Azure 儲存體中，輸入儲存體帳戶的認證以建立連線。

#### 建立連線

>[AZURE.INCLUDE [建立至 Azure Blob 儲存體的連線](../../includes/connectors-create-api-azureblobstorage.md)]
 
## 使用觸發程序

此連接器並沒有任何觸發程序。請使用其他觸發程序來啟動邏輯應用程式，例如循環觸發程序、HTTP Webhook 觸發程序、其他連接器適用的觸發程序等等。[建立邏輯應用程式](../app-service-logic/app-service-logic-create-a-logic-app.md)可提供範例。

## 使用動作
	
動作是由邏輯應用程式中定義的工作流程所執行的作業。

1. 選取加號。您會看到幾個選擇︰[新增動作]、[新增條件] 或其中一個 [其他] 選項。

	![](./media/connectors-create-api-azureblobstorage/add-action.png)

2. 選擇 [新增動作]。

3. 在文字方塊中，輸入「blob」以取得所有可用動作的清單。

	![](./media/connectors-create-api-azureblobstorage/actions.png)

4. 在我們的範例中，選擇 [AzureBlob - 使用路徑取得檔案中繼資料]。如果連線已存在，請選取 [...] \(顯示選擇器) 按鈕來選取檔案。

	![](./media/connectors-create-api-azureblobstorage/sample-file.png)

	如果系統提示您輸入連線資訊，請輸入詳細資料以建立連線。本主題的[建立連線](connectors-create-api-azureblobstorage.md#create-the-connection)一節會說明這些屬性。

	> [AZURE.NOTE] 在此範例中，我們會取得檔案的中繼資料。若要查看中繼資料，請新增另一個動作，以使用另一個連接器建立新檔案。例如，新增 OneDrive 動作，以根據中繼資料建立新的「測試」檔案。

5. **儲存**您的變更 (工具列的左上角)。邏輯應用程式將會儲存，而且可能會自動啟用。

> [AZURE.TIP] [Storage Explorer](http://storageexplorer.com/) 是很適合用來管理多個儲存體帳戶的工具。

## 技術詳細資料

## 儲存體 Blob 動作

|動作|說明|
|--- | ---|
|[取得檔案中繼資料](connectors-create-api-azureblobstorage.md#get-file-metadata)|這項作業會使用檔案識別碼取得檔案中繼資料。|
|[更新檔案](connectors-create-api-azureblobstorage.md#update-file)|這項作業會更新檔案。|
|[刪除檔案](connectors-create-api-azureblobstorage.md#delete-file)|這項作業會刪除檔案。|
|[使用路徑來取得檔案中繼資料](connectors-create-api-azureblobstorage.md#get-file-metadata-using-path)|這項作業會使用路徑取得檔案中繼資料。|
|[使用路徑來取得檔案內容](connectors-create-api-azureblobstorage.md#get-file-content-using-path)|這項作業會使用路徑取得檔案內容。|
|[取得檔案內容](connectors-create-api-azureblobstorage.md#get-file-content)|這項作業會使用識別碼取得檔案內容。|
|[建立檔案](connectors-create-api-azureblobstorage.md#create-file)|這項作業會上傳檔案。|
|[複製檔案](connectors-create-api-azureblobstorage.md#copy-file)|這項作業會將檔案複製到 Azure Blob 儲存體。|
|[將封存檔案解壓縮到資料夾](connectors-create-api-azureblobstorage.md#extract-archive-to-folder)|這項作業會將封存檔案解壓縮到資料夾 (範例︰.zip)。|

### 動作詳細資料

在本節中，請查看每個動作的特定詳細資料，包括任何必要或選擇性的輸入屬性，以及任何與連接器相關聯的對應輸出。

#### 取得檔案中繼資料
這項作業會使用檔案識別碼取得檔案中繼資料。

|屬性名稱| 顯示名稱|說明|
| ---|---|---|
|識別碼*|檔案|選取檔案|

星號 (*) 代表必要屬性。

##### 輸出詳細資料
BlobMetadata

| 屬性名稱 | 資料類型 |
|---|---|
|識別碼|字串|
|名稱|字串|
|DisplayName|字串|
|Path|字串|
|LastModified|字串|
|大小|integer|
|MediaType|字串|
|IsFolder|布林值|
|ETag|字串|
|FileLocator|字串|


#### 更新檔案
這項作業會更新檔案。

|屬性名稱| 顯示名稱|說明|
| ---|---|---|
|識別碼*|檔案|選取檔案|
|body*|檔案內容|要更新之檔案的內容|

星號 (*) 代表必要屬性。

##### 輸出詳細資料
BlobMetadata

| 屬性名稱 | 資料類型 |
|---|---|
|識別碼|字串|
|名稱|字串|
|DisplayName|字串|
|Path|字串|
|LastModified|字串|
|大小|integer|
|MediaType|字串|
|IsFolder|布林值|
|ETag|字串|
|FileLocator|字串|


#### 刪除檔案
這項作業會刪除檔案。

|屬性名稱| 顯示名稱|說明|
| ---|---|---|
|識別碼*|檔案|選取檔案|

星號 (*) 代表必要屬性。

##### 輸出詳細資料
無。


#### 使用路徑來取得檔案中繼資料
這項作業會使用路徑取得檔案中繼資料。

|屬性名稱| 顯示名稱|說明|
| ---|---|---|
|path*|檔案路徑|選取檔案|

星號 (*) 代表必要屬性。

##### 輸出詳細資料
BlobMetadata

| 屬性名稱 | 資料類型 |
|---|---|
|識別碼|字串|
|名稱|字串|
|DisplayName|字串|
|Path|字串|
|LastModified|字串|
|大小|integer|
|MediaType|字串|
|IsFolder|布林值|
|ETag|字串|
|FileLocator|字串|


#### 使用路徑來取得檔案內容
這項作業會使用路徑取得檔案內容。

|屬性名稱| 顯示名稱|說明|
| ---|---|---|
|path*|檔案路徑|選取檔案|

星號 (*) 代表必要屬性。

##### 輸出詳細資料
無。


#### 取得檔案內容
這項作業會使用識別碼取得檔案內容。

|屬性名稱| 資料類型|說明|
| ---|---|---|
|識別碼*|字串|選取檔案|

星號 (*) 代表必要屬性。

##### 輸出詳細資料
無。


#### 建立檔案
這項作業會上傳檔案。

|屬性名稱| 顯示名稱|說明|
| ---|---|---|
|folderPath*|資料夾路徑|選取資料夾|
|name*|檔案名稱|要上傳之檔案的名稱|
|body*|檔案內容|要上傳之檔案的內容|

星號 (*) 代表必要屬性。

##### 輸出詳細資料
BlobMetadata

| 屬性名稱 | 資料類型 | 
|---|---|
|識別碼|字串|
|名稱|字串|
|DisplayName|字串|
|Path|字串|
|LastModified|字串|
|大小|integer|
|MediaType|字串|
|IsFolder|布林值|
|ETag|字串|
|FileLocator|字串|


#### 複製檔案
這項作業會將檔案複製到 Azure Blob 儲存體。

|屬性名稱| 顯示名稱|說明|
| ---|---|---|
|source*|來源 URL|指定來源檔案的 URL|
|destination*|目的地檔案路徑|指定目的地檔案路徑，包括目標檔案名稱|
|overwrite|覆寫？|是否應覆寫現有目的地檔案 (true/false)？ |

星號 (*) 代表必要屬性。

##### 輸出詳細資料
BlobMetadata

| 屬性名稱 | 資料類型 |
|---|---|
|識別碼|字串|
|名稱|字串|
|DisplayName|字串|
|Path|字串|
|LastModified|字串|
|大小|integer|
|MediaType|字串|
|IsFolder|布林值|
|ETag|字串|
|FileLocator|字串|

#### 將封存檔案解壓縮到資料夾
這項作業會將封存檔案解壓縮到資料夾 (範例︰.zip)。

|屬性名稱| 顯示名稱|說明|
| ---|---|---|
|source*|來源封存檔案路徑|選取封存檔案|
|destination*|目的地資料夾路徑|選取要解壓縮的內容|
|overwrite|覆寫？|是否應覆寫現有目的地檔案 (true/false)？|

星號 (*) 代表必要屬性。

##### 輸出詳細資料
BlobMetadata

| 屬性名稱 | 資料類型 |
|---|---|
|識別碼|字串|
|名稱|字串|
|DisplayName|字串|
|Path|字串|
|LastModified|字串|
|大小|integer|
|MediaType|字串|
|IsFolder|布林值|
|ETag|字串|
|FileLocator|字串|


## HTTP 回應

呼叫不同動作時，您可能會收到特定回應。下表概述回應及其說明︰

|名稱|說明|
|---|---|
|200|OK|
|202|已接受|
|400|不正確的要求|
|401|未經授權|
|403|禁止|
|404|找不到|
|500|內部伺服器錯誤。發生未知錯誤|
|預設值|作業失敗。|

## 後續步驟

[建立邏輯應用程式](../app-service-logic/app-service-logic-create-a-logic-app.md)。請到我們的 [API 清單](apis-list.md)探索 Logic Apps 中其他可用的連接器。

<!---HONumber=AcomDC_0727_2016-->