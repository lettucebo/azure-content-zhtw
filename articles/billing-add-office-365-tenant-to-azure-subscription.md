<properties
	pageTitle="搭配使用 Office 365 租用戶與 Azure 訂用帳戶 | Microsoft Azure"
	description="了解如何將 Office 365 目錄 (租用戶) 加入至 Azure 訂用帳戶，以便建立關聯。"
	services=""
	documentationCenter=""
	authors="jiangchen79"
	manager="mbaldwin"
	editor=""
	tags="billing,top-support-issue"/>

<tags
	ms.service="billing"
	ms.workload="na"
	ms.tgt_pltfrm="ibiza"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/16/2016"
	ms.author="cjiang"/>

# 將 Office 365 租用戶與 Azure 訂用帳戶產生關聯
案例︰您在過去分開取得 Azure 和 Office 365 訂用帳戶，而且想要能夠從 Azure 訂用帳戶存取 Office 365 租用戶。本文將說明達成此目的有多麼容易。

> [AZURE.NOTE] 本文不適用於 Enterprise 合約 (EA) 客戶。

## 簡要指引
如果您想要將 Office 365 租用戶與您的 Azure 訂用帳戶建立關聯，請使用您的 Azure 帳戶來新增 Office 365 租用戶，然後將您的 Azure 訂用帳戶與 Office 365 租用戶建立關聯。請參閱詳細步驟。

## 詳細步驟
在此案例中，Kelley Wall 這名使用者的 kelly.wall@outlook.com 帳戶名下已擁有 Azure 訂用帳戶。Kelley 在 kelley.wall@contoso.onmicrosoft.com 帳戶之下也有 Office 365 訂用帳戶。現在 Kelley 想要透過 Azure 訂用帳戶存取 Office 365 租用戶。

![msa-aad-status](./media/billing-add-office-365-tenant-to-azure-subscription/s31_msa-aad-status.png)

![office-365-user](./media/billing-add-office-365-tenant-to-azure-subscription/s32_office-365-user.png)

**必要條件：**

- 您需要 Azure 訂用帳戶的服務管理員認證。共同管理員無法執行一部分的步驟。
- 您需要 Office 365 租用戶的全域系統管理員認證。
- 服務管理員的電子郵件地址不得包含於 Office 365 租用戶中。
- 服務管理員的電子郵件地址不得符合 Office 365 租用戶的任何全域系統管理員。
- 如果您目前使用的電子郵件地址同時為 Microsoft 帳戶和組織帳戶，我們建議您暫時將 Azure 訂用帳戶的服務管理員變更成使用另一個 Microsoft 帳戶。這會限制程序具有的已知限制。您可以在 [Microsoft 帳戶註冊頁面](https://signup.live.com/)建立新的 Microsoft 帳戶。

	若要變更您的服務管理員，請遵循下列步驟：

	1. 登入[帳戶管理入口網站](https://account.windowsazure.com/subscriptions)。
	2. 選取您想變更的訂用帳戶。
	3. 按一下右側的 [編輯訂用帳戶詳細資料]。

		![azure-edit-subscription-details](./media/billing-add-office-365-tenant-to-azure-subscription/s33_azure-edit-subscription-details.png)

	4. 在 [服務管理員] 方塊中，輸入新的服務管理員的電子郵件地址。

		![change-subscription-service-admin](./media/billing-add-office-365-tenant-to-azure-subscription/s34_change-subscription-service-admin.png)

若要將 Office 365 租用戶與 Azure 訂用帳戶產生關聯，請遵循下列步驟︰

1. 	使用服務管理員認證登入[帳戶管理入口網站](https://account.windowsazure.com/subscriptions)。
2.	按一下左窗格上的 [ACTIVE DIRECTORY]。

	![classic-portal-active-directory-entry](./media/billing-add-office-365-tenant-to-azure-subscription/s35-classic-portal-active-directory-entry.png)

	> [AZURE.NOTE] 您應該不會看見 Office 365 租用戶。如果您看見，請略過下一個步驟。

	![aad-tenant-default](./media/billing-add-office-365-tenant-to-azure-subscription/s36-aad-tenant-default.png)

3. 將 Office 365 租用戶加入至您的 Azure 訂用帳戶。
	1. 按一下 [新增] > [目錄] > [自訂建立]。

		![aad-custom-create](./media/billing-add-office-365-tenant-to-azure-subscription/s37-aad-custom-create.png)

	2. 在 [新增目錄] 頁面上，選取 [目錄] 之下的 [使用現有的目錄]，按一下以選取 [我現在已經可以登出]，然後按一下 [完成] ![complete-icon](./media/billing-add-office-365-tenant-to-azure-subscription/s38_complete-icon.png)。

		![add-directory-use-existing](./media/billing-add-office-365-tenant-to-azure-subscription/s39_add-directory-use-existing.png)

	3. 在您登出之後，使用 Office 365 租用戶的全域系統管理員認證登入。

		![sign-in-global-admin-office-365](./media/billing-add-office-365-tenant-to-azure-subscription/s310_sign-in-global-admin-office-365.png)

	4. 按一下 [繼續]。

		![use-contoso-directory-azure-verify](./media/billing-add-office-365-tenant-to-azure-subscription/s311_use-contoso-directory-azure-verify.png)

	5. 按一下 [**立即登出**]。

		![use-contoso-directory-azure-confirm-and-sign-out](./media/billing-add-office-365-tenant-to-azure-subscription/s312_use-contoso-directory-azure-confirm-and-sign-out.png)

	6. 使用服務管理員認證登入[帳戶管理入口網站](https://account.windowsazure.com/subscriptions)。

		![azure-sign-in-service-admin](./media/billing-add-office-365-tenant-to-azure-subscription/s313_azure-sign-in-service-admin.png)

	7. 您應該會在儀表板中看到 Office 365 租用戶。

		![office-365-tenant-appear-in-azure](./media/billing-add-office-365-tenant-to-azure-subscription/s314_office-365-tenant-appear-in-azure.png)

4. 請變更與 Azure 訂用帳戶相關聯的目錄。

	1. 按一下 [設定]。

		![azure-classic-portal-settings-icon](./media/billing-add-office-365-tenant-to-azure-subscription/s315_azure-classic-portal-settings-icon.png)

	2. 按一下您的 Azure 訂用帳戶，然後按一下 [編輯目錄]。

		![azure-subscription-edit-directory](./media/billing-add-office-365-tenant-to-azure-subscription/s316_azure-subscription-edit-directory.png)

	3. 按 [下一步] ![next-icon](./media/billing-add-office-365-tenant-to-azure-subscription/s317_next-icon.png)。

		![azure-change-associated-directory](./media/billing-add-office-365-tenant-to-azure-subscription/s318_azure-change-associated-directory.png)

		> [AZURE.WARNING] 您會收到警告，表示將會移除所有的共同管理員。

		![azure-confirm-directory-mapping](./media/billing-add-office-365-tenant-to-azure-subscription/s322_azure-confirm-directory-mapping.png)

		>[AZURE.WARNING] 此外，在現有資源群組中具有指派存取權的所有[角色型存取控制 (RBAC)](./active-directory/role-based-access-control-configure.md) 使用者也會一併移除。不過，您收到的警告只提及共同管理員的移除。

		![assigned-users-removed-resource-groups](./media/billing-add-office-365-tenant-to-azure-subscription/s325_assigned-users-removed-resource-groups.png)

	4. 按一下 [完成] ![complete-icon](./media/billing-add-office-365-tenant-to-azure-subscription/s38_complete-icon.png)。

5. 您現在可以將您的 Office 365 組織帳戶新增為 AAD 租用戶的共同管理員。

	1. 按一下 [系統管理員] 索引標籤，然後按一下 [新增]。

		![azure-classic-portal-settings-administrators](./media/billing-add-office-365-tenant-to-azure-subscription/s319_azure-classic-portal-settings-administrators.png)

	2. 輸入您的 Office 365 租用戶組織帳戶，按一下以選取 Azure 訂用帳戶，然後按一下 [完成] ![complete-icon](./media/billing-add-office-365-tenant-to-azure-subscription/s38_complete-icon.png)。

		![azure-add-co-administrator](./media/billing-add-office-365-tenant-to-azure-subscription/s320_azure-add-co-administrator.png)

	3. 回到 [系統管理員] 索引標籤，您應該會看到組織帳戶顯示為共同管理員。

		![azure-co-administrator-added](./media/billing-add-office-365-tenant-to-azure-subscription/s321_azure-co-administrator-added.png)

6. 接下來，您可以測試共同管理員的存取。

	1. 登出 [帳戶管理入口網站]。
	2. 開啟[帳戶管理入口網站](https://account.windowsazure.com/subscriptions)或 [Azure 入口網站](https://portal.azure.com/)。
	3. 如果 Azure 登入頁面有 [使用您的組織帳戶登入] 連結，請按一下此連結。否則，請略過此步驟。

		![Azure-sign-in-with-orgid-link](./media/billing-add-office-365-tenant-to-azure-subscription/3-sign-in-to-azure.png)

	4. 輸入共同管理員的認證，然後按一下 [登入]。

		![Azure-sign-in-with-co-admin](./media/billing-add-office-365-tenant-to-azure-subscription/s324_azure-sign-in-with-co-admin.png)

## 後續步驟
相關案例︰您已經擁有 Office 365 訂用帳戶，並且已做好 Azure 訂用帳戶的準備，不過您想要將現有的 Office 365 使用者帳戶用於 Azure 訂用帳戶。或者，您是 Azure 訂閱者，並且想要為現有 Azure Active Directory 中的使用者取得 Office 365 訂用帳戶。若要了解如何完成這些工作，請參閱[搭配使用現有的 Office 365 帳戶與 Azure 訂用帳戶，反之亦然](billing-use-existing-office-365-account-azure-subscription.md)。

<!---HONumber=AcomDC_0921_2016-->