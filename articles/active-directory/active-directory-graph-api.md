<properties
   pageTitle="Azure Active Directory 圖形 API | Microsoft Azure"
   description="圖形 API 的概觀和快取入門指南，可讓您以程式設計方式透過 REST API 端點存取 Azure AD。"
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
   ms.date="09/16/2016"
   ms.author="mbaldwin" />

# Azure Active Directory 圖形 API

> [AZURE.IMPORTANT] Azure AD Graph API 功能也透過 [Microsoft Graph](https://graph.microsoft.io/) 提供使用，Microsoft Graph 是一個統一的 API，包含了來自其他 Microsoft 服務 (例如 Outlook、OneDrive、OneNote、Planner 和 Office Graph) 的 API，並可透過單一端點與使用單一存取權杖進行存取。

Azure Active Directory 圖形 API 支援以程式設計方式透過 REST API 端點存取 Azure AD。應用程式可以使用 Graph API，來執行有關目錄資料和物件的建立、讀取、更新及刪除 (CRUD) 作業。例如，圖形 API 支援對使用者物件執行下列常見的作業：

- 在目錄中建立新的使用者

- 取得使用者的詳細屬性，例如其群組

- 更新使用者的屬性 (例如其位置和電話號碼)，或變更其密碼

- 檢查使用者在角色型存取方面的群組成員資格

- 停用使用者帳戶或完全刪除

除了使用者物件，您也可以在其他物件上執行類似的作業，例如群組和應用程式。若要在目錄上呼叫圖形 API，應用程式必須向 Azure AD 註冊並設定為允許存取此目錄。這通常是透過使用者或系統管理員同意流程來達成。

若要開始使用 Azure Active Directory Graph API，請參閱[Graph API 快速入門指南](active-directory-graph-api-quickstart.md)，或檢視[互動式 Graph API 參考文件](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/api-catalog)。


## 特性

圖形 API 提供下列功能：

- **REST API 端點**：Graph API 是 RESTful 服務，由使用標準 HTTP 要求存取的端點所組成。圖形 API 支援要求和回應的 XML 或 Javascript 物件標記法 (JSON) 內容類型。如需詳細資訊，請參閱 [Azure AD Graph REST API 參考](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/api-catalog)。

- **向 Azure AD 驗證**：必須在要求的 Authorization 標頭中附加 JSON Web Token (JWT)，以驗證 Graph API 的每個要求。您可以向 Azure AD 的權杖端點提出要求並提供有效的認證，以取得此權杖。您可以使用 OAuth 2.0 用戶端認證流程或授權碼授與流程，以取得權杖來呼叫 Graph。如需詳細資訊，請參閱 [Azure AD 中的 OAuth 2.0](https://msdn.microsoft.com/library/azure/dn645545.aspx)。

- **以角色為基礎的授權 (RBAC)**：在 Graph API 中使用安全性群組執行 RBAC。例如，如果您想要判斷使用者是否能夠存取特定的資源，應用程式可以呼叫[檢查群組成員資格 (可移轉)](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/groups-operations#FunctionsandactionsongroupsCheckmembershipinaspecificgrouptransitive) 作業，它會傳回 true 或 false。

- **差異查詢**：如果您想要檢查目錄在兩段時間之間的變更，但不想對 Graph API 進行頻繁的查詢，您可以提出差異查詢要求。此類型的要求只會傳回前一個差異查詢要求與目前要求之間所做的變更。如需詳細資訊，請參閱 [Azure AD Graph API 差異查詢](https://msdn.microsoft.com/Library/Azure/Ad/Graph/howto/azure-ad-graph-api-differential-query)。

- **目錄延伸模組**：如果您正在開發的應用程式需要讀取或寫入目錄物件的唯一屬性，您可以使用 Graph API 註冊並使用延伸模組值。例如，如果應用程式需要每個使用者都有 Skype ID 屬性，您可以在目錄中註冊新的屬性，每個使用者物件上就會有此屬性。如需詳細資訊，請參閱 [Azure AD Graph API 目錄結構描述延伸模組](https://msdn.microsoft.com/Library/Azure/Ad/Graph/howto/azure-ad-graph-api-directory-schema-extensions)。

- **受權限範圍保護**：AAD 圖形 API 會公開權限範圍，以啟用 AAD 資料的安全/同意存取，並支援各種用戶端應用程式類型，包括︰
 - 透過登入使用者 (委派) 授權，指定具有資料委派存取權之使用者介面的類型
  - 使用服務/服務精靈用戶端 (應用程式角色) 等應用程式定義角色型存取控制的類型

    委派和應用程式角色權限範圍都代表圖形 API 公開的權限，而且用戶端應用程式可以透過應用程式註冊權限要求它們 ([Azure 傳統入口網站中的功能](https://manage.windowsazure.com))。用戶端可以驗證被授與的權限範圍：檢查委派權限的存取權杖中所收到的範圍 ("scp") 宣告，和應用程式角色權限的角色 (“roles”) 宣告。深入了解 [Azure AD 圖形 API 權限範圍](https://msdn.microsoft.com/Library/Azure/Ad/Graph/howto/azure-ad-graph-api-permission-scopes)。


## 案例

圖形 API 支援許多應用程式案例。以下是最常見的案例：

- **企業營運 (單一租用戶) 應用程式**：在此案例中，企業開發人員任職於具有 Office 365 訂用帳戶的組織中。開發人員正在建置的 Web 應用程式會與 Azure AD 互動來執行工作，例如指派授權給使用者。這項工作需要存取圖形 API，所以開發人員在 Azure Ad 中註冊單一租用戶應用程式，並設定圖形 API 的讀取和寫入權限。然後，將應用程式設定為使用它自己的認證，或目前登入之使用者的認證，以取得權杖來呼叫圖形 API。

- **軟體即服務應用程式 (多租用戶)**：在此案例中，獨立軟體廠商 (ISV) 正在開發託管的多租用戶 Web 應用程式，目的是為使用 Azure AD 的其他組織提供使用者管理功能。這些功能需要存取目錄物件，所以此應用程式需要呼叫圖形 API。開發人員在 Azure AD 中註冊此應用程式，將它設定為需要圖形 API 的讀取和寫入權限，然後啟用外部存取，讓其他組織同意在其目錄中使用此應用程式。當另一個組織中的使用者第一次向應用程式驗證時，就會出現同意對話方塊及此應用程式所要求的權限。同意之後，就會給予所要求的權限，讓應用程式在使用者的目錄中存取圖形 API。如需同意架構的詳細資訊，請參閱[同意架構的概觀](active-directory-integrating-applications.md)。

## 另請參閱

[Azure AD Graph API 快速入門指南](active-directory-graph-api-quickstart.md)

[AD Graph REST 文件](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/api-catalog)

[Azure Active Directory 開發人員指南](active-directory-developers-guide.md)

<!---HONumber=AcomDC_0921_2016-->