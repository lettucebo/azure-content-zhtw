<properties
	pageTitle="使用 Azure 入口網站複製 Azure SQL Database | Microsoft Azure"
	description="建立 Azure SQL Database 的複本"
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



# 使用 Azure 入口網站複製 Azure SQL Database

> [AZURE.SELECTOR]
- [概觀](sql-database-copy.md)
- [Azure 入口網站](sql-database-copy-portal.md)
- [PowerShell](sql-database-copy-powershell.md)
- [T-SQL](sql-database-copy-transact-sql.md)

下列步驟說明如何利用 [Azure 入口網站](https://portal.azure.com)，將 SQL Database 複製到相同的伺服器或不同的伺服器。

若要複製 SQL Database，您需要下列項目：

- Azure 訂用帳戶。如果需要 Azure 訂用帳戶，可以先按一下此頁面頂端的 [免費試用]，然後再回來完成這篇文章。
- 要複製的 SQL Database。如果沒有 SQL Database，請遵循本文中以下的步驟：[建立您的第一個 Azure SQL Database](sql-database-get-started.md)。


## 複製您的 SQL Database

針對您要複製的資料庫開啟 SQL Database 頁面：

1.	移至 [Azure 入口網站](https://portal.azure.com)。
2.	按一下 [更多服務] > [SQL Database]，然後按一下想要的資料庫。
3.	在 [SQL Database] 頁面上，按一下 [複製]：

    ![SQL Database](./media/sql-database-copy-portal/sql-database-copy.png)

1.  在 [複製] 頁面上，系統會提供預設的資料庫名稱。視需要輸入不同的名稱 (伺服器上所有的資料庫都必須有一個唯一的名稱)。
2.  選取 [目標伺服器]。目標伺服器就是要建立資料庫副本的位置。您可以將資料庫複製到相同的伺服器或不同的伺服器。您可以建立伺服器，或從清單中選取現有的伺服器。
3.  選取 [目標伺服器] 之後，將會啟用 [彈性資料庫集區] 和 [定價層] 選項。如果伺服器有集區，您便可以將資料庫複製到該集區。
3.  按一下 [確定] 開始複製程序。

    ![SQL Database](./media/sql-database-copy-portal/copy-page.png)


## 監視複製作業的進度

- 開始複製之後，按一下入口網站通知以取得詳細資訊。

    ![通知][3]
 
    ![monitor][4]


## 確認伺服器上的資料庫為線上狀態

- 按一下 [更多服務] > [SQL Database]，然後確認新資料庫為 [線上] 狀態。


## 解析登入

若要在複製作業完成之後解析登入，請參閱[解析登入](sql-database-copy-transact-sql.md#resolve-logins-after-the-copy-operation-completes)


## 後續步驟

- 如需複製 Azure SQL Database 的概觀，請參閱[複製 Azure SQL Database](sql-database-copy.md)。
- 若要使用 PowerShell 複製資料庫，請參閱[使用 PowerShell 複製 Azure SQL Database](sql-database-copy-powershell.md)。
- 若要使用 Transact-SQL 複製資料庫，請參閱[使用 T-SQL 複製 Azure SQL Database](sql-database-copy-transact-sql.md)。
- 請參閱[如何管理災害復原後的 Azure SQL Database 安全性](sql-database-geo-replication-security-config.md)，以了解如何在將資料庫複製到不同的邏輯伺服器時管理使用者與登入。



## 其他資源

- [管理登入](sql-database-manage-logins.md)
- [使用 SQL Server Management Studio 連接到 SQL Database 並執行範例 T-SQL 查詢](sql-database-connect-query-ssms.md)
- [將資料庫匯出至 BACPAC](sql-database-export.md)
- [商務持續性概觀](sql-database-business-continuity.md)
- [SQL Database 文件](https://azure.microsoft.com/documentation/services/sql-database/)




<!--Image references-->
[1]: ./media/sql-database-copy-portal/copy.png
[2]: ./media/sql-database-copy-portal/copy-ok.png
[3]: ./media/sql-database-copy-portal/copy-notification.png
[4]: ./media/sql-database-copy-portal/monitor-copy.png

<!---HONumber=AcomDC_0921_2016-->