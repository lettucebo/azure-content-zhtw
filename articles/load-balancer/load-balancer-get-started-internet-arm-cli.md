<properties 
   pageTitle="使用 Azure CLI 在資源管理員中建立網際網路面向的負載平衡器 | Microsoft Azure"
   description="了解如何使用 Azure CLI 在資源管理員中建立網際網路面向的負載平衡器"
   services="load-balancer"
   documentationCenter="na"
   authors="sdwheeler"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"
/>
<tags
   ms.service="load-balancer"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="08/31/2016"
   ms.author="sewhee" />

# 開始使用 Azure CLI 建立內部負載平衡器

[AZURE.INCLUDE [load-balancer-get-started-internet-arm-selectors-include.md](../../includes/load-balancer-get-started-internet-arm-selectors-include.md)]

[AZURE.INCLUDE [load-balancer-get-started-internet-intro-include.md](../../includes/load-balancer-get-started-internet-intro-include.md)]

[AZURE.INCLUDE [azure-arm-classic-important-include](../../includes/azure-arm-classic-important-include.md)] 本文涵蓋之內容包括資源管理員部署模型。您也可以[了解如何使用傳統部署建立網際網路面向的負載平衡器](load-balancer-get-started-internet-classic-portal.md)


[AZURE.INCLUDE [load-balancer-get-started-internet-scenario-include.md](../../includes/load-balancer-get-started-internet-scenario-include.md)]

## 使用 Azure CLI 來部署方案

下列步驟說明如何搭配 CLI 使用 Azure Resource Manager，來建立網際網路對向負載平衡器。使用 Azure Resource Manager 時，會個別建立並設定每項資源，然後放在一起來建立一項資源。

您必須建立並設定下列物件，才能部署負載平衡器：

- 前端 IP 組態 - 包含傳入網路流量的公用 IP 位址。
- 後端位址集區 - 包含虛擬機器的網路介面 (NIC)，可從負載平衡器接收網路流量。
- 負載平衡規則 - 包含將負載平衡器上的公用連接埠對應至後端位址集區中連接埠的規則。
- 輸入 NAT 規則 - 包含將負載平衡器上的公用連接埠對應至後端位址集區中特定虛擬機器之連接埠的規則。
- 探查 - 包含用來檢查後端位址集區中虛擬機器執行個體可用性的健全狀態探查。

如需詳細資訊，請參閱 [Azure Resource Manager 的負載平衡器支援](load-balancer-arm.md)。

## 設定 CLI 以使用 Resource Manager

1. 如果您從未用過 Azure CLI，請參閱[安裝和設定 Azure CLI](../../articles/xplat-cli-install.md)，並依照指示進行，直到選取您的 Azure 帳戶和訂用帳戶。

2. 執行 **azure config mode** 命令，以切換為 Azure 資源管理員模式，如下所示。

        azure config mode arm

    預期的輸出：

        info:    New mode is arm

## 建立前端 IP 集區的虛擬網路和公用 IP 位址

1. 使用名為 *NRPRG* 的資源群組，在美國東部位置建立名為 *NRPVnet* 的虛擬網路 (VNet)。

        azure network vnet create NRPRG NRPVnet eastUS -a 10.0.0.0/16

    建立名為 *NRPVnetSubnet* 的子網路，其中包含 *NRPVnet* 中 10.0.0.0/24 的 CIDR 區塊。

        azure network vnet subnet create NRPRG NRPVnet NRPVnetSubnet -a 10.0.0.0/24

2. 建立名為 *NRPPublicIP* 的公用 IP 位址，以供 DNS 名稱為 *loadbalancernrp.eastus.cloudapp.azure.com* 的前端 IP 集區使用。下列命令使用靜態配置類型和 4 分鐘的閒置逾時。

        azure network public-ip create -g NRPRG -n NRPPublicIP -l eastus -d loadbalancernrp -a static -i 4

    >[AZURE.IMPORTANT] 負載平衡器將使用公用 IP 的網域標籤作為其 FQDN。這是一項來自傳統部署的變更，該部署使用雲端服務作為負載平衡器 FQDN。在此範例中，FQDN 是 *loadbalancernrp.eastus.cloudapp.azure.com*。

## 建立負載平衡器

下列命令會在「美國東部」Azure 位置中的 *NRPRG* 資源群組內，建立名為 *NRPlb* 的負載平衡器。

    azure network lb create NRPRG NRPlb eastus

## 建立前端 IP 集區與後端位址集區

此範例會建立前端 IP 集區來接收負載平衡器上的傳入網路流量，以及建立後端 IP 集區，供前端集區傳送已負載平衡的網路流量。

1. 建立將在上個步驟中建立的公用 IP 與負載平衡器關聯的前端 IP 集區。

        azure network lb frontend-ip create nrpRG NRPlb NRPfrontendpool -i nrppublicip

2. 設定用來從前端 IP 集區接收傳入流量的後端位址集區。

        azure network lb address-pool create NRPRG NRPlb NRPbackendpool

## 建立 LB 規則、NAT 規則及探查

此範例會建立下列項目：

- 一個可將連接埠 21 上的所有傳入流量轉譯至連接埠 22<sup>1</sup> 的 NAT 規則
- 一個可將連接埠 23 上的所有傳入流量轉譯至連接埠 22 的 NAT 規則
- 一個可將連接埠 80 上所有傳入流量負載平衡至後端集區中位址上連接埠 80 的負載平衡器規則。
- 一個可在名為 *HealthProbe.aspx* 的頁面上檢查健全狀態的探查規則。

<sup>1</sup> NAT 規則會關聯到負載平衡器後方的特定虛擬機器執行個體。系統會將抵達連接埠 21 的網路流量會傳送給與此 NAT 規則關聯之連接埠 22 上的特定虛擬機器。您必須為 NAT 規則指定通訊協定 (UDP 或 TCP)。無法將兩種通訊協定指派到相同的連接埠。

1. 建立 NAT 規則。

        azure network lb inbound-nat-rule create -g nrprg -l nrplb -n ssh1 -p tcp -f 21 -b 22
        azure network lb inbound-nat-rule create -g nrprg -l nrplb -n ssh2 -p tcp -f 23 -b 22

    參數：
    * **-g** - 資源群組名稱
    * **-l** - 負載平衡器名稱
    * **-n** - 資源的名稱，不論是 nat 規則、探查還是 LB 規則。
    * **-p** - 通訊協定 (可以是 TCP 或 UDP)
    * **-f** - 要使用的前端連接埠 (探查命令會使用 -f 來定義探查路徑)
    * **-b** - 要使用的後端連接埠

2. 建立負載平衡器規則。

        azure network lb rule create nrprg nrplb lbrule -p tcp -f 80 -b 80 -t NRPfrontendpool -o NRPbackendpool

3. 建立健全狀況探查。

        azure network lb probe create -g nrprg -l nrplb -n healthprobe -p "http" -o 80 -f healthprobe.aspx -i 15 -c 4

    參數：
    * **-g** - 資源群組
    * **-l** - 負載平衡器集的名稱
    * **-n** - 健全狀況探查的名稱
    * **-p** - 健全狀況探查所使用的通訊協定
    * **-i** - 探查間隔 (單位為秒)
    * **-c** - 檢查次數

4. 檢查您的設定。

        azure network lb show nrprg nrplb

    預期的輸出：

        info:    Executing command network lb show
        + Looking up the load balancer "nrplb"
        + Looking up the public ip "NRPPublicIP"
        data:    Id                              : /subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb
        data:    Name                            : nrplb
        data:    Type                            : Microsoft.Network/loadBalancers
        data:    Location                        : eastus
        data:    Provisioning State              : Succeeded
        data:    Frontend IP configurations:
        data:      Name                          : NRPfrontendpool
        data:      Provisioning state            : Succeeded
        data:      Public IP address id          : /subscriptions/####################################/resourceGroups/NRPRG/providers/Microsoft.Network/publicIPAddresses/NRPPublicIP
        data:      Public IP allocation method   : Static
        data:      Public IP address             : 40.114.13.145
        data:
        data:    Backend address pools:
        data:      Name                          : NRPbackendpool
        data:      Provisioning state            : Succeeded
        data:
        data:    Load balancing rules:
        data:      Name                          : HTTP
        data:      Provisioning state            : Succeeded
        data:      Protocol                      : Tcp
        data:      Frontend port                 : 80
        data:      Backend port                  : 80
        data:      Enable floating IP            : false
        data:      Idle timeout in minutes       : 4
        data:      Frontend IP configuration     : /subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/frontendIPConfigurations/NRPfrontendpool
        data:      Backend address pool          : /subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/backendAddressPools/NRPbackendpool
        data:
        data:    Inbound NAT rules:
        data:      Name                          : ssh1
        data:      Provisioning state            : Succeeded
        data:      Protocol                      : Tcp
        data:      Frontend port                 : 21
        data:      Backend port                  : 22
        data:      Enable floating IP            : false
        data:      Idle timeout in minutes       : 4
        data:      Frontend IP configuration     : /subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/frontendIPConfigurations/NRPfrontendpool
        data:
        data:      Name                          : ssh2
        data:      Provisioning state            : Succeeded
        data:      Protocol                      : Tcp
        data:      Frontend port                 : 23
        data:      Backend port                  : 22
        data:      Enable floating IP            : false
        data:      Idle timeout in minutes       : 4
        data:      Frontend IP configuration     : /subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/frontendIPConfigurations/NRPfrontendpool
        data:
        data:    Probes:
        data:      Name                          : healthprobe
        data:      Provisioning state            : Succeeded
        data:      Protocol                      : Http
        data:      Port                          : 80
        data:      Interval in seconds           : 15
        data:      Number of probes              : 4
        data:
        info:    network lb show command OK

## 建立 NIC

您需要建立 NIC (或修改現有的) 並將它們關聯至 NAT 規則、負載平衡器規則和探查。

1. 建立名為 *lb-nic1-be* 的 NIC，並將它與 *rdp1* NAT 規則及 *NRPbackendpool* 後端位址集區建立關聯。

        azure network nic create -g nrprg -n lb-nic1-be --subnet-name nrpvnetsubnet --subnet-vnet-name nrpvnet -d "/subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/backendAddressPools/NRPbackendpool" -e "/subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/inboundNatRules/rdp1" eastus

    參數：

    * **-g** - 資源群組名稱
    * **-n** - NIC 資源的名稱
    * **--subnet-name** - 子網路的名稱
    * **--subnet-vnet-name** - 虛擬網路的名稱
    * **-d** - 後端集區資源的識別碼 - 開頭為 /subscription/{subscriptionID/resourcegroups/<resourcegroup-name>/providers/Microsoft.Network/loadbalancers/<load-balancer-name>/backendaddresspools/<name-of-the-backend-pool>
    * **-e** - 要與 NIC 資源產生關聯的 NAT 規則識別碼 - 開頭為 /subscriptions/####################################/resourceGroups/<resourcegroup-name>/providers/Microsoft.Network/loadBalancers/<load-balancer-name>/inboundNatRules/<nat-rule-name>

    預期的輸出：

        info:    Executing command network nic create
        + Looking up the network interface "lb-nic1-be"
        + Looking up the subnet "nrpvnetsubnet"
        + Creating network interface "lb-nic1-be"
        + Looking up the network interface "lb-nic1-be"
        data:    Id                              : /subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/networkInterfaces/lb-nic1-be
        data:    Name                            : lb-nic1-be
        data:    Type                            : Microsoft.Network/networkInterfaces
        data:    Location                        : eastus
        data:    Provisioning state              : Succeeded
        data:    Enable IP forwarding            : false
        data:    IP configurations:
        data:      Name                          : NIC-config
        data:      Provisioning state            : Succeeded
        data:      Private IP address            : 10.0.0.4
        data:      Private IP Allocation Method  : Dynamic
        data:      Subnet                        : /subscriptions/####################################/resourceGroups/NRPRG/providers/Microsoft.Network/virtualNetworks/NRPVnet/subnets/NRPVnetSubnet
        data:      Load balancer backend address pools
        data:        Id                          : /subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/backendAddressPools/NRPbackendpool
        data:      Load balancer inbound NAT rules:
        data:        Id                          : /subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/inboundNatRules/rdp1
        data:
        info:    network nic create command OK

2. 建立名為 *lb-nic2-be* 的 NIC，並將它與 *rdp2* NAT 規則及 *NRPbackendpool* 後端位址集區建立關聯。

        azure network nic create -g nrprg -n lb-nic2-be --subnet-name nrpvnetsubnet --subnet-vnet-name nrpvnet -d "/subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/backendAddressPools/NRPbackendpool" -e "/subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/inboundNatRules/rdp2" eastus

3. 建立名為 *web1* 的虛擬機器 (VM)，並將它與名為 *lb-nic1-be* 的 NIC 產生關聯。系統先建立了名為 *web1nrp* 的儲存體帳戶，再執行下方命令。

        azure vm create --resource-group nrprg --name web1 --location eastus --vnet-name nrpvnet --vnet-subnet-name nrpvnetsubnet --nic-name lb-nic1-be --availset-name nrp-avset --storage-account-name web1nrp --os-type Windows --image-urn MicrosoftWindowsServer:WindowsServer:2012-R2-Datacenter:4.0.20150825

    >[AZURE.IMPORTANT] 負載平衡器中的 VM 必須在相同的可用性設定組中。使用 `azure availset create` 建立可用性設定組。

    輸出應該類似如下範例：

        info:    Executing command vm create
        + Looking up the VM "web1"
        Enter username: azureuser
        Enter password for azureuser: *********
        Confirm password: *********
        info:    Using the VM Size "Standard_A1"
        info:    The [OS, Data] Disk or image configuration requires storage account
        + Looking up the storage account web1nrp
        + Looking up the availability set "nrp-avset"
        info:    Found an Availability set "nrp-avset"
        + Looking up the NIC "lb-nic1-be"
        info:    Found an existing NIC "lb-nic1-be"
        info:    Found an IP configuration with virtual network subnet id "/subscriptions/####################################/resourceGroups/NRPRG/providers/Microsoft.Network/virtualNetworks/NRPVnet/subnets/NRPVnetSubnet" in the NIC "lb-nic1-be"
        info:    This is a NIC without publicIP configured
        + Creating VM "web1"
        info:    vm create command OK

    >[AZURE.NOTE] 預期會顯示「此 NIC 未設有 publicIP」訊息，因為針對負載平衡器建立的 NIC 會使用負載平衡器公用 IP 位址連線到網際網路。

    由於 *lb-nic1-be* NIC 會與 *rdp1* NAT 規則相關聯，因此您可以使用 RDP 透過負載平衡器上的連接埠 3441 連線至 *web1*。

4. 建立名為 *web2* 的虛擬機器 (VM)，並將它與名為 *lb-nic2-be* 的 NIC 產生關聯。在執行下列命令之前，會先建立名為 *web1nrp* 的儲存體帳戶。

        azure vm create --resource-group nrprg --name web2 --location eastus --vnet-name nrpvnet --vnet-subnet-name nrpvnetsubnet --nic-name lb-nic2-be --availset-name nrp-avset --storage-account-name web2nrp --os-type Windows --image-urn MicrosoftWindowsServer:WindowsServer:2012-R2-Datacenter:4.0.20150825

## 更新現有負載平衡器

您可以新增參考現有負載平衡器的規則。在下一個範例中，會將新負載平衡器規則新增到現有的負載平衡器 **NRPlb**

    azure network lb rule create -g nrprg -l nrplb -n lbrule2 -p tcp -f 8080 -b 8051 -t frontendnrppool -o NRPbackendpool

參數：

* **-g** - 資源群組名稱
* **-l** - 負載平衡器名稱
* **-n** - 負載平衡器規則名稱
* **-p** - 通訊協定
* **-f** - 前端連接埠
* **-b** - 後端連接埠
* **-t** - 前端集區名稱
* **-b** - 後端集區名稱

## 刪除負載平衡器

請使用下列命令來移除負載平衡器：

    azure network lb delete -g nrprg -n nrplb

其中 **nrprg** 是資源群組，而 **nrplb** 是負載平衡器名稱。

## 後續步驟

[開始設定內部負載平衡器](load-balancer-get-started-ilb-arm-cli.md)

[設定負載平衡器分配模式](load-balancer-distribution-mode.md)

[設定負載平衡器的閒置 TCP 逾時設定](load-balancer-tcp-idle-timeout.md)

<!----HONumber=AcomDC_0914_2016-->
