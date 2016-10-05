<properties
	pageTitle="使用 PowerShell 移轉到 Resource Manager | Microsoft Azure"
	description="本文提供使用 Azure PowerShell 命令進行平台支援之 IaaS 資源移轉 (從傳統移轉至 Azure Resource Manager) 的逐步解說"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="dlepow"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/16/2016"
	ms.author="danlep"/>

# 使用 Azure PowerShell 將 IaaS 資源從傳統移轉至 Azure Resource Manager

以下步驟說明如何使用 Azure PowerShell 命令，將基礎結構即服務 (IaaS) 資源從傳統部署模型移轉至 Azure Resource Manager 部署模型。這些步驟遵循填空方法來移轉您的自訂環境。採用命令並將變數 (開頭為 "$" 的行) 取代為您自己的值即可。

如果您想要的話，也可以使用 [Azure 命令列介面 (Azure CLI)](virtual-machines-linux-cli-migration-classic-resource-manager.md) 來移轉資源。

如需了解有關支援之移轉案例的背景，請參閱[平台支援的 IaaS 資源移轉 (從傳統移轉至 Azure Resource Manager)](virtual-machines-windows-migration-classic-resource-manager.md)。如需詳細的指導方針和移轉逐步解說，請參閱[平台支援的從傳統移轉至 Azure Resource Manager 的技術深入探討](virtual-machines-windows-migration-classic-resource-manager-deep-dive.md)。

## 步驟 1︰為移轉做規劃

以下是您評估將 IaaS 資源從傳統移轉至 Resource Manager 時，我們所建議的一些最佳做法：

- 將[支援及不支援的功能和組態](virtual-machines-windows-migration-classic-resource-manager.md)看一遍。如果您的虛擬機器使用不支援的組態或功能，建議您等到宣布支援該組態/功能之後，再進行移轉。或者，移除該功能或移出該組態以利移轉進行 (如果這麼做符合您的需求)。
-	如果您是使用自動化指令碼來部署現今的基礎結構和應用程式，請使用這些指令碼來嘗試建立相似的測試設定以進行移轉。或者，您也可以使用 Azure 入口網站來設定範例環境。

>[AZURE.IMPORTANT]目前不支援使用虛擬網路閘道 (站對站、Azure ExpressRoute、應用程式閘道、點對站) 從傳統環境移轉至 Resource Manager。若要使用閘道來移轉傳統虛擬網路，請先移除閘道，再執行「認可」作業來移動網路。(您可以執行「準備」步驟而不刪除閘道)。 然後，在完成移轉之後，於 Azure Resource Manager 中重新連接閘道。

## 步驟 2：安裝最新版的 Azure PowerShell

Azure PowerShell 的主要安裝選項有兩個：[PowerShell 資源庫](https://www.powershellgallery.com/profiles/azure-sdk/)和 [Web Platform Installer (WebPI)](http://aka.ms/webpi-azps)。WebPI 接收每月更新。PowerShell 資源庫則是持續接收更新。本文是以 Azure PowerShell 2.1.0 為基礎。

如需安裝指示，請參閱[如何安裝和設定 Azure PowerShell](../powershell-install-configure.md)。

## 步驟 3︰設定您的訂用帳戶並註冊以進行移轉

首先，開啟 PowerShell 提示字元。針對移轉案例，您必須為傳統和 Resource Manager 模型設定您的環境。

登入您的 Resource Manager 模型帳戶。

	Login-AzureRmAccount

請使用下列命令來取得可用的訂用帳戶：

	Get-AzureRMSubscription | Sort SubscriptionName | Select SubscriptionName

設定目前工作階段的 Azure 訂用帳戶。以正確的名稱取代括號中 (包括 < 和 > 字元) 的所有內容。

	Select-AzureRmSubscription –SubscriptionName "<subscription name>"

>[AZURE.NOTE] 註冊是一次性步驟，但您必須在嘗試移轉之前完成。如果不註冊，您會看到下列錯誤訊息：

>	*BadRequest : Subscription is not registered for migration.* 

請使用下列命令向移轉資源提供者註冊：
	
	Register-AzureRmResourceProvider -ProviderNamespace Microsoft.ClassicInfrastructureMigrate

請等候 5 分鐘讓註冊完成。您可以使用下列命令來檢查核准狀態：

	Get-AzureRmResourceProvider -ProviderNamespace Microsoft.ClassicInfrastructureMigrate

請先確定 RegistrationState 是 `Registered`，再繼續進行。

現在，登入您的傳統模型帳戶。

	Add-AzureAccount

請使用下列命令來取得可用的訂用帳戶：

	Get-AzureSubscription | Sort SubscriptionName | Select SubscriptionName

設定目前工作階段的 Azure 訂用帳戶。以正確的名稱取代括號中 (包括 < 和 > 字元) 的所有內容。

	Select-AzureSubscription –SubscriptionName "<subscription name>"

## 步驟 4︰確定您目前的部署或 VNET 的 Azure 區域中有足夠的 Azure Resource Manager 虛擬機器核心

您可以使用下列 PowerShell 命令來檢查您目前在 Azure Resource Manager 中擁有的核心數目。若要深入了解核心配額，請參閱[限制和 Azure Resource Manager](../azure-subscription-service-limits.md#limits-and-the-azure-resource-manager)。

```
Get-AzureRmVMUsage -Location "<Your VNET or Deployment's Azure region"
```

## 步驟 5︰執行命令來移轉 IaaS 資源

>[AZURE.NOTE] 下述所有作業都是等冪的。如果您有不支援的功能或組態錯誤以外的任何問題，建議您重新嘗試準備、中止或認可作業。平台將會重新嘗試該動作。

### 移轉雲端服務中的虛擬機器 (不在虛擬網路中)

使用下列命令來取得雲端服務清單，然後選擇您想要移轉的雲端服務。如果雲端服務中的 VM 是在虛擬網路中，或是具有 Web 角色或背景工作角色，命令就會傳回錯誤訊息。

	Get-AzureService | ft Servicename

請使用下列命令來取得雲端服務的部署名稱：

	$serviceName = "<service name>"
	$deployment = Get-AzureDeployment -ServiceName $serviceName
	$deploymentName = $deployment.DeploymentName

準備好雲端服務中的虛擬機器以進行移轉。有兩個選項可供您選擇。

* **選項 1：將 VM 移轉到平台建立的虛擬網路**

	首先，請使用下列命令來驗證您是否可以移轉雲端服務︰

		$validate = Move-AzureService -Validate -ServiceName $serviceName -DeploymentName $deploymentName -CreateNewVirtualNetwork
		$validate.ValidationMessages

	上述命令會顯示封鎖移轉的任何警告及錯誤。如果驗證成功，您便可以繼續進行下列「準備」步驟：

		Move-AzureService -Prepare -ServiceName $serviceName -DeploymentName $deploymentName -CreateNewVirtualNetwork

* **選項 2：移轉到 Resource Manager 部署模型中的現有虛擬網路**

		$existingVnetRGName = "<Existing VNET's Resource Group Name>"
		$vnetName = "<Virtual Network Name>"
		$subnetName = "<Subnet name>"

	首先，請使用下列命令來驗證您是否可以移轉雲端服務︰

		$validate = Move-AzureService -Validate -ServiceName $serviceName -DeploymentName $deploymentName -UseExistingVirtualNetwork -VirtualNetworkResourceGroupName $existingVnetRGName -VirtualNetworkName $vnetName -SubnetName $subnetName
		$validate.ValidationMessages

	上述命令會顯示封鎖移轉的任何警告及錯誤。如果驗證成功，您便可以繼續進行下列「準備」步驟：

		Move-AzureService -Prepare -ServiceName $serviceName -DeploymentName $deploymentName -UseExistingVirtualNetwork -VirtualNetworkResourceGroupName $existingVnetRGName -VirtualNetworkName $vnetName -SubnetName $subnetName

上述其中一個選項的「準備」作業成功之後，請查詢 VM 的移轉狀態。確定它們是處於 `Prepared` 狀態。

	$vmName = "<vm-name>"
	$vm = Get-AzureVM -ServiceName $serviceName -Name $vmName
	$migrationState = $vm.VM.MigrationState

使用 PowerShell 或 Azure 入口網站來檢查已備妥之資源的組態。如果您尚未準備好進行移轉，而想要回到舊狀態，請使用下列命令：

	Move-AzureService -Abort -ServiceName $serviceName -DeploymentName $deploymentName

如果備妥的組態看起來沒問題，您就可以繼續進行並使用下列命令來認可資源：

	Move-AzureService -Commit -ServiceName $serviceName -DeploymentName $deploymentName

### 移轉虛擬網路中的虛擬機器

若要移轉虛擬網路中的虛擬機器，您將需要移轉網路。虛擬機器會自動隨著網路移轉。選取您想要移轉的虛擬網路。

	$vnetName = "<Virtual Network Name>"

>[AZURE.NOTE] 如果虛擬網路包含 Web 角色或背景工作角色，或有具備不支援之組態的 VM，您就會收到驗證錯誤訊息。

首先，請使用下列命令來驗證您是否可以移轉虛擬網路︰

	Move-AzureVirtualNetwork -Validate -VirtualNetworkName $vnetName

上述命令會顯示封鎖移轉的任何警告及錯誤。如果驗證成功，您便可以繼續進行下列「準備」步驟：
	
	Move-AzureVirtualNetwork -Prepare -VirtualNetworkName $vnetName

請使用 Azure PowerShell 或 Azure 入口網站來檢查已備妥之虛擬機器的組態。如果您尚未準備好進行移轉，而想要回到舊狀態，請使用下列命令：

	Move-AzureVirtualNetwork -Abort -VirtualNetworkName $vnetName

如果備妥的組態看起來沒問題，您就可以繼續進行並使用下列命令來認可資源：

	Move-AzureVirtualNetwork -Commit -VirtualNetworkName $vnetName

### 移轉儲存體帳戶

完成虛擬機器的移轉之後，建議您移轉儲存體帳戶。

請使用下列命令來準備每個要移轉的儲存體帳戶：

	$storageAccountName = "<storage account name>"
	Move-AzureStorageAccount -Prepare -StorageAccountName $storageAccountName

請使用 Azure PowerShell 或 Azure 入口網站來檢查已備妥之儲存體帳戶的組態。如果您尚未準備好進行移轉，而想要回到舊狀態，請使用下列命令：

	Move-AzureStorageAccount -Abort -StorageAccountName $storageAccountName

如果備妥的組態看起來沒問題，您就可以繼續進行並使用下列命令來認可資源：

	Move-AzureStorageAccount -Commit -StorageAccountName $storageAccountName

## 後續步驟

- 如需有關移轉的詳細資訊，請參閱[平台支援的 IaaS 資源移轉 (從傳統移轉至 Azure Resource Manager)](virtual-machines-windows-migration-classic-resource-manager.md)。
- 若要使用 Azure PowerShell 將其他網路資源移轉至 Resource Manager，請使用與 **Move-AzureNetworkSecurityGroup**、**Move-AzureReservedIP** 及 **Move-AzureRouteTable** 類似的步驟。
- 如需您可用來將 Azure 資源從傳統環境移轉至 Resource Manager 的開放原始碼指令碼，請參閱[可移轉至 Azure Resource Manager 的社群工具](virtual-machines-windows-migration-scripts.md)

<!---HONumber=AcomDC_0921_2016-->