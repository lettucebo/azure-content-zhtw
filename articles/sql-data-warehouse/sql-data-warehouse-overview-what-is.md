<properties
   pageTitle="什麼是 Azure SQL 資料倉儲？| Microsoft Azure"
   description="企業級分散式資料庫，可處理資料量高達 PB 的關聯式與非關聯式資料。它是業界首見能在幾秒內增加、縮減和暫停的雲端資料倉儲。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="lodipalm"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.devlang="NA"
   ms.topic="get-started-article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services"
   ms.date="09/27/2016"
   ms.author="lodipalm;barbkess;mausher;jrj;sonyama;kevin"/>


# 什麼是 Azure SQL 資料倉儲？

Azure SQL 資料倉儲是一種雲端架構、相應放大的資料庫，可處理大量的關聯式與非關聯式資料。SQL 資料倉儲是以我們的巨量平行處理 (MPP) 架構為基礎，可以處理您的企業工作負載。

SQL 資料倉儲：

- 結合 SQL Server 關聯式資料庫與 Azure 雲端相應放大功能。您可以在數秒內增加、減少、暫停或繼續計算。您可在有需要時相應放大 CPU，以及在非尖峰時間內減少使用量，進而節省成本。
- 運用 Azure 平台。可容易部署、順暢地維護，並透過自動備份提供完整容錯。
- 補充 SQL Server 生態系統。您可以使用熟悉的 SQL Server Transact-SQL (T-SQL) 和工具進行開發。

本文說明 SQL 資料倉儲的重要功能。

## 巨量平行處理架構

SQL 資料倉儲是大量平行處理 (MPP) 分散式資料庫系統。將資料和處理能力分割於多個節點之間，SQL 資料倉儲就能夠提供極大的延展性 - 遠超過任何單一系統。在幕後，SQL 資料倉儲會將您的資料分散於多個不共用的儲存和處理單元。資料會儲存在進階本地備援儲存體中，並連結至計算節點以便執行查詢。使用這個架構，SQL 資料倉儲可以採用「分治法」來執行載入和複雜的查詢。要求是由控制節點接收、最佳化，然後傳遞至計算節點來平行執行其工作。

藉由結合 MPP 架構與 Azure 儲存體功能，SQL 資料倉儲可以︰

- 擴大或縮小獨立計算的儲存體。
- 擴大或縮小計算，而不移動資料。
- 暫停計算功能，並使資料保持不變。
- 隨時可以恢復計算功能。

下圖更詳細說明此架構。

![SQL 資料倉儲架構][1]


**控制節點︰**控制節點會管理及最佳化查詢。它是與所有應用程式與連接互動的前端。在 SQL 資料倉儲中，控制節點採用 SQL Database 計算，而且連接它時的外觀及操作方式相同。控制節點會暗中協調對分散式資料執行平行查詢所需的所有資料移動和計算。當您將 TSQL 查詢提交到 SQL 資料倉儲時，控制節點會將其轉換為會在每個計算節點上平行執行的個別查詢。

**計算節點︰**計算節點是 SQL 資料倉儲背後的動力。它們是儲存資料和處理查詢的 SQL Database。當您新增資料時，SQL 資料倉儲會將資料列分散到各個計算節點。計算節點是會對您的資料執行平行查詢的背景工作角色。經過處理後，它們會將結果傳回控制節點。為了完成查詢，控制節點會彙總結果並傳回最終的結果。

**儲存體：**您的資料會儲存在 Azure Blob 儲存體中。當計算節點與您的資料互動時，它們會直接從 Blob 儲存體來回寫入和讀取。由於 Azure 儲存體可大幅地明確擴充，SQL 資料倉儲同樣也可以。計算和儲存體各自獨立，所以 SQL 資料倉儲可以自動調整儲存體，與計算分開調整，反之亦然。Azure Blob 儲存體也是完全容錯，並可簡化備份和還原程序。

**資料移動服務︰**資料移動服務 (DMS) 可在節點之間移動資料。DMS 可讓計算節點存取聯結和彙總所需的資料。DMS 不是 Azure 服務。它是一項在所有節點上與 SQL Database 並排執行的 Windows 服務。因為 DMS 會在幕後執行，所以您不會與它直接互動。不過，當您查看查詢計劃時，您會發現這類計劃包含一些 DMS 作業，因為資料移動必須平行執行每項查詢。


## 已針對資料倉儲工作負載最佳化

MPP 方法由許多資料倉儲特定效能最佳化輔助，包括：

- 一項分散式查詢最佳化工具和一組複雜的統計資料。使用資料大小和散發的資訊，服務就能夠藉由評估特定分散式查詢作業的成本來最佳化查詢。

- 整合到資料移動程序中的進階演算法和技術，能夠視需要在計算資源之間有效率地移動資料以執行查詢。這些資料移動作業均為內建，且資料移動服務的所有最佳化都會自動進行。

- 預設的叢集**資料行存放**區索引。與傳統的資料列導向儲存體相比，使用資料行儲存體的 SQL 資料倉儲最多可讓壓縮平均提升 5 倍，查詢效能提升 10 倍以上。需要掃描大量資料列的分析查詢適合使用資料行存放區索引。


## 可預測且可調整的效能

SQL 資料倉儲會分隔儲存體和計算功能，以便各自單獨進行調整。SQL 資料倉儲可以快速又簡單地調整成新增額外的計算資源。輔助這項功能就是使用 Azure Blob 儲存體。Blob 不只提供穩定、複寫儲存體，同時也提供低成本的輕鬆擴充基礎結構。SQL 資料倉儲運用雲端規模儲存體和 Azure 計算的組合，讓您只在需要時支付查詢效能和儲存體的費用。變更計算數量就像是在 Azure 入口網站中將滑桿向左或向右移動一樣簡單，也可以使用 T-SQL 和 PowerShell 排定。

SQL 資料倉儲提供獨立於儲存體且可完全控制計算數量的功能，可讓您完全暫停資料倉儲，這表示您不需要計算時便不用支付其費用。在保留您的儲存體位置的同時，所有計算會釋放給 Azure 的主集區，為您節省費用。有需要時，只要繼續計算，並讓工作負載使用資料和計算即可。

## 資料倉儲單位

SQL 資料倉儲的資源配置是使用資料倉儲單位 (DWU) 測量。DWU 是配置給 SQL 資料倉儲的基礎資源 (例如 CPU、記憶體、IOPS) 的量值。增加 DWU 數目可增加資源和效能。具體而言，DWU 有助於確保：

- 您可以輕鬆地調整您的資料倉儲，而不必擔心基礎硬體或軟體。

- 您可以在變更資料倉儲的大小之前，預測 DWU 層級的效能改進程度。

- 可以變更或移動您的執行個體的基礎硬體與軟體，而不會影響工作負載效能。

- Microsoft 可以調整服務的基礎架構，而不會影響您的工作負載的效能。

- Microsoft 可以快速提升 SQL 資料倉儲的效能，以可調整及平均影響系統的方式執行這項操作。

資料倉儲單位會提供包含三個精確度量的量值，而這些度量與資料倉儲工作負載效能高度相互關聯。目標是下列關鍵工作負載度量會以您針對資料倉儲選擇的 DWU 進行線性調整。

**掃描/彙總**：此工作負載度量會採用標準資料倉儲查詢，該查詢會掃描大量資料列，然後執行複雜的彙總。這是 I/O 和 CPU 密集作業。

**負載**：此度量會測量將資料擷取至服務的能力。PolyBase 從 Azure Blob 儲存體載入具代表性的資料集時完成負載。這項度量是設計來對服務的網路和 CPU 層面給予壓力。

**Create Table As Select (CTAS)**：CTAS 會測量複製資料表的能力。這牽涉到從儲存體讀取資料、跨應用裝置的節點散發，以及重新寫入至儲存體。這是 CPU、IO 和網路密集作業。

## 依需求暫停與縮放

當您需要更快的結果時，提升您的 DWU 並且為較高的效能支付款項。當您需要較小的計算能力時，減少您的 DWU 並且僅針對您需要的項目支付款項。在下列案例中，您可以考慮變更 DWU︰

- 不需執行查詢時 (也許是晚上或週末)，請停止查詢。然後暫停計算資源，以免在不需要 DWU 時支付其費用。

- 當系統要求不高時，請考慮將 DWU 減為小型大小。您仍然可以存取資料，且又能節省大量成本。

- 執行大量資料載入或轉換作業時，建議您相應增加以使您的資料更快速可供使用。

為了要了解您理想的 DWU 值，嘗試在載入您的資料之後相應增加和相應減少並執行幾個查詢。由於調整很快速，您可以在一小時內嘗試一些不同層級的效能。請記住，SQL 資料倉儲是設計用來處理大量資料，以及查看其真正的調整功能 (尤其是我們所提供的較大規模)，所以您會想要使用接近或超過 1 TB 的大型資料集。


## 建置在 SQL Server 上

SQL 資料倉儲以 SQL Server 關聯式資料庫引擎為基礎，且包含企業資料倉儲的許多功能。如果您已經熟悉 T-SQL，則可輕鬆地將您的知識傳輸到 SQL 資料倉儲。無論您是進階或新手使用者，文件範例都能夠協助您開始著手。整體來說，您可以考慮我們建構 SQL 資料倉儲的語言元素的方式，如下所示：

- SQL 資料倉儲會將 T-SQL 語法用於許多作業。並且支援一組廣泛的傳統 SQL 建構，例如預存程序、使用者定義函式、資料表資料分割、索引和定序。

- SQL 資料倉儲也包含一些較新的 SQL Server 功能，包括叢集**資料行存放區**索引、PolyBase 整合和資料稽核 (完整的威脅評估)。

- 較不常用於資料倉儲工作負載或是比 SQL Server 還新的特定 TSQL 語言元素目前可能無法使用。如需詳細資訊，請參閱[移轉文件][]。

透過 SQL Server、SQL 資料倉儲、SQL Database 和「分析平台系統」之間的 Transact-SQL 與功能共通性，您能夠開發符合您資料需求的解決方案。您可以根據效能、安全性和調整需求，決定儲存資料的位置，然後視需要在不同系統之間傳輸資料。

## 資料保護

SQL 資料倉儲會將所有資料儲存在 Azure Premium 本地備援儲存體中。並在當地的資料中心維護多份資料的同步複本，確保當地語系化失敗時能夠提供透明的資料保護。此外，SQL 資料倉儲會使用 Azure 儲存體快照，定期自動備份作用中 (未暫停) 的資料庫。若要深入了解備份和還原的運作方式，請參閱[備份與還原概觀][]。

## 與 Microsoft 工具整合

SQL 資料倉儲也整合了許多 SQL Server 使用者可能很熟悉的工具。其中包含：

**傳統的 SQL Server 工具**：SQL 資料倉儲會與 SQL Server Analysis Services、Integration Services 和 Reporting Services 完整整合。

**以雲端為基礎的工具**：SQL 資料倉儲可以與 Azure 中的數個新工具一起使用，包括 Data Factory、串流分析、機器學習服務和 Power BI。如需更完整的清單，請參閱[整合式工具概觀][]。

**協力廠商工具**：許多協力廠商工具提供者與 SQL 資料倉儲整合的工具都已經過認證。如需完整清單，請參閱 [SQL 資料倉儲解決方案合作夥伴][]。

## 混合式資料來源案例

搭配 PolyBase 使用 SQL 資料倉儲可以給予使用者前所未有的能力，在其生態系統之間移動資料，解除鎖定能力以設定具有非關聯式與內部部署資料來源的進階混合式案例。

Polybase 可讓您透過使用熟悉的 T-SQL 命令來運用不同來源的資料。Polybase 讓查詢 Azure Blob 儲存體中的非關聯式資料就像查詢一般資料表般容易。使用 Polybase 即可查詢非關聯式資料，或將非關聯式資料匯入 SQL 資料倉儲。

- PolyBase 使用外部資料表存取非關聯式資料。資料表定義會儲存在 SQL 資料倉儲中，您可以像存取一般關聯式資料一樣，使用 SQL 和工具進行存取。

- 在其整合中無從得知 Polybase。它會為其支援的所有來源提供相同的功能。Polybase 可讀取各種格式的資料，包括分隔檔案或 ORC 檔案。

- PolyBase 可用來存取 Blob 儲存體，該儲存體也用來做為 HDInsight 叢集的儲存體。這可讓您使用關聯式與非關聯式工具，以最新的方式存取相同的資料。

## 後續步驟

現在您已稍微了解 SQL 資料倉儲，請了解如何快速[建立 SQL 資料倉儲][]和[載入範例資料][]。如果您不熟悉 Azure，您可能會發現 [Azure 詞彙][]在您遇到新術語時很有幫助。或者，也可以看一下其中一些其他 SQL 資料倉儲資源。

- [客戶成功案例]
- [部落格]
- [功能要求]
- [影片]
- [客戶諮詢小組部落格]
- [建立支援票證]
- [MSDN 論壇]
- [Stack Overflow 論壇]
- [Twitter]


<!--Image references-->
[1]: ./media/sql-data-warehouse-overview-what-is/dwarchitecture.png

<!--Article references-->
[建立支援票證]: ./sql-data-warehouse-get-started-create-support-ticket.md
[載入範例資料]: ./sql-data-warehouse-load-sample-databases.md
[建立 SQL 資料倉儲]: ./sql-data-warehouse-get-started-provision.md
[移轉文件]: ./sql-data-warehouse-overview-migrate.md
[SQL 資料倉儲解決方案合作夥伴]: ./sql-data-warehouse-partner-business-intelligence.md
[整合式工具概觀]: ./sql-data-warehouse-overview-integrate.md
[備份與還原概觀]: ./sql-data-warehouse-restore-database-overview.md
[Azure 詞彙]: ../azure-glossary-cloud-terminology.md

<!--MSDN references-->

<!--Other Web references-->
[客戶成功案例]: https://customers.microsoft.com/search?sq=&ff=story_products_services%26%3EAzure%2FAzure%2FAzure%20SQL%20Data%20Warehouse%26%26story_product_families%26%3EAzure%2FAzure%26%26story_product_categories%26%3EAzure&p=0
[部落格]: https://azure.microsoft.com/blog/tag/azure-sql-data-warehouse/
[客戶諮詢小組部落格]: https://blogs.msdn.microsoft.com/sqlcat/tag/sql-dw/
[功能要求]: https://feedback.azure.com/forums/307516-sql-data-warehouse
[MSDN 論壇]: https://social.msdn.microsoft.com/Forums/azure/zh-TW/home?forum=AzureSQLDataWarehouse
[Stack Overflow 論壇]: http://stackoverflow.com/questions/tagged/azure-sqldw
[Twitter]: https://twitter.com/hashtag/SQLDW
[影片]: https://azure.microsoft.com/documentation/videos/index/?services=sql-data-warehouse

<!---HONumber=AcomDC_0928_2016-->