<properties 
	pageTitle="預覽具有 MongoDB 的通訊協定支援的 DocumentDB 帳戶的開發指導方針 | Microsoft Azure" 
	description="了解預覽具有 MongoDB 的通訊協定支援的 DocumentDB 帳戶的開發指導方針，現在可供預覽" 
	services="documentdb" 
	authors="andrewhoh" 
	manager="jhubbard" 
	editor="" 
	documentationCenter=""/>

<tags 
	ms.service="documentdb" 
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="09/15/2016" 
	ms.author="anhoh"/>

# 具有 MongoDB 的通訊協定支援的 DocumentDB 帳戶的開發指導方針

您可以透過任何開放原始碼 MongoDB 用戶端[驅動程式](https://docs.mongodb.org/ecosystem/drivers/)與 Azure DocumentDB 通訊。MongoDB 的通訊協定支援假設 MongoDB 用戶端驅動程式與 MongoDB 2.6 或更新版本的伺服器端點進行通訊。DocumentDB 藉由遵守 MongoDB [連線通訊協定](https://docs.mongodb.org/manual/reference/mongodb-wire-protocol/) 2.6 版來支援這項工作。

DocumentDB 支援核心 MongoDB API 函式來建立、讀取、更新和刪除 (CRUD) 資料，以及查詢資料庫。實作的功能已根據常見的平台、架構、工具和應用程式模式的需求設定優先順序。

## 集合

> [AZURE.IMPORTANT] DocumentDB 在集合層級使用保留的輸送量，傳遞保證、可預測的效能。因此，DocumentDB 中的集合是可計費的實體。

效能保留會在集合層級套用，使應用程式可以調整系統中的資料容器的最低層級效能。因此集合的價格取決於以每秒要求單位計量的集合的輸送量，以及已使用儲存體總計 (以 gb 為單位)。佈建的輸送量可以在集合存留期期間進行調整，以適應多變的處理需求和應用程式的存取模式。如需詳細資訊，請參閱 [DocumentDB 效能層級](documentdb-performance-levels.md)。

依預設，具有 MongoDB 通訊協定支援的 DocumentDB 的集合是以 1,000 RU/秒的佈建輸送量的標準定價層建立。您可以調整您的每個集合的佈建輸送量，如[使用 Azure 入口網站變更效能層級](documentdb-performance-levels.md#changing-performance-levels-using-the-azure-portal)中所述。

## CRUD 作業

完全支援 MongoDB 插入、讀取、更新、取代和刪除作業。這包括支援 MongoDB [更新運算子規格](https://docs.mongodb.org/manual/reference/operator/update/)所指定的欄位、陣列、位元和隔離更新。對於需要多個文件操作的更新運算子，DocumentDB 提供完整 ACID 語意與快照集隔離。

## 查詢作業

DocumentDB 支援 MongoDB 查詢的完整文法，並且只有一些例外狀況。除了支援 MongoDB 日期時間格式，在 JSON 相容集合的 [BSON 類型](https://docs.mongodb.org/manual/reference/bson-types/)上操作的查詢也受到支援。對於需要非 JSON 類型特定運算子的查詢，DocumentDB 支援 GUID 資料類型。

## 入口網站體驗
已啟用 MongoDB 通訊協定的帳戶會建立已啟用 MongoDB 通訊協定的帳戶的 Azure 入口網站體驗。我們正在尋求拓展體驗，但是我們需要您關於哪個入口網站功能對您最有用的[意見反應](mailto:askdocdb@microsoft.com?subject=DocumentDB%20Protocol%20Support%20for%20MongoDB%20Preview%20Portal%20Experience)。

## 支援矩陣


### CRUD 和查詢作業

功能|支援|支援
---|---|---
插入|InsertOne| 
 |InsertMany| 
 |插入| 
更新|UpdateOne| 
 |UpdateMany| 
 |更新| 
欄位更新|$inc、$mul、$rename、$set、$unset、$min、$max|$currentDate| 
陣列更新| |-all-
位元| |-all-
隔離| |-all-
將|ReplaceOne| 
刪除|DeleteOne | 
 |DeleteMany| 
 |移除| 
BulkWrite| |bulkWrite()
比較|-all-| 
邏輯|-all-| 
元素查詢| |-all-
評估|$mod、$regex |$text、$where
GeoSpatial|2dsphere、2d、polygon|其餘
Array|$all、$size、$elemMatch|
位元| |-all-
註解|-all-| 
投射| |-all-


### 資料庫命令

功能|支援|支援
---|---|---
彙總|Count| 
GeoSpatial| |-all-
查詢及寫入|find、insert、update、delete、getLastError、getMore、findAndModify| 
驗證|getnonce、logout、authenticate| 
系統管理|createIndex、listIndexes、dropIndexes、connectionStatus、reIndex| 
診斷|listDatabases、collStats、dbStats| 

## 後續步驟

- 了解如何[使用 MongoChef](documentdb-mongodb-mongochef.md) 和具有 MongoDB 通訊協定支援的 DocumentDB 帳戶。
- 探索具有 MongoDB 通訊協定支援的 DocumentDB 的[範例](documentdb-mongodb-samples.md)。

 

<!---HONumber=AcomDC_0921_2016-->