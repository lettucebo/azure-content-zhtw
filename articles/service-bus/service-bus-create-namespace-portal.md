<properties
    pageTitle="使用 Azure 入口網站建立服務匯流排命名空間 | Microsoft Azure"
    description="若要開始使用服務匯流排，您需要命名空間。以下會說明如何使用 Azure 入口網站建立命名空間。"
    services="service-bus"
    documentationCenter=".net"
    authors="jtaubensee"
    manager="timlt"
    editor=""/>

<tags
    ms.service="service-bus"
    ms.devlang="tbd"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="dotnet"
    ms.workload="na"
    ms.date="08/22/2016"
    ms.author="jotaub"/>

# 使用 Azure 入口網站建立服務匯流排命名空間

命名空間是所有訊息元件的共同容器。多個佇列和主題可以位於單一命名空間，而且命名空間通常會做為應用程式容器。目前有 2 個不同的方式可建立服務匯流排命名空間。

1.	Azure 入口網站 (本文)

2.	[Resource Manager 範本][create-namespace-using-arm]

## 在 Azure 入口網站中建立命名空間

[AZURE.INCLUDE [service-bus-create-namespace-portal](../../includes/service-bus-create-namespace-portal.md)]

恭喜！ 您現已建立服務匯流排訊息命名空間。

## 後續步驟

查看 [GitHub 儲存機制](https://github.com/Azure-Samples/azure-servicebus-messaging-samples][github-samples)，其中的範例會展示一些更進階的 Azure 服務匯流排訊息功能。

[create-namespace-using-arm]: ../service-bus-messaging/service-bus-resource-manager-overview.md
[github-samples]: https://github.com/Azure-Samples/azure-servicebus-messaging-samples

<!---HONumber=AcomDC_0928_2016-->