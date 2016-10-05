<properties
    pageTitle="在您的 Logic Apps 中新增 DB2 連接器 | Microsoft Azure"
    description="搭配 REST API 參數來使用 DB2 連接器的概觀"
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
   ms.date="09/19/2016"
   ms.author="plarsen"/>


# 開始使用 DB2 連接器
Microsoft Connector for DB2 可將 Logic Apps 連接至 IBM DB2 資料庫中儲存的資源。此連接器包含透過 TCP/IP 網路與遠端 DB2 伺服器電腦通訊的 Microsoft 用戶端，其中包括雲端資料庫 (例如在 Azure 虛擬化中執行的 IBM Bluemix dashDB 或 IBM DB2 for Windows) 和使用內部部署資料閘道的內部部署資料庫。請參閱本主題結尾處支援的 IBM DB2 平台和版本清單。

DB2 連接器支援下列資料庫作業：

- 列出資料庫資料表
- 使用 SELECT 讀取一個資料列
- 使用 SELECT 讀取所有資料列
- 使用 INSERT 加入一個資料列
- 使用 UPDATE 變更一個資料列
- 使用 DELETE 移除一個資料列

本主題說明如何在邏輯應用程式中使用連接器來處理資料庫作業。

>[AZURE.NOTE] 這個版本的文章適用於 Logic Apps 公開上市版本 (GA)。

若要深入了解 Logic Apps，請參閱[建立邏輯應用程式](../app-service-logic/app-service-logic-create-a-logic-app.md)。

## 邏輯應用程式動作
連接器支援下列邏輯應用程式動作：

- Getables
- GetRow
- GetRows
- InsertRow
- UpdateRow
- DeleteRow


## 定義邏輯應用程式以列出資料表
針對由透過 Microsoft Azure 入口網站執行的許多步驟所組成的任何作業，定義邏輯應用程式。您可以定義邏輯應用程式動作來列出 DB2 資料庫中的資料表，以便指示連接器處理 DB2 結構描述陳述式 (CALL SYSIBM.SQLTABLES)。

### 定義邏輯應用程式執行個體
1.	在 **Azure 開始面板**中，依序選取 **+** (加號)、[Web + 行動] 和 [邏輯應用程式]。
2.	輸入**名稱** (例如"**Db2getTables**")、[訂用帳戶]、[資源群組]、[位置] 和 [App Service 方案]。按一下 [釘選到儀表板]，然後選取 [建立]。

### 定義邏輯應用程式觸發程序和動作
1.	在 [Logic Apps 設計工具] 中，請在 [範本] 清單中按一下 [空白 LogicApp]。
2.	在**觸發程序**清單中，按一下 [週期性]。
3.	在 [週期性] 觸發程序中，按一下 [編輯]，按一下 [頻率] 下拉式清單以選取 [天]，然後按一下 [間隔] 以輸入 **7**。
4.	按一下 [+ 新增步驟] 方塊，然後再按一下 [新增動作]。
5.	在**動作**清單中，於 [搜尋更多動作] 編輯方塊中輸入 **db2**，然後按一下 [DB2 - 取得資料表 (預覽)]。

	![](./media/connectors-create-api-db2/Db2connectorActions.png)

6.	在 [DB2 - 取得資料表] 組態窗格中，按一下**核取方塊**以啟用 [透過內部部署資料閘道連接]。請查看從雲端至內部部署的設定變更。
	- 以地址或別名冒號連接埠號碼的格式，輸入 [伺服器] 的值。例如，輸入 **ibmserver01:50000**。
	- 輸入 [資料庫] 的值。例如，輸入 **nwind**。
	- 選取 [驗證] 的值。例如，選取 [基本]。
	- 輸入 [使用者名稱] 的值。例如，輸入 **db2admin**。
	- 輸入 [密碼] 的值。例如，輸入 **Password1**。
	- 選取 [閘道] 的值。例如，按一下 **datagateway01**。
7. 按一下 [建立]，然後按一下 [儲存]。

	![](./media/connectors-create-api-db2/Db2connectorOnPremisesDataGatewayConnection.png)

8.	在 [Db2getTables] 刀鋒視窗中，於 [摘要] 之下的 [所有執行] 清單中，按一下第一個列出的項目 (最近一次執行)。
9.	在 [邏輯應用程式執行] 刀鋒視窗中，按一下 [執行詳細資料]。按一下 [動作] 清單中的 **Get\_tables**。請查看 [狀態] 的值，應該是 [成功]。按一下 [輸入連結]。檢視輸入。按一下 [輸出連結]。檢視輸出，其中應包含資料表清單。

	![](./media/connectors-create-api-db2/Db2connectorGetTablesLogicAppRunOutputs.png)

## 定義 DB2 連接
連接器支援使用下列連接屬性來連接到內部部署資料庫和雲端資料庫。

屬性 | 說明
--- | ---
伺服器 | 必要的伺服器屬性接受代表 TCP/IP 位址或別名的字串值，其採用 IPv4 或 IPv6 格式，後面接著 (以冒號分隔) TCP/IP 通訊埠編號。 
資料庫 | 必要的資料庫屬性接受代表 DRDA 關聯式資料庫名稱 (RDBNAM) 的字串值。DB2 for z/OS 接受 16 個位元組的字串 (資料庫也稱為 IBM DB2 for z/OS 位置)。DB2 for i5/OS 接受 18 個位元組的字串 (資料庫也稱為 IBM DB2 for i 關聯式資料庫)。DB2 for LUW 接受 8 個位元組的字串。
驗證 | 選擇性驗證屬性接受清單項目值 ([基本] 或 Windows (kerberos))。 
使用者名稱 | 必要的使用者名稱屬性接受字串值。DB2 for z/OS 接受 8 個位元組的字串。DB2 for i 接受 10 個位元組的字串。DB2 for Linux 或 UNIX 接受 8 個位元組的字串。DB2 for Windows 接受 30 個位元組的字串。
密碼 | 必要的密碼屬性接受字串值。
閘道 | 必要的閘道屬性接受清單項目值，該值代表對儲存群組內的 Logic Apps 定義的內部部署資料閘道。  

## 定義內部部署閘道連接
連接器可以透過內部部署閘道存取內部部署 DB2 資料庫。如需詳細資訊，請參閱閘道主題。

1. 在 [閘道] 組態窗格中，按一下**核取方塊**以啟用 [透過閘道連接]。請查看從雲端至內部部署的設定變更。
2. 以地址或別名冒號連接埠號碼的格式，輸入 [伺服器] 的值。例如，輸入 **ibmserver01:50000**。
3. 輸入 [資料庫] 的值。例如，輸入 **nwind**。
4. 選取 [驗證] 的值。例如，選取 [基本]。
5. 輸入 [使用者名稱] 的值。例如，輸入 **db2admin**。
6. 輸入 [密碼] 的值。例如，輸入 **Password1**。
7. 選取 [閘道] 的值。例如，按一下 **datagateway01**。
8. 按一下 [建立] 繼續作業。

	![](./media/connectors-create-api-db2/Db2connectorOnPremisesDataGatewayConnection.png)

## 定義雲端連接
連接器可以存取雲端 DB2 資料庫。

1. 在 [閘道] 組態窗格中，讓**核取方塊**停用 (取消選取) [透過閘道連接]。
2. 輸入 [連線名稱] 的值。例如，輸入 **hisdemo2**。
3. 以地址或別名冒號連接埠號碼的格式，輸入 [DB2 伺服器名稱] 的值。例如，輸入 **hisdemo2.cloudapp.net:50000**。
3. 輸入 [DB2 資料庫名稱] 的值。例如，輸入 **nwind**。
4. 輸入 [使用者名稱] 的值。例如，輸入 **db2admin**。
5. 輸入 [密碼] 的值。例如，輸入 **Password1**。
6. 按一下 [建立] 繼續作業。

	![](./media/connectors-create-api-db2/Db2connectorCloudConnection.png)

## 定義邏輯應用程式以使用 SELECT 擷取所有資料列
您可以定義邏輯應用程式動作來擷取 DB2 資料表中的所有資料列，以便指示連接器處理 DB2 SELECT 陳述式 (例如，**SELECT * FROM AREA**)。

### 定義邏輯應用程式執行個體
1.	在 **Azure 開始面板**中，依序選取 **+** (加號)、[Web + 行動] 和 [邏輯應用程式]。
2.	輸入**名稱** (例如"**Db2getRows**")、[訂用帳戶]、[資源群組]、[位置] 和 [App Service 方案]。按一下 [釘選到儀表板]，然後選取 [建立]。

### 定義邏輯應用程式觸發程序和動作
1.	在 [Logic Apps 設計工具] 中，請在 [範本] 清單中按一下 [空白 LogicApp]。
2.	在**觸發程序**清單中，按一下 [週期性]。
3.	在 [週期性] 觸發程序中，按一下 [編輯]，按一下 [頻率] 下拉式清單以選取 [天]，然後按一下 [間隔] 以輸入 **7**。
4.	按一下 [+ 新增步驟] 方塊，然後再按一下 [新增動作]。
5.	在**動作**清單中，於 [搜尋更多動作] 編輯方塊中輸入 **db2**，然後按一下 [DB2 - 取得資料列 (預覽)]。
6. 在 [取得資料列 (預覽)] 動作中，按一下 [變更連線]。
7. 在 [連線] 組態窗格中，按一下 [新建]。

	![](./media/connectors-create-api-db2/Db2connectorNewConnection.png)
  
8. 在 [閘道] 組態窗格中，讓**核取方塊**停用 (取消選取) [透過閘道連接]。
	- 輸入 [連線名稱] 的值。例如，輸入 **HISDEMO2**。
	- 以地址或別名冒號連接埠號碼的格式，輸入 [DB2 伺服器名稱] 的值。例如，輸入 **HISDEMO2.cloudapp.net:50000**。
	- 輸入 [DB2 資料庫名稱] 的值。例如，輸入 **NWIND**。
	- 輸入 [使用者名稱] 的值。例如，輸入 **db2admin**。
	- 輸入 [密碼] 的值。例如，輸入 **Password1**。
9. 按一下 [建立] 繼續作業。

	![](./media/connectors-create-api-db2/Db2connectorCloudConnection.png)

10. 在 [資料表名稱] 清單中，按一下**向下箭號**，然後按一下 **AREA**。
11. (選擇性) 按一下 [顯示進階選項] 來指定查詢選項。
12. 按一下 [儲存]。

	![](./media/connectors-create-api-db2/Db2connectorGetRowsTableName.png)

13.	在 [Db2getRows] 刀鋒視窗中，於 [摘要] 之下的 [所有執行] 清單中，按一下第一個列出的項目 (最近一次執行)。
14.	在 [邏輯應用程式執行] 刀鋒視窗中，按一下 [執行詳細資料]。按一下 [動作] 清單中的 **Get\_rows**。請查看 [狀態] 的值，應該是 [成功]。按一下 [輸入連結]。檢視輸入。按一下 [輸出連結]。檢視輸出，其中應包含資料列清單。

	![](./media/connectors-create-api-db2/Db2connectorGetRowsOutputs.png)

## 定義邏輯應用程式以使用 INSERT 加入一個資料列
您可以定義邏輯應用程式動作以在 DB2 資料表中新增一個資料列，以便指示連接器處理 DB2 INSERT 陳述式 (例如，**INSERT INTO AREA (AREAID, AREADESC, REGIONID) VALUES ('99999', 'Area 99999', 102)**)。

### 定義邏輯應用程式執行個體
1.	在 **Azure 開始面板**中，依序選取 **+** (加號)、[Web + 行動] 和 [邏輯應用程式]。
2.	輸入**名稱** (例如"**Db2insertRow**")、[訂用帳戶]、[資源群組]、[位置] 和 [App Service 方案]。按一下 [釘選到儀表板]，然後選取 [建立]。

### 定義邏輯應用程式觸發程序和動作
1.	在 [Logic Apps 設計工具] 中，請在 [範本] 清單中按一下 [空白 LogicApp]。
2.	在**觸發程序**清單中，按一下 [週期性]。
3.	在 [週期性] 觸發程序中，按一下 [編輯]，按一下 [頻率] 下拉式清單以選取 [天]，然後按一下 [間隔] 以輸入 **7**。
4.	按一下 [+ 新增步驟] 方塊，然後再按一下 [新增動作]。
5.	在**動作**清單中，於 [搜尋更多動作] 編輯方塊中輸入 **db2**，然後按一下 [DB2 - 插入資料列 (預覽)]。
6. 在 [取得資料列 (預覽)] 動作中，按一下 [變更連線]。
7. 在 [連線] 組態窗格中，按一下以選取連接。例如，按一下 **hisdemo2**。

	![](./media/connectors-create-api-db2/Db2connectorChangeConnection.png)

8. 在 [資料表名稱] 清單中，按一下**向下箭號**，然後按一下 **AREA**。
9. 輸入所有必要資料行 (請見紅色星號) 的值。例如，針對 [AREAID] 輸入 "**99999**"，針對 [AREADESC] 輸入「區域 99999」，以及針對 [REGIONID] 輸入 "**102**"。
10. 按一下 [儲存]。

	![](./media/connectors-create-api-db2/Db2connectorInsertRowValues.png)
 
11.	在 [Db2insertRow] 刀鋒視窗中，於 [摘要] 之下的 [所有執行] 清單中，按一下第一個列出的項目 (最近一次執行)。
12.	在 [邏輯應用程式執行] 刀鋒視窗中，按一下 [執行詳細資料]。按一下 [動作] 清單中的 **Get\_rows**。請查看 [狀態] 的值，應該是 [成功]。按一下 [輸入連結]。檢視輸入。按一下 [輸出連結]。檢視輸出，其中應包含新的資料列。

	![](./media/connectors-create-api-db2/Db2connectorInsertRowOutputs.png)

## 定義邏輯應用程式以使用 SELECT 擷取一個資料列
您可以定義邏輯應用程式動作來擷取 DB2 資料表中的一個資料列，以便指示連接器處理 DB2 SELECT WHERE 陳述式 (例如，**SELECT FROM AREA WHERE AREAID = '99999'**)。

### 定義邏輯應用程式執行個體
1.	在 **Azure 開始面板**中，依序選取 **+** (加號)、[Web + 行動] 和 [邏輯應用程式]。
2.	輸入**名稱** (例如"**Db2getRow**")、[訂用帳戶]、[資源群組]、[位置] 和 [App Service 方案]。按一下 [釘選到儀表板]，然後選取 [建立]。

### 定義邏輯應用程式觸發程序和動作
1.	在 [Logic Apps 設計工具] 中，請在 [範本] 清單中按一下 [空白 LogicApp]。
2.	在**觸發程序**清單中，按一下 [週期性]。
3.	在 [週期性] 觸發程序中，按一下 [編輯]，按一下 [頻率] 下拉式清單以選取 [天]，然後按一下 [間隔] 以輸入 **7**。
4.	按一下 [+ 新增步驟] 方塊，然後再按一下 [新增動作]。
5.	在**動作**清單中，於 [搜尋更多動作] 編輯方塊中輸入 **db2**，然後按一下 [DB2 - 取得資料列 (預覽)]。
6. 在 [取得資料列 (預覽)] 動作中，按一下 [變更連線]。
7. 在 [連線] 組態窗格中，按一下以選取現有連接。例如，按一下 **hisdemo2**。

	![](./media/connectors-create-api-db2/Db2connectorChangeConnection.png)

8. 在 [資料表名稱] 清單中，按一下**向下箭號**，然後按一下 **AREA**。
9. 輸入所有必要資料行 (請見紅色星號) 的值。例如，針對 [AREAID] 輸入 "**99999**"。
10. (選擇性) 按一下 [顯示進階選項] 來指定查詢選項。
11. 按一下 [儲存]。

	![](./media/connectors-create-api-db2/Db2connectorGetRowValues.png)

12.	在 [Db2getRow] 刀鋒視窗中，於 [摘要] 之下的 [所有執行] 清單中，按一下第一個列出的項目 (最近一次執行)。
13.	在 [邏輯應用程式執行] 刀鋒視窗中，按一下 [執行詳細資料]。按一下 [動作] 清單中的 **Get\_rows**。請查看 [狀態] 的值，應該是 [成功]。按一下 [輸入連結]。檢視輸入。按一下 [輸出連結]。檢視輸出，其中應包含資料列。

	![](./media/connectors-create-api-db2/Db2connectorGetRowOutputs.png)

## 定義邏輯應用程式以使用 UPDATE 變更一個資料列
您可以定義邏輯應用程式動作來變更 DB2 資料表中的一個資料列，以便指示連接器處理 DB2 UPDATE 陳述式 (例如，**UPDATE AREA SET AREAID = '99999', AREADESC = 'Area 99999', REGIONID = 102)**)。

### 定義邏輯應用程式執行個體
1.	在 **Azure 開始面板**中，依序選取 **+** (加號)、[Web + 行動] 和 [邏輯應用程式]。
2.	輸入**名稱** (例如"**Db2updateRow**")、[訂用帳戶]、[資源群組]、[位置] 和 [App Service 方案]。按一下 [釘選到儀表板]，然後選取 [建立]。

### 定義邏輯應用程式觸發程序和動作
1.	在 [Logic Apps 設計工具] 中，請在 [範本] 清單中按一下 [空白 LogicApp]。
2.	在**觸發程序**清單中，按一下 [週期性]。
3.	在 [週期性] 觸發程序中，按一下 [編輯]，按一下 [頻率] 下拉式清單以選取 [天]，然後按一下 [間隔] 以輸入 **7**。
4.	按一下 [+ 新增步驟] 方塊，然後再按一下 [新增動作]。
5.	在**動作**清單中，於 [搜尋更多動作] 編輯方塊中輸入 **db2**，然後按一下 [DB2 - 更新資料列 (預覽)]。
6. 在 [取得資料列 (預覽)] 動作中，按一下 [變更連線]。
7. 在 [連線] 組態窗格中，按一下以選取現有連接。例如，按一下 **hisdemo2**。

	![](./media/connectors-create-api-db2/Db2connectorChangeConnection.png)

8. 在 [資料表名稱] 清單中，按一下**向下箭號**，然後按一下 **AREA**。
9. 輸入所有必要資料行 (請見紅色星號) 的值。例如，針對 [AREAID] 輸入 "**99999**"，針對 [AREADESC] 輸入「已更新 99999」，以及針對 [REGIONID] 輸入 "**102**"。
10. 按一下 [儲存]。

	![](./media/connectors-create-api-db2/Db2connectorUpdateRowValues.png)

11.	在 [Db2updateRow] 刀鋒視窗中，於 [摘要] 之下的 [所有執行] 清單中，按一下第一個列出的項目 (最近一次執行)。
12.	在 [邏輯應用程式執行] 刀鋒視窗中，按一下 [執行詳細資料]。按一下 [動作] 清單中的 **Get\_rows**。請查看 [狀態] 的值，應該是 [成功]。按一下 [輸入連結]。檢視輸入。按一下 [輸出連結]。檢視輸出，其中應包含新的資料列。

	![](./media/connectors-create-api-db2/Db2connectorUpdateRowOutputs.png)

## 定義邏輯應用程式以使用 DELETE 移除一個資料列
您可以定義邏輯應用程式動作來移除 DB2 資料表中的一個資料列，以便指示連接器處理 DB2 DELETE 陳述式 (例如，**DELETE FROM AREA WHERE AREAID = '99999'**)。

### 定義邏輯應用程式執行個體
1.	在 **Azure 開始面板**中，依序選取 **+** (加號)、[Web + 行動] 和 [邏輯應用程式]。
2.	輸入**名稱** (例如"**Db2deleteRow**")、[訂用帳戶]、[資源群組]、[位置] 和 [App Service 方案]。按一下 [釘選到儀表板]，然後選取 [建立]。

### 定義邏輯應用程式觸發程序和動作
1.	在 [Logic Apps 設計工具] 中，請在 [範本] 清單中按一下 [空白 LogicApp]。
2.	在**觸發程序**清單中，按一下 [週期性]。
3.	在 [週期性] 觸發程序中，按一下 [編輯]，按一下 [頻率] 下拉式清單以選取 [天]，然後按一下 [間隔] 以輸入 **7**。
4.	按一下 [+ 新增步驟] 方塊，然後再按一下 [新增動作]。
5.	在**動作**清單中，於 [搜尋更多動作] 編輯方塊中按一下 **db2**，然後按一下 [DB2 - 刪除資料列 (預覽)]。
6. 在 [取得資料列 (預覽)] 動作中，按一下 [變更連線]。
7. 在 [連線] 組態窗格中，按一下以選取現有連接。例如，按一下 **hisdemo2**。

	![](./media/connectors-create-api-db2/Db2connectorChangeConnection.png)

8. 在 [資料表名稱] 清單中，按一下**向下箭號**，然後按一下 **AREA**。
9. 輸入所有必要資料行 (請見紅色星號) 的值。例如，針對 [AREAID] 輸入 "**99999**"。
10. 按一下 [儲存]。

	![](./media/connectors-create-api-db2/Db2connectorDeleteRowValues.png)

11.	在 [Db2deleteRow] 刀鋒視窗中，於 [摘要] 之下的 [所有執行] 清單中，按一下第一個列出的項目 (最近一次執行)。
12.	在 [邏輯應用程式執行] 刀鋒視窗中，按一下 [執行詳細資料]。按一下 [動作] 清單中的 **Get\_rows**。請查看 [狀態] 的值，應該是 [成功]。按一下 [輸入連結]。檢視輸入。按一下 [輸出連結]。檢視輸出，其中應包含已刪除的資料列。

	![](./media/connectors-create-api-db2/Db2connectorDeleteRowOutputs.png)

## 技術詳細資料

## 動作
動作是由邏輯應用程式中定義的工作流程所執行的作業。DB2 資料庫連接器包含下列動作。

|動作|說明|
|--- | ---|
|[GetRow](connectors-create-api-db2.md#get-row)|從 DB2 資料表擷取單一資料列|
|[GetRows](connectors-create-api-db2.md#get-rows)|從 DB2 資料表擷取多個資料列|
|[InsertRow](connectors-create-api-db2.md#insert-row)|在 DB2 資料表中插入新的資料列|
|[DeleteRow](connectors-create-api-db2.md#delete-row)|刪除 DB2 資料表中的資料列|
|[GetTables](connectors-create-api-db2.md#get-tables)|從 DB2 資料庫擷取資料表|
|[UpdateRow](connectors-create-api-db2.md#update-row)|更新 DB2 資料表中的現有資料列|

### 動作詳細資料

在本節中，請查看每個動作的特定詳細資料，包括任何必要或選擇性的輸入屬性，以及任何與連接器相關聯的對應輸出。

#### 取得單一資料列 
從 DB2 資料表擷取單一資料列。

| 屬性名稱| 顯示名稱 |說明|
| ---|---|---|
|資料表 * | 資料表名稱 |DB2 資料表的名稱|
|識別碼 * | 資料列識別碼 |要擷取之資料列的唯一識別碼|

星號 (*) 代表必要屬性。

##### 輸出詳細資料
項目

| 屬性名稱 | 資料類型 |
|---|---|
|ItemInternalId|string|


#### 取得多個資料列 
從 DB2 資料表擷取多個資料列。

|屬性名稱| 顯示名稱|說明|
| ---|---|---|
|資料表 *|資料表名稱|DB2 資料表的名稱|
|$skip|略過計數|要略過的項目數目 (預設值 = 0)|
|$top|最大取得計數|要擷取的項目數目上限 (預設值 = 256)|
|$filter|篩選查詢|用來限制項目數目的 ODATA filter 查詢|
|$orderby|排序依據|用來指定項目順序的 ODATA orderBy 查詢|

星號 (*) 代表必要屬性。

##### 輸出詳細資料
ItemsList

| 屬性名稱 | 資料類型 |
|---|---|
|value|array|


#### 插入資料列 
在 DB2 資料表中插入新的資料列。

|屬性名稱| 顯示名稱|說明|
| ---|---|---|
|資料表 *|資料表名稱|DB2 資料表的名稱|
|項目 *|資料列|要在指定的 DB2 資料表中插入的資料列|

星號 (*) 代表必要屬性。

##### 輸出詳細資料
項目

| 屬性名稱 | 資料類型 |
|---|---|
|ItemInternalId|string|


#### 刪除資料列 
刪除 DB2 資料表中的資料列。

|屬性名稱| 顯示名稱|說明|
| ---|---|---|
|資料表 *|資料表名稱|DB2 資料表的名稱|
|識別碼*|資料列識別碼|要刪除的資料列的唯一識別碼|

星號 (*) 代表必要屬性。

##### 輸出詳細資料
無。

#### 取得資料表 
從 DB2 資料庫擷取資料表。

這個呼叫沒有參數。

##### 輸出詳細資料 
TablesList

| 屬性名稱 | 資料類型 |
|---|---|
|value|array|

#### 更新資料列 
更新 DB2 資料表中的現有資料列。

|屬性名稱| 顯示名稱|說明|
| ---|---|---|
|資料表 *|資料表名稱|DB2 資料表的名稱|
|識別碼*|資料列識別碼|要更新的資料列的唯一識別碼|
|項目 *|資料列|值已更新的資料列|

星號 (*) 代表必要屬性。

##### 輸出詳細資料  
項目

| 屬性名稱 | 資料類型 |
|---|---|
|ItemInternalId|string|


### HTTP 回應

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


## 支援 DB2 平台和版本
連接器支援這些 IBM DB2 平台和版本，以及 IBM DB2 相容產品 (例如 IBM Bluemix dashDB)，而這些產品支援分散式關聯資料庫架構 (DRDA) SQL 存取管理員 (SQLAM) 版本 10 和 11。

- IBM DB2 for z/OS 11.1
- IBM DB2 for z/OS 10.1
- IBM DB2 for i 7.3
- IBM DB2 for i 7.2
- IBM DB2 for i 7.1
- IBM DB2 for LUW 11
- IBM DB2 for LUW 10.5


## 後續步驟

[建立邏輯應用程式](../app-service-logic/app-service-logic-create-a-logic-app.md)。請到我們的 [API 清單](apis-list.md)探索 Logic Apps 中其他可用的連接器。

<!---HONumber=AcomDC_0921_2016-->