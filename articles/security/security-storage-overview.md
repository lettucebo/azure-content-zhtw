<properties
   pageTitle="Azure 儲存體安全性概觀 | Microsoft Azure"
   description=" Azure 儲存體是現代應用程式的雲端儲存體解決方案，這些應用程式仰賴持續性、可用性和可調整性來滿足其客戶的需求。本文提供可用於 Azure 儲存體的核心 Azure 安全性功能概觀。"
   services="security"
   documentationCenter="na"
   authors="TerryLanfear"
   manager="MBaldwin"
   editor="TomSh"/>

<tags
   ms.service="security"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="09/16/2016"
   ms.author="terrylan"/>

# Azure 儲存體安全性概觀

Azure 儲存體是現代應用程式的雲端儲存體解決方案，這些應用程式仰賴持續性、可用性和可調整性來滿足其客戶的需求。Azure 儲存體提供一組完整的安全性功能：

- 您可以使用角色型存取控制與 Azure Active Directory 來保護儲存體帳戶。
- 您可以使用用戶端加密、HTTP 或 SMB 3.0，在應用程式和 Azure 之間進行傳輸時保護資料的安全。
- 使用儲存體服務加密寫入 Azure 儲存體時，可將資料設定為自動加密。
- 您可以使用 Azure 磁碟加密，將虛擬機器所使用的作業系統和資料磁碟設定為加密。
- Azure 儲存體中資料物件的委派存取權可以使用「共用存取簽章」來授與。
- 可以使用儲存體分析來追蹤某人存取儲存體時所使用的驗證方法。

若要深入了解「Azure 儲存體」中的安全性，請參閱 [Azure 儲存體安全性指南](../storage/storage-security-guide.md)。本指南深入探討 Azure 儲存體的安全性功能，例如儲存體帳戶金鑰、傳輸中資料和待用資料的加密，以及儲存體分析。

本文提供可與「Azure 儲存體」搭配使用的 Azure 安全性功能概觀。針對提供每項功能詳細資料的文章都提供了連結，以讓您深入了解。

以下是本文所涵蓋的核心功能：

- 角色型存取控制
- 儲存體物件的委派存取權
- 傳輸中加密
- 待用加密/儲存體服務加密
- Azure 磁碟加密
- Azure 金鑰保存庫

## 角色型存取控制 (RBAC)

您可以使用角色型存取控制 (RBAC) 來保護儲存體帳戶。對於想要強制執行資料存取安全性原則的組織，根據[需要知道](https://en.wikipedia.org/wiki/Need_to_know)和[最低權限](https://en.wikipedia.org/wiki/Principle_of_least_privilege)安全性原則限制存取權限是必須做的事。在特定範圍將適當的 RBAC 角色指派給群組和應用程式，即可授與這些存取權限。您可以使用[內建的 RBAC 角色](../active-directory/role-based-access-built-in-roles.md) (例如儲存體帳戶參與者) 將權限指派給使用者。

深入了解：

- [Azure Active Directory 角色型存取控制](../active-directory/role-based-access-control-configure.md)

## 儲存體物件的委派存取權

共用存取簽章 (SAS) 可提供您儲存體帳戶中資源的委派存取。SAS 意謂著您可以將儲存體帳戶中物件的有限權限授與用戶端，讓該用戶端可以在一段指定的時間內使用一組指定的權限來進行存取。您可以在不須分享您帳戶存取金鑰的情況下，授與這些有限的權限。SAS 是一種 URI，此 URI 會在其查詢參數中包含對儲存體資源進行驗證式存取所需的一切資訊。若要使用 SAS 存取儲存體資源，用戶端只需在適當的建構函式或方法中傳入 SAS 即可。

深入了解：

- [了解 SAS 模型](../storage/storage-dotnet-shared-access-signature-part-1.md)
- [透過 Blob 儲存體來建立與使用 SAS](../storage/storage-dotnet-shared-access-signature-part-2.md)

## 傳輸中加密
傳輸中加密是透過網路傳輸資料時用來保護資料的機制。使用 Azure 儲存體，您可以使用下列各向來保護資料︰

- [傳輸層級加密](../storage/storage-security-guide.md#encryption-in-transit)，例如從 Azure 儲存體傳入或傳出資料時的 HTTPS。
- [連線加密](../storage/storage-security-guide.md#using-encryption-during-transit-with-azure-file-shares)，例如 Azure 檔案共用的 SMB 3.0 加密。
- [用戶端加密](../storage/storage-security-guide.md#using-client-side-encryption-to-secure-data-that-you-send-to-storage)，以在將資料傳輸至儲存體之前加密資料，以及自儲存體傳出後解密資料。

深入了解用戶端加密︰

- [Microsoft Azure 儲存體的用戶端加密](https://blogs.msdn.microsoft.com/windowsazurestorage/2015/04/28/client-side-encryption-for-microsoft-azure-storage-preview/)
- [雲端安全性控制項系列︰加密傳輸中的資料](http://blogs.microsoft.com/cybertrust/2015/08/10/cloud-security-controls-series-encrypting-data-in-transit/)

## 待用加密

對許多組織來說，[待用資料加密](https://blogs.microsoft.com/cybertrust/2015/09/10/cloud-security-controls-series-encrypting-data-at-rest/)是達到資料隱私性、法規遵循及資料主權的必要步驟。有三個 Azure 功能可提供「待用」資料的加密。

- [儲存體服務加密](../storage/storage-security-guide.md#encryption-at-rest)可讓您要求儲存體服務在將資料寫入 Azure 儲存體時自動加密資料。
- [用戶端加密](../storage/storage-security-guide.md#client-side-encryption)也會提供待用加密的功能。
- [Azure 磁碟加密](../storage/storage-security-guide.md#using-azure-disk-encryption-to-encrypt-disks-used-by-your-virtual-machines)允許您加密 IaaS 虛擬機器所使用的作業系統磁碟和資料磁碟。

深入了解儲存體服務加密：

- [Azure 儲存體服務加密](https://azure.microsoft.com/services/storage/)適用於 [Azure Blob 儲存體](https://azure.microsoft.com/services/storage/blobs/)。如需其他 Azure 儲存體類型的詳細資訊，請參閱[檔案](https://azure.microsoft.com/services/storage/files/)、[磁碟 (進階儲存體)](https://azure.microsoft.com/services/storage/premium-storage/)、[資料表](https://azure.microsoft.com/services/storage/tables/)和[佇列](https://azure.microsoft.com/services/storage/queues/)。
- [待用資料的 Azure 儲存體服務加密](../storage/storage-service-encryption.md)

## Azure 磁碟加密

適用於虛擬機器 (VM) 的 Azure 磁碟加密會使用您在 [Azure 金鑰保存庫](https://azure.microsoft.com/services/key-vault/)所控制的金鑰與原則將 VM 磁碟 (包括開機和資料磁碟) 加密，協助您達成組織安全性與相容性需求。

適用於 VM 的磁碟加密可用於 Linux 與 Windows 作業系統。也會使用金鑰保存庫，協助您防護、管理和稽核您的磁碟加密金鑰的使用情形。VM 磁碟中的所有資料皆在待用時，透過 Azure 儲存體帳戶中符合業界標準的加密技術進行加密。Windows 的磁碟加密解決方案是建基於 [Microsoft BitLocker 磁碟機加密](https://technet.microsoft.com/library/cc732774.aspx)，而 Linux 解決方案是建基於 [dm-crypt](https://en.wikipedia.org/wiki/Dm-crypt)。

深入了解：

- [適用於 Windows 和 Linux IaaS 虛擬機器的 Azure 磁碟加密](https://gallery.technet.microsoft.com/Azure-Disk-Encryption-for-a0018eb0)

## Azure 金鑰保存庫

「Azure 磁碟加密」會使用 [Azure 金鑰保存庫](https://azure.microsoft.com/services/key-vault/)，既幫助您控制和管理您的金鑰保存庫訂用帳戶中的磁碟加密金鑰和密碼，同時也確保虛擬機器磁碟中的所有資料在您「Azure 儲存體」中待用時會受到加密。您應使用金鑰保存庫來稽核金鑰和原則使用方式。

深入了解：

- [什麼是 Azure 金鑰保存庫？](../key-vault/key-vault-whatis.md)
- [開始使用 Azure 金鑰保存庫](../key-vault/key-vault-get-started.md)

<!---HONumber=AcomDC_0921_2016-->