<properties
   pageTitle="使用入口網站建立應用程式閘道 | Microsoft Azure"
   description="了解如何使用入口網站來建立「應用程式閘道」"
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

# 使用入口網站建立應用程式閘道

Azure 應用程式閘道是第 7 層負載平衡器。不論是在雲端或內部部署中，此閘道均提供在不同伺服器之間進行容錯移轉及效能路由傳送 HTTP 要求。應用程式閘道提供許多應用程式傳遞控制器 (ADC) 功能，包括 HTTP 負載平衡、以 Cookie 為基礎的工作階段同質性、安全通訊端層 (SSL) 卸載、自訂健全狀態探查、多網站支援，以及許多其他功能。若要尋找完整的支援功能清單，請瀏覽[應用程式閘道概觀](application-gateway-introduction.md)

> [AZURE.SELECTOR]
- [Azure 入口網站](application-gateway-create-gateway-portal.md)
- [Azure Resource Manager PowerShell](application-gateway-create-gateway-arm.md)
- [Azure 傳統 PowerShell](application-gateway-create-gateway.md)
- [Azure Resource Manager 範本](application-gateway-create-gateway-arm-template.md)
- [Azure CLI](application-gateway-create-gateway-cli.md)

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

## 建立應用程式閘道

### 步驟 1

瀏覽至 Azure 入口網站，按一下 [新增] > [網路] > [應用程式閘道]。

![建立應用程式閘道][1]

### 步驟 2

接著，填入應用程式閘道的相關基本資訊。完成時，按一下 [確定]。

基本設定所需的資訊為︰

- **名稱** - 應用程式閘道的名稱。
- **SKU 大小** - 此設定是應用程式閘道的大小，可用的選項為 (小型、中型及大型)。
- **執行個體計數** - 執行個體數目，此值應該是介於 2 到 10 之間的數字。
- **資源群組** - 保存應用程式閘道的資源群組，它可以是現有的資源群組或新的資源群組。
- **位置** - 應用程式閘道的區域，它是與資源群組所在相同的位置。位置相當重要，因為虛擬網路和公用 IP 必須與閘道位於相同位置。

![顯示基本設定的刀鋒視窗][2]

>[AZURE.NOTE] 若要進行測試，可以選擇執行個體計數 1。請務必了解 SLA 不涵蓋任何低於兩個執行個體的執行個體計數，因此不建議使用。小型閘道適用於開發測試，不適合在生產環境中使用。

### 步驟 3

定義基本設定之後，下一步是定義要使用的虛擬網路。虛擬網路會裝載應用程式閘道為其進行負載平衡的應用程式。

請按一下 [選擇虛擬網路] 來設定虛擬網路。

![顯示應用程式閘道設定的刀鋒視窗][3]

### 步驟 4

在 [選擇虛擬網路] 刀鋒視窗中，按一下 [建立新項目]。

雖然此案例中未說明，但是您可以在此時選取現有的「虛擬網路」。如果使用現有的虛擬網路，請務必了解虛擬網路需要空的子網路或僅限應用程式閘道資源的子網路，才能加以使用。

![[選擇虛擬網路] 刀鋒視窗][4]

### 步驟 5

在 [建立虛擬網路] 刀鋒視窗中，如先前的[案例](#scenario)說明所述，填入網路資訊。

![已輸入資訊的 [建立虛擬網路] 刀鋒視窗][5]

### 步驟 6

建立虛擬網路之後，下一步是定義應用程式閘道的前端 IP。此時，可以為前端選擇公用或私人 IP 位址。選擇取決於應用程式是網際網路對向還是僅供內部使用。此案例假設使用公用 IP 位址。若要選擇的私人 IP 位址，可以按一下 [私人] 按鈕。系統會選擇自動指派的 IP 位址，或者您也可以按一下 [選擇特定的私人 IP 位址] 核取方塊來手動輸入一個位址。

### 步驟 7

按一下 [選擇公用 IP 位址]。如果有現有的公用 IP 位址可用，此時便可選擇它，在此案例中，您會建立新的公用 IP 位址。按一下 [建立新項目]。

![[選擇公用 IP 位址] 刀鋒視窗][6]

### 步驟 8

接著，為公用 IP 位址提供一個易記的名稱，然後按一下 [確定]。

![[建立公用 IP 位址] 刀鋒視窗][7]

### 步驟 9

建立應用程式閘道時，所要設定的最後一項設定就是接聽程式組態。如果使用的是 **http**，則沒有其他要設定的剩餘項目，您可以按一下 [確定]。若要使用 **https**，則需要設定進一步的組態。

若要使用 **https**，必須提供憑證。需要有憑證的私密金鑰，因此必須提供憑證的 .pfx 匯出及檔案的密碼。


![[設定] 刀鋒視窗上空白的 [接聽程式組態] 區段][8]

### 步驟 10

按一下 [HTTPS]，再按一下 [上傳 PFX 憑證] 文字方塊旁邊的 [資料夾] 圖示。瀏覽至您檔案系統上的.pfx 憑證檔案。選取憑證檔案之後，請為憑證提供一個易記的名稱，然後輸入 .pfx 檔案的密碼。

完成之後，請按一下 [確定] 以檢閱「應用程式閘道」的設定。

![[設定] 刀鋒視窗上的 [接聽程式組態] 區段][9]

### 步驟 11

檢閱 [摘要] 頁面，然後按一下 [確定]。現在應用程式閘道已排入佇列並建立。

### 步驟 12

建立應用程式閘道之後，請在入口網站中瀏覽至該閘道，以繼續設定應用程式閘道組態。

![應用程式閘道資源檢視][10]

這些步驟會建立一個具有接聽程式、後端集區、後端 http 設定及規則之預設設定的基本應用程式閘道。佈建成功之後，您可以依據您的部署需求修改這些設定。

## 後續步驟

參閱[建立自訂健康狀態探查](application-gateway-create-probe-portal.md)，以了解如何建立自訂健康狀態探查

參閱[設定 SSL 卸載](application-gateway-ssl-portal.md)，以了解如何設定「SSL 卸載」並將耗費資源的 SSL 解密從您的 Web 伺服器中移除

<!--Image references-->
[1]: ./media/application-gateway-create-gateway-portal/figure1.png
[2]: ./media/application-gateway-create-gateway-portal/figure2.png
[3]: ./media/application-gateway-create-gateway-portal/figure3.png
[4]: ./media/application-gateway-create-gateway-portal/figure4.png
[5]: ./media/application-gateway-create-gateway-portal/figure5.png
[6]: ./media/application-gateway-create-gateway-portal/figure6.png
[7]: ./media/application-gateway-create-gateway-portal/figure7.png
[8]: ./media/application-gateway-create-gateway-portal/figure8.png
[9]: ./media/application-gateway-create-gateway-portal/figure9.png
[10]: ./media/application-gateway-create-gateway-portal/figure10.png
[scenario]: ./media/application-gateway-create-gateway-portal/scenario.png

<!---HONumber=AcomDC_0921_2016-->