<properties
   pageTitle="刪除 Azure 叢集和其資源 | Microsoft Azure"
   description="了解如何透過刪除內含叢集的資源群組，或選擇性地刪除資源，完全刪除 Service Fabric 叢集。"
   services="service-fabric"
   documentationCenter=".net"
   authors="ChackDan"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="09/09/2016"
   ms.author="chackdan"/>

# 刪除 Azure 上的 Service Fabric 叢集和其使用的資源

Service Fabric 叢集是由叢集資源本身和許多其他 Azure 資源所構成。因此，若要完全刪除 Service Fabric 叢集，您也必須刪除組成叢集的所有資源。做法有兩種︰刪除叢集所在的資源群組 (這會刪除叢集資源和資源群組中的任何其他資源)，或明確刪除叢集資源和其相關聯資源 (而不是資源群組中的其他資源)。

>[AZURE.NOTE] 刪除叢集資源並「不」會刪除組成 Service Fabric 叢集的所有其他資源。

## 刪除 Service Fabric 叢集所在的整個資源群組 (RG)

若要確保刪除所有與叢集相關聯的所有資源 (包括資源群組)，這是最簡單的方法。您可以使用 PowerShell 或透過 Azure 入口網站來刪除資源群組。如果您的資源群組具有與 Service Fabric 叢集無關的資源，則可以刪除特定資源。

### 使用 Azure PowerShell 刪除資源群組

您也可以執行下列 Azure PowerShell Cmdlet 來刪除資源群組。請確定您的電腦已安裝 Azure PowerShell 1.0 或更新版本。如果您從未安裝過，請遵循[如何安裝和設定 Azure PowerShell](../powershell-install-configure.md) 中所述的步驟來進行。

開啟 PowerShell 視窗，並執行下列 PS Cmdlet：

```powershell
Login-AzureRmAccount

Remove-AzureRmResourceGroup -Name <name of ResouceGroup> -Force
```

如果您未使用 *-Force* 選項，則系統會提示您確認刪除。一旦確認，就會刪除 RG 和其所包含的所有資源。

### 在 Azure 入口網站中刪除資源群組  

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 瀏覽至您想要刪除的 Service Fabric 叢集。
3. 按一下叢集基本功能頁面上的 [資源群組] 名稱。
4. 這會顯示 [資源群組基本功能] 頁面。
5. 按一下 [刪除]。
6. 遵循該頁面上的指示，以完成資源群組的刪除。

![資源群組刪除][ResourceGroupDelete]


## 刪除叢集資源和其所使用的資源，而不是資源群組中的其他資源

如果您的資源群組只有您想要刪除之 Service Fabric 叢集相關的資源，則很容易就可以刪除整個資源群組。如果您想要選擇性地逐一刪除資源群組中的資源，請遵循這些步驟。

如果您已使用入口網站或使用範本庫中的其中一個 Service Fabric Resource Manager 範本來部署叢集，則叢集所使用的所有資源都會標上下列兩個標記。您可以使用它們來決定您想要刪除的資源。

***Tag#1：***索引鍵 = clusterName，值 = '叢集的名稱'

***Tag#2：***索引鍵 = resourceName，值 = ServiceFabric

### 在 Azure 入口網站中刪除特定資源

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 瀏覽至您想要刪除的 Service Fabric 叢集。
3. 在基本功能刀鋒視窗上，移至 [所有設定]。
4. 在設定刀鋒視窗中，按一下 [資源管理] 下方的 [標記]。
5. 按一下標記刀鋒視窗中的其中一個 [標記]，以取得具有該標記的所有資源清單。

    ![資源標記][ResourceTags]

6. 具有已加上標記之資源的清單之後，請按一下每個資源，然後予以刪除。

    ![已加上標記的資源][TaggedResources]

### 使用 Azure PowerShell 刪除資源

您可以執行下列 Azure PowerShell Cmdlet 來逐一刪除資源。請確定您的電腦已安裝 Azure PowerShell 1.0 或更新版本。如果您從未安裝過，請遵循[如何安裝和設定 Azure PowerShell](../powershell-install-configure.md) 中所述的步驟來進行。

開啟 PowerShell 視窗，並執行下列 PS Cmdlet：

```powershell
Login-AzureRmAccount
```
對於您想要刪除的每個資源，執行下列動作︰

```powershell
Remove-AzureRmResource -ResourceName "<name of the Resource>" -ResourceType "<Resource Type>" -ResourceGroupName "<name of the resource group>" -Force
```

若要刪除叢集資源，請執行下列動作︰

```powershell
Remove-AzureRmResource -ResourceName "<name of the Resource>" -ResourceType "Microsoft.ServiceFabric/clusters" -ResourceGroupName "<name of the resource group>" -Force
```

## 後續步驟
也請參閱下列文章，了解如何升級叢集以及分割服務︰

- [了解叢集升級](service-fabric-cluster-upgrade.md)
- [了解如何分割具狀態服務來達到最大規模](service-fabric-concepts-partitioning.md)


<!--Image references-->
[ResourceGroupDelete]: ./media/service-fabric-cluster-delete/ResourceGroupDelete.PNG

[ResourceTags]: ./media/service-fabric-cluster-delete/ResourceTags.png

[TaggedResources]: ./media/service-fabric-cluster-delete/TaggedResources.PNG

<!---HONumber=AcomDC_0921_2016-->