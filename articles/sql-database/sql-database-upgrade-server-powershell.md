<properties
	pageTitle="使用 PowerShell 升級至 Azure SQL Database V12 | Microsoft Azure"
	description="說明如何使用 PowerShell 升級至 Azure SQL Database V12，包括如何升級 Web 和商務資料庫，以及如何將 V11 伺服器的資料庫直接移轉至彈性資料庫集區來升級 V11 伺服器。"
	services="sql-database"
	documentationCenter=""
	authors="stevestein"
	manager="jhubbard"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.workload="data-management"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/19/2016"
	ms.author="sstein"/>

# 使用 PowerShell 升級至 Azure SQL Database V12


> [AZURE.SELECTOR]
- [Azure 入口網站](sql-database-upgrade-server-portal.md)
- [PowerShell](sql-database-upgrade-server-powershell.md)


SQL Database V12 是最新的版本，因此建議升級至 SQL Database V12。SQL Database V12 具有[舊版所欠缺的許多優點](sql-database-v12-whats-new.md)，包括：

- 提升與 SQL Server 的相容性。
- 提供改進的高階效能和新的效能等級。
- [彈性資料庫集區](sql-database-elastic-pool.md)。

本文提供將現有的 SQL Database V11 伺服器和資料庫升級至 SQL Database V12 的說明。

在升級至 V12 的過程中，您會將所有 Web 和商務資料庫都升級至新的服務層級，因此本文也包含了升級 Web 和商務資料庫的說明。

此外，與升級至單一資料庫的個別效能等級 (定價層) 相比，移轉至[彈性資料庫集區](sql-database-elastic-pool.md)更符合成本效益。集區也可以簡化資料庫管理，因為您只需要管理集區的效能設定，而不需分開管理個別資料庫的效能等級。如果您的資料庫位於多部伺服器上，請考慮將它們移到相同的伺服器，並利用將它們放入集區所帶來的優點。

只要依照本文中的步驟操作，您就可以輕鬆地將資料庫從 V11 伺服器直接移轉至彈性資料庫集區。

請注意，您的資料庫會維持在線上，並且在整個升級作業中都會繼續保持運作。在實際轉換到新的效能等級時，資料庫連線可能會暫時中斷一段非常短的時間，通常約 90 秒，但最長可達 5 分鐘。如果您的應用程式[對於連線終止有暫時性的錯誤處理方式](sql-database-connectivity-issues.md)，就足以在升級結束時防止連線中斷。

升級至 SQL Database V12 後即無法復原。在升級之後，即無法將伺服器還原至 V11。

升級至 V12 之後，將不會立即提供[服務層級建議](sql-database-service-tier-advisor.md)和[彈性集區建議](sql-database-elastic-pool-create-portal.md)，必須等到服務有時間評估您在新伺服器上的工作負載之後，才會提供。V11 伺服器建議記錄不適用於 V12 伺服器，因此不會保留。

## 準備升級

- **升級所有 Web 和商務資料庫**：請參閱下面的[升級所有 Web 和商務資料庫](sql-database-v12-upgrade.md#upgrade-all-web-and-business-databases)一節，或使用 [PowerShell 來升級資料庫和伺服器](sql-database-upgrade-server-powershell.md)。
- **檢閱和暫停異地複寫**：如果您的 Azure SQL Database 已針對「異地複寫」做設定，您應該記錄其目前的設定並[停止異地複寫](sql-database-geo-replication-portal.md#remove-secondary-database)。在升級完成後，請針對「異地複寫」重新設定您的資料庫。
- **如果您的用戶端位於 Azure VM 上，請開啟這些連接埠**：如果在您的用戶端於 Azure 虛擬機器 (VM) 上執行時，用戶端程式連接至 SQL Database V12，您就必須開啟此 VM 上 11000-11999 和 14000-14999 範圍的連接埠。如需詳細資訊，請參閱 [SQL Database V12 的連接埠](sql-database-develop-direct-route-ports-adonet-v12.md)。


## 必要條件

若要使用 PowerShell 將伺服器升級至 V12，您必須安裝並執行最新的 Azure PowerShell。如需詳細資訊，請參閱[如何安裝和設定 Azure PowerShell](../powershell-install-configure.md)。


## 設定您的認證並選取您的訂用帳戶

若要針對您的 Azure 訂用帳戶執行 PowerShell Cmdlet，您必須先建立對您 Azure 帳戶的存取權。請執行下列命令，然後您就會看到可供輸入認證的登入畫面。請使用與登入 Azure 入口網站相同的電子郵件和密碼。

	Add-AzureRmAccount

成功登入後，您應該會在畫面中看到一些資訊，包括您用來登入的 ID 及您可存取的 Azure 訂用帳戶。

若要選取要使用的訂用帳戶，您必須提供訂用帳戶 ID (**-SubscriptionId**) 或訂用帳戶名稱 (**-SubscriptionName**)。您可以複製上一個步驟中的資訊，或者，如果您有多個訂用帳戶，則可以執行 **Get-AzureRmSubscription** Cmdlet，然後從結果集中複製所需的訂用帳戶資訊。

使用您的訂用帳戶資訊執行下列 Cmdlet，以設定您目前的訂用帳戶：

	Set-AzureRmContext -SubscriptionId 4cac86b0-1e56-bbbb-aaaa-000000000000

下列命令會針對您剛才在上方選取的訂用帳戶執行。

## 取得建議

若要取得伺服器升級建議，請執行下列 Cmdlet：

    $hint = Get-AzureRmSqlServerUpgradeHint -ResourceGroupName “resourcegroup1” -ServerName “server1”

如需詳細資訊，請參閱[建立彈性資料庫集區](sql-database-elastic-pool-create-portal.md)和 [Azure SQL Database 定價層建議](sql-database-service-tier-advisor.md)。



## 開始升級

若要開始升級伺服器，請執行下列 Cmdlet：

    Start-AzureRmSqlServerUpgrade -ResourceGroupName “resourcegroup1” -ServerName “server1” -ServerVersion 12.0 -DatabaseCollection $hint.Databases -ElasticPoolCollection $hint.ElasticPools  


當您執行此命令時，升級程序將會開始。您可以自訂建議的輸出，並提供此 Cmdlet 的已編輯建議。


## 升級伺服器


    # Adding the account
    #
    Add-AzureRmAccount

    # Setting the variables
    #
    $SubscriptionName = 'YOUR_SUBSCRIPTION'
    $ResourceGroupName = 'YOUR_RESOURCE_GROUP'
    $ServerName = 'YOUR_SERVER'

    # Selecting the right subscription
    #
    Set-AzureRmContext -SubscriptionName $SubscriptionName

    # Getting the upgrade recommendations
    #
    $hint = Get-AzureRmSqlServerUpgradeHint -ResourceGroupName $ResourceGroupName -ServerName $ServerName

    # Starting the upgrade process
    #
    Start-AzureRmSqlServerUpgrade -ResourceGroupName $ResourceGroupName -ServerName $ServerName -ServerVersion 12.0 -DatabaseCollection $hint.Databases -ElasticPoolCollection $hint.ElasticPools  


## 自訂升級對應

如果建議不適合您的伺服器和商務案例，您可以選擇資料庫的升級方式，並將它們對應到單一或彈性資料庫。

ElasticPoolCollection 和 DatabaseCollection 參數都是選擇性項目：

    # Creating elastic pool mapping
    #
    $elasticPool = New-Object -TypeName Microsoft.Azure.Management.Sql.Models.UpgradeRecommendedElasticPoolProperties
    $elasticPool.DatabaseDtuMax = 100
    $elasticPool.DatabaseDtuMin = 0
    $elasticPool.Dtu = 800
    $elasticPool.Edition = "Standard"
    $elasticPool.DatabaseCollection = ("DB1", “DB2”, “DB3”, “DB4”)
    $elasticPool.Name = "elasticpool_1"
    $elasticPool.StorageMb = 800

    # Creating single database mapping for 2 databases. DBMain1 mapped to S0 and DBMain2 mapped to S2
    #
    $databaseMap1 = New-Object -TypeName Microsoft.Azure.Management.Sql.Models.UpgradeDatabaseProperties
    $databaseMap1.Name = "DBMain1"
    $databaseMap1.TargetEdition = "Standard"
    $databaseMap1.TargetServiceLevelObjective = "S0"

    $databaseMap2 = New-Object -TypeName Microsoft.Azure.Management.Sql.Models.UpgradeDatabaseProperties
    $databaseMap2.Name = "DBMain2"
    $databaseMap2.TargetEdition = "Standard"
    $databaseMap2.TargetServiceLevelObjective = "S2"

    # Starting the upgrade
    #
    Start-AzureRmSqlServerUpgrade –ResourceGroupName resourcegroup1 –ServerName server1 -ServerVersion 12.0 -DatabaseCollection @($databaseMap1, $databaseMap2) -ElasticPoolCollection @($elasticPool)



## 在升級至 SQL Database V12 後監視資料庫


升級之後，建議您主動監視資料庫，以確保應用程式達到所需的執行效能，並視需要將使用情況調整到最佳狀態。

除了監視個別的資料庫之外，您也可以[使用入口網站](sql-database-elastic-pool-manage-portal.md)或藉由 [PowerShell](sql-database-elastic-pool-manage-powershell.md) 監視彈性資料庫集區。


**資源耗用量資料：**Basic、Standard 及 Premium 資料庫的資源耗用量資料會透過使用者資料庫中的 [sys.dm_ db_ resource\_stats](http://msdn.microsoft.com/library/azure/dn800981.aspx) DMV 提供。此 DMV 以 15 秒的間隔提供幾乎即時的前一小時作業資源耗用量資訊。某一間隔的 DTU 百分比耗用量會以 CPU、IO 及記錄檔方面的最大百分比耗用量來計算。下列是計算前一小時之平均 DTU 百分比耗用量的查詢：

    SELECT end_time
    	 , (SELECT Max(v)
             FROM (VALUES (avg_cpu_percent)
                         , (avg_data_io_percent)
                         , (avg_log_write_percent)
    	   ) AS value(v)) AS [avg_DTU_percent]
    FROM sys.dm_db_resource_stats
    ORDER BY end_time DESC;

其他監視資訊：

- [單一資料庫的 Azure SQL Database 效能指引](http://msdn.microsoft.com/library/azure/dn369873.aspx)。
- [彈性資料庫集區的價格和效能考量](sql-database-elastic-pool-guidance.md)。
- [使用動態管理檢視監視 Azure SQL Database](sql-database-monitoring-with-dmvs.md)



**警示：**在 Azure 入口網站中設定「警示」，即可在已升級之資料庫的 DTU 耗用量接近特定的高層級時通知您。您可以在 Azure 入口網站中為各種效能計量 (例如 DTU、CPU、IO 及記錄檔) 設定資料庫警示。請瀏覽至您的資料庫，然後在 [設定] 刀鋒視窗中，選取 [警示規則]。

例如，您可以設定若過去 5 分鐘的平均 DTU 百分比值超出 75% 則發出「 DTU 百分比 」電子郵件警示。若要深入了解如何設定警示通知，請參閱[接收警示通知](../azure-portal/insights-receive-alert-notifications.md)。



## 後續步驟

- [建立彈性資料庫集區](sql-database-elastic-pool-create-portal.md)，並將部分或全部資料庫新增至集區。
- [變更您資料庫的服務層級和效能等級](sql-database-scale-up.md)。



## 相關資訊

- [Get-AzureRmSqlServerUpgrade](https://msdn.microsoft.com/library/azure/mt603582.aspx)
- [Start-AzureRmSqlServerUpgrade](https://msdn.microsoft.com/library/azure/mt619403.aspx)
- [Stop-AzureRmSqlServerUpgrade](https://msdn.microsoft.com/library/azure/mt603589.aspx)

<!---HONumber=AcomDC_0921_2016-->