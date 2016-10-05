<properties
   pageTitle="Service Fabric 反向 Proxy | Microsoft Azure"
   description="使用 Service Fabric 反向 Proxy 從叢集內部和外部與微服務進行通訊"
   services="service-fabric"
   documentationCenter=".net"
   authors="BharatNarasimman,vturecek"
   manager="timlt"
   editor="vturecek"/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="required"
   ms.date="07/26/2016"
   ms.author="vturecek"/>

# Service Fabric 反向 Proxy

Service Fabric 反向 Proxy 是內建到 Service Fabric 的反向 Proxy，可讓您處理 Service Fabric 叢集中公開 HTTP 端點的微服務。

## 微服務通訊模型

Service Fabric 中的微服務通常在叢集中的 VM 子集上執行，而且會因為各種原因在不同的 VM 之間移動。因此，微服務的端點可以動態變更。與微服務通訊的一般模式是如下的解析迴圈：

1. 起初透過命名服務解析服務位置。
2. 連線至服務。
3. 判斷連線失敗的原因，必要時再重新解析服務位置。

此程序通常涉及將用戶端通訊程式庫包裝在重試迴圈，以實作服務解析再重試原則。如需有關本主題的詳細資訊，請參閱[與服務進行通訊](service-fabric-connect-and-communicate-with-services.md)。

### 透過 SF 反向 Proxy 進行通訊
Service Fabric 反向 Proxy 會在叢集的所有節點上執行。它代表用戶端執行整個服務的解析程序，接著再轉送用戶端要求。因此，在叢集上執行的用戶端只要使用任一用戶端 HTTP 通訊程式庫，就能透過同一節點本端上執行的 SF 反向 Proxy 與目標服務通訊。

![內部通訊][1]

## 從叢集外部連線到微服務
微服務的預設外部通訊模型是 [選擇加入]，每項服務預設無法從外部用戶端直接存取。[Azure Load Balancer](../load-balancer/load-balancer-overview.md) 是微服務與外部用戶端之間的網路界限，負責執行網路位址轉譯，並將外部要求轉送給內部的 **IP:port** 端點。為了讓微服務的端點能直接存取外部用戶端，Azure Load Balancer 必須先設為轉送流量到叢集服務使用的每個連接埠。此外，大部分的微服務 (尤其是可設定狀態微服務) 不存在叢集的所有節點上，而且在容錯移轉時可於節點之間移動，在這種情況下，Azure Load Balancer 無法有效地判斷要轉送流量的複本目標節點位於何處。

### 從叢集外部透過 SF 反向 Proxy 連線到微服務

您不需要在 Azure Load Balancer 設定個別服務的連接埠，只需在 Azure 負載平衡器設定 SF 反向 Proxy 連接埠。這樣可允許叢集外部的用戶端透過反向 Proxy 連線到叢集內部的服務，而不需要額外設定。

![外部通訊][0]

>[AZURE.WARNING] 在負載平衡器上設定反向 Proxy 的連接埠，將叢集中公開 http 端點的所有微服務設為可從叢集外部尋址。


## 透過反向 Proxy 定址服務的 URI 格式

反向 Proxy 使用特定的 URI 格式來識別連入要求應該轉送到哪個服務資料分割︰

```
http(s)://<Cluster FQDN | internal IP>:Port/<ServiceInstanceName>/<Suffix path>?PartitionKey=<key>&PartitionKind=<partitionkind>&Timeout=<timeout_in_seconds>
```

 - **http(s)：** 反向 Proxy 可設定為接受 HTTP 或 HTTPS 流量。如果是 HTTPS 流量，SSL 終止會發生在反向 Proxy。反向 Proxy 是透過 HTTP 轉送要求到叢集中的服務。
 - **叢集的 FQDN| internal IP:** For external clients, the reverse proxy can be configured so that it is reachable through the cluster domain (e.g., mycluster.eastus.cloudapp.azure.com). By default the reverse proxy runs on every node, so for internal traffic it can be reached on localhost or on any internal node IP (e.g., 10.0.0.1).
 - **連接埠︰**為反向 Proxy 指定的連接埠。例如 19008。
 - **ServiceInstanceName：**這是您嘗試連線到 "fabric: /" 配置的 SANS 服務的完整部署服務執行個體名稱 。例如，若要連線到服務 fabric: / myapp/myservice/，可以使用 myapp/myservice。
 - **尾碼路徑︰**這是要連線服務的實際 URL 路徑。例如，myapi/values/add/3
 - **PartitionKey：**若為資料分割服務，這是您想要連線的資料分割計算資料分割金鑰。請注意，這不是資料分割識別碼 GUID。使用單一資料分割配置的服務不需要這個參數。
 - **PartitionKind：**服務資料分割配置。這可以是 'Int64Range' 或 'Named'。使用單一資料分割配置的服務不需要這個參數。
 - **逾時︰**這會指定反向 Proxy 建立的 http 要求代替用戶端要求傳送到服務的逾時。預設值為 60 秒。這是選擇性參數。

### 使用方式範例

例如，我們接受在下列 URL 上開啟 HTTP 接聽程式的服務 **fabric:/MyApp/MyService**：

```
http://10.0.05:10592/3f0d39ad-924b-4233-b4a7-02617c6308a6-130834621071472715/
```

使用下列資源：

 - `/index.html`
 - `/api/users/<userId>`

如果服務使用單一資料分割配置，則不需要 PartitionKey 和 PartitionKind 查詢字串參數，可透過閘道以下列方式連線到服務︰

 - 外部：`http://mycluster.eastus.cloudapp.azure.com:19008/MyApp/MyService`
 - 內部：`http://localhost:19008/MyApp/MyService`

如果服務使用統一 Int64 資料分割配置，則必須使用 PartitionKey 和 PartitionKind 查詢字串參數來連線到服務的資料分割︰

 - 外部：`http://mycluster.eastus.cloudapp.azure.com:19008/MyApp/MyService?PartitionKey=3&PartitionKind=Int64Range`
 - 內部：`http://localhost:19008/MyApp/MyService?PartitionKey=3&PartitionKind=Int64Range`

若要連線到服務公開的資源，只要將資源路徑放在 URL 的服務名稱之後︰

 - 外部：`http://mycluster.eastus.cloudapp.azure.com:19008/MyApp/MyService/index.html?PartitionKey=3&PartitionKind=Int64Range`
 - 內部：`http://localhost:19008/MyApp/MyService/api/users/6?PartitionKey=3&PartitionKind=Int64Range`

閘道會將這些要求轉送到服務的 URL：

 - `http://10.0.05:10592/3f0d39ad-924b-4233-b4a7-02617c6308a6-130834621071472715/index.html`
 - `http://10.0.05:10592/3f0d39ad-924b-4233-b4a7-02617c6308a6-130834621071472715/api/users/6`

## 連接埠共用服務特殊處理

應用程式閘道會嘗試重新解析服務位址，並在無法連線到服務時重試要求。這是閘道的主要好處之一，因為用戶端程式碼不需要實作自己的服務解決方案和解析迴圈。

通常，當服務無法連線時，即表示服務執行個體或複本已移至另一個節點，這是其一般生命週期的過程。發生這種情況時，閘道可能會收到指出端點已不在原先解析的位址上開啟的網路連線錯誤。

不過，複本或服務執行個體可以共用主機處理序，在由以 http.sys 為基礎的 Web 伺服器主控時也可以共用連接埠，這些 Web 伺服器包括︰

 - [System.Net.HttpListener](https://msdn.microsoft.com/library/system.net.httplistener%28v=vs.110%29.aspx)
 - [ASP.NET Core WebListener](https://docs.asp.net/latest/fundamentals/servers.html#weblistener)
 - [Katana](https://www.nuget.org/packages/Microsoft.AspNet.WebApi.OwinSelfHost/)

在此情況下，可能是 Web 伺服器可用於主機處理序和回應要求，但已解析的服務執行個體或複本已無法再於主機上提供。在此情況下，閘道將從 Web 伺服器收到 HTTP 404 回應。因此 HTTP 404 有兩個不同意義︰

 1. 服務位址正確，但使用者所要求的資源不存在。
 2. 服務位址不正確，而且使用者所要求的資源實際上可能存在不同的節點上。

在第一個案例中，這是一般的 HTTP 404，會被視為使用者錯誤。不過，在第二個案例中，使用者要求的資源存在，但服務本身已被移動因此閘道無法找到，在此情況下，閘道必須重新解析位址並重試。

因此閘道需要能夠區分這兩種案例的方法。為了區分，則需要來自伺服器的提示。

 - 根據預設，應用程式閘道會假設是案例 #2，並嘗試重新解析並重新發出要求。
 - 若要向應用程式閘道表明是案例 #1，服務應傳回下列 HTTP 回應標頭︰

`X-ServiceFabric : ResourceNotFound`

此 HTTP 回應標頭表示指出一般的 HTTP 404 情況，即要求的資源不存在，閘道將不會嘗試重新解析服務位址。

## 設定和組態
您可以透過 [Azure Resource Manager 範本](./service-fabric-cluster-creation-via-arm.md)啟用叢集的 Service Fabric 反向 Proxy。

當您取得想部署的叢集範本後 (透過範例範本或建立自訂的 Resource Manager 範本)，即可透過下列步驟在範本中啟用反向 Proxy。

1. 在範本的[參數區段](../resource-group-authoring-templates.md)定義反向 Proxy 的連接埠。

    ```json
    "SFReverseProxyPort": {
        "type": "int",
        "defaultValue": 19008,
        "metadata": {
            "description": "Endpoint for Service Fabric Reverse proxy"
        }
    },
    ```
2. 在**叢集** [resources type 區段](../resource-group-authoring-templates.md)中為每個節點類型物件指定連接埠

    ```json
    {
        "apiVersion": "2016-03-01",
        "type": "Microsoft.ServiceFabric/clusters",
        "name": "[parameters('clusterName')]",
        "location": "[parameters('clusterLocation')]",
        ...
       "nodeTypes": [
          {
           ...
           "httpApplicationGatewayEndpointPort": "[parameters('SFReverseProxyPort')]",
           ...
          },
        ...
        ],
        ...
    }
    ```
3. 若要從 Azure 叢集外部定址反向 Proxy，請為步驟 1 指定的連接埠設定 **Azure Load Balancer 規則**。

    ```json
    {
        "apiVersion": "[variables('lbApiVersion')]",
        "type": "Microsoft.Network/loadBalancers",
        ...
        ...
        "loadBalancingRules": [
            ...
            {
                "name": "LBSFReverseProxyRule",
                "properties": {
                    "backendAddressPool": {
                        "id": "[variables('lbPoolID0')]"
                    },
                    "backendPort": "[parameters('SFReverseProxyPort')]",
                    "enableFloatingIP": "false",
                    "frontendIPConfiguration": {
                        "id": "[variables('lbIPConfig0')]"
                    },
                    "frontendPort": "[parameters('SFReverseProxyPort')]",
                    "idleTimeoutInMinutes": "5",
                    "probe": {
                        "id": "[concat(variables('lbID0'),'/probes/SFReverseProxyProbe')]"
                    },
                    "protocol": "tcp"
                }
            }
        ],
        "probes": [
            ...
            {
                "name": "SFReverseProxyProbe",
                "properties": {
                    "intervalInSeconds": 5,
                    "numberOfProbes": 2,
                    "port":     "[parameters('SFReverseProxyPort')]",
                    "protocol": "tcp"
                }
            }  
        ]
    }
    ```
4. 若要在連接埠上設定反向 Proxy 的 SSL 憑證，請將憑證新增至**叢集** [resources type 區段](../resource-group-authoring-templates.md)中的 httpApplicationGatewayCertificate 屬性。

    ```json
    {
        "apiVersion": "2016-03-01",
        "type": "Microsoft.ServiceFabric/clusters",
        "name": "[parameters('clusterName')]",
        "location": "[parameters('clusterLocation')]",
        "dependsOn": [
            "[concat('Microsoft.Storage/storageAccounts/', parameters('supportLogStorageAccountName'))]"
        ],
        "properties": {
            ...
            "httpApplicationGatewayCertificate": {
                "thumbprint": "[parameters('sfReverseProxyCertificateThumbprint')]",
                "x509StoreName": "[parameters('sfReverseProxyCertificateStoreName')]"
            },
            ...
            "clusterState": "Default",
        }
    }
    ```

## 後續步驟
 - 請參閱 [GitHUb 上的範例專案](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started/tree/master/Services/WordCount)中服務之間的 HTTP 通訊範例。

 - [使用 Reliable Services 遠端服務進行遠端程序呼叫](service-fabric-reliable-services-communication-remoting.md)

 - [在 Reliable Services 中使用 OWIN 的 Web API](service-fabric-reliable-services-communication-webapi.md)

 - [使用 Reliable Services 的 WCF 通訊](service-fabric-reliable-services-communication-wcf.md)


[0]: ./media/service-fabric-reverseproxy/external-communication.png
[1]: ./media/service-fabric-reverseproxy/internal-communication.png

<!---HONumber=AcomDC_0921_2016-->