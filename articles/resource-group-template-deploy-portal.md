<properties 
	pageTitle="使用 Azure 入口網站部署 Azure 資源 | Microsoft Azure" 
	description="使用 Azure 入口網站和 Azure Resource Manager 來部署資源。" 
	services="azure-resource-manager,azure-portal" 
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
	ms.date="09/15/2016" 
	ms.author="tomfitz"/>

# 使用 Resource Manager 範本與 Azure 入口網站來部署資源

> [AZURE.SELECTOR]
- [PowerShell](resource-group-template-deploy.md)
- [Azure CLI](resource-group-template-deploy-cli.md)
- [入口網站](resource-group-template-deploy-portal.md)
- [REST API](resource-group-template-deploy-rest.md)

本主題示範如何使用 [Azure 入口網站](https://portal.azure.com)搭配 [Azure Resource Manager](resource-group-overview.md) 來部署您的 Azure 資源。若要了解如何管理您的資源，請參閱[透過入口網站管理 Azure 資源](./azure-portal/resource-group-portal.md)。

目前並非所有服務都支援入口網站或資源管理員。針對這些服務，您必須使用[傳統入口網站](https://manage.windowsazure.com)。如需每個服務的狀態，請參閱 [Azure 入口網站可用性圖表](https://azure.microsoft.com/features/azure-portal/availability/)。

## 建立資源群組

1. 若要建立空的資源群組，請選取 [新增] > [管理] > [資源群組]。

    ![建立空的資源群組](./media/resource-group-template-deploy-portal/create-empty-group.png)

2. 提供其名稱和位置，再視需要選取訂用帳戶。您需要提供資源群組的位置，因為資源群組會儲存資源的相關中繼資料。為了符合法規，您可能會想要指定中繼資料的儲存位置。一般情況下，我們建議您指定大部分資源所在的位置。使用相同位置可簡化範本。

    ![設定群組值](./media/resource-group-template-deploy-portal/set-group-properties.png)

## 從 Marketplace 部署資源

建立資源群組之後，您可以將資源從 Marketplace 部署至該群組。Marketplace 針對常見的案例提供預先定義的解決方案。

1. 若要開始部署，請選取 [新增] 以及您想要部署的資源類型。然後，尋找您要部署的特定版本資源。

    ![部署資源](./media/resource-group-template-deploy-portal/deploy-resource.png)

2. 如果沒有看到要部署的特定解決方案，您可以在 Marketplace 搜尋。

    ![搜尋 Marketplace](./media/resource-group-template-deploy-portal/search-resource.png)

3. 根據所選資源的類型，您必須在部署前設定相關的屬性集合。這些選項並未在這裡顯示，因為它們會隨著資源類型而有所不同。對於所有類型，您必須選取目的地資源群組。下圖顯示如何建立 Web 應用程式並將其部署至您建立的資源群組。

    ![建立資源群組](./media/resource-group-template-deploy-portal/select-existing-group.png)

    或者，您也可以決定在部署資源時建立資源群組。選取 [新建] 並提供資源群組的名稱。

    ![建立新的資源群組](./media/resource-group-template-deploy-portal/select-new-group.png)

4. 您的部署隨即開始。部署可能需要幾分鐘的時間。部署完成後，您就會看到通知。

    ![檢視通知](./media/resource-group-template-deploy-portal/view-notification.png)

5. 部署您的資源之後，您可以在資源群組刀鋒視窗上使用 [新增] 命令，將更多資源新增至資源群組。

    ![新增資源](./media/resource-group-template-deploy-portal/add-resource.png)

## 從自訂範本部署資源

如果您想要執行部署，但不使用 Marketplace 中的任何範本，您可建立自訂範本以定義您的解決方案的基礎結構。若要了解如何建立範本，請參閱[撰寫 Azure 資源管理員範本](resource-group-authoring-templates.md)。

1. 若要透過入口網站來部署自訂的範本，請選取 [新增]，並開始搜尋 [範本部署]，直到您可以從選項中選取它為止。

    ![搜尋範本部署](./media/resource-group-template-deploy-portal/search-template.png)

2. 從可用的資源中選取 [範本部署]。

    ![選取範本部署](./media/resource-group-template-deploy-portal/select-template.png)

3. 啟動範本部署之後，開啟可供自訂的空白範本。

    ![建立範本](./media/resource-group-template-deploy-portal/show-custom-template.png)

    在編輯器中，新增定義要部署資源的 JSON 語法。完成時選取 [儲存]。如需撰寫 JSON 語法的指導方針，請參閱 [Resource Manager 範本逐步解說](resource-manager-template-walkthrough.md)。

    ![編輯範本](./media/resource-group-template-deploy-portal/edit-template.png)

4. 或者，您可以從 [Azure 快速入門範本](https://azure.microsoft.com/documentation/templates/)選取既存的範本。這些範本是由社群所貢獻。其中涵蓋許多常見案例，而其他人可能會新增類似於您嘗試部署的範本。您可以搜尋範本以找出符合您的案例的範本。

    ![選取快速入門範本](./media/resource-group-template-deploy-portal/select-quickstart-template.png)

    您可以在編輯器中檢視選取的範本。

5. 提供所有其他的值之後，請選取 [建立] 來部署範本。

    ![部署範本](./media/resource-group-template-deploy-portal/create-custom-deploy.png)

## 從儲存至您帳戶的範本部署資源

入口網站可讓您將範本儲存至您的 Azure 帳戶，並於日後部署它。如需使用這些已儲存範本的詳細資訊，請參閱[開始在 Azure 入口網站上使用私人範本](./marketplace-consumer/mytemplates-getstarted.md)。

1. 若要尋找您已儲存的範本，請選取 [瀏覽] > [範本]。

    ![瀏覽範本](./media/resource-group-template-deploy-portal/browse-templates.png)

2. 在儲存至您帳戶的範本清單中，選取您要使用的範本。

    ![已儲存的範本](./media/resource-group-template-deploy-portal/saved-templates.png)

3. 選取 [部署] 來重新部署這個已儲存的範本。

    ![部署已儲存的範本](./media/resource-group-template-deploy-portal/deploy-saved-template.png)

## 後續步驟

- 若要檢視稽核記錄檔，請參閱[使用 Resource Manager 來稽核作業](resource-group-audit.md)。
- 若要針對部署錯誤進行疑難排解，請參閱[使用 Azure 入口網站對資源群組部署進行疑難排解](resource-manager-troubleshoot-deployments-portal.md)。
- 若要從部署或資源群組擷取範本，請參閱[從現有資源匯出 Azure Resource Manager 範本](resource-manager-export-template.md)。

<!---HONumber=AcomDC_0921_2016-->