<properties
pageTitle="在 Azure Active Directory 預覽版中自訂登入頁面 | Microsoft Azure"
description="了解如何將公司商標新增到 Azure 登入頁面"
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
ms.topic="article"
ms.date="09/12/2016"
ms.author="curtand"/>

# 在 Azure Active Directory 預覽版中將公司商標新增到登入頁面

為了避免混淆，許多公司都想對其管理的所有網站和服務套用一致的外觀與風格。Azure Active Directory 是透過讓您利用公司標誌和自訂色彩配置來自訂登入頁面外觀的方式，提供這項功能。[預覽版有何功能？](active-directory-preview-explainer.md) 登入頁面是當您登入 Office 365 或其他使用 Azure AD 作為識別提供者的 Web 型應用程式時，所顯示的頁面。您可以與此頁面互動來輸入您的認證。

如果您想要在此頁面上顯示您的公司商標、色彩和其他可自訂的元素，請參閱下列影像以了解這兩種做法的差異。

下列螢幕擷取畫面顯示桌上型電腦上 Office 365 登入頁面的自訂「前」範例︰

![自訂前的 Office 365 登入頁面](./media/active-directory-branding-custom-signon-azure-portal/sign-in-page-before-customization.png)

下列螢幕擷取畫面顯示桌上型電腦上 Office 365 登入頁面的自訂「後」範例︰

![自訂後的 Office 365 登入頁面](./media/active-directory-branding-custom-signon-azure-portal/sign-in-page-after-customization.png)


## 自訂登入頁面

一般而言，如果您需要透過瀏覽器存取貴組織訂閱的雲端應用程式和服務，您可使用登入頁面。

如果您已對登入頁面套用變更，最多需要一小時變更才會出現。

只有當您使用租用戶特定 URL (例如 https://outlook.com/**contoso**.com 或 https://mail.**contoso**.com) 來造訪服務時，才會顯示加上商標的登入頁面。

當您使用非租用戶特定 URL (例如 https://mail.office365.com) 來造訪服務時，則會顯示沒有商標的登入頁面。在此情況下，只要您一輸入使用者識別碼或是選取使用者磚，就會顯示您的商標。

> [AZURE.NOTE]
>
- 在您已設定商標之 Azure 入口網站的 [網域] 部分，您的網域名稱必須顯示為 [作用中]。如需詳細資訊，請參閱[新增自訂網域名稱](active-directory-domains-add-azure-portal.md)。
- 登入頁面商標不會延續到 Microsoft 的消費者登入頁面。如果您使用 Microsoft 帳戶進行登入，您可能會看到 Azure AD 所呈現且已加上商標的使用者磚清單，但是您組織的商標不會套用到 Microsoft 帳戶登入頁面。

**將公司商標新增到您的目錄：**

1.  使用具備目錄全域管理員身分的帳戶來登入 [Azure 入口網站](https://portal.azure.com)。

2.  選取 [更多服務]，在文字方塊中輸入「使用者和群組」，然後選取 **Enter**。

    ![開啟使用者管理](./media/active-directory-branding-custom-signon-azure-portal/user-management.png)

3. 在 [使用者和群組] 刀鋒視窗上，選取 [公司商標]。

4. 在 [使用者和群組 - 公司商標] 刀鋒視窗上，選取 [編輯] 命令。

    ![編輯自訂商標](./media/active-directory-branding-custom-signon-azure-portal/edit-branding.png)

5. 修改您想要自訂的元素。所有元素都是選用的。

6. 按一下 [儲存]。

您對登入頁面商標所做的任何變更可能最多需要一個小時才會顯示。

## 後續步驟

[新增語言特定公司商標](active-directory-branding-localize-azure-portal.md)

<!---HONumber=AcomDC_0914_2016-->