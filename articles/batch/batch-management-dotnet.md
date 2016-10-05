<properties
	pageTitle="使用 Batch Management .NET 進行帳戶管理 | Microsoft Azure"
	description="使用 Batch Management .NET 程式庫在您的應用程式中建立、刪除和修改 Azure Batch 帳戶。"
	services="batch"
	documentationCenter=".net"
	authors="mmacy"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="batch"
	ms.devlang="multiple"
	ms.topic="article"
	ms.tgt_pltfrm="vm-windows"
	ms.workload="big-compute"
	ms.date="08/03/2016"
	ms.author="marsma"/>

# 使用 Batch Management .NET 管理 Azure Batch 帳戶和配額

> [AZURE.SELECTOR]
- [Azure 入口網站](batch-account-create-portal.md)
- [Batch Management .NET](batch-management-dotnet.md)

您可以透過使用 [Batch Management .NET][api_mgmt_net] 程式庫來自動化 Batch 帳戶的建立、刪除、金鑰管理和配額探索，可降低 Azure Batch 應用程式中的維護負擔。

- 在任何區域內**建立和刪除 Batch 帳戶**。比方說，身為獨立軟體廠商 (ISV)，您為每個獲派不同 Batch 帳戶的客戶針對計費目的提供服務，您可以將帳戶建立和刪除功能加入至客戶入口網站。
- 以程式設計的方式為您的任何 Batch 帳戶**擷取和重新產生帳戶金鑰**。這尤其有利於保持符合可能會對帳戶金鑰強制執行定期變換或過期的安全性原則。當您在各種不同的 Azure 區域中有許多 Batch 帳戶時，將此變換程序自動化將增加解決方案的效率。
- **檢查帳戶配額**並採取反復試驗的猜測，判斷哪一個 Batch 帳戶有哪些限制。藉由開始作業、建立集區或加入計算節點之前檢查您的帳戶配額，您可以主動地調整建立計算資源的位置或時機。您可以決定在帳戶中配置其他資源之前，哪些帳戶需要增加配額。
- 藉由在同一個應用程式中利用 Batch Management .NET、[Azure Active Directory][aad_about] 和 [Azure Resource Manager][resman_overview]，**結合其他 Azure 服務的功能**可獲得完整功能的管理體驗。透過使用這些功能和其 API，您可以提供順暢的驗證體驗、建立和刪除資源群組的能力，以及上面所述的功能，以獲得端對端管理解決方案。

> [AZURE.NOTE] 雖然這篇文章著重在以程式設計方式管理 Batch 帳戶、金鑰和配額，您可以使用 [Azure 入口網站][azure_portal]執行許多活動。如需詳細資訊，請參閱[使用 Azure 入口網站建立 Azure Batch 帳戶](batch-account-create-portal.md)和 [Azure Batch 服務的配額和限制](batch-quota-limit.md)。

## 建立和刪除 Batch 帳戶

如上所述，Batch Management API 的主要功能之一就是在 Azure 區域內建立和刪除 Batch 帳戶。若要這樣做，您將使用 [BatchManagementClient.Account.CreateAsync][net_create] 和 [DeleteAsync][net_delete]，或其同步對應項目。

下列程式碼片段會建立一個帳戶、從 Batch 服務取得新建立的帳戶，然後將它刪除。在此部分與在本文中的其他程式碼片段中，`batchManagementClient` 是 [BatchManagementClient][net_mgmt_client] 完整初始化的執行個體。

```csharp
// Create a new Batch account
await batchManagementClient.Account.CreateAsync("MyResourceGroup",
	"mynewaccount",
	new BatchAccountCreateParameters() { Location = "West US" });

// Get the new account from the Batch service
AccountResource account = await batchManagementClient.Account.GetAsync(
	"MyResourceGroup",
	"mynewaccount");

// Delete the account
await batchManagementClient.Account.DeleteAsync("MyResourceGroup", account.Name);
```

> [AZURE.NOTE] 使用 Batch Management .NET 程式庫和其 BatchManagementClient 類別的應用程式需要**服務管理員**或**共同管理員**存取權，以使用擁有要管理的 Batch 帳戶的訂用帳戶。如需詳細資訊，請參閱以下的 [Azure Active Directory](#azure-active-directory) 一節和 [AccountManagement][acct_mgmt_sample] 程式碼範例。

## 擷取和重新產生帳戶金鑰

使用 [ListKeysAsync][net_list_keys] 從您的訂用帳戶內的任何 Batch 帳戶取得主要和次要帳戶金鑰。您可以使用 [RegenerateKeyAsync][net_regenerate_keys] 重新產生這些金鑰。

```csharp
// Get and print the primary and secondary keys
BatchAccountListKeyResult accountKeys =
	await batchManagementClient.Account.ListKeysAsync(
		"MyResourceGroup",
		"mybatchaccount");
Console.WriteLine("Primary key:   {0}", accountKeys.Primary);
Console.WriteLine("Secondary key: {0}", accountKeys.Secondary);

// Regenerate the primary key
BatchAccountRegenerateKeyResponse newKeys =
	await batchManagementClient.Account.RegenerateKeyAsync(
		"MyResourceGroup",
		"mybatchaccount",
		new BatchAccountRegenerateKeyParameters() {
			KeyName = AccountKeyType.Primary
			});
```

> [AZURE.TIP] 您可以為您的管理應用程式建立簡化的連線工作流程。首先，取得您想要使用 [ListKeysAsync][net_list_keys] 管理的 Batch 帳戶的帳戶金鑰。接著，在初始化 Batch .NET 程式庫的 [BatchSharedKeyCredentials][net_sharedkeycred] 類別 (初始化時 [BatchClient][net_batch_client] 時使用) 時使用此金鑰。

## 檢查 Azure 訂用帳戶和 Batch 帳戶配額

Azure 訂用帳戶和 Batch 之類的個別 Azure 服務均具有預設配額，限制其中特定實體的數目。如需 Azure 訂用帳戶的預設配額，請參閱 [Azure 訂用帳戶和服務限制、配額及條件約束](./../azure-subscription-service-limits.md)。如需 Batch 服務的預設配額，請參閱 [Azure Batch 服務的配額和限制](batch-quota-limit.md)。藉由使用 Batch Management .NET 程式庫，您可以檢查您的應用程式中的這些配額。這可讓您在加入帳戶或計算資源 (如集區和計算節點) 之前進行配置決策。

### 檢查 Azure 訂用帳戶和 Batch 帳戶配額

在區域中建立 Batch 帳戶之前，您可以檢查您的 Azure 訂用帳戶，以查看您是否能夠將帳戶加入該區域中。

在以下的程式碼片段中，我們先使用 [BatchManagementClient.Account.ListAsync][net_mgmt_listaccounts] 來取得訂用帳戶內所有 Batch 帳戶的集合。在我們取得此集合後，我們會判斷目標區域中有多少個帳戶。接著，我們使用 [BatchManagementClient.Subscriptions][net_mgmt_subscriptions] 來取得 Batch 帳戶配額，並判斷該區域中可以建立多少個帳戶 (如果有的話)。

```csharp
// Get a collection of all Batch accounts within the subscription
BatchAccountListResponse listResponse =
		await batchManagementClient.Account.ListAsync(new AccountListParameters());
IList<AccountResource> accounts = listResponse.Accounts;
Console.WriteLine("Total number of Batch accounts under subscription id {0}:  {1}",
	creds.SubscriptionId,
	accounts.Count);

// Get a count of all accounts within the target region
string region = "westus";
int accountsInRegion = accounts.Count(o => o.Location == region);

// Get the account quota for the specified region
SubscriptionQuotasGetResponse quotaResponse = await batchManagementClient.Subscriptions.GetSubscriptionQuotasAsync(region);
Console.WriteLine("Account quota for {0} region: {1}", region, quotaResponse.AccountQuota);

// Determine how many accounts can be created in the target region
Console.WriteLine("Accounts in {0}: {1}", region, accountsInRegion);
Console.WriteLine("You can create {0} accounts in the {1} region.", quotaResponse.AccountQuota - accountsInRegion, region);
```

在上述程式碼片段中，`creds` 是 [TokenCloudCredentials][azure_tokencreds] 的執行個體。若要查看建立此物件的範例，請參閱 GitHub 上的 [AccountManagement][acct_mgmt_sample] 程式碼範例。

### 檢查 Batch 帳戶的計算資源配額

在增加 Batch 解決方案內的計算資源之前，您可以檢查以確定您想要配置的資源不會超出目前既有的帳戶配額。在下列程式碼片段中，我們只要列印名為`mybatchaccount` 的 Batch 帳戶的配額資訊。但在您自己的應用程式中，您可以使用這類資訊來判斷帳戶是否可以處理您想要建立的其他資源。

```csharp
// First obtain the Batch account
BatchAccountGetResponse getResponse =
	await batchManagementClient.Account.GetAsync("MyResourceGroup", "mybatchaccount");
AccountResource account = getResponse.Resource;

// Now print the compute resource quotas for the account
Console.WriteLine("Core quota: {0}", account.Properties.CoreQuota);
Console.WriteLine("Pool quota: {0}", account.Properties.PoolQuota);
Console.WriteLine("Active job and job schedule quota: {0}", account.Properties.ActiveJobAndJobScheduleQuota);
```

> [AZURE.IMPORTANT] 雖然 Azure 訂用帳戶和服務有預設配額，這許多限制都可以透過在 [Azure 入口網站][azure_portal]中提出要求來提高。例如，請參閱 [Azure Batch 服務的配額和限制](batch-quota-limit.md)以取得增加您的 Batch 帳戶配額的指示。

## Batch Management .NET、Azure AD 和資源管理員

使用 Batch Management .NET 程式庫時，您通常會利用 [Azure Active Directory][aad_about] \(Azure AD) 和 [Azure Resource Manager][resman_overview] 兩者的功能。下面討論的範例專案，在示範 Batch Management .NET API 時同時運用 Azure Active Directory 和資源管理員。

### Azure Active Directory

Azure 本身會使用 Azure AD 來驗證其客戶、服務管理員和組織的使用者。在 Batch Management .NET 內容中，您將使用它來驗證的訂用帳戶管理員或共同管理員。如此將允許管理程式庫查詢 Batch 服務並執行本文中所述的作業。

在以下討論的範例專案中，Azure [Active Directory 驗證程式庫][aad_adal] \(ADAL) 用來提示使用者輸入他們的 Microsoft 認證。提供服務管理員或共同管理員認證時，讓應用程式查詢 Azure 訂用帳戶的清單，以及建立和刪除資源群組和 Batch 帳戶。

### Resource Manager

使用 Batch Management .NET 程式庫建立 Batch 帳戶時，您通常會在[資源群組][resman_overview]內建立帳戶。您可以使用[資源管理員.NET][resman_api] 程式庫中的 [ResourceManagementClient][resman_client] 類別，以程式設計方式建立資源群組。或您可以將帳戶加入至您先前使用 [Azure 入口網站][azure_portal]建立的現有資源群組。

## GitHub 上的範例專案

查看 GitHub 上的 [AccountManagment][acct_mgmt_sample] 範例專案，以查看 Batch Management .NET 程式庫實務。此主控台應用程式顯示 [BatchManagementClient][net_mgmt_client] 和 [ResourceManagementClient][resman_client] 的建立與使用方式。它也會示範兩個用戶端所需的 Azure [Active Directory 驗證程式庫][aad_adal] \(ADAL) 使用方式。

若要成功執行範例應用程式，您必須先使用 Azure 入口網站向 Azure AD 登錄它。遵循[整合應用程式與 Azure Active Directory](../active-directory/active-directory-integrating-applications.md#adding-an-application)之[加入應用程式][aad_integrate]一節中的步驟，在您自己帳戶的預設目錄內註冊範例應用程式。請務必針對應用程式的類型選取**原生用戶端應用程式**，且您可以針對**重新導向 URI**指定任何有效的 URI (例如 `http://myaccountmanagementsample`) - 不需要是實際的端點。

加入您的應用程式之後，在入口網站的應用程式設定中，將**以組織身分存取 Azure 服務管理**權限委派給 *Windows Azure 服務管理 API* 應用程式：

![Azure 入口網站的應用程式權限][2]

> [AZURE.TIP] 如果 [其他應用程式的權限] 下沒有顯示 **Windows Azure 服務管理 API**，請按一下 [加入應用程式]，選取 [Windows Azure 服務管理 API]，然後按一下核取記號按鈕。接著，依上文指定方式委派權限。

一旦您已經如上面所述加入應用程式，請使用您的應用程式的重新導向 URI 和用戶端 ID，更新 [AccountManagment][acct_mgmt_sample] 範例專案中的 `Program.cs`。您可以在您的應用程式的 [設定] 索引標籤中找到這些值：

![Azure 入口網站的應用程式組態][3]

[AccountManagment][acct_mgmt_sample] 範例應用程式會示範下列作業：

1. 使用 [ADAL][aad_adal] 向 Azure AD 取得安全性權杖。如果使用者尚未登入，將提示使用者輸入其 Azure 認證。
2. 使用從 Azure AD 取得的安全性權杖，建立 [SubscriptionClient][resman_subclient] 以查詢 Azure 與帳戶相關聯的訂用帳戶清單。這可讓使用者選取一個訂用帳戶 (如果找到多個)。
3. 建立與選取的訂用帳戶相關聯的新認證物件。
4. 使用新認證來建立 [ResourceManagementClient][resman_client]。
5. 使用 [ResourceManagementClient][resman_client] 來建立新資源群組。
6. 使用 [BatchManagementClient][net_mgmt_client] 可執行許多 Batch 帳戶作業：
  - 在新建立的資源群組中建立新 Batch 帳戶。
  - 從 Batch 服務取得新建立的帳戶。
  - 列印新帳戶的帳戶金鑰。
  - 重新產生帳戶的新主要金鑰。
  - 列印帳戶的配額資訊。
  - 列印訂用帳戶的配額資訊。
  - 列印訂用帳戶內的所有帳戶。
  - 刪除新建立的帳戶。
7. 刪除資源群組。

刪除新建立的 Batch 帳戶和資源群組之前，您可以在 [Azure 入口網站][azure_portal]中檢查這兩者：

![顯示資源群組和 Batch 帳戶的 Azure 入口網站][1] <br /> *顯示新的資源群組和 Batch 帳戶的 Azure 入口網站*

[aad_about]: ../active-directory/active-directory-whatis.md "什麼是 Azure Active Directory？"
[aad_adal]: ../active-directory/active-directory-authentication-libraries.md
[aad_auth_scenarios]: ../active-directory/active-directory-authentication-scenarios.md "Azure AD 的驗證案例"
[aad_integrate]: ../active-directory/active-directory-integrating-applications.md "整合應用程式與 Azure Active Directory"
[acct_mgmt_sample]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/AccountManagement
[api_net]: http://msdn.microsoft.com/library/azure/mt348682.aspx
[api_mgmt_net]: https://msdn.microsoft.com/library/azure/mt463120.aspx
[azure_portal]: http://portal.azure.com
[azure_storage]: https://azure.microsoft.com/services/storage/
[azure_tokencreds]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.tokencloudcredentials.aspx
[batch_explorer_project]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/BatchExplorer
[net_batch_client]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.batchclient.aspx
[net_list_keys]: https://msdn.microsoft.com/library/azure/microsoft.azure.management.batch.accountoperationsextensions.listkeysasync.aspx
[net_create]: https://msdn.microsoft.com/library/azure/microsoft.azure.management.batch.accountoperationsextensions.createasync.aspx
[net_delete]: https://msdn.microsoft.com/library/azure/microsoft.azure.management.batch.accountoperationsextensions.deleteasync.aspx
[net_regenerate_keys]: https://msdn.microsoft.com/library/azure/microsoft.azure.management.batch.accountoperationsextensions.regeneratekeyasync.aspx
[net_sharedkeycred]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.auth.batchsharedkeycredentials.aspx
[net_mgmt_client]: https://msdn.microsoft.com/library/azure/microsoft.azure.management.batch.batchmanagementclient.aspx
[net_mgmt_subscriptions]: https://msdn.microsoft.com/library/azure/microsoft.azure.management.batch.batchmanagementclient.subscriptions.aspx
[net_mgmt_listaccounts]: https://msdn.microsoft.com/library/azure/microsoft.azure.management.batch.iaccountoperations.listasync.aspx
[resman_api]: https://msdn.microsoft.com/library/azure/mt418626.aspx
[resman_client]: https://msdn.microsoft.com/library/azure/microsoft.azure.management.resources.resourcemanagementclient.aspx
[resman_subclient]: https://msdn.microsoft.com/library/azure/microsoft.azure.subscriptions.subscriptionclient.aspx
[resman_overview]: ../resource-group-overview.md

[1]: ./media/batch-management-dotnet/portal-01.png
[2]: ./media/batch-management-dotnet/portal-02.png
[3]: ./media/batch-management-dotnet/portal-03.png

<!---HONumber=AcomDC_0831_2016-->