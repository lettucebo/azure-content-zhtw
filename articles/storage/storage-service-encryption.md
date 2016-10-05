<properties
	pageTitle="待用資料的 Azure 儲存體服務加密 | Microsoft Azure"
	description="使用 Azure 儲存體服務加密功能，在儲存資料時於服務端加密您的 Azure Blob 儲存體，在擷取資料時將其解密。"
	services="storage"
	documentationCenter=".net"
	authors="robinsh"
	manager="carmonm"
	editor="tysonn"/>

<tags
	ms.service="storage"
	ms.workload="storage"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/08/2016"
	ms.author="robinsh"/>

# 待用資料的 Azure 儲存體服務加密

待用資料的 Azure 儲存體服務加密 (SSE) 會協助您保護資料安全，以符合組織安全性和法規遵循承諾。利用此功能，Azure 儲存體會自動加密資料，再保存到儲存體，以及在擷取之前解密。以完全無感的方式處理所有加密、解密和金鑰管理。

下列章節提供有關如何使用儲存體服務加密功能的詳細指引，以及支援的案例和使用者體驗。

## 概觀


Azure 儲存體提供一組完整的安全性功能，讓開發人員能夠共同建置安全應用程式。您可以使用[用戶端加密](storage-client-side-encryption.md)、HTTP 或 SMB 3.0，在應用程式和 Azure 之間進行傳輸時保護資料的安全。儲存體服務加密可提供待用加密，並以完全透明的方式處理加密、解密和金鑰管理。所有資料都使用 256 位元 [AES 加密](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) (可用的最強區塊加密方式之一) 進行加密。

SSE 運作方式是在資料寫入 Azure 儲存體時進行加密，並可使用於區塊 Blob、分頁 Blob 和附加 Blob。它也適用於下列各項︰

-   一般用途儲存體帳戶和 Blob 儲存體帳戶
-   「標準」儲存體和「進階」儲存體
-   所有備援層級 (LRS、ZRS、GRS、RA-GRS)
-   Azure Resource Manager 儲存體帳戶 (但不是傳統帳戶)
-   所有區域

這個螢幕擷取畫面顯示如何使用 [Azure 入口網站](https://azure.portal.com)來尋找「儲存體服務加密」設定。在此畫面上，按一下 [加密] 以繼續。

![入口網站螢幕擷取畫面顯示 [加密] 選項](./media/storage-service-encryption/image1.png)

按一下 [加密] 設定之後，您可以啟用或停用「儲存體服務加密」。

![入口網站螢幕擷取畫面顯示 [加密] 屬性](./media/storage-service-encryption/image2.png)

##加密案例

可以在儲存體帳戶層級啟用儲存體服務加密。它支援下列客戶案例：

-   區塊 Blob、附加 Blob 和分頁 Blob 的加密。

-   從內部部署移至 Azure 的封存 VHD 和範本的加密。

-   使用您的 VHD 建立的 IaaS VM 基礎 OS 和資料磁碟的加密。

SSE 有下列限制：

-   不支援傳統儲存體帳戶的加密。

-   不支援移轉至 Resource Manager 儲存體帳戶的傳統儲存體帳戶的加密。

-   現有資料 - SSE 只會加密啟用加密之後新建立的資料。例如，如果您建立新的 Resource Manager 儲存體帳戶但是未開啟加密，然後您將 blob 或封存 VHD 上傳至該儲存體帳戶，然後開啟 SSE，那些 blob 將不會加密，除非它們重新寫入或複製。

-   Marketplace 支援 - 允許使用 [Azure 入口網站](https://portal.azure.com)、PowerShell 及 Azure CLI 來加密從 Marketplace 建立的 VM。VHD 基底影像將保持未加密狀態；不過，在 VM 啟動之後完成的任何寫入將會加密。

-   資料表、佇列和檔案資料將不會加密。

##開始使用

###步驟 1：[建立新的儲存體帳戶](storage-create-storage-account.md)。

###步驟 2︰啟用加密。

您可以使用 [Azure 入口網站](https://portal.azure.com)來啟用加密。

> [AZURE.NOTE] 如果您想要以程式設計方式啟用或停用儲存體帳戶上的儲存體服務加密，您可以使用 [Azure 儲存體資源提供者 REST API](https://msdn.microsoft.com/library/azure/mt163683.aspx)、[適用於 .NET 的儲存體資源提供者用戶端程式庫](https://msdn.microsoft.com/library/azure/mt131037.aspx)、[Azure PowerShell](../powershell-install-configure.md) 或 [Azure CLI](storage-azure-cli.md)。

###步驟 3︰將資料複製到儲存體帳戶

如果您在儲存體帳戶上啟用 SSE，然後將 Blob 寫入儲存體帳戶，Blob 將會加密。已經位於該儲存體帳戶的任何 Blob 直到改寫後，才會加密。您可以將資料從一個儲存體帳戶複製到已啟用 SSE 的儲存體帳戶，或甚至啟用 SSE 並在容器之間複製 Blob，以確保先前的資料已加密。您也可以使用下列任何工具來完成此作業。

#### 使用 AzCopy

AzCopy 是個 Windows 命令列公用程式，專為使用簡單命令高效率地將資料複製到和複製出 Microsoft Azure Blob、檔案和表格儲存體所設計。您可以使用此公用程式將 Blob 從某個儲存體帳戶複製到另一個已啟用 SSE 的儲存體帳戶。

若要深入了解，請參閱[使用 AzCopy 命令列公用程式傳輸資料](storage-use-azcopy.md)。

#### 使用儲存體用戶端程式庫

您可以使用我們豐富的儲存體用戶端程式庫集合 (包括 .NET、C++、Java、Android、Node.js、PHP、Python 和 Ruby)，從 Blob 儲存體來回複製 Blob 資料或在儲存體帳戶之間進行複製。

若要深入了解，請參閱[以 .NET 開始使用 Azure Blob 儲存體](storage-dotnet-how-to-use-blobs.md)。

#### 使用儲存體總管

您可以使用儲存體總管來建立儲存體帳戶、上傳和下載資料、檢視 blob 的內容，以及瀏覽目錄。您可以使用其中一種將 blob 上傳至儲存體帳戶，並且啟用加密。使用某些儲存體總管，您也可以將資料從現有的 Blob 儲存體複製到儲存體帳戶中的不同容器或已啟用 SSE 的新儲存體帳戶。

若要深入了解，請參閱 [Azure 儲存體總管](storage-explorers.md)。

###步驟 4︰查詢加密資料的狀態

已部署儲存體用戶端程式庫的更新版本，可讓您查詢物件的狀態，以判斷它是否加密。在不久的將來會在這份文件中加入範例。

在此同時，您可以呼叫[取得帳戶屬性](https://msdn.microsoft.com/library/azure/mt163553.aspx)來確認儲存體帳戶是否已啟用加密，或在 Azure 入口網站中檢視儲存體帳戶屬性。

##加密和解密工作流程

以下是加密/解密工作流程的簡短描述：

-   客戶會在儲存體帳戶上啟用加密。

-   當客戶將新資料 (放置 Blob、放置區塊、放置頁面等等) 寫入至 Blob 儲存體，每個寫入是使用 256 位元 [AES 加密](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) (可用的最強區塊加密方式之一) 進行加密。

-   當客戶需要存取資料 (取得 Blob 等) 時，資料會在傳回給使用者之前自動解密。

-   如果已停用加密，就不會再加密新的寫入，現有加密資料在使用者重新寫入之前會保持加密。啟用加密時，寫入至 Blob 儲存體將會加密。資料的狀態在使用者於啟用/停用儲存體帳戶的加密之間切換時不會變更。

-   所有加密金鑰會由 Microsoft 儲存、加密及管理。

##待用資料的儲存體服務加密的常見問題集

**問：我有現有的傳統儲存體帳戶。可以在其上啟用 SSE 嗎？**

答︰否，只有 Resource Manager 儲存體帳戶支援 SSE。

**問：我要如何在我的傳統儲存體帳戶中加密資料？**

答︰您可以建立新的 Resource Manager 儲存體帳戶，並且使用 [AzCopy](storage-use-azcopy.md)，從現有的傳統儲存體帳戶將資料複製到新建立的 Resource Manager 儲存體帳戶。

**問：我有現有的 Resource Manager 儲存體帳戶。可以在其上啟用 SSE 嗎？**

答︰是，但只會加密新寫入的 Blob。並不會返回及加密已經存在的資料。

**問︰我想要加密現有 Resource Manager 儲存體帳戶中目前的資料？**

答︰您可以在 Resource Manager 儲存體帳戶中隨時啟用 SSE。不過，不會加密已經存在的 Blob。若要加密這些 Blob，您可以將它們複製到另一個名稱或另一個容器，然後移除未加密的版本。

**問︰我使用進階儲存體，可以使用 SSE 嗎？**

答︰是，SSE 支援標準儲存體和進階儲存體。

**問︰如果我建立新的儲存體帳戶，並啟用 SSE，然後使用該儲存體帳戶建立新的 VM，是否表示我的 VM 會加密？**

答： 會。使用新的儲存體帳戶建立的任何磁碟將會加密，只要它們是在啟用 SSE 之後建立。如果 VM 是使用 Azure Market Place 建立，VHD 基底影像將保持未加密狀態；不過，在 VM 啟動之後完成的任何寫入將會加密。

**問︰是否可以使用 Azure PowerShell 和 Azure CLI 建立新的儲存體帳戶並且啟用 SSE？**

答： 會。

**問︰如果已啟用 SSE，Azure 儲存體的成本會多出多少？**

答：沒有任何額外成本。

**問︰誰負責管理加密金鑰？**

答︰金鑰是由 Microsoft 管理。

**問︰我可以使用自己的加密金鑰嗎？**

答︰我們正在努力提供功能讓客戶使用他們自己的加密金鑰。

**問︰是否可以撤銷加密金鑰的存取權？**

答︰目前不行。金鑰完全是由 Microsoft 管理。

**問︰我建立新的儲存體帳戶時是否預設啟用 SSE？**

答：預設未啟用 SSE；您可以使用 Azure 入口網站來啟用它。您也可以以程式設計方式，使用儲存體資源提供者 REST API 來啟用這項功能。

**問︰這項功能與 Azure 磁碟機加密有什麼不同？**

答︰這項功能是用來加密 Azure Blob 儲存體中的資料。Azure 磁碟加密可用來加密作業系統和 IaaS VM 中的資料磁碟。如需詳細資訊，請參閱我們的[儲存體安全性指南](storage-security-guide.md)。

**問︰如果我啟用 SSE，然後在磁碟上進入並啟用 Azure 磁碟加密？**

答︰這會順暢地運作。這兩種方法都會加密您的資料。

**問︰我的儲存體帳戶設定為異地備援複寫。如果啟用 SSE，我的備援複本是否也會加密？**

答︰是，會加密儲存體帳戶的所有複本，並且支援所有備援選項 – 本地備援儲存體 (LRS)、區域備援儲存體 (ZRS)、異地備援儲存體 (GRS) 和讀取權限異地備援儲存體 (RA-GRS)。

**問︰我無法在我的儲存體帳戶上啟用加密。**

答︰這是 Resource Manager 儲存體帳戶嗎？ 不支援傳統儲存體帳戶。

**問︰是否只有在特定區域中允許 SSE？**

答︰所有區域均可使用 SSE。

**問︰如果我有任何問題或想要提供意見反應，要如何連絡相關人員？**

答︰如果有儲存體服務加密的相關問題，請連絡 [ssediscussions@microsoft.com](mailto:ssediscussions@microsoft.com)。

##後續步驟

Azure 儲存體提供一組完整的安全性功能，讓開發人員能夠共同建置安全應用程式。如需詳細資訊，請參閱[儲存體安全性指南](storage-security-guide.md)。

<!---HONumber=AcomDC_0921_2016-->