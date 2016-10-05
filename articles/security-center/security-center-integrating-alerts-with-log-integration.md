<properties
   pageTitle="整合 Azure 資訊安全中心警示與 Azure 記錄整合 (預覽) | Microsoft Azure"
   description="本文可協助您開始整合資訊安全中心警示與 Azure 記錄整合。"
   services="security-center"
   documentationCenter="na"
   authors="TerryLanfear"
   manager="MBaldwin"
   editor=""/>

<tags
   ms.service="security-center"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="08/08/2016"
   ms.author="terrylan"/>

# 整合資訊安全中心警示與 Azure 記錄整合 (預覽)

許多安全性作業和事件回應小組依賴安全性資訊及事件管理 (SIEM) 方案對安全性警示進行分級和調查做為起點。利用 Azure 記錄整合，客戶可以將 Azure 資訊安全中心警示，以及 Azure 診斷和 Azure 稽核記錄檔所收集的虛擬機器安全性事件，與其 Log Analytics 或 SIEM 方案以接近即時的方式進行同步處理。

Azure 記錄整合可與 HP ArcSight、Splunk、IBM QRadar 等等搭配運作。

## 可以整合哪些記錄檔？

Azure 會為每項服務產生大量記錄。這些記錄檔會分類為︰

- **控制/管理記錄檔**：可讓您看到 Azure Resource Manager 的 CREATE、UPDATE 和 DELETE 作業。
- **資料平面記錄檔**：可讓您看到使用 Azure 資源時所引發的事件。其中一例就是 Windows 事件記錄檔，這是虛擬機器中的安全性和應用程式記錄檔。

Azure 記錄檔整合目前支援下列項目的整合︰

- Azure VM 記錄檔
- Azure 稽核記錄檔
- Azure 資訊安全中心警示

## 安裝 Azure 記錄檔整合

下載 [Azure 記錄檔整合](https://www.microsoft.com/download/details.aspx?id=53324)。

Azure 記錄檔整合服務會從其安裝所在的電腦收集遙測資料。所收集的遙測資料是︰

- Azure 記錄檔整合的執行期間所發生的例外狀況
- 所處理之查詢和事件數目的相關度量
- 正在使用哪些 Azlog.exe 命令列選項的相關統計資料

> [AZURE.NOTE] 您可以取消核取此選項，以關閉遙測資料的收集。

## 整合 Azure 稽核記錄檔和資訊安全中心警示

1. 開啟命令提示字元，並使用 **cd** 命令前往 **c:\\Program Files\\Microsoft Azure Log Integration**。

2. 執行 **azlog createazureid** 命令，在裝載 Azure 訂用帳戶的 Azure Active Directory (AD) 租用戶中建立 [Azure Active Directory 服務主體](../active-directory/active-directory-application-objects.md)。

    系統會提示您登入 Azure。

    > [AZURE.NOTE] 您必須是訂用帳戶擁有者或訂用帳戶的共同管理員。

    對 Azure 的驗證會透過 Azure AD 來進行。建立 Azure 記錄檔整合的服務主體將會建立 Azure AD 身分識別，以獲得 Azure 訂用帳戶的讀取權限。

3. 執行 **azlog authorize <SubscriptionID>** 命令，以將訂用帳戶的讀取者權限指派給在步驟 2 中建立的服務主體。如果您未指定 **SubscriptionID**，則會對服務主體指派您可以存取之所有訂用帳戶的讀取者角色。

    > [AZURE.NOTE] 在 Azure AD 帳戶建立後到帳戶可供使用中間會有一些延遲時間，因此，如果您在 **createazureid** 命令之後緊接著執行 **authorize** 命令，將可能會看見警告。如果您在執行 **createazureid** 命令後等待約 10 秒再執行 **authorize** 命令，應該就不會看到這些警告。

4. 檢查下列資料夾以確認其中有稽核記錄檔 JSON 檔案︰

  - **c:\\Users\\azlog\\AzureResourceManagerJson**
  - **c:\\Users\\azlog\\AzureResourceManagerJsonLD**

5. 檢查下列資料夾以確認其中有資訊安全中心警示︰

  - **c:\\Users\\azlog\\ AzureSecurityCenterJson**
  - **c:\\Users\\azlog\\AzureSecurityCenterJsonLD**

6. 將標準的 SIEM 檔案轉寄站連接器指向適當資料夾，以透過管線將資料傳送至 SIEM 執行個體。請參閱 [SIEM 組態](https://azsiempublicdrops.blob.core.windows.net/drops/ALL.htm)以取得您的 SIEM 組態。

如果您有關於「Azure 記錄整合」的問題，請傳送電子郵件給 [AzSIEMteam@microsoft.com](mailto:AzSIEMteam@microsoft.com)

## 後續步驟

若要深入了解 Azure 稽核記錄檔和屬性定義，請參閱︰

- [使用 Resource Manager 來稽核作業](../resource-group-audit.md)
- [列出訂用帳戶中的管理事件](https://msdn.microsoft.com/library/azure/dn931934.aspx) - 以擷取稽核記錄檔事件。

如要深入了解資訊安全中心，請參閱下列主題：

- [管理與回應 Azure 資訊安全中心的安全性警示](security-center-managing-and-responding-alerts.md) — 了解如何管理與回應安全性警示。
- [Azure 資訊安全中心常見問題集](security-center-faq.md) — 尋找有關使用服務的常見問題。
- [Azure 安全性部落格](http://blogs.msdn.com/b/azuresecurity/) — 取得最新的 Azure 安全性新聞和資訊。

<!---HONumber=AcomDC_0921_2016-->