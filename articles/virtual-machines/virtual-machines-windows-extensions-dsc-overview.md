<properties
   pageTitle="Azure 的預期狀態組態概觀 | Microsoft Azure"
   description="將 Microsoft Azure 擴充功能處理常式用於 PowerShell 預期狀態組態的概觀。包括必要條件、架構、Cmdlet..."
   services="virtual-machines-windows"
   documentationCenter=""
   authors="zjalexander"
   manager="timlt"
   editor=""
   tags="azure-service-management,azure-resource-manager"
   keywords=""/>

<tags
   ms.service="virtual-machines-windows"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="vm-windows"
   ms.workload="na"
   ms.date="09/15/2016"
   ms.author="zachal"/>

# Azure 期望狀態組態擴充功能處理常式簡介 #

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-both-include.md)]

Azure VM 代理程式和相關聯的擴充功能是 Microsoft Azure 基礎結構服務的一部分。VM 擴充功能是可擴充 VM 功能及簡化各種 VM 管理作業的軟體元件。例如，VMAccess 擴充功能可用來重設系統管理員的密碼，「自訂指令碼」擴充功能可用來在 VM 上執行指令碼。

本文將介紹隸屬於 Azure PowerShell SDK 的 Azure VM PowerShell 期望狀態組態 (DSC) 擴充功能。透過新的 Cmdlet，您可以將 PowerShell DSC 組態上傳到由 PowerShell DSC 擴充功能啟用的 Azure VM 上，並予以套用。PowerShell DSC 擴充功能會呼叫 PowerShell DSC，以在 VM 上套用所收到的 DSC 組態。您也可以透過 Azure 入口網站取得這項功能。

## 必要條件 ##
**本機電腦**：若要與 Azure VM 擴充功能互動，您需要使用 Azure 入口網站或 Azure PowerShell SDK。

**客體代理程式**：將由 DSC 組態設定的 Azure VM 必須是支援 Windows Management Framework (WMF) 4.0 或 5.0 的 OS。如需所支援作業系統版本的完整清單，請參閱 [DSC 擴充功能版本歷程記錄](https://blogs.msdn.microsoft.com/powershell/2014/11/20/release-history-for-the-azure-dsc-extension/)。

## 詞彙和概念 ##
本指南假設您已熟悉下列概念︰

組態 - DSC 組態文件。

節點 - DSC 組態的目標。在本文件中，「節點」一律是指 Azure VM。

組態資料 - 包含組態之環境資料的 .psd1 檔案

## 架構概觀 ##

Azure DSC 擴充功能會使用「Azure VM 代理程式」架構來傳遞、套用在 Azure VM 上執行的 DSC 組態，並針對這些組態提出報告。DSC 擴充功能需具備一個 .zip 檔案 (其中至少包含一份設定文件)，以及一組透過 Azure PowerShell SDK 或 Azure 入口網站提供的參數。

首次呼叫擴充功能時，它會執行安裝程序。此程序會使用下列邏輯來安裝某個 Windows Management Framework (WMF) 版本︰

1. 如果 Azure VM 作業系統是 Windows Server 2016，則不採取任何動作。Windows Server 2016 已安裝最新版的 PowerShell。
2. 如果已指定 `wmfVersion` 屬性，則除非該 WMF 版本與 VM 的作業系統不相容，否則會逕行安裝。
3. 如果未指定 `wmfVersion` 屬性，則會安裝 WMF 的最新適用版本。

安裝 WMF 需要重新開機。重新開機後，擴充功能會下載 `modulesUrl` 屬性所指定的 .zip 檔案。如果此位置在 Azure Blob 儲存體中，您可以在 `sasToken` 屬性中指定 SAS 權杖來存取檔案。將 .zip 下載並解壓縮之後，系統會執行 `configurationFunction` 中定義的組態函數來產生 MOF 檔案。接著，擴充功能就會在產生的 MOF 檔案上執行 `Start-DscConfiguration -Force`。擴充功能會擷取輸出並寫回 Azure 狀態通道。DSC LCM 會從這裡開始正常處理監視和更正。

## PowerShell Cmdlet ##

PowerShell Cmdlet 可與 ARM 或 ASM 搭配使用，來封裝、發佈和監視 DSC 擴充功能部署。下列 Cmdlet 是 ASM 模組，不過您可以將 "Azure" 取代為 "AzureRm" 來使用 ARM 模型。例如，`Publish-AzureVMDscConfiguration` 會使用 ASM，而 `Publish-AzureRmVMDscConfiguration` 則使用 ARM。

`Publish-AzureVMDscConfiguration` 會接受組態檔、掃描其中是否有相依的 DSC 資源，然後建立包含了組態及套用組態所需之 DSC 資源的 .zip 檔案。它也可以使用 `-ConfigurationArchivePath` 參數在本機建立封裝。否則，它會將 .zip 檔案發佈至 Azure Blob 儲存體，然後以 SAS 權杖加以保護。

對於這個 Cmdlet 所建立的 .zip 檔案，.ps1 組態指令碼位於封存資料夾的根目錄。資源會將模組資料夾放置在封存資料夾中。

`Set-AzureVMDscExtension` 會將 PowerShell DSC 擴充功能所需的設定插入 VM 組態物件中，接著即可使用 `Update-AzureVM` 將此物件套用至 Azure VM。

`Get-AzureVMDscExtension` 會擷取特定 VM 的 DSC 擴充功能狀態。

`Get-AzureVMDscExtensionStatus` 會擷取 DSC 擴充功能處理常式所套用 DSC 組態的狀態。此動作可在單一 VM 或一組 VM 上執行。

`Remove-AzureVMDscExtension` 會從指定的虛擬機器移除擴充功能處理常式。此 Cmdlet「不會」移除組態、將 WMF 解除安裝或變更虛擬機器上已套用的設定。它只會移除擴充功能處理常式。

**ASM 和 ARM Cmdlet 的主要差異**

- ARM Cmdlet 具備同步性；ASM Cmdlet 具備非同步性。
- ResourceGroupName、VMName、ArchiveStorageAccountName、Version 及 Location 皆為新的必要參數。
- ArchiveResourceGroupName 是 ARM 的新選擇性參數。當儲存體帳戶所屬的資源群組與建立虛擬機器的資源群組不同時，您可以指定此參數。
- ConfigurationArchive 在 ARM 中名為 ArchiveBlobName
- ContainerName 在 ARM 中名為 ArchiveContainerName
- StorageEndpointSuffix 在 ARM 中名為 ArchiveStorageEndpointSuffix
- 我們已將 AutoUpdate 參數加入 ARM，讓擴充功能處理常式能在有最新版本時自動更新。請注意，當新版本的 WMF 發行時，此參數有可能會導致 VM 重新啟動。


## Azure 入口網站功能 ##
瀏覽至傳統的 VM。在 [設定] -> [一般] 底下，按一下 [擴充功能]。 系統會建立新窗格。按一下 [加入]，然後選取 [PowerShell DSC]。

入口網站需要輸入。**組態模組或指令碼**︰這是必要欄位。需要一個包含組態指令碼的 .ps1 檔案，或一個 .ps1 組態指令碼位於根目錄而所有相依資源位於模組資料夾內的 .zip 檔案。您可以使用 Azure PowerShell SDK 隨附的 `Publish-AzureVMDscConfiguration -ConfigurationArchivePath` Cmdlet 來建立此檔案。系統會將 .zip 檔案上傳到受 SAS 權杖保護的使用者 Blob 儲存體中。

**組態資料 PSD1 檔案**︰這是選擇性欄位。如果您的組態要求的是在 .psd1 中的組態資料檔，請使用此欄位來選取檔案，然後將它上傳到受 SAS 權杖保護的使用者 Blob 儲存體。
 
**組態的模組完整名稱**：.ps1 檔案可以有多個組態函數。請輸入組態 .ps1 指令碼的名稱，後面加上 '' 和組態函數的名稱。例如，如果 .ps1 指令碼的名稱為 "configuration.ps1"，而組態為 "IisInstall"，您應輸入：`configuration.ps1\IisInstall`

**組態引數**︰如果組態函數接受引數，請以 `argumentName1=value1,argumentName2=value2` 格式在這裡輸入引數。請注意，此格式與透過 PowerShell Cmdlet 或 Resource Manager 範本接受組態引數的方式所採用的格式不同。

## 開始使用 ##

Azure DSC 擴充功能會接受 DSC 組態文件，然後在 Azure VM 上套用這些組態文件。以下是簡單的組態範例。以 "IisInstall.ps1" 的名稱將它儲存在本機：

```powershell
configuration IISInstall 
{ 
    node "localhost"
    { 
        WindowsFeature IIS 
        { 
            Ensure = "Present" 
            Name = "Web-Server"                       
        } 
    } 
}
```

下列步驟會將 IisInstall.ps1 指令碼置於指定的 VM 上、執行組態，然後回報狀態。
 
```powershell
#Azure PowerShell cmdlets are required
Import-Module Azure

#Use an existing Azure Virtual Machine, 'DscDemo1'
$demoVM = Get-AzureVM DscDemo1

#Publish the configuration script into user storage.
Publish-AzureVMDscConfiguration -ConfigurationPath ".\IisInstall.ps1" -StorageContext $storageContext -Verbose -Force

#Set the VM to run the DSC configuration
Set-AzureVMDscExtension -VM $demoVM -ConfigurationArchive "demo.ps1.zip" -StorageContext $storageContext -ConfigurationName "runScript" -Verbose

#Update the configuration of an Azure Virtual Machine
$demoVM | Update-AzureVM -Verbose

#check on status
Get-AzureVMDscExtensionStatus -VM $demovm -Verbose
```

## 記錄 ##

記錄檔位於︰

C:\\WindowsAzure\\Logs\\Plugins\\Microsoft.Powershell.DSC[Version Number]

## 後續步驟 ##

如需有關 PowerShell DSC 的詳細資訊，請[瀏覽 PowerShell 文件中心](https://msdn.microsoft.com/powershell/dsc/overview)。

查看[適用於 DSC 擴充功能的 Azure Resource Manager 範本](virtual-machines-windows-extensions-dsc-template.md)。

若要尋找您可以使用 PowerShell DSC 來管理的其他功能，請[瀏覽 PowerShell 資源庫](https://www.powershellgallery.com/packages?q=DscResource&x=0&y=0)以取得更多 DSC 資源。

如需有關將敏感性參數傳遞到組態中的詳細資訊，請參閱[使用 DSC 擴充功能處理常式安全地管理認證](virtual-machines-windows-extensions-dsc-credentials.md)。

<!---HONumber=AcomDC_0921_2016-->