<properties
   pageTitle="用 Docker 電腦在 Azure 中建立 Docker 主機 | Microsoft Azure"
   description="說明如何使用 Docker 電腦在 Azure 中建立 Docker 主機。"
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

# 使用 Docker-Machine 在 Azure 中建立 Docker 主機

執行 [Docker](https://www.docker.com/) 容器時需要執行 Docker 精靈的主機 VM。本主題說明如何使用 [docker-machine](https://docs.docker.com/machine/) 命令，來建立在 Azure 中執行並使用 Docker 精靈所設定的新 Linux VM。

**注意：**
- *本文根據 docker-machine 0.7.0 版或更新版本*
- *在不久的未來，將透過 docker-machine 支援 Windows Containers*

## 使用 Docker 電腦建立 VM

搭配使用 `docker-machine create` 命令與 `azure` 驅動程式，在 Azure 中建立 Docker 主機 VM。

Azure 驅動程式將需要您的訂用帳戶識別碼。您可以使用 [Azure CLI](xplat-cli-install.md) 或 [Azure 入口網站](https://portal.azure.com)來擷取您的「Azure 訂用帳戶」。

**使用 Azure 入口網站**
- 從左導覽頁面中選取 [訂用帳戶]，並複製到訂用帳戶識別碼。

**使用 Azure CLI**
- 輸入 ```azure account list```，並複製訂用帳戶識別碼。

輸入 `docker-machine create --driver azure` 以查看選項和其預設值。您也可以查看 [Docker Azure 驅動程式文件](https://docs.docker.com/machine/drivers/azure/)，以取得詳細資訊。

下列範例依賴預設值，但會選擇性地開啟 VM 上的連接埠 80 來進行網際網路存取。

```
docker-machine create -d azure --azure-subscription-id <Your AZURE_SUBSCRIPTION_ID> --azure-open-port 80 mydockerhost
```

## 使用 docker-machine 選擇 Docker 主機
docker-machine 中有您主機的項目之後，即可在執行 docker 命令時設定預設主機。
##使用 PowerShell

```powershell
docker-machine env MyDockerHost | Invoke-Expression 
```

##使用 Bash

```bash
eval $(docker-machine env MyDockerHost)
```

您現在可以對指定的主機執行 docker 命令

```
docker ps
docker info
```

## 執行容器

設定主機之後，您現在可以執行簡單的 Web 伺服器，來測試是否已正確設定主機。我們在此使用標準 nginx 映像，指定它應該接聽連接埠 80，而且如果主機 VM 重新啟動，則容器也會重新啟動 (`--restart=always`)。

```bash
docker run -d -p 80:80 --restart=always nginx
```

輸出應該類似下面這樣：

```
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
efd26ecc9548: Pull complete
a3ed95caeb02: Pull complete
83f52fbfa5f8: Pull complete
fa664caa1402: Pull complete
Digest: sha256:12127e07a75bda1022fbd4ea231f5527a1899aad4679e3940482db3b57383b1d
Status: Downloaded newer image for nginx:latest
25942c35d86fe43c688d0c03ad478f14cc9c16913b0e1c2971cb32eb4d0ab721
```

## 測試容器

使用 `docker ps` 檢查執行中容器：

```bash
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                         NAMES
d5b78f27b335        nginx               "nginx -g 'daemon off"   5 minutes ago       Up 5 minutes        0.0.0.0:80->80/tcp, 443/tcp   goofy_mahavira
```

查看執行中容器，並輸入 `docker-machine ip <VM name>` 以尋找要在瀏覽器中輸入的 IP 位址：

```
PS C:\> docker-machine ip MyDockerHost
191.237.46.90
```

![執行 ngnix 容器](./media/vs-azure-tools-docker-machine-azure-config/nginxsuccess.png)

##摘要
運用 docker-machine，您可以在 Azure 中輕鬆地佈建 docker 主機來進行個別 docker 主機驗證。如需實際執行裝載容器，請參閱 [Azure Container Service](http://aka.ms/AzureContainerService)

若要使用 Visual Studio 開發 .NET Core 應用程式，請參閱 [Docker Tools for Visual Studio](http://aka.ms/DockerToolsForVS)

<!---HONumber=AcomDC_0921_2016-->