<properties
   pageTitle="將 ASP.NET Core Linux Docker 容器部署到遠端 Docker 主機 | Microsoft Azure"
   description="了解如何使用 Visual Studio Tools for Docker，將 ASP.NET Core Web 應用程式部署到在 Azure Docker Host Linux VM 上執行的 Docker 容器"   
   services="azure-container-service"
   documentationCenter=".net"
   authors="mlearned"
   manager="douge"
   editor=""/>

<tags
   ms.service="azure-container-service"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="06/08/2016"
   ms.author="mlearned"/>

# 將 ASP.NET 容器部署到遠端 Docker 主機

## Overview
Docker 是輕量級容器引擎，與虛擬機器在某些方面類似，您可以用它來裝載應用程式和服務。本教學課程將逐步引導您使用 [Visual Studio 2015 Tools for Docker](http://aka.ms/DockerToolsForVS) 擴充，透過 PowerShell 將 ASP.NET Core 應用程式部署至 Azure 上的 Docker 主機。

## 必要條件
需要下列項目，才能完成本教學課程：

- 按照[如何使用 Docker 電腦搭配 Azure](./virtual-machines/virtual-machines-linux-docker-machine.md) 中所述，建立 Azure Docker 主機 VM
- 安裝 [Visual Studio 2015 Update 3](https://go.microsoft.com/fwlink/?LinkId=691129)
- [Microsoft ASP.NET 核心 1.0 SDK](https://go.microsoft.com/fwlink/?LinkID=809122)
- [Visual Studio 2015 RC Tools for Docker - 預覽](http://aka.ms/DockerToolsForVS)

## 1\.建立 ASP.NET 核心 Web 應用程式
下列步驟將逐步引導您建立將在本教學課程中使用的基本 ASP.NET 核心應用程式。

[AZURE.INCLUDE [create-aspnet5-app](../includes/create-aspnet5-app.md)]

## 2\.新增 Docker 支援

[AZURE.INCLUDE [create-aspnet5-app](../includes/vs-azure-tools-docker-add-docker-support.md)]

## 3\.使用 DockerTask.ps1 PowerShell 指令碼 

1.  將 PowerShell 提示開啟至您的專案的根目錄。

    ```
    PS C:\Src\WebApplication1>
    ```

1.  驗證遠端主機是否正在執行。您應該會看到狀態 = 執行中

    ```
    docker-machine ls
    NAME         ACTIVE   DRIVER   STATE     URL                        SWARM   DOCKER    ERRORS
    MyDockerHost -        azure    Running   tcp://xxx.xxx.xxx.xxx:2376         v1.10.3
    ```

    > [AZURE.NOTE] 如果您使用 Docker Beta 版，您的主機將不會列在此處。

1.  使用 -Build 參數建置應用程式

    ```
    PS C:\Src\WebApplication1> .\Docker\DockerTask.ps1 -Build -Environment Release -Machine mydockerhost
    ```  

    > [AZURE.NOTE] 如果您使用 Docker Beta 版，請省略 -Machine 引數
    > 
    > ```
    > PS C:\Src\WebApplication1> .\Docker\DockerTask.ps1 -Build -Environment Release 
    > ```  


1.  使用 -Run 參數執行應用程式

    ```
    PS C:\Src\WebApplication1> .\Docker\DockerTask.ps1 -Run -Environment Release -Machine mydockerhost
    ```

    > [AZURE.NOTE] 如果您使用 Docker Beta 版，請省略 -Machine 引數
    > 
    > ```
    > PS C:\Src\WebApplication1> .\Docker\DockerTask.ps1 -Run -Environment Release 
    > ```

	Docker 完成後，您應該會看到類似下面的結果：

    ![檢視您的應用程式][3]

[0]: ./media/vs-azure-tools-docker-hosting-web-apps-in-docker/docker-props-in-solution-explorer.png
[1]: ./media/vs-azure-tools-docker-hosting-web-apps-in-docker/change-docker-machine-name.png
[2]: ./media/vs-azure-tools-docker-hosting-web-apps-in-docker/launch-application.png
[3]: ./media/vs-azure-tools-docker-hosting-web-apps-in-docker/view-application.png

<!---HONumber=AcomDC_0921_2016-->