<properties
   pageTitle="預期狀態設定 Resource Manager 範本 | Microsoft Azure"
   description="Azure 中適用於預期狀態設定的 Resource Manager 範本定義 (包含範例和疑難排解)"
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

# 採用 Azure Resource Manager 範本的 Windows VMSS 和預期狀態設定
本文說明適用於[預期狀態設定擴充處理常式](virtual-machines-windows-extensions-dsc-overview.md) 的 Resource Manager 範本。

## Windows VM 的範本範例

下列程式碼片段會進入範本的 Resource 區段。

```json
			"name": "Microsoft.Powershell.DSC",
			"type": "extensions",
             "location": "[resourceGroup().location]",
             "apiVersion": "2015-06-15",
             "dependsOn": [
                  "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
              ],
              "properties": {
                  "publisher": "Microsoft.Powershell",
                  "type": "DSC",
                  "typeHandlerVersion": "2.20",
                  "autoUpgradeMinorVersion": true,
                  "forceUpdateTag": "[parameters('dscExtensionUpdateTagVersion')]",
                  "settings": {
                      "configuration": {
                          "url": "[concat(parameters('_artifactsLocation'), '/', variables('dscExtensionArchiveFolder'), '/', variables('dscExtensionArchiveFileName'))]",
                          "script": "dscExtension.ps1",
                          "function": "Main"
                      },
                      "configurationArguments": {
                          "nodeName": "[variables('vmName')]"
                      }
                  },
                  "protectedSettings": {
                      "configurationUrlSasToken": "[parameters('_artifactsLocationSasToken')]"
                  }

```

## Windows VMSS 的範本範例

VMSS 節點具有 "properties" 區段以及 "VirtualMachineProfile"、"extensionProfile" 屬性。DSC 已加入至 "extensions" 之下。

```json
"extensionProfile": {
            "extensions": [
                {
                    "name": "Microsoft.Powershell.DSC",
                    "properties": {
                        "publisher": "Microsoft.Powershell",
                        "type": "DSC",
                        "typeHandlerVersion": "2.20",
                        "autoUpgradeMinorVersion": true,
                        "forceUpdateTag": "[parameters('DscExtensionUpdateTagVersion')]",
                        "settings": {
                            "configuration": {
                                "url": "[concat(parameters('_artifactsLocation'), '/', variables('DscExtensionArchiveFolder'), '/', variables('DscExtensionArchiveFileName'))]",
                                "script": "DscExtension.ps1",
                                "function": "Main"
                            },
                            "configurationArguments": {
                                "nodeName": "localhost"
                            }
                        },
                        "protectedSettings": {
                            "configurationUrlSasToken": "[parameters('_artifactsLocationSasToken')]"
                        }
                    }
                }
            ]
```

## 詳細的設定資訊

下列結構描述是用於 Azure Resource Manager 範本中 Azure DSC 擴充功能的 settings 部分。

```json

"settings": {
  "wmfVersion": "latest",
  "configuration": {
    "url": "http://validURLToConfigLocation",
    "script": "ConfigurationScript.ps1",
    "function": "ConfigurationFunction"
  },
  "configurationArguments": {
    "argument1": "Value1",
    "argument2": "Value2"
  },
  "configurationData": {
    "url": "https://foo.psd1"
  },
  "privacy": {
    "dataCollection": "enable"
  },
  "advancedOptions": {
    "downloadMappings": {
      "customWmfLocation": "http://myWMFlocation"
    }
  } 
},
"protectedSettings": {
  "configurationArguments": {
    "parameterOfTypePSCredential1": {
      "userName": "UsernameValue1",
      "password": "PasswordValue1"
    },
    "parameterOfTypePSCredential2": {
      "userName": "UsernameValue2",
      "password": "PasswordValue2"
    }
  },
  "configurationUrlSasToken": "?g!bber1sht0k3n",
  "configurationDataUrlSasToken": "?dataAcC355T0k3N"
}

```

## 詳細資料
| 屬性名稱 | 類型 | 說明 |
| --- | --- | --- |
| settings.wmfVersion | string | 指定應該安裝於 VM 的 Windows Management Framework 版本。將此屬性設定為 'latest' 即可安裝最新版的 WMF。此屬性目前僅有的可能值為 **'4.0'、'5.0'、'5.0PP' 和 'latest'**。這些可能的值可能會更新。預設值是 'latest'。|
| settings.configuration.url | string | 指定要從中下載 DSC 組態 zip 檔的 URL 位置。如果提供的 URL 需要 SAS 權杖才能存取，您必須將 protectedSettings.configurationUrlSasToken 屬性設定為 SAS 權杖的值。如果已定義 settings.configuration.script 和/或 settings.configuration.function，則需要這個屬性。 |
| settings.configuration.script | string | 指定指令碼的檔案名稱，其中包含 DSC 組態的定義。此指令碼必須位於從 configuration.url 屬性所指定的 URL 下載之 zip 檔案的根資料夾中。如果已定義 settings.configuration.url 和/或 settings.configuration.script，則需要這個屬性。 |
| settings.configuration.function | string | 指定 DSC 組態的名稱。命名的組態必須包含在 configuration.script 所定義的指令碼中。如果已定義 settings.configuration.url 和/或 settings.configuration.function，則需要這個屬性。 |
| settings.configurationArguments | 集合 | 定義任何您想要傳遞至 DSC 組態的參數。這個屬性並未加密。 |
| settings.configurationData.url | string | 指定 URL，從中下載您的組態資料 (.pds1) 檔案以做為 DSC 組態的輸入。如果提供的 URL 需要 SAS 權杖才能存取，您必須將 protectedSettings.configurationDataUrlSasToken 屬性設定為 SAS 權杖的值。|
| settings.privacy.dataEnabled | string | 啟用或停用遙測收集。此屬性僅有的可能值為 **'Enable'、'Disable'、'' 或 $null**。將此屬性保持空白或 null 即可啟用遙測。預設值為 ''。[相關資訊](https://blogs.msdn.microsoft.com/powershell/2016/02/02/azure-dsc-extension-data-collection-2/) |
| settings.advancedOptions.downloadMappings | 集合 | 定義要從中下載 WMF 的替代位置。[相關資訊](http://blogs.msdn.com/b/powershell/archive/2015/10/21/azure-dsc-extension-2-2-amp-how-to-map-downloads-of-the-extension-dependencies-to-your-own-location.aspx) |
| protectedSettings.configurationArguments | 集合 | 定義任何您想要傳遞至 DSC 組態的參數。這個屬性已加密。 |
| protectedSettings.configurationUrlSasToken | string | 指定 SAS 權杖，以存取 configuration.url 所定義的 URL。這個屬性已加密。 |
| protectedSettings.configurationDataUrlSasToken | string | 指定 SAS 權杖，以存取 configurationData.url 所定義的 URL。這個屬性已加密。 |

## Settings 與ProtectedSettings
所有設定都會儲存在 VM 上的 settings 文字檔中。'settings' 之下的屬性是公用屬性，因為它們並未在 settings 文字檔中加密。'ProtectedSettings' 之下的屬性已使用憑證加密，所以不會顯示在 VM 上此檔案的純文字中。

如果設定時需要認證，它們可以包含在 protectedSettings 中：

```json
"protectedSettings": {
    "configurationArguments": {
        "parameterOfTypePSCredential1": {
       	    "userName": "UsernameValue1",
       	    "password": "PasswordValue1"
        }
    }
}
```

## 範例

下列範例衍生自 [DSC 擴充處理常式概觀頁面](virtual-machines-windows-extensions-dsc-overview.md)的〈開始使用〉一節。此範例使用 Resource Manager 範本 (而不是Cmdlet) 來部署擴充功能。儲存 「"IisInstall.ps1" 組態，將它放在 .ZIP 檔中，然後以可存取的 URL 上傳此檔案。此範例會使用 Azure Blob 儲存體，但可能從任意位置下載 .ZIP 檔案。

在 Azure Resource Manager 範本中，下列程式碼會指示 VM 下載正確的檔案並執行適當的 PowerShell 函式：

```json
"settings": {
    "configuration": {
        "url": "https://demo.blob.core.windows.net/",
        "script": "IisInstall.ps1",
        "function": "IISInstall"
    }
    } 
},
"protectedSettings": {
    "configurationUrlSasToken": "odLPL/U1p9lvcnp..."
}
```

## 從先前的格式進行更新
先前格式 (包含 ModulesUrl、ConfigurationFunction、SasToken 或 Properties 等公用屬性) 中的任何設定都會自動調整成目前的格式，並以之前的相同方式執行。

下列結構描述就是先前的 settings 結構描述看起來的樣子︰

```json
"settings": {
    "WMFVersion": "latest",
    "ModulesUrl": "https://UrlToZipContainingConfigurationScript.ps1.zip",
    "SasToken": "SAS Token if ModulesUrl points to private Azure Blob Storage",
    "ConfigurationFunction": "ConfigurationScript.ps1\\ConfigurationFunction",
    "Properties":  {
        "ParameterToConfigurationFunction1":  "Value1",
        "ParameterToConfigurationFunction2":  "Value2",
        "ParameterOfTypePSCredential1": {
            "UserName": "UsernameValue1",
            "Password": "PrivateSettingsRef:Key1" 
        },
        "ParameterOfTypePSCredential2": {
            "UserName": "UsernameValue2",
            "Password": "PrivateSettingsRef:Key2"
        }
    }
},
"protectedSettings": { 
    "Items": {
        "Key1": "PasswordValue1",
        "Key2": "PasswordValue2"
    },
    "DataBlobUri": "https://UrlToConfigurationDataWithOptionalSasToken.psd1"
}
```

以下是先前的格式如何調整成目前的格式︰

| 屬性名稱 | 先前結構描述對等項目 |
| --- | --- |
| settings.wmfVersion | settings.WMFVersion |
| settings.configuration.url | settings.ModulesUrl |
| settings.configuration.script | settings.ConfigurationFunction 的第一個部分 ('\\\' 之前) |
| settings.configuration.function | settings.ConfigurationFunction 的第二個部分 ('\\\' 之後) |
| settings.configurationArguments | settings.Properties |
| settings.configurationData.url | protectedSettings.DataBlobUri (不含 SAS 權杖) |
| settings.privacy.dataEnabled | settings.Privacy.DataEnabled |
| settings.advancedOptions.downloadMappings | settings.AdvancedOptions.DownloadMappings |
| protectedSettings.configurationArguments | protectedSettings.Properties |
| protectedSettings.configurationUrlSasToken | settings.SasToken |
| protectedSettings.configurationDataUrlSasToken | SAS token from protectedSettings.DataBlobUri |


## 疑難排解 - 錯誤碼 1100
錯誤碼 1100 表示對 DSC 擴充的使用者輸入有問題。這些錯誤的文字是可變的並可能變更。以下是一些您可能會遇到的錯誤及其修正方式。

### 無效的值
「Privacy.dataCollection 為 '{0}'。僅有的可能值為 ''、'Enable' 和 'Disable'」 「WmfVersion 為 '{0}'。僅有的可能值為 … 和 'latest'」

問題︰不允許所提供的值。

解決方式︰將無效的值變更為有效的值。請參閱〈詳細資料〉一節中的表格。

### 無效的 URL
「ConfigurationData.url 為 '{0}'。這不是有效的 URL」 「DataBlobUri 為 '{0}'。這不是有效的 URL」 「Configuration.url 為 '{0}'。這不是有效的 URL」

問題︰所提供的 URL 無效。

解決方式︰檢查您提供的所有 URL。請確定所有 URL 都解析成擴充功能可以在遠端電腦上存取的有效位置。

### 無效的 ConfigurationArgument 類型
「無效的 configurationArguments 類型 {0}」

問題︰ConfigurationArguments 屬性無法解析成 Hashtable 物件。

解決方式︰讓 ConfigurationArguments 屬性變成雜湊表。請依照上述範例中提供的格式。請留意引號、逗號和括號。

### 重複的 ConfigurationArguments
「在公用和受保護的 configurationArguments 中找到重複的引數 '{0}'」

問題︰公用設定中的 ConfigurationArguments 和受保護設定中的 ConfigurationArguments 包含相同名稱的屬性。

解決方式︰移除其中一個重複的屬性。

### 遺漏的屬性
「Configuration.function 要求指定 configuration.url 或 configuration.module」

「Configuration.url 要求指定 configuration.script」

「Configuration.script 要求指定 configuration.url」

「Configuration.url 要求指定 configuration.function」

「ConfigurationUrlSasToken 要求指定 configuration.url」

「ConfigurationDataUrlSasToken 要求指定 configurationData.url」

問題︰定義的屬性需要另一個遺漏的屬性。

解決方式︰
- 提供遺漏的屬性。
- 移除需要遺漏屬性的屬性。


## 後續步驟
[深入了解 DSC 和 VMSS](virtual-machines-scale-sets-dsc.md)

如需詳細資料，請參閱 [DSC 的安全認證管理](virtual-machines-windows-extensions-dsc-credentials.md)。

如需有關 Azure DSC 擴充功能處理常式的詳細資訊，請參閱 [Azure 期望狀態組態擴充功能處理常式簡介](virtual-machines-windows-extensions-dsc-overview.md)。

如需有關 PowerShell DSC 的詳細資訊，請[瀏覽 PowerShell 文件中心](https://msdn.microsoft.com/powershell/dsc/overview)。

<!---HONumber=AcomDC_0921_2016-->