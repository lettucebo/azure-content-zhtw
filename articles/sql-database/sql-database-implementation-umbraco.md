<properties
   pageTitle="Azure SQL Database Azure 案例研究 - Umbraco| Microsoft Azure"
   description="了解 Umbraco 如何使用 SQL Database 為雲端中數千個租用戶快速佈建和調整服務"
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

# Umbraco 使用 Azure SQL Database 為雲端中數千個租用戶快速佈建和調整服務

![Umbraco 標誌](./media/sql-database-case-study-umbraco/umbracologo.png)

Umbraco 是一個熱門的開放原始碼內容管理系統 (CMS)，從小型行銷活動或簡冊站台，到適用於 Fortune 500 大公司及全球媒體網站的複雜應用程式，全部都能執行。

> 「我們有很大的開發人員社群使用此系統，其中有超過 100,000 個開發人員在我們的論壇上，以及 350,000 個執行 Umbraco 的即時站台。」

> — Morten Christensen，Umbraco 技術主管

> [AZURE.VIDEO azure-sql-database-case-study-umbraco]

為了簡化客戶部署，Umbraco 新增了 Umbraco 即服務 (UaaS)：這是一個軟體即服務 (SaaS) 方案，此方案既不需要內部部署又提供內建的調整功能，而且還透過讓開發人員能夠專注於產品創新而非解決方案管理，免除了管理上的額外負荷。Umbraco 藉由倚賴 Microsoft Azure 所提供的彈性平台即服務 (PaaS) 模型而得以提供所有這些優點。

UaaS 讓 SaaS 客戶能夠使用他們先前無法觸及的 Umbraco CMS 功能。針對這些客戶，會佈建一個包含生產環境資料庫的可運作 CMS 環境。客戶可以依其需求，最多新增兩個額外的資料庫來用於開發和預備環境。發出新環境要求時，自動化程序會自動為該客戶指派一個資料庫。新資料庫幾秒內就會準備就緒，因為 Umbraco 早就已經從 Azure 的可用資料庫彈性集區預先佈建資料庫 (請參閱圖 1)。


![圖 1](./media/sql-database-case-study-umbraco/figure1.png)

圖 1.Umbraco 即服務 (UaaS) 的佈建生命週期
 
##Azure 彈性集區和自動化可簡化部署

藉由使用 Azure SQL Database 和其他 Azure 服務，Umbraco 客戶可自行佈建其環境，而 Umbraco 則可以在直覺式工作流程中輕鬆地監視和管理資料庫︰

1.	佈建

    Umbraco 維持一個含有 200 個可用資料庫的容量，這些資料庫都是來自彈性集區的預先佈建資料庫。當有新客戶註冊 UaaS 時，Umbraco 會透過從可用性集區指派資料庫給客戶，以近乎即時的方式為客戶提供新的 CMS 環境。

    當可用性集區達到其臨界值時，系統就會建立新的彈性集區，並預先佈建新資料庫以便視需要指派給客戶。

    此實作是使用 C# 管理程式庫和「Azure 服務匯流排」佇列來完全自動化執行。

2.	利用

    客戶會使用一到三個環境 (用於生產、預備和/或開發)，每個環境都有自己的資料庫。客戶資料庫是在彈性資料庫集區中，這讓 Umbraco 不需過度佈建即可提供有效率的調整。

    ![圖 2](./media/sql-database-case-study-umbraco/figure2.png)

    ![圖 3](./media/sql-database-case-study-umbraco/figure3.png)

    圖 2.顯示專案總覽和詳細資料的 Umbraco 即服務 (UaaS) 客戶網站

    Azure SQL Database 使用「資料庫交易單位」(DTU) 來代表真實世界資料庫交易所需的相對能力。就 UaaS 客戶而言，資料庫一般是以大約 10 DTU 來運作，但每個資料庫都可以依需求彈性調整。這意謂著 UaaS 可以確保客戶一律擁有必要的資源，甚至是在尖峰時間也一樣。舉例來說，在最近某個星期天的夜間體育活動，一個 UaaS 客戶的資料庫在比賽的持續時間經歷了高達 100 個 DTU 的尖峰。Azure 彈性集區使得 Umbraco 既能夠支援該高需求，又不導致效能降低。

3.	監視

    Umbraco 使用 Azure 入口網站中的儀表板以及自訂的電子郵件警示來監視資料庫活動。

4.	災害復原

    Azure 提供兩種災害復原 (DR) 選項：「作用中異地複寫」和「異地還原」。公司應該選取的 DR 選項取決於其[商務持續性目標](sql-database-business-continuity.md)。

    「作用中異地複寫」可在發生停機狀況時，提供最快層級的回應。透過「作用中異地複寫」，您可以在不同區域的伺服器上最多建立四個可讀取的次要資料庫，然後可以在發生失敗時起始容錯移轉，來移轉至其中任何一個次要資料庫。

    Umbraco 不需要使用「異地複寫」，但是它的確運用了「Azure 異地還原」來協助確保在發生中斷時將停機時間縮到最短。「異地還原」需倚賴異地備援 Azure 儲存體中的資料庫備份。這可讓使用者在主要區域中發生中斷時，從備份複本還原。

5.	取消佈建

    刪除專案環境時，於「Azure 服務匯流排」佇列清除期間，將會移除所有關聯的資料庫 (開發、預備或即時)。此自動化程序會將未使用的資料庫還原至 Umbraco 的彈性資料庫可用性集區，如此既可充分利用這些資料庫，又可將這些資料庫用於未來的佈建。

##彈性集區可讓 UaaS 輕鬆進行調整

透過利用 Azure 彈性資料庫集區，Umbraco 可以將其客戶的效能最佳化，而無須擔心過度佈建或佈建不足。Umbraco 目前擁有 3,000 個遍佈在 19 個彈性資料庫集區的資料庫，不論是現有 325,000 個客戶中的任何一個客戶，還是已準備好要在雲端部署 CMS 的新客戶，Umbraco 都能夠輕鬆地視需要調整來配合其需求。

事實上，Umbraco 的技術主管 Morten Christensen 表示：「UaaS 現在大約是以每天 30 個新客戶的速度在成長。我們的客戶很高興能夠便利地在幾秒內就佈建好新的專案、使用「單鍵部署」功能從開發環境立即將更新發佈至他們的即時站台，以及以同樣快速的方式在發現錯誤時進行變更。」

如果客戶不再需要第二個和/或第三個環境，可以直接將這些環境移除。這樣會釋出資源成為 Umbraco 彈性資料庫可用性集區的一部分，而可供用於其他客戶。

![圖 4](./media/sql-database-case-study-umbraco/figure4.png)

圖 3.Microsoft Azure 上的 UaaS 部署架構

##從資料中心到雲端的路徑

當 Umbraco 開發人員一開始決定移至 SaaS 模型時，即已知道他們需要一個符合成本效益且可調整的方式來建置服務。

> 「彈性資料庫集區是最適合我們 SaaS 方案的選項，因為我們可以視需要上下調整容量。佈建相當簡單，再搭配上我們的設定，我們便可以做最充分的利用」。

> — Morten Christensen，Umbraco 技術主管

「我們想要把時間花在解決客戶的問題上，而不是花在管理基礎結構上。我們想要讓客戶輕輕鬆鬆就能夠獲取最大價值。」Umbraco 的創辦人 Niels Hartvig 說道。「一開始時，我們考慮自行裝載伺服器，但容量規劃是一個夢魘」。 碰巧的是，Umbraco 並未雇用任何資料庫系統管理員，而這強調了一個使用 UaaS 的主要價值主張。

Umbraco 開發人員的其中一個重要的目標就是，為 UaaS 客戶提供一個快速且沒有容量限制的環境佈建方式。但是在 Umbraco 資料中心提供專屬的託管服務將需要大量的額外容量，以應付處理時的高載狀況。這意謂著要新增大量一般不會充分利用的計算基礎結構。

此外，Umbraco 開發團隊想要的是一個能讓他們儘量重複使用現有程式碼的解決方案。如 Umbraco 開發人員 Mikkel Hansen 所述：「我們非常滿意我們已經熟悉的 Microsoft 開發工具，像是 Microsoft SQL Server、Microsoft Azure SQL Database、ASP.net，以及 Internet Information Services (IIS)。在投資 IaaS 或 PaaS 雲端解決方案之前，我們想要確定它可以支援我們的 Microsoft 工具與平台，如此我們才不需要對我們的程式碼基底進行大規模變更。」

為了符合其所有準則，Umbraco 尋找了符合下列資格的雲端合作夥伴︰

-	足夠的容量與可靠性
-	支援 Microsoft 開發工具，如此 Umbraco 工程師才不會被迫完全重新創建其開發環境
-	在 UaaS 參與競爭的所有地理市場中都存在 (企業需要確保它們可以快速地存取其資料，並且其資料是儲存在符合其區域法規要求的位置)

##為什麼 Umbraco 選擇將 Azure 用於 UaaS

Morten Christensen 表示：「在考量我們的所有選項之後，我們選擇了 Azure，因為它符合我們從管理性、延展性再到熟悉度及符合成本效益方面的所有準則。我們在 Azure VM 上設定環境，每個環境都有自己的 Azure SQL 資料庫執行個體，而所有執行個體都在彈性資料庫集區中。透過將開發、預備及即時環境之間的資料庫加以區隔，我們可以為客戶提供與規模相符的強大效能隔離，這是一項極大的優點。」

Morten 繼續說道：「以前，我們必須手動佈建 Web 資料庫的伺服器。現在，我們完全無須思考這項工作。從佈建到清除的所有作業都是自動化完成。」

Morten 也很滿意 Azure 所提供的調整功能。「彈性資料庫集區是最適合我們 SaaS 方案的選項，因為我們可以視需要上下調整容量。佈建相當簡單，再搭配上我們的設定，我們便可以做最充分的利用」。 Morten 表示：「彈性集區的簡單性，再加上以服務層為基礎之 DTU 的保證，使得我們能夠依需求佈建新的資源集區。最近，我們一個較大客戶的即時環境尖峰達到了 100 個 DTU。透過使用 Azure，我們的彈性集區為客戶的資料庫提供了它們所需的即時資源，而不需預測 DTU 需求。簡單地說，我們的客戶達到了預期的周轉時間，而我們則符合了效能服務等級協定。」

Mikkel Hansen 總結：「除了將「Azure 服務匯流排」與 Azure SQL Database 搭配使用的基礎技術以外，我們也採用了強大的 Azure 演算法，將常見的 SaaS 案例 (讓新客戶即時大規模上線) 與我們的應用程式模式 (預先佈建開發資料庫和即時資料庫) 連接。」

##透過 Azure，UaaS 的表現超出客戶期望

由於選擇 Azure 作為雲端合作夥伴，因此 Umbraco 不需像自我裝載解決方案那樣需要投資 IT 資源，就能夠為 UaaS 客戶提供最佳化的內容管理效能。如 Morten 所述：「我們相當滿意 Azure 為開發人員提供的便利性和延展性，而我們的客戶也為所獲得的功能和可靠性振奮不已。整體而言，對我來說是一大勝利！」
 
## 詳細資訊

- 若要深入了解 Azure 彈性資料庫集區，請參閱[彈性資料庫集區](sql-database-elastic-pool.md)。

- 若要深入了解「Azure 服務匯流排」，請參閱 [Azure 服務匯流排](https://azure.microsoft.com/services/service-bus/)。

- 若要深入了解 Web 角色和背景工作角色，請參閱[背景工作角色](../fundamentals-introduction-to-azure.md#compute)。

- 若要深入了解虛擬網路，請參閱[虛擬網路](https://azure.microsoft.com/documentation/services/virtual-network/)。

- 若要深入了解備份與復原，請參閱[商務持續性](sql-database-business-continuity.md)。

- 若要深入了解監視集區，請參閱[監視集區](sql-database-elastic-pool-manage-portal.md)。

- 若要深入了解 Umbraco，請參閱 [Umbraco](https://umbraco.com/cloud)。

<!---HONumber=AcomDC_0921_2016-->