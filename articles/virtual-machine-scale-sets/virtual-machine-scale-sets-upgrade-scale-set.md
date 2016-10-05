<properties
	pageTitle="在虛擬機器擴展集上部署應用程式 | Microsoft Azure"
	description="在虛擬機器擴展集上部署應用程式"
	services="virtual-machine-scale-sets"
	documentationCenter=""
	authors="gbowerman"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machine-scale-sets"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/13/2016"
	ms.author="guybo"/>


# 升級虛擬機器擴展集

本文說明如何在不需停機的情況下，對「Azure VM 擴展集」推出 OS 更新。在此背景環境下，OS 更新不是變更 OS 的版本/SKU，就是變更自訂映像的 URI。在不需停機的情況下進行更新意謂著一次更新一部 VM，或依群組 (例如一次一個容錯網域) 更新 VM，而不是全部一起更新。透過這種方式，任何非升級中的 VM 都可繼續執行。

為了避免混淆不清，讓我們區分您可能會想要進行的三種 OS 更新：

1. 變更平台映像的版本或 SKU。例如，將 Ubuntu 14.04.2-LTS 版本從 14.04.201506100 變更為 14.04.201507060，或將 Ubuntu 15.10/最新 SKU 變更為 16.04.0-LTS/最新 SKU。本文涵蓋此案例。

2. 您已建立新版本的自訂映像，而想要變更指向映像的 URI (properties->virtualMachineProfile->storageProfile->osDisk->image->uri)。本文涵蓋此案例。

3. 從 VM 內部修補 OS (例如︰安裝安全性修補程式、使用 Windows Update 等)。支援此案例，但本文並未涵蓋此案例。

前兩種案例是支援的需求。針對第三種案例，至少就現在而言，您必須建立新的擴展集，才能執行該項操作。本文涵蓋選項 1 和 2。注意︰這裡未涵蓋隨 [Service Fabric](https://azure.microsoft.com/services/service-fabric/) 一起部署的「VM 擴展集」。

變更平台映像的 OS 版本/SKU 或變更自訂映像之 URI 的基本順序看起來如下：

* 取得 VMSS 模型。

* 變更模型中的版本、SKU 或 URI 值。

* 更新模型。

* 對擴展集中的 VM 進行 manualUpgrade 呼叫。只有當您「擴展集」的 upgradePolicy 屬性設定為 “Manual” 時，此步驟才相關。如果設定為 “Automatic”，則會同時升級所有 VM，因而造成停機。


將此背景資訊謹記在心，讓我們看看如何在 PowerShell 中使用 REST API 來更新擴展集的版本。這些範例涵蓋平台映像的案例，但是希望我們所提供的資訊已足以讓您將此程序應用在自訂映像上。

## PowerShell

此範例會將「Windows VM 擴展集」更新成新版本 “4.0.20160229”。更新完模型之後，它會一次更新一個 VM 執行個體。

```powershell
$rgname = "myrg"
$vmssname = "myvmss"
$newversion = "4.0.20160229"
$instanceid = "1"

# get the VMSS model
$vmss = Get-AzureRmVmss -ResourceGroupName $rgname -VMScaleSetName $vmssname

# set the new version in the model data
$vmss.virtualMachineProfile.storageProfile.imageReference.version = $newversion

# update the VMSS model
Update-AzureRmVmss -ResourceGroupName $rgname -Name $vmssname -VirtualMachineScaleSet $vmss

# now start updating instances
Update-AzureRmVmssInstance -ResourceGroupName $rgname -VMScaleSetName $vmssname -InstanceId $instanceId
```

如果您更新的是自訂映像的 URI，而不是變更平台映像版本，則會以類似以下的內容取代 “set the new version” 行：

```powershell
# set the new version in the model data
$vmss.virtualMachineProfile.storageProfile.osDisk.image.uri= $newURI
```


## 使用 REST API

以下是幾個使用 Azure REST API 來推出 OS 版本更新的 Python 範例。兩者都使用 Azure REST API 包裝函式的輕量型 [azurerm](https://pypi.python.org/pypi/azurerm) 程式庫對擴展集模型執行 GET，然後對所更新的模型執行 PUT。它們也會查看 VM 執行個體檢視，以依據更新網域識別 VM。

### vmssupgrade

vmssupgrade 是一個 Python 指令碼，能夠一次以一個更新網域為範圍，對執行中的「VM 擴展集」推出 OS 升級。您可以在[這裡](https://github.com/gbowerman/vmsstools)找到它。

![vmssupgrade 螢幕擷取畫面](./media/virtual-machine-scale-sets-upgrade-scale-set/vmssupgrade-screenshot.png)

此指令碼可讓您選擇特定的 VM 來進行更新，或指定某個更新網域，並且支援變更平台映像版本或變更自訂映像的 URI。

### vmsseditor

此工具是「VM 擴展集」的一般用途編輯器，能夠以熱力圖的形式顯示 VM 狀態，其中一個資料列代表一個更新網域。除此之外，您還可以使用新的版本、SKU 或自訂映像 URI 來更新 VMSS 的模型，然後選擇要升級的「容錯網域」。當您這麼做時，該更新網域中的所有 VM 都會升級成新模型。或者，您也可以根據您選擇的批次大小執行輪流升級。您可以在此 [github 儲存機制](https://github.com/gbowerman/vmssdashboard)中找到 vmsseditor

例如，在這裡，我已將擴展集的模型更新成 Ubuntu 14.04-2LTS 版本 14.04.201507060。請注意，此螢幕擷取畫面是舊版畫面；有許多其他選項已經新增到此工具中。

![vmsseditor 螢幕擷取畫面 1](./media/virtual-machine-scale-sets-upgrade-scale-set/vmssEditor1.png)

再次按一下 [升級] 然後按 [取得詳細資料] 之後，UD 0 中的 VM 便會開始更新。

![vmsseditor 螢幕擷取畫面 2](./media/virtual-machine-scale-sets-upgrade-scale-set/vmssEditor2.png)

<!---HONumber=AcomDC_0921_2016-->