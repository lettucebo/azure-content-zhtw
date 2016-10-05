<properties
	pageTitle="新增來自 Azure Active Directory 中其他目錄或合作夥伴公司的使用者 | Microsoft Azure"
	description="說明如何在 Azure Active Directory 中新增使用者或變更使用者資訊，包括外部使用者和來賓使用者。"
	services="active-directory"
	documentationCenter=""
	authors="curtand"
	manager="femila"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="get-started-article"
	ms.date="09/22/2016"
	ms.author="curtand"/>

# 新增來自 Azure Active Directory 中其他目錄或合作夥伴公司的使用者

> [AZURE.SELECTOR]
- [Azure 入口網站](active-directory-users-create-external-azure-portal.md)
- [Azure 傳統入口網站](active-directory-create-users-external.md)

本文說明如何新增 Azure Active Directory 中其他目錄的使用者或新增合作夥伴公司的使用者。如需新增您的組織內之使用者以及新增具有 Microsoft 帳戶之使用者的相關資訊，請參閱[將新的使用者新增到 Azure Active Directory](active-directory-create-users.md)。新增的使用者預設不會有系統管理員權限，但是您可以隨時指派角色給他們。

## 新增使用者

1. 使用屬於目錄全域管理員的帳戶登入 [Azure 傳統入口網站](https://manage.windowsazure.com)。

2. 選取 [Active Directory]，然後開啟您的目錄。

3. 選取 [使用者] 索引標籤，然後在命令列中選取 [新增使用者]。

4. 在 [告訴我們這位使用者] 頁面上，於 [使用者類型] 底下選取下列其中一項：

	- **另一個 Azure AD 目錄中的使用者** – 將源自另一個 Azure AD 目錄的使用者帳戶新增至您的目錄。只有在您也是另一個目錄的成員時，才能選取該目錄中的使用者。
	- **合作夥伴公司中的使用者** - 邀請並授權合作夥伴公司使用者使用您的目錄 ([請參閱 Azure Active Directory B2B 共同作業](active-directory-b2b-what-is-azure-ad-b2b.md))。您將需要[上傳指定電子郵件地址的 CSV 檔案](active-directory-b2b-references-csv-file-format.md)。

6. 在 [使用者設定檔] 頁面上，提供姓氏和名字、使用者易記名稱，並從 [角色] 清單中選擇使用者角色。如需有關使用者和系統管理員角色的詳細資訊，請參閱[在 Azure AD 中指派系統管理員角色](active-directory-assign-admin-roles.md)。指定是否為使用者**啟用 Multi-Factor Authentication**。

7. 在 [取得暫時密碼] 頁面上，選取 [建立]。

> [AZURE.IMPORTANT] 如果您的組織使用多個網域，當您新增使用者帳戶時，請注意下列問題：
>
> - 若要跨網域新增具有相同使用者主體名稱 (UPN) 的使用者帳戶，請「先」新增 geoffgrisso@contoso.onmicrosoft.com，「再」新增 geoffgrisso@contoso.com。
> - 請「勿」先新增 geoffgrisso@contoso.com 再新增 geoffgrisso@contoso.onmicrosoft.com。此順序很重要，事後想要復原會很麻煩。

如果您變更其身分識別已與您的內部部署 Active Directory 服務同步處理之使用者的資訊，您就無法在 Azure 傳統入口網站中變更使用者資訊。若要變更此使用者資訊，請使用您的內部部署 Active Directory 管理工具。

## 新增外部使用者

您也可以從另一個所屬 Azure AD 目錄，或從合作夥伴公司上傳 CSV 檔案來新增使用者。若要新增外部使用者，請在 [使用者類型] 中指定 [另一個 Microsoft Azure AD 目錄中的使用者] 或 [合作夥伴公司中的使用者]。

這兩種類型的使用者是源自另一個目錄，並且新增為 [外部使用者]。外部使用者可以和目錄中的其他使用者共同作業，而不需要另外新增帳戶和認證。外部使用者登入時會使用其主目錄進行驗證，驗證結果適用於其已加入的其他所有目錄。

## 外部使用者管理和限制

當您將另一個目錄的使用者加入至您的目錄時，該使用者在您的目錄中就是外部使用者。顯示名稱和使用者名稱會從其主目錄複製而來，並用於您的目錄中的外部使用者。從此以後，外部使用者帳戶的屬性就會完全獨立。如果對使用者在其主目錄中變更屬性，這些變更不會傳播到您的目錄中的外部使用者帳戶。

兩個帳戶之間的唯一連結是，使用者永遠針對其主目錄或使用其 Microsoft 帳戶進行驗證。這就是為什麼您沒有看到針對外部使用者重設密碼或啟用 Multi-Factor Authentication 的選項。目前，主目錄或 Microsoft 帳戶的驗證原則是使用者登入時唯一會評估的原則。

> [AZURE.NOTE]
您仍然可以停用目錄中的外部使用者，以阻止其存取您的目錄。

如果使用者在其主目錄中被刪除，或取消其 Microsoft 帳戶，外部使用者仍存在您的目錄中。不過，您的目錄中的使用者無法存取資源，因為這些資源無法使用主目錄或 Microsoft 帳戶進行驗證。

### 目前支援讓 Azure AD 外部使用者存取的服務

- **Azure 傳統入口網站**：允許身為多個目錄的管理員的外部使用者，管理這些目錄。
- **SharePoint Online**：如果啟用外部共用，允許外部使用者存取 SharePoint Online 授權資源。
- **Dynamics CRM**：如果使用者透過 PowerShell 獲得授權，允許外部使用者存取 Dynamics CRM 中的授權資源。
- **Dynamics AX**：如果使用者透過 PowerShell 獲得授權，允許外部使用者存取 Dynamics AX 中的授權資源。[Azure AD 外部使用者](#known-limitations-of-azure-ad-external-users)的限制也適用於 Dynamics AX 中的外部使用者。

### Azure AD 外部使用者的已知限制

- 身為管理員的外部使用者無法將來自合作夥伴公司的使用者加入至其主目錄以外的目錄 (B2B 共同作業)
- 外部使用者無法同意在其主目錄以外的目錄中的多租用戶應用程式
- PowerBI 目前不支援讓外部使用者存取
- Office 入口網站不支援授權外部使用者
- 對於 Azure AD PowerShell，外部使用者會登入其主目錄，因此無法管理其身分為外部使用者的目錄


## 後續步驟

- [將新的使用者加入 Azure Active Directory](active-directory-create-users.md)
- [管理 Azure AD](active-directory-administer.md)
- [在 Azure AD 中管理密碼](active-directory-manage-passwords.md)
- [在 Azure AD 中管理群組](active-directory-manage-groups.md)

<!---HONumber=AcomDC_0928_2016-->