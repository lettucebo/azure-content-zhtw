<properties
   pageTitle="如何啟用或停用角色 | Microsoft Azure"
   description="了解如何利用 Azure Privileged Identity Management 應用程式，啟用特殊權限身分識別的角色。"
   services="active-directory"
   documentationCenter=""
   authors="kgremban"
   manager="femila"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="identity"
   ms.date="09/16/2016"
   ms.author="kgremban"/>

# Azure AD Privileged Identity Management：如何啟用或停用角色

Azure Active Directory (AD) Privileged Identity Management 簡化了企業管理以特殊權限身分存取 Azure AD 中的資源和其他 Microsoft 線上服務 (如 Office 365 或 Microsoft Intune) 的方式。

如果您已被設為符合系統管理角色資格，即表示您可以在需要執行必須具備該角色才能執行的工作時，啟用該角色。例如，如果您偶爾會管理 Office 365 功能，則貴組織的特殊權限角色管理員可能不會讓您成為永久全域管理員，因為該角色也會影響其他服務。他們反而會讓您符合 Azure AD 角色 (例如「Exchange Online 管理員」) 的資格。您可以在需要權限時，要求暫時指派該角色，然後您將會在預定的時段內擁有系統管理控制權。

本文適用對象是必須在 Azure AD Privileged Identity Management (PIM) 中啟用其角色的系統管理員。它會逐步引導您在需要權限時啟用角色，並在完成時停用角色。


## 加入 Privileged Identity Management 應用程式

請在 [Azure 入口網站](https://portal.azure.com/)中使用 Azure AD Privileged Identity Management 應用程式來要求角色啟用，即使您是要在另一個入口網站中或透過 PowerShell 來操作，也是如此。如果您在 Azure 入口網站沒有 Azure AD Privileged Identity Management 應用程式，請遵循下列步驟操作。

1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 選取 Azure 入口網站右上角的使用者名稱，然後選取您要操作的目錄。
3. 選取 [更多服務] 並使用 [篩選器] 文字方塊來搜尋 [Azure AD Privileged Identity Management]。
4. 選取 [釘選到儀表板]，然後按一下 [建立]。Privileged Identity Management 應用程式隨即開啟。

## 啟用角色

當您需要擔任某個角色時，您可以使用 Azure AD Privileged Identity Management 應用程式中的 [啟用我的角色] 按鈕來要求啟用。


1. 登入 [Azure 入口網站](https://portal.azure.com/)，選取 [Azure AD Privileged Identity Management] 圖格。
2. 選取 [啟用我的角色]。已指派給您的角色清單隨即顯示。
3. 選取您想要啟用的角色。
4. 選取 [**啟用**]。[要求啟用角色] 刀鋒視窗隨即出現。
5. 有些角色會要求必須先執行 Multi-Factor Authentication (MFA) 才能啟用。您只需在每個工作階段驗證一次。

    ![在啟用角色之前先以 MFA 驗證 - 螢幕擷取畫面][2]

6. 在啟用要求的文字欄位中輸入啟用原因。有些角色會要求您提供問題票證號碼。
7. 選取 [確定]。角色現已啟用，而角色變更也會顯示在 Microsoft Online Services 中。

## 停用角色

角色一經啟用，就會在達到時間限制時自動停用。

如果您提早完成作業，也可以在 Azure AD Privileged Identity Management 應用程式中手動停用角色。選取 [啟用我的角色]、選擇您已使用完畢的角色，然後選取 [停用]。


<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## 後續步驟

如果您有興趣進一步了解 Azure AD Privileged Identity Management，下列連結會提供詳細資訊。

[AZURE.INCLUDE [active-directory-privileged-identity-management-toc](../../includes/active-directory-privileged-identity-management-toc.md)]

<!--Image references-->

[1]: ./media/active-directory-privileged-identity-management-configure/PIM_EnablePim.png
[2]: ./media/active-directory-privileged-identity-management-how-to-activate-role/PIM_activation_MFA.png

<!---HONumber=AcomDC_0921_2016-->