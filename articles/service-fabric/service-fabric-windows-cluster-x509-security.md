<properties
   pageTitle="連接到安全的私人叢集 | Microsoft Azure"
   description="本文說明如何保護獨立或私人叢集內以及用戶端與叢集之間的通訊。"
   services="service-fabric"
   documentationCenter=".net"
   authors="dsk-2015"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="07/08/2016"
   ms.author="dkshir"/>

# 使用 X.509 憑證保護 Windows 上的獨立叢集

本文說明如何保護獨立 Windows 叢集的各個節點之間的通訊，以及如何使用 X.509 憑證來驗證連線到此叢集的用戶端。這可確保只有已獲授權的使用者可以存取叢集和已部署的應用程式，以及執行管理工作。憑證安全性應在叢集建立之時先在叢集上啟用。

如需諸如節點對節點安全性、用戶端對節點安全性、角色型存取控制等有關叢集安全性的詳細資訊，請參閱[叢集安全性案例](service-fabric-cluster-security.md)。

## 您需要哪些憑證？

一開始，請[將獨立叢集封裝](service-fabric-cluster-creation-for-windows-server.md#downloadpackage)下載至叢集中的其中一個節點。在下載的套件中，您會找到 **ClusterConfig.X509.MultiMachine.json** 檔案。開啟檔案，檢閱 **properties** 區段下的 **security** 區段：

    "security": {
        "metadata": "The Credential type X509 indicates this is cluster is secured using X509 Certificates. The thumbprint format is - d5 ec 42 3b 79 cb e5 07 fd 83 59 3c 56 b9 d5 31 24 25 42 64.",
        "ClusterCredentialType": "X509",
        "ServerCredentialType": "X509",
        "CertificateInformation": {
			"ClusterCertificate": {
                "Thumbprint": "[Thumbprint]",
                "ThumbprintSecondary": "[Thumbprint]",
                "X509StoreName": "My"
            },
            "ServerCertificate": {
                "Thumbprint": "[Thumbprint]",
                "ThumbprintSecondary": "[Thumbprint]",
                "X509StoreName": "My"
            },
            "ClientCertificateThumbprints": [{
                "CertificateThumbprint": "[Thumbprint]",
                "IsAdmin": false
            }, {
                "CertificateThumbprint": "[Thumbprint]",
                "IsAdmin": true
            }],
            "ClientCertificateCommonNames": [{
                "CertificateCommonName": "[CertificateCommonName]",
                "CertificateIssuerThumbprint" : "[Thumbprint]",
                "IsAdmin": true
            }]
        }
    },

此區段描述保護獨立 Windows 叢集所需的憑證。若要啟用憑證型安全性，將 **ClusterCredentialType** 和 **ServerCredentialType** 的值設定為 X509。

>[AZURE.NOTE] [指紋](https://en.wikipedia.org/wiki/Public_key_fingerprint)是憑證的主要身分識別。閱讀[如何擷取憑證的指紋](https://msdn.microsoft.com/library/ms734695.aspx)，以找出您所建立的憑證指紋。

下表列出您在設定叢集時所需的憑證：

|**CertificateInformation 設定**|**說明**|
|-----------------------|--------------------------|
|ClusterCertificate|需有此憑證，才能保護叢集上節點之間的通訊。您可以使用兩個不同的憑證 (主要和次要) 進行更新。在 **Thumbprint** 區段中設定主要憑證的指紋，以及在 **ThumbprintSecondary** 變數中設定次要憑證的指紋。|
|ServerCertificate|用戶端嘗試連線到此叢集時，會向用戶端此憑證顯示此憑證。為了方便起見，您可以選擇對 *ClusterCertificate* 和 *ServerCertificate* 使用相同的憑證。您可以使用兩個不同的伺服器憑證 (主要和次要) 進行更新。在 **Thumbprint** 區段中設定主要憑證的指紋，以及在 **ThumbprintSecondary** 變數中設定次要憑證的指紋。 |
|ClientCertificateThumbprints|這是您想在經過驗證的用戶端上安裝的一組憑證。在您要允許存取叢集的電腦上，您可以安裝數個不同的用戶端憑證。在 **CertificateThumbprint** 變數中設定每個憑證的指紋。如果您將 **IsAdmin** 設為 true，則已安裝此憑證的用戶端可以對叢集執行系統管理員管理活動。如果 **IsAdmin** 是 false，有此憑證的用戶端只能執行其使用者存取權限允許的動作，通常是唯讀。如需角色的詳細資訊，請參閱[角色型存取控制 (RBAC)](service-fabric-cluster-security.md/#role-based-access-control-rbac) |
|ClientCertificateCommonNames|針對 **CertificateCommonName** 設定第一個用戶端憑證的一般名稱。**CertificateIssuerThumbprint** 是此憑證的簽發者指紋。閱讀[使用憑證](https://msdn.microsoft.com/library/ms731899.aspx)，以深入了解一般名稱和簽發者。|

以下是範例叢集組態，其中已提供叢集、 伺服器和用戶端憑證。

 ```
 {
    "name": "SampleCluster",
    "clusterConfigurationVersion": "1.0.0",
    "apiVersion": "2016-09-26",
    "nodes": [{
        "nodeName": "vm0",
		"metadata": "Replace the localhost below with valid IP address or FQDN",
        "iPAddress": "10.7.0.5",
        "nodeTypeRef": "NodeType0",
        "faultDomain": "fd:/dc1/r0",
        "upgradeDomain": "UD0"
    }, {
      "nodeName": "vm1",
      		"metadata": "Replace the localhost with valid IP address or FQDN",
        "iPAddress": "10.7.0.4",
        "nodeTypeRef": "NodeType0",
        "faultDomain": "fd:/dc1/r1",
        "upgradeDomain": "UD1"
    }, {
        "nodeName": "vm2",
      "iPAddress": "10.7.0.6",
      		"metadata": "Replace the localhost with valid IP address or FQDN",
        "nodeTypeRef": "NodeType0",
        "faultDomain": "fd:/dc1/r2",
        "upgradeDomain": "UD2"
    }],
    "properties": {
		"diagnosticsStore": {
        "metadata":  "Please replace the diagnostics store with an actual file share accessible from all cluster machines.",
        "dataDeletionAgeInDays": "7",
        "storeType": "FileShare",
        "IsEncrypted": "false",
        "connectionstring": "c:\\ProgramData\\SF\\DiagnosticsStore"
		}
        "security": {
            "metadata": "The Credential type X509 indicates this is cluster is secured using X509 Certificates. The thumbprint format is - d5 ec 42 3b 79 cb e5 07 fd 83 59 3c 56 b9 d5 31 24 25 42 64.",
            "ClusterCredentialType": "X509",
            "ServerCredentialType": "X509",
            "CertificateInformation": {
                "ClusterCertificate": {
                    "Thumbprint": "a8 13 67 58 f4 ab 89 62 af 2b f3 f2 79 21 be 1d f6 7f 43 26",
                    "X509StoreName": "My"
                },
                "ServerCertificate": {
                    "Thumbprint": "a8 13 67 58 f4 ab 89 62 af 2b f3 f2 79 21 be 1d f6 7f 43 26",
                    "X509StoreName": "My"
                },
                "ClientCertificateThumbprints": [{
                    "CertificateThumbprint": "c4 c18 8e aa a8 58 77 98 65 f8 61 4a 0d da 4c 13 c5 a1 37 6e",
                    "IsAdmin": false
                }, {
                    "CertificateThumbprint": "71 de 04 46 7c 9e d0 54 4d 02 10 98 bc d4 4c 71 e1 83 41 4e",
                    "IsAdmin": true
                }]
            }
        },
        "reliabilityLevel": "Bronze",
        "nodeTypes": [{
            "name": "NodeType0",
            "clientConnectionEndpointPort": "19000",
            "clusterConnectionEndpoint": "19001",
            "httpGatewayEndpointPort": "19080",
            "applicationPorts": {
                "startPort": "20001",
                "endPort": "20031"
            },
            "ephemeralPorts": {
                "startPort": "20032",
                "endPort": "20062"
            },
            "isPrimary": true
        }
         ],
        "fabricSettings": [{
            "name": "Setup",
            "parameters": [{
                "name": "FabricDataRoot",
                "value": "C:\\ProgramData\\SF"
            }, {
                "name": "FabricLogRoot",
                "value": "C:\\ProgramData\\SF\\Log"
            }]
        }]
    }
}
 ```

## 取得 X.509 憑證
若要保護叢集內的通訊，您必須先取得叢集節點的 X.509 憑證。此外，若要將此叢集的連線限制於經過授權的電腦/使用者，您必須取得並安裝這些用戶端電腦的憑證。

對於執行生產環境工作負載的叢集，您應該使用[憑證授權單位 (CA)](https://en.wikipedia.org/wiki/Certificate_authority) 簽署的 X.509 憑證來保護叢集。如需有關如何取得這些憑證的詳細資訊，請移至[做法：取得憑證](http://msdn.microsoft.com/library/aa702761.aspx)。

若是用於測試的叢集，您可以選擇使用自我簽署憑證。

## 選擇性：建立自我簽署憑證
要建立您可以正確地保護的自我簽署憑證，其中一個做法是使用 C:\\Program Files\\Microsoft SDKs\\Service Fabric\\ClusterSetup\\Secure 目錄下 Service Fabric SDK 資料夾中的CertSetup.ps1 指令碼。編輯檔案，並使用此檔案建立具適當名稱的憑證。

現在將憑證匯出至 PFX 檔並設定保護密碼。首先，您必須取得憑證的指紋。執行 certmgr.exe 應用程式。瀏覽至 [本機電腦\\個人] 資料夾，找到您剛建立的憑證。按兩下憑藉以開啟它，選取 [詳細資料] 索引標籤，然後向下捲動至 [指紋] 欄位。將憑證指紋值複製到以下的 PowerShell 命令，移除空格。變更 $pswd 的值為適用性安全密碼以保護它，然後執行此 PowerShell︰

```   
$pswd = ConvertTo-SecureString -String "1234" -Force –AsPlainText
Get-ChildItem -Path cert:\localMachine\my<Thumbprint> | Export-PfxCertificate -FilePath C:\mypfx.pfx -Password $pswd
```

若要檢視電腦上安裝之憑證的詳細資訊，可以執行下列 PowerShell 命令︰

```
$cert = Get-Item Cert:\LocalMachine\My<Thumbprint>
Write-Host $cert.ToString($true)
```

或者，如果您有 Azure 訂用帳戶，可以依照[取得 X.509 憑證](service-fabric-secure-azure-cluster-with-certs.md#acquirecerts)一節來建立您的憑證。

## 安裝憑證
有了憑證之後，可以將它們安裝在叢集節點上。節點上須已安裝最新的 Windows PowerShell 3.x。您必須針對叢集和伺服器憑證及任何次要憑證，在每個節點上重複這些步驟。

1. 將 .pfx 檔案複製到節點。
2. 以系統管理員身分開啟 PowerShell 視窗並輸入下列命令。以您用來建立此憑證的密碼取代 $pswd。以複製到這個節點之 .pfx 的完整路徑取代 $PfxFilePath。

    ```
    $pswd = "1234"
    $PfcFilePath ="C:\mypfx.pfx"
    Import-PfxCertificate -Exportable -CertStoreLocation Cert:\LocalMachine\My -FilePath $PfxFilePath -Password (ConvertTo-SecureString -String $pswd -AsPlainText -Force)
    ```

3. 接下來，您必須在此憑證上設定存取控制，讓在「網路服務」帳戶下執行的 Service Fabric 程序可以藉由執行下列指令碼來使用它。提供憑證的指紋和服務帳戶的「網路服務」。您可以使用 certmgr.exe 工具並以 [管理私密金鑰] 查看憑證，以檢查憑證上的 ACL 是否正確。

    ```
    param
    (
        [Parameter(Position=1, Mandatory=$true)]
        [ValidateNotNullOrEmpty()]
        [string]$pfxThumbPrint,

        [Parameter(Position=2, Mandatory=$true)]
        [ValidateNotNullOrEmpty()]
        [string]$serviceAccount
        )

        $cert = Get-ChildItem -Path cert:\LocalMachine\My | Where-Object -FilterScript { $PSItem.ThumbPrint -eq $pfxThumbPrint; };

        # Specify the user, the permissions and the permission type
        $permission = "$($serviceAccount)","FullControl","Allow"
        $accessRule = New-Object -TypeName System.Security.AccessControl.FileSystemAccessRule -ArgumentList $permission;

        # Location of the machine related keys
        $keyPath = $env:ProgramData + "\Microsoft\Crypto\RSA\MachineKeys";
        $keyName = $cert.PrivateKey.CspKeyContainerInfo.UniqueKeyContainerName;
        $keyFullPath = $keyPath + $keyName;

        # Get the current acl of the private key
        $acl = (Get-Item $keyFullPath).GetAccessControl('Access')

        # Add the new ace to the acl of the private key
        $acl.SetAccessRule($accessRule);

        # Write back the new acl
        Set-Acl -Path $keyFullPath -AclObject $acl -ErrorAction Stop

        #Observe the access rights currently assigned to this certificate.
        get-acl $keyFullPath| fl
        ```

4. 為每個伺服器憑證重複上述步驟。您也可以使用上述步在您要允許存取叢集的電腦上安裝用戶端憑證。

## 建立安全的叢集
設定 **ClusterConfig.X509.MultiMachine.json** 檔案的 **security** 區段後，您可以繼續進行[建立叢集](service-fabric-cluster-creation-for-windows-server.md#createcluster)一節，以設定節點和建立獨立叢集。請記得在建立叢集時使用 **ClusterConfig.X509.MultiMachine.json** 檔案。例如，您的命令可能如下所示：

```
.\CreateServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.X509.MultiMachine.json -MicrosoftServiceFabricCabFilePath .\MicrosoftAzureServiceFabric.cab -AcceptEULA $true
```

順利執行安全的獨立 Windows 叢集，並已設定經過驗證的用戶端以進行連接之後，請依照[使用 PowerShell 來連線到安全的叢集](service-fabric-connect-to-secure-cluster.md#connectsecurecluster)一節來連接它。例如：

```
Connect-ServiceFabricCluster -ConnectionEndpoint 10.7.0.4:19000 -KeepAliveIntervalInSec 10 -X509Credential -ServerCertThumbprint 057b9544a6f2733e0c8d3a60013a58948213f551 -FindType FindByThumbprint -FindValue 057b9544a6f2733e0c8d3a60013a58948213f551 -StoreLocation CurrentUser -StoreName My
```

如果您是登入叢集中的其中一部機器，因為它已本機安裝憑證，您只要執行 Powershell 命令就可以連接到該叢集，並顯示節點的清單︰

```
Connect-ServiceFabricCluster
Get-ServiceFabricNode
```
若要移除叢集，請呼叫下列命令：

```
.\RemoveServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.X509.MultiMachine.json   -MicrosoftServiceFabricCabFilePath .\MicrosoftAzureServiceFabric.cab
```

<!---HONumber=AcomDC_0921_2016-->