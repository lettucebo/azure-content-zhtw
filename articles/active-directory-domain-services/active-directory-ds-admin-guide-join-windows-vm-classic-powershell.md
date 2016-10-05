<properties
	pageTitle="Azure Active Directory Domain Services：管理指南 | Microsoft Azure"
	description="使用 Azure PowerShell 與傳統部署模型將 Windows 虛擬機器加入受管理的網域。"
	services="active-directory-ds"
	documentationCenter=""
	authors="mahesh-unnikrishnan"
	manager="stevenpo"
	editor="curtand"/>

<tags
	ms.service="active-directory-ds"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/20/2016"
	ms.author="maheshu"/>


# 使用 PowerShell 將 Windows Server 虛擬機器加入受管理的網域

> [AZURE.SELECTOR]
- [Azure 傳統入口網站 - Windows](active-directory-ds-admin-guide-join-windows-vm.md)
- [PowerShell - Windows](active-directory-ds-admin-guide-join-windows-vm-classic-powershell.md)

<br>

> [AZURE.IMPORTANT] Azure 建立和處理資源的部署模型有二種：[資源管理員和傳統](../resource-manager-deployment-model.md)。本文涵蓋之內容包括使用傳統部署模型。Azure AD 網域服務目前不支援 Resource Manager 模型。

下列步驟將示範如何使用建置組塊自訂一組 Azure PowerShell 命令，建立和預先設定以 Windows 為基礎的 Azure 虛擬機器。這些步驟可協助您建置 Windows 型 Azure 虛擬機器，並將它加入 Azure AD 網域服務受管理的網域。

這些步驟遵循建立 Azure PowerShell 命令集合的填空方法。如果您是 PowerShell 的新手或想知道可指定哪些值來成功設定組態，這個方法相當實用。進階的 PowerShell 使用者可以使用命令並取代本身的變數值 (以「$」為開頭的行)。

如果您尚未這樣做，請按照[如何安裝和設定 Azure PowerShell](../powershell-install-configure.md) 中的操作方法，在本機電腦安裝 Azure PowerShell。然後，開啟 Windows PowerShell 命令提示字元。

## 步驟 1︰加入您的帳戶

1. 在 PowerShell 提示字元處，輸入 **Add-AzureAccount**，然後按一下 **Enter** 鍵。
2. 輸入與您 Azure 訂用帳戶相關聯的電子郵件地址，並按一下 [繼續]。
3. 輸入您帳戶的密碼。
4. 按一下 [**登入**]。

## 步驟 2：設定您的訂用帳戶和儲存體帳戶

在 Windows PowerShell 命令提示字元處執行這些命令，來設定您的 Azure 訂用帳戶與儲存體帳戶。以正確的名稱取代括號中 (包括 < 和 > 字元) 的所有內容。

	$subscr="<subscription name>"
	$staccount="<storage account name>"
	Select-AzureSubscription -SubscriptionName $subscr –Current
	Set-AzureSubscription -SubscriptionName $subscr -CurrentStorageAccountName $staccount

您可以從 **Get-AzureSubscription** 命令輸出的 SubscriptionName 屬性，取得正確的訂用帳戶名稱。執行 **Select-AzureSubscription** 命令後，您就能從 **Get-AzureStorageAccount** 命令輸出的標籤屬性取得正確的儲存體帳戶名稱。


## 步驟 3︰逐步解說 - 佈建虛擬機器並將它加入受管理的網域。
以下是建立這個虛擬機器的相對應 Azure PowerShell 命令集，各個區塊之間均有空白行，以求編排清晰。

指定要佈建的 Windows 虛擬機器相關資訊。

    $family="Windows Server 2012 R2 Datacenter"
    $vmname="Contoso100-test"
    $vmsize="ExtraSmall"

如需 D、DS 或 G 系列虛擬機器的 InstanceSize 值，請參閱 [Azure 的虛擬機器和雲端服務大小](https://msdn.microsoft.com/library/azure/dn197896.aspx)。

提供受管理網域的相關資訊。

    $domaindns="contoso100.com"
    $domacctdomain="contoso100"

指定雲端服務的名稱。

    $svcname="Contoso100-test"

指定 VM 應該加入的虛擬網路名稱。請確定這個虛擬網路中有 AAD-DS 受管理的網域可用。

    $vnetname="MyPreviewVnet"

選取要用來佈建 VM 的 VM 映像。

    $image=Get-AzureVMImage | where { $_.ImageFamily -eq $family } | sort PublishedDate -Descending | select -ExpandProperty ImageName -First 1

設定 VM - 設定 VM 名稱、執行個體大小和要使用的映像。

    $vm1=New-AzureVMConfig -Name $vmname -InstanceSize $vmsize -ImageName $image

取得 VM 的本機系統管理員認證。選擇本機系統管理員的強式密碼。要檢查密碼強度，請參閱[密碼檢查程式：使用強式密碼](https://www.microsoft.com/security/pc-security/password-checker.aspx)。

    $localadmincred=Get-Credential –Message "Type the name and password of the local administrator account."

取得屬於「AAD DC 系統管理員」群組的使用者帳戶認證，將 VM 加入受管理的網域中。不指定網域名稱 - 例如，我們在範例中指定 'bob' 為使用者名稱。

    $domainadmincred=Get-Credential –Message "Now type the name (DO NOT INCLUDE THE DOMAIN) and password of an account in the AAD DC Administrators group, that has permission to add the machine to the domain."

設定 VM - 指定網域聯結需求和所需的認證。

    $vm1 | Add-AzureProvisioningConfig -AdminUsername $localadmincred.Username -Password $localadmincred.GetNetworkCredential().Password -WindowsDomain -Domain $domacctdomain -DomainUserName $domainadmincred.Username -DomainPassword $domainadmincred.GetNetworkCredential().Password -JoinDomain $domaindns

設定 VM 的子網路。

    $vm1 | Set-AzureSubnet -SubnetNames "Subnet-1"

選擇性︰指向網域的 IP 位址。如果 Azure AD 網域服務受管理網域的 IP 位址設定成虛擬網路的 DNS 伺服器，就不需要這個步驟。

    $dns = New-AzureDns -Name 'contoso100-dc1' -IPAddress '10.0.0.4'

現在，佈建已加入網域的 Windows VM。

    New-AzureVM –ServiceName $svcname -VMs $vm1 -VNetName $vnetname -Location "Central US" -DnsSettings $dns

<br>

## 佈建 Windows VM 並自動將它聯結至 AAD 網域服務受管理網域的指令碼
這個 PowerShell 命令集建立企業營運伺服器的虛擬機器：

- 使用 Windows Server 2012 R2 Datacenter 映像。
- 是超小的虛擬機器。
- 具有名稱 contoso-test。
- 會自動加入 contoso100 受管理的網域。
- 和受管理的網域加入相同的虛擬網路。

以下的完整範例指令碼，可建立 Windows 虛擬機器，並自動將它加入 Azure AD 網域服務受管理的網域。

    $family="Windows Server 2012 R2 Datacenter"
    $vmname="Contoso100-test"
    $vmsize="ExtraSmall"

    $domaindns="contoso100.com"
    $domacctdomain="contoso100"

    $svcname="Contoso100-test"
    $vnetname="MyPreviewVnet"

    $image=Get-AzureVMImage | where { $_.ImageFamily -eq $family } | sort PublishedDate -Descending | select -ExpandProperty ImageName -First 1

    $vm1=New-AzureVMConfig -Name $vmname -InstanceSize $vmsize -ImageName $image

    $localadmincred=Get-Credential –Message "Type the name and password of the local administrator account."

    $domainadmincred=Get-Credential –Message "Now type the name (not including the domain) and password of an account in the AAD DC Administrators group, that has permission to add the machine to the domain."

    $vm1 | Add-AzureProvisioningConfig -AdminUsername $localadmincred.Username -Password $localadmincred.GetNetworkCredential().Password -WindowsDomain -Domain $domacctdomain -DomainUserName $domainadmincred.Username -DomainPassword $domainadmincred.GetNetworkCredential().Password -JoinDomain $domaindns

    $vm1 | Set-AzureSubnet -SubnetNames "Subnet-1"

    $dns = New-AzureDns -Name 'contoso100-dc1' -IPAddress '10.0.0.4'

    New-AzureVM –ServiceName $svcname -VMs $vm1 -VNetName $vnetname -Location "Central US" -DnsSettings $dns

<br>

## 相關內容
- [Azure AD 網域服務 - 開始使用](./active-directory-ds-getting-started.md)

- [Administer an Azure AD Domain Services managed domain (管理 Azure AD 網域服務受管理的網域)](./active-directory-ds-admin-guide-administer-domain.md)

<!---HONumber=AcomDC_0921_2016-->