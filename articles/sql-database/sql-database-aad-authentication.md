<properties
   pageTitle="使用 Azure Active Directory 驗證連線到 SQL Database 或 SQL 資料倉儲 | Microsoft Azure"
   description="了解如何使用 Azure Active Directory 驗證連接到 SQL Database"
   services="sql-database"
   documentationCenter=""
   authors="BYHAM"
   manager="jhubbard"
   editor=""
   tags=""/>

<tags
   ms.service="sql-database"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="data-management"
   ms.date="09/16/2016"
   ms.author="rick.byham@microsoft.com"/>

# 使用 Azure Active Directory 驗證連線到 SQL Database 或 SQL 資料倉儲

Azure Active Directory 驗證是 Azure Active Directory (Azure AD) 中使用身分識別連接到 Microsoft Azure SQL Database 和 [SQL 資料倉儲](../sql-data-warehouse/sql-data-warehouse-overview-what-is.md)的機制。您可以使用 Azure Active Directory 驗證，在單一中央位置集中管理資料庫使用者和其他 Microsoft 服務的身分識別。中央識別碼管理提供單一位置以管理資料庫使用者並簡化權限管理。包括以下優點：

- 它提供 SQL Server 驗證的替代方案。
- 協助停止跨資料庫伺服器使用過多的使用者身分識別。
- 允許在單一位置的密碼替換
- 客戶可以管理使用外部 (AAD) 群組的資料庫權限。
- 它可以藉由啟用整合式 Windows 驗證和 Azure Active Directory 支援的其他形式驗證來避免儲存密碼。
- Azure Active Directory 驗證會使用自主資料庫使用者，在資料庫層級驗證身分。
- Azure Active Directory 針對連線到 SQL Database 的應用程式支援權杖型驗證。
- Azure Active Directory 驗證本機 Azure Active Directory 的 ADFS (網域同盟) 或原生使用者/密碼驗證，而不需進行網域同步處理。
- Azure Active Directory 支援來自 SQL Server Management Studio 的連線，其中使用包含 Multi-Factor Authentication (MFA) 的 Active Directory 通用驗證。MFA 包含增強式驗證功能，其中提供一系列簡易的驗證選項，例如電話、簡訊、含有 PIN 的智慧卡或行動應用程式通知。如需詳細資訊，請參閱[適用於與 SQL Database 和 SQL 資料倉儲搭配使用之 Azure AD MFA 的 SSMS 支援](sql-database-ssms-mfa-authentication.md)。

設定步驟包括以下設定和使用 Azure Active Directory 驗證的程序。

1. 建立和填入 Azure Active Directory。
2. 確定您的資料庫是 Azure SQL Database V12。(SQL 資料倉儲並不需要)。
3. 選用：和目前與您的 Azure 訂用帳戶相關聯的 Active Directory 產生關聯並加以變更
4. 建立 Azure SQL Server 或 [Azure SQL 資料倉儲](https://azure.microsoft.com/services/sql-data-warehouse/)的 Azure Active Directory 系統管理員。
5. 設定用戶端電腦。
6. 在對應至 Azure AD 身分識別的資料庫中建立自主資料庫使用者。
7. 使用 Azure AD 身分識別連接到您的資料庫。


## 信認架構

下列高階圖表摘要說明搭配使用 Azure AD 驗證與 Azure SQL Database 的解決方案架構。相同的概念適用於 SQL 資料倉儲。若要支援 Azure Active Directory 原生使用者密碼，只會考慮雲端部分和 Azure AD/Azure SQL Database。若要支援同盟驗證 (或 Windows 認證的使用者/密碼)，需要與 ADFS 區塊進行通訊。箭頭表示通訊路徑。

![aad 驗證圖表][1]

下圖表示允許用戶端藉由提交權杖連線到資料庫的同盟、信任和主控關聯性。此權杖是由 Azure AD 所驗證，並受到資料庫信任。客戶 1 可以代表具有原生使用者的 Azure Active Directory 或具有同盟使用者的 Azure Active Directory。客戶 2 代表包含已匯入使用者的可能解決方案；在此範例中，來自同盟 Azure Active Directory 且 ADFS 正與 Azure Active Directory 進行同步處理。請務必了解使用 Azure AD 驗證存取資料庫的必要條件是裝載訂用帳戶要與 Azure Active Directory 相關聯。您必須使用相同的訂用帳戶來建立裝載 Azure SQL Database 或 SQL 資料倉儲的 SQL Server。

![訂用帳戶關聯性][2]

## 系統管理員結構

使用 Azure AD 驗證時，SQL Database 伺服器會有兩個系統管理員帳戶：原始的 SQL Server 系統管理員和 Azure AD 系統管理員。相同的概念適用於 SQL 資料倉儲。只有以 Azure AD 帳戶為基礎的系統管理員可以在使用者資料庫中建立第一個 Azure AD 自主資料庫使用者。Azure AD 系統管理員登入可以是 Azure AD 使用者或 Azure AD 群組。當系統管理員是群組帳戶時，它可以供任何群組成員使用，啟用 SQL Server 執行個體的多個 Azure AD 系統管理員。以系統管理員的身分使用群組帳戶，可讓您集中新增和移除 Azure AD 中的群組成員，而不需要變更 SQL Database 中的使用者或權限，藉以增強管理性。一律只能設定一個 Azure AD 系統管理員 (使用者或群組)。

![系統管理員結構][3]

## 權限

若要建立新的使用者，您必須具有資料庫中的 `ALTER ANY USER` 權限。任何資料庫使用者皆可授與 `ALTER ANY USER` 權限。`ALTER ANY USER` 權限的擁有者還包括伺服器系統管理員帳戶、具備資料庫之 `CONTROL ON DATABASE` 或 `ALTER ON DATABASE` 權限的資料庫使用者，以及 `db_owner` 資料庫角色的成員。

若要在 Azure SQL Database 或 SQL 資料倉儲中建立自主資料庫使用者，您必須使用 Azure AD 身分識別連線到資料庫。若要建立第一個自主資料庫使用者，您必須使用 Azure Active Directory 系統管理員 (資料庫的擁有者) 連接到資料庫。如以下步驟 4 和 5 所示。只有針對 Azure SQL Database 或 SQL 資料倉儲伺服器建立 Azure Active Directory 系統管理員後，才可能進行任何 Azure Active Directory 驗證。如果已從伺服器移除 Azure Active Directory 系統管理員，則先前在 SQL Server 內建立的現有 Azure Active Directory 使用者便無法再使用其 Azure Active Directory 認證連線到資料庫。

## Azure AD 功能和限制

下列 Azure Active Directory 的成員可在 Azure SQL Server 或 SQL 資料倉儲中佈建：

- 原生成員：在受管理網域或客戶網域的 Azure AD 中建立的成員。如需詳細資訊，請參閱[將您自己的網域名稱新增至 Azure AD](../active-directory/active-directory-add-domain.md)。

- 同盟網域成員：利用同盟網域在 Azure AD 中建立的成員。如需詳細資訊，請參閱 [Microsoft Azure 現在支援 Windows Server Active Directory 的同盟](https://azure.microsoft.com/blog/2012/11/28/windows-azure-now-supports-federation-with-windows-server-active-directory/)。

- 從其他 Azure Active Directory 匯入，且為原生網域或同盟網域成員者。

- 建立 Active Directory 群組作為安全性群組。

不支援 Microsoft 帳戶 (例如 outlook.com、hotmail.com、live.com) 或其他來賓帳戶 (例如 gmail.com、yahoo.com)。如果您可以使用帳戶和密碼登入 [https://login.live.com](https://login.live.com)，則您就能使用 Azure SQL Database 或 Azure SQL 資料倉儲的 Azure AD 驗證不支援的 Microsoft 帳戶。

### 其他考量

- 若要增強管理性，建議您以系統管理員身分佈建專用的 Azure Active Directory 群組。
- 一個 Azure SQL Server 或 Azure SQL 資料倉儲一律只能設定一個 Azure AD 系統管理員 (使用者或群組)。
- 只有 SQL Server 的 Azure Active Directory 系統管理員可以一開始就使用 Azure Active Directory 帳戶連接到 Azure SQL Server 或 Azure SQL 資料倉儲。Active Directory 系統管理員可以設定後續的 Azure Active Directory 資料庫使用者。
- 建議將連接逾時設定為 30 秒。
- SQL Server 2016 Management Studio 和 SQL Server Data Tools for Visual Studio 2015 (版本 14.0.60311.1 (2016 年 4 月) 或更新版本) 支援 Azure Active Directory 驗證。(**.NET Framework Data Provider for SqlServer** 支援 Azure Active Directory 驗證，最低版本 .NET Framework 4.6)。因此，這些工具和資料層應用程式 (DAC 和 .bacpac) 的最新版本可以使用 Azure Active Directory 驗證。
- [ODBC 13.1 版](https://www.microsoft.com/download/details.aspx?id=53339)支援 Azure Active Directory 驗證，不過，`bcp.exe` 無法使用 Azure Active Directory 驗證進行連線，因為它們使用較舊的 ODBC 提供者。
- `sqlcmd` 從 13.1 版 (可從[下載中心](http://go.microsoft.com/fwlink/?LinkID=825643)取得) 開始即支援 Azure Active Directory 驗證。
- SQL Server Data Tools for Visual Studio 2015 至少需要 2016 年 4 月版本的 Data Tools (版本 14.0.60311.1)。Azure Active Directory 使用者目前不會顯示在 SSDT 物件總管中。解決方法是在 [sys.database\_principals](https://msdn.microsoft.com/library/ms187328.aspx) 中檢視使用者。
- [Microsoft JDBC Driver 6.0 for SQL Server](https://www.microsoft.com/zh-TW/download/details.aspx?id=11774) 支援 Azure Active Directory 驗證。此外，請參閱[設定連接屬性](https://msdn.microsoft.com/library/ms378988.aspx)。
- PolyBase 無法使用 Azure Active Directory 驗證進行驗證。
- 不支援某些工具，例如 BI 和 Excel。
- Azure 入口網站的 [匯入資料庫] 和 [匯出資料庫] 刀鋒視窗支援 SQL Database 的 Azure Active Directory 驗證。PowerShell 命令也支援使用 Azure Active Directory 驗證的匯入和匯出。


## 1\.建立和填入 Azure AD

建立 Azure Active Directory 並利用使用者和群組填入。Azure Active Directory 可以是 Azure AD 受管理網域的初始網域。Azure Active Directory 也可以是與 Azure Active Directory 同盟的內部部署 Active Directory 網域服務。

如需詳細資訊，請參閱[整合內部部署身分識別與 Azure Active Directory](../active-directory/active-directory-aadconnect.md)、[將您自己的網域名稱新增至 Azure AD](../active-directory/active-directory-add-domain.md)、[Microsoft Azure now supports federation with Windows Server Active Directory (Microsoft Azure 現在支援與 Windows Server Active Directory 同盟)](https://azure.microsoft.com/blog/2012/11/28/windows-azure-now-supports-federation-with-windows-server-active-directory/)、[管理您的 Azure AD 目錄](https://msdn.microsoft.com/library/azure/hh967611.aspx)和[使用 Windows PowerShell 管理 Azure AD](https://msdn.microsoft.com/library/azure/jj151815.aspx)。

## 2\.確定您的 SQL Database 是第 12 版

最新的 SQL Database V12 支援 Azure Active Directory 驗證。如需 SQL Database V12 的相關資訊並了解是否供您所在區域使用，請參閱[最新 SQL Database Update V12 的新功能](sql-database-v12-whats-new.md)。Azure SQL 資料倉儲不需要執行這個步驟，因為 SQL 資料倉儲只適用於 V12。

如果您目前有資料庫，請藉由連線至資料庫 (例如，使用 SQL Server Management Studio) 並執行 `SELECT @@VERSION;` 來確認它裝載於 SQL Database V12 中。SQL Database V12 中的資料庫預期輸出至少為 **Microsoft SQL Azure (RTM) - 12.0**。如果您的資料庫並非裝載於 SQL Database V12，請參閱[規劃和準備升級至 SQL Database V12](sql-database-v12-plan-prepare-upgrade.md)，然後造訪 Azure 傳統入口網站將資料庫移轉至 SQL Database V12。

或者，遵循[建立您的第一個 Azure SQL Database](sql-database-get-started.md) 中所列的步驟，在 SQL Database V12 中建立新的資料庫。**秘訣**：選取新資料庫的訂用帳戶之前參閱下一個步驟。

## 3\.選用：和目前與您的 Azure 訂用帳戶相關聯的 active directory 產生關聯並加以變更

若要將您的資料庫與貴組織的 Azure AD 目錄產生關聯，請讓目錄成為裝載資料庫之 Azure 訂用帳戶信任的目錄。如需詳細資訊，請參閱 [Azure 訂用帳戶如何與 Azure AD 產生關聯](https://msdn.microsoft.com/library/azure/dn629581.aspx)。

**其他資訊：**每個 Azure 訂用帳戶都會與 Azure AD 執行個體有信任關係。這表示它信任該目錄來驗證使用者、服務和裝置。多個訂用帳戶可以信任相同的目錄，但是一個訂用帳戶只能信任一個目錄。您可以在 [設定] 索引標籤下的 [https://manage.windowsazure.com/](https://manage.windowsazure.com/) 看到訂用帳戶所信任的目錄。這個訂用帳戶與目錄之間存在的信任關係不同於訂用帳戶與所有其他 Azure 資源 (網站、資料庫等) 之間的關係，後者比較像是訂用帳戶的子資源。如果訂用帳戶已過期，則也會停止存取與該訂用帳戶相關聯的其他資源。但目錄會保留在 Azure 中，而且您可以將其他訂用帳戶與該目錄產生關聯，並繼續管理目錄使用者。如需有關資源的詳細資訊，請參閱[了解 Azure 中的資源存取](https://msdn.microsoft.com/library/azure/dn584083.aspx)。

下列程序會提供如何變更特定訂用帳戶的相關聯目錄的逐步指示。

1. 使用 Azure 訂用帳戶管理員連接到您的 [Azure 傳統入口網站](https://manage.windowsazure.com/)。
2. 在左邊的橫幅中，選取 [設定]。
3. 您的訂用帳戶會出現在 [設定] 畫面中。如果未出現所需的訂用帳戶，請按一下頂端的 [訂用帳戶]，拉下 [依目錄篩選] 方塊並選取包含您訂用帳戶的目錄，然後按一下 [套用]。

	![select subscription][4]
4. 在 [設定] 區域中，按一下您的訂用帳戶，然後按一下頁面底部的 [編輯目錄]。

	![ad-settings-portal][5]
5. 在 [編輯目錄] 方塊中，選取與您的 SQL Server 或 SQL 資料倉儲相關聯的 Azure Active Directory，然後按一下箭號進行下一步。

	![edit-directory-select][6]
6. 在 [確認目錄對應] 對話方塊中，確認「**將移除所有的共同管理員**」。

	![edit-directory-confirm][7]
7. 按一下核取記號重新載入入口網站。

> [AZURE.NOTE] 當您變更目錄時，所有共同管理員、Azure AD 使用者和群組以及以目錄為基礎之資源使用者的存取權都會被移除，他們將不再有此訂用帳戶或其資源的存取權。只有具備服務管理員身分的您能夠根據新的目錄設定主體的存取權。這項變更可能需要相當長的時間才會傳播到所有資源。變更目錄也會變更 SQL Database 和「SQL 資料倉儲」的 Azure AD 系統管理員，並且不允許任何現有 Azure AD 使用者進行資料庫存取。必須重設 Azure AD 系統管理員 (如下所述) 且必須建立新的 Azure AD 使用者。

## 4\.建立 Azure SQL Server 的 Azure AD 系統管理員

每個 Azure SQL Server (裝載 SQL Database 或「SQL 資料倉儲」) 一開始都只有一個伺服器系統管理員帳戶，也就是整個 Azure SQL Server 的系統管理員。第二個 SQL Server 系統管理員必須建立，也就是 Azure AD 帳戶。這個主體會在 master 資料庫中建立為自主資料庫使用者。身為系統管理員，伺服器系統管理員帳戶是每個使用者資料庫中的 **db\_owner** 角色成員，並且會進入每個使用者資料庫做為 **dbo** 使用者。如需伺服器系統管理員帳戶的詳細資訊，請參閱 [Azure SQL Database 安全性方針和限制](sql-database-security-guidelines.md)中的[管理資料庫和登入 Azure SQL Database](sql-database-manage-logins.md) 和**登入和使用者** 等節。

將 Azure Active Directory 與異地複寫搭配使用時，必須為主要和次要伺服器設定 Azure Active Directory 系統管理員。如果伺服器沒有 Azure Active Directory 系統管理員，則 Azure Active Directory 登入和使用者將會收到「無法連線至伺服器」錯誤。

> [AZURE.NOTE] 使用者如果不是以 Azure AD 帳戶 (包括 Azure SQL Server 系統管理員帳戶) 為基礎，就無法建立以 Azure AD 為基礎的使用者，因為他們不具備向 Azure AD 驗證建議之資料庫使用者的權限。

### 使用 Azure 入口網站佈建 Azure SQL Server 的 Azure Active Directory 系統管理員

1. 在 [Azure 入口網站](https://portal.azure.com/)的右上角，按一下您的連線以拉下可能的 Active Directory 清單。選擇正確的 Active Directory 做為預設 Azure AD。此步驟利用 Azure SQL Server 連結與 Active Directory 相關聯的訂用帳戶，確定 Azure AD 和 SQL Server 使用相同的訂用帳戶。(Azure SQL Server 可以裝載 Azure SQL Database 或 Azure SQL 資料倉儲。)

	![choose-ad][8]
2. 在左邊的橫幅中選取 [SQL 伺服器]、選取您的 **SQL Server**，然後在 [SQL Server] 刀鋒視窗的頂端按一下 [設定]。

	![ad 設定][9]
3. 在 [設定] 刀鋒視窗中，按一下 [**Active Directory 系統管理員]。
4. 在 [Active Directory 系統管理員] 刀鋒視窗中，按一下 [Active Directory 系統管理員]，然後在頂端按一下 [設定系統管理員]。
5. 在 [新增系統管理員] 刀鋒視窗中，搜尋使用者，選取使用者或群組成為系統管理員，然後按一下 [選取]。([Active Directory 系統管理員] 刀鋒視窗會顯示您 Active Directory 的所有成員與群組。呈現灰色的使用者或群組無法選取，因為他們不受支援成為 Azure AD 系統管理員。(請參閱以上 **Azure AD 功能和限制**中支援的系統管理員清單。) 以角色為基礎的存取控制 (RBAC) 只會套用至入口網站，並且不會傳播至 SQL Server。
6. 在 [Active Directory 系統管理員] 刀鋒視窗頂端，按一下 [儲存]。![選擇系統管理員][10]

	變更系統管理員的程序可能需要幾分鐘的時間。接著，新的系統管理員就會出現在 [Active Directory 系統管理員] 方塊中。

> [AZURE.NOTE] 設定 Azure AD 系統管理員時，新的系統管理員名稱 (使用者或群組) 不可以已經存在於虛擬主要資料庫中作為 SQL Server 驗證使用者。如果存在，Azure AD 系統管理員設定將會失敗；其中會復原其建立並指出這樣的系統管理員 (名稱) 已經存在。由於這類 SQL Server 驗證使用者並非 Azure AD 的成員，因此使用 Azure AD 驗證來連線到伺服器的一切努力都會失敗。

若要稍後移除系統管理員，請在 [Active Directory 系統管理員] 刀鋒視窗頂端，按一下 [移除系統管理員]，然後按一下 [儲存]。

### 使用 PowerShell 佈建 Azure SQL Server 的 Azure AD 系統管理員

若要執行 PowerShell Cmdlet，Azure PowerShell 必須已安裝且正在執行中。如需詳細資訊，請參閱[如何安裝和設定 Azure PowerShell](../powershell-install-configure.md)。

若要佈建 Azure AD 系統管理員，請執行下列 Azure PowerShell 命令：

- Add-AzureRmAccount
- Select-AzureRmSubscription


用來佈建和管理 Azure AD 系統管理員的 Cmdlet：

| Cmdlet 名稱 | 說明 |
|---------------------------------------------------|-----------------------------------------------------------------------------------------------------------------|
| [Set-AzureRmSqlServerActiveDirectoryAdministrator](https://msdn.microsoft.com/library/azure/mt603544.aspx) | 佈建 Azure SQL Server 或 Azure SQL 資料倉儲的 Azure Active Directory 系統管理員(必須來自目前的訂用帳戶。) |
| [Remove-AzureRmSqlServerActiveDirectoryAdministrator](https://msdn.microsoft.com/library/azure/mt619340.aspx) | 移除 Azure SQL Server 或 Azure SQL 資料倉儲的 Azure Active Directory 系統管理員 |
| [Get-AzureRmSqlServerActiveDirectoryAdministrator](https://msdn.microsoft.com/library/azure/mt603737.aspx) | 傳回目前為 Azure SQL Server 或 Azure SQL 資料倉儲設定的 Azure Active Directory 系統管理員的相關資訊。 |

使用 PowerShell 命令 get-help 來查看這當中每個命令的詳細資料，例如 ``get-help Set-AzureRmSqlServerActiveDirectoryAdministrator``。

下列指令碼會在名為 **Group-23** 的資源群組中，為 **demo\_server** 伺服器佈建名為 **DBA\_Group** (物件識別碼 `40b79501-b343-44ed-9ce7-da4c8cc7353f`) 的 Azure AD 系統管理員群組：

```
Set-AzureRmSqlServerActiveDirectoryAdministrator –ResourceGroupName "Group-23"
–ServerName "demo_server" -DisplayName "DBA_Group"
```

**DisplayName** 輸入參數可接受 Azure AD 顯示名稱或「使用者主體名稱」。例如 ``DisplayName="John Smith"`` 和 ``DisplayName="johns@contoso.com"``。Azure AD 群組只支援 Azure AD 顯示名稱。

> [AZURE.NOTE] Azure PowerShell 命令 ```Set-AzureRmSqlServerActiveDirectoryAdministrator``` 不會阻止您為不支援的使用者佈建 Azure AD 系統管理員。您可以佈建不支援的使用者，但是該使用者無法連線到資料庫。(請參閱以上 **Azure AD 功能和限制**中支援的系統管理員清單)。

下列範例使用選用的 **ObjectID**：

```
Set-AzureRmSqlServerActiveDirectoryAdministrator –ResourceGroupName "Group-23"
–ServerName "demo_server" -DisplayName "DBA_Group" -ObjectId "40b79501-b343-44ed-9ce7-da4c8cc7353f"
```

> [AZURE.NOTE] 當 **DisplayName** 並非唯一時，就需要 Azure AD **ObjectID**。若要擷取 **ObjectID** 和 **DisplayName** 的值，請使用 Azure 傳統入口網站的 [Active Directory] 區段，然後檢視使用者或群組的屬性。

下列範例會傳回 Azure SQL Server 的目前 Azure AD 系統管理員的相關資訊：

```
Get-AzureRmSqlServerActiveDirectoryAdministrator –ResourceGroupName "Group-23" –ServerName "demo_server" | Format-List
```

下列範例會移除 Azure AD 系統管理員：
```
Remove-AzureRmSqlServerActiveDirectoryAdministrator -ResourceGroupName "Group-23" –ServerName "demo_server"
```

您也可以使用 REST API 來佈建 Azure Active Directory 系統管理員。如需詳細資訊，請參閱 [Azure SQL Database 之 Azure SQL Database 作業的 Service Management REST API 參考和作業](https://msdn.microsoft.com/library/azure/dn505719.aspx)

## 5\.設定用戶端電腦

在您的應用程式或使用者使用 Azure AD 身分識別連接到 Azure SQL Database 或 Azure SQL 資料倉儲的所有用戶端電腦上，您必須安裝下列軟體：

- 從 [https://msdn.microsoft.com/library/5a4x27ek.aspx](https://msdn.microsoft.com/library/5a4x27ek.aspx) 安裝 .NET Framework 4.6 或更新版本。
- 從下載中心的 [Microsoft Active Directory Authentication Library for Microsoft SQL Server](http://www.microsoft.com/download/details.aspx?id=48742)，可以下載多種語言 (x86 和 amd64) 的 Azure Active Directory Authentication Library for SQL Server (**ADALSQL.DLL**)。

### 工具

- 安裝 [SQL Server 2016 Management Studio](https://msdn.microsoft.com/library/mt238290.aspx) 或 [SQL Server Data Tools for Visual Studio 2015](https://msdn.microsoft.com/library/mt204009.aspx) 皆可符合 .NET Framework 4.6 需求。
- SSMS 安裝 x86 版 **ADALSQL.DLL**。
- SSDT 會安裝 amd64 版的 **ADALSQL.DLL**。
- 來自 [Visual Studio 下載](https://www.visualstudio.com/downloads/download-visual-studio-vs)的最新 Visual Studio 符合 .NET Framework 4.6 需求，但不會安裝必要的 amd64 版 **ADALSQL.DLL**。

## 6\.在對應至 Azure AD 身分識別的資料庫中建立自主資料庫使用者

### 關於自主資料庫使用者

Azure Active Directory 驗證需要建立資料庫使用者做為自主資料庫使用者。以 Azure AD 身分識別為基礎的自主資料庫使用者係指在 master 資料庫中沒有登入身分的資料庫使用者，並且此使用者會對應至 Azure AD 目錄中與資料庫關聯的身分識別。Azure AD 身分識別可以是個別的使用者帳戶或群組。如需有關自主資料庫使用者的詳細資訊，請參閱[自主資料庫使用者 - 使資料庫可攜](https://msdn.microsoft.com/library/ff929188.aspx)。

> [AZURE.NOTE] 您無法使用入口網站來建立資料庫使用者 (系統管理員除外)。RBAC 角色不會傳播至 SQL Server、SQL Database 或「SQL 資料倉儲」。Azure RBAC 角色可用來管理 Azure 資源，並不會套用到資料庫權限。例如，「SQL Server 參與者」角色不會授與可連線到 SQL Database 或「SQL 資料倉儲」的存取權。存取權限必須使用 Transact-SQL 陳述式直接在資料庫中授與。

### 使用 SQL Server Management Studio 或 SQL Server Data Tools 連接到使用者資料庫或資料倉儲

若要確認 Azure AD 系統管理員已正確設定，請使用 Azure AD 系統管理員帳戶連接到 **master** 資料庫。若要佈建以 Azure AD 為基礎的自主資料庫使用者 (而非擁有資料庫的伺服器系統管理員)，請利用有權存取資料庫的 Azure AD 身分識別連線到資料庫。

> [AZURE.IMPORTANT] Visual Studio 2015 中的 [SQL Server 2016 Management Studio](https://msdn.microsoft.com/library/mt238290.aspx) 和 [SQL Server Data Tools](https://msdn.microsoft.com/library/mt204009.aspx) 提供 Azure Active Directory 驗證支援。SSMS 的 2016 年 8 月版本也支援 Active Directory 通用驗證，讓系統管理員能夠使用電話、簡訊、含有 PIN 的智慧卡或行動應用程式通知來要求 Multi-Factor Authentication。

#### 使用 Active Directory 整合式驗證進行連接

如果您已使用 Azure Active Directory 認證從同盟網域登入 Windows，請使用這個方法。

1. 啟動 Management Studio 或 Data Tools，並在 [連接到伺服器]\(或 [連接到 Database Engine]) 對話方塊的 [驗證] 方塊中，選取 [Active Directory 整合式驗證]。不需要密碼或沒有密碼可輸入，因為現有的認證將會在連接時出現。

![選取 AD 整合式驗證][11]

2. 按一下 [選項] 按鈕，然後在 [連接屬性] 頁面的 [連接到資料庫] 方塊中，輸入您想要連線的使用者資料庫名稱。

![選取資料庫名稱][13]


#### 使用 Active Directory 密碼驗證進行連接

使用 Azure AD 受管理網域連接到 Azure AD 主體名稱時，請使用這個方法。您也可以將其用於沒有網域存取權的同盟帳戶，例如在遠端運作時。

如果您使用認證從未與 Azure 建立同盟的網域登入 Windows，或在使用 Azure AD 驗證時使用以初始或用戶端網域為基礎的 Azure AD，請使用這個方法。

1. 啟動 Management Studio 或 Data Tools，並在 [連接到伺服器]\(或 [連接到 Database Engine]) 對話方塊的 [驗證] 方塊中，選取 [Active Directory 密碼驗證]。
2. 在 [使用者名稱] 方塊中，以 **username@domain.com** 格式輸入您的 Azure Active Directory 使用者名稱。這必須是來自 Azure Active Directory 的帳戶或來自與 Azure Active Directory 建立同盟之網域的帳戶。
3. 在 [密碼] 方塊中，輸入您的 Azure Active Directory 帳戶或同盟網域帳戶的使用者密碼。

	![選取 AD 密碼驗證][12]

4. 按一下 [選項] 按鈕，然後在 [連接屬性] 頁面的 [連接到資料庫] 方塊中，輸入您想要連線的使用者資料庫名稱。(請參閱上一個選項中的圖形。)


### 在使用者資料庫中建立 Azure AD 自主資料庫使用者

若要建立以 Azure AD 為基礎的自主資料庫使用者 (而非擁有資料庫的伺服器系統管理員)，請以至少具有 **ALTER ANY USER** 權限的使用者身分，使用 Azure AD 身分識別來連線到資料庫。然後使用下列的 Transact-SQL 語法：

	CREATE USER <Azure_AD_principal_name>
	FROM EXTERNAL PROVIDER;


*Azure\_AD\_principal\_name* 可以是 Azure AD 使用者的使用者主體名稱或 Azure AD 群組的顯示名稱。

**範例：**
建立代表 Azure AD 同盟或受管理網域使用者的自主資料庫使用者：

	CREATE USER [bob@contoso.com] FROM EXTERNAL PROVIDER;
	CREATE USER [alice@fabrikam.onmicrosoft.com] FROM EXTERNAL PROVIDER;

若要建立代表 Azure AD 或同盟網域群組的自主資料庫使用者，請輸入安全性群組的顯示名稱：

	CREATE USER [ICU Nurses] FROM EXTERNAL PROVIDER;

若要建立代表使用 Azure AD 權杖進行連線之應用程式的自主資料庫使用者︰

	CREATE USER [appName] FROM EXTERNAL PROVIDER;

如需有關根據 Azure Active Directory 身分識別建立自主資料庫使用者的詳細資訊，請參閱 [CREATE USER (Transact-SQL)](http://msdn.microsoft.com/library/ms173463.aspx)。


> [AZURE.NOTE] 移除 Azure SQL Server 的 Azure Active Directory 系統管理員可防止任何 Azure AD 驗證使用者連線到伺服器。必要時，SQL Database 系統管理員可以手動刪除無法使用的 Azure AD 使用者。

當您建立資料庫使用者時，該使用者會獲得 **CONNECT** 權限，而可以用 **PUBLIC** 角色的成員身分連線到該資料庫。一開始提供給使用者的權限僅限於已授與 **PUBLIC** 角色的任何權限，或已授與其所屬任何 Windows 群組的任何權限。一旦您佈建以 Azure AD 為基礎的自主資料庫使用者，您可以使用您授與權限給任何其他類型使用者的相同方式，授與該使用者額外的權限。通常會授與權限給資料庫角色並新增使用者至角色。如需詳細資訊，請參閱 [Database Engine 權限基本概念](http://social.technet.microsoft.com/wiki/contents/articles/4433.database-engine-permission-basics.aspx)。如需有關特殊 SQL Database 角色的詳細資訊，請參閱[管理 Azure SQL Database 的資料庫和登入](sql-database-manage-logins.md)。匯入至管理網域的同盟網域使用者必須使用受管理的網域身分識別。

> [AZURE.NOTE] Azure AD 使用者會在資料庫中繼資料中標示為類型 E (EXTERNAL\_USER)，而群組則標示為類型 X (EXTERNAL\_GROUPS)。如需詳細資訊，請參閱 [sys.database\_principals](https://msdn.microsoft.com/library/ms187328.aspx)。


## 7\.使用 Azure AD 身分識別連接

Azure Active Directory 驗證支援下列方法，使用 Azure AD 身分識別連接至資料庫：

- 使用整合式 Windows 驗證
- 使用 Azure AD 主體名稱和密碼
- 使用應用程式權杖驗證

### 7\.1.使用整合式 (Windows) 驗證進行連接

若要使用整合式 Windows 驗證，您網域的 Active Directory 必須與 Azure Active Directory 建立同盟關係。連接到資料庫的用戶端應用程式 (或服務) 必須以使用者的網域認證在已加入網域的電腦上執行。

若要使用整合式驗證以及 Azure AD 身分識別連接至資料庫，資料庫連接字串中的驗證關鍵字必須設定為 Active Directory 整合式。下列 C# 程式碼範例會使用 ADO.NET。

	string ConnectionString =
	@"Data Source=n9lxnyuzhv.database.windows.net; Authentication=Active Directory Integrated; Initial Catalog=testdb;";
	SqlConnection conn = new SqlConnection(ConnectionString);
	conn.Open();

請注意，不支援使用連接字串關鍵字 ``Integrated Security=True`` 來連線到 Azure SQL Database。請注意，建立 ODBC 連接時，您必須移除空格，並將「驗證」設為 'ActiveDirectoryIntegrated'。

### 7\.2.使用 Azure AD 主體名稱與密碼進行連接
若要使用整合式驗證和 Azure AD 身分識別來連接到資料庫，Authentication 關鍵字就必須設定為 Active Directory Password。連接字串必須包含使用者識別碼 (UID) 及密碼 (PWD) 關鍵字和值。下列 C# 程式碼範例會使用 ADO.NET。

	string ConnectionString =
	  @"Data Source=n9lxnyuzhv.database.windows.net; Authentication=Active Directory Password; Initial Catalog=testdb;  UID=bob@contoso.onmicrosoft.com; PWD=MyPassWord!";
	SqlConnection conn = new SqlConnection(ConnectionString);
	conn.Open();

深入了解使用 [Azure AD 驗證 GitHub 示範](https://github.com/Microsoft/sql-server-samples/tree/master/samples/features/security/azure-active-directory-auth)上所提供之示範程式碼範例的 Azure AD 驗證方法。


### 使用 Azure AD 權杖的 7.3 連接
這種驗證方法可以從 Azure Active Directory (AAD) 取得權杖，讓中介層服務連接到 Azure SQL Database 或 Azure SQL 資料倉儲。這可容許包含憑證型驗證的複雜案例。您必須完成四個基本步驟，才能使用 Azure AD 權杖驗證︰

1. 向 Azure Active Directory 註冊您的應用程式，並取得程式碼的用戶端識別碼。
2. 建立代表應用程式的資料庫使用者。(稍早在步驟 6 中已完成)。
3. 在執行應用程式的用戶端電腦上建立憑證。
4. 將憑證加入應用程式當做索引鍵。

範例連接字串︰

```
string ConnectionString =@"Data Source=n9lxnyuzhv.database.windows.net; Initial Catalog=testdb;"
SqlConnection conn = new SqlConnection(ConnectionString);
connection.AccessToken = "Your JWT token"
conn.Open();
```

如需詳細資訊，請參閱 [SQL Server 安全性部落格](https://blogs.msdn.microsoft.com/sqlsecurity/2016/02/09/token-based-authentication-support-for-azure-sql-db-using-azure-ad-auth/)。

### 使用 sqlcmd 進行連線  
下列陳述式中使用 sqlcmd 13.1 進行連線，從[下載中心](http://go.microsoft.com/fwlink/?LinkID=825643)即可取得此版本。

```
sqlcmd -S Target_DB_or_DW.testsrv.database.windows.net  -G  
sqlcmd -S Target_DB_or_DW.testsrv.database.windows.net -U bob@contoso.com -P MyAADPassword -G -l 30
```

## 另請參閱

[管理 Azure SQL Database 的資料庫和登入](sql-database-manage-logins.md)

[自主資料庫使用者](https://msdn.microsoft.com/library/ff929071.aspx)

[CREATE USER (Transact-SQL)](http://msdn.microsoft.com/library/ms173463.aspx)


<!--Image references-->

[1]: ./media/sql-database-aad-authentication/1aad-auth-diagram.png
[2]: ./media/sql-database-aad-authentication/2subscription-relationship.png
[3]: ./media/sql-database-aad-authentication/3admin-structure.png
[4]: ./media/sql-database-aad-authentication/4select-subscription.png
[5]: ./media/sql-database-aad-authentication/5ad-settings-portal.png
[6]: ./media/sql-database-aad-authentication/6edit-directory-select.png
[7]: ./media/sql-database-aad-authentication/7edit-directory-confirm.png
[8]: ./media/sql-database-aad-authentication/8choose-ad.png
[9]: ./media/sql-database-aad-authentication/9ad-settings.png
[10]: ./media/sql-database-aad-authentication/10choose-admin.png
[11]: ./media/sql-database-aad-authentication/11connect-using-int-auth.png
[12]: ./media/sql-database-aad-authentication/12connect-using-pw-auth.png
[13]: ./media/sql-database-aad-authentication/13connect-to-db.png

<!---HONumber=AcomDC_0921_2016-->