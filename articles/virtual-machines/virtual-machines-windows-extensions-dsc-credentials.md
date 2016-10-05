<properties
   pageTitle="使用 DSC 將認證傳遞至 Azure |Microsoft Azure"
   description="使用 PowerShell 期望的狀態組態將認證安全地傳遞到 Azure 虛擬機器的概觀"
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

# 將認證傳遞至 Azure DSC 延伸模組處理常式 #

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-both-include.md)]

本文涵蓋適用於 Azure 的期望的狀態組態延伸模組。如需 DSC 延伸模組處理常式的概觀，請參閱 [Azure 期望的狀態組態延伸模組處理常式簡介](virtual-machines-windows-extensions-dsc-overview.md)。


## 傳入認證
設定過程中，您可能需要在使用者內容中設定使用者帳戶、存取服務，或安裝程式。若要執行這些動作，您需要提供認證。

DSC 允許使用參數化的組態，其中會將認證傳入組態並安全地儲存在 MOF 檔案中。Azure 延伸模組處理常式會提供憑證的自動管理功能，藉以簡化認證的管理。

請考慮使用下列 DSC 組態指令碼，此指令碼會建立包含指定的密碼的本機使用者帳戶︰

*user\_configuration.ps1*

```
configuration Main
{
    param(
        [Parameter(Mandatory=$true)]
        [ValidateNotNullorEmpty()]
        [PSCredential]
        $Credential
    )    
    Node localhost {       
        User LocalUserAccount
        {
            Username = $Credential.UserName
            Password = $Credential
            Disabled = $false
            Ensure = "Present"
            FullName = "Local User Account"
            Description = "Local User Account"
            PasswordNeverExpires = $true
        } 
    }  
} 
```

請務必將 *node localhost* 納入為組態的一部分。如果遺漏此陳述式，接下來的步驟將無法運作，因為延伸模組處理常式會特別尋找節點 localhost 陳述式。也請務必納入 typecast *[PsCredential]*，因為這個特定的類型會觸發擴充功能將認證加密。

將此指令碼發佈至 Blob 儲存體︰

`Publish-AzureVMDscConfiguration -ConfigurationPath .\user_configuration.ps1`

設定 Azure DSC 延伸模組，並提供認證︰

```
$configurationName = "Main"
$configurationArguments = @{ Credential = Get-Credential }
$configurationArchive = "user_configuration.ps1.zip"
$vm = Get-AzureVM "example-1"
 
$vm = Set-AzureVMDSCExtension -VM $vm -ConfigurationArchive $configurationArchive 
-ConfigurationName $configurationName -ConfigurationArgument @configurationArguments
 
$vm | Update-AzureVM
```
## 保護認證的方式
執行此程式碼會提示您輸入認證。一旦提供認證之後，就會很快地儲存在記憶體中。使用 `Set-AzureVmDscExtension` Cmdlet 發佈認證時，會透過 HTTPS 傳輸到 VM，Azure 就會使用本機 VM 憑證，以加密的形式將該認證儲存在 VM 的磁碟上。接著，它會在記憶體中短暫地解密後再重新加密，以便傳遞給 DSC。

此行為與[使用不含擴充功能處理常式的安全組態](https://msdn.microsoft.com/powershell/dsc/securemof)不同。Azure 環境提供一個透過憑證來安全地傳輸組態資料的方式。使用 DSC 擴充功能處理常式時，不需要在 ConfigurationData 中提供 $CertificatePath 或 $CertificateID / $Thumbprint 項目。


## 後續步驟 ##

如需有關 Azure DSC 擴充功能處理常式的詳細資訊，請參閱 [Azure 期望狀態組態擴充功能處理常式簡介](virtual-machines-windows-extensions-dsc-overview.md)。

查看[適用於 DSC 擴充功能的 Azure Resource Manager 範本](virtual-machines-windows-extensions-dsc-template.md)。

如需有關 PowerShell DSC 的詳細資訊，請[瀏覽 PowerShell 文件中心](https://msdn.microsoft.com/powershell/dsc/overview)。

若要尋找您可以使用 PowerShell DSC 來管理的其他功能，請[瀏覽 PowerShell 資源庫](https://www.powershellgallery.com/packages?q=DscResource&x=0&y=0)以取得更多 DSC 資源。

<!---HONumber=AcomDC_0921_2016-->