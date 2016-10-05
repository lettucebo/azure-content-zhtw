<properties
	pageTitle="Stretch Database 概觀 | Microsoft Azure"
	description="了解 Stretch Database 能以無訊息方式安全地將冷資料移轉到 Microsoft Azure 雲端。"
	services="sql-server-stretch-database"
	documentationCenter=""
	authors="douglaslMS"
	manager=""
	editor=""/>

<tags
	ms.service="sql-server-stretch-database"
	ms.workload="data-management"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="get-started-article"
	ms.date="06/27/2016"
	ms.author="douglasl"/>

# Stretch Database 概觀

Stretch Database 能以無訊息方式安全地將冷資料移轉到 Microsoft Azure 雲端。

如果您只想立即開始使用 Stretch Database，請參閱 [Get started by running the Enable Database for Stretch Wizard (開始執行為資料庫啟用延伸功能精靈)](sql-server-stretch-database-wizard.md)。

## Stretch Database 有哪些優點？
Stretch Database 提供下列優點：

### 能以符合成本效益的方式提供冷資料
使用 SQL Server Stretch Database 即可以動態方式將暖交易資料和冷交易資料從 SQL Server 延伸到 Microsoft Azure。不同於一般的冷資料儲存體，您的資料會隨時保持上線狀態並可用來執行查詢。您可以延長資料保留期，卻又不致於讓用來存放大型資料表 (如客戶訂單歷史資料) 的空間塞爆。受益於 Azure 的低成本，而非擴充價格昂貴的內部部署儲存體。您可以在 Azure 入口網站選擇定價層及進行設定，以控制成本。並可視需要擴大或縮小規模。如需詳細資料，請瀏覽 [SQL Server Stretch Database 價格](https://azure.microsoft.com/pricing/details/sql-server-stretch-database/)頁面。

### 不必變更查詢或應用程式
不論是位於內部部署環境或延伸到雲端的 SQL Server 資料皆能順暢存取。您可以設定原則來決定資料的儲存位置，並由 SQL Server 負責在背景處理資料的移動。整份資料表隨時保持上線狀態並可供查詢。而且，Stretch Database 並不需要您變更現有查詢或應用程式，而且應用程式對於資料的所在位置一清二楚。

### 簡化內部部署資料的維護工作
減少在內部部署環境維護和儲存資料的工作。內部部署資料的備份速度加快，因而可在維護期間內完成。自動執行雲端資料的備份。內部部署的儲存體需求量大幅降低。使用 Azure 儲存體的成本比新增到內部部署 SSD 低 80%。

### 即使在移轉期間也能確保資料安全
放心地將最重要的應用程式安全延伸到雲端。SQL Server 的永遠加密功能可為傳輸中的資料加密。資料列層級安全性 (RLS) 和其他進階的 SQL Server 安全性功能也可與 Stretch Database 搭配運作來保護資料。

## Stretch Database 有何作用？
為 SQL Server 執行個體、資料庫和至少一個資料表啟用 Stretch Database 後，Stretch Database 便會以無訊息模式開始將冷資料移轉至 Azure。

-   如果您將冷資料儲存在個別的資料表上，您可以移轉整個資料表。

-   如果您的資料表同時包含熱資料和冷資料，您可以指定篩選函式以選取要移轉的資料列。

**您不必變更現有查詢和用戶端應用程式。** 即使正在移轉資料，也能繼續順暢存取本機和遠端的資料。進行遠端查詢時會有些許延遲，但在查詢冷資料時也只會遇到這方面的延遲。

**Stretch Database 可確保資料不會在移轉失敗時遺失**。此外，它還有重試邏輯可處理移轉期間可能發生的連線問題。動態管理檢視則能提供移轉的狀態。

**您可以暫停資料移轉**，以便針對本機伺服器問題進行疑難排解或讓可用網路頻寬達到最大。

![Stretch Database 概觀][StretchOverviewImage1]

## Stretch Database 是否適合您？
如果您的情況符合下列陳述，Stretch Database 應該可以滿足您的需求並幫助您解決問題。

|如果您是決策人員|如果您是資料庫管理員|
|------------------------------|-------------------|
|我必須長時間保存交易資料。|我的資料表大小逐漸失控。|
|我有時必須查詢冷資料。|使用者表示他們需要存取冷資料，但實際上卻很少使用。|
|我有不想更新的應用程式，包括較舊版的應用程式。|我必須不斷購買和增加更多儲存體。|
|我想要設法節省儲存體成本。|我無法在 SLA 內備份或還原如此大型的資料表。|

## 何種資料庫和資料表是 Stretch Database 的適用對象？
Stretch Database 的目標是擁有大量冷資料，且通常儲存在少量資料表中的交易資料庫。這些資料表所包含的資料列數目可能超過 10 億個。

如果您使用 SQL Server 2016 的時態表功能，請使用 Stretch Database 將相關聯的全部或部分歷史資料表移轉至符合成本效益的 Azure 儲存體。如需詳細資訊，請參閱[管理已設定系統版本之時態表中歷史資料的保留期](https://msdn.microsoft.com/library/mt637341.aspx)。

使用 SQL Server 2016 Upgrade Advisor 的 Stretch Database Advisor 功能來識別適用於 Stretch Database 的資料庫和資料表。如需詳細資訊，請參閱[為 Stretch Database 識別資料庫和資料表](sql-server-stretch-database-identify-databases.md)。若要深入了解潛在的阻礙問題，請參閱 [Stretch Database 的限制](sql-server-stretch-database-limitations.md)。

## 測試 Stretch Database
**請使用 AdventureWorks 範例資料庫測試 Stretch Database。** 若要取得 AdventureWorks 範例資料庫，請從[這裡](https://www.microsoft.com/download/details.aspx?id=49502)至少下載資料庫檔案和範例與指令碼。將範例資料庫還原到 SQL Server 2016 執行個體後，解壓縮範例檔案，然後從 Stretch DB 資料夾開啟 Stretch DB 範例檔案。執行此檔案中的指令碼，檢查資料在啟用 Stretch Database 前後所使用的空間、追蹤資料移轉進度，以及確認您可以在資料移轉期間和之後繼續查詢現有資料和插入新資料。

## 後續步驟
**識別可做為 Stretch Database 適用對象的資料庫和資料表。** 下載 SQL Server 2016 Upgrade Advisor 並執行 Stretch Database Advisor 以識別可做為 Stretch Database 適用對象的資料庫和資料表。Stretch Database Advisor 也能找出阻礙問題。如需詳細資訊，請參閱[為 Stretch Database 識別資料庫和資料表](sql-server-stretch-database-identify-databases.md)。

<!--Image references-->
[StretchOverviewImage1]: ./media/sql-server-stretch-database-overview/StretchDBOverview.png
[StretchOverviewImage2]: ./media/sql-server-stretch-database-overview/StretchDBOverview1.png
[StretchOverviewImage3]: ./media/sql-server-stretch-database-overview/StretchDBOverview2.png

<!---HONumber=AcomDC_0629_2016--->