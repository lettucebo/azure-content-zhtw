<properties 
    pageTitle="使用 Transact-SQL 複製 Azure SQL Database | Microsoft Azure" 
    description="使用 Transact-SQL 建立 Azure SQL Database 的複本" 
	services="sql-database"
	documentationCenter=""
	authors="stevestein"
	manager="jhubbard"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.devlang="NA"
	ms.date="09/19/2016"
	ms.author="sstein"
	ms.workload="data-management"
	ms.topic="article"
	ms.tgt_pltfrm="NA"/>


# 使用 Transact-SQL 複製 Azure SQL Database


> [AZURE.SELECTOR]
- [概觀](sql-database-copy.md)
- [Azure 入口網站](sql-database-copy-portal.md)
- [PowerShell](sql-database-copy-powershell.md)
- [T-SQL](sql-database-copy-transact-sql.md)


下列這些步驟說明如何利用 Transact-SQL，將 SQL Database 複製到相同的伺服器或不同的伺服器。資料庫複製作業會使用 [CREATE DATABASE](https://msdn.microsoft.com/library/ms176061.aspx) 陳述式。

若要完成這篇文章中的步驟，您需要下列項目︰

- Azure 訂用帳戶。如果需要 Azure 訂用帳戶，可以先按一下此頁面頂端的 [免費試用]，然後再回來完成這篇文章。
- Azure SQL Database。如果沒有 SQL Database，請遵循本文中以下的步驟：[建立您的第一個 Azure SQL Database](sql-database-get-started.md)。
- [SQL Server Management Studio (SSMS)](https://msdn.microsoft.com/library/ms174173.aspx)。如果您沒有 SSMS，或無法使用本文中描述的功能，請[下載最新版本](https://msdn.microsoft.com/library/mt238290.aspx)。


## 複製您的 SQL Database

使用伺服器層級主體登入或建立您要複製之資料庫的登入來登入 master 資料庫。不是伺服器層級主體的登入必須是 dbmanager 角色的成員才能複製資料庫。如需登入與連接到伺服器的詳細資訊，請參閱[管理登入](sql-database-manage-logins.md)。

使用 [CREATE DATABASE](https://msdn.microsoft.com/library/ms176061.aspx) 陳述式，開始複製來源資料庫。執行此陳述式會起始資料庫複製程序。因為複製資料庫是非同步程序，所以 CREATE DATABASE 陳述式會在資料庫完成複製之前傳回。


### 將 SQL Database 複製到相同伺服器

使用伺服器層級主體登入或建立您要複製之資料庫的登入來登入 master 資料庫。不是伺服器層級主體的登入必須是 dbmanager 角色的成員才能複製資料庫。

此命令會將 Database1 複製到相同伺服器上名為 Database2 的新資料庫。視資料庫大小而定，複製作業可能需要一些時間才能完成。

    -- Execute on the master database.
    -- Start copying.
    CREATE DATABASE Database1_copy AS COPY OF Database1;

### 將 SQL Database 複製到不同伺服器

登入目的地伺服器的 master 資料庫，也就是即將建立新資料庫的 Azure SQL Database 伺服器。使用具有與來源 Azure SQL Database 伺服器上來源資料庫的資料庫擁有者 (DBO) 相同之名稱和密碼的登入。目的地伺服器上的登入必須也是 dbmanager 角色的成員或是伺服器層級主體登入。

此命令會將 server1 上的 Database1 複製到 server2 上名為 Database2 的新資料庫。視資料庫大小而定，複製作業可能需要一些時間才能完成。


    -- Execute on the master database of the target server (server2)
    -- Start copying from Server1 to Server2
    CREATE DATABASE Database1_copy AS COPY OF server1.Database1;
    

## 監視複製作業的進度

藉由查詢 sys.databases 和 sys.dm\_database\_copies 檢視來監視複製程序。當複製正在進行時，新資料庫的 sys.databases 檢視的 state\_desc 資料行會設定為 COPYING。


- 如果複製失敗，新資料庫的 sys.databases 檢視的 state\_desc 資料行會設定為 SUSPECT。在此情況下，在新的資料庫上執行 DROP 陳述式，稍後再試一次。
- 如果複製成功，新資料庫的 sys.databases 檢視的 state\_desc 資料行會設定為 ONLINE。在此情況下，複製已完成且新資料庫是一般資料庫，能夠與來源資料庫分開進行變更。

> [AZURE.NOTE] - 如果您決定在進行複製時予以取消，請在新資料庫上執行 [DROP DATABASE](https://msdn.microsoft.com/library/ms178613.aspx) 陳述式。或者，在來源資料庫上執行 DROP DATABASE 陳述式也會取消複製程序。


## 在複製作業完成之後解析登入

在新資料庫於目的地伺服器上線之後，使用 [ALTER USER](https://msdn.microsoft.com/library/ms176060.aspx) 陳述式將使用者從新的資料庫重新對應至目的地伺服器上的登入。若要解析被遺棄的使用者，請參閱[被遺棄使用者疑難排解](https://msdn.microsoft.com/library/ms175475.aspx)。另請參閱[如何管理災害復原後的 Azure SQL Database 安全性](sql-database-geo-replication-security-config.md)。

新資料庫中的所有使用者都保有其在來源資料庫中原有的權限。起始資料庫複製的使用者會變成新資料庫的資料庫擁有者，並且被指派新的安全性識別碼 (SID)。在複製成功之後，重新對應其他使用者之前，只有起始複製的登入 (也就是資料庫擁有者 (DBO)) 可以登入新的資料庫。


## 後續步驟

- 如需複製 Azure SQL Database 的概觀，請參閱[複製 Azure SQL Database](sql-database-copy.md)。
- 若要使用 Azure 入口網站來複製資料庫，請參閱[使用 Azure 入口網站複製 Azure SQL Database](sql-database-copy-portal.md)。
- 若要使用 PowerShell 複製資料庫，請參閱[使用 PowerShell 複製 Azure SQL Database](sql-database-copy-powershell.md)。
- 請參閱[如何管理災害復原後的 Azure SQL Database 安全性](sql-database-geo-replication-security-config.md)，以了解如何在將資料庫複製到不同的邏輯伺服器時管理使用者與登入。



## 其他資源

- [管理登入](sql-database-manage-logins.md)
- [使用 SQL Server Management Studio 連接到 SQL Database 並執行範例 T-SQL 查詢](sql-database-connect-query-ssms.md)
- [將資料庫匯出至 BACPAC](sql-database-export.md)
- [商務持續性概觀](sql-database-business-continuity.md)
- [SQL Database 文件](https://azure.microsoft.com/documentation/services/sql-database/)

<!---HONumber=AcomDC_0921_2016-->