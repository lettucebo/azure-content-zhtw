<properties
   pageTitle="使用 Resource Manager 中的 Azure CLI 建立應用程式閘道 | Microsoft Azure"
   description="了解如何使用 Resource Manager 中的 Azure CLI 建立應用程式閘道"
   services="application-gateway"
   documentationCenter="na"
   authors="georgewallace"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"
/>
<tags  
   ms.service="application-gateway"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="09/09/2016"
   ms.author="gwallace" />

# 使用 Azure CLI 建立應用程式閘道

Azure 應用程式閘道是第 7 層負載平衡器。不論是在雲端或內部部署中，此閘道均提供在不同伺服器之間進行容錯移轉及效能路由傳送 HTTP 要求。應用程式閘道具有下列應用程式傳遞功能：HTTP 負載平衡、以 Cookie 為基礎的工作階段同質性、「安全通訊端層」(SSL) 卸載、自訂健康狀態探查，以及多站台支援。

> [AZURE.SELECTOR]
- [Azure 入口網站](application-gateway-create-gateway-portal.md)
- [Azure Resource Manager PowerShell](application-gateway-create-gateway-arm.md)
- [Azure 傳統 PowerShell](application-gateway-create-gateway.md)
- [Azure Resource Manager 範本](application-gateway-create-gateway-arm-template.md)
- [Azure CLI](application-gateway-create-gateway-cli.md)

## 必要條件：安裝 Azure CLI

若要執行本文的步驟，您需要[安裝適用於 Mac、Linux 和 Windows 的 Azure 命令列介面 (Azure CLI)](../xplat-cli-install.md)，而且需要[登入 Azure](../xplat-cli-connect.md)。

> [AZURE.NOTE] 如果您沒有 Azure 帳戶，就需要申請一個。請[在此處註冊免費試用](../active-directory/sign-up-organization.md)。

## 案例

在此案例中，您將了解如何使用 Azure 入口網站來建立應用程式閘道。

此案例將會：

- 建立含有兩個執行個體的中型應用程式閘道。
- 建立名為 AdatumAppGatewayVNET 且含有 10.0.0.0/16 保留 CIDR 區塊的虛擬網路。
- 建立名為 Appgatewaysubnet 且使用 10.0.0.0/28 做為其 CIDR 區塊的子網路。
- 為 SSL 卸載設定憑證。

![案例範例][scenario]

>[AZURE.NOTE] 其他應用程式閘道組態 (包括自訂健康狀況探查、後端集區位址及其他規則) 會在設定應用程式閘道設定之後才進行設定，而不會在初始部署期間設定。

## 開始之前

「Azure 應用程式閘道」需要有自己的子網路。建立虛擬網路時，請確定您保留足夠的位址空間，以便擁有多個子網路。將應用程式閘道部署到子網路之後，就只能將額外的應用程式閘道新增到該子網路。

## 登入 Azure

開啟 [Microsoft Azure 命令提示字元] 並登入。

    azure login

輸入上述範例後會提供程式碼。在瀏覽器中瀏覽至 https://aka.ms/devicelogin 以繼續登入程序。

![顯示裝置登入的 cmd][1]

在瀏覽器中，輸入收到的程式碼。系統會將您重新導向至 [登入] 頁面。

![要輸入程式碼的瀏覽器][2]

輸入程式碼後您已登入，請關閉瀏覽器以繼續進行案例。

![已順利登入][3]

## 切換至 Resource Manager 模式

    azure config mode arm

## 建立資源群組

建立應用程式閘道之前，會建立資源群組以包含應用程式閘道。以下顯示命令。

    azure group create -n AdatumAppGatewayRG -l eastus

## 建立虛擬網路

建立資源群組後，就會為應用程式閘道建立虛擬網路。在下列範例中，位址空間為上述案例注意事項中所定義的 10.0.0.0/16。

    azure network vnet create -n AdatumAppGatewayVNET -a 10.0.0.0/16 -g AdatumAppGatewayRG -l eastus

## 建立子網路

建立虛擬網路之後，就會為應用程式閘道新增子網路。如果您打算使用應用程式閘道搭配與其裝載於相同虛擬網路中的 Web 應用程式，請務必保留足夠的空間給另一個子網路使用。

    azure network vnet subnet create -g AdatumAppGatewayRG -n Appgatewaysubnet -v AdatumAppGatewayVNET -a 10.0.0.0/28 

## 建立應用程式閘道

建立虛擬網路和子網路後，便已完成應用程式閘道的先決條件。此外下列步驟也需要先前匯出的 .pfx 憑證和憑證密碼︰用於後端的 IP 位址是後端伺服器的 IP 位址。這些值可以是虛擬網路中的私人 IP、公用 IP 或後端伺服器的完整網域名稱。

    azure network application-gateway create -n AdatumAppGateway -l eastus -g AdatumAppGatewayRG -e AdatumAppGatewayVNET -m Appgatewaysubnet -r 134.170.185.46,134.170.188.221,134.170.185.50 -y c:\AdatumAppGateway\adatumcert.pfx -x P@ssw0rd -z 2 -a Standard_Medium -w Basic -j 443 -f Enabled -o 80 -i http -b https -u Standard



此範例會建立一個具有接聽程式、後端集區、後端 http 設定及規則之預設設定的基本應用程式閘道。也會設定 SSL 卸載。佈建成功之後，您可以依據您的部署需求修改這些設定。如果在先前步驟中已經以後端集區定義 Web 應用程式，一旦建立之後，負載平衡即開始。

## 後續步驟

參閱[建立自訂健康狀態探查](application-gateway-create-probe-portal.md)，以了解如何建立自訂健康狀態探查

參閱[設定 SSL 卸載](application-gateway-ssl-arm.md)，以了解如何設定「SSL 卸載」並將耗費資源的 SSL 解密從您的 Web 伺服器中移除

<!--Image references-->

[scenario]: ./media/application-gateway-create-gateway-cli/scenario.png
[1]: ./media/application-gateway-create-gateway-cli/figure1.png
[2]: ./media/application-gateway-create-gateway-cli/figure2.png
[3]: ./media/application-gateway-create-gateway-cli/figure3.png

<!---HONumber=AcomDC_0921_2016-->