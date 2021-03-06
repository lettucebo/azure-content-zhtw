<properties
   pageTitle="使用 Datadog 監視 Azure 容器服務叢集 | Microsoft Azure"
   description="使用 Datadog 監視 Azure 容器服務叢集。使用 DC/OS Web UI 將 Datadog 代理程式部署至您的叢集。"
   services="container-service"
   documentationCenter=""
   authors="rbitia"
   manager="timlt"
   editor=""
   tags="acs, azure-container-service"
   keywords="容器, DC/OS, Docker Swarm, Azure"/>

<tags
   ms.service="container-service"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure"   
   ms.date="07/28/2016"
   ms.author="t-ribhat"/>

# 使用 Datadog 監視 Azure 容器服務叢集

本文中，我們會將 Datadog 代理程式部署到 Azure 容器服務叢集中的所有代理程式節點。您將需要 Datadog 帳戶以進行這項設定。

## 必要條件 

[部署](container-service-deployment.md)和[連接](container-service-connect.md) Azure 容器服務所設定的叢集。瀏覽 [Marathon UI](container-service-mesos-marathon-ui.md)。移至 [http://datadoghq.com](http://datadoghq.com) 以設定 Datadog 帳戶。

## Datadog 

Datadog 是一項監視服務，會從 Azure 容器服務叢集內的容器收集監視資料。Datadog 有 Docker 整合儀表板，可供您查看容器內的特定度量。從容器收集到的度量會依 CPU、記憶體、網路和 I/O 來加以整理。Datadog 會將度量分割成容器和映像。以下是 CPU 使用量之 UI 樣貌的範例。

![Datadog UI](./media/container-service-monitoring/datadog4.png)

## 使用 Marathon 設定 Datadog 部署

這些步驟將說明如何使用 Marathon 設定並將 Datadog 應用程式部署到您的叢集。

透過 [http://localhost:80/](http://localhost:80/) 存取 DC/OS UI。在進入 DC/OS UI 後，瀏覽至位於左下方的 [Universe]，然後搜尋「Datadog」並按一下 [安裝]。

![DC/OS Universe 內的 Datadog 封裝](./media/container-service-monitoring/datadog1.png)

現在，若要完成設定，您將需要 Datadog 帳戶或免費試用帳戶。登入 Datadog 網站後請看左邊，然後移至 [整合] -> [API]。

![Datadog API 金鑰](./media/container-service-monitoring/datadog2.png)

接下來將 API 金鑰輸入到 DC/OS Universe 中的 Datadog 設定。

![DC/OS Universe 中的 Datadog 設定](./media/container-service-monitoring/datadog3.png)

在上面的設定中，執行個體會設為 10000000，因此每當有新節點新增至叢集時，Datadog 就會自動將代理程式部署到該新節點。這是過渡解決方案。一旦您安裝了封裝，請瀏覽回到 Datadog 網站並尋找「儀表板」。 從該處，您會看到自訂和整合儀表板。Docker 整合儀表板會擁有為了監視叢集所需的所有容器度量。

<!---HONumber=AcomDC_0810_2016------>