<properties
   pageTitle="Azure SQL Database Azure 案例研究 - Daxko/CSI| Microsoft Azure"
   description="了解 Daxko/CSI 如何使用 SQL Database 來加速其開發週期及提升其客戶服務和效能"
   services="sql-database"
   documentationCenter=""
   authors="carlrabeler"
   manager="jhubbard"
   editor=""/>

<tags
   ms.service="sql-database"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="09/08/2016"
   ms.author="carlrab"/>
   
# Daxko/CSI 使用 Azure 來加速其開發週期及提升其客戶服務和效能

![標誌](./media/sql-database-case-study-daxko/csidaxkologo25.png)

Daxko/CSI Software 面臨挑戰︰歸功於其全方位企業軟體解決方案的成功，其體適能和娛樂中心的客戶群快速成長，但是要跟上該不斷成長的客戶群的 IT 基礎結構需求則考驗著該公司的 IT 人員。該公司逐漸受到增加的作業額外負荷限制，特別是在管理日益成長的資料庫方面。更糟的是，這些作業額外負荷已佔用新計畫 (例如該公司軟體的新行動功能) 的部分開發資源。

根據 Daxko/CSI 的產品開發主任 David Molina 所述，Azure 為 CSI Software 提供了它在簡化資料庫管理、增加延展性以及釋出資源來專注於軟體而非作業等方面，所需要的平台即服務 (PaaS) 模型。「Azure SQL Database 是非常適合我們的選項。不需要操心維護 SQL Server、容錯移轉叢集，以及所有其他基礎結構需求，對我們來說相當理想。」

由於移轉至 Azure，因此 CSI Software 只需要兩位作業人員來管理超過 600 個客戶資料庫。該公司使用 Azure SQL Database 彈性集區，根據大小和需求來移動客戶資料庫。

Molina 繼續說道：「我們的客戶立即感受到改變。在使用彈性集區之前，他們在高載期間有時會遇到逾時和其他問題。在使用 Azure 彈性集區之後，他們可以視需要激增用量，而且使用軟體也沒有任何問題。」

除了改善客戶的效能之外，Azure 彈性資料庫集區也釋出了 CSI Software 資源，以專注於開發新的服務和功能，而不是處理作業和管理。這些 IT 資源協助了 CSI Software 改進其企業軟體方案 SpectrumNG，以幫助吸引健身房會員參與互動、提升員工效率，以及讓員工和會員能夠透過行動裝置存取互動式工作和即時通知。

此外，Azure 也透過啟用自動化選項，協助 CSI Software 加速及改進開發和品質保證 (QA) 週期。透過該公司的 Azure 實作，建置管理員只要按一下按鈕，便能封裝元件。如 Molina 所述：「QA 現在能夠於發行週期內，在 Azure 中部署到精確模擬生產環境堆疊的測試環境。我們可以將組建立即部署到開發環境來檢查變更。這對我們而言是一大優點，因為之前我們沒有類似的環境可供進行測試。」

## 移轉至雲端

在移到雲端之前，CSI Software 已在其休士頓當地的資料中心成功建立自己的多租用戶基礎結構。隨著公司的擴大，它在採購、佈建及維護用以支援其客戶所需的所有軟硬體方面，面臨了不斷增加難題。增補 IT 人員來處理作業成為另一個瓶頸，導致拖慢佈建新資源及向客戶推出新服務的速度。

為了消除該額外負荷，CSI Software 研究了雲端選項，以便能夠將焦點放在程式碼，而不是作業。該公司發現許多頂尖的雲端提供者都只提供基礎結構即服務 (IaaS) 解決方案，而這種解決方案仍然需要大量 IT 人員來管理 IaaS 堆疊。最後，CSI Software 判斷 Azure PaaS 解決方案是最符合其需求的選項。Molina 解釋道：「Azure 為我們排除了硬體和系統軟體的障礙，讓我們可以專注於我們的軟體方案，同時也減少我們 IT 額外負荷。」

## 轉換至 Azure

在選取 Azure 作為其 PaaS 解決方案之後，CSI Software 便開始將其後端基礎結構和資料庫移轉至雲端。在使用 Azure 之前，SpectrumNG 客戶必須安裝用戶端應用程式，才能與位於後端的 Windows Communication Foundation (WCF) 服務進行通訊。Molina 表示：「雖然有些客戶將所有項目都裝載在自己的資料中心內，但是我們是將產品建置成多租用戶產品。我們使用 SQL Server 作為資料存放區，將所有項目都裝載在休士頓的資料中心內。」

「我們的產品方案也包含一個成員面向入口網站 (以 ASP.net 撰寫)，其設計目的是要以白標籤方式對應客戶的網站空間；還包含一個 SOAP API，可用來支援線上頁面和任何協力廠商整合。」

將架構移轉至雲端不需花費很長的時間。Molina 表示：「大部分的心力是用在修改我們讀取組態檔資訊的方式、集中的連接字串修改，以及將發行版本的封裝、上傳和部署自動化。」

為了開發組建自動化程序，CSI Software 工程師使用了 Azure PowerShell 和 REST API 來建立套件並將它們上傳到預備環境，以供每晚發行。轉換至 Azure 雲端型部署的整體過程對 CSI Software IT 團隊而言，相當快速且順暢。Molina 解釋道：「總言之，我們在承接專案的三到四週內，在雲端就有了 Beta 環境。對我們而言是一項意外的勝利。」

在設定並測試環境之後，CSI Software 便開始移轉客戶。對於已經在使用 CSI Software 託管服務的客戶來說，這項轉換幾乎是無縫接軌。對於從內部部署環境進行移轉的客戶來說，移轉至雲端需要多花一些時間，但就客戶和 CSI Software 而言，根本上仍然都相當輕鬆即可完成。

針對新客戶，CSI Software 的 IT 人員會使用下列程序讓他們在 Azure 上線：

1.	Azure PowerShell 指令碼會用來為客戶啟動新資料庫；所有客戶都是從進階層開始，以確保有足夠的初始輸送量來進行轉換。
2.	如果可能，CSI Software 會使用「Azure SQL 移轉精靈」將現有的資料移至 Azure SQL 資料庫執行個體。
3.	最後，Microsoft SQL Server Integration Services (SSIS) 會用來協調資料中任何不一致的地方，或視需要執行任何資料清除。

現在，大約 99% 的 CSI Software 都是裝載在 Azure 中，遍及四個區域資料中心 (中北部、中南部、東部及西部)。透過在每個客戶的地理區域中都有資料中心，將延遲降到了最低。

## Azure 彈性資料庫集區釋出 IT 資源

Azure 的數個功能協助了 CSI Software 從以基礎結構和作業為焦點，轉換成以功能和開發為焦點。或許最大的好處是來自彈性資料庫集區。

CSI Software 目前為客戶提供大約 550 個資料庫。在使用彈性集區之前，很難在一層結構內管理這麼多資料庫。作業管理員必須根據客戶的高載需求指派效能層，而這會造成嚴重的 IT 資源額外負荷。透過彈性資料庫集區，管理員則可以視情況為租用戶指派高階或標準集區，然後再根據大小和需求來移動客戶。客戶幾乎是立即感受到彈性資料庫集區的效果；在使用彈性集區之前，客戶在高載使用量期間遇到了逾時和其他問題，但在使用彈性集區之後，客戶可以視需要體驗活動高載情況，而且繼續使用 SpectrumNG 也沒有任何問題。

## Azure 作用中異地複寫加速報表處理速度

有數個 CSI Software 客戶也利用「Azure 作用中異地複寫」功能。使用「作用中異地複寫」時，可在相同或不同的資料中心區域中，最多設定 4 個可讀取的次要資料庫。CSI Software 以兩種方式利用「作用中異地複寫」︰首先是在資料中心中斷服務或在無法連線至主要資料庫時，可以使用次要資料庫；其次是可以讀取次要資料庫並用來卸載唯讀工作負載，例如報告工作。有些 CSI Software 客戶會利用這個優點來加速報告處理工作流程。

## CSI Software 應用程式邏輯與架構

SpectrumNG 使用 Web 角色。由於應用程式是多租用戶應用程式，因此會使用 WCF 服務來處理來自客戶的初始連線要求。如 Molina 所述：「要求會識別每一個客戶，然後這會讓我們建置出一個可連至其資料庫的連接字串，以執行我們所需執行的任何動作。」

針對此服務的 Web 層，CSI Software 會根據日期與時間，利用 Azure 自動調整功能。根據每個區域資料中心的時區，在營業時間內，可用資源會自動增加來配合較高的使用量。資源也會設為在週末客戶需求降低時進行縮減。

     
![圖 1](./media/sql-database-case-study-daxko/figure1.png)

圖 1.雲端服務背景工作角色會從 Azure SQL Database 抽取結構化資料，以及從資料表儲存體抽取半結構化資料。SpectrumNG 使用者會透過雲端服務 Web 角色與該資料進行互動。

## 將 Web 應用程式和 Web 方案層用於行動應用程式

使用 Azure SQL Database 會釋出資源，讓 CSI Software 能夠進行新計畫，包括以裝載於 Azure Web 應用程式中的自訂 API 為基礎的完整行動平台。此平台可讓健身房會員及員工使用行動裝置來查看排程、預約課程及接收訊息。

此平台使用服務導向架構 (SOA) 來擷取單一元件 (例如銷售點系統 (POS) 或銷售系統)、快速將它移到另一個 Web 方案，然後啟動服務來支援該元件，同時又讓所有其他項目保留在原來的 Web 方案中。此功能為 CSI Software 提供了極大的彈性，並且有助於降低成本。

## Azure 讓 CSI Software 開發人員專注於應用程式和服務

Azure SQL Database 不僅為 SpectrumNG 客戶帶來好處，讓他們享有快速、可靠的服務，也為 CSI Software 的 IT 人員和開發人員帶來豐碩的成果。透過將作業卸載到雲端中的 Azure，CSI Software 不僅降低了其資源和基礎結構的額外負荷、大幅加速其開發週期，而且也不再需要對資料庫進行微觀管理來將其租用戶的效能最佳化。

## 詳細資訊

- 若要深入了解 Azure 彈性資料庫集區，請參閱[彈性資料庫集區](sql-database-elastic-pool.md)。

- 若要深入了解資料庫工具和彈性調整，請參閱[彈性資料庫工具和彈性調整](sql-database-elastic-scale-get-started.md)。

- 若要深入了解如何移轉 SQL Server 資料庫，請參閱 [Azure SQL 移轉精靈](sql-database-cloud-migrate-compatible-using-ssms-migration-wizard.md)。

- 若要深入了解主動式異地複寫，請參閱[主動式異地複寫](sql-database-geo-replication-overview.md)。

- 若要深入了解 Web 角色和背景工作角色，請參閱[背景工作角色](../fundamentals-introduction-to-azure.md#compute)。

- 若要深入了解「Azure 服務匯流排」，請參閱 [Azure 服務匯流排](https://azure.microsoft.com/services/service-bus/)。

- 若要深入了解自動調整，請參閱[調整雲端服務](../cloud-services/cloud-services-how-to-scale.md)。

<!---HONumber=AcomDC_0921_2016-->