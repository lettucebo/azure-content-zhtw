<properties
	pageTitle="Resource Manager 範本中的處理狀態 | Microsoft Azure"
	description="示範透過「Azure 資源管理員」範本與連結的範本，使用複雜物件來共用狀態資料的建議做法"
	services="azure-resource-manager"
	documentationCenter=""
	authors="tfitzmac"
	manager="timlt"
	editor="tysonn"/>

<tags
	ms.service="azure-resource-manager"
	ms.workload="multiple"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="07/12/2016"
	ms.author="tomfitz"/>

# Azure 資源管理員範本中的共用狀態

這個主題說明在範本中管理和共用狀態的最佳做法。本主題所顯示的參數與變數為您可以定義的物件類型範例，方便您用來組織部署需求。在這些範例中，您可以實作自己的物件與您環境適用的屬性值。

本主題是較大份白皮書的一部分。若要閱讀整份文件，請下載 [世界級 ARM 範本注意事項和證明可行的作法](http://download.microsoft.com/download/8/E/1/8E1DBEFA-CECE-4DC9-A813-93520A5D7CFE/World Class ARM Templates - Considerations and Proven Practices.pdf)。


## 提供標準的組態設定

與其給予可提供總彈性和無數個變化的範本，其實常見的模式是提供可選取已知組態的能力 - 實際上，是諸如沙箱、小型、中型和大型等標準 T 恤尺寸。T 恤尺寸的其他範例包括產品供應項目，例如社群版本或企業版本。在其他情況下，這可能是某種技術的工作負載特定組態， 例如，對應減少或沒有 SQL。

有了複雜物件，您可以建立變數，其中包含有時也稱為「屬性包」的資料集合，並使用該資料在範本中驅動資源宣告。這種方法可針對預先為客戶設定好的各種大小提供良好且已知的組態。如果沒有已知組態，客戶就必須自行判斷叢集大小、納入平台資源限制，以及進行數學運算來識別儲存體帳戶所產生的資料分割和其他資源 (因叢集大小和資源限制而導致) 。除了為客戶提供更好的經驗，少量的已知組態可讓您更輕鬆地提供支援，並協助您提供較高的密度等級。

下列範例將顯示如何定義包含複雜物件以代表資料集合的變數。此集合定義的值用於虛擬機器大小、網路設定、作業系統設定，以及可用性設定。

    "variables": {
      "tshirtSize": "[variables(concat('tshirtSize', parameters('tshirtSize')))]",
      "tshirtSizeSmall": {
        "vmSize": "Standard_A1",
        "diskSize": 1023,
        "vmTemplate": "[concat(variables('templateBaseUrl'), 'database-2disk-resources.json')]",
        "vmCount": 2,
        "storage": {
          "name": "[parameters('storageAccountNamePrefix')]",
          "count": 1,
          "pool": "db",
          "map": [0,0],
          "jumpbox": 0
        }
      },
      "tshirtSizeMedium": {
        "vmSize": "Standard_A3",
        "diskSize": 1023,
        "vmTemplate": "[concat(variables('templateBaseUrl'), 'database-8disk-resources.json')]",
        "vmCount": 2,
        "storage": {
          "name": "[parameters('storageAccountNamePrefix')]",
          "count": 2,
          "pool": "db",
          "map": [0,1],
          "jumpbox": 0
        }
      },
      "tshirtSizeLarge": {
        "vmSize": "Standard_A4",
        "diskSize": 1023,
        "vmTemplate": "[concat(variables('templateBaseUrl'), 'database-16disk-resources.json')]",
        "vmCount": 3,
        "slaveCount": 2,
        "storage": {
          "name": "[parameters('storageAccountNamePrefix')]",
          "count": 2,
          "pool": "db",
          "map": [0,1,1],
          "jumpbox": 0
        }
      },
      "osSettings": {
        "scripts": [
          "[concat(variables('templateBaseUrl'), 'install_postgresql.sh')]",
          "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/shared_scripts/ubuntu/vm-disk-utils-0.1.sh"
        ],
        "imageReference": {
          "publisher": "Canonical",
          "offer": "UbuntuServer",
          "sku": "14.04.2-LTS",
          "version": "latest"
        }
      },
      "networkSettings": {
        "vnetName": "[parameters('virtualNetworkName')]",
        "addressPrefix": "10.0.0.0/16",
        "subnets": {
          "dmz": {
            "name": "dmz",
            "prefix": "10.0.0.0/24",
            "vnet": "[parameters('virtualNetworkName')]"
          },
          "data": {
            "name": "data",
            "prefix": "10.0.1.0/24",
            "vnet": "[parameters('virtualNetworkName')]"
          }
        }
      },
      "availabilitySetSettings": {
        "name": "pgsqlAvailabilitySet",
        "fdCount": 3,
        "udCount": 5
      }
    }

請注意，**tshirtSize** 變數會串連您透過參數 (**小**、**中**、**大**) 提供給文字 **tshirtSize** 的 T 恤大小。您會使用這個變數來擷取該 T 恤大小的相關聯複雜物件變數。

您稍後可以在範本中參考這些變數。參考具名變數和其屬性的能力可簡化範本語法，而且可以讓您更容易了解內容。下列範例使用如上所示的物件來設定值，以定義要部署的資源。例如，請注意 VM 大小是透過擷取 `variables('tshirtSize').vmSize` 的值來設定，而磁碟大小的值則是擷取自 `variables('tshirtSize').diskSize`。此外，連結之範本的 URI 是使用 `variables('tshirtSize').vmTemplate` 的值來設定。

    "name": "master-node",
    "type": "Microsoft.Resources/deployments",
    "apiVersion": "2015-01-01",
    "dependsOn": [
        "[concat('Microsoft.Resources/deployments/', 'shared')]"
    ],
    "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[variables('tshirtSize').vmTemplate]",
          "contentVersion": "1.0.0.0"
        },
        "parameters": {
          "adminPassword": {
            "value": "[parameters('adminPassword')]"
          },
          "replicatorPassword": {
            "value": "[parameters('replicatorPassword')]"
          },
          "osSettings": {
            "value": "[variables('osSettings')]"
          },
          "subnet": {
            "value": "[variables('networkSettings').subnets.data]"
          },
          "commonSettings": {
            "value": {
              "region": "[parameters('region')]",
              "adminUsername": "[parameters('adminUsername')]",
              "namespace": "ms"
            }
          },
          "storageSettings": {
            "value":"[variables('tshirtSize').storage]"
          },
          "machineSettings": {
            "value": {
              "vmSize": "[variables('tshirtSize').vmSize]",
              "diskSize": "[variables('tshirtSize').diskSize]",
              "vmCount": 1,
              "availabilitySet": "[variables('availabilitySetSettings').name]"
            }
          },
          "masterIpAddress": {
            "value": "0"
          },
          "dbType": {
            "value": "MASTER"
          }
        }
      }
    }

## 將狀態傳遞至範本

您可以透過於部署期間直接提供的參數，在範本中共用狀態。

下表列出在範本中的常用參數。

名稱 | 值 | 說明
---- | ----- | -----------
location | 來自 Azure 區域之條件約束清單的字串 | 將部署資源的位置。
storageAccountNamePrefix | String | 將放置 VM 磁碟之儲存體帳戶的唯一 DNS 名稱
domainName | String | 可公開存取的 jumpbox VM 網域名稱格式為：**{domainName}.{location}.cloudapp.com** 例如：**mydomainname.westus.cloudapp.azure.com**
adminUsername | String | VM 的使用者名稱
adminPassword | String | VM 的密碼
tshirtSize | 來自提供 T 恤大小之條件約束清單的字串 | 要佈建的具名縮放單位大小。例如，"Small"、"Medium"、"Large"
virtualNetworkName | String | 取用者想使用的虛擬網路名稱。
enableJumpbox | 來自條件約束清單的字串 (enabled/disabled) | 識別是否要為環境啟用 JumpBox 的參數。值："enabled"、"disabled"

上節中所用的 **tshirtSize** 參數定義如下：

    "parameters": {
      "tshirtSize": {
        "type": "string",
        "defaultValue": "Small",
        "allowedValues": [
          "Small",
          "Medium",
          "Large"
        ],
        "metadata": {
          "Description": "T-shirt size of the MongoDB deployment"
        }
      }
    }


## 將狀態傳遞至連結的範本

當連線到連結的範本時，您將經常混合使用靜態變數與產生的變數。

### 靜態變數

靜態變數通常用於提供基底值，例如在範本中全程使用的 URL。

在下面的範本摘要中，`templateBaseUrl` 會指定 GitHub 中範本的根位置。下一行會建置新的變數 `sharedTemplateUrl`，它會將 基底 URL 與共用資源範本的已知名稱串連在一起。接下來，使用複雜物件變數儲存 T 恤尺寸，在此將基底 URL 和已知組態範本位置串連在一起，儲存在 `vmTemplate` 屬性中。

這個方法的優點是，如果範本位置變更，您只需要變更一個地方的靜態變數，就可以將它傳遞到所有連結的範本。

    "variables": {
      "templateBaseUrl": "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/postgresql-on-ubuntu/",
      "sharedTemplateUrl": "[concat(variables('templateBaseUrl'), 'shared-resources.json')]",
      "tshirtSizeSmall": {
        "vmSize": "Standard_A1",
        "diskSize": 1023,
        "vmTemplate": "[concat(variables('templateBaseUrl'), 'database-2disk-resources.json')]",
        "vmCount": 2,
        "slaveCount": 1,
        "storage": {
          "name": "[parameters('storageAccountNamePrefix')]",
          "count": 1,
          "pool": "db",
          "map": [0,0],
          "jumpbox": 0
        }
      }
    }

### 產生的變數

除了靜態變數，有一些變數是動態產生的。本節將說明一些產生的變數的常見類型。

#### tshirtSize

您已熟悉這個從上述範例產生的變數。

#### networkSettings

在容量、功能或端對端範圍的解決方案範本中，連結的範本通常會建立存在於網路上的資源。一個直接的方法是使用複雜物件來儲存網路設定，並將其傳送到連結的範本。

通訊網路設定的範例如下所示。

    "networkSettings": {
      "vnetName": "[parameters('virtualNetworkName')]",
      "addressPrefix": "10.0.0.0/16",
      "subnets": {
        "dmz": {
          "name": "dmz",
          "prefix": "10.0.0.0/24",
          "vnet": "[parameters('virtualNetworkName')]"
        },
        "data": {
          "name": "data",
          "prefix": "10.0.1.0/24",
          "vnet": "[parameters('virtualNetworkName')]"
        }
      }
    }

#### availabilitySettings

連結的範本中所建立的資源通常會放置在可用性集合中。在下列範例中，已指定可用性集合名稱，以及要使用的容錯網域和更新網域計數。

    "availabilitySetSettings": {
      "name": "pgsqlAvailabilitySet",
      "fdCount": 3,
      "udCount": 5
    }

如果您需要多個可用性集合 (例如，一個用於主要節點，其他用於資料節點)，您可以使用名稱做為前置詞，指定多個可用性集合，或依照稍早所示的模型建立特定 T 恤大小的變數。

#### storageSettings

儲存體詳細資料通常會與連結的範本共用。在下面的範例中，*storageSettings* 物件會提供有關儲存體帳戶和容器名稱的詳細資料。

    "storageSettings": {
        "vhdStorageAccountName": "[parameters('storageAccountName')]",
        "vhdContainerName": "[variables('vmStorageAccountContainerName')]",
        "destinationVhdsContainer": "[concat('https://', parameters('storageAccountName'), variables('vmStorageAccountDomain'), '/', variables('vmStorageAccountContainerName'), '/')]"
    }

#### osSettings

利用連結的範本，您可能需要傳遞作業系統設定到各種節點類型，並橫跨不同的已知組態類型。複雜物件是一種用來儲存及共用作業系統資訊的簡單方式，也讓支援多個作業系統部署的選擇變得更輕鬆。

下列範例顯示 *osSettings* 的物件：

    "osSettings": {
      "imageReference": {
        "publisher": "Canonical",
        "offer": "UbuntuServer",
        "sku": "14.04.2-LTS",
        "version": "latest"
      }
    }

#### machineSettings

產生的變數 *machineSettings* 為複雜物件，其中包含用來建立新 VM 的混合核心變數：系統管理員使用者名稱與密碼、VM 名稱的前置詞，以及作業系統映像參考，如下所示：

    "machineSettings": {
        "adminUsername": "[parameters('adminUsername')]",
        "adminPassword": "[parameters('adminPassword')]",
        "machineNamePrefix": "mongodb-",
        "osImageReference": {
            "publisher": "[variables('osFamilySpec').imagePublisher]",
            "offer": "[variables('osFamilySpec').imageOffer]",
            "sku": "[variables('osFamilySpec').imageSKU]",
            "version": "latest"
        }
    },

請注意 *osImageReference* 會擷取在主要範本中所定義之 *osSettings* 變數的值。這表示您可以輕鬆地變更 VM 的作業系統—完全地或根據範本取用者的喜好設定。

#### vmScripts

*vmScripts* 物件包含要在 VM 執行個體上下載及執行之指令碼的詳細資料，包括外部和內部參考。外部參考包含基礎結構。內部參考包含已安裝的軟體和組態。

您可以使用 *scriptsToDownload* 屬性列出要下載到 VM 的指令碼。

如以下範例所示，此物件也包含對不同動作類型之命令列引數的參考。這些動作包括為每個個別節點執行預設安裝、部署所有節點後所執行的安裝，以及可能為給定範本指定的任何其他指令碼。

這個範例來自用於部署 MongoDB 的範本，需要有仲裁程式以提供高可用性。*arbiterNodeInstallCommand* 已加入到 *vmScripts* 中以安裝仲裁程式。

您可以在變數區段中找到定義特定文字以搭配適當的值執行指令碼的變數。

    "vmScripts": {
        "scriptsToDownload": [
            "[concat(variables('scriptUrl'), 'mongodb-', variables('osFamilySpec').osName, '-install.sh')]",
            "[concat(variables('sharedScriptUrl'), 'vm-disk-utils-0.1.sh')]"
        ],
        "regularNodeInstallCommand": "[variables('installCommand')]",
        "lastNodeInstallCommand": "[concat(variables('installCommand'), ' -l')]",
        "arbiterNodeInstallCommand": "[concat(variables('installCommand'), ' -a')]"
    },


## 從範本傳回狀態

您不只可以將資料傳遞到範本中，也可以與發出呼叫的範本共用資料。在已連結範本的 **outputs** 區段中，您可以提供可供來源範本使用的機碼/值組。

下列範例顯示如何傳遞在連結的範本中產生的私人 IP 位址。

    "outputs": {
        "masterip": {
            "value": "[reference(concat(variables('nicName'),0)).ipConfigurations[0].properties.privateIPAddress]",
            "type": "string"
         }
    }

在主要範本中，您可以透過下列語法使用該資料：

    "[reference('master-node').outputs.masterip.value]"

這個運算式可以用在主要範本的輸出區段或資源區段。因為這個運算式依賴執行階段狀態，所以您無法在變數區段使用它。若要從主要範本傳回這個值，請使用︰

    "outputs": { 
      "masterIpAddress": {
        "value": "[reference('master-node').outputs.masterip.value]",
        "type": "string"
      }
     
如需使用已連結範本的輸出區段傳回虛擬機器之資料磁碟的範例，請參閱 [Creating multiple data disks for a Virtual Machine (為虛擬機器建立多個資料磁碟)](resource-group-create-multiple.md#creating-multiple-data-disks-for-a-virtual-machine)。

## 為虛擬機器定義驗證設定

您可以使用與上文所示的相同組態設定模式，指定虛擬機器的驗證設定。您要建立在驗證類型中傳遞的參數。

    "parameters": {
      "authenticationType": {
        "allowedValues": [
          "password",
          "sshPublicKey"
        ],
        "defaultValue": "password",
        "metadata": {
          "description": "Authentication type"
        },
        "type": "string"
      }
    }

您要加入不同的驗證類型變數，有一個變數會根據參數值，儲存這個部署所用的類型。

    "variables": {
      "osProfile": "[variables(concat('osProfile', parameters('authenticationType')))]",
      "osProfilepassword": {
        "adminPassword": "[parameters('adminPassword')]",
        "adminUsername": "notused",
        "computerName": "[parameters('vmName')]",
        "customData": "[base64(variables('customData'))]"
      },
      "osProfilesshPublicKey": {
        "adminUsername": "notused",
        "computerName": "[parameters('vmName')]",
        "customData": "[base64(variables('customData'))]",
        "linuxConfiguration": {
          "disablePasswordAuthentication": "true",
          "ssh": {
            "publicKeys": [
              {
                "keyData": "[parameters('sshPublicKey')]",
                "path": "/home/notused/.ssh/authorized_keys"
              }
            ]
          }
        }
      }
    }

定義虛擬機器時，您要將 **osProfile** 設成您建立的變數。

    {
      "type": "Microsoft.Compute/virtualMachines",
      ...
      "osProfile": "[variables('osProfile')]"
    }


## 後續步驟
- 如要了解範本的各區段，請參閱[編寫 Azure Resource Manager 範本](resource-group-authoring-templates.md)。
- 若要查看範本中可以使用的函數，請參閱 [Azure Resource Manager 範本函數](resource-group-template-functions.md)。

<!---HONumber=AcomDC_0713_2016-->