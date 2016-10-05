<properties 
	pageTitle="重新部署 Windows 虛擬機器 | Microsoft Azure" 
	description="說明如何重新部署 Windows 虛擬機器，以減輕 RDP 連線問題。" 
	services="virtual-machines-windows" 
	documentationCenter="virtual-machines" 
	authors="iainfoulds" 
	manager="timlt"
	tags="azure-resource-manager,top-support-issue" 
/>
	

<tags 
	ms.service="virtual-machines-windows" 
	ms.devlang="na" 
	ms.topic="support-article" 
	ms.tgt_pltfrm="vm-windows"
	ms.workload="infrastructure" 
	ms.date="09/19/2016" 
	ms.author="iainfou" 
/>


# 將虛擬機器重新部署至新的 Azure 節點

如果您在疑難排解以 Windows 為基礎的 Azure 虛擬機器 (VM) 的遠端桌面 (RDP) 連線或應用程式存取時一直遇到問題，重新部署 VM 也許可以解決。重新部署 VM 時會將 VM 移到 Azure 基礎結構內的新節點，然後重新開啟它的電源，保留所有組態選項和相關聯的資源。本文將說明如何使用 Azure PowerShell 或 Azure 入口網站來重新部署 VM。

> [AZURE.NOTE] 重新部署 VM 之後，暫存磁碟會遺失，而系統會更新與虛擬網路介面關聯的動態 IP 位址。

## 使用 Azure PowerShell

確定您的電腦上已安裝最新版本的 Azure PowerShell 1.x。如需詳細資訊，請參閱[如何安裝和設定 Azure PowerShell](../powershell-install-configure.md)。

使用這個 Azure PowerShell 命令來重新部署您的虛擬機器：

	Set-AzureRmVM -Redeploy -ResourceGroupName $rgname -Name $vmname 


[AZURE.INCLUDE [virtual-machines-common-redeploy-to-new-node](../../includes/virtual-machines-common-redeploy-to-new-node.md)]


## 後續步驟
如果您在連接至 VM 時發生問題，您可以在[針對 RDP 連線進行疑難排解](virtual-machines-windows-troubleshoot-rdp-connection.md)或[詳細的 RDP 疑難排解步驟](virtual-machines-windows-detailed-troubleshoot-rdp.md)中找到具體的說明。如果無法存取在您 VM 上執行的應用程式，您也可以參閱[應用程式疑難排解問題](virtual-machines-windows-troubleshoot-app-connection.md)。

<!---HONumber=AcomDC_0921_2016-->