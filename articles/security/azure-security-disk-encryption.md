<properties
   pageTitle="Windows 和 Linux IaaS VM 適用的 Azure 磁碟加密 | Microsoft Azure"
   description="本文件提供 Windows 和 Linux IaaS VM 適用的 Microsoft Azure 磁碟加密的概觀。"
   services="security"
   documentationCenter="na"
   authors="YuriDio"
   manager="swadhwa"
   editor="TomSh"/>

<tags
   ms.service="security"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="09/20/2016"
   ms.author="devtiw"/>


#Windows 和 Linux IaaS VM 適用的 Azure 磁碟加密

Microsoft Azure 強烈承諾確保您的資料隱私權、資料主權，並透過一系列進階技術來加密、控制和管理加密金鑰、控制和稽核資料存取，讓您控制您的 Azure 託管資料。這會提供 Azure 客戶靈活度，可選擇最符合其商務需求的解決方案。本文中，我們將為您介紹新的技術解決方案「Windows 和 Linux IaaS VM 適用的 Azure 磁碟加密」，以協助保護及保障您的資料，以便符合組織的安全性和符合性的承諾。本文提供有關如何使用 Azure 磁碟加密功能的詳細指引，包括支援的案例和使用者體驗。

**附註**：本文包含的特定建議可能會導致資料、網路或計算資源使用量增加，導致額外的授權或訂用帳戶成本。

## 概觀

Azure 磁碟加密是可讓您加密您的 Windows 和 Linux IaaS 虛擬機器磁碟的新功能。Azure 磁碟加密利用 Windows 的業界標準 [BitLocker](https://technet.microsoft.com/library/cc732774.aspx) 功能和 Linux 的 [DM-Crypt](https://en.wikipedia.org/wiki/Dm-crypt) 功能，為 OS 和資料磁碟提供磁碟區加密。此解決方案與 [Azure 金鑰保存庫](https://azure.microsoft.com/documentation/services/key-vault/)整合，可幫助您控制和管理您的金鑰保存庫訂用帳戶中的磁碟加密金鑰和密碼，同時確保虛擬機器磁碟中的所有資料會在您的 Azure 儲存體中輕鬆加密。

> [AZURE.NOTE] 適用於 Windows IaaS VM 的 Azure 磁碟加密現已[公開上市](https://blogs.msdn.microsoft.com/azuresecurity/2016/05/20/azure-disk-encryption-for-windows-virtual-machines-reaches-general-availability/)。

### 加密案例

Azure 磁碟加密解決方案支援下列客戶案例：

- 對透過加密的 VHD 和加密金鑰建立的新 IaaS VM 啟用加密
- 對透過 Azure 資源庫映像建立的新 IaaS VM 啟用加密
- 在 Azure 中已執行的現有 IaaS VM 上啟用加密
- 在 Windows IaaS VM 上停用加密

在 Microsoft Azure 中啟用時，解決方案會對 IaaS VM 支援下列各項：

- 與 Azure 金鑰保存庫整合

- 標準 [A、D 和 G 系列 IaaS VM](https://azure.microsoft.com/pricing/details/virtual-machines/)

- 在 Windows 和 Linux IaaS VM 上啟用加密

- 在 Windows IaaS VM 上停用加密

- 在執行 Windows Client OS 的 IaaS VM 上啟用加密

- 在具有掛接路徑的磁碟區上啟用加密

- 公開預覽版本支援所有的 Azure 公用區域，而適用於 Windows IaaS VM 的 GA 支援 Azure 澳洲區域

解決方案不支援此版本中的下列案例、功能和技術：

- 基本 VM 和標準 DS (進階儲存體) 系列 IaaS VM

- 使用傳統的 VM 建立方法建立的 IaaS VM

- 在 Azure 中已執行的 Linux IaaS VM 上啟用 OS 磁碟加密

- 在 Linux IaaS VM 上停用透過 Azure 磁碟加密而啟用的加密

- 與您的內部部署金鑰管理服務整合

- Windows Server 2016 Technical Preview 3 和更新版本

- Azure 檔案 (Azure 檔案共用)、網路檔案系統 (NFS)、動態磁碟區、以軟體為基礎的 RAID 系統


### 加密功能

為 Azure IaaS VM 啟用並部署 Azure 磁碟加密時，會啟用下列功能，視提供的組態而定：

- 加密作業系統磁碟區以輕鬆保護客戶儲存體中的開機磁碟區

	- 已在 Azure 中執行的 Linux IaaS VM 上目前不支援 OS 磁碟區加密。僅針對預先加密的 VHD 案例支援 Linux IaaS VM 的 OS 磁碟區加密
	
- 加密資料磁碟區以輕鬆保護客戶儲存體中的資料磁碟區

- 在 Windows IaaS VM 上停用加密

- 保護客戶 Azure 金鑰保存庫訂用帳戶中的加密金鑰和密碼

- 報告加密 IaaS VM 的加密狀態

- 從 IaaS 虛擬機器移除磁碟加密組態設定

Windows 和 Linux IaaS VM 適用的 Azure 磁碟加密解決方案包含 Windows 適用的磁碟加密擴充、Linux 適用的磁碟加密擴充、磁碟加密 PowerShell Cmdlet、磁碟加密 CLI Cmdlet 和磁碟加密 Azure Resource Manager 範本。在執行 Windows 或 Linux OS 的 IaaS VM 上支援 Azure 磁碟加密解決方案。如需支援的作業系統的詳細資訊，請參閱以下的必要條件一節。

**注意：**使用 Azure 磁碟加密來加密 VM 磁碟完全免費。

### 價值主張

Azure 磁碟加密管理解決方案可帶來雲端中的下列商務需求：

-   IaaS VM 會使用業界標準的加密技術輕鬆加以保護，解決組織的安全性與法務遵循的需求。

-   IaaS VM 會在客戶控制金鑰和原則下開機，而且客戶可以在金鑰保存庫中稽核其使用方式。


### 加密工作流程
為 Windows 和 Linux VM 啟用磁碟加密所需的高階步驟如下：

1. 客戶從上述的 3 個加密案例選擇加密案例

2. 客戶選擇透過 Azure 磁碟加密 ARM 範本、PS Cmdlet 或 CLI 命令啟用磁碟加密，並指定加密組態

    - 針對客戶加密的 VHD 案例，客戶上傳加密的 VHD 至其儲存體帳戶和加密金鑰資訊到其金鑰保存庫，並提供加密組態以在新 IaaS VM 上啟用加密

    - 針對透過 Azure 資源庫和已在 Azure 中執行的現有 VM 建立新的 VM，客戶會提供加密組態以在 IaaS VM 上啟用加密

3. 客戶會授與存取給 Azure 平台，以從其金鑰保存庫讀取加密金鑰資料 (Windows 系統的 BitLocker 加密金鑰和 Linux 的複雜密碼)，藉以在 IaaS VM 上啟用加密

4. 客戶提供 Azure AD 身分識別以將加密金鑰資料寫入其金鑰保存庫，在上述的案例 2 和 3 的 IaaS VM 上啟用加密。

5.  Azure 服務管理會使用加密和金鑰保存庫組態更新 VM 服務模型，並為客戶佈建加密的 VM

![Azure 中的 Microsoft Antimalware](./media/azure-security-disk-encryption/disk-encryption-fig1.JPG)

### 解密工作流程

為 Windows IaaS VM 停用磁碟加密所需的高階步驟如下：

1. 客戶選擇透過 Azure 磁碟加密 ARM 範本或 PS Cmdlet，在 Azure 中的執行中 Windows IaaS VM 上停用加密 (解密)，並指定解密組態。

2. 只有 Windows IaaS VM 支援停用加密步驟，而 Linux IaaS VM 不提供支援。

3. 停用加密步驟會在執行中 Windows IaaS VM 上停用 OS 或資料磁碟區或兩者的加密。

4. Azure 服務管理會更新 VM 服務模型，且 Windows IaaS VM 會標示為已解密。VM 的待用內容不會再加密。

5. 停用加密並不會刪除客戶金鑰保存庫和加密金鑰資料 (適用於 Windows 的 BitLocker 加密金鑰和適用於 Linux 的複雜密碼)。

## 必要條件

以下是針對概觀一節提到支援的案例，在 Azure IaaS VM 上啟用 Azure 磁碟加密的必要條件

- 使用者必須擁有有效的作用中 Azure 訂用帳戶，才能在 Azure 支援的區域中建立資源

- 在下列 Windows 伺服器 SKU 上支援 Azure 磁碟加密 - Windows Server 2008 R2、Windows Server 2012、Windows Server 2012 R2。Windows Server 2008 作業系統上不支援此解決方案。此版本中不支援 Windows Server 2016 Technical Preview 3。

在下列 Windows 用戶端 SKU 上支援 Azure 磁碟加密 - Windows 8 Client 和 Windows 10 Client。

**附註**：針對 Windows Server 2008 R2，必須先安裝 .NET Framework 4.5，才能在 Azure 中啟用加密。您可以透過安裝選用的更新「Windows Server 2008 R2 x64 型系統的 Microsoft .NET Framework 4.5.2 ([KB2901983](https://support.microsoft.com/kb/2901983))」，從 Windows Uupdate 安裝它。

- 在下列 Linux 伺服器 SKU 上支援 Azure 磁碟加密 - Ubuntu、CentOS、SUSE、SUSE Linux Enterprise Server (SLES) 和 Red Hat Enterprise Linux。

- 所有資源 (例如：金鑰保存庫、儲存體帳戶、VM 等等) 必須屬於相同的 Azure 區域和訂用帳戶。

**注意** - Azure 磁碟加密需要金鑰保存庫和 VM 位於相同的 Azure 區域。在不同區域中設定它們會導致啟用 Azure 磁碟加密功能失敗。

- 若要針對 Azure 磁碟加密的使用方式設定與配置 Azure 金鑰保存庫，請參閱本文的＜必要條件＞一節中的**針對 Azure 磁碟加密的使用方式設定與配置 Azure 金鑰保存庫**章節。

- 若要針對 Azure 磁碟加密的使用方式在 Azure Active Directory 安裝與設定 Azure AD 應用程式 ，請參閱本文的＜必要條件＞一節中的**在 Azure Active Directory 中設定 Azure AD 應用程式**章節。

- 若要設定和配置 Azure AD 應用程式的金鑰保存庫存取原則，請參閱本文的＜必要條件＞一節中的**設定 Azure AD 應用程式的金鑰保存庫存取原則**章節。

- 若要準備預先加密的 Windows VHD，請參閱本文附錄的**準備已預先加密的 Windows VHD**一節。

- 若要準備預先加密的 Linux VHD，請參閱本文附錄的**準備已預先加密的 Linux VHD**一節。

- Azure 平台需要存取客戶 Azure 金鑰保存庫中的加密金鑰或密碼，才能讓它們可供 VM 用來開機和解密虛擬機器作業系統磁碟區。若要授與權限給 Azure 平台以存取金鑰保存庫，必須針對此需求在金鑰保存庫上設定 **enabledForDiskEncryption** 屬性。如需詳細資訊，請參閱本文附錄的**針對 Azure 磁碟加密的使用方式設定與配置 Azure 金鑰保存庫**一節。

- 金鑰保存庫密碼和金鑰加密金鑰 (KEK) URL 必須已設定版本。Azure 服務管理會強制執行設定版本的這項限制。請參閱以下範例以取得有效的密碼和 KEK URL：

	- 有效密碼 URL 的範例：

		*https://contosovault.vault.azure.net/secrets/BitLockerEncryptionSecretWithKek/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx*

	- 有效 KRK KEK 的範例：

		*https://contosovault.vault.azure.net/keys/diskencryptionkek/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx*

- Azure 磁碟加密不支援將連接埠號碼指定為金鑰保存庫密碼和 KEK URL 的一部分。請參閱以下範例以取得支援的金鑰保存庫 URL：

 	- 不被接受的金鑰保存庫 URL

		*https://contosovault.vault.azure.net:443/secrets/contososecret/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx*

	- 可接受的金鑰保存庫 URL

		*https://contosovault.vault.azure.net/secrets/contososecret/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx*

- 若要啟用 Azure 磁碟加密功能，IaaS VM 必須符合下列網路端點組態需求：

	- IaaS VM 必須能夠連接到 Azure Active Directory 端點 [Login.windows.net]，以取得用來連接到 Azure 金鑰保存庫的權杖

	- IaaS VM 必須能夠連接到 Azure 金鑰保存庫端點，以將加密金鑰寫入客戶金鑰保存庫

	- IaaS VM 必須能夠連接到託管 Azure 擴充儲存機制的 Azure 儲存體端點，和託管 VHD 檔案的 Azure 儲存體帳戶

**附註：**如果您的安全性原則會限制從 Azure VM 到網際網路的存取，您可以解析上述要與其連線的 URI，並設定特定的規則以允許和這些 IP 的輸出連線。

- 使用最新版的 Azure PowerShell SDK 版本來設定 Azure 磁碟加密。下載最新版的 [Azure PowerShell 1.3.0](https://github.com/Azure/azure-powershell/releases/download/v1.3.0-March2016/azure-powershell.1.3.0.msi) 和更新版本

**注意：**[Azure PowerShell SDK 1.1.0 版](https://github.com/Azure/azure-powershell/releases/tag/v1.1.0-January2016)不支援 Azure 磁碟加密。如果您收到與使用 Azure PowerShell 1.1.0 相關的錯誤，請參閱[與 Azure PowerShell 1.1.0 相關的 Azure 磁碟加密錯誤](http://blogs.msdn.com/b/azuresecurity/archive/2016/02/10/azure-disk-encryption-error-related-to-azure-powershell-1-1-0.aspx)文。

- 若要執行任何 Azure CLI 命令並將它與您的 Azure 訂用帳戶產生關聯，您必須先安裝 Azure CLI 版本：

	- 若要安裝 Azure CLI 並將它與 Azure 訂用帳戶建立關聯，請參閱[如何安裝和設定 Azure CLI](../xplat-cli-install.md)

	- 搭配使用適用於 Mac、Linux 和 Windows 的 Azure CLI 與 Azure 資源管理員，請參閱[這裡](azure-cli-arm-commands.md)

- Azure 磁碟加密解決方案對 Windows IaaS VM 使用 BitLocker 外部金鑰保護裝置。如果您的 VM 加入網域，請勿推送會強制使用 TPM 保護裝置的任何群組原則。請參閱[這篇文章](https://technet.microsoft.com/library/ee706521)以取得「在不含相容 TPM 的情形下允許使用 BitLocker」群組原則的詳細資訊。

- Azure 磁碟加密的必要條件，PowerShell 指令碼，以建立 Azure AD 應用程式、建立新的金鑰保存庫或設定現有的金鑰保存庫並啟用加密，位於[這裡](https://github.com/Azure/azure-powershell/blob/dev/src/ResourceManager/Compute/Commands.Compute/Extension/AzureDiskEncryption/Scripts/AzureDiskEncryptionPreRequisiteSetup.ps1)。

### 針對 Azure 磁碟加密的使用方式設定與配置 Azure 金鑰保存庫

Azure 磁碟加密會保護 Azure 金鑰保存庫中的磁碟加密金鑰和密碼。請依照以下各節的步驟，為 Azure 磁碟加密使用方式設定金鑰保存庫。

#### 建立新金鑰保存庫
若要建立新的金鑰保存庫，請使用下列兩個選項之一：

- 使用位於[這裡](https://github.com/Azure/azure-quickstart-templates/blob/master/101-create-key-vault/azuredeploy.json)的 "101-Create-KeyVault" ARM 範本
- 使用 Azure PowerShell 金鑰保存庫 Cmdlet。

**附註：**如果您已為您的訂用帳戶設定金鑰保存庫，請繼續下一節。

#### 佈建金鑰加密金鑰 (選擇性)

如果您想要使用金鑰加密金鑰 (KEK) 獲得額外層次的安全性，以包裝 BitLocker 加密金鑰，您應將 KEK 加入您的金鑰保存庫，才能在佈建程序中使用。使用 [Add-AzureKeyVaultKey](https://msdn.microsoft.com/library/dn868048.aspx) Cmdlet 可在金鑰保存庫中建立新的金鑰加密金鑰。如需詳細資訊，請參閱[金鑰保存庫文件](https://azure.microsoft.com/documentation/services/key-vault/)。

    Add-AzureKeyVaultKey [-VaultName] <string> [-Name] <string> -Destination <string> {HSM | Software}

#### 設定金鑰保存庫權限以允許 Azure 平台存取金鑰和密碼

Azure 平台需要存取您的 Azure 金鑰保存庫中的加密金鑰或密碼，才能讓它們可供 VM 用來開機和解密磁碟區。若要授與權限給 Azure 平台，使它可以存取金鑰保存庫，必須在金鑰保存庫上設定 *enabledForDiskEncryption* 屬性。您可以使用金鑰保存庫 PS Cmdlet，在您的金鑰保存庫上設定 enabledForDiskEncryption 屬性：

    Set-AzureRmKeyVaultAccessPolicy -VaultName <yourVaultName> -ResourceGroupName <yourResourceGroup> -EnabledForDiskEncryption

您必須如之前所提及，在您的金鑰保存庫上設定 *enabledForDiskEncryption* 屬性。您可以透過瀏覽 https://resources.azure.com 來設定屬性。確定該以上詳述的屬性已正確設定，否則部署將會失敗。

#### 在 Azure Active Directory 中設定 Azure AD 應用程式

若需要在 Azure 中執行中的 VM 上啟用加密，Azure 磁碟加密會產生並將加密金鑰寫入金鑰保存庫。在金鑰保存庫中管理加密金鑰需要 Azure AD 驗證。

基於此目的，應該建立 Azure AD 應用程式。註冊應用程式的詳細步驟可以在這裡找到，在此[部落格文章](http://blogs.technet.com/b/kv/archive/2015/06/02/azure-key-vault-step-by-step.aspx)的＜取得應用程式的身分識別＞一節中。這篇文章也包含一些有關佈建及設定金鑰保存庫的實用範例。針對驗證目的，可以使用用戶端密碼式驗證或用戶端憑證式 Azure AD 驗證。

##### Azure AD 的用戶端密碼式驗證

下列各節含有為 Azure AD 設定用戶端密碼型驗證的必要步驟。

##### 使用 Azure PowerShell 建立新 Azure AD 應用程式

使用以下的 PowerShell Cmdlet 來建立新的 Azure AD 應用程式：

    $aadClientSecret = “yourSecret”
    $azureAdApplication = New-AzureRmADApplication -DisplayName "<Your Application Display Name>" -HomePage "<https://YourApplicationHomePage>" -IdentifierUris "<https://YouApplicationUri>" -Password $aadClientSecret
    $servicePrincipal = New-AzureRmADServicePrincipal –ApplicationId $azureAdApplication.ApplicationId

**附註：**$azureAdApplication.ApplicationId 是 Azure AD ClientID，而 $aadClientSecret 是用戶端密碼，您稍後應該會用該資訊來啟用 ADE。您應該適當地保護 Azure AD 用戶端密碼。


##### 從 Azure 服務管理入口網站佈建 Azure AD 用戶端識別碼和密碼

Azure AD 用戶端識別碼和密碼也可使用在 https://manage.windowsazure.com 的 Azure 服務管理入口網站來佈建，請遵循下列步驟來執行這項工作：

1\. 按一下 [Active Directory] 索引標籤，如下圖所示：

![Azure 磁碟加密](./media/azure-security-disk-encryption/disk-encryption-fig3.JPG)

2\. 按一下 [加入應用程式]，並輸入如下所示的應用程式名稱：

![Azure 磁碟加密](./media/azure-security-disk-encryption/disk-encryption-fig4.JPG)

3\. 按一下箭號按鈕並設定應用程式的屬性，如下所示：

![Azure 磁碟加密](./media/azure-security-disk-encryption/disk-encryption-fig5.JPG)

4\. 按一下左下角的核取記號來完成。應用程式的組態頁面隨即出現。注意 Azure AD 用戶端識別碼位於頁面底部，如下圖所示。

![Azure 磁碟加密](./media/azure-security-disk-encryption/disk-encryption-fig6.JPG)

5\. 按一下 [儲存] 按鈕來儲存 Azure AD 用戶端密碼。按一下 [儲存] 按鈕並記下 [金鑰] 文字方塊中的密碼，這是 Azure AD 用戶端密碼。您應該適當地保護 Azure AD 用戶端密碼。

![Azure 磁碟加密](./media/azure-security-disk-encryption/disk-encryption-fig7.JPG)


**注意：**入口網站不支援上述的流程。

##### 使用現有的應用程式

為了執行下列命令，您需要 Azure AD PowerShell 模組，其可從[這裡](https://technet.microsoft.com/library/jj151815.aspx)取得。

**附註：**必須從新的 PowerShell 視窗執行下列命令。請勿使用 Azure PowerShell 或 Azure Resource Manager 視窗來執行這些命令。這項建議的原因是因為這些 Cmdlet 在 MSOnline 模組或 Azure AD PowerShell 中。

    $clientSecret = ‘<yourAadClientSecret>’
    $aadClientID = '<Client ID of your AAD app>'
    connect-msolservice
    New-MsolServicePrincipalCredential -AppPrincipalId $aadClientID -Type password -Value $clientSecret

#### Azure AD 的憑證式驗證

下列各節含有為 Azure AD 設定憑證型驗證的必要步驟。

##### 建立新 Azure AD 應用程式

執行以下的 PowerShell Cmdlet 來建立新的 Azure AD 應用程式：

**附註：** 將「您的密碼」字串取代為您的安全密碼，並設定密碼保護。

    $cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate("C:\certificates\examplecert.pfx", "yourpassword")
    $keyValue = [System.Convert]::ToBase64String($cert.GetRawCertData())
    $azureAdApplication = New-AzureRmADApplication -DisplayName "<Your Application Display Name>" -HomePage "<https://YourApplicationHomePage>" -IdentifierUris "<https://YouApplicationUri>" -KeyValue $keyValue -KeyType AsymmetricX509Cert
    $servicePrincipal = New-AzureRmADServicePrincipal –ApplicationId $azureAdApplication.ApplicationId

一旦您完成此步驟，請將 .pfx 檔案上傳至金鑰保存庫，並啟用將該憑證部署到 VM 所需的存取原則。

##### 使用現有的 Azure AD 應用程式
如果您要為現有應用程式設定憑證式驗證，請使用以下的 PowerShell Cmdlet。務必從新的 PowerShell 視窗執行它們。

    $certLocalPath = 'C:\certs\myaadapp.cer'
    $aadClientID = '<Client ID of your AAD app>'
    connect-msolservice
    $cer = New-Object System.Security.Cryptography.X509Certificates.X509Certificate
    $cer.Import($certLocalPath)
    $binCert = $cer.GetRawCertData()
    $credValue = [System.Convert]::ToBase64String($binCert);
    New-MsolServicePrincipalCredential -AppPrincipalId $aadClientID -Type asymmetric -Value $credValue -Usage verify

一旦您完成此步驟，請將 .pfx 檔案上傳至金鑰保存庫，並啟用將該憑證部署到 VM 所需的存取原則。

##### 上傳 PFX 檔案至金鑰保存庫
如需此程序如何運作的詳細說明，您可以閱讀此[部落格文章](http://blogs.technet.com/b/kv/archive/2015/07/14/vm_2d00_certificates.aspx)。不過，對於這項工作您只需要下列 PowerShell Cmdlet。務必從 Azure PowerShell 主控台執行它們：

**附註：** 將「您的密碼」字串取代為您的安全密碼，並設定密碼保護。

    $certLocalPath = 'C:\certs\myaadapp.pfx'
    $certPassword = "yourpassword"
    $resourceGroupName = ‘yourResourceGroup’
    $keyVaultName = ‘yourKeyVaultName’
    $keyVaultSecretName = ‘yourAadCertSecretName’

    $fileContentBytes = get-content $certLocalPath -Encoding Byte
    $fileContentEncoded = [System.Convert]::ToBase64String($fileContentBytes)

    $jsonObject = @"
    {
    "data": "$filecontentencoded",
    "dataType" :"pfx",
    "password": "$certPassword"
    }
    "@

    $jsonObjectBytes = [System.Text.Encoding]::UTF8.GetBytes($jsonObject)
    $jsonEncoded = [System.Convert]::ToBase64String($jsonObjectBytes)

    Switch-AzureMode -Name AzureResourceManager
    $secret = ConvertTo-SecureString -String $jsonEncoded -AsPlainText -Force
    Set-AzureKeyVaultSecret -VaultName $keyVaultName -Name $keyVaultSecretName -SecretValue $secret
    Set-AzureRmKeyVaultAccessPolicy -VaultName $keyVaultName -ResourceGroupName $resourceGroupName –EnabledForDeployment

##### 將金鑰保存庫中的憑證部署到現有 VM
完成上傳 PFX 之後，請使用下列步驟來將金鑰保存庫中的憑證部署到現有 VM：

    $resourceGroupName = ‘yourResourceGroup’
    $keyVaultName = ‘yourKeyVaultName’
    $keyVaultSecretName = ‘yourAadCertSecretName’
    $vmName = ‘yourVMName’
    $certUrl = (Get-AzureKeyVaultSecret -VaultName $keyVaultName -Name $keyVaultSecretName).Id
    $sourceVaultId = (Get-AzureRmKeyVault -VaultName $keyVaultName -ResourceGroupName $resourceGroupName).ResourceId
    $vm = Get-AzureRmVM -ResourceGroupName $resourceGroupName -Name $vmName
    $vm = Add-AzureRmVMSecret -VM $vm -SourceVaultId $sourceVaultId -CertificateStore "My" -CertificateUrl $certUrl
    Update-AzureRmVM -VM $vm  -ResourceGroupName $resourceGroupName


#### 設定 Azure AD 應用程式的金鑰保存庫存取原則

您的 Azure AD 應用程式需要權限，才能存取保存庫中的金鑰或密碼。使用 [Set-AzureKeyVaultAccessPolicy](https://msdn.microsoft.com/library/azure/dn903607.aspx) Cmdlet 可授與應用程式權限，使用用戶端識別碼 (登錄應用程式時所產生) 做為 -ServicePrincipalName 參數值。您可以閱讀[此部落格文章](http://blogs.technet.com/b/kv/archive/2015/06/02/azure-key-vault-step-by-step.aspx)，以取得相關的一些範例。下面還有如何透過 PowerShell 執行這項工作中的範例：

    $keyVaultName = ‘yourKeyVaultName’
    $aadClientID = '<youAadAppClientID>'
    Set-AzureRmKeyVaultAccessPolicy -VaultName $keyVaultName -ServicePrincipalName $aadClientID -PermissionsToKeys all -PermissionsToSecrets all

## 術語

使用術語表做為了解一些此技術所使用的一般術語的參考：


| 術語 | 定義 |
|-----------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Azure AD | Azure AD [是 Azure Active Directory](https://azure.microsoft.com/documentation/services/active-directory/)。Azure AD 帳戶是從金鑰保存庫驗證、儲存和擷取密碼的必要條件。 |
| Azure 金鑰保存庫 [AKV] | Azure 金鑰保存庫是密碼編譯金鑰管理服務，其基於 FIPS 驗證的硬體安全性模組來安全地保護您的密碼編譯金鑰和敏感性密碼。如需詳細資料，請參閱[金鑰保存庫](https://azure.microsoft.com/services/key-vault/)文件。 |
| ARM | Azure Resource Manager |
| BitLocker | [BitLocker](https://technet.microsoft.com/library/hh831713.aspx) 是一種業界認可的 Windows 磁碟區加密技術，用來在 Windows IaaS VM 上啟用磁碟加密 |
| BEK | BitLocker 加密金鑰可用來加密作業系統開機磁碟區和資料磁碟區。BitLocker 金鑰會在客戶的 Azure 金鑰保存庫中以密碼形式保護。 |
| CLI | [Azure 命令列介面](../xplat-cli-install.md) |
| DM-Crypt | [DM-Crypt](https://en.wikipedia.org/wiki/Dm-crypt) 是基於 Linux 的透明磁碟加密子系統，用來在 Linux IaaS VM 上啟用磁碟加密 |
| KEK | 金鑰加密金鑰是非對稱金鑰 (RSA 2048)，用來在需要時保護或包裝密碼。您可以提供 HSM 保護的金鑰或軟體保護的金鑰。如需詳細資訊，請參閱 [Azure 金鑰保存庫](https://azure.microsoft.com/services/key-vault/)文件 |
| PS Cmdlet | [Azure PowerShell Cmdlet](powershell-install-configure.md) |

## 磁碟加密部署案例和使用者體驗

在許多案例下您可以啟用磁碟加密，而步驟可能會因案例而有所不同。下列各節將對這些案例的詳細資料進一步討論。

### 對透過 Azure 資源庫建立的新 IaaS VM 啟用加密

可以使用[這裡](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-create-new-vm-gallery-image)發行的 ARM 範本，從 Azure 中的 Azure 資源庫啟用新 IaaS Windows VM 上的磁碟加密。按一下 Azure 快速入門範本上的 [部署至 Azure] 按鈕，在參數刀鋒視窗中輸入加密組態，然後按一下 [確定]。選取訂用帳戶、資源群組、資源群組位置、法律條款與協議，並按一下 [建立] 按鈕以在新 IaaS VM 上啟用加密。

**附註：**此範本會使用 Windows Server 2012 的資源庫映像來建立新的加密 Windows VM。

您可以在下表中查看使用 Azure AD 用戶端識別碼、來自 Azure 資源庫案例的新 VM 的 ARM 範本參數詳細資料：

| 參數 | 說明|
|-------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| adminUserName | 虛擬機器的系統管理使用者名稱 |
| adminPassword | 虛擬機器的系統管理使用者密碼 |
| newStorageAccountName | 要儲存作業系統和資料 VHD 的儲存體帳戶名稱 |
| vmSize | VM 的大小。目前僅支援標準的 A、D 和 G 系列 |
| virtualNetworkName | VM NIC 應該隸屬的 VNet 的名稱。 |
| subnetName | VM NIC 應該隸屬的 VNet 中子網路的名稱 |
| AADClientID | 具有權限可將密碼寫入金鑰保存庫之 Azure AD 應用程式的用戶端識別碼 |
| AADClientSecret | 具有權限可將密碼寫入金鑰保存庫之 Azure AD 應用程式的用戶端密碼 |
| keyVaultResourceID,ResourceID | 識別 ARM 中的金鑰保存庫資源。您可以使用 PowerShell Cmdlet 取得它：(Get-AzureRmKeyVault -VaultName,-ResourceGroupName ).ResourceId |
| keyVaultURL | 應上傳 Bitlocker 金鑰的金鑰保存庫的 URL。您可以使用 Cmdlet 取得它：(Get-AzureRmKeyVault -VaultName,-ResourceGroupName ).VaultURI |
| keyEncryptionKeyURL | 用來加密所產生 BitLocker 金鑰的金鑰加密金鑰的 URL。這是選擇性。 |
| vmName | 要執行加密作業所在 VM 的名稱


**附註：**KeyEncryptionKeyURL 是選擇性參數。您可以使用自己的 KEK，在金鑰保存庫中進一步保護資料加密金鑰 (複雜密碼)。

### 對透過客戶加密的 VHD 和加密金鑰建立的新 IaaS VM 啟用加密

在這個案例中，您可以使用 ARM 範本、PowerShell Cmdlet 或 CLI 命令啟用加密。下列各節將更進一步說明 ARM 範本和 CLI 命令的詳細資料。

#### 使用 ARM 範本

可以使用[這裡](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-create-pre-encrypted-vm)發行的 ARM 範本，在客戶加密的 VHD 上啟用磁碟加密。按一下 Azure 快速入門範本上的 [部署至 Azure] 按鈕，在參數刀鋒視窗中輸入加密組態，然後按一下 [確定]。選取訂用帳戶、資源群組、資源群組位置、法律條款與協議，並按一下 [建立] 按鈕以在新 IaaS VM 上啟用加密。

下表將描述客戶加密的 VHD 案例的 ARM 範本參數詳細資料：

| 參數 | 說明|
|-------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| newStorageAccountName | 要儲存加密的作業系統 VHD 的儲存體帳戶名稱。應該已在與 VM 相同的資源群組和相同的位置中建立此儲存體帳戶 |
| osVhdUri | 來自儲存體帳戶作業系統 VHD 的 URI |
| osType | 作業系統產品類型 (Windows/Linux) |
| virtualNetworkName | VM NIC 應該隸屬的 VNet 的名稱。應該已在與 VM 相同的資源群組和相同的位置中建立此項目 |
| subnetName | VM NIC 應該隸屬的 VNet 中子網路的名稱 |
| vmSize | VM 的大小。目前僅支援標準的 A、D 和 G 系列 |
| keyVaultResourceID | 識別 ARM 中金鑰保存庫資源的 ResourceID。您可以使用 PowerShell Cmdlet 取得它：(Get-AzureRmKeyVault -VaultName &lt;yourKeyVaultName&gt; -ResourceGroupName &lt;yourResourceGroupName&gt;).ResourceId |
| keyVaultSecretUrl | ​金鑰保存庫中佈建的磁碟加密金鑰的 URL |
| keyVaultKekUrl | 用來加密所產生磁碟加密金鑰的金鑰加密金鑰的 URL |
| ​vmName | ​IaaS VM 的名稱   



####使用 PowerShell Cmdlet

可以使用[這裡](https://msdn.microsoft.com/library/azure/mt603746.aspx)發行的 PS Cmdlet，在客戶加密的 VHD 上啟用磁碟加密。

####使用 CLI 命令

遵循下列步驟使用 CLI 命令啟用此案例的磁碟加密：

1. 設定金鑰保存庫的存取原則：
	- 設定 ‘EnabledForDiskEncryption’ 旗標：“azure keyvault set-policy --vault-name <keyVaultName> --enabled-for-disk-encryption true”
	- 設定權限給 Azure AD 應用程式，以將密碼寫入 KeyVault：“azure keyvault set-policy --vault-name <keyVaultName> --spn <aadClientID> --perms-to-keys ["all"] --perms-to-secrets ["all"]”
2. 若要啟用現有/執行中 VM 上的加密，請輸入：azure vm enable-disk-encryption --resource-group <resourceGroupName> --name <vmName> --aad-client-id <aadClientId> --aad-client-secret <aadClientSecret> --disk-encryption-key-vault-url <keyVaultURL> --disk-encryption-key-vault-id <keyVaultResourceId>
3. 取得加密狀態：“azure vm show-disk-encryption-status --resource-group <resourceGroupName> --name <vmName> --json”
4. 若要從客戶加密的 VHD 啟用新 VM 上的加密，請使用以下參數搭配 “azure vm create” 命令：
	- disk-encryption-key-vault-id <disk-encryption-key-vault-id>
	- disk-encryption-key-url <disk-encryption-key-url>
	- key-encryption-key-vault-id <key-encryption-key-vault-id>
	- key-encryption-key-url <key-encryption-key-url>


### 在 Azure 中現有或執行中的 IaaS Windows VM 上啟用加密

在這個案例中，您可以使用 ARM 範本、PowerShell Cmdlet 或 CLI 命令啟用加密。下列各節將更進一步說明如何使用 ARM 範本和 CLI 命令來啟用它的詳細資料。

#### 使用 ARM 範本

可以使用[這裡](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-windows-vm)發行的 ARM 範本，在 Azure 中現有/執行中 IaaS Windows VM 上啟用磁碟加密。按一下 Azure 快速入門範本上的 [部署至 Azure] 按鈕，在參數刀鋒視窗中輸入加密組態，然後按一下 [確定]。選取訂用帳戶、資源群組、資源群組位置、法律條款與協議，並按一下 [建立] 按鈕以在現有/執行中 IaaS VM 上啟用加密。

使用 Azure AD 用戶端識別碼的現有/執行中 VM 案例的 ARM 範本參數詳細資料，可在下表中取得：

| 參數 | 說明|
|-------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ​AADClientID | ​具有權限可將密碼寫入金鑰保存庫之 Azure AD 應用程式的用戶端識別碼 |
| AADClientSecret | ​具有權限可將密碼寫入金鑰保存庫之 Azure AD 應用程式的用戶端密碼 |
| keyVaultName | 應上傳 Bitlocker 金鑰的金鑰保存庫的名稱。您可以使用 Cmdlet 取得它：(Get-AzureRmKeyVault -ResourceGroupName <yourResourceGroupName>)。Vaultname |
| ​ keyEncryptionKeyURL | 用來加密所產生 BitLocker 金鑰的金鑰加密金鑰的 URL。如果您在 UseExistingKek 下拉式清單中選取 [nokek]，這是選擇性的。如果您在 UseExistingKek 下拉式清單中選取 [kek]，您必須輸入 keyEncryptionKeyURL 值。 |
| ​volumeType | ​執行加密作業所在磁碟區的類型。有效值為 "OS"、"Data"、"All" |
| sequenceVersion | BitLocker 作業的順序版本。每當在相同的 VM 上執行磁碟加密作業時便遞增此版本號碼 |
| ​vmName | ​要執行加密作業所在 VM 的名稱


**附註：**KeyEncryptionKeyURL 是選擇性參數。您可以使用自己的 KEK，在金鑰保存庫中進一步保護資料加密金鑰 (BitLocker 加密密碼)。

#### 使用 PowerShell Cmdlet

如需如何使用 PS Cmdlet 啟用使用 Azure 磁碟加密的加密的詳細資訊，請參閱**使用 Azure PowerShell 探索 Azure 磁碟加密**部落格文章[第 1 部分](http://blogs.msdn.com/b/azuresecurity/archive/2015/11/17/explore-azure-disk-encryption-with-azure-powershell.aspx) 和 [第 2 部分](http://blogs.msdn.com/b/azuresecurity/archive/2015/11/21/explore-azure-disk-encryption-with-azure-powershell-part-2.aspx)。

#### 使用 CLI 命令

請依照下列步驟，在 Azure 中使用 CLI 命令來啟用現有/執行中 IaaS Windows VM 上的加密：

1. 設定金鑰保存庫的存取原則：
	- 設定 ‘EnabledForDiskEncryption’ 旗標：“azure keyvault set-policy --vault-name <keyVaultName> --enabled-for-disk-encryption true”
	- 設定權限給 Azure AD 應用程式，以將密碼寫入 KeyVault：“azure keyvault set-policy --vault-name <keyVaultName> --spn <aadClientID> --perms-to-keys ["all"] --perms-to-secrets ["all"]”
2. 若要啟用現有/執行中 VM 上的加密，請輸入：azure vm enable-disk-encryption --resource-group <resourceGroupName> --name <vmName> --aad-client-id <aadClientId> --aad-client-secret <aadClientSecret> --disk-encryption-key-vault-url <keyVaultURL> --disk-encryption-key-vault-id <keyVaultResourceId>
3. 取得加密狀態：“azure vm show-disk-encryption-status --resource-group <resourceGroupName> --name <vmName> --json”
4. 若要從客戶加密的 VHD 啟用新 VM 上的加密，請使用以下參數搭配 “azure vm create” 命令：
	- disk-encryption-key-vault-id <disk-encryption-key-vault-id>
	- disk-encryption-key-url <disk-encryption-key-url>
	- key-encryption-key-vault-id <key-encryption-key-vault-id>
	- key-encryption-key-url <key-encryption-key-url>


### 在 Azure 中現有或執行中的 IaaS Linux VM 上啟用加密

可以使用[這裡](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-linux-vm)發行的 ARM 範本，在 Azure 中現有/執行中 IaaS Linux VM 上啟用磁碟加密。按一下 Azure 快速入門範本上的 [部署至 Azure] 按鈕，在參數刀鋒視窗中輸入加密組態，然後按一下 [確定]。選取訂用帳戶、資源群組、資源群組位置、法律條款與協議，並按一下 [建立] 按鈕以在現有/執行中 IaaS VM 上啟用加密。

使用 Azure AD 用戶端識別碼的現有/執行中 VM 案例的 ARM 範本參數詳細資料，將在下表中說明：

| 參數 | 說明|
|-------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ​AADClientID | ​具有權限可將密碼寫入金鑰保存庫之 Azure AD 應用程式的用戶端識別碼 |
| AADClientSecret | ​具有權限可將密碼寫入金鑰保存庫之 Azure AD 應用程式的用戶端密碼 |
| keyVaultName | 應上傳 Bitlocker 金鑰的金鑰保存庫的名稱。您可以使用 Cmdlet 取得它：(Get-AzureRmKeyVault -ResourceGroupName <yourResourceGroupName>)。Vaultname |
| ​ keyEncryptionKeyURL | 用來加密所產生 BitLocker 金鑰的金鑰加密金鑰的 URL。如果您在 UseExistingKek 下拉式清單中選取 [nokek]，這是選擇性的。如果您在 UseExistingKek 下拉式清單中選取 [kek]，您必須輸入 keyEncryptionKeyURL 值。 |
| ​volumeType | ​執行加密作業所在磁碟區的類型。有效的支援值為 "Data"。Linux VM 不支援在執行 Linux VM 的作業系統磁碟區上啟用加密 |
| sequenceVersion | BitLocker 作業的順序版本。每當在相同的 VM 上執行磁碟加密作業時便遞增此版本號碼 |
| ​vmName | ​要執行加密作業所在 VM 的名稱
| passPhrase | 輸入強式複雜密碼做為資料加密金鑰 |                                                                                                                                                                                                                                                      

**附註：**KeyEncryptionKeyURL 是選擇性參數。您可以使用自己的 KEK，在金鑰保存庫中進一步保護資料加密金鑰 (複雜密碼)。

#### CLI 命令

可以使用透過[這裡](../xplat-cli-install.md)安裝的 CLI 命令，在客戶加密的 VHD 上啟用磁碟加密。請依照下列步驟，在 Azure 中使用 CLI 命令來啟用現有/執行中 IaaS Linux VM 上的加密：

1. 設定金鑰保存庫的存取原則：
	- 設定 ‘EnabledForDiskEncryption’ 旗標：“azure keyvault set-policy --vault-name <keyVaultName> --enabled-for-disk-encryption true”
	- 設定權限給 Azure AD 應用程式，以將密碼寫入 KeyVault：“azure keyvault set-policy --vault-name <keyVaultName> --spn <aadClientID> --perms-to-keys ["all"] --perms-to-secrets ["all"]”
2. 若要啟用現有/執行中 VM 上的加密，請輸入：azure vm enable-disk-encryption --resource-group <resourceGroupName> --name <vmName> --aad-client-id <aadClientId> --aad-client-secret <aadClientSecret> --disk-encryption-key-vault-url <keyVaultURL> --disk-encryption-key-vault-id <keyVaultResourceId>
3. 取得加密狀態：“azure vm show-disk-encryption-status --resource-group <resourceGroupName> --name <vmName> --json”
4. 若要從客戶加密的 VHD 啟用新 VM 上的加密，請使用以下參數搭配 “azure vm create” 命令。
	- *disk-encryption-key-vault-id <disk-encryption-key-vault-id>*
	- *disk-encryption-key-url <disk-encryption-key-url>*
	- *key-encryption-key-vault-id <key-encryption-key-vault-id>*
	- *key-encryption-key-url <key-encryption-key-url>*

### 取得加密的 IaaS VM 的加密狀態

您可以使用 Azure 管理入口網站、[PowerShell Cmdlet](https://msdn.microsoft.com/library/azure/mt622700.aspx) 或 CLI 命令取得加密狀態。下列章節將說明如何使用 Azure 入口網站和 CLI 命令來取得加密狀態。

#### 使用 Azure 管理入口網站取得加密 IaaS VM 的加密狀態

您可以從 Azure 管理入口網站取得 IaaS VM 的加密狀態。登入位於 https://portal.azure.com/ 的 Azure 入口網站，在左功能表中的虛擬機器連結上按一下以查看您的訂用帳戶中虛擬機器的摘要檢視。您可以從 [訂用帳戶] 下拉式清單中選取訂用帳戶名稱，以篩選虛擬機器檢視。按一下位於虛擬機器頁面功能表頂端的資料行。從選擇資料行刀鋒視窗選取 [磁碟加密] 資料行，並按一下 [更新]。您應該會看到對於每個 VM，磁碟加密資料行顯示加密狀態為「已啟用」或「未啟用」，如下圖所示。

![Azure 中的 Microsoft Antimalware](./media/azure-security-disk-encryption/disk-encryption-fig2.JPG)

#### 使用磁碟加密 PS Cmdlet 取得加密 IaaS VM 的加密狀態
您可以透過磁碟加密 PS Cmdlet “Get-AzureRmVMDiskEncryptionStatus” 來取得 IaaS VM 的加密狀態。若要取得您的 VM 的加密設定，請在您的 Azure PowerShell 工作階段中輸入：

    PS C:\Windows\System32\WindowsPowerShell\v1.0> Get-AzureRmVMDiskEncryptionStatus -ResourceGroupName <yourResourceGroupName> -VMName <yourVMName>

    OsVolumeEncrypted: True
    OsVolumeEncryptionSettings : {
      "DiskEncryptionKey": {
       SecretUrl":"https://contosovault.vault.azure.net/secrets/BitLockerEncryptionSecretWithKek/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
            "SourceVault": {
            "ReferenceUri": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/xxxxxxx/providers/Mi                            crosoft.KeyVault/vaults/xxxxxxx"
                                }
                            },
                    "KeyEncryptionKey": null
                             }
    DataVolumesEncrypted: True

OSVolumeEncrypted 和 DataVolumesEncrypted 設定值設定為 "True"，顯示這兩個磁碟區都是使用 Azure 磁碟加密來加密。如需如何使用 PS Cmdlet 啟用使用 Azure 磁碟加密的加密的詳細資訊，請參閱**使用 Azure PowerShell 探索 Azure 磁碟加密**部落格文章[第 1 部分](http://blogs.msdn.com/b/azuresecurity/archive/2015/11/17/explore-azure-disk-encryption-with-azure-powershell.aspx) 和 [第 2 部分](http://blogs.msdn.com/b/azuresecurity/archive/2015/11/21/explore-azure-disk-encryption-with-azure-powershell-part-2.aspx)。

#### 透過磁碟加密 CLI 命令取得 IaaS VM 的加密狀態。

您可以透過磁碟加密 CLI 命令 *azure vm show-disk-encryption-status* 取得 IaaS VM 的加密狀態。若要取得您的 VM 的加密設定，請在您的 Azure CLI 工作階段中輸入：

    azure vm show-disk-encryption-status --resource-group <yourResourceGroupName> --name <yourVMName> --json  

#### 在執行中 Windows IaaS VM 上停用加密

您可以透過 Azure 磁碟加密 ARM 範本或 PS Cmdlet，在執行中 Windows IaaS VM 上停用加密，並指定解密組態。只有 Windows IaaS VM 支援停用加密步驟，而 Linux IaaS VM 不提供支援。停用加密步驟會在執行中 Windows IaaS VM 上停用 OS 或資料磁碟區或兩者的加密。您無法停用 OS 磁碟區並將資料磁碟區保持加密狀態。執行停用加密步驟時，Azure 服務管理會更新 VM 服務模型，且 Windows IaaS VM 會標示為已解密。VM 的待用內容不會再加密。停用加密並不會刪除客戶金鑰保存庫和加密金鑰資料 (適用於 Windows 的 BitLocker 加密金鑰和適用於 Linux 的複雜密碼)。

##### 使用 ARM 範本在 Azure 中的現有/執行中 IaaS Windows 上停用加密

可以使用[這裡](https://github.com/Azure/azure-quickstart-templates/tree/master/201-decrypt-running-windows-vm)發行的 ARM 範本，在執行中 Windows IaaS VM 上停用磁碟加密。按一下 Azure 快速入門範本上的 [部署至 Azure] 按鈕，在參數刀鋒視窗中輸入解密組態，然後按一下 [確定]。選取訂用帳戶、資源群組、資源群組位置、法律條款與協議，並按一下 [建立] 按鈕以在新 IaaS VM 上啟用加密。

可供在執行中 Windows IaaS VM 上停用加密的 ARM 範本參數詳細資料︰

| ​vmName | ​要執行加密作業所在 VM 的名稱 |
|-----------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ​volumeType | ​執行解密作業所在磁碟區的類型。有效值為 "OS"、"Data"、"All"。**注意︰**若未在 "Data" 磁碟區上停用加密，則無法在執行中 Windows IaaS VM OS/開機磁碟區上停用加密。 |
| sequenceVersion | BitLocker 作業的順序版本。每當在相同的 VM 上執行磁碟解密作業時便遞增此版本號碼 |

##### 使用 PS Cmdlet 在 Azure 中的現有/執行中 IaaS Windows 上停用加密

若要使用 PS Cmdlet 進行停用，[Disable-AzureRmVMDiskEncryption](https://msdn.microsoft.com/library/azure/mt715776.aspx) Cmdlet 會在基礎結構即服務 (IaaS) 虛擬機器上停用加密。只有 Windows 虛擬機器支援此 Cmdlet，而在 Linux 虛擬機器不提供支援。此 Cmdlet 會在虛擬機器上安裝延伸模組以停用加密。如果未指定 Name 參數，則會建立使用預設名稱 "AzureDiskEncryption for Windows VMs" 的延伸模組。

**注意**︰此 Cmdlet 會重新啟動虛擬機器。

## 附錄

### 連線至您的訂用帳戶

繼續之前，請務必檢閱本文件的＜必要條件＞一節。確保已滿足所有必要條件之後，請依照下列步驟來連接到您的訂用帳戶：

1\. 開始 Azure PowerShell 工作階段，並使用下列命令登入您的 Azure 帳戶：

    Login-AzureRmAccount

2\. 如果您有多個訂用帳戶，並想要指定要使用的特定一個訂用帳戶，請輸入下列內容以查看您帳戶的訂用帳戶：

    Get-AzureRmSubscription

3\. 若要指定要使用的訂用帳戶，請輸入：

    Select-AzureRmSubscription -SubscriptionName <Yoursubscriptionname>

4\. 若要確認訂用帳戶設定正確無誤，請輸入：

    Get-AzureRmSubscription

5\. 若要確認已安裝 Azure 磁碟加密 Cmdlet，請輸入：

    Get-command *diskencryption*

6\. 您應該會看到確認 Azure 磁碟加密 PowerShell 安裝的下列輸出：

    PS C:\Windows\System32\WindowsPowerShell\v1.0> get-command *diskencryption*
    CommandType  Name                               	 Version    Source                                                             
    Cmdlet       Get-AzureRmVMDiskEncryptionStatus       1.1.0      AzureRM.Compute                                                    
    Cmdlet       Remove-AzureRmVMDiskEncryptionExtension 1.1.0      AzureRM.Compute                                                    
    Cmdlet       Set-AzureRmVMDiskEncryptionExtension    1.1.0      AzureRM.Compute                                                     

### 準備已預先加密的 Windows VHD
下列各節是準備預先加密的 Windows VHD 以在 Azure IaaS 中部署為加密的 VHD 的必要項目。這些步驟可用來準備和啟動 Hyper-V 或 Azure 上的全新 Windows VM (VHD)。

#### 更新群組原則以對作業系統保護允許非 TPM
您需要設定稱為「BitLocker 磁碟機加密」的 BitLocker 群組原則設定，其位於此路徑：本機電腦原則\\電腦設定\\系統管理範本\\Windows 元件。如下圖所示，將這個設定變更為：*作業系統磁碟機 - 啟動時需要其他驗證 - 在不含相容 TPM 的情形下允許使用 BitLocker*：

![Azure 中的 Microsoft Antimalware](./media/azure-security-disk-encryption/disk-encryption-fig8.JPG)

#### 安裝 BitLocker 功能元件
針對 Windows Server 2012 和以上版本，請使用下列命令：

    dism /online /Enable-Feature /all /FeatureName:Bitlocker /quiet /norestart

針對 Windows Server 2008 R2，請使用下列命令：

    ServerManagerCmd -install BitLockers

#### 使用 bdehdcfg 準備用於 BitLocker 的作業系統磁碟區

執行下面的命令來壓縮作業系統磁碟分割及為 BitLocker 準備電腦。

    bdehdcfg -target c: shrink -quiet

#### 使用 BitLocker 保護作業系統磁碟區
使用 [manage-bde](https://technet.microsoft.com/library/ff829849.aspx) 命令以使用外部的金鑰保護裝置在開機磁碟區上啟用加密，並將外部金鑰 (.bek 檔) 放在外部磁碟機或磁碟區上。下次重新開機後，將會在系統/開機磁碟區上啟用加密。

    manage-bde -on %systemdrive% -sk [ExternalDriveOrVolume]
    reboot

**附註：**必須先使用不同的資料/資源 VHD 準備 VM，才能使用 BitLocker 取得外部金鑰。

#### 準備已預先加密的 Linux VHD

##### Ubuntu 14.

1\. 在 /usr/local/sbin/azure\_crypt\_key.sh 下，使用以下指令碼的內容建立檔案。注意 KeyFileName，因為它是 Azure 放置的複雜密碼檔案名稱。

    #!/bin/sh
    MountPoint=/tmp-keydisk-mount
    KeyFileName=LinuxPassPhraseFileName
    echo "Trying to get the key from disks ..." >&2
    mkdir -p $MountPoint
    modprobe vfat >/dev/null 2>&1
    modprobe ntfs >/dev/null 2>&1
    sleep 2
    OPENED=0
    cd /sys/block
    for DEV in sd*; do
        echo "> Trying device: $DEV ..." >&2
        mount -t vfat -r /dev/${DEV}1 $MountPoint >/dev/null||
        mount -t ntfs -r /dev/${DEV}1 $MountPoint >/dev/null
        if [ -f $MountPoint/$KeyFileName ]; then
                cat $MountPoint/$KeyFileName
                umount $MountPoint 2>/dev/null
                OPENED=1
                break
        fi
        umount $MountPoint 2>/dev/null
    done

      if [ $OPENED -eq 0 ]; then
        echo "FAILED to find suitable passphrase file ..." >&2
        echo -n "Try to enter your password: " >&2
        read -s -r A </dev/console
        echo -n "$A"
     else
        echo "Success loading keyfile!" >&2
    fi


2\. 在 */etc/crypttab* 中變更 crypt 設定。它看起來應該如下所示：

    Sda5_crypt uuid=xxxxxxxxxxxxxxxxxxxxx none luks,discard,keyscript=/usr/local/sbin/azure_crypt_key.sh

3\. 如果您要在 Windows 中編輯 *azure\_crypt\_key.sh* 並將它複製到 Linux，請記得執行 *dos2unix /usr/local/sbin/azure\_crypt\_key.sh*。4. 以加上程式碼行的方式編輯 */etc/initramfs-tools/modules*︰

    vfat
    ntfs
    nls_cp437
    nls_utf8
    nls_iso8859-1

5\. 執行 *update-initramfs -u -k all* 以更新 initramfs，keyscript 才能生效。

##### openSUSE 13.2.

1\. 編輯 /etc/dracut.conf add\_drivers+="vfat ntfs nls\_cp437 nls\_iso8859-1"

2\. 註解化檔案 “/usr/lib/dracut/modules.d/90crypt/module-setup.sh” 結尾的這幾行程式碼：

    #    inst_multiple -o \
    #        $systemdutildir/system-generators/systemd-cryptsetup-generator \
    #        $systemdutildir/systemd-cryptsetup \
    #        $systemdsystemunitdir/systemd-ask-password-console.path \
    #        $systemdsystemunitdir/systemd-ask-password-console.service \
    #        $systemdsystemunitdir/cryptsetup.target \
    #        $systemdsystemunitdir/sysinit.target.wants/cryptsetup.target \
    #        systemd-ask-password systemd-tty-ask-password-agent
    #        inst_script "$moddir"/crypt-run-generator.sh /sbin/crypt-run-generator


3\. 在檔案 “/usr/lib/dracut/modules.d/90crypt/parse-crypt.sh” 開頭附加 DRACUT\_SYSTEMD=0，並將所有 “if [ -z "$DRACUT\_SYSTEMD" ]; then” 變更為 “if [ 1 ]; then”

4\. 編輯 /usr/lib/dracut/modules.d/90crypt/cryptroot-ask.sh 並將此附加在 “# Open LUKS device” 之後

    MountPoint=/tmp-keydisk-mount
    KeyFileName=LinuxPassPhraseFileName
    echo "Trying to get the key from disks ..." >&2
    mkdir -p $MountPoint >&2
    modprobe vfat >/dev/null >&2
    modprobe ntfs >/dev/null >&2
    for SFS in /dev/sd*; do
       echo "> Trying device:$SFS..." >&2
       mount ${SFS}1 $MountPoint -t vfat -r >&2 ||
       mount ${SFS}1 $MountPoint -t ntfs -r >&2
       if [ -f $MountPoint/$KeyFileName ]; then
          echo "> keyfile got..." >&2
          luksfile=$MountPoint/$KeyFileName
          break
       fi
    done

5\. 執行 “dracut –f -v” 來更新 initrd

##### CentOS 7
1\. 編輯 /etc/dracut.conf add\_drivers+=" vfat ntfs nls\_cp437 nls\_iso8859-1"

2\. 註解化檔案 “/usr/lib/dracut/modules.d/90crypt/module-setup.sh” 結尾的這幾行程式碼：

    #        inst_multiple -o \
    #        $systemdutildir/system-generators/systemd-cryptsetup-generator \
    #        $systemdutildir/systemd-cryptsetup \
    #        $systemdsystemunitdir/systemd-ask-password-console.path \
    #        $systemdsystemunitdir/systemd-ask-password-console.service \
    #        $systemdsystemunitdir/cryptsetup.target \
    #        $systemdsystemunitdir/sysinit.target.wants/cryptsetup.target \
    #        systemd-ask-password systemd-tty-ask-password-agent
    #        inst_script "$moddir"/crypt-run-generator.sh /sbin/crypt-run-generator



3\. 在檔案 “/usr/lib/dracut/modules.d/90crypt/parse-crypt.sh” 開頭附加 DRACUT\_SYSTEMD=0，並將所有 “if [ -z "$DRACUT\_SYSTEMD" ]; then” 變更為 “if [ 1 ]; then”

4\. 編輯 /usr/lib/dracut/modules.d/90crypt/cryptroot-ask.sh 並將此附加在 “# Open LUKS device” 之後

    MountPoint=/tmp-keydisk-mount
    KeyFileName=LinuxPassPhraseFileName
    echo "Trying to get the key from disks ..." >&2
    mkdir -p $MountPoint >&2
    modprobe vfat >/dev/null >&2
    modprobe ntfs >/dev/null >&2
    for SFS in /dev/sd*; do
    echo "> Trying device:$SFS..." >&2
    mount ${SFS}1 $MountPoint -t vfat -r >&2 ||
    mount ${SFS}1 $MountPoint -t ntfs -r >&2
    if [ -f $MountPoint/$KeyFileName ]; then
        echo "> keyfile got..." >&2
        luksfile=$MountPoint/$KeyFileName
        break
    fi
    done


5\. 執行 “/usr/sbin/dracut -f -v” 來更新 initrd。

###上傳加密的 VHD 至 Azure 儲存體帳戶
一旦啟用 BitLocker 加密或 DM-Crypt 加密，便需要將本機加密的 VHD 上傳至儲存體帳戶。

    Add-AzureRmVhd [-Destination] <Uri> [-LocalFilePath] <FileInfo> [[-NumberOfUploaderThreads] <Int32> ] [[-BaseImageUriToPatch] <Uri> ] [[-OverWrite]] [ <CommonParameters>]

### 上傳已預先加密 VM 的磁碟加密密碼至金鑰保存庫
必須將先前取得的磁碟加密密碼上傳，做為金鑰保存庫中的密碼。

#### 未使用 KEK 加密的磁碟加密密碼
使用 [Set-AzureKeyVaultSecret](https://msdn.microsoft.com/library/dn868050.aspx) 在金鑰保存庫中佈建密碼。如果是 Windows 虛擬機器，bek 檔案會以 base64 字串編碼，然後使用 Set-AzureKeyVaultSecret Cmdlet 上傳至金鑰保存庫。如果是 Linux，複雜密碼會以 base64 字串編碼，然後上傳至金鑰保存庫。此外，請確定在金鑰保存庫中建立密碼時會設定下列標記。

    "tags":
    {
       “DiskEncryptionKeyEncryptionAlgorithm”: “RSA-OAEP (optional)”
       "DiskEncryptionKeyFileName": "Bek file name (windows) or Passphrase filename (linux)"
    }

    param(
      [Parameter(Mandatory=$True)]
      [String]$BekFilePath = "C:\vm\nbox\2640EE52-41B3-426C-87B9-484232452CE4.BEK",
      [String]$VaultName = "DiskEncryptionTestAus",
      [String]$SecretName = "BitLockerKey"
      )

    #"EAN//ojeIQk="
    $bekFileName = split-path $BekFilePath -leaf
    echo "Bek file name = $bekFileName"

    $secretBytes = [System.IO.File]::ReadAllBytes($BekFilePath);
    $secret = [Convert]::ToBase64String($secretBytes);
    echo "Secret = $secret"

    $secureSecret = ConvertTo-SecureString $secret -AsPlainText -Force
    $tags = @{"DiskEncryptionKeyFileName" = "$bekFileName"}

    echo "Tags = $tags"
    echo "Vault = $VaultName"
    echo "Secret name = $SecretName"
    echo "Adding secret to Key vault"

    Set-AzureKeyVaultSecret -VaultName $VaultName -Name $SecretName -SecretValue $secureSecret -tags $tags


#### 使用 KEK 加密的磁碟加密密碼

可以選擇性地使用金鑰加密金鑰來加密密碼，之後再上傳到金鑰保存庫。使用包裝 [API](https://msdn.microsoft.com/library/azure/dn878066.aspx) 先加密使用金鑰加密金鑰的密碼。這個包裝作業的輸出是 base64 URL 編碼的字串，然後使用 [Set-AzureKeyVaultSecret](https://msdn.microsoft.com/library/dn868050.aspx) Cmdlet 將它上傳做為密碼。


##下載此指南
您可以從 [TechNet 資源庫](https://gallery.technet.microsoft.com/Azure-Disk-Encryption-for-a0018eb0)下載此指南。


## 如需 Blob 的詳細資訊，
[探索使用 Azure PowerShell 的 Azure 磁碟加密](http://blogs.msdn.com/b/azuresecurity/archive/2015/11/16/explore-azure-disk-encryption-with-azure-powershell.aspx?wa=wsignin1.0)

[探索使用 Azure PowerShell 的 Azure 磁碟加密 - 第 2 部分](http://blogs.msdn.com/b/azuresecurity/archive/2015/11/21/explore-azure-disk-encryption-with-azure-powershell-part-2.aspx)

<!---HONumber=AcomDC_0921_2016-->