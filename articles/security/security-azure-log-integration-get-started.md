<properties
   pageTitle="開始使用 Azure 記錄整合 |Microsoft Azure"
   description="學習如何安裝 Azure 記錄整合服務，和來自 Azure 儲存體、Azure 稽核記錄檔以及 Azure 資訊安全中心警示的整合記錄檔。"
   services="security"
   documentationCenter="na"
   authors="TomShinder"
   manager="MBaldwin"
   editor="TerryLanfear"/>

<tags
   ms.service="security"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="08/24/2016"
   ms.author="TomSh"/>

# 開始使用 Azure 記錄整合 (預覽)

Azure 記錄整合可讓您將來自 Azure 資源的未經處理記錄，整合到內部部署安全性資訊及事件管理 (SIEM) 系統內。這項整合提供您內部部署或在雲端中所有資產統一的儀表板，以便您彙總、相互關聯、分析和警示與應用程式相關聯的安全性事件。

本教學課程將逐步引導您安裝 Azure 記錄整合，和來自 Azure 儲存體、Azure 稽核記錄檔以及 Azure 資訊安全中心警示的整合記錄檔。本教學課程的預估完成時間是一小時。

## 必要條件

若要完成本教學課程，您必須具備下列項目：

- 要安裝 Azure 記錄整合服務 (內部部署或在雲端) 的電腦。這台電腦執行的必須是 64 位元 Windows 作業系統，並已安裝 .Net 4.5.1。這台電腦稱為 **Azlog Integrator**。
- Azure 訂用帳戶。如果您沒有訂用帳戶，您可以註冊[免費帳戶](https://azure.microsoft.com/free/)。
- 已啟用 Azure 虛擬機器 (VM) 的 Azure 診斷。若要啟用雲端服務的診斷功能，請參閱[啟用 Azure 雲端服務中的 Azure 診斷](../cloud-services/cloud-services-dotnet-diagnostics.md)。若要啟用執行 Windows 的 Azure VM，請參閱[使用 PowerShell 在執行 Windows 的虛擬機器中啟用 Azure 診斷](../virtual-machines/virtual-machines-windows-ps-extensions-diagnostics.md)。
- 從 Azlog 整合器到 Azure 儲存體，以及驗證和授權 Azure 訂用帳戶用的連線。
- 對於 Azure VM 記錄檔，SIEM 代理程式 (例如，Splunk Universal Forwarder、HP ArcSight Windows Event Collector 代理程式或 IBM QRadar WinCollect) 必須安裝在 Azlog 整合器上。

## 部署考量

如果事件量很大，您可以執行多個 Azlog 整合器執行個體。Windows 的 Azure 診斷儲存體帳戶 *(WAD)* 和提供給執行個體的訂用帳戶數目之間的負載平衡應該要根據容量。

在 8 個處理器 (核心) 的電腦上，單一的 Azlog 整合器每日可以處理大約 2400 萬個事件 (~1 百萬個/小時)。

在 4 個處理器 (核心) 的電腦上，單一的 Azlog 整合器每日可以處理大約 1500 萬個事件 (~6 萬 2 千 5 百個/小時)。

## 安裝 Azure 記錄檔整合

下載 [Azure 記錄檔整合](https://www.microsoft.com/download/details.aspx?id=53324)。

Azure 記錄檔整合服務會從其安裝所在的電腦收集遙測資料。所收集的遙測資料是︰

- Azure 記錄檔整合的執行期間所發生的例外狀況
- 所處理之查詢和事件數目的相關度量
- 正在使用哪些 Azlog.exe 命令列選項的相關統計資料

> [AZURE.NOTE] 您可以取消核取此選項，以關閉遙測資料的收集。

## 整合來自您 Azure 診斷儲存體帳戶的 Azure VM 記錄檔

1. 檢查上面所列的必要條件以確保您的 WAD 儲存體帳戶會收集記錄檔，然後再繼續您的 Azure 記錄整合。如果您的 WAD 儲存體帳戶不會收集記錄檔，請不要執行下列步驟。

2. 開啟命令提示字元，並使用 **cd** 命令前往 **c:\\Program Files\\Microsoft Azure Log Integration**。

3. 執行命令

        azlog source add <FriendlyNameForTheSource> WAD <StorageAccountName> <StorageKey>

      以設定為接收來自您 VM 診斷事件的 Azure 儲存體帳戶名稱，來取代 StorageAccountName。

        azlog source add azlogtest WAD azlog9414 fxxxFxxxxxxxxywoEJK2xxxxxxxxxixxxJ+xVJx6m/X5SQDYc4Wpjpli9S9Mm+vXS2RVYtp1mes0t9H5cuqXEw==

      如果您想要訂用帳戶識別碼顯示在事件 XML 中，請將訂用帳戶識別碼附加至易記名稱︰

        azlog source add <FriendlyNameForTheSource>.<SubscriptionID> WAD <StorageAccountName> <StorageKey>

4. 等候 30-60 分鐘 (可能需要長達一小時)，然後檢視從儲存體帳戶提取的事件。若要檢視，請在 Azlog 整合器上開啟 [事件檢視器] > [Windows 記錄] > [轉送的事件]。

5. 請確定您的電腦上所安裝的標準 SIEM 連接器已設定為從 [轉送的事件] 資料夾挑選事件，然後將其輸送至您的 SIEM 執行個體。檢閱 SIEM 特定的組態以設定並查看正在整合的記錄檔。

## 要是資料未顯示在 [轉送的事件] 資料夾呢？

如果一個小時之後資料未顯示在 [轉送的事件] 資料夾，然後︰

1. 檢查電腦，並確認它可以存取 Azure。若要測試連線，請嘗試從瀏覽器開啟 [Azure 入口網站](http://portal.azure.com)。

2. 請確定使用者帳戶 **azlog** 具有 **users\\azlog** 資料夾的寫入權限。

3. 請確定執行 **azlog source list** 命令時，會列出 **azlog source add** 命令中新增的儲存體帳戶。
4. 移至 [事件檢視器] > [Windows 記錄] > [應用程式]，以查看是否有任何來自 Azure 記錄整合的錯誤報告。

如果您仍然看不到事件，然後︰

1. 下載 [Microsoft Azure 儲存體總管](http://storageexplorer.com/)。

2. 連線到 **azlog source add** 命令中新增的儲存體帳戶。

3. 在 Microsoft Azure 儲存體總管中，瀏覽至 **WADWindowsEventLogsTable** 資料表以查看是否有任何資料。如果沒有，則在 VM 中的診斷設定不正確。

## 整合 Azure 稽核記錄檔和資訊安全中心警示

1. 開啟命令提示字元，並使用 **cd** 命令前往 **c:\\Program Files\\Microsoft Azure Log Integration**。

2. 執行命令

        azlog createazureid

      此命令會提示您登入 Azure。此命令接著會建立託管 Azure 訂用帳戶 (其中登入的使用者是系統管理員、共同管理員或擁有者) 的 Azure AD 租用戶中的 [Azure Active Directory 服務主體](../active-directory/active-directory-application-objects.md)。若登入的使用者只是 Azure AD 租用戶中的來賓使用者，則此命令將會失敗。對 Azure 的驗證會透過 Azure Active Directory (AD) 來進行。建立 Azure 記錄整合的服務主體會建立 Azure AD 身分識別，以獲得 Azure 訂用帳戶的讀取權限。

3. 執行命令

        azlog authorize <SubscriptionID>

      這會將在訂用帳戶上的讀取者存取權，指派給在步驟 2 中所建立的服務主體。如果您未指定 SubscriptionID，則會嘗試將服務主體的讀取者角色，指派給您有任何存取權的所有訂用帳戶。

        azlog authorize 0ee9d577-9bc4-4a32-a4e8-c29981025328

      > [AZURE.NOTE] 如果您在執行 **createazureid** 命令之後立即執行 **authorize** 命令，可能會看見警告。建立 Azure AD 帳戶到帳戶可供使用之間會有一些延遲。如果您在執行 **createazureid** 命令後等待約 10 秒再執行 **authorize** 命令，應該就不會看到這些警告。

4. 檢查下列資料夾以確認其中有稽核記錄檔 JSON 檔案︰

  - **c:\\Users\\azlog\\AzureResourceManagerJson**
  - **c:\\Users\\azlog\\AzureResourceManagerJsonLD**

5. 檢查下列資料夾以確認其中有資訊安全中心警示︰

  - **c:\\Users\\azlog\\ AzureSecurityCenterJson**
  - **c:\\Users\\azlog\\AzureSecurityCenterJsonLD**

6. 將標準的 SIEM 檔案轉寄站連接器指向適當資料夾，以透過管線將資料傳送至 SIEM 執行個體。您可能需要根據所使用的 SIEM 產品進行某些欄位對應。

如果您有關於「Azure 記錄整合」的問題，請傳送電子郵件給 [AzSIEMteam@microsoft.com](mailto:AzSIEMteam@microsoft.com)

## 後續步驟

在本教學課程中，您已了解如何安裝 Azure 記錄整合，並整合來自 Azure 儲存體的記錄檔。若要深入了解，請參閱下列文章：

- [Azure 記錄檔的 Microsoft Azure 記錄整合 (預覽)](https://www.microsoft.com/download/details.aspx?id=53324) – Azure 記錄整合詳細資料、系統需求和安裝指示的下載中心。
- [Azure 記錄整合簡介](security-azure-log-integration-overview.md) – 這份文件為您介紹 Azure 記錄整合、其主要功能以及運作方式。
- [合作夥伴設定步驟](https://blogs.msdn.microsoft.com/azuresecurity/2016/08/23/azure-log-siem-configuration-steps/) – 此部落格文章說明如何設定 Azure 記錄整合，以搭配使用合作夥伴解決方案 Splunk、HP ArcSight 和 IBM QRadar。
- [Azure 記錄整合常見問題集 (FAQ)](security-azure-log-integration-faq.md) - 此常見問題集會回答有關 Azure 記錄整合的問題。
- [以 Azure 記錄整合來整合資訊安全中心警示](../security-center/security-center-integrating-alerts-with-log-integration.md) - 這份文件說明如何將資訊安全中心警示以及 Azure 診斷和 Azure 稽核記錄檔所收集的虛擬機器安全性事件，與您的 Log Analytics 或 SIEM 方案進行同步處理。
- [Azure 診斷和 Azure 稽核記錄檔的新功能](https://azure.microsoft.com/blog/new-features-for-azure-diagnostics-and-azure-audit-logs/) – 此部落格文章為您介紹 Azure 稽核記錄檔和其他功能，協助您深入了解您的 Azure 資源的作業。

<!---HONumber=AcomDC_0921_2016-->