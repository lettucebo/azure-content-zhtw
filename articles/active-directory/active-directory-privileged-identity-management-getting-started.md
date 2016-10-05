<properties
   pageTitle="開始使用 Azure AD Privileged Identity Management | Microsoft Azure"
   description="了解如何在 Azure 入口網站中使用 Azure Active Directory Privileged Identity Management 應用程式來管理特殊權限身分識別。"
   services="active-directory"
   documentationCenter=""
   authors="kgremban"
   manager="femila"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.devlang="na"
   ms.topic="get-started-article"
   ms.tgt_pltfrm="na"
   ms.workload="identity"
   ms.date="09/16/2016"
   ms.author="kgremban"/>

# 開始使用 Azure AD Privileged Identity Management


您可以利用 Azure Active Directory (AD) Privileged Identity Management 來管理、控制和監視組織內的存取行為。這包括存取 Azure AD 中的資源和其他 Microsoft 線上服務，例如 Office 365 或 Microsoft Intune。

本文會告訴您如何將 Azure AD PIM 應用程式加入 Azure 入口網站儀表板中。

## 加入 Privileged Identity Management 應用程式

使用 Azure AD Privileged Identity Management 之前，您需要先將應用程式加入 Azure 入口網站儀表板中。

1. 以目錄的全域系統管理員身分登入 [Azure 入口網站](https://portal.azure.com/)。
2. 如果貴組織有多個目錄，請選取 Azure 入口網站右上角的使用者名稱。選取您將在其中使用 PIM 的目錄。
3. 選取 [更多服務] 並使用 [篩選器] 文字方塊來搜尋 [Azure AD Privileged Identity Management]。
4. 選取 [釘選到儀表板]，然後按一下 [建立]。Privileged Identity Management 應用程式隨即開啟。


如果您是在目錄中使用 Azure AD Privileged Identity Management 的第一個人，則[安全性精靈](active-directory-privileged-identity-management-security-wizard.md)會引導您完成初始指派體驗。之後，您就會自動成為目錄的第一個「安全性系統管理員」和「特殊權限角色管理員」。只有特殊權限角色管理員可以存取這個應用程式來管理其他管理員的存取權。

## 瀏覽至您的工作

Azure AD Privileged Identity Management 設定完畢後，您會在每次開啟應用程式時看到瀏覽刀鋒視窗。使用此刀鋒視窗來完成您的身分識別管理工作。

![PIM 的最上層工作 - 螢幕擷取畫面](./media/active-directory-privileged-identity-management-getting-started/pim_tasks.png)

- **啟用我的角色**會帶您前往已指派給您的角色清單。您將在其中啟用任何您具備資格的角色。

- **管理特殊權限角色**是特殊權限角色管理員用來管理角色指派、變更角色啟用設定、開始存取權檢閱等的儀表板。對於任何不是特殊權限角色管理員的人員，此儀表板中的選項會停用。

- **檢閱特殊權限存取**會帶您前往任何需要完成的擱置中存取權檢閱 (無論您是否正在檢閱自己或他人的存取權)。


<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## 後續步驟

[Azure AD Privileged Identity Management 概觀](active-directory-privileged-identity-management-configure.md)包含如何管理貴組織系統管理存取的更多詳細資訊。

[AZURE.INCLUDE [active-directory-privileged-identity-management-toc](../../includes/active-directory-privileged-identity-management-toc.md)]

<!--Image references-->

[1]: ./media/active-directory-privileged-identity-management-configure/PIM_EnablePim.png

<!---HONumber=AcomDC_0921_2016--->