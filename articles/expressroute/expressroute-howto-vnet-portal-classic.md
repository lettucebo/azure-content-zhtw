<properties
   pageTitle="在傳統入口網站中設定 ExpressRoute 的虛擬網路和閘道 | Microsoft Azure"
   description="本文將逐步引導您使用傳統部署模型和傳統入口網站來設定 ExpressRoute 的虛擬網路。"
   documentationCenter="na"
   services="expressroute"
   authors="cherylmc"
   manager="carmonm"
   editor=""
   tags="azure-service-management"/>

<tags 
   ms.service="expressroute"
   ms.devlang="na"
   ms.topic="article" 
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services" 
   ms.date="09/20/2016"
   ms.author="cherylmc"/>

# 在傳統入口網站中建立 ExpressRoute 的虛擬網路

本文中的步驟將引導您使用傳統部署模型與傳統入口網站來設定虛擬網路和虛擬網路閘道，以和 ExpressRoute 搭配使用。

如果您需要 Resource Manager 部署模型的指示，您可以使用下列文章：[使用 PowerShell 建立虛擬網路](../virtual-network/virtual-networks-create-vnet-arm-ps.md)及[將 VPN 閘道加入 ExpressRoute 的 Resource Manager VNet](expressroute-howto-add-gateway-resource-manager.md)。

**關於 Azure 部署模型**

[AZURE.INCLUDE [vpn-gateway-clasic-rm](../../includes/vpn-gateway-classic-rm-include.md)]

## 建立傳統 VNet 閘道

下列步驟會建立傳統 VNet 和虛擬網路閘道。如果您已經有傳統 VNet，請參閱本文中的[設定現有的傳統 VNet](#config) 一節。

1. 登入 [Azure 傳統入口網站](http://manage.windowsazure.com)。

2. 按一下螢幕左下角的 [新增]。在導覽窗格中依序按一下 [網路服務] 和 [虛擬網路]。按一下 [**自訂建立**] 開始組態精靈。

3. 在 [虛擬網路詳細資料] 頁面上，輸入下列資訊：

	- **名稱**：為虛擬網路命名。當您部署 VM 和 PaaS 執行個體時，將會使用此虛擬網路名稱，因此您可能不會想要太過複雜的名稱。
	- **位置**：位置會與您要存放資源 (VM) 的實體位置 (區域) 直接相關。例如，如果您要讓部署到此虛擬網路的 VM 實際上位於 East US 中，請選取該位置。建立關聯之後，您就無法變更與虛擬網路相關聯的區域。

4. 在 [DNS Servers and VPN Connectivity] 頁面上，輸入下列資訊，然後按一下右下角的下一步箭頭。

	- **DNS 伺服器** - 輸入 DNS 伺服器名稱和 IP 位址，或從捷徑功能表中選取先前註冊的 DNS 伺服器。此設定不會建立 DNS 伺服器。它可讓您指定要用於此虛擬網路之名稱解析的 DNS 服務。
	- **站對站連線能力** - 選取 [設定站對站 VPN] 核取方塊。
	- **ExpressRoute**：選取 [使用 ExpressRoute] 核取方塊。唯有在選取 [設定站對站 VPN] 時才會出現此選項。
	- **區域網路** - 您必須建立 ExpressRoute 的區域網路網站。但是，在 ExpressRoute 連接的案例中，指定用於區域網路網站的位址前置詞會被忽略。相反地，透過 ExpressRoute 電路向 Microsoft 通告的位址前置詞將供路由之用。<BR>如果您已為 ExpressRoute 連接建立區域網路，您可以從下拉式清單中選取。如果未建立，請選取 [指定新的區域網路]。

5. 如果您在上一個步驟中選取指定新的區域網路，則會顯示 [站對站連線能力] 頁面。若要設定區域網路，請輸入下列資訊，然後按下一步箭頭。

	- **名稱**：您想要命名區域 (內部部署) 網站的名稱。
	- **位址空間**：包括起始 IP 和 CIDR (位址計數)。您可以指定任何位址範圍，只要不會與虛擬網路的位址範圍重疊即可。一般而言，這會指定您的內部部署網路的位址範圍，但是在 ExpressRoute 案例中，不會使用這些設定。不過，需要這項設定才能在您使用傳統入口網站時建立區域網路。
	- **加入位址空間**：此設定與 ExpressRoute 無關。


6. 在 [Virtual Network Address Spaces] 頁面上，輸入下列資訊，然後按一下右下角的核取記號來設定您的網路。

	- **位址空間**：包括起始 IP 和位址計數。請確認您指定的位址空間沒有與您在區域網路上所擁有的任何位址空間重疊。
	- **新增子網路** - 包括起始 IP 和位址計數。不需要其他子網路。
	- **新增閘道器子網路**：按一下以新增閘道器子網路。閘道器子網路僅用於虛擬網路閘道，而且為這個組態的必要項目。<BR>ExpressRoute 的閘道子網路 CIDR (位址計數) 必須是 /28 或更大 (/27、/26 等等)。如此可讓該子網路中有足夠的 IP 位址，允許組態工作。在傳統入口網站中，如果您已選取核取方塊以使用 ExpressRoute，入口網站會以 /28 指定閘道子網路。您無法在傳統入口網站中調整 CIDR 位址計數。閘道子網路在傳統入口網站中會顯示為**閘道**，雖然建立的閘道子網路的實際名稱是 **GatewaySubnet**。您可以使用 PowerShell 或 Azure 入口網站來檢視此名稱。

7. 按一下頁面底部的核取記號，然後您的虛擬網路即會開始建立。完成時，您將在傳統入口網站的 [網路] 頁面上看到 [狀態] 下列出 [已建立]。

## <a name="gw"></a>建立閘道

1. 在 [網路] 頁面上按一下您剛才建立的虛擬網路，然後在頁面頂端按一下 [儀表板]。

2. 按一下 [儀表板] 頁面底部的 [建立閘道]，然後選取 [動態路由]。按一下 [是] 確認要建立閘道。

3. 當閘道器建立開始時，系統會顯示訊息來通知您閘道器已啟動。閘道建立作業最多可能需要花費 45 分鐘。

4. 將網路連結至線路。請依照[如何將 VNet 連結至 ExpressRoute 電路](expressroute-howto-linkvnet-classic.md)文章中的指示進行。

## <a name="config"></a>設定 ExpressRoute 的現有傳統 VNet

如果您已經有傳統 VNet，您可以設定它以在傳統入口網站中連接到 ExpressRoute。設定與上述章節中的設定相同，因此請詳閱這些章節以熟悉必要設定。如果您想要建立 ExpressRoute/站對站並存連接，請參閱[這篇文章](expressroute-howto-coexist-classic.md)以取得步驟。它們與這篇文章的步驟不同。
 
1. 您必須在更新其他 VNet 設定之前先建立區域網路。若要建立新的區域網路 (透過傳統入口網站設定 ExpressRoute 時為必要項目)，則按一下 [新增] **>** [網絡服務] **>** [虛擬網路] **>** [新增區域網路]。遵循精靈步驟來建立區域網路。

2. 使用 [設定] 頁面更新其餘的 VNet 設定，並且將 VNet 關聯至區域網路。

3. 設定完成後，請移至本文的[建立閘道](#gw)區段，以建立閘道。


## 後續步驟

- 如果您想要將虛擬機器新增至虛擬網路，請參閱[虛擬機器學習路徑](https://azure.microsoft.com/documentation/learning-paths/virtual-machines/)。
- 如果您想要深入了解 ExpressRoute，請參閱 [ExpressRoute 概觀](expressroute-introduction.md)。


 

<!---HONumber=AcomDC_0921_2016-->