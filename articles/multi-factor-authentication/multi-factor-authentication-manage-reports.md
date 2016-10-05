<properties 
	pageTitle="Azure Multi-Factor Authentication 報告"
	description="說明如何使用 Azure Multi-Factor Authentication 功能 - 報告。"
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
	ms.topic="article"
	ms.date="08/04/2016"
	ms.author="kgremban"/>

# Azure Multi-Factor Authentication 中的報告

Azure Multi-Factor Authentication 提供數個供您和貴組織使用的報告。這些報告可以透過 Multi-Factor Authentication 管理入口網站存取，而且您必須擁有 Azure MFA 提供者，或是 Azure MFA、Azure AD Premium 或 Enterprise Mobility Suite 授權。以下是可用的報告清單。

您可以透過 Azure 管理入口網站來存取報告。

Name| 說明
:------------- | :------------- |
使用量 | 使用量報告顯示有關整體使用量、使用者摘要和使用者詳細資料等資訊。
伺服器狀態|此報告會顯示與帳戶相關聯之 Multi-Factor Authentication Server 的狀態。
已封鎖的使用者歷程記錄|這些報告會顯示要求封鎖或解除封鎖使用者之申請的歷程記錄。
已略過的使用者歷程記錄|顯示要求略過使用者電話號碼之 Multi-Factor Authentication 的申請歷程記錄。
詐騙警示|顯示在指定日期範圍內提交之詐騙警示的歷程記錄。
已排入佇列|列出已排入佇列並等候處理的報告和其狀態。當報告完成時，系統會提供下載或檢視報告的連結。

## 檢視報告

1.	登入 http://azure.microsoft.com
2.	在左側選取 [Active Directory]。
3.	選取下列其中一個選項：
	- **選項 1**：按一下 [多因素驗證提供者] 索引標籤。選取您的 MFA 提供者，然後按一下底部的 [管理] 按鈕。
	- **選項 2**：選取您的目錄，然後按一下 [設定] 索引標籤。在 [Multi-Factor Authentication] 區段底下，選取 [管理服務設定]。在 [MFA 服務設定] 頁面底部，按一下 [移至入口網站] 連結。
4.	在 Azure Multi-Factor Authentication 管理入口網站的左側導覽中，您將會看到 [檢視報告] 區段。在這裡，您可以選取前文所述的報告。

<center>![Cloud](./media/multi-factor-authentication-manage-reports/report.png)</center>


**其他資源**

* [適用於使用者](multi-factor-authentication-end-user.md)
* [MSDN 上的 Azure Multi-Factor Authentication](https://msdn.microsoft.com/library/azure/dn249471.aspx)

<!---HONumber=AcomDC_0921_2016-->