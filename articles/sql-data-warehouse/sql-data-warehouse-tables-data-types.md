<properties
   pageTitle="SQL 資料倉儲中的資料表的資料類型 | Microsoft Azure"
   description="開始使用 Azure SQL 資料倉儲資料表的資料類型。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="jrowlandjones"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services"
   ms.date="06/29/2016"
   ms.author="jrj;barbkess;sonyama"/>

# SQL 資料倉儲中的資料表的資料類型

> [AZURE.SELECTOR]
- [概觀][]
- [資料類型][]
- [散發][]
- [Index][]
- [資料分割][]
- [統計資料][]
- [暫存][]

SQL 資料倉儲支援最常用的資料類型。以下是 SQL 資料倉儲支援的資料類型清單。如需有關資料類型支援的詳細資訊，請參閱[建立資料表][]。

|**支援的資料類型**|||
|---|---|---|
[bigint][]|[decimal][]|[smallint][]|
[binary][]|[float][]|[smallmoney][]|
[bit][]|[int][]|[sysname][]|
[char][]|[money][]|[time][]|
[date][]|[nchar][]|[tinyint][]|
[datetime][]|[nvarchar][]|[uniqueidentifier][]|
[datetime2][]|[real][]|[varbinary][]|
[datetimeoffset][]|[smalldatetime][]|[varchar][]|


## 資料類型的最佳作法

 在定義資料行類型時，使用可支援您的資料的最小資料類型，將能夠改善查詢效能。這對 CHAR 和 VARCHAR 資料行尤其重要。如果資料行中最長的值是 25 個字元，請將您的資料行定義為 VARCHAR(25)。避免將所有字元資料行定義為較大的預設長度。此外，將資料行定義為 VARCHAR (當它只需要這樣的大小時) 而非使用 [NVARCHAR][]。儘可能使用 NVARCHAR(4000) 或 VARCHAR(8000)，而非 NVARCHAR(MAX) 或 VARCHAR(MAX)。

## Polybase 限制

如果您正在使用 PolyBase 以載入您的資料表，請定義您的資料表，使最大可能的資料列大小 (包括可變長度資料行的完整長度) 不超過 32,767 個位元組。雖然您定義的資料列可以有超過此寬度的可變長度資料，並透過 BCP 載入資料列，您將無法使用 PolyBase 載入此資料。PolyBase 針對廣泛資料列的支援將會於近期內新增。

## 不支援的資料類型

如果您從另一個 SQL 平台 (例如 Azure SQL Database) 移轉您的資料庫，在移轉時，您可能會遇到 SQL 資料倉儲不支援的某些資料類型。以下是不支援的資料類型，以及一些您可以用來取代不支援的資料類型的替代方案。

|資料類型|因應措施|
|---|---|
|[geometry][]|[varbinary][]|
|[geography][]|[varbinary][]|
|[hierarchyid][]|[nvarchar][](4000)|
|[image][ntext,text,image]|[varbinary][]|
|[文字][ntext,text,image]|[varchar][]|
|[ntext][ntext,text,image]|[nvarchar][]|
|[sql\_variant][]|將資料行分割成數個強型別資料行。|
|[資料表][]|轉換成暫存資料表。|
|[timestamp][]|修改程式碼來使用 [datetime2][] 和 `CURRENT_TIMESTAMP` 函式。僅支援常數做為預設值，因此 current\_timestamp 不可定義為預設條件約束。如果您需要從 timestamp 類型資料行移轉資料列版本值，請對 NOT NULL 或 NULL 資料列版本值使用 [BINARY][] \(8) 或 [VARBINARY][BINARY](8)。|
|[xml][]|[varchar][]|
|[使用者定義型別][]|可能的話，轉換回原生型別|
|預設值|預設值僅支援常值和常數。不支援不具決定性的運算式或函式，例如 `GETDATE()` 或 `CURRENT_TIMESTAMP`。|

在您目前的 SQL 資料庫上，可執行以下 SQL 來識別 Azure SQL 資料倉儲不支援的資料行︰

```sql
SELECT  t.[name], c.[name], c.[system_type_id], c.[user_type_id], y.[is_user_defined], y.[name]
FROM sys.tables  t
JOIN sys.columns c on t.[object_id]    = c.[object_id]
JOIN sys.types   y on c.[user_type_id] = y.[user_type_id]
WHERE y.[name] IN ('geography','geometry','hierarchyid','image','text','ntext','sql_variant','timestamp','xml')
 AND  y.[is_user_defined] = 1;
```

## 後續步驟

若要深入了解，請參閱[資料表概觀][Overview]、[散發資料表][Distribute]、[編製資料表的索引][Index]、[分割資料表][Partition]、[維護資料表統計資料][Statistics]和[暫存資料表][Temporary]等文章。若要深入了解最佳做法，請參閱 [SQL Data 資料倉儲最佳做法][]。

<!--Image references-->

<!--Article references-->
[Overview]: ./sql-data-warehouse-tables-overview.md
[概觀]: ./sql-data-warehouse-tables-overview.md
[資料類型]: ./sql-data-warehouse-tables-data-types.md
[Distribute]: ./sql-data-warehouse-tables-distribute.md
[散發]: ./sql-data-warehouse-tables-distribute.md
[Index]: ./sql-data-warehouse-tables-index.md
[Partition]: ./sql-data-warehouse-tables-partition.md
[資料分割]: ./sql-data-warehouse-tables-partition.md
[Statistics]: ./sql-data-warehouse-tables-statistics.md
[統計資料]: ./sql-data-warehouse-tables-statistics.md
[Temporary]: ./sql-data-warehouse-tables-temporary.md
[暫存]: ./sql-data-warehouse-tables-temporary.md
[SQL Data 資料倉儲最佳做法]: ./sql-data-warehouse-best-practices.md

<!--MSDN references-->

<!--Other Web references-->
[建立資料表]: https://msdn.microsoft.com/library/mt203953.aspx
[bigint]: https://msdn.microsoft.com/library/ms187745.aspx
[binary]: https://msdn.microsoft.com/library/ms188362.aspx
[bit]: https://msdn.microsoft.com/library/ms177603.aspx
[char]: https://msdn.microsoft.com/library/ms176089.aspx
[date]: https://msdn.microsoft.com/library/bb630352.aspx
[datetime]: https://msdn.microsoft.com/library/ms187819.aspx
[datetime2]: https://msdn.microsoft.com/library/bb677335.aspx
[datetimeoffset]: https://msdn.microsoft.com/library/bb630289.aspx
[decimal]: https://msdn.microsoft.com/library/ms187746.aspx
[float]: https://msdn.microsoft.com/library/ms173773.aspx
[geometry]: https://msdn.microsoft.com/library/cc280487.aspx
[geography]: https://msdn.microsoft.com/library/cc280766.aspx
[hierarchyid]: https://msdn.microsoft.com/library/bb677290.aspx
[int]: https://msdn.microsoft.com/library/ms187745.aspx
[money]: https://msdn.microsoft.com/library/ms179882.aspx
[nchar]: https://msdn.microsoft.com/library/ms186939.aspx
[nvarchar]: https://msdn.microsoft.com/library/ms186939.aspx
[ntext,text,image]: https://msdn.microsoft.com/library/ms187993.aspx
[real]: https://msdn.microsoft.com/library/ms173773.aspx
[smalldatetime]: https://msdn.microsoft.com/library/ms182418.aspx
[smallint]: https://msdn.microsoft.com/library/ms187745.aspx
[smallmoney]: https://msdn.microsoft.com/library/ms179882.aspx
[sql\_variant]: https://msdn.microsoft.com/library/ms173829.aspx
[sysname]: https://msdn.microsoft.com/library/ms186939.aspx
[資料表]: https://msdn.microsoft.com/library/ms175010.aspx
[time]: https://msdn.microsoft.com/library/bb677243.aspx
[timestamp]: https://msdn.microsoft.com/library/ms182776.aspx
[tinyint]: https://msdn.microsoft.com/library/ms187745.aspx
[uniqueidentifier]: https://msdn.microsoft.com/library/ms187942.aspx
[varbinary]: https://msdn.microsoft.com/library/ms188362.aspx
[varchar]: https://msdn.microsoft.com/library/ms186939.aspx
[xml]: https://msdn.microsoft.com/library/ms187339.aspx
[使用者定義型別]: https://msdn.microsoft.com/library/ms131694.aspx

<!---HONumber=AcomDC_0706_2016-->