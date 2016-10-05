<properties 
   pageTitle="Azure 自動化 DSC 概觀 | Microsoft Azure" 
   description="Azure 自動化期望狀態組態 (DSC)、其條款和已知問題的概觀" 
   services="automation" 
   documentationCenter="dev-center-name" 
   authors="coreyp-at-msft" 
   manager="stevenka" 
   editor="tysonn"
   keywords="powershell dsc, 需要的狀態組態, powershell dsc azure"/>

<tags
   ms.service="automation"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="powershell"
   ms.workload="TBD" 
   ms.date="05/10/2016"
   ms.author="magoedte;coreyp"/>

# Azure 自動化 DSC 概觀 #

##什麼是 Azure 自動化 DSC？##
部署和維護伺服器與應用程式資源的期望狀態可能會非常繁瑣又容易出錯。使用 Azure 自動化期望狀態組態 (DSC)，您可以一致地部署、可靠地監視和自動更新雲端的所有 IT 資源的期望狀態。建置在 PowerShell DSC 上，自動化 DSC 可以對齊機器組態與跨實體和虛擬機器 (VM) 的特定狀態 (使用 Windows 或 Linux，以及在雲端或內部部署)。您可以啟用連續 IT 服務傳遞，具有一致的控制項，並且輕鬆地管理跨異質混合式 IT 環境的快速變更。

Azure 自動化 DSC 建置在 PowerShell DSC 中推出的基礎上，提供您更輕鬆的組態管理經驗。Azure Automation DSC 為 [PowerShell 期望狀態組態](https://msdn.microsoft.com/powershell/dsc/overview)帶來與現今的 Azure 自動化提供的 PowerShell 指令碼相同的管理層。

Azure 自動化 DSC 可讓您 [編寫和管理 PowerShell 期望狀態組態](https://technet.microsoft.com/library/dn249918.aspx)、匯入 [DSC 資源](https://technet.microsoft.com/library/dn282125.aspx)，並產生 DSC 節點組態 (MOF 文件)，全都在雲端。這些 DSC 項目將會放在 Azure 自動化 [DSC 提取伺服器](https://technet.microsoft.com/library/dn249913.aspx)上，以便在雲端或內部的目標節點 (例如實體和虛擬機器) 可以加以提取，自動符合它們指定的狀態，並向 Azure 自動化回報其期望狀態的符合性。

寧可觀賞也不要閱讀？ 請觀賞以下在 2015 年 5 月發佈的影片，此為首次發佈 Azure 自動化 DSC 的時間。**附註：**儘管這段影片中所討論的概念和週期都是正確的，但是 Azure 自動化 DSC 自從這段影片錄製之後已經進步許多。它現在已正式發行，在 Azure 入口網站中具有更豐富的 UI，並支援額外的功能。

> [AZURE.VIDEO microsoft-ignite-2015-heterogeneous-configuration-management-using-microsoft-azure-automation]



## Azure 自動化 DSC 詞彙 ##
### 組態 ###
PowerShell DSC 引進了稱為組態的新概念。組態可讓您透過 PowerShell 語法定義您的環境所需的狀態。若要使用 DSC 來設定您的環境，請先使用組態關鍵字定義 Windows PowerShell 指令碼區塊，接著使用識別項，然後以大括弧 ({}) 來分隔區塊。

![替代文字](./media/automation-dsc-overview/AADSC_1.png)

在組態區塊內，您可以定義節點組態區塊，指定一組環境中應該完全相同的節點 (電腦) 的期望組態。如此一來，節點組態可表示要設想的一或多個節點的「角色」。節點組態區塊開頭為 node 關鍵字。在此關鍵字之後加上角色名稱，它可以是變數或運算式。在角色名稱之後，使用大括弧 {} 來分隔節點組態區塊。

![替代文字](./media/automation-dsc-overview/AADSC_2.png)
 
在節點組態區塊內，您可以定義資源區塊來設定特定 DSC 資源。資源區塊的開頭是資源的名稱，後面接著要為該區塊指定的識別碼，然後是大括弧 {} 用來分隔區塊。

![替代文字](./media/automation-dsc-overview/AADSC_3.png)

如需組態關鍵字的詳細資訊，請參閱：[了解期望狀態組態中的組態關鍵字](http://blogs.msdn.com/b/powershell/archive/2013/11/05/understanding-configuration-keyword-in-desired-state-configuration.aspx "了解期望狀態組態中的組態關鍵字")

執行 (編譯) DSC 組態將會產生一或多個 DSC 節點組態 (MOF 文件)，這是 DSC 節點會套用以符合所需狀態的組態。

Azure 自動化 DSC 可讓您在 Azure 自動化中匯入、編寫並編譯 DSC 組態，類似於匯入、編寫並在 Azure 自動化中開始使用 Runbook 的方式。

>[AZURE.IMPORTANT] 在 Azure 自動化 DSC 中，組態應該僅包含一個組態區塊，具有與組態相同的名稱。


###節點組態###

編譯了 DSC 組態之後，根據組態中的節點區塊，會產生一或多個節點組態。節點組態與「MOF」或「組態文件」是一樣的 (如果您熟悉這些 PS DSC 詞彙)，而代表「角色」，例如 webserver 或worker，應該設想一或多個節點的期望狀態，或檢查其相容性。Azure 自動化 DSC 中的節點組態名稱會採用 Configuration Name.NodeConfigurationBlockName 的形式。

可察覺節點組態的 PS DSC 節點，應該透過 DSC 發送或提取方法來執行。Azure Automation DSC 依賴 DSC 提取方法，其中要求節點組態的節點應該從 Azure 自動化 DSC 提取伺服器套用。由於節點會對 Azure 自動化 DSC 提出要求，因此這些節點可以在防火牆之後、將所有輸入連接埠關閉等等。它們只需要對網際網路的輸出存取權 (直接或透過 proxy)。


###節點###

DSC 節點是由 DSC 所管理的任何一部機器的組態。包括 Windows 或 Linux Azure VM、內部部署 VM、實體主機或其他公用雲端中的機器。節點執行節點組態以成為並保持符合它們定義的期望狀態，而且也可以回報給報告伺服器其相關的組態狀態和符合性與期望狀態。

Azure Automation DSC 使將節點上架以由 Azure Automation DSC 進行管理更為輕鬆，並允許變更指派給每個節點伺服器端的節點組態，因此下一次節點檢查伺服器的指示時，會設想不同的角色並變更它的設定方式和它應該報告的相容性狀態來符合組態。


###資源###
DSC 資源是可用來定義 Windows PowerShell 期望狀態組態 (DSC) 組態的建置組塊。DSC 隨附一組內建功能資源，例如檔案和資料夾、伺服器功能和角色、登錄設定、環境變數，以及服務和程序。若要了解內建 DSC 資源的完整清單，以及如何使用它們，請參閱 [內建的 Windows PowerShell 期望狀態組態資源](https://technet.microsoft.com/library/dn249921.aspx)。

DSC 資源也可匯入成為 PowerShell 模組的一部分，以擴充內建的 DSC 資源集。如果應執行節點的節點組態包含這些資源的參考，非預設的資源會由 DSC 節點從 DSC 提取伺服器撤下。若要了解如何建立自訂資源，請參閱[建置自訂 Windows PowerShell 期望狀態組態資源](https://technet.microsoft.com/library/dn249927.aspx)。

Azure 自動化 DSC 隨附與 PS DSC 相同的所有內建 DSC 資源。透過匯入包含資源的 PowerShell 模組到 Azure 自動化，即可加入其他資源至 Azure 自動化 DSC。


###編譯工作###
Azure 自動化 DSC 中的編譯工作是組態編譯的執行個體，以建立一或多個節點組態。它們類似於 Azure 自動化 Runbook 工作，除了它們未實際執行除了建立節點組態以外的任何工作。編譯工作所建立的任何節點組態都會自動放在 Azure 自動化 DSC 提取伺服器上，並覆寫舊版節點組態 (如果此組態具有舊版節點組態)。編譯工作產生之節點組態的名稱會採用 ConfigurationName.NodeConfigurationBlockName 的形式。例如，編譯下列組態會產生稱為 "MyConfiguration.webserver" 的單一節點組態


![替代文字](./media/automation-dsc-overview/AADSC_5.png)


>[AZURE.NOTE] 就像 Runbook，一樣可以發行組態。這與將 DSC 項目放入 Azure 自動化 DSC 提取伺服器無關。編譯工作造成 DSC 項目放在 Azure 自動化 DSC 提取伺服器上。如需有關 Azure 自動化中的「發行」的詳細資訊，請參閱[發行 Runbook](https://msdn.microsoft.com/library/dn903765.aspx)。


##Azure 自動化 DSC 週期##
從空的自動化帳戶到一組受管理的正確設定節點，會包含一組用來定義組態、將這些組態變為節點組態，並將節點上架至 Azure 自動化 DSC 及節點組態的程序。下圖說明 Azure 自動化 DSC 週期：

![替代文字](./media/automation-dsc-overview/DSCLifecycle.png)


下圖說明 DSC 的生命週期的詳細逐步程序。它包含組態匯入和套用至 Azure 自動化中的節點的不同方式、內部部署電腦所需的元件以支援 DSC 和不同元件之間互動。

![DSC 架構](./media/automation-dsc-overview/dsc-architecture.png)

##錯誤/已知問題：##

- 升級至 WMF 5 RTM 時，如果機器已註冊為 Azure Automation DSC 的節點，請從 Azure Automation DSC 將其取消註冊，並且在 WMF 5 RTM 升級之後重新註冊。

- Azure 自動化 DSC 目前不支援進行部分或複合 DSC 組態。不過，DSC 複合資源可以在 Azure 自動化 DSC 組態中匯入及使用，就像在本機 PowerShell 中，讓組態重複使用。

- 必須安裝 WMF 5 的最新版本，Windows 適用的 PowerShell DSC 代理程式才能與 Azure 自動化通訊。必須安裝 Linux 適用之 PowerShell DSC 代理程式的最新版本，Linux 才能與 Azure 自動化通訊。

- 傳統的 PowerShell DSC 提取伺服器預期模組 Zip 會以格式 **ModuleName\_Version.zip** 放置在提取伺服器上。Azure 自動化預期 PowerShell 模組會以 **ModuleName.zip** 名稱格式匯入。請參閱[此部落格文章](https://azure.microsoft.com/blog/2014/12/15/authoring-integration-modules-for-azure-automation/)，以取得將模組匯入至 Azure 自動化所需的整合模組格式的詳細資訊。

- 匯入到 Azure 自動化的 PowerShell 模組不能包含 .doc 或 .docx 檔案。包含 DSC 資源的某些 PowerShell 模組包含這些檔案，供說明之用。匯入到 Azure 自動化之前，應該先從模組移除這些檔案。

- 若已先向 Azure 自動化帳戶註冊節點，或已將該節點變更為對應至不同的節點組態伺服器端，則它的狀態將會是「相容」，即使節點的狀態實際上與現在對應的節點組態不相容亦然。當節點執行它的首次提取並傳送它的第一份報告之後，在註冊或節點組態對應變更之後，均可信任節點狀態。

- 上架 Azure Windows VM 以讓 Azure Automation DSC 使用任何直接上架方法進行管理時，可能需要一個小時讓 VM 顯示為 Azure 自動化中的 DSC 節點。這是因為 VM 上憑藉著 Azure VM DSC 延伸模組的 Windows Management Framework 5.0 安裝，需要它才能將 VM 上架到 Azure 自動化 DSC。

- 在註冊之後，每個節點會自動交涉唯一的驗證憑證，該憑證於一年之後到期。此時，當憑證即將過期時，PowerShell DSC 註冊通訊協定便無法自動更新憑證，因此您必須在一年之後重新註冊這些節點。在重新登錄之前，請確定每個節點都正在執行 Windows Management Framework 5.0 RTM。如果節點的驗證憑證過期，而且該節點尚未註冊，則該節點將無法與 Azure 自動化通訊，並將標示為「未回應」。 註冊執行方式與您一開始註冊節點時相同。與憑證到期時間相距 90 天或更短時間內執行的註冊，或是憑證到期時間之後任何時間點執行的註冊，將會產生新的憑證並予以使用。

- 升級至 WMF 5 RTM 時，如果機器已註冊為 Azure Automation DSC 的節點，請從 Azure Automation DSC 將其取消註冊，並且在 WMF 5 RTM 升級之後重新註冊。在重新註冊之前，刪除 $env:windir\\system32\\configuration\\DSCEngineCache.mof 檔案。

- 如果將 WMF 5 RTM 安裝於 WMF 5 Production Preview 上，PowerShell DSC Cmdlet 可能不會運作。若要修正此問題，請在提高權限的 PowerShell 工作階段 (以系統管理員身分執行) 中執行下列命令︰`mofcomp $env:windir\system32\wbem\DscCoreConfProv.mof`
 

##相關文章##

- [上架由 Azure 自動化 DSC 管理的機器](../automation/automation-dsc-onboarding.md)
- [編譯 Azure 自動化 DSC 中的組態](../automation/automation-dsc-compile.md)
- [Azure 自動化 DSC Cmdlet](https://msdn.microsoft.com/library/mt244122.aspx)
- [Azure 自動化 DSC 價格](https://azure.microsoft.com/pricing/details/automation/)
- [使用 Azure 自動化 DSC 和 Chocolatey 的 IaaS VM 持續部署](automation-dsc-cd-chocolatey.md)

<!---HONumber=AcomDC_0921_2016-->