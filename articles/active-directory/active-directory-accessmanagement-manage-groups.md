<properties

	pageTitle="Managing groups in Azure Active Directory | Microsoft Azure"
	description="How to create and manage groups to manage Azure users using Azure Active Directory."
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


# 在 Azure Active Directory 中管理群組

Azure Active Directory (Azure AD) 使用者管理的其中一項功能是能夠建立使用者群組。您可以使用群組來執行管理工作，例如一次指派授權或權限給多名使用者。您也可以使用群組來指派存取權限給

- 資源，例如目錄中的物件
- 目錄外部的資源，例如 SaaS 應用程式、Azure 服務、SharePoint 網站或內部部署資源

此外，資源擁有者也可以指派資源存取權給其他人擁有的 Azure AD 群組。這項指派會將資源的存取權授與該群組的成員。然後，群組擁有者負責管理群組中的成員資格。實際上，資源擁有者是將指派使用者至其資源的權限委派給群組擁有者。

## 如何建立群組？

根據組織已訂閱的服務，您可以使用下列其中一項來建立群組︰
- Azure 傳統入口網站
- Office 365 帳戶入口網站
- Windows Intune 帳戶入口網站

我們會說明在 Azure 傳統入口網站中執行的工作。如需使用非 Azure 入口網站管理 Azure AD 目錄的詳細資訊，請參閱[管理 Azure AD 目錄](active-directory-administer.md)。

1. 在 [Azure 傳統入口網站](https://manage.windowsazure.com)中選取 [Active Directory]，然後選取組織的目錄名稱。

2. 選取 [群組] 索引標籤。

3. 選取 [新增群組]。

4. 在 [加入群組] 視窗中，指定群組的名稱與描述。


## 如何新增或移除安全性群組中的個別使用者？

**新增個別使用者到群組**

1. 在 [Azure 傳統入口網站](https://manage.windowsazure.com)中選取 [Active Directory]，然後選取組織的目錄名稱。

2. 選取 [群組] 索引標籤。

3. 開啟要在其中新增成員的群組。開啟所選取群組的 [成員] 索引標籤 (如果尚未顯示)。

4. 選取 [新增成員]。

5. 在 [新增成員] 頁面上，選取您想要加入成為此群組成員之使用者或群組的名稱。確定這個名稱加入了 [已選取] 窗格中。


**從群組中移除個別使用者**

1. 在 [Azure 傳統入口網站](https://manage.windowsazure.com)中選取 [Active Directory]，然後選取組織的目錄名稱。

2. 選取 [群組] 索引標籤。

3. 開啟要從中移除成員的群組。

4. 選取 [成員] 索引標籤，選取您想要從這個群組移除之成員的名稱，然後按一下 [移除]。

6. 在提示中確認您想要從群組中移除這個成員。


## 如何動態管理群組的成員資格？

在 Azure AD 中，您可以非常輕鬆地設定一個簡單的規則來判斷哪些使用者要成為群組的成員。簡單的規則是指僅進行一項比較的規則。例如，如果某個群組指派給 SaaS 應用程式，您即可設定規則來新增職稱為「業務代表」的使用者。 此規則接著便會授與此 SaaS 應用程式的存取權給目錄中所有具有該職稱的使用者。

當使用者的任何屬性變更時，系統會評估目錄中的所有動態群組規則，以查看使用者的屬性變更是否會觸發任何的群組新增或移除。如果使用者滿足群組規則，則使用者會新增為該群組的成員。如果他們不再滿足其所屬群組的規則，則會從該群組的成員中移除。

> [AZURE.NOTE] 您可以為安全性群組或 Office 365 群組的動態成員資格設定規則。目前對應用程式的群組式指派並不支援巢狀群組成員資格。
>
> 群組的動態成員資格需要將 Azure AD Premium 授權指派給：
>
> - 負責管理群組規則的系統管理員
> - 群組的所有成員

**啟用群組的動態成員資格**

1. 在 [Azure 傳統入口網站](https://manage.windowsazure.com)中選取 [Active Directory]，然後選取組織的目錄名稱。

2. 選取 [群組] 索引標籤，然後開啟您想要編輯的群組。

3. 選取 [設定] 索引標籤，然後將 [啟用動態成員資格] 設定為 [是]。

4. 為群組設定一個簡單的規則，以控制此群組的動態成員資格的運作方式。請確定已選取 [**新增使用者位置**] 選項，然後從清單 (例如，department、jobTitle 等) 中選取一個使用者屬性。

5. 接著，選取一個條件 (不等於、等於、開頭不是、開頭為、不包含、包含、不符合、符合)。

6. 指定所選使用者屬性的比較值。

若要了解如何為動態群組成員資格建立「進階」規則 (可包含多個比較的規則)，請參閱[使用屬性來建立進階規則](active-directory-accessmanagement-groups-with-advanced-rules.md)。

## 其他資訊

這些文章提供有關 Azure Active Directory 的其他資訊。

* [使用 Azure Active Directory 群組管理資源的存取權](active-directory-manage-groups.md)

* [設定群組設定的 Azure Active Directory Cmdlet](active-directory-accessmanagement-groups-settings-cmdlets.md)

* [Article Index for Application Management in Azure Active Directory (Azure Active Directory 中應用程式管理的文件索引)](active-directory-apps-index.md)

* [什麼是 Azure Active Directory？](active-directory-whatis.md)

* [整合內部部署身分識別與 Azure Active Directory](active-directory-aadconnect.md)

<!---HONumber=AcomDC_0928_2016-->