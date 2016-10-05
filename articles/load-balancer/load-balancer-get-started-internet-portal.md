<properties
   pageTitle="使用 Azure 入口網站在 Resource Manager 中建立網際網路面向的負載平衡器 | Microsoft Azure"
   description="了解如何使用 Azure 入口網站在 Resource Manager 中建立網際網路面向的負載平衡器"
   services="load-balancer"
   documentationCenter="na"
   authors="anavinahar"
   manager="narayan"
   editor=""
   tags="azure-resource-manager"
/>
<tags
   ms.service="load-balancer"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="09/14/2016"
   ms.author="annahar" />

# 開始使用 Azure 入口網站建立網際網路面向的負載平衡器

[AZURE.INCLUDE [load-balancer-get-started-internet-arm-selectors-include.md](../../includes/load-balancer-get-started-internet-arm-selectors-include.md)]

[AZURE.INCLUDE [load-balancer-get-started-internet-intro-include.md](../../includes/load-balancer-get-started-internet-intro-include.md)]

[AZURE.INCLUDE [azure-arm-classic-important-include](../../includes/azure-arm-classic-important-include.md)] 本文涵蓋之內容包括資源管理員部署模型。您也可以[了解如何使用傳統部署建立網際網路面向的負載平衡器](load-balancer-get-started-internet-classic-portal.md)

[AZURE.INCLUDE [load-balancer-get-started-internet-scenario-include.md](../../includes/load-balancer-get-started-internet-scenario-include.md)]

這會說明必須完成才能建立負載平衡器的個別工作順序，並詳細說明若要達到此目標需要做些什麼。

## 若要建立網際網路面向的負載平衡器，需要哪些項目？

您需要建立和設定下列物件以部署負載平衡器。

- 前端 IP 組態 - 包含傳入網路流量的公用 IP 位址。

- 後端位址集區 - 包含虛擬機器的網路介面 (NIC)，可從負載平衡器接收網路流量。

- 負載平衡規則 - 包含將負載平衡器上的公用連接埠對應至後端位址集區中連接埠的規則。

- 輸入 NAT 規則 - 包含將負載平衡器上的公用連接埠對應至後端位址集區中特定虛擬機器之連接埠的規則。

- 探查 - 包含用來檢查後端位址集區中虛擬機器執行個體可用性的健全狀態探查。

您可以在 [Azure Resource Manager 的負載平衡器支援](load-balancer-arm.md)中取得關於負載平衡器元件與 Azure Resource Manager 的詳細資訊。


## 在 Azure 入口網站設定負載平衡器

> [AZURE.IMPORTANT] 此範例假設您擁有稱為 **myVNet** 的虛擬網路。請參閱[建立虛擬網路](../virtual-network/virtual-networks-create-vnet-arm-pportal.md)以便進行此作業。此範例也假設您有一個稱為 **LB-Subnet-BE** 的子網路 (位於 **myVNet** 內) 以及兩個分別稱為 **web1** 和 **web2** 的 VM (位於 **myVNet** 中稱為 **myAvailSet** 的相同可用性設定組內)。請參閱[此連結](../virtual-machines/virtual-machines-windows-hero-tutorial.md)以建立 VM。


1. 透過瀏覽器瀏覽至 Azure 入口網站 [http://portal.azure.com](http://portal.azure.com)，並使用您的 Azure 帳戶登入。

2. 在畫面左側選取 [新增] > [網路] > [負載平衡器]。

3. 在 [建立負載平衡器] 刀鋒視窗中，輸入負載平衡器的名稱。在此我們將其稱為 **myLoadBalancer**。

4. 在 [類型] 底下選取 [公用]。

5. 在 [公用 IP 位址] 底下建立稱為 **myPublicIP** 的新公用 IP。

6. 在 [資源群組] 底下選取 [myRG]。選取適當 [位置]，然後按一下 [確定]。負載平衡器接著會開始部署，並且需要幾分鐘的時間才能順利完成部署。

![更新負載平衡器的資源群組](./media/load-balancer-get-started-internet-portal/1-load-balancer.png)


## 建立後端位址集區

1. 順利部署負載平衡器之後，請從資源內加以選取。在 [設定] 底下選取 [後端集區]。輸入後端集區的名稱。然後按一下出現的刀鋒視窗中靠近上方的 [新增] 按鈕。

2. 按一下 [新增後端集區] 刀鋒視窗中的 [新增虛擬機器]。選取 [可用性設定組] 底下的 [選擇可用性設定組]，然後選取 [myAvailSet]。接下來，選取刀鋒視窗的 [虛擬機器] 區段底下的 [選擇虛擬機器]，然後按一下 [web1] 和 [web2]，隨即會建立兩個用於負載平衡的 VM。請確定這兩個 VM 左邊都有藍色核取記號，如下圖所示。然後，依序按一下該刀鋒視窗中的 [選取]、[選擇虛擬機器] 刀鋒視窗中的 [確定]，以及 [新增後端集區] 刀鋒視窗中的 [確定]。

    ![新增至後端位址集區](./media/load-balancer-get-started-internet-portal/3-load-balancer-backend-02.png)

3. 檢查確定您的通知下拉式清單中有關於儲存負載平衡器後端集區的更新，以及會更新 **web1** 和 **web2** 這兩部 VM 的網路介面。


## 建立探查、LB 規則和 NAT 規則

1. 建立健全狀況探查。

    在負載平衡器的 [設定] 底下選取 [探查]。然後按一下位於刀鋒視窗頂端的 [新增]。

    有兩種方式可以設定探查：HTTP 或 TCP。此範例示範 HTTP，但您可以透過類似方式設定 TCP。更新所需資訊。如前所述，**myLoadBalancer** 會對連接埠 80 上的流量進行負載平衡。選取的路徑為 HealthProbe.aspx、間隔為 15 秒，而狀況不良臨界值為 2。完成後，按一下 [確定] 以建立探查。

    將指標停留在 ‘i’ 圖示上可深入了解這些個別組態，以及如何變更它們以符合您的需求。

    ![新增探查](./media/load-balancer-get-started-internet-portal/4-load-balancer-probes.png)

2. 建立負載平衡器規則。

    按一下負載平衡器的 [設定] 區段中的負載平衡規則。在新的刀鋒視窗中按一下 [新增]。為規則命名。在這裡，其名稱為 HTTP。選擇前端連接埠和後端連接埠。在這裡，兩者皆選擇 80。選擇 [LB-backend] 做為 [後端集區]，並以先前建立的 [HealthProbe] 做為 [探查]。您可以根據需求設定其他組態。然後按一下 [確定] 以儲存負載平衡規則。

    ![新增負載平衡規則](./media/load-balancer-get-started-internet-portal/5-load-balancing-rules.png)

3. 建立輸入 NAT 規則

    按一下負載平衡器的 [設定] 區段底下的 [輸入 NAT 規則]。在新的刀鋒視窗中按一下 [新增]。然後為輸入 NAT 規則命名。在此我們將其稱為 **inboundNATrule1**。目的地應為先前建立的公用 IP。選取 [服務] 底下的 [自訂]，然後選取您要使用的通訊協定。此處選取 TCP。輸入連接埠 3441 和目標連接埠 (在此案例中) 3389。然後按一下 [確定] 以儲存此規則。

    在建立了第一個規則之後，從連接埠 3442 到目標連接埠 3389 針對稱為 inboundNATrule2 的第二個輸入 NAT 規則重複此步驟。

    ![新增輸入 NAT 規則](./media/load-balancer-get-started-internet-portal/6-load-balancer-inbound-nat-rules.png)

## 移除負載平衡器

若要刪除負載平衡器，請選取您要移除的負載平衡器。在 [負載平衡器] 刀鋒視窗中，按一下位於刀鋒視窗頂端的 [刪除]。然後在提示時選取 [是]。

## 後續步驟

[開始設定內部負載平衡器](load-balancer-get-started-ilb-arm-cli.md)

[設定負載平衡器分配模式](load-balancer-distribution-mode.md)

[設定負載平衡器的閒置 TCP 逾時設定](load-balancer-tcp-idle-timeout.md)

<!---HONumber=AcomDC_0921_2016-->