<properties 
	pageTitle="開始使用 Azure Multi-Factor Authentication Server"
	description="這是說明如何開始使用 Azure MFA Server 的 Azure Multi-Factor Authentication 頁面。"
	services="multi-factor-authentication"
	keywords="驗證伺服器, azure multi factor authentication 應用程式啟動頁面, 驗證伺服器下載"
	documentationCenter=""
	authors="kgremban"
	manager="femila"
	editor="curtand"/>

<tags
	ms.service="multi-factor-authentication"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="get-started-article"
	ms.date="08/15/2016"
	ms.author="kgremban"/>

# 開始使用 Azure Multi-Factor Authentication Server




<center>![Cloud](./media/multi-factor-authentication-get-started-server/server2.png)</center>

既然我們已經決定是否要使用內部部署 Multi-Factor Authentication，那麼我們就開始著手進行吧。本頁面探討新伺服器安裝，以及透過內部部署 Active Directory 予以設定。如果您已安裝 Phonefactor Server 並且正在尋求升級，請參閱[升級為 Azure Mult-Factor Authentication Server](multi-factor-authentication-get-started-server-upgrade.md)；如果您正在尋找只安裝 Web 服務的相關資訊，請參閱[部署 Azure Multi-Factor Authentication Server Mobile App Web 服務](multi-factor-authentication-get-started-server-webservice.md)。


## 下載 Azure Multi-Factor Authentication Server



下載 Azure Multi-Factor Authentication Server 的方法有兩種。兩者都是透過 Azure 入口網站來完成。第一個是藉由直接管理 Multi-Factor Auth Provider。第二個是透過服務設定。第二個選項需要 Multi-Factor Auth Provider 或 Azure MFA、Azure AD Premium 或 Enterprise Mobility Suite 授權。


### 從 Azure 入口網站下載 Azure Multi-Factor Authentication Server
--------------------------------------------------------------------------------

1. 以系統管理員身分登入 Azure 入口網站。
2. 在左側選取 [Active Directory]。
3. 在 [Active Directory] 頁面頂端，按一下 [**多因素驗證提供者**]
4. 在底部按一下 [**管理**]
5. 這會開啟新的頁面。按一下 [**下載**] ![下載](./media/multi-factor-authentication-sdk/download.png)
6. 在 [**產生啟用認證**] 上方，按一下 [**下載**] ![下載](./media/multi-factor-authentication-get-started-server/download4.png)
7. 儲存下載內容。



### 透過服務設定下載 Azure Multi-Factor Authentication Server


1. 以系統管理員身分登入 Azure 入口網站。
2. 在左側選取 [Active Directory]。
3. 按兩下您的 Azure AD 的執行個體。
4. 在頂端按一下 [**設定**] ![下載](./media/multi-factor-authentication-sdk/download2.png)
5. 在多因素驗證底下選取 [**管理服務設定**]
6. 在 [服務設定] 頁面上，於畫面底部按一下 [**移至入口網站**]。![下載](./media/multi-factor-authentication-get-started-server/servicesettings.png)
7. 這會開啟新的頁面。按一下 [**下載**]。
8. 在 [**產生啟用認證**] 上方，按一下 [**下載**]。
9. 儲存下載內容。




## 安裝及設定 Azure Multi-Factor Authentication Server
既然您已下載伺服器，現在可以進行安裝和設定。請確認要安裝的伺服器符合下列需求：



Azure Multi-Factor Authentication Server 需求|說明|
:------------- | :------------- |
硬體|<li>200 MB 的硬碟空間</li><li>x32 或 x64 處理器</li><li>1 GB 或更高的 RAM</li>
軟體|<li>Windows Server 2008 或更新版本 (如果主機是伺服器作業系統)</li><li>Windows 7 或更新版本 (如果主機是用戶端作業系統)</li><li>Microsoft.NET 4.0 Framework</li><li>IIS 7.0 或更新版本 (如果要安裝使用者入口網站或 Web 服務 SDK)</li>

### Azure Multi-Factor Authentication Server 防火牆需求
--------------------------------------------------------------------------------
每部 MFA Server 都必須要能夠透過連接埠 443 與以下位址進行輸出通訊：

- https://pfd.phonefactor.net
- https://pfd2.phonefactor.net
- https://css.phonefactor.net

如果輸出防火牆會限制連接埠 443，您必須開啟以下 IP 位址範圍：

IP 子網路|網路遮罩|IP 範圍
:------------- | :------------- | :------------- |
134\.170.116.0/25|255\.255.255.128|134\.170.116.1 – 134.170.116.126
134\.170.165.0/25|255\.255.255.128|134\.170.165.1 – 134.170.165.126
70\.37.154.128/25|255\.255.255.128|70\.37.154.129 – 70.37.154.254

如果您不使用 Azure Multi-Factor Authentication 事件確認功能，而且使用者不會在公司網路中利用裝置上的 Multi-Factor Auth 行動應用程式進行驗證，可以將 IP 範圍縮小為如下所示：


IP 子網路|網路遮罩|IP 範圍
:------------- | :------------- | :------------- |
134\.170.116.72/29|255\.255.255.248|134\.170.116.72 – 134.170.116.79
134\.170.165.72/29|255\.255.255.248|134\.170.165.72 – 134.170.165.79
70\.37.154.200/29|255\.255.255.248|70\.37.154.201 – 70.37.154.206


### 安裝及設定 Azure Multi-Factor Authentication Server
--------------------------------------------------------------------------------


1. 按兩下可執行檔。安裝作業隨即會開始。
2. 在 [選取安裝資料夾] 畫面中，請確認資料夾正確，然後按一下 [下一步]。
3. 當安裝完成時，請按一下 [完成]。這會啟動組態精靈。
4. 在組態精靈歡迎畫面上，勾選 [**略過使用驗證組態精靈**]，然後按 [**下一步**]。  這會關閉精靈並啟動伺服器。
![雲端](./media/multi-factor-authentication-get-started-server/skip2.png)

5. 回到下載伺服器的頁面，按一下 [**產生啟用認證**] 按鈕。將此資訊複製到 Azure MFA Server 提供的方塊中，然後按一下 [**啟用**]。


上述步驟說明使用組態精靈進行快速安裝。您可以從伺服器的 [工具] 功能表選取驗證精靈，以便重新執行。



##從 Active Directory 匯入使用者

既然您已安裝及設定伺服器，現在快速地將使用者匯入 Azure MFA Server。

### 從 Active Directory 匯入使用者
--------------------------------------------------------------------------------


1. 在 Azure MFA Server 的左側選取 [**使用者**]。
2. 在底部選取 [**從 Active Directory 匯入**]。
3. 現在您可以搜尋個別使用者，或在 AD 中搜尋含使用者的 OU。在此情況下，我們會指定使用者的 OU。
4. 反白顯示右側的所有使用者，然後按一下 [**匯入**]。您應該會看到指出成功完成作業的快顯視窗。關閉匯入視窗。

![雲端](./media/multi-factor-authentication-get-started-server/import2.png)

## 傳送電子郵件給使用者
既然已將您的使用者匯入到 Azure Multi-Factor Authentication 伺服器，建議您傳送電子郵件給使用者，告知他們已註冊多因素驗證。

Azure Multi-Factor Authentication Server 提供各種方式來設定您的使用者使用多因素驗證。比方說，如果您知道使用者的電話號碼，或是能夠將他們公司目錄中的電話號碼匯入到 Azure Multi-Factor Authentication Server，此電子郵件會讓使用者知道他們已設定為使用 Azure Multi-Factor Authentication、提供一些關於使用 Azure Multi-Factor Authentication Server 的指示，並告知將會撥打給使用者進行驗證的電話號碼。

電子郵件的內容隨著使用者已設定的驗證方法而不同 (例如電話、簡訊、行動應用程式)。例如，如果使用者驗證時需要使用 PIN，電子郵件將告訴他們已設定的初始 PIN。使用者通常必須在第一次驗證時變更他們的 PIN。

如果使用者的電話號碼尚未設定或匯入到 Azure Multi-Factor Authentication Server，或使用者已預先設定為使用行動應用程式進行驗證，您可以傳送電子郵件給他們，告知他們已設定為使用 Azure Multi-Factor Authentication，並將引導他們透過 Azure Multi-factor Authentication 使用者入口網站完成帳戶註冊。將會包含超連結，讓使用者按一下以存取使用者入口網站。當使用者按一下超連結時，網頁瀏覽器會開啟並進入公司的 Azure Multi-Factor Authentication 使用者入口網站。


### 設定電子郵件和電子郵件範本

您可以按一下左邊的電子郵件圖示，以設定用來傳送這些電子郵件的設定。這裡可讓您輸入郵件伺服器的 SMTP 資訊，還可讓您核取 [傳送郵件給使用者] 核取方塊來傳送電子郵件給全體使用者。

![電子郵件設定](./media/multi-factor-authentication-get-started-server/email1.png)

在 [電子郵件內容] 索引標籤中，您會看到所有可供選擇的各種電子郵件範本。根據您如何設定使用者來使用多因素驗證，您可以選擇最適合的範本。

![Email templates](./media/multi-factor-authentication-get-started-server/email2.png)

## Azure Multi-Factor Authentication Server 如何處理使用者資料？

當您使用 Multi-Factor Authentication (MFA) Server 內部部署時，使用者的資料會儲存在內部部署伺服器中。雲端中不會儲存任何持續性的使用者資料。當使用者執行雙因素驗證時，MFA Server 會將資料傳送至 Azure MFA 雲端服務，以執行驗證。當這些驗證要求傳送至雲端服務時，要求和記錄檔中會傳送下列欄位，供客戶的驗證/使用情況報告中使用。某些欄位是選擇性，可以在 Multi-Factor Authentication Server 內啟用或停用。從 MFA Server 到 MFA 雲端服務的通訊使用透過連接埠 443 輸出的 SSL/TLS。這些欄位包括：

- 唯一識別碼 - 使用者名稱或內部的 MFA 伺服器識別碼
- 名字和姓氏 - 選擇性
- 電子郵件地址 - 選擇性
- 電話號碼 - 進行語音通話或簡訊驗證時
- 安全性權杖 - 執行行動應用程式驗證時
- 驗證模式
- 驗證結果
- MFA Server 名稱
- MFA Server IP
- 用戶端 IP – 如果有的話



除了上述欄位，驗證結果 (成功/拒絕) 和任何拒絕的原因也與驗證資料一起儲存，可透過驗證/使用情況報告取得。


## 進階 Azure Multi-Factor Authentication Server 組態
如需進階設定的詳細資訊和組態資訊，請使用下表。

方法|說明
:------------- | :------------- |
[使用者入口網站](multi-factor-authentication-get-started-portal.md)| 有關設定使用者入口網站的資訊，包括部署和使用者自助。
[Active Directory Federation Service](multi-factor-authentication-get-started-adfs.md)|有關搭配 AD FS 設定 Azure Multi-Factor Authentication 的資訊。
[RADIUS 驗證](multi-factor-authentication-get-started-server-radius.md)| 有關搭配 RADIUS 設定 Azure MFA Server 的資訊。
[IIS 驗證](multi-factor-authentication-get-started-server-iis.md)|有關搭配 IIS 設定 Azure MFA Server 的資訊。
[Windows 驗證](multi-factor-authentication-get-started-server-windows.md)| 有關搭配 Windows 驗證設定 Azure MFA Server 的資訊。
[LDAP 驗證](multi-factor-authentication-get-started-server-ldap.md)|有關搭配 LDAP 驗證設定 Azure MFA Server 的資訊。
[使用 RADIUS 的遠端桌面閘道器和 Azure Multi-Factor Authentication Server](multi-factor-authentication-get-started-server-rdg.md)| 有關使用 RADIUS 搭配遠端桌面閘道器設定 Azure MFA Server 的資訊。
[與 Windows Server Active Directory 同步處理](multi-factor-authentication-get-started-server-dirint.md)|有關設定 Active Directory 與 Azure MFA Server 之間同步處理的資訊。
[部署 Azure Multi-Factor Authentication Server 行動應用程式 Web 服務](multi-factor-authentication-get-started-server-webservice.md)|有關設定 Azure MFA 伺服器 Web 服務的資訊。

<!---HONumber=AcomDC_0928_2016-->