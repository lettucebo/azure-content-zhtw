<properties
   pageTitle="應用程式閘道 WebSocket 支援 | Microsoft Azure"
   description="本頁面提供應用程式閘道 WebSocket 支援的概觀。"
   documentationCenter="na"
   services="application-gateway"
   authors="amsriva"
   manager="rossort"
   editor="amsriva"/>
<tags
   ms.service="application-gateway"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="09/16/2016"
   ms.author="amsriva"/>

# 應用程式閘道 WebSocket 支援

以 [RFC6455](https://tools.ietf.org/html/rfc6455) 標準化的 WebSocket 通訊協定可透過長時間執行的 TCP 連線讓伺服器和用戶端之間能夠進行全雙工通訊。此功能可讓 Web 伺服器和用戶端之間進行互動性更高的通訊，此通訊可以是雙向的，而不需要像 HTTP 型實作所要求的進行輪詢。不同於 HTTP，WebSocket 的負荷很低，而且可以對多個要求/回應重複使用相同的 TCP 連線，進而提升資源使用效率。WebSocket 通訊協定設計為透過傳統 HTTP 連接埠 80 和 443 進行運作。

應用程式閘道可跨所有閘道大小對 WebSocket 提供原生支援。使用者無法進行設定來選擇要啟用或停用 WebSocket 支援。您可以在連接埠 80/443 上繼續使用標準 HTTPListener 來接收 WebSocket 流量。WebSocket 流量接著會使用應用程式閘道規則中指定的適當後端集區，來導向到已啟用 WebSocket 的後端伺服器。

後端伺服器必須回應應用程式閘道探查，[健全狀態探查概觀](application-gateway-probe-overview.md)一節中會有所說明。應用程式閘道健全狀態探查僅限使用 HTTP/HTTPS，這表示每一部後端伺服器都必須回應 HTTP 探查，應用程式閘道才能將 WebSocket 流量路由傳送至伺服器。


## 接聽程式組態元素

現有的 HTTPListener 可用來支援 WebSocket。以下是來自範例範本檔案中 HttpListeners 元素的程式碼片段。您必須同時擁有 HTTP 和 HTTPS 接聽程式才能支援 WebSocket 和保護 WebSocket 流量。同樣地，您可以使用[入口網站](application-gateway-create-gateway-portal.md)或 [PowerShell](application-gateway-create-gateway-arm.md) 在連接埠 80/443 上建立具有接聽程式的應用程式閘道，才能支援 WebSocket 流量。


    "httpListeners": [
                {
                    "name": "appGatewayHttpsListener",
                    "properties": {
                        "FrontendIPConfiguration": {
                            "Id": "/subscriptions/<subid>/resourceGroups/<rgName>/providers/Microsoft.Network/applicationGateways/applicationGateway1/frontendIPConfigurations/DefaultFrontendPublicIP"
                        },
                        "FrontendPort": {
                            "Id": "/subscriptions/<subid>/resourceGroups/<rgName>/providers/Microsoft.Network/applicationGateways/applicationGateway1/frontendPorts/appGatewayFrontendPort443'"
                        },
                        "Protocol": "Https",
                        "SslCertificate": {
                            "Id": "/subscriptions/<subid>/resourceGroups/<rgName>/providers/Microsoft.Network/applicationGateways/applicationGateway1/sslCertificates/appGatewaySslCert1'"
                        },
                    }
                },
                {
                    "name": "appGatewayHttpListener",
                    "properties": {
                        "FrontendIPConfiguration": {
                            "Id": "/subscriptions/<subid>/resourceGroups/<rgName>/providers/Microsoft.Network/applicationGateways/applicationGateway1/frontendIPConfigurations/appGatewayFrontendIP'"
                        },
                        "FrontendPort": {
                            "Id": "/subscriptions/<subid>/resourceGroups/<rgName>/providers/Microsoft.Network/applicationGateways/applicationGateway1/frontendPorts/appGatewayFrontendPort80'"
                        },
                        "Protocol": "Http",
                    }
                }
            ],

## BackendAddressPool、BackendHttpSetting 和路由規則組態

若後端集區具有已啟用 WebSocket 的伺服器，應以 BackendAddressPool 加以定義。BackendHttpSetting 只應該使用後端連接埠 80/443 進行定義。Cookie 型同質性和 requestTimeouts 的屬性與 WebSocket 流量無關。無須在路由規則中進行任何變更。應該繼續使用路由規則 'Basic'，以將適當的接聽程式繫結到對應的後端位址集區。

	"requestRoutingRules": [{
		"name": "<ruleName1>",
		"properties": {
			"RuleType": "Basic",
			"httpListener": {
				"id": "/subscriptions/<subid>/resourceGroups/<rgName>/providers/Microsoft.Network/applicationGateways/applicationGateway1/httpListeners/appGatewayHttpsListener')]"
			},
			"backendAddressPool": {
				"id": "/subscriptions/<subid>/resourceGroups/<rgName>/providers/Microsoft.Network/applicationGateways/applicationGateway1/backendAddressPools/ContosoServerPool')]"
			},
			"backendHttpSettings": {
				"id": "/subscriptions/<subid>/resourceGroups/<rgName>/providers/Microsoft.Network/applicationGateways/applicationGateway1/backendHttpSettingsCollection/appGatewayBackendHttpSettings')]"
			}
		}

	}, {
		"name": "<ruleName2>",
		"properties": {
			"RuleType": "Basic",
			"httpListener": {
				"id": "/subscriptions/<subid>/resourceGroups/<rgName>/providers/Microsoft.Network/applicationGateways/applicationGateway1/httpListeners/appGatewayHttpListener')]"
			},
			"backendAddressPool": {
				"id": "/subscriptions/<subid>/resourceGroups/<rgName>/providers/Microsoft.Network/applicationGateways/applicationGateway1/backendAddressPools/ContosoServerPool')]"
			},
			"backendHttpSettings": {
				"id": "/subscriptions/<subid>/resourceGroups/<rgName>/providers/Microsoft.Network/applicationGateways/applicationGateway1/backendHttpSettingsCollection/appGatewayBackendHttpSettings')]"
			}

		}
	}]

## 已啟用 WebSocket 的後端

後端必須在設定的連接埠 (通常是 80/443) 上執行 HTTP/HTTPS Web 伺服器，WebSocket 才能運作。此需求是因為 WebSocket 通訊協定要求初始交握必須是 HTTP，並以升級為 WebSocket 通訊協定做為標頭欄位。

	GET /chat HTTP/1.1
    Host: server.example.com
    Upgrade: websocket
    Connection: Upgrade
    Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
    Origin: http://example.com
    Sec-WebSocket-Protocol: chat, superchat
    Sec-WebSocket-Version: 13

另一個原因是該應用程式閘道後端健全狀態探查只支援 HTTP/HTTPS 通訊協定。如果後端伺服器未回應 HTTP/HTTPS 探查，就會將它移出後端集區，而且包括 WebSocket 要求在內的要求都將無法到達此後端。

## 後續步驟

在了解 WebSocket 支援之後，請移至[建立應用程式閘道](application-gateway-create-gateway.md)以開始使用已啟用 WebSocket 的 Web 應用程式。

<!---HONumber=AcomDC_0921_2016-->