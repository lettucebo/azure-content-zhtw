<properties 
	pageTitle="使用 Azure MFA 的安全性最佳做法"
	description="本文件提供搭配 Azure 帳戶使用 Azure MFA 的最佳做法"
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
	ms.date="08/04/2016"
	ms.author="kgremban"/>

# 搭配 Azure AD 帳戶使用 Azure Multi-Factor Authentication 的安全性最佳做法

提到加強驗證程序時，Multi-Factor Authentication 是大多數組織的首選。Azure Multi-Factor Authentication (MFA) 可讓公司符合其安全性和法規遵循需求，同時為使用者提供簡單的登入體驗。本文將介紹一些您在規劃採用 Azure MFA 時應該考慮的最佳做法。

## 雲端中的 Azure Multi-Factor Authentication 的最佳做法
為了提供所有使用者 Multi-Factor Authentication 並且利用 Azure Multi-factor Authentication 提供的擴充功能，您必須為所有使用者啟用 Azure Multi-Factor Authentication。可以藉由使用下列其中一個項目來完成：

- Azure AD Premium 或 Enterprise Mobility Suite
- Multi-Factor Auth Provider

### Azure AD Premium/Enterprise Mobility Suite

![EMS](./media/multi-factor-authentication-security-best-practices/ems.png)

使用 Azure AD Premium 或 Enterprise Mobility Suite 在雲端中採用 Azure MFA 的第一個建議步驟，是指派授權給您的使用者。Azure Multi-Factor Authentication 是這些套件的一部分，因此您的組織不需要任何其他項目即可將 Multi-Factor Authentication 功能延伸到所有使用者。

設定 Multi-Factor Authentication 時，請考慮下列項目：

- 您不需要建立 Multi-Factor Auth Provider。Azure AD Premium 和 Enterprise Mobility Suite 隨附於 Azure Multi-Factor Authentication。如果您建立 Auth Provider，可能會向您雙重計費。
- 如果您要讓內部部署 Active Directory 環境與 Azure AD 目錄同步處理，Azure AD Connect 只是一項需求。如果您只要使用不與內部部署 Active Directory 執行個體同步處理的 Azure AD 目錄，就不需要 Azure AD Connect。


### Multi-Factor Auth Provider

![Multi-Factor Auth Provider](./media/multi-factor-authentication-security-best-practices/authprovider.png)

如果您沒有 Azure AD Premium 或 Enterprise Mobility Suite，則在雲端中採用 Azure MFA 的第一個建議步驟是建立 MFA Auth Provider。雖然 MFA 預設可供擁有 Azure Active Directory 的全域系統管理員使用，當您為您的組織部署 MFA 時，您需要將 Multi-Factor Authentication 功能延伸到所有使用者，若要完成這項操作，您需要 Multi-Factor Auth Provider。選取 Auth Provider 時，您必須選取目錄並考慮下列項目：

- 您不需要 Azure AD 目錄即可建立 Multi-Factor Auth Provider。
- 如果您想要將 Multi-Factor Authentication 延伸到所有使用者，以及/或想要讓全域管理員充分利用管理入口網站、自訂問候語及報告等功能，您需要建立 Multi-Factor Auth Provider 與 Azure AD 目錄之間的關聯。
- 如果您要讓內部部署 Active Directory 環境與 Azure AD 目錄同步處理，DirSync 或 AAD Sync 只是一項需求。如果您只要使用不與內部部署 Active Directory 執行個體同步處理的 Azure AD 目錄，就不需要 DirSync 或 AAD Sync。
- 如果您擁有 Azure AD Premium 或 Enterprise Mobility Suite，便不需要建立 Multi-Factor Auth Provider。您只需要將授權指派給使用者，接著就可以開始為使用者開啟 MFA。
- 請務必選擇適合您企業 (每個驗證或每個已啟用使用者) 的正確使用量模型，一旦您選取使用量模型之後就無法變更它。

### 使用者帳戶
為使用者啟用 MFA 時，使用者帳戶可以是三個核心狀態的其中一個：已停用、已啟用或強制。使用以下的指導方針來確保您為您的部署使用最適當的選項：

- 當使用者的狀態設定為已停用時，該使用者不使用 Multi-Factor Authentication。這是預設狀態。
- 當使用者的狀態設定為已啟用時，表示使用者已啟用但是尚未完成註冊程序。系統會提示他們在下次登入時完成程序。此設定不會影響非瀏覽器應用程式。所有應用程式都會繼續運作，直到註冊程序完成為止。
- 當使用者的狀態設定為強制時，表示使用者可能已完成註冊。如果他們已完成註冊程序，表示他們正在使用 Multi-Factor Authentication。否則，系統會提示使用者在下次登入時完成註冊程序此設定會影響非瀏覽器應用程式。除非建立及使用應用程式密碼，否則這些應用程式將無法運作。
- 使用可於[開始在雲端中使用 Azure Multi-Factor Authentication](multi-factor-authentication-get-started-cloud.md) 取得的使用者通知範本，傳送有關 MFA 採用的電子郵件給您的使用者。

### 支援能力

因為大多數使用者已經習慣僅使用密碼來驗證，貴公司務必讓所有使用者了解此程序。了解程度可以降低使用者針對與 MFA 相關的一些小問題尋求技術支援人員協助的可能性。不過，有一些案例是需要暫時停用 MFA。使用以下的指導方針了解如何處理這些案例：

- 請確定您的技術支援人員都受過訓練，可以處理行動應用程式或電話未收到通知或來電，以及因為上述原因讓使用者無法登入的案例。他們可以啟用一次性略過選項，讓使用者「略過」Multi-Factor Authentication 來通過驗證一次。略過的效力是暫時的，因此會在指定的秒數過後到期。
- 如有必要，您可以利用 Azure MFA 中的信任的 IP 功能。此功能可賦予受管理或同盟租用戶管理員，讓從公司近端內部網路登入之使用者略過 Multi-Factor Authentication 的能力。這些功能適用於擁有 Azure AD Premium、Enterprise Mobility Suite 或 Azure Multi-Factor Authentication 授權的 Azure AD 租用戶。


## 內部部署 Azure Multi-Factor Authentication 的最佳做法
如果您的公司決定運用自己的基礎結構來啟用 MFA，必須內部部署 Azure Multi-factor Authentication Server。下圖顯示 MFA Server 元件：

![Multi-Factor Auth Provider](./media/multi-factor-authentication-security-best-practices/server.png) 「預設未安裝」* 安裝但預設未啟用


Azure Multi-Factor Authentication Server 可以用於保護由 Azure AD 帳戶存取的雲端資源和內部部署資源的安全。不過，這僅能透過使用同盟來完成。也就是您必須擁有 AD FS 並讓它與您的 Azure AD 租用戶同盟。設定 Multi-Factor Authentication Server 時，請考慮下列項目：

- 如果您使用 Active Directory Federation Services 保護 Azure AD 資源的安全，則驗證的第一個因素是使用 AD FS 在內部部署。第二個因素是遵守宣告在內部部署。
- 不一定要在 AD FS 同盟伺服器上安裝 Azure Multi-Factor Authentication Server，但必須在執行 AD FS 的 Windows Server 2012 R2 上安裝適用於 AD FS 的 Multi-Factor Authentication Adapter。您可以在不同的電腦上安裝伺服器 (只要是支援的版本即可) 以及在 AD FS 同盟伺服器上個別安裝 AD FS 配接器。如需個別安裝配接器的相關指示，請參閱下列程序。
- Multi-Factor Authentication AD FS Adapter 安裝精靈會在 Active Directory 中建立名為 PhoneFactor Admins 的安全性群組，然後將 Federation Service 的 AD FS 服務帳戶加入至這個群組。建議您在網域控制站上確認確實已建立 PhoneFactor Admins 群組，而且 AD FS 服務帳戶是此群組的成員。如有必要，請以手動方式將 AD FS 服務帳戶加入至網域控制站上的 PhoneFactor Admins 群組。

### 使用者入口網站
此入口網站會在 Internet Information Server (IIS) 網站執行，允許自助式功能以及提供一組完整的使用者管理功能。遵循下列指導方針以設定此元件：

- 需要 IIS 6 或以上版本
- 必須安裝並註冊 ASP.NET v2.0.507207
- 此伺服器可以部署在周邊網路中。



### 應用程式密碼
如果您的組織使用 SSO 與 Azure AD 同盟，而且您想要使用 Azure MFA，當使用應用程式密碼時請注意下列項目，請記住，這僅適用於使用同盟 (SSO) 時：

- 應用程式密碼由 Azure AD 驗證，因此會略過同盟。唯有在設定應用程式密碼時才會使用同盟。
- 對於同盟 (SSO) 使用者，密碼會儲存在組織識別碼中。如果使用者離開公司，這些資訊必須使用 DirSync 即時流向組織識別碼。帳戶停用/刪除最長可能需要 3 個小時才能完成同步處理，導致 Azure AD 中的應用程式密碼停用/刪除延遲。
- 應用程式密碼不會遵守內部部署用戶端存取控制設定
- 應用程式密碼不適用內部部署驗證記錄 / 稽核功能
- Microsoft Lync 2013 用戶端需要更多使用者教育。
- 某些進階架構設計在使用 Multi-Factor Authentication 時，可能需要搭配使用組織使用者名稱和密碼及應用程式密碼，須視驗證的位置而定。對於根據內部部署基礎結構進行驗證的用戶端，您可以使用組織使用者名稱和密碼。對於根據 Azure AD 進行驗證的用戶端，您需要使用應用程式密碼。
- 依預設，使用者無法建立應用程式密碼，如果貴公司需要，或如果您要允許使用者在某些情況下建立應用程式密碼，請確定已選取允許使用者建立應用程式密碼以登入非瀏覽器應用程式的選項。

## 其他考量
使用以下清單以了解內部部署的每個元件的一些考量和最佳做法：

方法|說明
:------------- | :------------- |
[Active Directory Federation Service](multi-factor-authentication-get-started-adfs.md)|有關搭配 AD FS 設定 Azure Multi-Factor Authentication 的資訊。
[RRADIUS 驗證](multi-factor-authentication-get-started-server-radius.md)| 有關搭配 RADIUS 設定 Azure MFA Server 的資訊。
[IIS 驗證](multi-factor-authentication-get-started-server-iis.md)|有關搭配 IIS 設定 Azure MFA Server 的資訊。
[Windows 驗證](multi-factor-authentication-get-started-server-windows.md)| 有關搭配 Windows 驗證設定 Azure MFA Server 的資訊。
[LDAP 驗證](multi-factor-authentication-get-started-server-ldap.md)|有關搭配 LDAP 驗證設定 Azure MFA Server 的資訊。
[使用 RADIUS 的遠端桌面閘道器和 Azure Multi-Factor Authentication Server](multi-factor-authentication-get-started-server-rdg.md)| 有關使用 RADIUS 搭配遠端桌面閘道器設定 Azure MFA Server 的資訊。
[與 Windows Server Active Directory 同步處理](multi-factor-authentication-get-started-server-dirint.md)|有關設定 Active Directory 與 Azure MFA Server 之間同步處理的資訊。
[部署 Azure Multi-Factor Authentication Server 行動應用程式 Web 服務](multi-factor-authentication-get-started-server-webservice.md)|有關設定 Azure MFA 伺服器 Web 服務的資訊。
[使用 Azure Multi-Factor Authentication 的進階 VPN 組態](multi-factor-authentication-advanced-vpn-configurations.md)|使用 LDAP 或 RADIUS 設定 Cisco ASA、Citrix Netscaler 和 Juniper/Pulse Secure VPN 應用裝置的相關資訊。


## 其他資源
雖然這篇文章強調 Azure MFA 的一些最佳做法，還有其他資源您也可以在規劃 MFA 部署時使用。以下清單有可以在此程序期間協助您的一些關鍵文章：

- [Azure Multi-Factor Authentication 中的報告](multi-factor-authentication-manage-reports.md)
- [Azure Multi-factor Authentication 的設定體驗](multi-factor-authentication-end-user-first-time.md)
- [Azure Multi-Factor Authentication 常見問題集](multi-factor-authentication-faq.md)

<!---HONumber=AcomDC_0921_2016-->