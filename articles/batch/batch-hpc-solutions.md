<properties
   pageTitle="雲端中的 Batch 和 HPC 解決方案 | Microsoft Azure"
   description="了解批次和高效能計算 (HPC 和 Big Compute) 案例，以及在 Azure 中的解決方案選項"
   services="batch, virtual-machines, cloud-services"
   documentationCenter=""
   authors="dlepow"
   manager="timlt"
   editor=""/>

<tags
   ms.service="batch"
   ms.devlang="NA"
   ms.topic="get-started-article"
   ms.tgt_pltfrm="NA"
   ms.workload="big-compute"
   ms.date="07/27/2016"
   ms.author="danlep"/>

# Azure 雲端中的 Batch 和 HPC 解決方案

Azure 針對批次和高效能運算 (HPC) 提供有效、可調整的雲端解決方案 - 又稱為「Big Compute」。在這裡了解 Big Compute 工作負載和支援它們的 Azure 的服務，或是直接跳至本文稍後的[解決方案案例](#scenarios)。本文主要適用於技術決策者、IT 經理和獨立軟體廠商，但其他 IT 專業人員和開發人員也可以使用它來熟悉這些解決方案。

組織有大規模的運算問題：工程設計和分析、影像轉譯、複雜模型、Monte Carlo 模擬和財務風險計算等等。Azure 可使用組織所需的資源、級別和排程來協助組織解決這些問題。有了 Azure，組織就可以：

* 建立混合式解決方案，擴充內部部署 HPC 叢集將尖峰工作負載卸載到雲端

* 完全在 Azure 中執行 HPC 叢集工具和工作負載

* 使用受管理及可調整的 Azure 服務 (例如[批次](https://azure.microsoft.com/documentation/services/batch/)) 以執行大量運算工作負載，而不需要部署和管理計算基礎結構

雖然超出本文的範圍，Azure 也提供開發人員和合作夥伴一組完整的功能、架構選擇和開發工具，來建置大型、自訂 Big Compute 工作流程。日益成長的合作夥伴生態系統可協助您在 Azure 雲端，讓您的 Big Compute 工作負載具生產力。


## 批次和 HPC 應用程式

不同於 Web 應用程式和許多特定業務應用程式、批次和 HPC 應用程式擁有既定的開始和結束，而且它們可以排程或隨選執行，有時候數小時或更久。大部分可分成兩個主要類別：「本質平行」(有時稱為「窘迫平行」，因為他們解決的問題能讓他們在多部電腦或處理器上平行執行) 和「緊密結合」。如需這些應用程式類型的詳細資訊，請參閱下表。某些 Azure 解決方案方法比較適合一種類型或其他類型。

>[AZURE.NOTE] 在批次和 HPC 解決方案中，應用程式正在執行的執行個體通常稱為「作業」，而每項作業可能會分成「工作」。且應用程式的叢集計算資源通常稱為「運算節點」。

類型 | 特性 | 範例
------------- | ----------- | ---------------
**本質平行**<br/><br/>![本質平行][parallel] |• 個別的電腦獨立執行應用程式邏輯<br/><br/>• 加入電腦允許應用程式調整並縮短計算時間<br/><br/>• 應用程式包含個別的可執行檔，或是分成用戶端叫用的服務群組 (服務導向的架構、或 SOA、應用程式) |• 財務風險模型<br/><br/>• 影像轉譯和影像處理<br/><br/>• 媒體編碼及轉碼<br/><br/>• Monte Carlo 模擬<br/><br/>• 軟體測試
**緊密結合**<br/><br/>![緊密結合][coupled] |• 應用程式需要進行互動或交換中繼結果的計算節點<br/><br/>• 計算節點可能會使用訊息傳遞介面 (MPI) 進行通訊，MPI 是適用於平行計算的常見通訊協定<br/><br/>• 應用程式很容易受到網路延遲和頻寬影響<br/><br/>• 可以改善應用程式效能，方法是使用支援高速網路技術，例如 InfiniBand 和遠端直接記憶體存取 (RDMA) 的計算基礎結構 |• 石油與天然氣貯存槽模型<br/><br/>• 工程設計和分析，例如：計算流體動力學<br/><br/>• 實體模擬，像是車輛衝撞和核子反應<br/><br/>• 天氣預報

### 在雲端中執行批次和 HPC 應用程式的考量

您可以輕易地移轉針對 Azure 或混合式 (跨單位) 環境設計，在內部部署 HPC 叢集中執行的許多應用程式。不過，可能會有一些限制或考量，包括：


* **雲端資源的可用性** - 根據您使用的雲端計算資源類型，您可能無法在作業執行時保證機器持續運行。狀態處理和進度檢查指向是處理可能的暫時性失敗的常見技巧，且在使用雲端資源時更是不可或缺。


* **資料存取** - 企業叢集中一般會提供的資料存取技術 (例如 NFS) 在雲端時可能需要特殊組態。或者，您可能需要針對雲端採用不同的資料存取作法和模式。

* **資料移動** - 對於處理大量資料的應用程式，需要採行策略將資料移至雲端儲存體和計算資源。您可能需要高速跨單位網路，例如 [Azure ExpressRoute](https://azure.microsoft.com/services/expressroute/)。也請考慮法律、法規或原則對儲存或存取該資料的限制。


* **授權** - 請向廠商確認任何商業應用程式在雲端中的執行授權或其他限制。並非所有廠商都提供隨用隨付授權。您可能需要視您的解決方案，在雲端中規劃授權伺服器，或連接至內部部署授權伺服器。


### 大量運算或巨量資料？

Big Compute 與巨量資料應用程式之間的界限不一定很清楚，而且有些應用程式可能兼具這兩種特性。兩者都涉及執行大規模計算，通常是在電腦的叢集上。但是，解決方案和支援工具可以不同。

• **大量運算**傾向於涉及依賴 CPU 效能和記憶體的應用程式，例如工程模擬、財務風險模型和數位轉譯。Big Compute 解決方案的基礎結構可能包含具有特殊的多核心處理器來執行原始計算，和專用的高速網路硬體來連接電腦的電腦。

• **巨量資料**可解決單一電腦或資料庫管理系統無法管理的大量資料之資料分析問題。範例包括大量 Web 記錄或其他商業智慧資料。巨量資料對於磁碟容量和 I/O 效能的依賴通常更甚於 CPU 能力。另外，巨量資料還有專門的工具 (像是 Apache Hadoop) 來管理叢集和分割資料。(如需 Azure HDInsight 和其他 Azure Hadoop 解決方案的相關資訊，請參閱 [Hadoop](https://azure.microsoft.com/solutions/hadoop/)。)

## 計算管理和作業排程

執行批次和 HPC 應用程式通常包括「叢集管理員」和「作業排程器」來協助管理叢集的計算資源，以及將其配置給執行作業的應用程式。這些函式可能會透過不同的工具或某種整合式工具或服務來完成。

* **叢集管理員** - 佈建、發行及管理計算資源 (或運算節點)。叢集管理員可能會在計算節點上自動安裝作業系統映像和應用程式；根據需求調整計算資源；以及監視節點的效能。

* **作業排程器** - 指定應用程式需要的資源 (例如處理器或記憶體)，以及其會執行的條件。工作排程器會維護工作佇列，並根據指派的優先權或其他特性將資源分配給它們。

以 Windows 為基礎與以 Linux 為基礎的叢集的叢集和作業排程工具，可順暢移轉至 Azure。例如，[Microsoft HPC Pack](https://technet.microsoft.com/library/cc514029)，Microsoft 針對 Windows 和 Linux HPC 工作負載的免費計算叢集解決方案，提供數個在 Azure 中執行的選項。您也可以建置 Linux 叢集來執行開放原始碼工具，例如 Torque 和 SLURM。您也可以將商業格線解決方案帶入 Azure 中，例如 [TIBCO DataSynapse GridServer](http://www.tibco.com/company/news/releases/2016/tibco-to-accelerate-cloud-adoption-of-banking-and-capital-markets-customers-via-microsoft-collaboration)、[IBM Platform Symphony](http://www-01.ibm.com/support/docview.wss?uid=isg3T1023592) 和 [Univa Grid Engine](http://www.univa.com/products/grid-engine)。

如下列各節中所示，您也可以利用 Azure 服務來管理計算資源及排程作業，而不需要 (或同時使用) 傳統叢集管理工具。


## 案例

以下是使用現有的 HPC 叢集解決方案、Azure 服務或兩者的組合，在 Azure 中執行 Big Compute 工作負載的三個常見案例。選擇每個案例的重要考量事項都會列出，但並不詳盡。更多關於您可能會在解決方案中使用的 Azure 服務的相關資訊會於本文稍後提及。

 | 案例 | 為何選擇它？
------------- | ----------- | ---------------
**將 HPC 叢集高載至 Azure**<br/><br/>[![叢集高載][burst_cluster]](./media/batch-hpc-solutions/burst_cluster.png) <br/><br/> 深入了解：<br/>• [使用 Microsoft HPC Pack 高載至 Azure](https://technet.microsoft.com/library/gg481749.aspx)<br/><br/>• [使用 HPC Pack 設定混合式計算叢集](../cloud-services/cloud-services-setup-hybrid-hpcpack-cluster.md)<br/><br/>• [使用 HPC Pack 將量擴大到 Azure Batch](https://technet.microsoft.com/library/mt612877.aspx)<br/><br/>|• 在混合式解決方案中結合您的 [Microsoft HPC Pack](https://technet.microsoft.com/library/cc514029) 或其他內部部署叢集與額外的 Azure 資源。<br/><br/>• 擴充您的 Big Compute 工作負載以在平台即服務 (PaaS) 虛擬機器執行個體上執行 (目前僅限 Windows Server)。<br/><br/>• 藉由使用選擇性 Azure 虛擬網路以存取內部部署授權伺服器或資料存放區。|• 您有現有的 HPC 叢集，並且需要更多資源 <br/><br/>• 您不想要購買和管理額外的 HPC 叢集基礎結構<br/><br/>• 您有暫時性尖峰需求期間或特殊專案。
**完全在 Azure 中建立 HPC 叢集**<br/><br/>[![IaaS 中的叢集][iaas_cluster]](./media/batch-hpc-solutions/iaas_cluster.png)<br/><br/>深入了解：<br/>• [Azure 中的 HPC 叢集解決方案](./big-compute-resources.md)<br/><br/>|• 在標準或自訂 Windows 或 Linux 基礎結構即服務 (IaaS) 虛擬機器上，快速且一致地部署您的應用程式和叢集工具。<br/><br/>• 藉由使用您選擇的作業排程解決方案，執行各種 Big Compute 工作負載。<br/><br/>• 使用額外的 Azure 服務 (包括網路和儲存體) 以建立完整的以雲端為基礎的解決方案。 |• 您不想要購買和管理額外的 Linux 或 Windows HPC 叢集基礎結構<br/><br/>• 您有暫時性尖峰需求期間或特殊專案<br/><br/>• 您一時需要額外的叢集，但是不想要投資用來部署叢集的電腦和空間<br/><br/>• 您想要卸載 Big Compute 的應用程式，讓它完全在雲端中做為服務執行
**相應放大平行應用程式至 Azure**<br/><br/>[![Azure Batch][batch_proc]](./media/batch-hpc-solutions/batch_proc.png)<br/><br/>深入了解：<br/>• [Azure Batch 的基本](./batch-technical-overview.md)<br/><br/>• [開始使用適用於 .NET 的 Azure Batch 程式庫](./batch-dotnet-get-started.md)|• 以 [Azure Batch](https://azure.microsoft.com/documentation/services/batch/) 進行開發以相應放大各種 Big Compute 工作負載，以在 Windows 或 Linux 虛擬機器的集區上執行。<br/><br/>• 使用 Azure 平台服務以管理部署和自動調整虛擬機器、作業排程、災害復原、資料移動、相依性管理和應用程式部署。|• 您不想要管理計算資源或作業排程器，而是想要著重在執行您的應用程式<br/><br/>• 您想要卸載大量計算的應用程式，讓它在雲端中做為服務執行<br/><br/>• 您想要自動調整您的計算資源以符合計算工作負載


## 適合大量運算的 Azure 服務

以下是一些關於計算、資料、網路和相關服務的詳細資訊，您可將其結合以使用於 Big Compute 解決方案和工作流程。如需 Azure 服務的詳細指引，請參閱 Azure 服務[文件](https://azure.microsoft.com/documentation/)。本文中稍早的[案例](#scenarios)僅顯示一些使用這些服務的方法。

>[AZURE.NOTE] Azure 定期導入了對於您的案例可能有用的新服務。如果您有疑問，請連絡 [Azure 合作夥伴](https://pinpoint.microsoft.com/zh-TW/search?keyword=azure)或寄電子郵件到 bigcompute@microsoft.com。

### 計算服務

Azure 計算服務是 Big Compute 解決方案的核心，且不同的計算服務針對不同的案例提供優點。在基本層級中，這些服務為使用 Windows Server Hyper-V 技術，由 Azure 提供的虛擬機器計算執行個體上執行的應用程式提供不同的模式。這些執行個體可以執行標準和自訂的 Linux 和 Windows 作業系統和工具。Azure 可以讓您選擇[執行個體大小](../virtual-machines/virtual-machines-windows-sizes.md)，搭配 CPU 核心、記憶體、磁碟容量和其他特性等不同組態。您可視您的需求將執行個體擴充至數千個核心，並在需要較少的資源時相應減少。

>[AZURE.NOTE] 利用 Azure 計算密集型執行個體，以改善 HPC 工作負載的效能和延展性，包括需要低延遲和高輸送量應用程式網路的平行 MPI 應用程式。請參閱[關於 H 系列和計算密集型 A 系列 VM](../virtual-machines/virtual-machines-windows-a8-a9-a10-a11-specs.md)。

服務 | 說明
------------- | -----------
**[虛擬機器](https://azure.microsoft.com/documentation/services/virtual-machines/)**<br/><br/> |• 使用 Microsoft Hyper-V 技術來提供計算基礎結構即服務 (IaaS)<br/><br/>• 可讓您從 [Azure Marketplace](https://azure.microsoft.com/marketplace/) 中的標準 Windows Server 或 Linux 映像、或您自行提供的映像和資料磁碟中，更靈活地佈建和管理永續性雲端電腦<br/><br/>• 可以 [VM 調整集](https://azure.microsoft.com/documentation/services/virtual-machine-scale-sets/)的形式部署或管理，以從相同的虛擬機器建置大規模服務，並具有自動調整功能可自動增加或減少容量<br/><br/>• 在雲端中完全執行內部部署計算叢集工具和應用程式<br/><br/>
**[雲端服務](https://azure.microsoft.com/documentation/services/cloud-services/)**<br/><br/> |• 可以在背景工作角色執行個體中執行大量運算應用程式，這是執行 Windows Server 版本，而且完全受 Azure 管理的虛擬機器<br/><br/>• 啟用含低管理負擔，在平台即服務 (PaaS) 模型中執行的可調整、可靠的應用程式<br/><br/>• 可能需要額外的工具或開發以整合內部部署 HPC 叢集解決方案
**[批次](https://azure.microsoft.com/documentation/services/batch/)**<br/><br/> |• 在完全受管理的服務中執行大規模的平行與批次工作負載<br/><br/>• 提供虛擬機器之受管理集區的作業排程和自動調整<br/><br/>• 允許開發人員建置和執行做為服務的應用程式，或啟用雲端的現有應用程式<br/>

### 儲存體服務

大量運算解決方案通常會在一組輸入資料上操作，並產生其結果的資料。Big Compute 解決方案中使用的部分 Azure 儲存體服務包括：

* [Blob、資料表和佇列儲存體](https://azure.microsoft.com/documentation/services/storage/) - 分別管理大量非結構化資料、NoSQL 資料，和工作流程和通訊的訊息。例如，您可以將 Blob 儲存體使用於大型技術資料集，或應用程式處理的輸入影像或媒體檔案。您可以在解決方案中使用佇列以速行非同步通訊。請參閱 [Microsoft Azure 儲存體簡介](../storage/storage-introduction.md)。

* [Azure 檔案儲存體](https://azure.microsoft.com/services/storage/files/) - 在 Azure 中使用標準 SMB 通訊協定 (為一些 HPC 叢集解決方案必備) 來共用一般檔案和資料。

* [Data Lake Store](https://azure.microsoft.com/services/data-lake-store/) - 為雲端提供超大規模的 Apache Hadoop 分散式檔案系統，適用於批次分析、即時分析和互動式分析。

### 資料和分析服務

若干大量運算案例牽涉到大型資料流，或會產生需要進一步處理或分析的資料。Azure 提供幾種資料和分析服務，包括︰

* [Data Factory](https://azure.microsoft.com/documentation/services/data-factory/) - 建置資料導向的工作流程 (管線)，以聯結、彙總和轉換來自內部部署、雲端型和網際網路資料存放區的資料。

* [SQL Database](https://azure.microsoft.com/documentation/services/sql-database/) - 在受管理服務中提供 Microsoft SQL Server 關聯式資料庫管理系統的主要功能。

* [HDInsight](https://azure.microsoft.com/documentation/services/hdinsight/) - 在雲端中部署和佈建 Windows Server 或 Linux 架構的 Apache Hadoop 叢集，以管理、分析和報告巨量資料。

* [機器學習服務](https://azure.microsoft.com/documentation/services/machine-learning/) - 可幫助您在完全管理的服務中建立、測試、操作和管理預測分析解決方案。

### 其他服務

Big Compute 解決方案可能需要其他 Azure 服務，才能連線至內部部署或其他環境中的資源。範例包括：

* [虛擬網路](https://azure.microsoft.com/documentation/services/virtual-network/) - 在 Azure 中建立邏輯隔離的區段，以將 Azure 資源彼此連接或連接到您的內部部署資料中心。透過跨單位虛擬網路，Big Compute 應用程式可存取內部部署資料、Active Directory 服務和授權伺服器

* [ExpressRoute](https://azure.microsoft.com/documentation/services/expressroute/) - 可在 Microsoft 資料中心和內部部署或共置環境中的基礎結構之間建立私人連線。相較於透過網際網路的一般連線，ExpressRoute 提供更高安全性、可靠性、更快速度以及更低延遲。

* [服務匯流排](https://azure.microsoft.com/documentation/services/service-bus/) - 提供數種機制，讓應用程式進行通訊或交換資料，不管它們位於 Azure、其他雲端平台，還是資料中心。

## 後續步驟

* 請參閱 [Batch 和 HPC 的技術資源](big-compute-resources.md)來尋找建置您的解決方案的技術指導。

* 與合作夥伴討論 Azure 選項，包括 Cycle Computing 和 UberCloud。

* 閱讀 [Towers Watson](https://customers.microsoft.com/Pages/CustomerStory.aspx?recid=18222)、[Altair](https://azure.microsoft.com/blog/availability-of-altair-radioss-rdma-on-microsoft-azure/)、[ANSYS](https://azure.microsoft.com/blog/ansys-cfd-and-microsoft-azure-perform-the-best-hpc-scalability-in-the-cloud/) 和 [d3VIEW](https://customers.microsoft.com/Pages/CustomerStory.aspx?recid=22088) 所提供的 Azure Big Compute 解決方案。

* 最新宣告請參閱 [Microsoft HPC 和批次的小組部落格](http://blogs.technet.com/b/windowshpc/)以及[Azure 部落格](https://azure.microsoft.com/blog/tag/hpc/)。

<!--Image references-->
[parallel]: ./media/batch-hpc-solutions/parallel.png
[coupled]: ./media/batch-hpc-solutions/coupled.png
[iaas_cluster]: ./media/batch-hpc-solutions/iaas_cluster.png
[burst_cluster]: ./media/batch-hpc-solutions/burst_cluster.png
[batch_proc]: ./media/batch-hpc-solutions/batch_proc.png

<!---HONumber=AcomDC_0928_2016-->