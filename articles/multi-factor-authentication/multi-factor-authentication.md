<properties
	pageTitle="什麼是 Azure Multi-Factor Authentication？ | Microsoft Azure"
	description="本主題說明什麼是 Multi-Factor Authentication (mfa)、為什麼有人使用 MFA，還有 Multi-Factor Authentication 用戶端、不同的方法及可用版本的詳細資訊。Azure Multi-Factor Authentication 是除了使用使用者名稱與密碼之外，需要再利用其他方法驗證您的身份的驗證方法。它可以為使用者登入和交易提供額外層級的安全性。"
	keywords="MFA 的簡介, mfa 概觀, 什麼是 mfa"
	services="multi-factor-authentication"
	documentationCenter=""
	authors="kgremban"
	manager="femila"
	editor="curtland"/>

<tags
	ms.service="multi-factor-authentication"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/22/2016"
	ms.author="kgremban"/>

# 什麼是 Azure Multi-Factor Authentication？
多因素驗證 (MFA) 是需要使用多種驗證方法，並在使用者登入和交易中新增重要的第二層安全性的驗證方法。其運作方式需要下列其中任何二或多個驗證方法：

- 您知道的某些資訊 (通常是密碼)
- 您擁有的某些東西 (不容易輕易複製的信任裝置，例如電話)
- 您身上的某些特徵 (生物識別技術)

<center>![Username and Password](./media/multi-factor-authentication/pword.png) &#160;&#160;&#160;&#160;&#160;![Certificates](./media/multi-factor-authentication/phone.png) &#160;&#160;&#160;&#160;&#160;![Smart Phone](./media/multi-factor-authentication/hware.png) &#160;&#160;&#160;&#160;&#160;![Smart Card](./media/multi-factor-authentication/smart.png) &#160;&#160;&#160;&#160;&#160;![Virtual Smart Card](./media/multi-factor-authentication/vsmart.png) &#160;&#160;&#160;&#160;&#160;![Username and Password](./media/multi-factor-authentication/cert.png)</center>



Azure Multi-Factor Authentication 是除了使用使用者名稱與密碼之外，需要再利用其他方法驗證您的身份的驗證方法。它可以為使用者登入和交易提供第二層安全性。

Azure Multi-Factor Authentication 有助於保護對資料與應用程式的存取，同時可以滿足使用者對簡單登入程序的需求。它可以透過一些簡單的驗證選項 (例如電話、文字訊息，或行動應用程式通知或驗證代碼，以及第三方 OATH 權杖) 來提供強大的驗證功能。

如需 Azure Multi-Factor Authentication 運作方式的概觀，請參閱以下影片。


>[AZURE.VIDEO multi-factor-authentication-overview]

##為何使用 Azure Multi-Factor Authentication？

與以前比較起來，現在人們連線網路的時間越來越長。透過智慧型手機、平板電腦、膝上型電腦以及電腦，人們有幾種不同選擇可隨時用來連線網路並維持連線。人們可以從任何地方存取他們的帳戶與應用程式，這表示他們可以完成更多工作並為客戶提供更好的服務。

Azure Multi-Factor Authentication 是一個容易使用、可調整且可靠的解決方案，可提供第二種驗證方法讓您的使用者永遠受到保護。

![容易使用](./media/multi-factor-authentication/simple.png)| ![可調整](./media/multi-factor-authentication/scalable.png)| ![永遠受到保護](./media/multi-factor-authentication/protected.png)|![可靠](./media/multi-factor-authentication/reliable.png)
:-------------: | :-------------: | :-------------: | :-------------: |
**容易使用**|**可調整**|**永遠受到保護**|**可靠**

- **容易使用** - Azure Multi-Factor Authentication 很容易設定及使用。Azure Multi-Factor Authentication 提供的額外保護功能可讓使用者使用及管理他們自己的裝置，而且在許多情況下，只要簡單地按幾下就可以設定完成。
- **可調整** - Azure Multi-Factor Authentication 採用雲端技術且與您內部部署的 AD 與自訂應用程式整合。此保護功能甚至可以擴充以因應您高任務關鍵性的狀況。
- **永遠受到保護** - Azure Multi-Factor Authentication 使用最高工業標準提供強大驗證功能。
- **可靠** - 我們保證 Azure Multi-Factor Authentication 的可用性可達到 99.9%。當服務無法接收或處理多因素驗證的驗證要求時，服務會被視為無法使用。

如需為何使用 Azure Multi-Factor Authentication 的其他資訊，請參閱以下影片。

>[AZURE.VIDEO windows-azure-multi-factor-authentication]


## Azure Multi-Factor Authentication 的作用

多因素驗證的安全性仰賴其分層方法。使用多重驗證因素會為攻擊者帶來相當程度的挑戰。即使攻擊者試圖打探使用者的密碼，在不持有信任裝置的情況下便沒有任何意義。如果使用者遺失裝置，拾獲該裝置的人仍然無法使用此裝置，除非他或她剛好知道使用者的密碼。

![Proofup](./media/multi-factor-authentication-how-it-works/howitworks.png)



Azure Multi-Factor Authentication 有助於保護對資料與應用程式的存取，同時可以滿足使用者對簡單登入程序的需求。它藉由要求第二種形式的驗證提供額外的安全性，並透過一系列簡單的驗證選項提供增強式驗證：

- 撥打電話
- 簡訊
- 行動應用程式通知，讓使用者選擇偏好的方法
- 行動應用程式驗證碼
- 協力廠商 OATH 權杖

如需其作用的詳細資訊，請觀看以下影片。

>[AZURE.VIDEO multi-factor-authentication-deep-dive-securing-access-on-premises]

## 適用於 Multi-Factor Authentication 的方法
當使用者登入時，系統會將額外的驗證傳送給使用者。以下是適用於這個第二次驗證的方法清單。

驗證方法 | 說明
------------- | ------------- |
撥打電話 | 撥打使用者的智慧型手機，要求他們按 # 符號來驗證登入。如此將可完成驗證程序。這是可設定的選項，您可以變更為指定的代碼。
簡訊 | 傳送含 6 位數代碼的簡訊到使用者智慧型手機。輸入此代碼可完成驗證程序。
行動應用程式通知 | 將驗證要求傳送到使用者的智慧型手機，要求他們在行動應用程式中選取 [驗證] 來完成驗證。如果您將應用程式通知選為主要驗證方法，這種情況便會發生。如果它們在未登入時收到通知，他們可以選擇提報詐騙。</li><br><p> Microsoft 驗證器應用程式適用於 [Windows Phone](http://go.microsoft.com/fwlink/?Linkid=825071)、[Android](http://go.microsoft.com/fwlink/?Linkid=825072) 和 [IOS](http://go.microsoft.com/fwlink/?Linkid=825073)。
透過行動應用程式的驗證碼 | 將驗證碼傳送到在使用者智慧型手機中運作的行動應用程式。如果您選擇驗證碼做為主要驗證方法，便會發生這種情況。</li><br><p> Microsoft 驗證器應用程式適用於 [Windows Phone](http://go.microsoft.com/fwlink/?Linkid=825071)、[Android](http://go.microsoft.com/fwlink/?Linkid=825072) 和 [IOS](http://go.microsoft.com/fwlink/?Linkid=825073)。


## 可用的 Azure Multi-Factor Authentication 版本
Azure Multi-Factor Authentication 備有三個不同的版本。下表是每種版本的詳述。

版本 | 說明
------------- | ------------- |
Multi-Factor Authentication for Office 365 | 這個版本專門搭配 Office 365 應用程式運作，並且可從 Office 365 入口網站管理。系統管理員現在可以使用 Multi-Factor Authentication 來保護其 Office 365 資源的安全。此版本隨附於 Office 365 訂用帳戶。
適用於 Azure 系統管理員的 Multi-Factor Authentication | 適用於 Office 365 的 Multi-Factor Authentication 功能的相同子集將免費提供給所有 Azure 系統管理員。Azure 訂用帳戶的每個管理帳戶現可透過啟用此 Multi-Factor Authentication 核心功能來取得額外防護。因此想要存取 Azure 入口網站以建立 VM、網站、管理儲存體、行動服務或任何其他 Azure 服務的系統管理員，可在其系統管理員帳戶中新增多因素驗證。
Azure Multi-Factor Authentication | Azure Multi-Factor Authentication 提供最豐富的功能。它能透過 Azure 管理入口網站、進階報告及支援一系列內部部署和雲端應用程式來提供其他組態選項。Azure Multi-Factor Authentication 隨附於 Azure Active Directory Premium 和 Enterprise Mobility Suite 中。

## 版本的功能比較
下表提供 Azure Multi-Factor Authentication 各版本中可用的功能清單。


功能 | Multi-Factor Authentication for Office 365 (隨附於 Office 365 SKU 中)|適用於 Azure 系統管理員的 Multi-Factor Authentication (隨附於 Azure 訂用帳戶中) | Azure Multi-Factor Authentication (隨附於 Azure AD Premium 和 Enterprise Mobility Suite 中)
------------- | :-------------: |:-------------: |:-------------: |
系統管理員可以透過 MFA 保護帳戶| * | * (僅適用於 Azure 系統管理員帳戶)|*
以行動應用程式做為第二個因素|* | * | *
以撥打電話做為第二個因素|* | * | *
以 SMS 做為第二個因素|* | * | *
用戶端應用程式密碼不支援 MFA|* | * | *
系統管理員控制驗證方法| *|* | *
PIN 模式| | | *
詐騙警示| | | *
MFA 報告| | | *
一次性略過| | | *
通話的自訂問候語| | | *
自訂通話來電者 ID| | | *
事件確認| | | *
信任的 IP| | | *
記住受信任裝置的 MFA |* | * | *
MFA SDK | | | * 需要 Multi-Factor Auth Provider 和完整的 Azure 訂用帳戶
對使用 MFA Server 的內部部署應用程式進行 MFA| | | *



## 如何取得 Azure Multi-Factor Authentication

如果您想要 Azure Multi-Factor Authentication 所提供的完整功能，而不是只針對 Office 365 使用者和 Azure 系統管理員所提供的功能，有數個可取得 Azure Multi-Factor Authentication 的選項：

1.	購買 Azure Multi-Factor Authentication 授權，並將其指派給您的使用者。
2.	購買其中隨附 Azure Multi-Factor Authentication 的授權 (例如 Azure Active Directory Premium、Enterprise Mobility Suite 或 Enterprise Cloud Suite)，並將其指派給您的使用者。
3.	在 Azure 訂用帳戶中建立 Azure Multi-Factor Authentication 提供者。如果您還沒有 Azure 訂用帳戶，可以註冊一個 Azure 試用訂用帳戶。在試用到期之前，必須將試用訂用帳戶轉換為一般訂用帳戶。

使用 Azure Multi-Factor Authentication 提供者時，有兩種使用量模型可透過您的 Azure 訂用帳戶計費：


- **每位使用者**。適用於想要為一群定期需要驗證之固定數量員工啟用 Multi-Factor Authentication 的企業。
- **每次驗證**。適用於想要為大量不定期需要驗證之外部使用者啟用 Multi-Factor Authentication 的企業。

Azure Multi-Factor Authentication 為雲端與伺服器提供可選取的驗證方法。這表示您可以選擇可供使用者用於多重要素驗證的方法。在雲端版的多重要素驗證中，這項功能目前為公開預覽狀態。如需其他資訊，請參閱[可選取的驗證方法](multi-factor-authentication-whats-next.md#selectable-verification-methods)。

如需定價詳細資料，請參閱 [Azure MFA 定價](https://azure.microsoft.com/pricing/details/multi-factor-authentication/)。

選擇最適合您組織的每一基座或消費型模型。接下來，若要開始使用，請參閱[開始使用](multi-factor-authentication-get-started.md)。

## 選擇合適的多重因素安全性解決方案

因為 Azure Multi-Factor Authentication 的種類繁多，我們必須從幾個重點來判斷，以便找出最合適的版本。這些重點包括：

-	[我要保護什麼](#what-am-i-trying-to-secure)
-	[使用者位於何處](#where-are-the-users-located)

下列小節將提供如何判斷以上重點的指引。

### 我要保護什麼？

為了判斷出正確的 Multi-Factor Authentication 解決方案，首先我們必須回答一個問題：您試圖使用第二個驗證方法來保護什麼？它是 Azure 中的應用程式嗎？ 或者是遠端存取系統之類的系統。藉由判斷我們嘗試保護的東西，我們可以找出該在何處啟用 Multi-Factor Authentication 這個問題的答案。



您想要保護什麼| 雲端中的 Multi-Factor Authentication|Multi-Factor Authentication Server
------------- | :-------------: | :-------------: |
第一方 Microsoft 應用程式|* |* |
應用程式資源庫中的 Saas 應用程式|* |* |
透過 Azure AD App Proxy 發佈的 IIS 應用程式|* |* |
非透過 Azure AD App Proxy 發佈的 IIS 應用程式 | |* |
VPN、RDG 等遠端存取| |* |



### 使用者位於何處

接下來，不論是雲端中的 Multi-Factor Authentication 或使用 MFA Server 的內部部署 Multi-Factor Authentication，只要根據使用者的所在位置，我們就可以判斷出合適的解決方案。



使用者位置| 方案
------------- | :------------- |
Azure Active Directory| 雲端中的 Multi-Factor Authentication|
Azure AD 和使用 AD FS 同盟的內部部署 AD| 雲端中的 MFA 和 MFA Server 都是可用選項
Azure AD 和使用 DirSync、Azure AD Sync、Azure AD Connect 的內部部署 AD - 而沒有密碼同步|雲端中的 MFA 和 MFA Server 都是可用選項
Azure AD 和使用 DirSync、Azure AD Sync、Azure AD Connect 的內部部署 AD - 使用密碼同步|雲端中的 Multi-Factor Authentication
內部部署 Active Directory|Multi-Factor Authentication Server

下表是雲端中 Multi-Factor Authentication 和 Multi-factor Authentication Server 的功能比較。

 | 雲端中的 Multi-Factor Authentication | Multi-Factor Authentication Server
------------- | :-------------: | :-------------: |
以行動應用程式通知做為第二個因素 | ● | ● |
以行動應用程式驗證碼做為第二個因素 | ● | ●
以撥打電話做為第二個因素 | ● | ●
以單向 SMS 做為第二個因素 | ● | ●
以雙向 SMS 做為第二個因素 | | ●
以硬體權杖做為第二個因素 | | ●
用戶端應用程式密碼不支援 MFA | ● |  
系統管理員控制驗證方法 | ● | ●
PIN 模式 | | ●
詐騙警示 | ● | ●
MFA 報告 | ● | ●
一次性略過 | | ●
通話的自訂問候語 | ● | ●
可自訂的通話來電者 ID | ● | ●
信任的 IP | ● | ●
記住受信任裝置的 MFA| ● |  
條件式存取 | ● | ●
快取 | | ●

既然我們已判斷出要使用雲端 Multi-Factor Authentication 或內部部署 MFA Server，接下來可以開始設定及使用 Azure Multi-Factor Authentication。**選取代表您案例的圖示！**

<center> [![Cloud](./media/multi-factor-authentication-get-started/cloud2.png)](multi-factor-authentication-get-started-cloud.md) &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;[![Proofup](./media/multi-factor-authentication-get-started/server2.png)](multi-factor-authentication-get-started-server.md) &#160;&#160;&#160;&#160;&#160; </center>

<!---HONumber=AcomDC_0921_2016-->