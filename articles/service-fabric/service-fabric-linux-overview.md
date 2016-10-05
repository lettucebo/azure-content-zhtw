<properties
   pageTitle="Linux 上的 Azure Service Fabric | Microsoft Azure"
   description="Service Fabric 叢集支援 Linux 和 Java，這表示您能夠在 Linux 上部署和裝載以 Java 撰寫的 Service Fabric 應用程式。"
   services="service-fabric"
   documentationCenter=".net"
   authors="mani-ramaswamy"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.devlang="Java"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="09/14/2016"
   ms.author="SubramaR"/>

# Linux 上的 Service Fabric

[這篇部落格文章](https://azure.microsoft.com/blog/service-fabric-on-linux-support-available-this-month/)宣佈預覽版將在 9 月 26 日公開發行。Linux 上的 Service Fabric 預覽版可讓您如同在 Windows 上一樣，在 Linux 上建置、部署和管理高可用性、可高度調整的應用程式。此外，高階的 Service Fabric 架構 (Reliable Services 和 Reliable Actors) 現在也能在 Linux 的 Java 中使用。

> [AZURE.VIDEO service-fabric-linux-preview]

## 支援的作業系統和程式設計語言

有限預覽版除了支援在 Azure 中執行 Ubuntu Server 16.04 的多部電腦叢集，還支援建立一整體開發叢集。

您可以使用任何語言或架構來建置[來賓可執行服務](service-fabric-deploy-existing-app.md)。除了協調 Docker 容器之外，您也可以使用 Java 或 C# 來建置以 Reliable Services 和 Reliable Actor 架構為基礎的服務。

>[AZURE.NOTE] Linux 尚未支援 Reliable Collections。

Linux 上的 Service Fabric 在概念上相當於 Windows 上的 Service Fabric (除了作業系統細節和程式設計語言支援)。因此，我們大部分的[現有文件](http://aka.ms/servicefabricdocs)都可協助您熟悉這項技術。

## 後續步驟

請熟悉 [Reliable Actors](service-fabric-reliable-actors-introduction.md) 和 [Reliable Services](service-fabric-reliable-services-introduction.md) 程式設計架構。

<!---HONumber=AcomDC_0921_2016-->