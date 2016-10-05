<properties
   pageTitle="使用 Azure CLI 與範本部署資源 | Microsoft Azure"
   description="使用 Azure Resource Manager 和 Azure CLI，將資源部署至 Azure。資源會定義在 Resource Manager 範本中。"
   services="azure-resource-manager"
   documentationCenter="na"
   authors="tfitzmac"
   manager="timlt"
   editor="tysonn"/>

<tags
   ms.service="azure-resource-manager"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="08/15/2016"
   ms.author="tomfitz"/>

# 使用 Resource Manager 範本與 Azure CLI 部署資源

> [AZURE.SELECTOR]
- [PowerShell](resource-group-template-deploy.md)
- [Azure CLI](resource-group-template-deploy-cli.md)
- [入口網站](resource-group-template-deploy-portal.md)
- [REST API](resource-group-template-deploy-rest.md)

本主題說明如何使用 Azure CLI 與 Resource Manager 範本，將您的資源部署至 Azure。

> [AZURE.TIP] 如需部署期間偵錯錯誤的說明，請參閱︰
>
> - [使用 Azure CLI 來檢視部署作業](resource-manager-troubleshoot-deployments-cli.md)，以了解有關取得可協助您針對錯誤進行疑難排解的資訊
> - [針對使用 Azure Resource Manager 將資源部署至 Azure 時常見的錯誤進行疑難排解](resource-manager-common-deployment-errors.md)，以了解如何解決常見的部署錯誤

您的範本可以是本機檔案，或者是透過 URI 提供使用的外部檔案。當您的範本位於儲存體帳戶中時，您可以限制範本的存取權，並在部署期間提供共用存取簽章 (SAS) Token

## 部署的快速步驟

本文章說明部署期間提供您選擇的所有不同選項。不過，通常您只需要兩個簡單的命令。若要快速開始部署，請使用下列命令：

    azure group create -n ExampleResourceGroup -l "West US"
    azure group deployment create -f <PathToTemplate> -e <PathToParameterFile> -g ExampleResourceGroup -n ExampleDeployment

若要深入了解可能更適合您案例的部署選項，請繼續閱讀此文章。

[AZURE.INCLUDE [resource-manager-deployments](../includes/resource-manager-deployments.md)]

## 使用 Azure CLI 進行部署

如果您之前未曾搭配使用 Azure CLI 與資源管理員，請參閱[搭配使用適用於 Mac、Linux 和 Windows 的 Azure CLI 與 Azure 資源管理](xplat-cli-azure-resource-manager.md)。

1. 登入您的 Azure 帳戶。提供您的認證之後，命令會傳回您的登入的結果。

        azure login
  
        ...
        info:    login command OK

2. 如果您有多個訂用帳戶，請提供您想要用於部署的訂用帳戶識別碼。

        azure account set <YourSubscriptionNameOrId>

3. 切換至 Azure 資源管理員模組。您會收到新模式的確認。

        azure config mode arm
   
        info:     New mode is arm

4. 如果您沒有現有資源群組，請建立新的資源群組。提供您的解決方案所需的資源群組名稱和位置。您需要提供資源群組的位置，因為資源群組會儲存資源的相關中繼資料。為了符合法規，您可能會想要指定中繼資料的儲存位置。一般情況下，我們建議您指定大部分資源所在的位置。使用相同位置可簡化範本。

        azure group create -n ExampleResourceGroup -l "West US"

     隨即傳回新資源群組的摘要。
   
        info:    Executing command group create
        + Getting resource group ExampleResourceGroup
        + Creating resource group ExampleResourceGroup
        info:    Created resource group ExampleResourceGroup
        data:    Id:                  /subscriptions/####/resourceGroups/ExampleResourceGroup
        data:    Name:                ExampleResourceGroup
        data:    Location:            westus
        data:    Provisioning State:  Succeeded
        data:    Tags:
        data:
        info:    group create command OK

5. 請先執行 **azure group template validate** 命令來驗證部署，然後再執行部署。測試部署時，請提供與執行部署時完全一致的參數 (如下個步驟所示)。

        azure group template validate -f <PathToTemplate> -p "{"ParameterName":{"value":"ParameterValue"}}" -g ExampleResourceGroup

5. 若要將資源部署至資源群組，請執行下列命令，並提供必要的參數。參數會包含您部署的名稱、資源群組的名稱、範本的路徑或 URL，以及您案例所需的任何其他參數。
   
     您有下列三個選項可以用來提供參數值：

     1. 使用內嵌參數和本機範本。每個參數皆為此格式：`"ParameterName": { "value": "ParameterValue" }`。以下範例顯示的是含逸出字元的參數。

            azure group deployment create -f <PathToTemplate> -p "{"ParameterName":{"value":"ParameterValue"}}" -g ExampleResourceGroup -n ExampleDeployment

     2. 使用內嵌參數和範本的連結。

            azure group deployment create --template-uri <LinkToTemplate> -p "{"ParameterName":{"value":"ParameterValue"}}" -g ExampleResourceGroup -n ExampleDeployment

     3. 使用參數檔案。如需範本檔案的相關資訊，請參閱[參數檔案](./#parameter-file)。
    
            azure group deployment create -f <PathToTemplate> -e <PathToParameterFile> -g ExampleResourceGroup -n ExampleDeployment

     透過上述 3 個方法之一部署好資源之後，即可看到部署的摘要。
  
        info:    Executing command group deployment create
        + Initializing template configurations and parameters
        + Creating a deployment
        ...
        info:    group deployment create command OK

     若要執行完整部署，請將 **mode** 設為 **Complete**。使用此模式時請務必謹慎，因為您可以不小心刪除不在範本中的資源。

        azure group deployment create --mode Complete -f <PathToTemplate> -e <PathToParameterFile> -g ExampleResourceGroup -n ExampleDeployment

6. 如果您想要記錄部署的其他相關資訊，以助於針對任何部署錯誤進行疑難排解，請使用 **debug-setting** 參數。您可以指定在記錄部署作業時，一併記錄要求內容及/或回應內容。

        azure group deployment create --debug-setting All -f <PathToTemplate> -e <PathToParameterFile> -g ExampleResourceGroup -n ExampleDeployment

## 使用 SAS Token 從儲存體部署範本

您可以將範本加入儲存體帳戶，並在部署期間使用 SAS Token 連結它們。

> [AZURE.IMPORTANT] 遵循下列步驟，則僅有帳戶擁有者可以存取包含範本的 Blob。不過，當您建立 Blob 的 SAS Token 時，具備該 URI 的任何人都可以存取該 Blob。如果另一位使用者攔截了 URI，該使用者也能存取範本。使用 SAS Token 是限制存取您的範本的好方法，但您不應該將機密資料 (如密碼) 直接包含在範本中。

### 將私人範本新增至儲存體帳戶

下列步驟設定範本的儲存體帳戶：

1. 建立資源群組。

        azure group create -n "ManageGroup" -l "westus"

2. 建立儲存體帳戶。儲存體帳戶的名稱在整個 Azure 中必須是唯一的，因此請為帳戶提供您自己的名稱。

        azure storage account create -g ManageGroup -l "westus" --sku-name LRS --kind Storage storagecontosotemplates

3. 設定儲存體帳戶和金鑰的變數。

        export AZURE_STORAGE_ACCOUNT=storagecontosotemplates
        export AZURE_STORAGE_ACCESS_KEY={storage_account_key}

4. 建立容器。權限設為 [關閉] 表示容器僅限擁有者存取。

        azure storage container create --container templates -p Off 
        
4. 將您的範本新增至容器。

        azure storage blob upload --container templates -f c:\Azure\Templates\azuredeploy.json
        
### 在部署期間提供 SAS Token

若要在儲存體帳戶中部署私人範本，請擷取 SAS Token 並將它包含在範本的 URI 中。

1. 建立具有讀取權限與到期時間的 SAS Token，以限制存取權限。設定到期時間，以允許足夠的時間來完成部署。擷取包括 SAS Token 的完整範本 URI。

        expiretime=$(date -I'minutes' --date "+30 minutes")
        fullurl=$(azure storage blob sas create --container templates --blob azuredeploy.json --permissions r --expiry $expiretimetime --json  | jq ".url")

2. 提供包含 SAS Token 的 URI 來部署範本。

        azure group deployment create --template-uri $fullurl -g ExampleResourceGroup

如需使用包含已連結範本的 SAS Token 範例，請參閱[透過 Azure Resource Manager 使用連結的範本](resource-group-linked-templates.md)。

[AZURE.INCLUDE [resource-manager-parameter-file](../includes/resource-manager-parameter-file.md)]

## 後續步驟
- 如需透過 .NET 用戶端程式庫部署資源的範例，請參閱[使用 .NET 程式庫與範本部署資源](virtual-machines/virtual-machines-windows-csharp-template.md)。
- 若要在範本中定義參數，請參閱[編寫範本](resource-group-authoring-templates.md#parameters)。
- 如需將您的方案部署到不同環境的指引，請參閱 [Microsoft Azure 中的開發和測試環境](solution-dev-test-environments.md)。
- 如需有關使用 KeyVault 參考來傳遞安全值的詳細資料，請參閱[在部署期間傳遞安全值](resource-manager-keyvault-parameter.md)。

<!---HONumber=AcomDC_0921_2016-->