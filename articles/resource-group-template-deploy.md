<properties
   pageTitle="使用 PowerShell 與範本部署資源 | Microsoft Azure"
   description="使用 Azure Resource Manager 和 Azure PowerShell，將資源部署至 Azure。資源會定義在 Resource Manager 範本中。"
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

# 使用 Resource Manager 範本與 Azure PowerShell 來部署資源

> [AZURE.SELECTOR]
- [PowerShell](resource-group-template-deploy.md)
- [Azure CLI](resource-group-template-deploy-cli.md)
- [入口網站](resource-group-template-deploy-portal.md)
- [REST API](resource-group-template-deploy-rest.md)

本主題說明如何使用 Azure PowerShell 與 Resource Manager 範本，將您的資源部署至 Azure。

> [AZURE.TIP] 如需部署期間偵錯錯誤的說明，請參閱︰
>
> - [使用 Azure PowerShell 來檢視部署作業](resource-manager-troubleshoot-deployments-powershell.md)，以了解有關取得可協助您針對錯誤進行疑難排解的資訊
> - [針對使用 Azure Resource Manager 將資源部署至 Azure 時常見的錯誤進行疑難排解](resource-manager-common-deployment-errors.md)，以了解如何解決常見的部署錯誤

您的範本可以是本機檔案，或者是透過 URI 提供使用的外部檔案。當您的範本位於儲存體帳戶中時，您可以限制範本的存取權，並在部署期間提供共用存取簽章 (SAS) Token

## 部署的快速步驟

本文章說明部署期間提供您選擇的所有不同選項。不過，通常您只需要兩個簡單的命令。若要快速開始部署，請使用下列命令：

    New-AzureRmResourceGroup -Name ExampleResourceGroup -Location "West US"
    New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup -TemplateFile <PathToTemplate> -TemplateParameterFile <PathToParameterFile>

若要深入了解可能更適合您案例的部署選項，請繼續閱讀此文章。

[AZURE.INCLUDE [resource-manager-deployments](../includes/resource-manager-deployments.md)]

## 使用 PowerShell 部署

1. 登入您的 Azure 帳戶。

        Add-AzureRmAccount

     系統會傳回您帳戶的摘要。

        Environment : AzureCloud
        Account     : someone@example.com
        ...

2. 如果您有多個訂用帳戶，請使用 **Set-AzureRmContext** 命令提供您想要用於部署的訂用帳戶識別碼。

        Set-AzureRmContext -SubscriptionID <YourSubscriptionId>

3. 一般而言，在部署新範本時，您會希望建立新的資源群組以包含該資源。如果您想要部署到現有的資源群組，則您可以略過此步驟並使用該資源群組。

     若要建立新的資源群組，請提供資源群組的名稱和位置。您需要提供資源群組的位置，因為資源群組會儲存資源的相關中繼資料。為了符合法規，您可能會想要指定中繼資料的儲存位置。一般情況下，我們建議您指定大部分資源所在的位置。使用相同位置可簡化範本。

        New-AzureRmResourceGroup -Name ExampleResourceGroup -Location "West US"
   
     隨即傳回新資源群組的摘要。
   
        ResourceGroupName : ExampleResourceGroup
        Location          : westus
        ProvisioningState : Succeeded
        Tags              :
        Permissions       :
             Actions  NotActions
             =======  ==========
             *
        ResourceId        : /subscriptions/######/resourceGroups/ExampleResourceGroup

4. 在執行部署之前，您可以先驗證部署設定。**Test-AzureRmResourceGroupDeployment** Cmdlet 可讓您在實際建立資源之前發現問題。下列範例示範如何驗證部署。

        Test-AzureRmResourceGroupDeployment -ResourceGroupName ExampleResourceGroup -TemplateFile <PathToTemplate>

5. 若要將資源部署至資源群組，請執行 **New-AzureRmResourceGroupDeployment** 命令，並提供必要的參數。參數會包含您部署的名稱、資源群組的名稱、您建立之範本的路徑或 URL，以及您案例所需的任何其他參數。如未指定 **Mode** 參數，即會使用預設值 **Incremental**。若要執行完整部署，將 **Mode** 設為 **Complete**。使用完整模式時請務必謹慎，因為您可能會不小心刪除不在範本中的資源。

     若要部署本機範本，請使用 **TemplateFile** 參數︰

        New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup -TemplateFile <PathToTemplate>

     若要部署外部範本，請使用 **TemplateUri** 參數︰

        New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup -TemplateUri <LinkToTemplate>
   
     您有下列選項可以用來提供參數值：
   
     1. 使用內嵌參數。

            New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup -TemplateFile <PathToTemplate> -myParameterName "parameterValue"

     2. 使用參數物件。

            $parameters = @{"<ParameterName>"="<Parameter Value>"}
            New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup -TemplateFile <PathToTemplate> -TemplateParameterObject $parameters

     3. 使用本機參數檔案。如需範本檔案的相關資訊，請參閱[參數檔案](#parameter-file)。

            New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup -TemplateFile <PathToTemplate> -TemplateParameterFile <PathToParameterFile>

     4. 使用外部參數檔案。如需範本檔案的相關資訊，請參閱[參數檔案](#parameter-file)。

            New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup -TemplateUri <LinkToTemplate> -TemplateParameterUri <LinkToParameterFile>

        當使用外部參數檔案時，您無法傳遞內嵌或本機檔案的其他值。如需詳細資訊，請參閱[參數優先順序](#parameter-precendence)。

     部署資源之後，您會看到部署的摘要。

        DeploymentName    : ExampleDeployment
        ResourceGroupName : ExampleResourceGroup
        ProvisioningState : Succeeded
        Timestamp         : 4/14/2015 7:00:27 PM
        Mode              : Incremental
        ...

     如果您範本所含的參數名稱與 PowerShell 命令中的其中一個參數一樣，系統會提示您提供該參數的值。來自您範本的參數會包含後置詞 **FromTemplate**。例如，範本中名為 **ResourceGroupName** 的參數與 [New-AzureRmResourceGroupDeployment](https://msdn.microsoft.com/library/azure/mt679003.aspx) Cmdlet 中的 **ResourceGroupName** 參數發生衝突。系統會提示您為 **ResourceGroupNameFromTemplate** 提供值。一般而言，在為參數命名時，請勿使用與部署作業所用參數相同的名稱，以避免發生這種混淆的情形。

6. 如果您想要記錄部署的其他相關資訊，以助於針對任何部署錯誤進行疑難排解，請使用 **DeploymentDebugLogLevel** 參數。您可以指定在記錄部署作業時，一併記錄要求內容及/或回應內容。

        New-AzureRmResourceGroupDeployment -Name ExampleDeployment -DeploymentDebugLogLevel All -ResourceGroupName ExampleResourceGroup -TemplateFile <PathOrLinkToTemplate>
        
     如需使用此偵錯內容針對部署進行疑難排解的詳細資訊，請參閱[使用 Azure PowerShell 對資源群組部署進行疑難排解](resource-manager-troubleshoot-deployments-powershell.md)。

## 使用 SAS Token 從儲存體部署範本

您可以將範本加入儲存體帳戶，並在部署期間使用 SAS Token 連結它們。

> [AZURE.IMPORTANT] 遵循下列步驟，則僅有帳戶擁有者可以存取包含範本的 Blob。不過，當您建立 Blob 的 SAS Token 時，具備該 URI 的任何人都可以存取該 Blob。如果另一位使用者攔截了 URI，該使用者也能存取範本。使用 SAS Token 是限制存取您的範本的好方法，但您不應該將機密資料 (如密碼) 直接包含在範本中。

### 將私人範本新增至儲存體帳戶

下列步驟設定範本的儲存體帳戶：

1. 建立資源群組。

        New-AzureRmResourceGroup -Name ManageGroup -Location "West US"

2. 建立儲存體帳戶。儲存體帳戶的名稱在整個 Azure 中必須是唯一的，因此請為帳戶提供您自己的名稱。

        New-AzureRmStorageAccount -ResourceGroupName ManageGroup -Name storagecontosotemplates -Type Standard_LRS -Location "West US"

3. 設定目前的儲存體帳戶。

        Set-AzureRmCurrentStorageAccount -ResourceGroupName ManageGroup -Name storagecontosotemplates

4. 建立容器。權限設為 [關閉] 表示容器僅限擁有者存取。

        New-AzureStorageContainer -Name templates -Permission Off
        
5. 將您的範本新增至容器。

        Set-AzureStorageBlobContent -Container templates -File c:\Azure\Templates\azuredeploy.json
        
### 在部署期間提供 SAS Token

若要在儲存體帳戶中部署私人範本，請擷取 SAS Token 並將它包含在範本的 URI 中。

1. 如果您有變更目前的儲存體帳戶，請將目前的儲存體帳戶設為包含範本的帳戶。

        Set-AzureRmCurrentStorageAccount -ResourceGroupName ManageGroup -Name storagecontosotemplates

2. 建立具有讀取權限與到期時間的 SAS Token，以限制存取權限。擷取包括 SAS Token 的完整範本 URI。

        $templateuri = New-AzureStorageBlobSASToken -Container templates -Blob azuredeploy.json -Permission r -ExpiryTime (Get-Date).AddHours(2.0) -FullUri

3. 提供包含 SAS Token 的 URI 來部署範本。

        New-AzureRmResourceGroupDeployment -ResourceGroupName ExampleResourceGroup -TemplateUri $templateuri

如需使用包含已連結範本的 SAS Token 範例，請參閱[透過 Azure Resource Manager 使用連結的範本](resource-group-linked-templates.md)。

[AZURE.INCLUDE [resource-manager-parameter-file](../includes/resource-manager-parameter-file.md)]

## 參數優先順序

您可以在相同的部署作業中使用內嵌參數和本機參數檔案。例如，您可以在部署期間指定本機參數檔案中的某些值，並新增其他內嵌值。如果您同時為本機檔案中和內嵌的參數提供值，內嵌值的優先順序較高。

不過，您無法搭配使用內嵌參數與外部參數檔案。當您指定 **TemplateParameterUri** 參數中的參數檔案時，所有內嵌參數都會被忽略。您必須提供外部檔案中的所有參數值。如果您的範本包含不能包含在參數檔案中的機密值，將該值新增至金鑰保存庫並在外部參數檔案中參考金鑰保存庫，或以動態方式提供所有內嵌參數值。

如需有關使用 KeyVault 參考來傳遞安全值的詳細資料，請參閱[在部署期間傳遞安全值](resource-manager-keyvault-parameter.md)。

## 後續步驟
- 如需透過 .NET 用戶端程式庫部署資源的範例，請參閱[使用 .NET 程式庫與範本部署資源](virtual-machines/virtual-machines-windows-csharp-template.md)。
- 若要在範本中定義參數，請參閱[編寫範本](resource-group-authoring-templates.md#parameters)。
- 如需如何將解決方案部署到不同環境的指南，請參閱[Microsoft Azure 中開發和測試環境](solution-dev-test-environments.md)。

<!---HONumber=AcomDC_0921_2016-->