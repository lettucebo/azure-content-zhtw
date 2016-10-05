本文概述一組已經實證的做法，以便在 Azure 上執行 Windows 虛擬機器 (VM)，並注意延展性、可用性、管理性和安全性。

> [AZURE.NOTE] Azure 有兩個不同的部署模型：[Azure Resource Manager][resource-manager-overview] 和傳統。本文使用 Microsoft 建議用於新部署的資源管理員。

我們不建議將單一 VM 用於生產工作負載，因為 Azure 上的單一 VM 沒有正常運作服務等級協定 (SLA)。若要取得 SLA，您必須在[可用性設定組][availability-set]中部署多個 VM。如需詳細資訊，請參閱[在 Azure 上執行多個 Windows VM][multi-vm]。

## 架構圖表

在 Azure 中佈建 VM 牽涉的移動組建比 VM 本身更多。有計算、網路及儲存體元素。

![[0]][0]

- **資源群組。** [_資源群組_][resource-manager-overview]是保存相關資源的容器。建立資源群組以保存此 VM 的資源。

- **VM**。您可以從已發佈的映像清單，或從您上傳至 Azure Blob 儲存體的虛擬硬碟 (VHD) 檔案佈建 VM。

- **作業系統磁碟。** 作業系統磁碟是儲存在 [Azure 儲存體][azure-storage]中的 VHD。這表示即使主機電腦關閉，它仍持續存在。

- **暫存磁碟。** VM 是使用暫存磁碟 (Windows 上的 `D:` 磁碟機) 來建立。此磁碟會儲存在主機電腦的實體磁碟機上。_不會_儲存在 Azure 儲存體中，而且可能在重新開機期間和其他 VM 生命週期事件中消失。僅將此磁碟使用於暫存資料，例如分頁檔或交換檔。

- **資料磁碟。** [資料磁碟][data-disk]是用於應用程式資料的持續性 VHD。資料磁碟會儲存在 Azure 儲存體中，例如作業系統磁碟。

- **虛擬網路 (VNet) 和子網路。** 在 Azure 中的每個 VM 都會部署到 VNet 中，而虛擬網路會進一步分成數個子網路。

- **公用 IP 位址。** 必須要有公用 IP 位址才能與 VM 進行通訊，例如透過遠端桌面 (RDP)。

- **網路介面 (NIC)**。NIC 可讓 VM 與虛擬網路通訊。

- **網路安全性群組 (NSG)**。[NSG][nsg] 是用來允許/拒絕對子網路的網路流量。您可以將 NSG 與獨立的 NIC 或子網路關聯。如果您將它與子網路關聯，該 NSG 規則會套用至子網路中的所有 VM。
 
- **診斷。** 診斷記錄對於管理和針對 VM 進行疑難排解十分重要。

## 建議

### VM 建議

- 我們建議使用 DS 和 GS 系列，除非您有高效能運算等特殊工作負載。如需詳細資訊，請參閱[虛擬機器的大小][virtual-machine-sizes]。將現有的工作負載移至 Azure 時，從最符合您內部部署伺服器的 VM 大小開始。然後根據 CPU、記憶體和每秒的磁碟輸入/輸出作業 (IOPS) 測量您的實際工作負載效能，並視需要調整大小。此外，如果您需要多個 NIC，請留意每個大小的 NIC 限制。

- 當您佈建 VM 和其他資源時，您必須指定一個位置。一般而言，選擇最接近您的內部使用者或客戶的位置。不過，並非所有 VM 大小在所有位置都可用。如需詳細資訊，請參閱[依區域提供的服務][services-by-region]。若要列出指定位置可用的 VM 大小，請執行下列 Azure 命令列介面 (CLI) 命令：

    ```
    azure vm sizes --location <location>
    ```

- 如需選擇已發佈 VM 映像的相關資訊，請參閱[瀏覽並選取 Azure 虛擬機器映像][select-vm-image]。

### 磁碟和儲存體建議

- 為了達到最佳的磁碟 I/O 效能，我們建議使用[進階儲存體][premium-storage]，這會將資料儲存在固態硬碟 (SSD)。成本是依佈建的磁碟大小而定。IOPS 和輸送量也取決於磁碟大小，因此當您佈建磁碟時，請考慮以下三個因素 (容量、IOPS 和輸送量)。

- 一個儲存體帳戶可以支援 1 到 20 個 VM。

- 新增一或多個資料磁碟。當您建立新的 VHD 時，它仍未格式化。登入 VM 來格式化磁碟。

- 如果您有大量的資料磁碟，請注意儲存體帳戶的總 I/O 限制。如需詳細資訊，請參閱[虛擬機器磁碟限制][vm-disk-limits]。

- 為了達到最佳效能，請建立個別的儲存體帳戶來保存診斷記錄。標準本地備援儲存體 (LRS) 帳戶已足以保存診斷記錄。

- 若情況允許，請將應用程式安裝在資料磁碟，不要安裝在作業系統磁碟。不過，某些舊版的應用程式可能需要將元件安裝在 C: 磁碟機。在此情況下，您可以使用 PowerShell 來[重新調整作業系統磁碟大小][resize-os-disk]。

### 網路建議

- 此公用 IP 位址可以是動態或靜態。預設值為動態。

    - 如果您需要一個不會變更的固定 IP 位址 (例如，如果您需要在 DNS 中建立一個 A 記錄，或需要將 IP 位址列入允許清單)，請保留一個[靜態 IP 位址][static-ip]。

    - 您也可以建立 IP 位址的完整網域名稱 (FQDN)。然後您可以在 DNS 中註冊指向該 FQDN 的 [CNAME 記錄][cname-record]。如需詳細資訊，請參閱[在 Azure 入口網站中建立完整網域名稱][fqdn]。

- 所有 NSG 都包含一組[預設規則][nsg-default-rules]，包括一個封鎖所有網際網路輸入流量的規則。預設的規則不能刪除，但其他規則可以覆寫它們。若要啟用網際網路流量，請建立允許輸入流量輸入特定連接埠的規則 &mdash; 例如，允許連接埠 80 用於 HTTP。

- 若要啟用 RDP，請新增一個 NSG 規則，以允許將輸入流量輸入至 TCP 連接埠 3389。

## 延展性考量

- 您可以藉由[變更 VM 大小][vm-resize]來相應放大或相應縮小 VM。

- 若要水平相應放大，請將兩個以上的 VM 置於附載平衡器後方的可用性設定組中。如需詳細資料，請參閱[在 Azure 上執行多個 Windows VM][multi-vm]。

## 可用性考量

- 如上所述，單一 VM 沒有 SLA。若要取得 SLA，您必須在可用性設定組中部署多個 VM。

- 您的 VM 可能會受到[計劃性維護][planned-maintenance]或[非計劃性維護][manage-vm-availability]影響。您可以使用 [VM 重新啟動記錄檔][reboot-logs]來判斷 VM 重新啟動是否是因為計劃性維護所造成。

- VHD 是由 [Azure 儲存體][azure-storage]支援，而系統會複寫此儲存體來提供持久性和可用性。

- 為了防止在正常作業期間意外遺失資料 (例如，因使用者錯誤而造成)，您也應該使用 [Blob 快照][blob-snapshot]或其他工具來實作時間點備份。

## 管理性考量

- **資源群組。** 將關係密切且共用相同生命週期的資源置於同一個[資源群組][resource-manager-overview]。資源群組可讓您以群組為單位來部署和監視資源，並根據資源群組列出帳單成本。您也可以刪除整組資源，這對於測試部署非常有用。為資源提供有意義的名稱。這樣能更容易找到特定資源及了解其角色。請參閱 [Azure 資源的建議命名慣例][naming conventions]。

- **VM 診斷。** 啟用監視和診斷，包括基本健全狀況度量、診斷基礎結構記錄檔及[開機診斷][boot-diagnostics]。如果您的 VM 進入無法開機的狀態，開機診斷能協助您診斷開機失敗。如需詳細資訊，請參閱[啟用監視和診斷][enable-monitoring]。使用 [Azure 記錄檔收集][log-collector]擴充來收集 Azure 平台記錄檔並上傳至 Azure 儲存體。

    下列 CLI 命令會啟用診斷：

    ```text
    azure vm enable-diag <resource-group> <vm-name>
     ```

- **停止 VM。** Azure 會區分「已停止」和「已取消配置」狀態。當 VM 狀態為「已停止」時，您需要付費。VM 解除配置後，則不需付費。

    使用下列 CLI 命令來解除配置 VM：

    ```text
    azure vm deallocate <resource-group> <vm-name>
    ```

    Azure 入口網站中的 [停止] 按鈕也會將 VM 取消配置。不過，如果您是在登入時透過作業系統進行關閉，則會將 VM 停止但「不會」取消配置，因此您仍需付費。

- **刪除 VM。** 如果您刪除 VM，並不會刪除 VHD。這表示您可以放心地刪除 VM，而不會遺失任何資料。不過，您仍需支付儲存體費用。若要刪除 VHD，請將檔案從 [Blob 儲存體][blob-storage]中刪除。

  若要防止意外刪除，請使用[資源鎖定][resource-lock]來鎖定整個資源群組或鎖定個別資源 (例如 VM)。

## 安全性考量

- 使用 [Azure 資訊安全中心][security-center] 來集中檢視 Azure 資源的安全性狀態。資訊安全中心會監視潛在的安全性問題，例如系統更新、反惡意程式碼，並提供全面性的部署安全性健康狀態。

    - 資訊安全中心是依每個 Azure 訂用帳戶設定。按照[使用資訊安全中心]所述來啟用安全資料收集。

    - 啟用資料收集時，資訊安全性中心就會自動掃描任何該訂用帳戶建立的 VM。

- **修補程式管理。** 如果啟用，資訊安全性中心會檢查是否遺失安全性或重要更新。使用 VM 上的[群組原則設定][group-policy]來啟用自動系統更新。

- **反惡意程式碼。** 如果啟用，資訊安全性中心會檢查是已安裝反惡意程式碼軟體。您也可以使用資訊安全中心來從 Azure 入口網站內安裝反惡意程式碼軟體。

- 使用[角色型存取控制][rbac] \(RBAC) 來控制對您所部署 Azure 資源的存取。RBAC 可讓您指派授權角色給您 DevOps 小組的成員。例如，「讀取者」角色能檢視 Azure 資源但不能建立、管理或刪除它們。某些角色專門用於特定的 Azure 資源類型。例如，「虛擬機器參與者」角色能重新啟動或解除配置 VM、重設系統管理員密碼、建立新的 VM 等等。其他對此參考架構可能有用的[內建 RBAC 角色][rbac-roles]包括 [DevTest Lab 使用者][rbac-devtest]和[網路參與者][rbac-network]。使用者可以被指派多個角色，且您可以針對更詳細的權限建立角色。

    > [AZURE.NOTE] RBAC 不會限制使用者登入 VM 可執行的動作。這些權限是由客體 OS上的帳戶類型來決定。

- 若要重設本機系統管理員密碼，請執行 `vm reset-access` Azure CLI 命令。

    ```text
    azure vm reset-access -u <user> -p <new-password> <resource-group> <vm-name>
    ```

- 使用[稽核記錄檔][audit-logs]來查看佈建動作和其他 VM 事件。

- 如果您要加密作業系統和資料磁碟，請考慮使用 [Azure 磁碟加密][disk-encryption]。

## 解決方案元件

範例解決方案指令碼 [Deploy-ReferenceArchitecture.ps1][solution-script] 可供您用來實作遵循本文中所述建議的結構。此指令碼利用的是 [Resource Manager][ARM-Templates] 範本。範本可做為一組基本建置組塊使用，每個組塊都可以執行特定動作，例如建立 VNet 或設定 NSG。指令碼的用途是協調範本部署。

範例已經參數化，使用個別 JSON 檔案保留參數。您可以在這些檔案中修改參數來設定部署以符合您自己的需求。您不需要修改範本本身。請注意，您不可以在範本檔案中變更物件的結構描述。

當您編輯範本時，請依照 [Azure 資源的建議命名慣例][naming conventions]中所述的命名慣例來建立物件。

指令碼會參考以下參數檔案來建置 VM 和周圍的基礎結構：

- **[virtualNetwork.parameters.json][vnet-parameters]**。此檔案定義 VNet 設定，例如名稱、位址空間、子網路及任何必要 DNS 伺服器的位址。請注意，子網路位址必須納入 VNet 的位址空間。

	<!-- source: https://github.com/mspnp/reference-architectures/blob/master/guidance-compute-single-vm/parameters/windows/virtualNetwork.parameters.json#L4-L21 -->
	```json
  "parameters": {
    "virtualNetworkSettings": {
      "value": {
        "name": "ra-single-vm-vnet",
        "resourceGroup": "ra-single-vm-rg",
        "addressPrefixes": [
          "172.17.0.0/16"
        ],
        "subnets": [
          {
            "name": "ra-single-vm-sn",
            "addressPrefix": "172.17.0.0/24"
          }
        ],
        "dnsServers": [ ]
      }
    }
  }
	```

- **[networkSecurityGroups.parameters.json][nsg-parameters]**.此檔案包含 NSG 和 NSG 規則的定義。`virtualNetworkSettings` 區塊中的 `name` 參數可指定 NSG 要連接的 VNet。`networkSecurityGroupSettings` 區塊中的 `subnets` 參數可識別 VNet 中套用 NSG 規則的任何子網路。這些應該是 **virtualNetwork.parameters.json** 檔案中定義的項目。

	請注意，範例中顯示的預設安全性規則可讓使用者透過遠端桌面 (RDP) 連線來連線到 VM。您可以在 `securityRules` 陣列中新增其他項目來開啟額外的連接埠 (或拒絕透過特定連接埠的存取)。

	<!-- source: https://github.com/mspnp/reference-architectures/blob/master/guidance-compute-single-vm/parameters/windows/networkSecurityGroups.parameters.json#L4-L36 -->
	```json
  "parameters": {
    "virtualNetworkSettings": {
      "value": {
        "name": "ra-single-vm-vnet",
        "resourceGroup": "ra-single-vm-rg"
      }
    },
    "networkSecurityGroupsSettings": {
      "value": [
        {
          "name": "ra-single-vm-nsg",
          "subnets": [
            "ra-single-vm-sn"
          ],
          "networkInterfaces": [
          ],
          "securityRules": [
            {
              "name": "RDPAllow",
              "direction": "Inbound",
              "priority": 100,
              "sourceAddressPrefix": "*",
              "destinationAddressPrefix": "*",
              "sourcePortRange": "*",
              "destinationPortRange": "3389",
              "access": "Allow",
              "protocol": "Tcp"
            }
          ]
        }
      ]
    }
  }
	```

- **[virtualMachineParameters.json][vm-parameters]**。此檔案定義 VM 本身的設定，包括 VM 的名稱與大小、系統管理員使用者的安全性認證、要建立的磁碟，以及要保有這些磁碟的儲存體帳戶。

	您必須在 `imageReference` 區段中指定一個映像。下面顯示的值會建立包含 Windows Server 2012 R2 資料中心最新組建的 VM。您可以使用以下 Azure CLI 命令來取得區域 (此範例使用 westus 區域) 中所有可用 Windows 映像的清單：

	```text
	azure vm image list westus MicrosoftWindowsServer WindowsServer
	```

	`nics` 區段中的 `subnetName` 參數可指定 VM 的子網路。同樣地，`virtualNetworkSettings` 中的 `name` 參數可識別要使用的 VNet。這些值應該是 **virtualNetwork.parameters.json** 檔案中所定義的子網路和 VNet 的名稱。

	您可以透過修改 `buildingBlockSettings` 區段中的設定，建立共用儲存體帳戶或各自擁有儲存體帳戶的多個 VM。如果您建立多個 VM，您也必須在 `availabilitySet` 區段中指定要使用或建立的可用性設定組名稱。

	<!-- source: https://github.com/mspnp/reference-architectures/blob/master/guidance-compute-single-vm/parameters/windows/virtualMachine.parameters.json#L4-L64 -->
	```json
   "parameters": {
    "virtualMachinesSettings": {
      "value": {
        "namePrefix": "ra-single-vm",
        "computerNamePrefix": "cn",
        "size": "Standard_DS1_v2",
        "osType": "windows",
        "adminUsername": "testuser",
        "adminPassword": "AweS0me@PW",
        "sshPublicKey": "",
        "osAuthenticationType": "password",
        "nics": [
          {
            "isPublic": "true",
            "subnetName": "ra-single-vm-sn",
            "privateIPAllocationMethod": "dynamic",
            "publicIPAllocationMethod": "dynamic",
            "enableIPForwarding": false,
            "dnsServers": [
            ],
            "isPrimary": "true"
          }
        ],
        "imageReference": {
          "publisher": "MicrosoftWindowsServer",
          "offer": "WindowsServer",
          "sku": "2012-R2-Datacenter",
          "version": "latest"
        },
        "dataDisks": {
          "count": 2,
          "properties": {
            "diskSizeGB": 128,
            "caching": "None",
            "createOption": "Empty"
          }
        },
        "osDisk": {
          "caching": "ReadWrite"
        },
        "extensions": [ ],
        "availabilitySet": {
          "useExistingAvailabilitySet": "No",
          "name": ""
        }
      }
    },
    "virtualNetworkSettings": {
      "value": {
        "name": "ra-single-vm-vnet",
        "resourceGroup": "ra-single-vm-rg"
      }
    },
    "buildingBlockSettings": {
      "value": {
        "storageAccountsCount": 1,
        "vmCount": 1,
        "vmStartIndex": 0
      }
    }
  }
	```

## 解決方案部署

解決方案假設已符合下列必要條件：

- 您現在已經擁有可用來建立資源群組的 Azure 訂用帳戶。

- 您已經下載並安裝最新的 Azure PowerShell 組建。如需相關指示，請參閱[這裡][azure-powershell-download]。

執行部署解決方案的指令碼：

1. 建立內含子資料夾 `Scripts` 和 `Templates` 的資料夾。

2. 在 [範本] 資料夾中，建立另一個名為 [Windows] 的子資料夾。

3. 將 [Deploy-ReferenceArchitecture.ps1][solution-script] 檔案下載到 [指令碼] 資料夾。

4. 將下列檔案下載到 [範本/Windows] 資料夾：

	- [virtualNetwork.parameters.json][vnet-parameters]

	- [networkSecurityGroups.parameters.json][nsg-parameters]

	- [virtualMachineParameters.json][vm-parameters]

5. 編輯 [指令碼] 資料夾中的 Deploy-ReferenceArchitecture.ps1 檔案，並變更以下這一行來指定應建立或使用的資源群組，以用來保存指令碼所建立的 VM 和資源：

	<!-- source: https://github.com/mspnp/reference-architectures/blob/master/guidance-compute-single-vm/Deploy-ReferenceArchitecture.ps1#L37 -->
	```powershell
	$resourceGroupName = "ra-single-vm-rg"
	```
6. 編輯 [範本/Windows] 資料夾中的每個 JSON 檔案，以針對虛擬網路、NSG 及 VM 設定參數，如上面的＜解決方案元件＞小節中所述。

	>[AZURE.NOTE] 請務必將 virtualMachineParameters.json 檔案中 `virtualNetworkSettings` 區段的 `resourceGroup` 參數，設定成和您在 Deploy-ReferenceArchitecture.ps1 指令碼檔案中指定的參數相同。

7. 開啟 PowerShell 視窗，移動到 [指令碼] 資料夾，然後執行以下命令：

	```powershell
	.\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> Windows
	```

	使用您的 Azure 訂用帳戶識別碼來取代 `<subscription id>`。

	為 `<location>` 指定 Azure 區域，例如 `eastus` 或 `westus`。

8. 當指令碼完成時，請使用 Azure 入口網站確認已經成功建立網路、NSG 及 VM。

## 後續步驟

為了能適用[虛擬機器 SLA][vm-sla]，您必須在可用性設定組中部署兩個或更多個執行個體。如需詳細資訊，請參閱[在 Azure 上執行多個 VM][multi-vm]。

<!-- links -->

[audit-logs]: https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/
[availability-set]: ../articles/virtual-machines/virtual-machines-windows-create-availability-set.md
[azure-cli]: ../articles/virtual-machines-command-line-tools.md
[azure-storage]: ../articles/storage/storage-introduction.md
[blob-snapshot]: ../articles/storage/storage-blob-snapshots.md
[blob-storage]: ../articles/storage/storage-introduction.md
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[cname-record]: https://en.wikipedia.org/wiki/CNAME_record
[data-disk]: ../articles/virtual-machines/virtual-machines-windows-about-disks-vhds.md
[disk-encryption]: ../articles/security/azure-security-disk-encryption.md
[enable-monitoring]: ../articles/azure-portal/insights-how-to-use-diagnostics.md
[fqdn]: ../articles/virtual-machines/virtual-machines-windows-portal-create-fqdn.md
[group-policy]: https://technet.microsoft.com/zh-TW/library/dn595129.aspx
[log-collector]: https://azure.microsoft.com/blog/simplifying-virtual-machine-troubleshooting-using-azure-log-collector/
[manage-vm-availability]: ../articles/virtual-machines/virtual-machines-windows-manage-availability.md
[multi-vm]: ../articles/guidance/guidance-compute-multi-vm.md
[naming conventions]: ../articles/guidance/guidance-naming-conventions.md
[nsg]: ../articles/virtual-network/virtual-networks-nsg.md
[nsg-default-rules]: ../articles/virtual-network/virtual-networks-nsg.md#default-rules
[planned-maintenance]: ../articles/virtual-machines/virtual-machines-windows-planned-maintenance.md
[premium-storage]: ../articles/storage/storage-premium-storage.md
[rbac]: ../articles/active-directory/role-based-access-control-what-is.md
[rbac-roles]: ../articles/active-directory/role-based-access-built-in-roles.md
[rbac-devtest]: ../articles/active-directory/role-based-access-built-in-roles.md#devtest-lab-user
[rbac-network]: ../articles/active-directory/role-based-access-built-in-roles.md#network-contributor
[reboot-logs]: https://azure.microsoft.com/blog/viewing-vm-reboot-logs/
[resize-os-disk]: ../articles/virtual-machines/virtual-machines-windows-expand-os-disk.md
[Resize-VHD]: https://technet.microsoft.com/zh-TW/library/hh848535.aspx
[Resize virtual machines]: https://azure.microsoft.com/blog/resize-virtual-machines/
[resource-lock]: ../articles/resource-group-lock-resources.md
[resource-manager-overview]: ../articles/resource-group-overview.md
[security-center]: https://azure.microsoft.com/services/security-center/
[select-vm-image]: ../articles/virtual-machines/virtual-machines-windows-cli-ps-findimage.md
[services-by-region]: https://azure.microsoft.com/regions/#services
[static-ip]: ../articles/virtual-network/virtual-networks-reserved-public-ip.md
[storage-price]: https://azure.microsoft.com/pricing/details/storage/
[使用資訊安全中心]: ../articles/security-center/security-center-get-started.md#use-security-center
[virtual-machine-sizes]: ../articles/virtual-machines/virtual-machines-windows-sizes.md
[vm-disk-limits]: ../articles/azure-subscription-service-limits.md#virtual-machine-disk-limits
[vm-resize]: ../articles/virtual-machines/virtual-machines-linux-change-vm-size.md
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_0/
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
[solution-script]: https://github.com/mspnp/reference-architectures/tree/master/guidance-compute-single-vm/Deploy-ReferenceArchitecture.ps1
[vnet-parameters]: https://github.com/mspnp/reference-architectures/tree/master/guidance-compute-single-vm/parameters/windows/virtualNetwork.parameters.json
[nsg-parameters]: https://github.com/mspnp/reference-architectures/tree/master/guidance-compute-single-vm/parameters/windows/networkSecurityGroups.parameters.json
[vm-parameters]: https://github.com/mspnp/reference-architectures/tree/master/guidance-compute-single-vm/parameters/windows/virtualMachine.parameters.json
[azure-powershell-download]: https://azure.microsoft.com/documentation/articles/powershell-install-configure/
[0]: ./media/guidance-blueprints/compute-single-vm.png "Azure 中的單一 Windows VM 架構"

<!---HONumber=AcomDC_0831_2016-->