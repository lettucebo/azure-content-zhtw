<properties
   pageTitle="使用動態管理檢視監視 Azure SQL Database | Microsoft Azure"
   description="了解如何使用動態管理檢視監視 Microsoft Azure SQL Database 來偵測和診斷常見的效能問題。"
   services="sql-database"
   documentationCenter=""
   authors="CarlRabeler"
   manager="jhubbard"
   editor=""
   tags=""/>

<tags
   ms.service="sql-database"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="data-management"
   ms.date="09/20/2016"
   ms.author="carlrab"/>

# 使用動態管理檢視監視 Azure SQL Database

Microsoft Azure SQL Database 可使用動態管理檢視的子集來診斷可能因為封鎖或長時間執行的查詢、資源瓶頸、不佳的查詢計畫等等所造成的效能問題。本主題提供有關如何使用動態管理檢視來偵測常見效能問題的資訊。

SQL Database 部分支援動態管理檢視的三個類別目錄：

- 資料庫相關的動態管理檢視。
- 執行相關的動態管理檢視。
- 交易相關的動態管理檢視。

如需動態管理檢視的詳細資訊，請參閱《SQL Server 線上叢書》中的[動態管理檢視和函數 (Transact-SQL)](https://msdn.microsoft.com/library/ms188754.aspx)。

## 權限

在 SQL Database 中，查詢動態管理檢視需要 **VIEW DATABASE STATE** 權限。**VIEW DATABASE STATE** 權限會傳回目前資料庫中所有物件的相關資訊。若要授與 **VIEW DATABASE STATE** 權限給特定的資料庫使用者，請執行下列查詢：

```GRANT VIEW DATABASE STATE TO database_user; ```

在內部部署 SQL Server 的執行個體中，動態管理檢視會傳回伺服器狀態資訊。在 SQL Database 中，僅會傳回與您目前的邏輯資料庫相關的資訊。

## 正在計算資料庫大小

下列查詢會傳回資料庫的大小 (以 MB 為單位)：

```
-- Calculates the size of the database.
SELECT SUM(reserved_page_count)*8.0/1024
FROM sys.dm_db_partition_stats;
GO
```

下列查詢會傳回您資料庫中個別物件的大小 (以 MB 為單位)：

```
-- Calculates the size of individual database objects.
SELECT sys.objects.name, SUM(reserved_page_count) * 8.0 / 1024
FROM sys.dm_db_partition_stats, sys.objects
WHERE sys.dm_db_partition_stats.object_id = sys.objects.object_id
GROUP BY sys.objects.name;
GO
```

## 監視連線

您可以使用 [sys.dm\_exec\_connections](https://msdn.microsoft.com/library/ms181509.aspx) 檢視，擷取與特定 Azure SQL Database 伺服器建立之連接和每個連接之詳細資料的相關資訊。此外，[sys.dm\_exec\_sessions](https://msdn.microsoft.com/library/ms176013.aspx) 檢視有助於擷取所有作用中使用者的連接資訊和內部工作。下列查詢可擷取目前的連接資訊：

```
SELECT
    c.session_id, c.net_transport, c.encrypt_option,
    c.auth_scheme, s.host_name, s.program_name,
    s.client_interface_name, s.login_name, s.nt_domain,
    s.nt_user_name, s.original_login_name, c.connect_time,
    s.login_time
FROM sys.dm_exec_connections AS c
JOIN sys.dm_exec_sessions AS s
    ON c.session_id = s.session_id
WHERE c.session_id = @@SPID;
```

> [AZURE.NOTE] 執行 **sys.dm\_exec\_requests** 和 **sys.dm\_exec\_sessions views** 時，如果您具備資料庫的 **VIEW DATABASE STATE** 權限，您就會看到該資料庫上所有正在執行的工作階段；否則您只會看到目前的工作階段。

## 監視查詢效能

速度慢或長時間執行的查詢可能會耗用大量系統資源。本節示範如何使用動態管理檢視偵測幾個常見的查詢效能問題。在 Microsoft TechNet 上的文章[疑難排解 SQL Server 2008 的效能問題](http://download.microsoft.com/download/D/B/D/DBDE7972-1EB9-470A-BA18-58849DB3EB3B/TShootPerfProbs2008.docx)雖然較舊，但仍是針對疑難排解相當有用的參考資料。

### 尋找前 N 個查詢

下列範例會傳回以平均 CPU 時間排名的前五個查詢的相關資訊。此範例會根據查詢雜湊來彙總查詢，使在邏輯上等同的查詢依其累積資源耗用量進行分組。

```
SELECT TOP 5 query_stats.query_hash AS "Query Hash",
    SUM(query_stats.total_worker_time) / SUM(query_stats.execution_count) AS "Avg CPU Time",
    MIN(query_stats.statement_text) AS "Statement Text"
FROM
    (SELECT QS.*,
    SUBSTRING(ST.text, (QS.statement_start_offset/2) + 1,
    ((CASE statement_end_offset
        WHEN -1 THEN DATALENGTH(ST.text)
        ELSE QS.statement_end_offset END
            - QS.statement_start_offset)/2) + 1) AS statement_text
     FROM sys.dm_exec_query_stats AS QS
     CROSS APPLY sys.dm_exec_sql_text(QS.sql_handle) as ST) as query_stats
GROUP BY query_stats.query_hash
ORDER BY 2 DESC;
```

### 監視封鎖的查詢

速度慢或長時間執行的查詢會造成過度的資源耗用，並且會導致封鎖查詢的後果。導致封鎖的原因可能是不佳的應用程式設計、不良的查詢計畫、缺乏有用的索引等等。您可以使用 sys.dm\_tran\_locks 檢視，取得在您的 Azure SQL Database 中目前正鎖定之活動的相關資訊。如需範例程式碼，請參閱《SQL Server 線上叢書》中的 [sys.dm\_tran\_locks (Transact-SQL)](https://msdn.microsoft.com/library/ms190345.aspx)。

### 監視查詢計畫

效率不佳的查詢計畫也可能會增加 CPU 耗用量。下列範例使用 [sys.dm\_exec\_query\_stats](https://msdn.microsoft.com/library/ms189741.aspx) 檢視來判斷哪一個查詢使用最多的累計 CPU。

```
SELECT
    highest_cpu_queries.plan_handle,
    highest_cpu_queries.total_worker_time,
    q.dbid,
    q.objectid,
    q.number,
    q.encrypted,
    q.[text]
FROM
    (SELECT TOP 50
        qs.plan_handle,
        qs.total_worker_time
    FROM
        sys.dm_exec_query_stats qs
    ORDER BY qs.total_worker_time desc) AS highest_cpu_queries
    CROSS APPLY sys.dm_exec_sql_text(plan_handle) AS q
ORDER BY highest_cpu_queries.total_worker_time DESC;
```

## 另請參閱

[SQL Database 簡介](sql-database-technical-overview.md)

<!---HONumber=AcomDC_0921_2016-->