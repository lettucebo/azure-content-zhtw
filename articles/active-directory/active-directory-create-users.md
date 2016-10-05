<properties
	pageTitle="將新的使用者加入 Azure Active Directory | Microsoft Azure"
	description="說明如何在 Azure Active Directory 中新增新的使用者或變更使用者資訊。"
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

# 在 Azure Active Directory 新增新的使用者或具有 Microsoft 帳戶的使用者

請新增使用者來填入您的目錄。本文說明如何在組織中新增新的使用者，以及如何新增具有 Microsoft 帳戶的使用者。如需新增來自 Azure Active Directory 中其他目錄之使用者或是來自合作夥伴公司之使用者的詳細資訊，請參閱[新增來自 Azure Active Directory 中其他目錄或合作夥伴公司的使用者](active-directory-create-users-external.md)。新增的使用者預設不會有系統管理員權限，但是您可以隨時指派角色給他們。

## 新增使用者

1. 使用屬於目錄全域管理員的帳戶登入 [Azure 傳統入口網站](https://manage.windowsazure.com)。
2. 選取 [Active Directory]，然後選取貴組織目錄的名稱。
3. 選取 [使用者] 索引標籤，然後在命令列中選取 [新增使用者]。
4. 在 [告訴我們這位使用者] 頁面上，於 [使用者類型] 底下選取下列其中一項：

	- **您組織中的新使用者** – 在您的目錄中新增使用者帳戶。
	- **現有 Microsoft 帳戶的使用者** – 將現有的 Microsoft 取用者帳戶加入至您的目錄 (例如，Outlook 帳戶)。

5. 根據 [使用者類型] 輸入使用者名稱 (適用於新使用者) 或電子郵件地址 (適用於具有 Microsoft 帳戶的使用者)。
6. 在 [使用者設定檔] 頁面上，提供姓氏和名字、使用者易記名稱，並從 [角色] 清單中選擇使用者角色。如需有關使用者和系統管理員角色的詳細資訊，請參閱[在 Azure AD 中指派系統管理員角色](active-directory-assign-admin-roles.md)。指定是否為使用者**啟用 Multi-Factor Authentication**。
7. 在 [取得暫時密碼] 頁面上，選取 [建立]。

> [AZURE.IMPORTANT] 如果您的組織使用多個網域，當您新增使用者帳戶時，請注意下列問題：
>
> - 若要跨網域新增具有相同使用者主體名稱 (UPN) 的使用者帳戶，請「先」新增 geoffgrisso@contoso.onmicrosoft.com，「再」新增 geoffgrisso@contoso.com。
> - 請「勿」先新增 geoffgrisso@contoso.com 再新增 geoffgrisso@contoso.onmicrosoft.com。此順序很重要，事後想要復原會很麻煩。

## 變更使用者資訊

您可以變更任何使用者屬性，但物件識別碼除外。

1. 開啟您的目錄。
2. 選取 [使用者] 索引標籤，然後選取想要變更之使用者的顯示名稱。
3. 完成您的變更，然後按一下 [儲存]。

如果您嘗試變更的使用者已經與內部部署 Active Directory 服務同步處理，您將無法使用此程序來變更使用者資訊。若要變更此使用者，請使用您的內部部署 Active Directory 管理工具。

## 來賓使用者管理和限制

來賓帳戶是來自其他目錄的使用者，其已受邀至您的目錄以存取 SharePoint 文件、應用程式或其他 Azure 資源。您的目錄中的來賓帳戶會將其基礎 UserType 屬性設為 [來賓]。 一般使用者 (具體而言是指您目錄的成員) 的 UserType 屬性則為 [成員]。

來賓在目錄中有一組受限的權限。這些權限會禁止來賓探索目錄中其他使用者的相關資訊。不過，來賓使用者仍可與其使用之資源相關聯的使用者和群組互動。來賓使用者可以：

- 查看與他們被指派到的 Azure 訂用帳戶相關聯的使用者和群組
- 查看其所屬之群組的成員
- 如果他們已經知道使用者的完整電子郵件地址，則可以查閱目錄中的其他使用者
- 僅能查看他們查閱的使用者的有限屬性集 - 僅限於顯示名稱、電子郵件地址、使用者主體名稱 (UPN) 和相片縮圖
- 取得目錄中已驗證的網域清單
- 同意應用程式，授與它們與在您的目錄中相同的成員存取權

## 設定來賓使用者存取原則

目錄內的 [設定] 索引標籤內含可控制來賓使用者存取權限的選項。這些選項只可以由目錄全域管理員在 Azure 傳統入口網站中進行變更。目前沒有透過 PowerShell 或 API 的方法。

若要在 Azure 傳統入口網站中開啟 [設定] 索引標籤，請選取 [Active Directory]，然後選取目錄的名稱。

![在 Azure Active Directory 中設定索引標籤][1]

接著您可以編輯選項來控制來賓使用者的存取權限。

![來賓使用者的存取控制選項][2]


## 後續步驟

- [新增來自 Azure Active Directory 中其他目錄或合作夥伴公司的使用者](active-directory-create-users-external.md)
- [管理 Azure AD](active-directory-administer.md)
- [在 Azure AD 中管理密碼](active-directory-manage-passwords.md)
- [在 Azure AD 中管理群組](active-directory-manage-groups.md)

<!--Image references-->
[1]: ./media/active-directory-create-users/RBACDirConfigTab.png
[2]: ./media/active-directory-create-users/RBACGuestAccessControls.png

<!---HONumber=AcomDC_0928_2016-->