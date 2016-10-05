<properties 
	pageTitle="重新部署 Linux 虛擬機器 | Microsoft Azure" 
	description="說明如何重新部署 Linux 虛擬機器，以減輕 SSH 連線問題。" 
	services="virtual-machines-linux" 
	documentationCenter="virtual-machines" 
	authors="iainfoulds" 
	manager="timlt"
	tags="azure-resource-manager,top-support-issue" 
/>
	

<tags 
	ms.service="virtual-machines-linux" 
	ms.devlang="na" 
	ms.topic="support-article" 
	ms.tgt_pltfrm="vm-linux"
	ms.workload="infrastructure" 
	ms.date="09/19/2016" 
	ms.author="iainfou" 
/>

# 將虛擬機器重新部署至新的 Azure 節點

如果您在疑難排解 Azure 虛擬機器 (VM) 的 SSH 或應用程式存取時一直遇到問題，重新部署 VM 也許可以解決。重新部署 VM 時會將 VM 移到 Azure 基礎結構內的新節點，然後重新開啟它的電源，保留所有組態選項和相關聯的資源。本文將說明如何使用 Azure CLI 或 Azure 入口網站來重新部署 VM。

> [AZURE.NOTE] 重新部署 VM 之後，暫存磁碟會遺失，而系統會更新與虛擬網路介面關聯的動態 IP 位址。


## 使用 Azure CLI

請確定您的機器上已安裝[最新版本的 Azure CLI](../xplat-cli-install.md)，而且您已處於 Resource Manager 模式 (`azure config mode arm`)。

使用下列 Azure CLI 命令來重新部署您的虛擬機器：

```bash
azure vm redeploy --resourcegroup <resourcegroup> --vm-name <vmname> 
```

您可以看到 VM 隨著重新部署過程而變更狀態。在 VM 重新部署至新主機的過程中，它的 `PowerState` 會從「正在執行中」變成「正在更新」，接著變成「正在啟動」，最後變成「正在執行」。使用下列命令檢查資源群組內的 VM 狀態︰

```bash
azure vm list -g <resourcegroup>
```


[AZURE.INCLUDE [virtual-machines-common-redeploy-to-new-node](../../includes/virtual-machines-common-redeploy-to-new-node.md)]


## 後續步驟
如果您在連接至 VM 時發生問題，您可以在[針對 SSH 連線進行疑難排解](virtual-machines-linux-troubleshoot-ssh-connection.md)或[詳細的 SSH 疑難排解步驟](virtual-machines-linux-detailed-troubleshoot-ssh-connection.md)中找到具體的說明。如果無法存取在您 VM 上執行的應用程式，您也可以參閱[應用程式疑難排解問題](virtual-machines-linux-troubleshoot-app-connection.md)。

<!---HONumber=AcomDC_0921_2016-->