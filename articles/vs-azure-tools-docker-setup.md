<properties
   pageTitle="使用 VirtualBox 設定 Docker 主機 | Microsoft Azure"
   description="使用 Docker 機器和 VirtualBox 來設定預設 Docker 執行個體的逐步指示。"
   services="azure-container-service"
   documentationCenter="na"
   authors="mlearned"
   manager="douge"
   editor="" />
<tags
   ms.service="multiple"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="multiple"
   ms.date="06/08/2016"
   ms.author="mlearned" />

# 使用 VirtualBox 設定 Docker 主機

## Overview
本文會逐步引導您使用 Docker 機器和 VirtualBox 來設定預設 Docker 執行個體。如果您是使用 [Docker for Windows Beta 版](http://beta.docker.com/)，則不需要此組態。

## 必要條件
您的電腦必須安裝下列工具。

- [Docker 工具箱](https://www.docker.com/products/overview#/docker_toolbox)

## 使用 Windows PowerShell 設定 Docker 用戶端

若要設定 Docker 用戶端，只需開啟 Windows PowerShell，然後執行下列步驟︰

1. 建立預設 Docker 主機執行個體。

    ```PowerShell
    docker-machine create --driver virtualbox default
    ```
 
1. 確認預設執行個體已完成設定且正在執行。(您應該會看到名為「default」的執行個體正在執行。

    ```PowerShell
	docker-machine ls 
    ```
		
	![docker-machine ls 輸出][0]
 
1. 預設為目前的主機，並設定您的 Shell。

    ```PowerShell
    docker-machine env default | Invoke-Expression
    ```

1. 顯示作用中的 Docker 容器。清單應該是空的。

    ```PowerShell
	docker ps
    ```

	![docker ps 輸出][1]
 
> [AZURE.NOTE] 每次您將開發用機器重新開機時，就需要重新啟動您的本機 Docker 主機。做法是在命令提示字元發出下列命令：`docker-machine start default`。

[0]: ./media/vs-azure-tools-docker-setup/docker-machine-ls.png
[1]: ./media/vs-azure-tools-docker-setup/docker-ps.png

<!---HONumber=AcomDC_0921_2016-->