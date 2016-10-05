<properties
   pageTitle="Azure SQL Database Azure 案例研究 - GEP| Microsoft Azure"
   description="了解 GEP 如何使用 SQL Database 將觸角伸向更多全球客戶及提升效率"
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

# Azure 讓 GEP 全球化並更有效率

![GEP 標誌](./media/sql-database-case-study-gep/geplogo.png)

GEP 提供軟體和服務，可讓世界各地的採購領導者充分影響其業務營運、策略及財務表現。除了諮詢和受管理的服務，該公司還提供 SMART by GEP® 這個雲端型的全方位採購軟體平台。不過，GEP 在嘗試使用自己的內部部署資料中心來支援 SMART by GEP 時遇到一些限制︰所需的投資過高，而其他國家/地區的法規需求則令人在必要的投資上更為卻步。透過移至雲端，GEP 釋出了 IT 資源，讓它可以節省 IT 營運上的心力，而更專注於為其全球客戶開發新的價值來源。

## 透過使用 Azure 來擴大服務範圍和成長幅度

SMART by GEP 客戶喜歡此平台的功能和易用性；客戶可以隨時隨地透過任何裝置 (膝上型電腦、平板電腦或手機) 管理他們的程序。透過移至 Microsoft Azure，GEP 已經能夠順應其快速成長及擴展到新市場的可能性。GEP 技術副總裁 Dhananjay Nagalkar 表示：「Microsoft Azure 不僅讓我們能夠快速靈活地調整服務，還提供可協助我們符合全球客戶法規需求的區域資料中心，因此在 GEP 的成功上扮演了關鍵角色。」

## 自備資料中心的限制

在 2013 年，GEP 領導人承認了他們需要一個方法，來確保在他們發展客戶群時能夠保有延展性與效能。Nagalkar 解釋道：「若要以我們現有的資料中心滿足該要求，我們將必須大幅擴充基礎結構和 IT 資源。這項投資與時間的耗費都會相當龐大。」 內部部署的實體和虛擬機器需要大量的設定、管理、調整、備份及修補，其花費對 GEP 而言成本過高。而另一方面，雲端解決方案則提供簡單性和便利性，讓 GEP 能夠更專注於開發而不是管理大量且不斷成長的 IT 作業。Nagalkar 知道 GEP 可以透過移轉至雲端，來降低其基礎結構的採購、設定及管理額外負荷。

GEP 也需要一個方法來克服讓它無法進入部分全球市場的法規障礙。對 GEP 的許多潛在歐洲客戶而言，法規遵循會要求他們必須將資料儲存在當地的地理區域中。但對 GEP 來說，建置多個資料中心的做法並不切實際。「廣泛的基礎結構投資和 IT 人力成本會嚴重影響利潤」Nagalkar 說道。「因此，我們實際上被迫放棄需要當地儲存資料的潛在客戶。」

Nagalkar 知道雲端解決方案可能是這個窘境的解答。如果 GEP 能夠移至在全球都有據點的雲端提供者，就能將資料儲存在較接靠客戶實際位置的地方，而更符合其客戶的法規和效能需求。

## 在雲端自在翱翔

Nagalkar 及其團隊探索了數個雲端選項，但大多數都是基礎結構即服務 (IaaS) 型解決方案，需要 GEP 大量投資 IT 資源來設定及管理服務。最後證明 Azure 平台即服務 (PaaS) 解決方案是更合適的選項。

「使用 Azure，GEP 就不需要處理資料庫管理、虛擬機器設定、修補或其他基礎結構管理工作」Nagalkar 說道。「取而代之的是，我們可以集中資源來發揮所長：利用我們在採購方面的專長來撰寫真正為客戶帶來成效的軟體。」

事實上，移至 Azure 既讓 GEP 能夠縮減其 IT 作業人員，同時也能夠為客戶提供更強大的功能。

藉由利用全球各地的 Azure 資料中心，GEP 便可輕鬆將觸角延伸至歐洲和亞洲。這些全球資料中心除了讓 GEP 能夠靈活調整之外，還能符合客戶的當地儲存資料需求，既可減少延遲也可符合法規需求。

## SMART by GEP 架構

GEP 是在 Azure 上從頭開始建置 SMART by GEP。對 GEP 來說，其中一個重要誘因就是與 GEP 在內部部署環境所能達成的相比，使用 Azure SQL Database 所能體驗到的是更大的延展性、更短的停機時間，以及更低的維護成本。不過，在移至雲端之後，GEP 在雲端發現了能以更佳方式回應客戶需求的新開發機會 (例如快速的原型設計和精簡的工程)。在 Azure 中開發可讓 GEP 的開發人員擺脫在內部部署環境中可能需要面對的軟體授權問題。SMART by GEP 的核心是 Azure SQL Database，不過 GEP 也使用許多其他 Azure 服務來輕鬆快速地持續改進 SMART by GEP。

![圖 1](./media/sql-database-case-study-gep/figure1.png) 圖 1.SMART by GEP 架構

## 結構化資料

SMART by GEP 應用程式的核心是為企業採購管理解決方案提供技術支援的 Azure SQL Database 執行個體。當 GEP 進行 SMART by GEP 的工程時，已了解 Azure SQL Database 是其架構的最合適選項，既可讓公司達到最高程度的資料保護，又能符合其法規需求。GEP 使用 Azure SQL Database 所提供的多層資料保護，包括︰

- 透過透明資料加密來加密待用資料。
- 將 Azure SQL Database 與 Azure Active Directory 整合來提供驗證防護。
- 使用「資料列層級安全性」來僅限存取適當的資料子集。
- 透過原則為資料即時加上遮罩。
- 透過「Azure SQL Database 稽核」來追蹤資料庫事件。

> 「我們可以使用所有這些選項，而不進行任何主要的程式碼變更，並將對效能的影響降到最低」Nagalkar 說道。

使用 Azure SQL Database 與將同樣的花費用在內部部署環境中進行工程相比，GEP 可自動擁有更強大的災害復原能力，因為 Azure SQL Database 內建容錯功能。GEP 使用 Azure SQL Database 中的「作用中異地複寫」功能，結合了不同地理區域中的多個作用中、可讀取且在線上的次要複本 (Always On 可用性群組)，來形成高可用性組。在各個區域複寫 SMART by GEP 資料意謂著如果發生全區域災害，GEP 只要使用最低限度的復原點目標 (RPO) 和復原時間目標 (RTO)，即可輕鬆復原客戶資料。

每個 SMART by GEP 客戶都有兩個 Azure SQL Database 執行個體︰一個用於線上交易處理 (OLTP)，一個用於分析 (例如客戶開支和報表分析)。Azure SQL Database 彈性資料庫集區可讓 GEP 輕鬆管理全球數千個資料庫，以處理無法預測的資料庫資源需求。彈性集區為 GEP 提供一個方法，既能確保客戶資料庫可依需要調整，而不會有過度佈建或佈建不足的情況，同時也讓 GEP 能夠控制成本。此外，由於這是一項 PaaS 服務，因此 GEP 會隨著自動升級取得所有新的 Azure SQL Database 功能。

## 非結構化和半結構化資料

不過，有些 SMART by GEP 客戶資料需要較不嚴格的結構化儲存體。針對這類型的資料，GEP 採用 Azure Blob 儲存體、「Azure 資料表儲存體」及「Azure Redis 快取」。Azure Blob 儲存體上裝載了 SMART by GEP 使用者上傳到應用程式的所有附件。它也是 SMART by GEP 儲存靜態內容 (例如階層式樣式表 (CSS) 和 JavaScript 檔案) 的位置。

GEP 將非客戶面向資料 (例如 SMART by GEP 記錄資料) 儲存在「Azure 資料表儲存體」中，這為 GEP 有效地提供了符合成本效益的無限儲存體及快速的擷取時間，而不需擔心如何為資料設定結構描述。GEP 使用「Azure Redis 快取」作為主要快取。

## 驗證與路由

Azure 的「存取控制服務」(ACS) 為 SMART by GEP 使用者提供各種不同的選項來登入軟體。Azure ACS 可以與使用「安全性聲明標記語言」(SAML) 交換驗證資料的任何身分識別提供者 (例如，Active Directory Domain Services、Ping Identity、OneLogin 或 SiteMinder) 同盟。這可協助 GEP 為客戶實作單一登入 (SSO)，而不需擔心如何儲存使用者認證及維護客戶密碼原則。

登入之後，客戶便能存取 SMART by GEP 中正確的業務資源。GEP 使用「Azure 流量管理員」針對來自客戶行動裝置和瀏覽器工作階段的要求，進行重新導向和負載平衡。

## 其他 Azure 服務

GEP 採用一些其他 Azure 服務來讓 SMART by GEP 回應客戶需求。GEP 使用 Azure 雲端服務 (Web 角色和背景工作角色) 來裝載應用程式展示及受保護的商務邏輯服務。雲端服務不僅讓開發人員能夠管理基礎結構即程式碼 (IAC)，而且部署新 SMART by GEP 應用程式所花的時間也比在內部部署資料中心內部署所需的時間短很多，而這些全都不需要 IT 介入。GEP 開發人員可以使用雲端服務預備環境來測試新的版本，而不會影響目前的生產環境部署。測試之後，GEP 會在一分鐘內使用 Azure 雲端服務的 VIP 交換功能，將預備程式碼移到生產環境位置中，藉此縮短部署停機時間。

為了降低應用程式延遲，GEP 使用「Azure 內容傳遞網路」(CDN) 將儲存在 Azure Blob 儲存體中的靜態內容 (例如 CSS 和 JavaScript 檔案) 放在靠近使用者的邊緣伺服器上。GEP 使用「Azure 服務匯流排」來支援應用程式架構模式，範圍從「發佈-訂閱」到部分「命令查詢職責分離」(CQRS) 及含鬆散結合與非同步通訊的分層架構都包含在內。GEP 使用「Azure 媒體服務」來改善其客戶支援服務。GEP 發現它可以輕鬆地將使用者支援影片張貼在「Azure 媒體服務」上。這些影片回答了常見的使用者問題，除了減輕 GEP 客戶支援人員的一些支援負荷之外，也可協助提升 SMART by GEP 使用者滿意度。

為了傳送 SMART by GEP 每天產生的數千封交易郵件，公司使用 SendGrid .NET API 與 Azure 整合。對 GEP 開發人員來說，這很簡單 — Azure Marketplace 中就有適用於 Azure 的 SendGrid 附加元件。GEP 開發人員可以直接在 Microsoft Visual Studio 中使用 SendGrid NuGet 套件來設定 SMART by GEP；GEP IT 可以直接從 Azure 監視軟體的 SendGrid 電子郵件流量。

最後，SMART by GEP 會使用「Azure 虛擬機器」(Azure IaaS 服務) 來裝載在進行工程時還沒有意義的應用程式和服務，以取代軟體即服務 (SaaS) 或 PaaS 解決方案。例如，GEP 會將整合 API 服務裝載在虛擬機器中，以便進行與客戶內部部署企業資源規劃 (ERP) 系統 (例如 SAP、Oracle、PeopleSoft、JD Edwards、Microsoft Dynamics GP 及 Lawson) 以及與客戶 SaaS 解決方案的企業對企業 (B2B) 整合，來有效率地交換採購文件 (例如發票)。

> 「在 Microsoft Azure 雲端建置 SMART by GEP 完全免除了對內部部署 IT 的需要，不僅對 GEP 來說是如此，對我們客戶的採購作業來說也是如此。」

> — Dhananjay Nagalkar，技術解決方案副總裁

## 不需擴充 IT 規模即可擴大客戶滿意度

由於是從內部部署資料中心移轉至 Azure，並且在 Azure 平台上從頭建置 SMART by GEP，因此 GEP 不需擴充其基礎結構或 IT 人員編制，便可增加延展性和彈性。事實上，此公司已超過四年未增加 IT 資源。Azure 便利的 PaaS 模型讓 GEP 降低了在廠商支援和作業管理上的開支。因此，GEP 能夠將資源集中在軟體開發；而在雲端進行開發則是讓 GEP 開發人員能夠快速測試新的想法，而不需花費時間與 IT 協調或擔心內部部署授權需求。Azure SQL Database 協助 GEP 更能確保其客戶始終享有優異的服務與效能。
 
## 詳細資訊

- GEP 首頁︰[GEP](http://www.gep.com)
- Smart by GEP：[Smart by GEP](http://www.smartbygep.com)

##GEP 參與者

- Huzaifa Matawala，GEP 副主任 - 架構設計師
- Sathyan Narasingh，GEP 工程經理
- Deepa Velukutty，GEP 資料庫架構設計師

![Smart 標誌](./media/sql-database-case-study-gep/smartbygeplogo.png)

![GEP 標誌](./media/sql-database-case-study-gep/geplogo.png)

<!---HONumber=AcomDC_0921_2016-->