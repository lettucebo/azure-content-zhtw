<properties
	pageTitle="在雲端中監視內部部署身分識別基礎結構。"
	description="這是 Azure AD Connect Health 頁面，其中說明它的功能，以及您可能會使用它的原因。"
	services="active-directory"
	documentationCenter=""
	authors="karavar"
	manager="femila"
	editor="karavar"/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="get-started-article"
	ms.date="07/14/2016"
	ms.author="vakarand"/>

# 在雲端中監視內部部署身分識別基礎結構和同步處理服務。

Azure AD Connect Health 可協助您監視和了解內部部署身分識別基礎結構和同步處理服務。它提供了重要身分識別元件 (例如 AD FS 伺服器、Azure AD Connect 伺服器 (也稱為同步處理引擎)、Active Directory 網域控制站等) 的監視功能，讓您維護可靠的 Office 365 或 Microsoft Online Services 連線。它也使這些元件的相關重要資料點變得容易存取，以便輕鬆地取得使用情況和其他重要情資。

這些資訊會呈現在 [Azure AD Connect Health 入口網站](https://aka.ms/aadconnecthealth)中。您可以使用 Azure AD Connect Health 入口網站檢視警示、效能監視和使用情況分析等等。Azure AD Connect Health 可讓您經由重要身分識別元件的單一健康狀態功能濾鏡，在一個位置執行所有作業。

![Azure AD Connect Health 是什麼](./media/active-directory-aadconnect-health/aadconnecthealth2.png)

Azure AD Connect Health 未來更新的內容包括額外監視，以及額外身分識別元件的深入資訊。因此，它透過身分識別透鏡提供單一儀表板，讓您能擁有更穩健、良好的整合式環境，使用者便能夠充分利用以增進工作能力。

<!-- <center>![What is Azure AD Connect Health](./media/active-directory-aadconnect-health/logo1.png)</center> -->

## 使用 Azure AD Connect Health 的原因

將內部部署目錄與 Azure AD 整合可提供通用身分識別供存取雲端和內部部署資源，進而讓您的使用者更具生產力。然而，整合後帶來的挑戰是，必須確保此環境良好健全，才能讓使用者從任何裝置可靠地存取內部部署和雲端中的資源。Azure AD Connect Health 提供簡易的雲端方法，監視及深入了解用來存取 Office 365 或其他 Azure AD 應用程式的內部部署身分識別基本結構。使用方式相當簡單，您只需將代理程式安裝在各個內部部署身分識別伺服器中即可。

## [適用於 AD FS 的 Azure AD Connect Health](active-directory-aadconnect-health-adfs.md)

適用於 AD FS 的 Azure AD Connect Health 支援 Windows Server 2008 R2 上的 AD FS 2.0，以及 Windows Server 2012 和 Windows Server 2012R2 中的 AD FS。其中也包括提供外部網路存取驗證支援的 AD FS Proxy 或 Web 應用程式 Proxy 伺服器。Health 代理程式的安裝非常容易且成本低廉，而適用於 AD FS 的 Azure AD Connect Health 會提供下列一組主要功能：

- 利用警示進行監視，以了解 AD FS 與 AD FS Proxy 伺服器何時健康狀態不良
- 重大警示的電子郵件通知
- 檢視效能資料的趨勢，適合用於 AD FS 的容量規劃
- 透過不同樞紐 (應用程式、使用者、網路位置等) 登入 AD FS 的使用情況分析，有助於了解如何利用 AD FS。
- AD FS 報表，例如嘗試使用錯誤使用者名稱/密碼的前 50 名使用者

下列影片將提供適用於 AD FS 的 Azure AD Connect Health 概觀

> [AZURE.VIDEO azure-ad-connect-health--monitor-you-identity-bridge]

## [適用於同步處理的 Azure AD Connect Health](active-directory-aadconnect-health-sync.md)

適用於同步處理的 Azure AD Connect Health 可監視及提供內部部署 Active Directory 與 Azure Active Directory 之間發生的同步處理相關資訊。適用於同步處理的 Azure AD Connect Health 提供下列一組主要功能：

- 利用警示進行監視，以了解 Azure AD Connect 伺服器 (也稱為同步處理引擎) 何時健康狀態不良
- 重大警示的電子郵件通知
- 同步處理操作情資，包括同步處理作業的延遲圖表以及同步處理作業 (如新增、更新、刪除) 的趨勢。
- 有關同步處理屬性 (上次成功匯出至 Azure AD) 的快速概覽資訊

下列影片將提供適用於同步處理的 Azure AD Connect Health 概觀

>[AZURE.VIDEO azure-active-directory-connect-health-monitoring-the-sync-engine]

## [適用於 AD DS (預覽) 的 Azure AD Connect Health](active-directory-aadconnect-health-adds.md)

適用於 AD DS 的 Azure AD Connect Health 可以監視安裝在 Windows Server 2008 R2、Windows Server 2012 及 Windows Server 2012 R2 上的網域控制站。安裝簡單、低成本的健康狀態代理程式可讓您直接從雲端監視內部部署 AD DS 環境。適用於 AD DS 的 Azure AD Connect Health 提供下列一組主要功能：

- 監視警示可在網域控制站健全狀況不良時進行偵測，並發送重要警示的電子郵件通知。
- 網域控制站儀表板可讓您快速檢視網域控制站的健全狀況和操作狀態。
- 複寫狀態儀表板擁有最新的複寫資訊，以及偵測到錯誤時可用的疑難排解指南連結。
- 可在任一處快速存取常用效能計數器的效能資料圖形，其為疑難排解和監視用途所必需。

下列影片將提供適用於 AD DS 的 Azure AD Connect Health 概觀

>[AZURE.VIDEO azure-ad-connect-health-monitors-on-premises-ad-domain-services]

## 開始使用 Azure AD Connect Health
很容易就可以開始使用 Azure AD Connect Health。請依照下列步驟執行：

1. [取得 Azure AD Premium](active-directory-get-started-premium.md) 或 [開始試用](https://azure.microsoft.com/trial/get-started-active-directory/)

2. 在身分識別伺服器上 [下載和安裝 Azure AD Connect Health Agent](#download-and-install-azure-ad-connect-health-agent)。

3. 在 [https://aka.ms/aadconnecthealth](https://aka.ms/aadconnecthealth) 檢視 Azure AD Connect Health 儀表板

>[AZURE.NOTE]請記住，在 Azure AD Connect Health 儀表板中出現任何資料之前，您必須先在目標伺服器上安裝 Azure AD Connect Health 代理程式。

## 下載和安裝 Azure AD Connect Health 代理程式

- 請參閱 Azure AD Connect Health 的 [需求](active-directory-aadconnect-health-agent-install.md#Requirements)

- 若要開始使用適用於 AD FS 的 Azure AD Connect Health，您可以在此下載最新版的代理程式：[下載適用於 AD FS 的 Azure AD Connect Health 代理程式。](http://go.microsoft.com/fwlink/?LinkID=518973) [](active-directory-aadconnect-health-agent-install.md#installing-the-azure-ad-connect-health-agent-for-ad-fs)

- 若要開始使用適用於同步處理的 Azure AD Connect Health，請下載並安裝 [最新版的 Azure AD Connect](http://go.microsoft.com/fwlink/?linkid=615771)。Health 代理程式將會隨著 Azure AD Connect 安裝 (1.0.9125.0 或更高版本) 一起安裝。Azure AD Connect 支援從舊版就地升級。

- 若要開始使用適用於 AD DS 的 Azure AD Connect Health，您可以在此下載最新版的代理程式：[下載適用於 AD DS 的 Azure AD Connect Health 代理程式。](http://go.microsoft.com/fwlink/?LinkID=820540) [](active-directory-aadconnect-health-agent-install.md#installing-the-azure-ad-connect-health-agent-for-ad-fs)

## Azure AD Connect Health 入口網站
Azure AD Connect Health 入口網站可讓您檢視警示、效能監視和使用情況分析。https://aka.ms/aadconnecthealth 會帶您前往 Azure AD Connect Health 的主要刀鋒視窗。您可以將刀鋒視窗視為視窗。在 [主要] 刀鋒視窗上，您會看到 [快速啟動]、Azure AD Connect Health 中的服務以及其他設定選項。以下螢幕擷取畫面是各自的簡短說明。部署代理程式之後，健康狀態服務會自動識別 Azure AD Connect Health 所監視的服務。

![Azure AD Connect Health 入口網站](./media/active-directory-aadconnect-health/portal4.png)

- **開始使用** – 選取此選項將會開啟 [快速啟動] 刀鋒視窗。在這裡，您將可以透過選擇 [取得工具、存取文件及提供意見反應] 來下載 Azure AD Connect Health 代理程式。

- **Active Directory Federation Services** – 這表示 Azure AD Connect Health 目前正在監視的所有 AD FS 服務。選取其中一個執行個體後，便會開啟一個包含該服務執行個體相關資訊的刀鋒視窗。這項資訊包括概觀、屬性、警示、監視和使用情況分析。在 [這裡](active-directory-aadconnect-health-adfs.md) 深入了解相關功能。

- **Azure Active Directory Connect (同步處理)** – 這代表 Azure AD Connect Health 目前所監視的 Azure AD Connect 伺服器。選取此項目後，便會開啟一個包含 Azure AD Connect 伺服器相關資訊的刀鋒視窗。在 [這裡](active-directory-aadconnect-health-sync.md) 深入了解相關功能。
 
- **Active Directory 網域服務** – 這表示 Azure AD Connect Health 目前正在監視的所有 AD DS 樹系。選取其中一個樹系後，便會開啟一個包含該樹系相關資訊的刀鋒視窗。資訊包含基本資訊概觀、網域控制站儀表板、複寫狀態儀表板、警示及監視。在 [這裡](active-directory-aadconnect-health-adds.md) 深入了解相關功能。

- **設定** – 這可讓您開啟或關閉下列選項：

	1. 自動更新可將 Azure AD Connect Health 代理程式自動更新為最新版本：這表示您將會在發佈最新版的 Azure AD Connect Health 代理程式時，自動更新為最新版本。此選項預設為啟用狀態。

	2. 僅允許 Microsoft 存取您的 Azure AD 目錄的健康狀況資料供疑難排解之用：這表示如果啟用此選項，Microsoft 將可以看到您所看到的相同資料。這可能有助於進行問題的疑難排解和協助。此選項預設為停用狀態。


## 相關連結

* [Azure AD Connect Health 代理程式安裝](active-directory-aadconnect-health-agent-install.md)
* [Azure AD Connect Health 操作](active-directory-aadconnect-health-operations.md)
* [使用 Azure AD Connect Health 來搭配 AD FS](active-directory-aadconnect-health-adfs.md)
* [使用適用於同步處理的 Azure AD Connect Health](active-directory-aadconnect-health-sync.md)
* [在 AD DS 使用 Azure AD Connect Health](active-directory-aadconnect-health-adds.md)
* [Azure AD Connect Health 常見問題集](active-directory-aadconnect-health-faq.md)
* [Azure AD Connect Health 版本歷程記錄](active-directory-aadconnect-health-version-history.md)

<!---HONumber=AcomDC_0928_2016-->