<properties 
	pageTitle="RADIUS 驗證與 Azure Multi-Factor Authentication Server"
	description="此 Azure Multi-Factor Authentication 頁面協助您部署 RADIUS 驗證與 Azure Multi-Factor Authentication Server。"
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
	ms.topic="get-started-article"
	ms.date="08/15/2016"
	ms.author="kgremban"/>



# RADIUS 驗證與 Azure Multi-Factor Authentication Server

[RADIUS 驗證] 區段可讓您啟用及設定 Azure Multi-Factor Authentication Server 的 RADIUS 驗證。RADIUS 是接受驗證要求並處理這些要求的標準通訊協定。Azure Multi-Factor Authentication Server 做為 RADIUS 伺服器，插入在 RADIUS 用戶端 (例如 VPN 應用裝置) 和驗證目標 (可能是 Active Directory (AD)、LDAP 目錄或另一部 RADIUS 伺服器) 之間，以便新增 Azure Multi-Factor Authentication。為了讓 Azure Multi-Factor Authentication 運作，Azure Multi-Factor Authentication Server 必須設定為能夠與用戶端伺服器和驗證目標進行通訊。Azure Multi-Factor Authentication Server 接受來自 RADIUS 用戶端的要求、向驗證目標驗證認證、新增 Azure Multi-Factor Authentication，然後將回應傳回給 RADIUS 用戶端。只有當主要驗證和 Azure Multi-Factor Authentication 都成功時，整個驗證才會成功。

>[AZURE.NOTE]
MFA 伺服器在做為 RADIUS 伺服器時，僅支援 PAP (密碼驗證通訊協定) 和 MSCHAPv2 (Microsoft 的 Challenge-Handshake 驗證通訊協定) RADIUS 通訊協定。當 MFA Server 做為另一部 RADIUS 伺服器的 RADIUS Proxy，而此伺服器支援 Microsoft NPS 之類的通訊協定時，可以使用 EAP (可延伸的驗證通訊協定) 之類的其他通訊協定。</br> 在此組態中使用其他通訊協定時，單向 SMS 和 OATH 權杖沒有作用，因為 MFA Server 無法使用該通訊協定來起始成功的 RADIUS 挑戰回應。


![Radius 驗證](./media/multi-factor-authentication-get-started-server-rdg/radius.png)

## RADIUS 驗證組態

若要設定 RADIUS 驗證，請在 Windows 伺服器上安裝 Azure Multi-Factor Authentication Server。如果您有 Active Directory 環境，此伺服器應該加入網路內的網域。使用下列程序來設定 Azure Multi-Factor Authentication Server：

1. 在 Azure Multi-Factor Authentication Server 內，按一下左功能表中的 [RADIUS 驗證] 圖示。
2. 核取 [啟用 RADIUS 驗證] 核取方塊。
3. 如果 Azure Multi-Factor Authentication RADIUS 服務應該繫結到非標準連接埠，以接聽將設定的用戶端的 RADIUS 要求，請在 [用戶端] 索引標籤上變更 [驗證連接埠] 和 [帳戶處理連接埠]。
4. 按一下 [新增...] 按鈕。
5. 在 [新增 RADIUS 用戶端] 對話方塊中，輸入將向 Multi-Factor Authentication Server 驗證的應用裝置/伺服器的 IP 位址、[應用程式名稱] \(選擇性) 以及 [共用密碼]。共用密碼在 Azure Multi-Factor Authentication Server 和應用裝置/伺服器上必須相同。應用程式名稱會出現在 Azure Multi-Factor Authentication 報表中，而且可能顯示在簡訊或行動應用程式驗證訊息內。
6. 如果所有使用者都已經或將要匯入到「伺服器」，且必須接受多因素驗證，請核取 [需要進行 Multi-Factor Authentication 使用者比對] 方塊。如果有大量使用者尚未匯入伺服器及/或將免除多重要素驗證，請勿核取此方塊。如需此功能的其他資訊，請參閱說明檔。
7. 如果使用者將使用 Azure Multi-Factor Authentication 行動應用程式驗證，而且您想要使用 OATH 密碼做為頻外電話、簡訊或推播通知的遞補驗證，請核取 [啟用遞補 OATH 權杖] 方塊。
8. 按一下 [確定] 按鈕。
9. 您可以重複步驟 4 到 8，新增其他 RADIUS 用戶端。
10. 按一下 [目標] 索引標籤。
11. 如果 Azure Multi-Factor Authentication Server 安裝在 Active Directory 環境中已加入網域的伺服器上，請選取 [Windows 網域]。
12. 如果應該向 LDAP 目錄驗證使用者，請選取 [LDAP 繫結]。當使用 LDAP 繫結時，您必須按一下 [目錄整合] 圖示，並編輯 [設定] 索引標籤上的 LDAP 組態，讓「伺服器」可以繫結至您的目錄。在 LDAP Proxy 組態指南中可以找到設定 LDAP 的指示。
13. 如果應該向另一部 RADIUS 伺服器驗證使用者，選取 [RADIUS 伺服器]。
14. 按一下 [新增...] 按鈕，設定用來代理「伺服器」處理 RADIUS 要求的伺服器。
15. 在 [新增 RADIUS 伺服器] 對話方塊中，輸入 RADIUS 伺服器的 IP 位址和 [共用密碼]。共用密碼在 Azure Multi-Factor Authentication Server 和 RADIUS 伺服器上必須相同。如果 RADIUS 伺服器使用不同的通訊埠，請變更 [驗證連接埠] 和 [帳戶處理連接埠]。
16. 按一下 [確定] 按鈕。
17. 您必須在另一部 RADIUS 伺服器中新增 Azure Multi-Factor Authentication Server 做為 RADIUS 用戶端，它才會處理從 Azure Multi-Factor Authentication Server 送來的存取要求。您必須使用 Azure Multi-Factor Authentication Server 中設定的相同共用密碼。
18. 您可以重複此步驟來新增其他 RADIUS 伺服器，並使用 [上移] 和 [下移] 按鈕設定「伺服器」呼叫它們的順序。如此便完成 Azure Multi-Factor Authentication Server 組態。「伺服器」正在設定的連接埠上接聽來自設定的用戶端的 RADIUS 存取要求。


## RADIUS 用戶端組態

若要設定 RADIUS 用戶端，請遵循下列指導方針：

- 將您的應用裝置/伺服器設定為透過 RADIUS 向 Azure Multi-Factor Authentication Server (將做為 RADIUS 伺服器) 的 IP 位址驗證。
- 使用上述設定的相同共用密碼。
- 將 RADIUS 逾時設定為 30 至 60 秒，以保留足夠的時間來驗證使用者的認證、執行多因素驗證、接收回應，然後回應 RADIUS 存取要求。

<!---HONumber=AcomDC_0921_2016-->