<properties
	pageTitle="監視 Azure SQL Database 中的資料庫效能 | Microsoft Azure"
	description="了解使用 Azure 工具和動態管理檢視來監視您的資料庫的選項。"
	keywords="資料庫監視, 雲端資料庫效能"
	services="sql-database"
	documentationCenter=""
	authors="CarlRabeler"
	manager="jhubbard"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.devlang="na"
	ms.topic="get-started-article"
	ms.tgt_pltfrm="na"
	ms.workload="data-management"
	ms.date="09/27/2016"
	ms.author="carlrab"/>

# 監視 Azure SQL Database 中的資料庫效能
在 Azure 中監視 SQL Database 的效能，必須從監視您選擇之資料庫效能等級相關的資源使用率開始。監視可協助您判斷您的資料庫是否有超量的容量或因為資源超量而發生問題，然後判斷是否開始調整您資料庫的效能等級和[服務層](sql-database-service-tiers.md)。您可以使用 [Azure 入口網站](https://portal.azure.com)中的圖形化工具或使用 SQL [動態管理檢視](https://msdn.microsoft.com/library/ms188754.aspx)，監視您的資料庫。

## 使用 Azure 入口網站監視資料庫

在 [Azure 入口網站](https://portal.azure.com/)中，您可以透過選取資料庫，並按一下 [監視] 圖表，監視單一資料庫的使用率。如此會帶出您可變更的 [度量] 視窗，只要按一下 [編輯圖表] 按鈕即可。新增下列度量：

- CPU 百分比
- DTU 百分比
- 資料 IO 百分比
- 資料庫大小百分比

新增這些度量之後，您可以在 [**度量**] 視窗中，在含有詳細資料的 [**監視**] 圖表中繼續檢視它們。這四個度量都會顯示與資料庫 **DTU** 相對的平均使用率百分比。如需 DTU 的詳細資訊，請參閱[服務層](sql-database-service-tiers.md)文章。

![資料庫效能的服務層監視。](./media/sql-database-service-tiers/sqldb_service_tier_monitoring.png)

您也可以在效能度量中設定警示。按一下 [**度量**] 視窗中的 [**新增警示**] 按鈕。遵循精靈的指示以設定警示。您可以選擇在度量超出或低於特定臨界值時發出警示。

例如，如果您預期資料庫中的工作負載會成長，可以選擇設定電子郵件警示，以便在資料庫的任何效能度量達到 80% 時收到警示。您可以使用此警示做為早期警告，協助您判斷何時需要切換至更高的效能層級。

效能度量也可協助您判斷您是否能夠降級至較低的效能層級。假設您使用標準 S2 資料庫，且所有效能度量皆顯示在指定時間內，資料庫平均使用率沒有超過 10%。則該資料庫可能在標準 S1 中會運作得不錯。不過，在您決定將資料庫移至較低效能層級前，請先注意暴增或大幅變動的工作負載。

## 使用 DMV 監視資料庫

您也可以透過下列系統檢視取得公開在入口網站中的相同度量：伺服器的邏輯**主**資料庫中的 [sys.resource\_stats](https://msdn.microsoft.com/library/dn269979.aspx)，以及使用者資料庫中的 [sys.dm\_db\_resource\_stats](https://msdn.microsoft.com/library/dn800981.aspx)。如果您需要在一段長時間內監視較不細微的資料，請使用 **sys.resource\_stats**。如果您需要在較小的時間範圍內監視較細微的資料，請使用 **sys.dm\_db\_resource\_stats**。如需詳細資訊，請參閱 [Azure SQL Database 效能指引](sql-database-performance-guidance.md#monitoring-resource-use-with-sysresourcestats)。

>[AZURE.NOTE] 使用於 Web 和 Business Edition 資料庫 (已淘汰) 時，**sys.dm\_db\_resource\_stats** 會傳回空的結果集。

若為彈性資料庫集區，您可以監視其中的個別資料庫，技巧如本節所述。但您也可以監視集區整體。如需詳細資訊，請參閱[監視和管理彈性資料庫集區](sql-database-elastic-pool-manage-portal.md)。

<!---HONumber=AcomDC_0928_2016-->