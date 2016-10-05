<properties
   pageTitle="Azure Active Directory 報告「未知的執行者」| Microsoft Azure"
   description="說明 Azure Active Directory 報告中的「未知的執行者」事件"
   services="active-directory"
   documentationCenter=""
   authors="SSalahAhmed"
   manager="mbaldwin"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="identity"
   ms.date="09/16/2016"
   ms.author="saah"/>

# Azure Active Directory 報告「未知的執行者」事件

*這份文件是 [Azure Active Directory 報告指南](active-directory-reporting-guide.md)的一部分。*

在極少數情況下，您可能會在 Azure AD 報告的 [執行者] 或 [使用者] 欄位中看見不尋常的值。這是預期的行為，而且由下列兩個事件的其中一個所造成：

## 服務主體正作用於目錄中，而不需要使用者內容

在此情況下，服務主體 (應用程式) 正在執行目錄更新，而不需實際以使用者身分登入。這會造成服務主體的識別碼顯示為 [執行者]，而不是 UPN。以下是範例：

![](./media/active-directory-reporting-unknown-actor/spd-actor.png)

這是一個已知的 Bug，而我們正在尋求解決方法。

## 處理事件之前已從目錄中刪除使用者

在此案例中，在我們處理事件並將使用者名稱與其產生關聯之前，已從目錄刪除該使用者。以下是範例：

![](./media/active-directory-reporting-unknown-actor/unknown-actor.png)

這是一個已知的 Bug，而我們正在尋求解決方法。

<!-- ![](./media/active-directory-reporting-unknown-actor/uid-actor.png) -->

<!---HONumber=AcomDC_0921_2016-->