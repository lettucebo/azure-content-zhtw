<properties
	pageTitle="使用 Resource Manager 範本建立 VM | Microsoft Azure"
	description="使用 Resource Manager 範本和 PowerShell 輕鬆地建立新的 Windows 虛擬機器。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="davidmu1"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.workload="na"
	ms.tgt_pltfrm="vm-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.date="07/14/2016"
	ms.author="davidmu"/>

# 利用 Resource Manager 範本建立 Windows 虛擬機器

本文介紹 Azure Resource Manager 範本，並說明如何使用 PowerShell 進行部署。此範本會在具有單一子網路的新虛擬網路上，部署執行 Windows Server 的單一虛擬機器。

執行本文中的步驟約需 20 分鐘的時間。

> [AZURE.IMPORTANT] 如果您希望 VM 屬於可用性設定組的一部分，可在建立 VM 時將其加入設定組。目前不支援在建立 VM 之後，再將其加入可用性設定組。

## 步驟 1：建立範本檔案

您可以使用[編寫 Azure Resource Manager 範本](../resource-group-authoring-templates.md)中的資訊來建立專屬範本。您也可以從 [Azure 快速入門範本](https://azure.microsoft.com/documentation/templates/)部署已為您建立的範本。

1. 開啟您常用的文字編輯器，然後將此 JSON 資訊複製到名為 *VirtualMachineTemplate.json* 的新檔案中：

        {
          "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {
            "newStorageAccountName": {
              "type": "string",
              "metadata": {
                "Description": "The name of the storage account where the VM disk is stored."
              }
            },
            "adminUsername": {
              "type": "string",
              "metadata": {
                "Description": "The name of the administrator account on the VM."
              }
            },
            "adminPassword": {
              "type": "securestring",
              "metadata": {
                "Description": "The administrator account password on the VM."
              }
            },
            "dnsNameForPublicIP": {
              "type": "string",
              "metadata": {
                "Description": "The name of the public IP address used to access the VM."
              }
            }
          },
          "variables": {
            "location": "Central US",
            "imagePublisher": "MicrosoftWindowsServer",
            "imageOffer": "WindowsServer",
            "windowsOSVersion": "2012-R2-Datacenter",
            "OSDiskName": "osdisk1",
            "nicName": "nc1",
            "addressPrefix": "10.0.0.0/16",
            "subnetName": "sn1",
            "subnetPrefix": "10.0.0.0/24",
            "storageAccountType": "Standard_LRS",
            "publicIPAddressName": "ip1",
            "publicIPAddressType": "Dynamic",
            "vmStorageAccountContainerName": "vhds",
            "vmName": "vm1",
            "vmSize": "Standard_A0",
            "virtualNetworkName": "vn1",
            "vnetID": "[resourceId('Microsoft.Network/virtualNetworks',variables('virtualNetworkName'))]",
            "subnetRef": "[concat(variables('vnetID'),'/subnets/',variables('subnetName'))]"
          },
          "resources": [
            {
              "type": "Microsoft.Storage/storageAccounts",
              "name": "[parameters('newStorageAccountName')]",
              "apiVersion": "2015-06-15",
              "location": "[variables('location')]",
              "properties": {
                "accountType": "[variables('storageAccountType')]"
              }
            },
            {
              "apiVersion": "2016-03-30",
              "type": "Microsoft.Network/publicIPAddresses",
              "name": "[variables('publicIPAddressName')]",
              "location": "[variables('location')]",
              "properties": {
                "publicIPAllocationMethod": "[variables('publicIPAddressType')]",
                "dnsSettings": {
                  "domainNameLabel": "[parameters('dnsNameForPublicIP')]"
                }
              }
            },
            {
              "apiVersion": "2016-03-30",
              "type": "Microsoft.Network/virtualNetworks",
              "name": "[variables('virtualNetworkName')]",
              "location": "[variables('location')]",
              "properties": {
                "addressSpace": {
                  "addressPrefixes": [
                    "[variables('addressPrefix')]"
                  ]
                },
                "subnets": [
                  {
                    "name": "[variables('subnetName')]",
                    "properties": {
                      "addressPrefix": "[variables('subnetPrefix')]"
                    }
                  }
                ]
              }
            },
            {
              "apiVersion": "2016-03-30",
              "type": "Microsoft.Network/networkInterfaces",
              "name": "[variables('nicName')]",
              "location": "[variables('location')]",
              "dependsOn": [
                "[concat('Microsoft.Network/publicIPAddresses/', variables('publicIPAddressName'))]",
                "[concat('Microsoft.Network/virtualNetworks/', variables('virtualNetworkName'))]"
              ],
              "properties": {
                "ipConfigurations": [
                  {
                    "name": "ipconfig1",
                    "properties": {
                      "privateIPAllocationMethod": "Dynamic",
                      "publicIPAddress": {
                        "id": "[resourceId('Microsoft.Network/publicIPAddresses',variables('publicIPAddressName'))]"
                      },
                      "subnet": {
                        "id": "[variables('subnetRef')]"
                      }
                    }
                  }
                ]
              }
            },
            {
              "apiVersion": "2016-03-30",
              "type": "Microsoft.Compute/virtualMachines",
              "name": "[variables('vmName')]",
              "location": "[variables('location')]",
              "dependsOn": [
                "[concat('Microsoft.Storage/storageAccounts/', parameters('newStorageAccountName'))]",
                "[concat('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
              ],
              "properties": {
                "hardwareProfile": {
                  "vmSize": "[variables('vmSize')]"
                },
                "osProfile": {
                  "computername": "[variables('vmName')]",
                  "adminUsername": "[parameters('adminUsername')]",
                  "adminPassword": "[parameters('adminPassword')]"
                },
                "storageProfile": {
                  "imageReference": {
                    "publisher": "[variables('imagePublisher')]",
                    "offer": "[variables('imageOffer')]",
                    "sku" : "[variables('windowsOSVersion')]",
                    "version":"latest"
                  },
                  "osDisk" : {
                    "name": "osdisk",
                    "vhd": {
                      "uri": "[concat('http://',parameters('newStorageAccountName'),'.blob.core.windows.net/',variables('vmStorageAccountContainerName'),'/',variables('OSDiskName'),'.vhd')]"
                    },
                    "caching": "ReadWrite",
                    "createOption": "FromImage"
                  }
                },
                "networkProfile": {
                  "networkInterfaces": [
                    {
                      "id": "[resourceId('Microsoft.Network/networkInterfaces',variables('nicName'))]"
                    }
                  ]
                }
              }
            }
          ]
        }
        
    >[AZURE.NOTE] 本文會建立執行 Windows Server 作業系統版本的虛擬機器。若要深入了解如何選取其他映像，請參閱[使用 Windows PowerShell 和 Azure CLI 來瀏覽和選取 Azure 虛擬機器映像](virtual-machines-linux-cli-ps-findimage.md)。
    
2. 儲存範本檔案。

## 步驟 2︰建立參數檔案

若要為範本中所定義的資源參數指定值，您可以建立參數檔案，其中包含值並且將其提交至具有範本的資源管理員。

1. 在文字編輯器中，將此 JSON 資訊複製到名為 *Parameters.json* 的新檔案中：

        {
          "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json",
          "contentVersion": "1.0.0.0",
          "parameters": {
            "newStorageAccountName": { "value": "mytestsa1" },
            "adminUserName": { "value": "mytestacct1" },
            "adminPassword": { "value": "mytestpass1" },
            "dnsNameForPublicIP": { "value": "mytestdns1" }
          }
        }

    >[AZURE.NOTE] 進一步了解[使用者名稱和密碼需求](virtual-machines-windows-faq.md#what-are-the-username-requirements-when-creating-a-vm)。

2. 儲存參數檔案。

## 步驟 3：安裝 Azure PowerShell

如需如何安裝最新版 Azure PowerShell、選取要使用的訂用帳戶，以及登入 Azure 帳戶的相關資訊，請參閱[如何安裝和設定 Azure PowerShell](../powershell-install-configure.md)。

## 步驟 4：建立資源群組

所有資源都必須部署在資源群組中。如需詳細資訊，請參閱 [Azure Resource Manager 概觀](../resource-group-overview.md)。

1. 取得可以建立資源的可用位置清單。

	    Get-AzureRmLocation | sort DisplayName | Select DisplayName

2. 使用清單中的位置 (例如**美國中部**) 取代 **$locName** 的值。建立變數。

        $locName = "location name"
        
3. 使用新資源群組的名稱取代 **$rgName** 的值。建立變數和資源群組。

        $rgName = "resource group name"
        New-AzureRmResourceGroup -Name $rgName -Location $locName
        
    您應該會看到如下的結果：
    
        ResourceGroupName : myrg1
        Location          : centralus
        ProvisioningState : Succeeded
        Tags              :
        ResourceId        : /subscriptions/{subscription-id}/resourceGroups/myrg1

### 步驟 5 ︰使用範本和參數來建立資源

1. 使用部署名稱取代 **$deployName** 的值。使用範本檔案的路徑和名稱取代 **$templatePath** 的值。使用參數檔案的路徑和名稱取代 **$parameterFile** 的值。建立變數。

        $deployName="deployment name"
        $templatePath = "template path"
        $parameterFile = "parameter file"

2. 部署範本。

        New-AzureRmResourceGroupDeployment -ResourceGroupName "davidmurg6" -TemplateFile $templatePath -TemplateParameterFile $parameterFile

    您會看到類似下列畫面：

        DeploymentName    : VirtualMachineTemplate
        ResourceGroupName : myrg1
        ProvisioningState : Succeeded
        Timestamp         : 4/14/2016 8:11:37 PM
        Mode              : Incremental
        TemplateLink      :
        Parameters        :
                            Name             Type                       Value
                            ===============  =========================  ==========
                            newStorageAccountName  String                     mytestsa1
                            adminUsername    String                     mytestacct1
                            adminPassword    SecureString
                            dnsNameForPublicIP  String                     mytestdns1

        Outputs           :

    >[AZURE.NOTE] 您也可以從 Azure 儲存體帳戶部署範本和參數。若要深入了解，請參閱[搭配使用 Azure PowerShell 與 Azure 儲存體](../storage-powershell-guide-full.md)。

## 後續步驟

- 如果部署有問題，下一個步驟就是查看[使用 Azure 入口網站針對資源群組部署進行疑難排解](../resource-manager-troubleshoot-deployments-portal.md)。
- 檢閱[使用 Azure Resource Manager 和 PowerShell 管理虛擬機器](virtual-machines-windows-ps-manage.md)，以了解如何管理您剛建立的虛擬機器。

<!---HONumber=AcomDC_0921_2016-->