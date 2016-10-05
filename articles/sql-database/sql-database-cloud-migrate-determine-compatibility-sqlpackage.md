<properties
   pageTitle="使用 SqlPackage.exe 判斷 SQL Database 相容性 | Microsoft Azure"
   description="Microsoft Azure SQL Database, 資料庫移轉, SQL Database 相容性, SqlPackage"
   services="sql-database"
   documentationCenter=""
   authors="CarlRabeler"
   manager="jhubbard"
   editor=""/>

<tags
   ms.service="sql-database"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="sqldb-migrate"
   ms.date="08/24/2016"
   ms.author="carlrab"/>

# 使用 SqlPackage.exe 判斷 SQL Database 相容性

> [AZURE.SELECTOR]
- [SSDT](sql-database-cloud-migrate-fix-compatibility-issues-ssdt.md)
- [SqlPackage](sql-database-cloud-migrate-determine-compatibility-sqlpackage.md)
- [SSMS](sql-database-cloud-migrate-determine-compatibility-ssms.md)
- [Upgrade Advisor](http://www.microsoft.com/download/details.aspx?id=48119)
- [SAMW](sql-database-cloud-migrate-fix-compatibility-issues.md)

在本文中，您將學習判斷 SQL Server 資料庫是否相容，以使用 [SqlPackage](https://msdn.microsoft.com/library/hh550080.aspx) 命令提示字元公用程式將其移轉到 SQL Database。

## 使用 SqlPackage.exe

1. 開啟命令提示字元並變更包含最新版 sqlpackage.exe 的目錄。此公用程式隨附於最新版的 [SQL Server Management Studio](https://msdn.microsoft.com/library/mt238290.aspx) 和 [SQL Server Data Tools for Visual Studio](https://msdn.microsoft.com/library/mt204009.aspx)，或者您也可以直接從 Microsoft 下載中心下載最新版的 [SqlPackage](https://www.microsoft.com/zh-TW/download/details.aspx?id=53876)。
2. 以環境的下列引數執行下列 SqlPackage 命令：

	'sqlpackage.exe /Action:Export /ssn:< server\_name > /sdn:< database\_name > /tf:< target\_file > /p:TableData=< schema\_name.table\_name > > < output\_file > 2>&1'

	| 引數 | 說明 |
	|---|---|
	| < server\_name > | 來源伺服器名稱 |
	| < database\_name > | 來源資料庫名稱 |
	| < target\_file > | BACPAC 檔案的檔案名稱和位置 |
	| < schema\_name.table\_name > | 其資料要輸出到目標檔案的資料表 |
	| < output\_file > | 具有錯誤的輸出檔案的檔案名稱和位置 (若有的話) |

	之所以使用 /P:TableName 引數，是因為我們只想要測試資料庫相容性來了解是否能夠匯出到 Azure SQL DB V12，而不想要從所有資料表匯出資料。遺憾的是，sqlpackage.exe 的匯出引數不支援不擷取任何資料表。您必須指定至少一個資料表，例如單一的小型資料表。< output\_file > 包含任何錯誤的報告。"> 2>&1" 字串會將從命令執行產生的標準輸出和標準錯誤輸送到指定的輸出檔。

	![從 [工作] 功能表匯出資料層應用程式](./media/sql-database-cloud-migrate/TestForCompatibilityUsingSQLPackage01.png)

3. 開啟輸出檔案並檢視相容性錯誤 (如果有的話)。

	![從 [工作] 功能表匯出資料層應用程式](./media/sql-database-cloud-migrate/TestForCompatibilityUsingSQLPackage02.png)

## 後續步驟

- [最新版本的 SSDT](https://msdn.microsoft.com/library/mt204009.aspx) [最新版本的 SQL Server Management Studio](https://msdn.microsoft.com/library/mt238290.aspx)
- [修正資料庫移轉相容性問題](sql-database-cloud-migrate.md#fix-database-migration-compatibility-issues)
- [將相容 SQL Server 資料庫移轉到 SQL Database](sql-database-cloud-migrate.md#migrate-a-compatible-sql-server-database-to-sql-database)

## 其他資源

- [SQL Database V12](sql-database-v12-whats-new.md)
- [Transact-SQL 部分支援或不支援的函數](sql-database-transact-sql-information.md)
- [使用 SQL Server 移轉小幫手來移轉非 SQL Server 資料庫](http://blogs.msdn.com/b/ssma/)

<!---HONumber=AcomDC_0921_2016-->