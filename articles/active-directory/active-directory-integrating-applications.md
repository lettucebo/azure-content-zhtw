<properties
   pageTitle="整合應用程式與 Azure Active Directory | Microsoft Azure"
   description="如何在 Azure Active Directory (Azure AD) 中新增、更新或移除應用程式的詳細資料。"
   services="active-directory"
   documentationCenter=""
   authors="msmbaldwin"
   manager="mbaldwin"
   editor="mbaldwin" />
<tags
   ms.service="active-directory"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="identity"
   ms.date="07/25/2016"
   ms.author="mbaldwin;bryanla" />

# 整合應用程式與 Azure Active Directory

[AZURE.INCLUDE [active-directory-devguide](../../includes/active-directory-devguide.md)]

企業開發人員和軟體即服務 (SaaS) 提供者可以開發可與 Azure Active Directory (Azure AD) 整合的商業雲端服務或企業營運應用程式，以提供安全的登入和授權給其服務。若要整合應用程式或服務與 Azure AD，開發人員必須先透過 Azure 傳統入口網站，向 Azure AD 註冊他們的應用程式相關詳細資料。

本文將說明如何在 Azure AD 中新增、更新或移除應用程式。您將了解可與 Azure AD 整合的不同類型應用程式，以及如何設定您的應用程式存取其他資源，例如 Web API 等等。

若要深入了解代表已註冊應用程式的兩個 Azure AD 物件與其間的關係，請參閱[應用程式物件和服務主體物件](active-directory-application-objects.md)；若要深入了解利用 Azure Active Directory 開發應用程式時應使用的商標指導方針，請參閱[整合應用程式的商標指導方針](active-directory-branding-guidelines.md)。

## 新增應用程式

任何想要使用 Azure AD 功能的應用程式都必須先在 Azure AD 租用戶中註冊。此登錄程序牽涉到提供 Azure AD 應用程式的相關詳細資料，例如其所在的 URL、要在使用者驗證之後傳送回應的 URL，以及會識別應用程式的 URI 等。

如果您正在建置的 Web 應用程式只需要支援在 Azure AD 中登入使用者，您只需依照下列指示進行。若您的應用程式需要存取 Web API，或您想讓應用程式有多個租用戶以允許其他 Azure AD 租用戶的使用者進行存取，您必須繼續閱讀[更新應用程式](#updating-an-application)一節，以繼續設定您的應用程式。

### 在 Azure 傳統入口網站註冊新的應用程式

1. 登入 [Azure 傳統入口網站](https://manage.windowsazure.com)。

1. 按一下左側功能表中的 Active Directory 圖示，並按一下想要的目錄。

1. 在頂端功能表上，按一下 [**應用程式**]。如果您的目錄中尚未新增任何應用程式，此頁面僅會顯示新增應用程式連結。按一下該連結，或者您可以按一下命令列上的 [新增] 按鈕。

1. 在 [欲執行動作] 頁面上，按一下此連結以[加入我的組織正在開發的應用程式]。

1. 在 [告知我們您的應用程式] 頁面上，您必須指定應用程式的名稱並指出您向 Azure AD 註冊的應用程式類型。您可以從 Web 應用程式和/或 Web API (預設值，在 OAuth2 也稱為機密用戶端) 或原生用戶端應用程式 (代表安裝於電話或電腦等裝置上的應用程式，在 OAuth2 也稱為公用用戶端) 中選擇。完成之後，按一下頁面右下角的箭頭圖示。

1. 在 [應用程式屬性] 頁面上，如果您註冊的是 Web 應用程式或僅只原生用戶端應用程式的重新導向 URI，請提供登入 URL 和應用程式識別碼 URI，然後按一下頁面右下角的核取方塊。

1. 已新增您的應用程式，而且您將會進入應用程式的 [快速啟動] 頁面。根據您的應用程式為 Web 或原生應用程式，您會看到如何將其他功能加入至應用程式中的不同選項。一旦新增您的應用程式之後，就可以開始更新您的應用程式，讓使用者登入、在其他應用程式中存取 Web API 或設定多租用戶應用程式 (可讓其他組織存取您的應用程式)。

>[AZURE.NOTE] 根據預設，新建立的應用程式註冊會設定為允許來自您目錄的使用者登入您的應用程式。

## 更新應用程式

一旦已向 Azure AD 註冊您的應用程式，可能需要更新才能存取 Web API，或供其他組織使用等等。本節說明您可能需要用來進一步設定其他應用程式的各種方式。一開始我們將先概述同意架構，如果您正在建置資源/API 應用程式，而其將由貴組織或其他組織中的開發人員所建置的用戶端應用程式來使用，那麼請務必了解此架構。

如需如何在 Azure AD 中進行驗證的詳細資訊，請參閱 [Azure AD 的驗證案例](active-directory-authentication-scenarios.md)。

### 同意架構的概觀

Azure AD 的同意架構可讓您輕鬆地開發需要存取由 Azure AD 租用戶保護的多租用戶 Web 和原生用戶端應用程式 (不同於註冊用戶端應用程式的架構)。除了您自己的 Web API 之外，這些 Web API 包括圖形 API、Office 365 和其他 Microsoft 服務。此架構以使用者或系統管理員為根據，他們可同意讓應用程式在他們的目錄中註冊，並可能包括存取目錄資料。

例如，如果 Web 用戶端應用程式需要呼叫 Office 365 Web API/資源應用程式來讀取有關使用者的行事曆資訊，該使用者必須同意此用戶端應用程式。取得同意之後，用戶端應用程式可以代表使用者呼叫 Office 365 Web API，並視需要使用行事曆資訊。

同意架構建置在 OAuth 2.0 和各種不同流程上，例如授權碼授與和用戶端認證授與，使用公用或機密的用戶端。透過 OAuth 2.0，Azure AD 可以建置許多不同類型的用戶端應用程式，例如手機、平板電腦、伺服器或 Web 應用程式，並且存取所需的資源。

如需同意架構的詳細資訊，請參閱 [Azure AD 中的 OAuth 2.0](https://msdn.microsoft.com/library/azure/dn645545.aspx)、[Azure AD 的驗證案例](active-directory-authentication-scenarios.md)和 Office 365 主題 ([了解使用 Office 365 API 進行驗證](https://msdn.microsoft.com/office/office365/howto/common-app-authentication-tasks))。

#### 同意體驗的範例

下列步驟將示範如何將同意體驗用於應用程式開發人員和使用者。

1. 在 Azure 傳統入口網站的 Web 用戶端應用程式的組態頁面上，使用權限中的下拉式功能表，將您的應用程式所需的權限設定為其他應用程式的控制項。

    ![其他應用程式的權限](./media/active-directory-integrating-applications/permissions.png)

1. 請考量您的應用程式權限是否已更新、是否正在執行應用程式，以及使用者是否即將第一次使用它。如果應用程式尚未取得存取或重新整理權杖，應用程式必須移至 Azure AD 的授權端點以取得授權碼，用來取得新的存取和重新整理權杖。

1. 如果使用者尚未通過驗證，系統會要求他們登入 Azure AD。

    ![使用者或系統管理員登入 Azure AD](./media/active-directory-integrating-applications/useradminsignin.png)

1. 使用者登入之後，Azure AD 會判斷是否需要向使用者顯示同意頁面。此判斷根據使用者 (或其組織的系統管理員) 是否已經同意應用程式。如果尚未授與同意，Azure AD 會提示使用者取得同意，並顯示其運作所需的必要權限。同意對話方塊中顯示的權限集，和在 Azure 傳統入口網站的權限中選取做為其他應用程式控制項的內容相同。

    ![使用者同意體驗](./media/active-directory-integrating-applications/userconsent.png)

1. 使用者同意後，授權碼會傳回您的應用程式，藉以兌換取得存取權杖和重新整理權杖。如需此流程的詳細資訊，請參閱 [Azure AD 的驗證案例](active-directory-authentication-scenarios.md)中的 [Web 應用程式到 Web API 區段](active-directory-authentication-scenarios.md#web-application-to-web-api)一節。

### 設定用戶端應用程式以存取 Web API

當用戶端應用程式設定為存取資源應用程式所公開的 Web API (即 Azure AD 圖形 API)，以上所述的同意架構將確保用戶端取得必要的權限授與 (根據所要求的權限)。根據預設，所有應用程式都可以利用已根據預設選取的 Azure AD 「啟用登入並讀取使用者的設定檔」權限，從 Azure Active Directory (圖形 API) 和 Azure 服務管理 API 選擇權限。如果您的用戶端應用程式已在 Office 365 Azure AD 租用戶中註冊，則也可選取 SharePoint 與 Exchange Online 的 Web API 和權限。您可以從所需 Web API 旁之下拉式功能表中的兩種權限類型選取：

- 應用程式權限：您的用戶端應用程式本身需要直接存取 Web API (沒有使用者內容)。這種類型的權限需要系統管理員的同意，而且無法供原生用戶端應用程式使用。

- 委派權限：您的用戶端應用程式需要存取 Web API 做為已登入的使用者，但其存取權受到選取權限的限制。這種類型的權限可由使用者授與，除非權限設定為需要系統管理員的同意。

#### 新增 Web API 的存取

1. 登入 [Azure 傳統入口網站](https://manage.windowsazure.com)。

1. 按一下左側功能表中的 Active Directory 圖示，然後按一下想要的目錄。

1. 在頂端功能表上，按一下 [應用程式]，然後按一下您想要設定的應用程式。單一登入及其他組態資訊都會顯示 [快速啟動] 頁面。

1. 在 [快速啟動] 的 [其他應用程式] 區段中展開存取 Web API，然後按一下 [選取權限] 區段下的 [立即設定]連結。應用程式屬性頁面會隨即出現。

1. 向下捲動至 [其他應用程式] 區段的 [權限]。第一個資料行可讓您在公開 Web API 的目錄中從可用的資源應用程式中選取。一旦選取之後，您就可以選取 Web API 公開的應用程式和委派權限。

1. 選取之後，按一下命令列上的 [儲存] 按鈕。

>[AZURE.NOTE] 按一下 [儲存] 按鈕，也可以根據您設定的其他應用程式權限，在您的目錄中自動設定您的影用程式權限。您可以查看 [應用程式屬性] 索引標籤來檢視這些應用程式權限。

### 設定資源應用程式以公開 Web API

您可以開發 Web API，並藉由公開存取範圍使其可供用戶端應用程式使用。正確設定的 Web API 即可供使用，就像其他 Microsoft Web API 一樣，包括圖形 API 和 Office 365 API。存取範圍會透過您應用程式的資訊清單 (此為代表應用程式身分識別組態的 JSON 檔案) 公開。您可以瀏覽到 Azure 傳統入口網站中的應用程式，並按一下命令列上的 [應用程式資訊清單] 按鈕，即可公開存取範圍。

#### 新增資源應用程式的存取範圍

1. 登入 [Azure 傳統入口網站](https://manage.windowsazure.com)。

1. 按一下左側功能表中的 Active Directory 圖示，然後按一下想要的目錄。

1. 在頂端功能表上，按一下 [應用程式]，然後按一下您想要設定的資源應用程式。單一登入及其他組態資訊都會顯示 [快速啟動] 頁面。

1. 按一下命令列中的 [管理資訊清單] 按鈕，並選取 [下載資訊清單]。

1. 開啟 JSON 應用程式資訊清單檔並以下列 JSON 程式碼片段取代 "oauth2Permissions" 節點。此程式碼片段是如何公開範圍 (稱為「使用者模擬」) 的範例，讓資源擁有者可以為用戶端應用程式提供資源的委派存取類型。確定您變更了自己的應用程式文字及值：

		"oauth2Permissions": [
		{
			"adminConsentDescription": "Allow the application full access to the Todo List service on behalf of the signed-in 	user",
			"adminConsentDisplayName": "Have full access to the Todo List service",
			"id": "b69ee3c9-c40d-4f2a-ac80-961cd1534e40",
			"isEnabled": true,
			"type": "User",
			"userConsentDescription": "Allow the application full access to the todo service on your behalf",
			"userConsentDisplayName": "Have full access to the todo service",
			"value": "user_impersonation"
			}
		],

    識別碼值必須是您使用 [GUID 產生工具](https://msdn.microsoft.com/library/ms241442%28v=vs.80%29.aspx)或以程式設計方式建立的新產生 GUID。它代表由 Web API 所公開之權限的唯一識別碼。一旦您的用戶端已正確設定為要求存取您的 Web API 並且呼叫 Web API，它會顯示 OAuth 2.0 JWT 權杖，其範圍 (scp) 宣告已設為上述值，在此案例中為 user\_impersonation。

	>[AZURE.NOTE] 稍後您可以視需要公開其他範圍。請考慮您的 Web API 可能會公開多個與各種不同功能相關聯的範圍。現在您可以使用在 OAuth 2.0 JWT 權杖中收到的範圍 (scp) 宣告，控制 Web API 的存取權。

1. 儲存更新的 JSON 檔案並將其上傳，方法是按一下命令列中的 [管理資訊清單] 按鈕、選取 [上傳資訊清單]、瀏覽至您的已更新資訊清單檔案並將其選取。上傳之後，您的 Web API 現在已設定為可供目錄中的其他應用程式使用。

#### 確認已向目錄中的其他應用程式公開 Web API

1. 在頂端功能表上，按一下 [應用程式]，選取您想要設定 Web API 存取權所需的用戶端應用程式，然後按一下 [設定]。

1. 向下捲動至 [其他應用程式] 區段的 [權限]。按一下 [選取應用程式] 下拉式功能表，您就可以選取只想對其公開權限的 Web API。從 [委派的權限] 下拉式功能表，選取新的權限。

![顯示待辦事項清單權限](./media/active-directory-integrating-applications/listpermissions.png)

#### 應用程式資訊清單的詳細資料
應用程式資訊清單實際上可做為更新應用程式實體的一種機制，其可定義 Azure AD 應用程式身分識別組態的所有屬性，包括我們所討論的 API 存取範圍。如需應用程式實體的詳細資訊，請參閱[圖形 API 應用程式實體文件](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#application-entity)。您會在其中找到用來指定您 API 權限的應用程式實體成員之完整參考資訊：

- appRoles 成員，其為 [AppRole](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#approle-type) 實體集合，可用來定義 Web API 的 [應用程式權限]
- oauth2Permissions 成員，其為 [OAuth2Permission](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#oauth2permission-type) 實體集合，可用來定義 Web API 的 [委派的權限]

如需應用程式資訊清單一般概念的詳細資訊，請參閱[了解 Azure Active Directory 應用程式資訊清單](active-directory-application-manifest.md)。

### 存取 Azure AD Graph 和 Office 365 API

如前文所述，除了在自己的資源應用程式上公開/存取 API，您也可以更新用戶端應用程式以存取 Microsoft 資源所公開的 API。依預設，Azure AD 圖形 API (在其他應用程式的 [權限] 清單中稱為 “Azure Active Directory”) 可供向 Azure AD 註冊的所有應用程式使用。如果您在 Office 365 所佈建的 Azure AD 租用戶中註冊您的用戶端應用程式，也可以存取 API 對各種 Office 365 資源公開的所有權限。

如需以下項目所公開的存取範圍的完整討論：

- Azure AD 圖形 API，請參閱 [權限範圍 | 圖形 API 概念](https://msdn.microsoft.com/Library/Azure/Ad/Graph/howto/azure-ad-graph-api-permission-scopes) 一文。
- Office 365 API，請參閱[使用一般同意架構的驗證和授權](https://msdn.microsoft.com/office/office365/howto/application-manifest)一文。如需更多有關如何建置可與 Office 365 API 整合之用戶端應用程式的討論，請參閱[設定 Office 365 開發環境](https://msdn.microsoft.com/office/office365/HowTo/setup-development-environment)。

>[AZURE.NOTE] 由於目前的限制，如果原生用戶端應用程式使用「存取組織的目錄」權限，它們只能呼叫 Azure AD 圖形 API。這項限制不適用於 Web 應用程式。

### 設定多租用戶應用程式

將應用程式新增至 Azure AD 時，您可以只讓組織中的使用者存取您的應用程式。或者，您可能想讓外部組織的使用者存取您的應用程式。這兩個應用程式類型稱為單一租用戶和多租用戶應用程式。您可以修改單一租用戶應用程式的組態，使其成為多組用戶應用程式，將於本章節下方討論。

請務必注意單一租用戶與多租用戶應用程式之間的差異：

- 單一租用戶應用程式適合在一個組織中使用。它們通常是由企業開發人員撰寫的企業營運 (LoB) 應用程式。單一租用戶應用程式只需要由一個目錄中的使用者存取，因此，它只需要佈建在一個目錄中。
- 多租用戶應用程式適合在許多組織中使用。它們是通常由獨立軟體廠商 (ISV) 撰寫的軟體即服務 (SaaS) Web 應用程式。多租用戶應用程式需要佈建在將會用到它們的每個目錄中，而這需要使用者或系統管理員同意才能註冊 (透過 Azure AD 同意架構支援)。請注意，所有原生用戶端應用程式預設有多個租用戶，因為其安裝於資源擁有者的裝置上。如需同意架構的詳細資訊，請參閱上面的「同意架構的概觀」一節。

#### 讓外部使用者可以授與您的應用程式存取其資源的權限

如果您正在撰寫想要提供給組織外的客戶或合作夥伴使用的應用程式，您必須在 Azure 傳統入口網站中更新應用程式定義。

>[AZURE.NOTE] 啟用多租用戶時，您必須確定應用程式的應用程式識別碼 URI 屬於已驗證的網域。此外，傳回 URL 必須以 https:// 開頭。如需詳細資訊，請參閱[應用程式物件和服務主體物件](active-directory-application-objects.md)。

讓外部使用者存取您的應用程式：

1. 登入 [Azure 傳統入口網站](https://manage.windowsazure.com)。

1. 按一下左側功能表中的 Active Directory 圖示，然後按一下想要的目錄。

1. 在頂端功能表上，按一下 [應用程式]，然後按一下您想要設定的應用程式。[快速啟動] 頁面將會顯示組態選項。

1. 展開 [快速啟動] 的 [設定多租用戶應用程式] 區段，然後按一下 [啟用存取] 區段中的**立即設定**連結。應用程式屬性頁面會隨即出現。

1. 按一下 [應用程式為多租用戶] 旁的 [是] 按鈕，然後按一下命令列上的 [儲存] 按鈕。

一旦您進行上述變更，其他組織中的使用者和系統管理員將可以讓您的應用程式存取他們的目錄和其他資料。

#### 在執行階段觸發 Azure AD 同意架構

若要使用同意架構，多租用戶的用戶端應用程式必須使用 OAuth 2.0 要求授權。[程式碼範例](https://azure.microsoft.com/documentation/samples/?service=active-directory&term=multi-tenant)也可向您示範 Web 應用程式、原生應用程式或伺服器/精靈應用程式如何要求授權碼並存取權杖以呼叫 Web API。

Web 應用程式也可提供使用者的註冊體驗。如果您已提供註冊體驗，預期使用者會按一下註冊按鈕，將瀏覽器重新導向至 Azure AD OAuth2.0 授權端點或 OpenID Connect userinfo 端點。這些端點可讓應用程式藉由檢查 id\_token 取得新使用者的相關資訊。在註冊階段後，使用者將會看到類似上面「同意架構的概觀」一節所示的同意提示。

或者，您的 Web 應用程式可能還會提供可允許系統管理員「註冊我的公司」的體驗。這種體驗也會將使用者重新導向至 Azure AD OAuth 2.0 授權端點。在此情況下，您會將 prompt=admin\_consent 參數傳遞至授權端點，強迫系統管理員同意體驗，系統管理員會在其中代表其組織授與同意。只有使用屬於全域管理員角色的帳戶進行驗證的使用者可以提供同意；其他人會收到錯誤。成功同意時，回應將包含 admin\_consent=true。兌換存取權杖時，您也將獲得提供組織資訊的 id\_token，以及會註冊您的應用程式的系統管理員。

### 啟用單一頁面應用程式的 OAuth 2.0 隱含授權

單一頁面應用程式 (SPA) 通常會利用執行於瀏覽器中的 JavaScript-heavy 前端進行結構化，此前端會呼叫應用程式的 Web API 後端以執行其商務邏輯。針對裝載於 Azure AD 中的 SPA，您會使用 OAuth 2.0 隱含授權驗證具備 Azure AD 的使用者，並取得您可以使用的權杖以保護從應用程式的 JavaScript 用戶端到其後端 Web API 的呼叫。使用者授與同意權之後，這個相同的驗證通訊協定可用來取得權杖以保護用戶端和其他為應用程式設定之 Web API 資源之間的呼叫。若要深入了解隱含授權授與，並協助您決定其是否適合您的應用程式案例，請參閱 [了解 Azure Active Directory 中的 OAuth2 隱含授與流程](active-directory-dev-understanding-oauth2-implicit-grant.md)。

根據預設，應用程式的 OAuth 2.0 隱含授權會停用。您也可以設定[應用程式資訊清單](active-directory-application-manifest.md)中的 `oauth2AllowImplicitFlow`”` 值，以啟用應用程式的「OAuth 2.0 隱含授權」，此值為表示應用程式身分識別組態的 JSON 檔案。

#### 啟用 OAuth 2.0 隱含授權 

1. 登入 [Azure 傳統入口網站](https://manage.windowsazure.com)。
1. 按一下左側功能表中的 **Active Directory** 圖示，然後按一下想要的目錄。
1. 在頂端功能表上，按一下 [應用程式]，然後按一下您想要設定的應用程式。單一登入及其他組態資訊都會顯示 [快速啟動] 頁面。
1. 按一下命令列中的 [管理資訊清單] 按鈕，並選取 [下載資訊清單]。開啟 JSON 應用程式資訊清單檔案並將 "oauth2AllowImplicitFlow" 值設為 “true”。根據預設，它是 “false”。

    `"oauth2AllowImplicitFlow": true,`

1. 儲存更新的 JSON 檔案並將其上傳，方法是按一下命令列中的 [管理資訊清單] 按鈕、選取 [上傳資訊清單]、瀏覽至您的已更新資訊清單檔案並將其選取。上傳之後，您的 Web API 現在已設定為使用 OAuth 2.0 隱含授權來驗證使用者。


### 授與存取權的舊版體驗

本節描述 2014 年 3 月 12 日之前的舊版同意體驗。這種體驗仍受支援且如下所述。在使用新功能之前，您只能授與下列權限：

- 登入其組織的使用者

- 登入使用者並讀取其組織的目錄資料 (僅限應用程式)

- 登入使用者並讀取與寫入其組織的目錄資料 (僅限應用程式)

您可遵循[利用 Azure AD 開發多租用戶 Web 應用程式](https://msdn.microsoft.com/library/azure/dn151789.aspx)中的步驟，授與在 Azure AD 中註冊之新應用程式的存取權。請務必注意，新的同意架構可允許更強大的應用程式，也可讓使用者 (而不只是系統管理員) 同意這些應用程式。

#### 建置授與存取權給外部使用者的連結 (舊版)

為了讓外部使用者使用其組織帳戶註冊您的應用程式，您必須更新您的應用程式，在 Azure AD 上顯示連結至頁面的按鈕，讓他們可以授與存取權。這個註冊按鈕的商標指導方針將於[整合應用程式的商標指導方針](active-directory-branding-guidelines.md)主題中討論。使用者授與或拒絕存取權之後，Azure AD 授與存取頁面會利用回應將瀏覽器重新導向回您的應用程式。如需應用程式屬性的詳細資訊，請參閱[應用程式物件與服務原則](active-directory-application-objects.md)。

授與存取頁面由 Azure AD 建立，您可以在 Azure 傳統入口網站的應用程式 [組態] 頁面中找到連結。若要移至 [設定] 頁面，請按一下 Azure AD 租用戶頂端功能表中的 [應用程式] 連結、按一下您想要設定的應用程式，然後從 [快速啟動] 頁面的頂端功能表按一下 [設定]。

您的應用程式連結看起來像這樣：`http://account.activedirectory.windowsazure.com/Consent.aspx?ClientID=058eb9b2-4f49-4850-9b78-469e3176e247&RequestedPermissions=DirectoryReaders&ConsentReturnURL=https%3A%2F%2Fadatum.com%2FExpenseReport.aspx%3FContextId%3D123456`。下表描述部分的連結：

|參數|說明|
|---|---|
|ClientId|必要。新增應用程式期間取得的用戶端識別碼。|
|RequestedPermissions|選用。應用程式要求的存取層級，將會向授與應用程式存取權的使用者顯示。如果未指定，要求的存取層級會預設為僅單一登入。其他選項為 DirectoryReaders 和 DirectoryWriters。如需這些存取層級的詳細資訊，請參閱應用程式存取層級。|
|ConsentReturnUrl|選用。您希望傳回授與存取回應的目標 URL。此值必須以 URL 編碼，且必須和設定於應用程式定義中的回覆 URL 在相同的網域下。如果未提供，授與存取回應將重新導向至已設定的回覆 URL。|

指定 ConsentReturnUrl 和回覆 URL 彼此獨立可讓您的應用程式實作獨立的邏輯，以處理和回覆 URL 不同之 URL 上的回應 (通常會處理登入的 SAML 權杖)。您也可以在 ConsentReturnURL 編碼的 URL 中指定其他參數。這些參數會在重新導向時，以查詢字串參數的形式傳回至應用程式。此機制可用來維護其他資訊，或將存取權授與的應用程式要求繫結至來自 Azure AD 的回應。

#### 授與存取權使用者經驗和回應 (舊版)

當應用程式重新導向至授與存取連結時，下列映像會顯示使用者會體驗的內容。

如果使用者尚未登入，它們將收到登入的提示：

![登入 AAD](./media/active-directory-integrating-applications/signintoaad.png)

一旦使用者通過驗證，Azure AD 會將使用者重新導向至授與存取頁面：

![授與存取權](./media/active-directory-integrating-applications/grantaccess.png)

>[AZURE.NOTE] 只有外部組織的公司系統管理員可以授與存取權給您的應用程式。如果使用者不是公司系統管理員，他們將可以選擇傳送郵件給他們的公司系統管理員，以要求授與此應用程式的存取權。

客戶藉由按一下 [授與存取權] 將存取權授與您的應用程式，或者如果它們藉由按一下 [取消] 拒絕存取權之後，Azure AD 會傳送回應至 ConsentReturnUrl 或已設定的回覆 URL。此回應包含下列參數：

|參數|說明|
|---|---|
|TenantId|Azure AD 中已將存取權授與您應用程式的組織的唯一識別碼。只有在客戶已授與存取權時，才會指定此參數。|
|同意|如果存取權已授與應用程式，此值會設為「已授與」，如果此要求遭到拒絕，則設為「已拒絕」。|

如果其他參數指定做為 ConsentReturnUrl 編碼 URL 的一部分，它們會被傳回應用程式。以下是授與存取要求的範例回應，指出已授權應用程式，且其包含授與存取要求中提供的 ContextID：`https://adatum.com/ExpenseReport.aspx?ContextID=123456&Consent=Granted&TenantId=f03dcba3-d693-47ad-9983-011650e64134`。

>[AZURE.NOTE] 存取授與回應將不會包含使用指的安全性權杖。應用程式必須個別登入使用者。

以下是存取授與要求遭到拒絕的範例回應：`https://adatum.com/ExpenseReport.aspx?ContextID=123456&Consent=Denied`

#### 變換應用程式金鑰以使圖形 API 的存取不會中斷 (舊版)

在您的應用程式存留期間，您可能需要在呼叫 Azure AD 取得存取權杖以呼叫圖形 API 時變更您所使用的金鑰。經常變更的金鑰分成兩類：當您的金鑰遭到入侵時的緊急變換，或是目前的金鑰即將到期的變更。請遵循下列程序，在您重新整理金鑰時提供不中斷的存取權 (主要針對第二個案例)。

1. 在 Azure 傳統入口網站中，按一下目錄租用戶、按一下頂端功能表的 [應用程式]，然後按一下您想要設定的應用程式。單一登入及其他組態資訊都會顯示 [快速啟動] 頁面。

1. 若要查看您的應用程式屬性清單，按一下頂端功能表中的 [設定]，就會看到您的金鑰清單。

1. 在 [金鑰] 底下，按一下 [選取持續時間] 下拉式清單，挑選 1 或 2 年，然後按一下命令列中的 [儲存]。這會產生您應用程式的新密碼金鑰。複製這個新的密碼金鑰。此時現有和新的金鑰都可由您的應用程式用來從 Azure AD 取得存取權杖。

1. 回到您的應用程式並更新組態以開始使用新的密碼金鑰。如需此更新應於何處發生的範例，請參閱[使用圖形 API 查詢 Azure AD](https://msdn.microsoft.com/library/azure/dn151791.aspx)。

1. 您現在應該在整個生產環境啟用此變更 – 在其他所有服務節點上啟用此變更之前，先在一個服務節點上加以驗證。

1. 一旦在整個生產部署完成此更新之後，您就可以自由回到 Azure 傳統入口網站並移除舊的金鑰。

#### 啟用存取之後變更應用程式屬性 (舊版)

一旦您讓外部使用者存取您的應用程式，您可能還是會繼續在 Azure 傳統入口網站中變更您的應用程式屬性。不過，在您變更應用程式前授與您的應用程式存取權的客戶，在 Azure 傳統入口網站中檢視您的應用程式相關詳細資料時，將不會看到那些變更。一旦應用程式可供客戶使用之後，您在進行某些變更時必須要非常小心。例如，如果您更新應用程式識別碼 URI，在此變更之前授與存取權的現有客戶將無法使用其公司或學校帳戶登入您的應用程式。

如果您變更 RequestedPermissions 以要求更高的存取層級，現有客戶使用的應用程式功能目前已使用此提高的存取層級，利用新的 API 呼叫，所以現有客戶可能會從圖形 API 得到拒絕存取的回應。您的應用程式應該處理這些案例，並要求客戶將存取權授與您要求提高存取層級的應用程式。

## 移除應用程式

本節說明如何從 Azure AD 租用戶移除應用程式。

### 移除您的組織所編寫的應用程式
這些是 Azure AD 租用戶的主要 [應用程式] 頁面上 [我的公司擁有的應用程式] 篩選器之下顯示的應用程式。在技術層面，這些是您透過 Azure 傳統入口網站以手動方式註冊的應用程式，或透過 PowerShell 或圖形 API 以程式設計方式註冊的應用程式。更具體來說，它們會由您租用戶中的應用程式與服務主體物件所表示。如需詳細資訊，請參閱[應用程式物件和服務主體物件](active-directory-application-objects.md)。

#### 從您的目錄移除單一租用戶應用程式

1. 登入 [Azure 傳統入口網站](https://manage.windowsazure.com)。

1. 按一下左側功能表中的 Active Directory 圖示，並按一下想要的目錄。

1. 在頂端功能表上，按一下 [應用程式]，然後按一下您想要設定的應用程式。單一登入及其他組態資訊都會顯示 [快速啟動] 頁面。

1. 按一下命令列中的 [刪除] 按鈕。

1. 在確認訊息處按一下 [是]。

#### 從您的目錄移除多租用戶應用程式

1. 登入 [Azure 傳統入口網站](https://manage.windowsazure.com)。

1. 按一下左側功能表中的 Active Directory 圖示，並按一下想要的目錄。

1. 在頂端功能表上，按一下 [應用程式]，然後按一下您想要設定的應用程式。單一登入及其他組態資訊都會顯示 [快速啟動] 頁面。

1. 在 [應用程式是多租用戶] 區段中，按一下 [否]。這樣會將您的應用程式轉換為單一租用戶，但應用程式仍會留在授與其同意權的組織中。

1. 按一下命令列中的 [刪除] 按鈕。

1. 在確認訊息處按一下 [是]。

### 移除其他組織授權的多租用戶應用程式
這些是 Azure AD 租用戶的主要 [應用程式] 頁面上 [我的公司使用的應用程式] 篩選器之下顯示的部份應用程式，特別是未列在 [我的公司擁有的應用程式] 清單下的應用程式。在技術層面，這些是在同意處理期間註冊的多租用戶應用程式。更具體來說，它們僅由您租用戶中的服務主體物件表示。如需詳細資訊，請參閱[應用程式物件和服務主體物件](active-directory-application-objects.md)。

為了移除多租用戶應用程式對您的目錄的存取權 (在授與同意之後)，公司系統管理員必須具有 Azure 訂用帳戶才能透過 Azure 傳統入口網站移除存取權。您只需瀏覽至應用程式的 [設定] 頁面，並按一下底部的 [管理存取權] 按鈕。或者，公司系統管理員可以使用 [Azure AD PowerShell Cmdlet](http://go.microsoft.com/fwlink/?LinkId=294151) 移除存取權。

## 後續步驟

- 請參閱[整合應用程式的商標指導方針](active-directory-branding-guidelines.md)

- 了解[應用程式物件和服務主體物件](active-directory-application-objects.md)相關資訊

- 了解[Azure Active Directory 應用程式資訊清單](active-directory-application-manifest.md)相關資訊

- 請瀏覽 [Azure Active Directory 開發人員指南](active-directory-developers-guide.md)

<!---HONumber=AcomDC_0727_2016-->