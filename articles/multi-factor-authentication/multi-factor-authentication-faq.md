<properties
	pageTitle="Azure Multi-Factor Authentication 常見問題集"
	description="提供與 Azure Multi-Factor Authentication 相關的常見問題與答案清單。Multi-Factor Authentication 是一種驗證使用者身分識別的方法。它除了需要使用者名稱與密碼之外，還需要其他驗證方式。它可以為使用者登入和交易提供一層額外的安全性。"
	services="multi-factor-authentication"
	documentationCenter=""
	authors="kgremban"
	manager="femila"
	editor="curtand"/>

<tags
	ms.service="multi-factor-authentication"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/22/2016"
	ms.author="kgremban"/>

# Azure Multi-Factor Authentication 常見問題集


此常見問題集回答關於 Azure Multi-Factor Authentication 和使用 Multi-Factor Authentication 服務的常見問題，包括計費模式和可用性相關問題。

## 一般

**問：如何取得 Azure Multi-Factor Authentication 的協助？**

- [搜尋 Microsoft 支援知識庫](https://www.microsoft.com/zh-TW/Search/result.aspx?form=mssupport&q=phonefactor&form=mssupport)

  在知識庫搜尋常見技術問題的解決方案。

- [Microsoft Azure Active Directory 論壇](https://social.msdn.microsoft.com/Forums/azure/home?forum=WindowsAzureAD)

  在 [Azure Active Directory 論壇](https://social.msdn.microsoft.com/Forums/azure/newthread?category=windowsazureplatform&forum=WindowsAzureAD&prof=required)中，搜尋及瀏覽來自社群的技術問題與解答，或是提出您的問題。

- [密碼重設](mailto:phonefactorsupport@microsoft.com)

  如果您是舊版 PhoneFactor 的客戶且有問題或需要協助重設密碼，請使用[重設密碼](mailto:phonefactorsupport@microsoft.com)連結來建立支援案例。

- [Azure Multi-Factor Authentication Server (PhoneFactor) 支援](https://support.microsoft.com/oas/default.aspx?prid=14947)

  使用此連結可連絡 Microsoft 支援專業人員。他們會詢問您一些問題，以協助您決定您的支援選項，可能的選項有電子郵件、線上提交或電話支援。



**問：Azure Multi-Factor Authentication Server 如何處理使用者資料？**

當您使用 Multi-Factor Authentication Server 時，使用者的資料只會儲存在內部部署伺服器。雲端中不會儲存任何持續性的使用者資料。使用者執行雙因素驗證時，Multi-Factor Authentication Server 會將資料傳送到 Azure Multi-Factor Authentication 雲端服務以供驗證。當驗證要求傳送至雲端服務時，要求和記錄檔中會傳送下列清單欄位內的資料，以供在客戶的驗證和使用情況報告中使用。某些欄位是選擇性的，而且您可以在 Multi-Factor Authentication Server 中設定它們。從 Multi-Factor Authentication Server 到 Multi-Factor Authentication 雲端服務的通訊會透過傳出連接埠 443，使用安全通訊端層 (SSL) 或傳輸層安全性 (TLS) 通訊。雙因素驗證記錄檔中包含的資料欄位如下：

- **唯一識別碼** (使用者名稱或內部部署 Multi-Factor Authentication Server 識別碼兩者之一)
- **名字和姓氏** (選擇性)
- **電子郵件地址** (選擇性)
- **電話號碼** (進行語音通話或簡訊驗證時)
- **裝置權杖** (執行行動應用程式驗證時)
- **驗證模式**
- **驗證結果**
- **Multi-Factor Authentication Server 名稱**
- **Multi-Factor Authentication Server IP**
- **用戶端 IP** (如果有的話)



除了這些欄位，驗證結果 (成功或拒絕) 和任何拒絕的原因也會與驗證資料一起儲存，可在驗證和使用情況報告中取得。




## 計費

**問：透過電話或簡訊驗證我的使用者時，我的組織是否需要付費？**

所有成本都會計入服務的每位使用者或每次驗證成本。組織不需為透過 Azure Multi-Factor Authentication 對使用者撥打個別通話或傳送簡訊支付費用。電話擁有者可能會因為透過其電信服務業者接聽電話或接收簡訊，而造成漫遊相關或其他費用。

**問：組織使用 Azure Multi-Factor Authentication 時如何計費？**

Azure Multi-Factor Authentication 能當作獨立服務選購，並按照每位使用者和驗證次數來計費，也能與 Azure Active Directory Premium、Enterprise Mobility Suite 或 Enterprise Cloud Suite 合購。獨立服務可依據消費基礎針對組織的 Azure 承諾用量每月計費來取得，或是依據可透過 Microsoft Enterprise Agreement、Microsoft Open 授權方案、Microsoft 雲端解決方案提供者 (CSP) 方案或 Direct 取得的每位使用者每年授權來取得。

>[AZURE.IMPORTANT]
澳大利亞地區可供在澳大利亞或紐西蘭有營業據點的客戶使用。

計費模式 | 價格
------------- | ------------- |
每位使用者耗用量 (Azure 承諾用量)| 每個月 1.40 美元 (不限驗證次數)
每次驗證耗用量 (Azure 承諾用量)|每 10 次驗證 1.40 美元
每位使用者每年授權 (Direct)|每個月 1.40 美元 (不限驗證次數)
每位使用者每年授權 (大量授權)|請連絡您的 [Enterprise 合約代表](https://www.microsoft.com/zh-TW/licensing/licensing-programs/enterprise.aspx)

**問：「每位使用者」計費模式的收費是根據設定使用 Multi-Factor Authentication 的使用者數目或執行驗證的使用者數目？**

計費是根據設定使用 Multi-Factor Authentication 的使用者數目來計費。

**問：Multi-Factor Authentication 是如何計費？**

當組織使用「每位使用者」或「每次驗證」的耗用量計費和使用量模型時，組織的系統管理員會在系統管理員於 Azure 傳統入口網站中建立 Multi-Factor Authentication 提供者時選擇使用類型。這是根據組織的 Azure 訂用帳戶計費的資源，例如虛擬機器、網站等。當您使用授權模型時，需要購買 Azure Multi-Factor Authentication 授權，然後指派給使用者，就像 Office 365 和其他訂用帳戶產品一樣。

**問：是否有免費版本的 Azure Multi-Factor Authentication 供系統管理員使用？**

如果消費型 Azure Multi-Factor Authentication 提供者尚未連結至相對應的 Azure Active Directory 執行個體，我們會向 Azure Global Administrators 群組的成員免費提供一組稱為「Multi-Factor Authentication for Azure Administrators」的 Azure Multi-Factor Authentication 功能子集。使用 Multi-Factor Authentication 提供者時，會將目錄中已設定使用 Multi-Factor Authentication 的所有系統管理員和使用者升級為完整版本的 Azure Multi-Factor Authentication。

**問：是否有免費版本的 Azure Multi-Factor Authentication 供 Office 365 使用者使用？**

如果消費型 Azure Multi-Factor Authentication 提供者尚未連結至相對應 Azure Active Directory 的執行個體，我們會向已獲指派 Office 365 授權的使用者免費提供一組稱為「Multi-Factor Authentication for Office 365」的 Azure Multi-Factor Authentication 功能子集。使用 Multi-Factor Authentication 提供者時，會將目錄中已設定使用 Multi-Factor Authentication 的所有系統管理員和使用者升級為完整版本的 Azure Multi-Factor Authentication。

**問：我的組織是否能隨時在「每位使用者」與「每次驗證」耗用量計費模式之間切換？**

您的組織會在建立資源時選擇計費模式。資源佈建之後就無法變更計費模式。不過，您可以建立新的 Multi-Factor Authentication 資源來取代原始資源。請注意，使用者設定和組態選項無法轉移到新的資源。

**問：我的組織是否能隨時在「耗用量計費」和「授權」模式之間切換？**

您的組織可以隨時購買 Azure Multi-Factor Authentication、Azure Active Directory Premium、Enterprise Mobility Suite 和 Enterprise Cloud Suite 授權。對已有每位使用者 Azure Multi-Factor Authentication 提供者的目錄新增授權時，消費型計費也會減少所擁有的授權數量。如果所有設定使用 Multi-Factor Authentication 的使用者都已獲指派授權，系統管理員就可以刪除 Azure Multi-Factor Authentication 提供者。組織不能混用「每次驗證耗用量計費」與「授權」模式。當每次驗證 Multi-Factor Authentication 提供者連結至目錄時，不論組織是否擁有任何授權，都會對其收取所有 Multi-Factor Authentication 驗證要求的費用。

**問：我的組織是否必須使用和同步處理身分識別才能使用 Azure Multi-Factor Authentication？**

當組織使用消費型計費模式時，不需要 Azure Active Directory。您可以自行選擇是否要將 Multi-Factor Authentication Provider 連結至目錄。如果您的組織未連結至目錄，則可以將 Multi-Factor Authentication Server 或 Multi-Factor Authentication SDK 部署在內部。授權模式需要 Azure Active Directory，因為當您購買授權並指派給目錄中的使用者時，會將授權加入目錄。


## 可用性

**問：如果使用者沒有接到電話，或使用者無法取得該電話怎麼辦？**

如果使用者先前已設定備用電話，他們可再試一次，當登入頁面出現提示時選取該電話。如果使用者未設定其他方法，則應與組織的系統管理員連絡，請管理員更新指派為使用者主要電話 (手機或辦公室) 的號碼。


**問：如果使用者因無法再存取某個帳戶而連絡系統管理員，系統管理員要如何處理？**

系統管理員可以要求使用者再次進行註冊程序，以重設使用者的帳戶。深入了解[在雲端使用 Azure Multi-Factor Authentication 管理使用者和裝置設定](multi-factor-authentication-manage-users-and-devices.md)。

**問：如果使用者使用應用程式密碼的手機遺失或遭竊，系統管理員要如何處理？**

系統管理員可以刪除該使用者的所有應用程式密碼，以防止未經授權的存取。當使用者有替代的裝置之後，即可建立密碼。深入了解[在雲端使用 Azure Multi-Factor Authentication 管理使用者和裝置設定](multi-factor-authentication-manage-users-and-devices.md)。

**問：如果使用者無法登入非瀏覽器的應用程式該怎麼辦？**

- 已設定使用 Multi-Factor Authentication 的使用者，需要應用程式密碼才能登入一些非瀏覽器的應用程式。
- 使用者需要清除 (刪除) 登入資訊，重新啟動該應用程式，然後使用他們的使用者名稱和應用程式密碼來登入。

取得有關建立應用程式密碼的詳細資訊，及其他[應用程式密碼說明](multi-factor-authentication-end-user-app-passwords.md)。


>[AZURE.NOTE] 適用於 Office 2013 用戶端的新式驗證
>
> Office 2013 用戶端 (包括 Outlook) 支援新式驗證通訊協定。您可以將 Office 2013 設定成支援 Multi-Factor Authentication。在您設定好 Office 2013 之後，Office 2013 用戶端就不需要應用程式密碼了。如需詳細資訊，請參閱[發表 Office 2013 新式驗證公開預覽](https://blogs.office.com/2015/03/23/office-2013-modern-authentication-public-preview-announced/)。

**問：如果使用者未收到簡訊，或使用者回覆雙向簡訊但驗證逾時時，使用者該怎麼辦？**

Azure Multi-Factor Authentication 服務會透過簡訊彙總工具傳送簡訊。許多因素可能會影響簡訊傳遞和接收的可靠性，包括使用的彙總工具、目的地國家/地區、使用者的行動電話業者及訊號強度。基於此原因，不保證使用雙向簡訊時可正常傳遞簡訊及接收簡訊回覆。我們建議盡可能使用單向簡訊，而不要使用雙向簡訊。單向簡訊比較可靠，並可防止使用者因為回覆從其他國家/地區傳送的簡訊而產生全域簡訊費用。

在某些國家/地區 (例如美國和加拿大) 簡訊驗證更為可靠。我們鼓勵在使用 Azure Multi-Factor Authentication 時遭遇簡訊接收困難的使用者，改為選取行動應用程式或電話通話方法。行動應用程式是很棒的驗證方法，因為使用者透過行動數據和 Wi-Fi 連線，都能接收行動應用程式通知。此外，即使裝置收不到訊號，仍會顯示行動應用程式驗證碼。Microsoft 驗證器應用程式適用於 [Windows Phone](http://go.microsoft.com/fwlink/?Linkid=825071)、[Android](http://go.microsoft.com/fwlink/?Linkid=825072) 和 [IOS](http://go.microsoft.com/fwlink/?Linkid=825073)。

**問：我是否可以搭配 Azure Multi-Factor Authentication Server 使用硬體權杖？**

如果您正在使用 Azure Multi-Factor Authentication Server，您可以匯入協力廠商的 Open Authentication (OATH) 一次性限時密碼 (TOTP) 權杖，然後將它們用於 Multi-Factor Authentication。目前我們支援以 Gemalto 可為他們的權杖產生的較舊 Portable Symmetric Key Container (PSKC) 格式匯入協力廠商 OATH TOTP 權杖，且我們支援以 CSV 格式匯入權杖。如果您以 CSV 格式匯入權杖時，CSV 檔案必須包含一組序號、一組 Base32 格式的秘密金鑰及一段時間間隔 (通常是 30 秒)。

如果您可以將秘密金鑰檔案放在您可以匯入 Azure Multi-Factor Authentication Server 的 CSV 檔案中，您就可以使用屬於 OATH TOTP 權杖的 ActiveIdentity 權杖。您可以在下列情況下使用 OATH 權杖：搭配 Active Directory Federation Services (ADFS) 使用、搭配遠端驗證撥號使用者服務 (RADIUS) 使用、在用戶端系統能處理存取挑戰回應時使用，以及搭配 Internet Information Server (IIS) 表單型驗證使用。

**問：我是否可以使用 Azure Multi-Factor Authentication Server 保護「終端機服務」的安全？**

可以，但如果您使用的是 Windows Server 2012 R2 或更新版本，則只能使用遠端桌面閘道 (RD 閘道) 這麼做。

Windows Server 2012 R2 中的安全性變更已改變 Azure Multi-Factor Authentication Server 在 Windows Server 2012 及舊版本中，對本機安全性授權 (LSA) 安全性封裝的連接方式。對於 Windows Server 2012 或較舊版本中的終端機服務版本，您可以直接[使用 Windows 驗證來保護應用程式](multi-factor-authentication-get-started-server-windows.md#to-secure-an-application-with-windows-authentication-use-the-following-procedure)。如果您是使用 Windows Server 2012 R2，則需要 RD 閘道。

**問：為什麼設定來電顯示之後，使用者會收到來自匿名來電者的 Multi-Factor Authentication 來電？**

當透過公共電話網路發出 Multi-Factor Authentication 通話時，有時會透過不支援來電顯示的電信業者路由傳送。因此，即使 Multi-Factor Authentication 系統一律會傳送來電顯示號碼，但並不保證一定會顯示來電號碼。


## 錯誤數

**問：如果使用者使用行動應用程式通知驗證時，看到「驗證要求不適用於已啟用的帳戶」錯誤訊息的話，該怎麼辦？**

請遵循此程序：

1. 移至[您的 Azure 入口網站設定檔](https://account.activedirectory.windowsazure.com/profile/)，並使用您的組織帳戶登入。
2. 如果需要，請按一下 [其他驗證選項]，然後按一下其他選項以用於帳戶驗證。
3. 按一下 [其他安全性驗證]。
4. 將現有帳戶從行動應用程式移除。
5. 按一下 [設定]，然後依照指示來重新設定行動應用程式。




**問：如果使用者在嘗試使用非瀏覽器應用程式登入時，看到 0x800434D4L 錯誤訊息的話，使用者該怎麼辦？**

目前使用者只能在他可以透過其瀏覽器存取的應用程式和服務上，才能使用其他安全性驗證。本機電腦上安裝的非瀏覽器應用程式 (也稱為「豐富型用戶端應用程式」，例如 Windows PowerShell) 不適用於需要其他安全性驗證的帳戶。在此情況下，使用者可能會看到應用程式產生 0x800434D4L 錯誤。

此種情況的因應措施是使用不同的使用者帳戶，進行系統管理和非系統管理作業。您稍後可以連結系統管理帳戶與非系統管理帳戶之間的信箱，以便使用非系統管理帳戶登入 Outlook。如需詳細資訊，請了解如何[讓系統管理員能夠開啟及檢視使用者信箱的內容](http://help.outlook.com/141/gg709759.aspx?sl=1)。

<!---HONumber=AcomDC_0921_2016-->