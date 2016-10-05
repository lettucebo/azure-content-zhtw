<properties
   pageTitle="使用 Sysdig 監視 Azure 容器服務叢集 | Microsoft Azure"
   description="使用 Sysdig 監視 Azure 容器服務叢集。"
   services="container-service"
   documentationCenter=""
   authors="rbitia"
   manager="timlt"
   editor=""
   tags="acs, azure-container-service"
   keywords="容器，DC/OS，Azure"/>

<tags
   ms.service="container-service"
   ms.devlang="na"
   ms.topic="get-started-article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="08/08/2016"
   ms.author="t-ribhat"/>

# 使用 Sysdig 監視 Azure 容器服務叢集

本文中，我們會將 Sysdig 代理程式部署到 Azure 容器服務叢集中的所有代理程式節點。您需要 Sysdig 帳戶以進行這項設定。

## 必要條件 

[部署](container-service-deployment.md)和[連接](container-service-connect.md) Azure 容器服務所設定的叢集。瀏覽 [Marathon UI](container-service-mesos-marathon-ui.md)。移至 [http://app.sysdigcloud.com](http://app.sysdigcloud.com) 設定 Sysdig 雲端帳戶。

## Sysdig

Sysdig 是一項監視服務，可讓您在叢集內監視您的容器。Sysdig 以協助疑難排解而知名，不過 Sysdig 也具有針對 CPU、網路功能、記憶體和 I/O 的基本監視計量。Sysdig 讓您更容易查看哪些容器作用最密集或基本上使用最多記憶體和 CPU。此檢視位於 [概觀] 一節中，目前是 Beta 版。

![Sysdig UI](./media/container-service-monitoring-sysdig/sysdig6.png)

## 使用 Marathon 設定 Sysdig 部署

這些步驟將說明如何使用 Marathon 設定並將 Sysdig 應用程式部署到您的叢集。

透過 [http://localhost:80/](http://localhost:80/) 存取 DC/OS UI。在 DC/OS UI 中瀏覽至位於左下方的「Universe」，然後搜尋「Sysdig」。

![DC/OS Universe 中的 Sysdig](./media/container-service-monitoring-sysdig/sysdig1.png)

現在，若要完成設定需要 Sysdig 雲端帳戶或免費試用帳戶。登入至 Sysdig 雲端網站後，按一下使用者名稱，在頁面上應該會看到「存取金鑰」。

![Sysdig API 金鑰](./media/container-service-monitoring-sysdig/sysdig2.png)

接下來將存取金鑰輸入到 DC/OS Universe 中的 Sysdig 設定。

![DC/OS Universe 中的 Sysdig 設定](./media/container-service-monitoring-sysdig/sysdig3.png)

現在將執行個體設為 10000000，每當一個新節點新增至叢集時，Sysdig 會自動將代理程式部署到該新節點。這是過渡性解決方案，用以確保 Sysdig 會部署到叢集內的所有新代理程式。

![DC/OS Universe 中的 Sysdig 設定 - 執行個體](./media/container-service-monitoring-sysdig/sysdig4.png)

安裝套件之後回到 Sysdig UI，您就可以瀏覽叢集內不同的容器使用情況計量。

<!---HONumber=AcomDC_0810_2016--->