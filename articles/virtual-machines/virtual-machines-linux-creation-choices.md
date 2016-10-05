<properties
	pageTitle="建立 Linux VM 的不同方式 | Microsoft Azure"
	description="了解在 Azure 上建立 Linux 虛擬機器的不同方式，包含每種方法的工具和教學課程連結。"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="iainfoulds"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-linux"
	ms.devlang="na"
	ms.topic="get-started-article"
	ms.tgt_pltfrm="vm-linux"
	ms.workload="infrastructure-services"
	ms.date="09/27/2016"
	ms.author="iainfou"/>

# 在 Azure 中建立 Linux 虛擬機器的不同方式

您在 Azure 中可選擇使用您習慣的工具和工作流程來建立 Linux 虛擬機器 (VM)。本文摘要說明這些差異以及建立 Linux VM 的範例。


## Azure CLI 

Azure CLI 可透過 npm 套件、散發版本提供的套件或 Docker 容器，使用於各平台。您可以深入了解[如何安裝和設定 Azure CLI](../xplat-cli-install.md)。下列教學課程提供有關使用 Azure CLI 的範例。如需以下 CLI 快速入門命令的詳細資訊，請閱讀每篇文章：

- [從 Azure CLI 建立用於開發和測試的 Linux VM](virtual-machines-linux-quick-create-cli.md)
	- 下列範例使用名為 `azure_id_rsa.pub` 的公開金鑰建立 CoreOS VM：

	```bash
	azure vm quick-create -ssh-publickey-file ~/.ssh/azure_id_rsa.pub \
		--image-urn CoreOS
	```

- [使用 Azure 範本建立受保護的 Linux VM](virtual-machines-linux-create-ssh-secured-vm-from-template.md)
	- 下列範例使用 GitHub 上儲存的範本來建立 VM：

	```bash
	azure group create --name TestRG --location WestUS 
		--template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-sshkey/azuredeploy.json
	```

- [使用 Azure CLI 建立完整的 Linux 環境](virtual-machines-linux-create-cli-complete.md)
	- 包含在可用性設定組中建立負載平衡器和多個 VM。

- [在 Linux VM 中新增磁碟](virtual-machines-linux-add-disk.md)
	- 下列範例會將 5Gb 磁碟新增至名為 `TestVM` 的現有 VM：

	```bash
	azure vm disk attach-new --resource-group TestRG --vm-name TestVM \
		--size-in-GB 5
	```

## Azure 入口網站

[Azure 入口網站](https://portal.azure.com)可讓您快速建立 VM，因為您的系統不需安裝任何項目。使用 Azure 入口網站來建立 VM：

- [使用 Azure 入口網站建立 Linux VM](virtual-machines-linux-quick-create-portal.md)
- [使用 Azure 入口網站連接磁碟](virtual-machines-linux-attach-disk-portal.md)


## 作業系統和映像選項
建立 VM 時，根據您想要執行的作業系統來選擇映像。Azure 與其合作夥伴提供許多映像，其中有些包括已預先安裝的應用程式和工具。或者，上傳您自己的其中一個映像 (請參閱[下一節](#use-your-own-image))。

### Azure 映像
使用 `azure vm image` CLI 命令，查看可用的發佈者、散發版本和組建。

列出可用的發佈者，如下所示︰

```bash
azure vm image list-publishers --location WestUS
```

列出特定發佈者的可用產品 (提供項目)，如下所示︰

```bash
azure vm image list-offers --location WestUS --publisher Canonical
```

列出特定提供項目的可用 SKU (散發版本)，如下所示︰

```bash
azure vm image list-skus --location WestUS --publisher Canonical --offer UbuntuServer
```

列出特定版本的所有可用映像，如下所示︰

```bash
azure vm image list --location WestUS --publisher Canonical --offer UbuntuServer --sku 16.04.0-LTS
```

如需有關瀏覽和使用可用映像的更多範例，請參閱[使用 Azure CLI 瀏覽和選取 Azure 虛擬機器映像](virtual-machines-linux-cli-ps-findimage.md)。

`azure vm quick-create` 和 `azure vm create` 命令有一些別名，您可以用來快速存取更多常見的散發版本及其最新版本。使用別名通常比每次建立 VM 時指定發佈者、提供項目、SKU 和版本還要快速：

| Alias | 發佈者 | 提供項目 | SKU | 版本 |
|:----------|:----------|:-------------|:------------|:--------|
| CentOS | OpenLogic | Centos | 7\.2 | 最新 |
| CoreOS | CoreOS | CoreOS | Stable | 最新 |
| Debian | credativ | Debian | 8 | 最新 |
| openSUSE | SUSE | openSUSE | 13\.2 | 最新 |
| RHEL | Redhat | RHEL | 7\.2 | 最新 |
| SLES | SLES | SLES | 12-SP1 | 最新 |
| UbuntuLTS | Canonical | UbuntuServer | 14\.04.4-LTS | 最新 |

### 使用您自己的映像

如果您需要特定自訂項目，可以「擷取」現有的 Azure VM，使用以該 VM 為基礎的映像。也可以上傳在內部部署建立的映像。如需有關支援的散發版本以及如何使用自己的映像的詳細資訊，請參閱下列文件：

- [Azure 背書的散發套件](virtual-machines-linux-endorsed-distros.md)

- [非背書散發套件的資訊](virtual-machines-linux-create-upload-generic.md)

- [如何擷取 Linux 虛擬機器以做為 Resource Manager 範本](virtual-machines-linux-capture-image.md)。
	- 用來擷取現有 VM 的快速入門範例命令：

	```bash
	azure vm deallocate --resource-group TestRG --vm-name TestVM
	azure vm generalize --resource-group TestRG --vm-name TestVM
	azure vm capture --resource-group TestRG --vm-name TestVM --vhd-name-prefix CapturedVM
	```

## 後續步驟

- 透過[入口網站](virtual-machines-linux-quick-create-portal.md)、[CLI](virtual-machines-linux-quick-create-cli.md) 或 [Azure Resource Manager 範本](virtual-machines-linux-cli-deploy-templates.md)建立 Linux VM。

- 在建立 Linux VM 後，[新增資料磁碟](virtual-machines-linux-add-disk.md)。

- [重設密碼或 SSH 金鑰及管理使用者](virtual-machines-linux-using-vmaccess-extension.md)的快速步驟

<!---HONumber=AcomDC_0928_2016-->