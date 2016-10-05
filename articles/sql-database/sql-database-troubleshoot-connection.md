<properties
	pageTitle="目前無法使用伺服器上的資料庫，連接到 SQL Database | Microsoft Azure"
	description="針對應用程式連接到 SQL Database 時發生的「目前無法使用伺服器上的資料庫」錯誤進行。"
	services="sql-database"
	documentationCenter=""
	authors="dalechen"
	manager="felixwu"
	editor=""
	keywords="目前無法使用伺服器上的資料庫，連接到 SQL Database"/>

<tags
	ms.service="sql-database"
	ms.workload="data-management"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/21/2016"
	ms.author="daleche"/>

# 連接到 SQL Database 時發生「目前無法使用伺服器上的資料庫」錯誤
[AZURE.INCLUDE [支援免責聲明](../../includes/support-disclaimer.md)]

當應用程式連接到 Azure SQL Database 時，您會收到下列錯誤訊息︰

```
Error code 40613: "Database <x> on server <y> is not currently available. Please retry the connection later. If the problem persists, contact customer support, and provide them the session tracing ID of <z>"
```

> [AZURE.NOTE] 此錯誤訊息是通常是暫時性 (短期)。

當移動 (或重新設定) Azure 資料庫時會發生此錯誤，而且您的應用程式會失去與 SQL Database 的連接。SQL Database 重新設定事件由於規劃的事件 (例如，軟體升級) 或未規劃的事件 (例如，處理序損毀或負載平衡) 而發生。大部分的重新設定事件通常只是短期的，至多應在不到 60 秒的時間完成。不過，這些事件可能偶爾會需要更長時間才能完成，例如當大型交易導致長時間執行的復原時。

## 解決暫時性連線問題的步驟
1.	檢查 [Microsoft Azure 服務儀表板](https://azure.microsoft.com/status)，以取得應用程式報告錯誤期間發生的任何已知中斷。
2. 連接到雲端服務的應用程式 (例如 Azure SQL Database) 應該預期定期的重新設定事件，並實作應用程式重試邏輯來處理這些錯誤，而不是將這些錯誤當做應用程式錯誤呈現給使用者。如需詳細資訊和一般重試策略，請檢閱[暫時性錯誤](sql-database-connectivity-issues.md)一節以及 [SQL Database 開發概觀](sql-database-develop-overview.md)的最佳作法和設計指南。如需詳細資訊，請接著參閱[適用於 SQL Database 和 SQL Server 的連線庫](sql-database-libraries.md)的程式碼範例 。
3.	由於資料庫接近其資源限制，因此似乎是暫時性連線問題。請參閱[移難排解效能問題](sql-database-troubleshoot-performance.md)。
4.	如果連線問題繼續發生，或如果您的應用程式發生錯誤的持續時間超過 60 秒，或如果您在一天當中，看到錯誤多次發生，請在 [Azure 支援](https://azure.microsoft.com/support/options)網站上選取 [取得支援]，來提出 Azure 支援要求。

## 後續步驟
- 如果您收到不同的錯誤，請評估[錯誤訊息](sql-database-develop-error-messages.md)，找出問題的線索。
- 如果問題持續發生，請造訪[針對 Azure SQL Database 常見的連接問題進行疑難排解](sql-database-troubleshoot-common-connection-issues.md)中的指引。

<!---HONumber=AcomDC_0921_2016-->