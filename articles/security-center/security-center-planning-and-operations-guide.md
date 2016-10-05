<properties
   pageTitle="資訊安全中心規劃和操作指南 | Microsoft Azure"
   description="本文件可協助您在採用 Azure 資訊安全中心和日常作業相關考量之前進行規劃。"
   services="security-center"
   documentationCenter="na"
   authors="YuriDio"
   manager="swadhwa"
   editor=""/>

<tags
   ms.service="security-center"
   ms.topic="hero-article"
   ms.devlang="na"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="09/22/2016"
   ms.author="yurid"/>

# Azure 資訊安全中心規劃和操作指南
本指南適用於計劃採用 Azure 資訊安全中心的組織中的資訊技術 (IT) 專業人員、IT 架構設計人員、資訊安全性分析師和雲端系統管理員。

## 規劃指南
本指南涵蓋您可以遵循的一組步驟和工作，以根據您組織的安全性需求和雲端管理模型，將您的資訊安全中心使用最佳化。若要充分利用資訊安全中心，務必了解您組織中的不同人員或小組如何使用此服務，來滿足安全地開發和作業、監視、控管和事件回應的需求。規劃使用資訊安全中心時所應考量的主要領域如下︰

- 安全性角色和存取控制
- 安全性原則和建議
- 資料收集和儲存
- 持續安全性監視
- 事件回應

在下一節中，您將學習如何根據需求來規劃每個領域及套用這些建議。

> [AZURE.NOTE] 如需在設計和規劃階段也很好用的常見問題清單，請閱讀 [Azure 資訊安全中心常見問題集 (FAQ)](security-center-faq.md)。


## 安全性角色和存取控制
根據您組織的大小和結構，多個個人和小組可以使用資訊安全中心來執行不同的安全性相關工作。下圖是虛構人物與其各自角色和安全性責任的範例︰

![角色](./media/security-center-planning-and-operations-guide/security-center-planning-and-operations-guide-fig01-ga.png)

資訊安全中心可讓這些人符合不同的責任。例如：

**Jeff (雲端工作負載擁有者)**

- 管理雲端工作負載和其相關的資源
- 負責根據公司的安全性原則來實作和維護保護

**Ellen (CISO/CIO)**

- 負責公司安全性的所有層面
- 想要了解各雲端工作負載之間公司的安全性狀態
- 需要知道有關主要攻擊和風險的資訊

**David (IT 安全性)**

- 設定公司安全性原則，以確保適當的保護已就位
- 監控安全性原則的遵循狀態
- 產生報告以供主管或稽核人員使用

**Judy (安全性作業)**

- 隨時監控並回應安全性警示
- 呈報給雲端工作負載擁有者或 IT 安全性分析師

**Sam (安全性分析師)**

- 調查攻擊
- 補救警示的問題，或與雲端工作負載擁有者合作以套用補救方法


資訊安全中心會使用[角色型存取控制 (RBAC)](../active-directory/role-based-access-control-configure.md)，以提供可在 Azure 中指派給使用者、群組與服務的[內建角色](../active-directory/role-based-access-built-in-roles.md)。當使用者開啟資訊安全中心時，他們只能看到與自己能夠存取的資源相關的資訊。這表示會為使用者指派資源所屬訂用帳戶或資源群組的「擁有者」、「參與者」或「讀者」角色。使用在上圖中所述的人物，會需要下列 RBAC：

**Jeff (雲端工作負載擁有者)**

- 資源群組擁有者/共同作業者

**David (IT 安全性)**

- 訂用帳戶擁有者/共同作業者

**Judy (安全性作業)**

- 可檢視警示的訂用帳戶讀者
- 可解除警示的訂用帳戶擁有者/共同作業者

**Sam (安全性分析師)**

- 可檢視警示的訂用帳戶讀者
- 可補救或解除警示的訂用帳戶擁有者/共同作業者
- 可能需要存取儲存體

需要考量的其他重要資訊︰

- 只有訂用帳戶擁有者和參與者可以編輯安全性原則
- 只有訂用帳戶和資源群組擁有者和參與者可以套用資源的安全性建議

使用 RBAC 規劃資訊安全中心的存取控制時，請務必了解您的組織中有誰會使用資訊安全中心，以及他們會執行的工作，然後據此設定 RBAC。

> [AZURE.NOTE] 我們建議您指派所需的最寬鬆角色，以便使用者完成其工作。例如，只需要檢視資源安全性狀態的相關資訊，但不採取行動 (例如套用建議或編輯原則) 的使用者應被指派「讀者」角色。

## 安全性原則和建議
安全性原則定義了針對指定之訂用帳戶或資源群組內的資源，所建議的一組控制項。在資訊安全中心，您可以根據公司的安全性需求，以及應用程式的類型或資料的敏感性來定義原則。

在訂用帳戶層級啟用的原則將會自動傳播至訂用帳戶中的所有資源群組，如下圖所示︰

![安全性原則](./media/security-center-planning-and-operations-guide/security-center-planning-and-operations-guide-fig2-ga.png)

如上圖所示，資源群組的安全性原則可以繼承自訂用帳戶層級。

在某些情況下，資源群組中的資源可能需要一組不同的原則，您可以停用繼承並將自訂原則套用到特定的資源群組。

如果您需要特定資源群組中的自訂原則，您應該在此資源群組中停用繼承並變更安全性原則。比方說，如果您的某些工作負載不需要 SQL 透明資料加密原則，請關閉訂用帳戶層級的原則，只在需要 SQL TDE 的資源群組中加以啟用。

當您開始建立不同資源群組的自訂原則時，應該在規劃原則部署的同時，知道在發生原則衝突 (訂用帳戶與資源群組) 時，會以資源群組原則為準。

> [AZURE.NOTE] 如果您需要檢閱已變更的原則，可以使用 [Azure 稽核記錄檔](https://blogs.msdn.microsoft.com/cloud_solution_architect/2015/03/10/audit-logs-for-azure-events/)。原則變更一定都會記錄在 Azure 稽核記錄檔中。

### 安全性建議

在設定安全性原則之前，請檢閱每個[安全性建議](security-center-recommendations.md)，並判斷這些原則是否適合您的各種訂用帳戶和資源群組。此外，務必了解為了處理安全性建議而必須採取的動作。

**端點保護**︰如果虛擬機器並未啟用端點保護解決方案，資訊安全中心會建議您安裝一個。如果您已經有在內部部署採用的慣用端點保護解決方案，則必須決定是否對您的 Azure VM 使用相同的反惡意程式碼。資訊安全中心為您提供數個端點保護選項。您可以使用免費的 Microsoft Antimalware，或從整合式合作夥伴提供的端點保護解決方案中選擇。如需有關如何使用資訊安全中心部署反惡意程式碼的詳細資訊，請閱讀[在 Azure 資訊安全中心安裝端點保護](security-center-install-endpoint-protection.md)。

**系統更新**：資訊安全中心會識別缺少 IaaS 和雲端服務 (PaaS) 安全性或重大作業系統更新的虛擬機器。請考慮誰會在必要時負責套用更新，以及套用的方式。許多組織會使用 WSUS、Windows Update 或其他工具。

**基準設定**︰如果虛擬機器的作業系統設定不符合建議的基準，將會提供一項建議。您應該檢閱[這裡](https://gallery.technet.microsoft.com/Azure-Security-Center-a789e335)的基準集，並考慮如何套用作業系統設定。

**磁碟加密**︰如果您有未加密的虛擬機器磁碟，資訊安全中心會建議您套用 Azure 磁碟加密。此功能會利用 Windows 版的 BitLocker 和 Linux 版的 DM-Crypt 來提供作業系統和資料磁碟的磁碟區加密。這項建議會將您重新導向至[逐步指南](security-center-disk-encryption.md)，其中包含如何執行此加密的指示。

請注意，有幾個您需要處理的加密案例。您必須為每個案例規劃獨特的需求︰

- 您已使用自己的加密金鑰來加密之 VHD 中新 Azure 虛擬機器的加密
- 從 Azure 資源庫建立之新 Azure 虛擬機器的加密
- 已在 Azure 中執行之 Azure 虛擬機器的加密

每個案例的規劃需求都不同。如需每個案例的詳細資訊，請參閱 [Azure 磁碟加密白皮書](https://gallery.technet.microsoft.com/Azure-Disk-Encryption-for-a0018eb0)。

**Web 應用程式防火牆**：資訊安全中心會識別執行 Web 應用程式的虛擬機器，並建議您安裝 Web 應用程式防火牆 (WAF)。評估可用的合作夥伴解決方案，以判斷哪一個最適合用於您的組織，並判斷解決方案的授權方式 (合作夥伴可能支援「自備授權」和/或「隨用隨付」模型)。如需有關如何使用資訊安全中心在 Azure VM 中部署 Web 應用程式防火牆的詳細資訊，請參閱[在 Azure 資訊安全中心新增 Web 應用程式防火牆](security-center-add-web-application-firewall.md)。

**新一代防火牆**︰可讓您佈建領導廠商 (包括 Check Point 和不久之後的 Cisco 和 Fortinet) 所提供的虛擬應用裝置。這會提供超越 Azure 內建網路安全性群組的網路保護。資訊安全中心會探索建議使用新一代防火牆的部署，並可讓您佈建虛擬應用裝置。

**虛擬網路**：資訊安全中心會評估您的 [Azure 虛擬網路](https://azure.microsoft.com/documentation/services/virtual-network/)基礎結構和設定，以檢查[網路安全性](../virtual-network/virtual-networks-nsg.md)群組是否已套用輸入流量規則並正確進行設定。您應該考慮應定義哪些流量規則，並將此訊息傳達給要套用相關安全性建議的人員。

資訊安全中心會建議您針對您的 Azure 訂用帳戶提供安全性連絡人詳細資料。如果 Microsoft 安全性回應中心 (MSRC) 發現您的客戶資料遭到非法或未經授權的對象存取，Microsoft 會使用此資訊連絡您。如需如何啟用這項建議的詳細資訊，請閱讀[在 Azure 資訊安全中心提供安全性連絡人詳細資料](security-center-provide-security-contact-details.md)。

## 資料收集和儲存

強烈建議您為每個訂用帳戶開啟資料收集功能，因為這將確保安全性監視可用於所有的 VM。資料收集是透過 Azure 監視代理程式 (ASMAgentLauncher.exe) 和 Azure 安全性監視擴充功能 (ASMMonitoringAgent.exe) 來啟用。

Azure 安全性監視擴充功能會掃描各種安全性相關組態，並從虛擬機器收集安全性記錄檔。這項資料會傳送到您指定的儲存體帳戶。掃描管理員 (ASMSoftwareScanner.exe) 也會安裝在虛擬機器，並做為修補程式掃描器。

在安全性原則中啟用資料收集之後，監視代理程式和延伸模組會自動安裝在 Azure 中佈建的所有現有和任何新支援的虛擬機器上。代理程式的程序非侵入式，並不會影響 VM 的效能。

> [AZURE.NOTE] 若要針對與 Azure 安全性監視代理程式有關的問題進行疑難排解，請閱讀 [Azure 資訊安全中心疑難排解指南](security-center-troubleshooting-guide.md)。

如果您想在某個時間點停用資料收集，您可以在安全性原則中將它關閉。若要刪除先前部署的監視代理程式，請選取 [刪除代理程式] 功能表選項。

> [AZURE.NOTE] 若要尋找支援的 VM 清單，請閱讀 [Azure 資訊安全中心常見問題集 (FAQ)](security-center-faq.md)。

針對每個有虛擬機器在其中執行的區域，您需選擇儲存體帳戶，以儲存從這些虛擬機器收集到的資料。如不為每個區域選擇儲存體帳戶，系統會為您建立。您可以選擇每個區域的儲存位置，或將所有資訊集中儲存在一個位置。雖然安全性原則可以在 Azure 訂用帳戶層級和資源群組層級進行設定，但您的儲存體帳戶區域只可在訂用帳戶層級選取。

如果您使用在不同 Azure 資源之間共用的儲存體帳戶，請確認您已閱讀 [Azure 儲存體延展性和效能目標](../storage/storage-scalability-targets.md)一文，以取得大小限制和條件約束的詳細資訊。您的訂用帳戶也有儲存體帳戶限制，請檢閱 [Azure 訂用帳戶和服務限制、配額和條件約束](../azure-subscription-service-limits.md)來深入了解這些限制。

> [AZURE.NOTE] 與此儲存體相關的費用不包含在資訊安全中心服務價格中，而將以一般 [Azure 儲存體費率](https://azure.microsoft.com/pricing/details/storage/)分別計費。

您也應該根據 Azure 環境大小和取用您的儲存體帳戶的資源，規劃效能和延展性考量。如需詳細資訊，請參閱 [Microsoft Azure 儲存體效能與延展性檢查清單](../storage/storage-performance-checklist.md)。

## 持續安全性監視

初始設定和套用資訊安全中心建議之後，下一步是考量資訊安全中心操作程序。

若要從 Azure 入口網站存取資訊安全中心，您可以按一下 [瀏覽] 並在 [篩選] 欄位中輸入 [資訊安全中心]。使用者會根據所套用的篩選條件取得檢視。

資訊安全中心不會干擾一般作業程序，它會被動地監視您的部署，並根據您啟用的安全性原則提供建議。

資訊安全中心儀表板分成兩個主要部分︰

- 預防
- 偵測

當您第一次在目前 Azure 環境的資訊安全中心啟用資料收集時，請務必檢閱所有的建議，您可以在 [建議] 刀鋒視窗中檢閱，或依照資源 ([虛擬機器]、[網路]、[SQL] 和 [應用程式]) 檢閱。

一旦處理完所有的建議，所有已處理資源的 [預防] 區段應該是綠色。持續監視此時變得更容易，因為您只會根據資源安全性健康情況和建議圖格中的變更採取動作。

[偵測] 區段的反應會更明顯，這些是正在發生，或過去曾發生，且資訊安全中心控制項和第三方系統所偵測到之問題的相關警示。[安全性警示] 圖格會顯示橫條圖，代表每天找到的威脅偵測警示數目，以及其分布在不同嚴重性分類 (低、中、高) 的情形。如需安全性警示的詳細資訊，請閱讀[管理與回應 Azure 資訊安全中心的安全性警示](security-center-managing-and-responding-alerts.md)。

> [AZURE.NOTE] 您也可以利用 Microsoft Power BI，將資訊安全中心資料視覺化。請閱讀[使用 Power BI 從 Azure 資訊安全中心的資料取得見解](security-center-powerbi.md)。

### 監視新的或已變更的資源

大多數 Azure 環境是動態的，包含定期上下波動的新資源、組態或變更等。資訊安全中心有助於確保您看得到這些新資源的安全性狀態。

當您將新資源 (VM、SQL DB) 新增至 Azure 環境時，資訊安全中心會自動探索這些資源並開始監視其安全性。這也包括 PaaS Web 角色和背景工作角色。如果已在[安全性原則](security-center-policies.md)中啟用資料收集，則會自動為您的虛擬機器啟用其他監視功能。

![主要領域](./media/security-center-planning-and-operations-guide/security-center-planning-and-operations-guide-fig3-ga.png)

1.	針對虛擬機器，存取 [資源安全性健全狀況] 圖格，並按一下 [虛擬機器]。[監視建議] 區段會呈現有關啟用資料收集的問題或相關建議。
2.	檢視 [建議] 以查看針對新資源所找出的任何 (若有的話) 安全性風險。
3.	將新的 VM 新增至您的環境時，通常一開始只會安裝作業系統。資源擁有者可能需要一些時間來部署這些 VM 將使用的其他應用程式。在理想情況下，您應該知道此工作負載的最終目的。它會成為應用程式伺服器？ 以這個新工作負載將會成為的項目為基礎，您可以啟用適當的**安全性原則** (這是此工作流程中的第三個步驟)。
4.	將新資源新增至 Azure 環境時，[安全性警示] 圖格中可能會出現新的警示。請隨時確認此圖格中是否有新的警示，並根據資訊安全中心的建議採取動作。

您也會想要定期監視現有資源的狀態，以找出已造成安全性風險、偏離建議基準和安全性警示的組態變更。從資訊安全中心儀表板開始。在這裡，您可以用一致的方式檢閱三個主要區域。

![Operations](./media/security-center-planning-and-operations-guide/security-center-planning-and-operations-guide-fig4.png)

1.	**資源安全性健全狀況**面板可供您快速存取關鍵資源。使用此選項來監視虛擬機器、網路、SQL 和應用程式。
2.	**建議**面板可讓您檢閱資訊安全中心建議。在持續監視期間，您可能會發現您沒有每天收到建議，這是正常現象，因為您已處理初始資訊安全中心設定的所有建議。基於這個理由，這個區段中可能不會每天都有新資訊，您只需要視需要進行存取。
3.	**偵測**面板的變更頻率可能非常頻繁或非常不頻繁。請隨時檢閱安全性警示，並根據資訊安全中心的建議採取動作。

## 事件回應

資訊安全中心會偵測並在發生威脅時警示您。組織應監視新的安全性警示並視需要採取動作，進一步調查或修補攻擊。如需資訊安全中心威脅偵測運作方式的詳細資訊，請閱讀 [Azure 資訊安全中心的偵測功能](security-center-detection-capabilities.md)。

雖然本文目的並非要幫助您建立自己的事件回應計劃，但我們會在雲端生命週期中使用 Microsoft Azure 安全性回應作為事件回應階段的基礎。這些階段如下圖所示：

![可疑的活動](./media/security-center-planning-and-operations-guide/security-center-planning-and-operations-guide-fig5-1.png)

> [AZURE.NOTE] 您可以使用美國國家標準技術局 (NIST) 的 [Computer Security Incident Handling Guide](http://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf) 做為參考來協助您建置自己的計劃。

在下列階段期間，您可以使用資訊安全中心警示︰

- **偵測**︰識別一或多個資源中的可疑活動。
- **評估**︰執行初始評估，以取得可疑活動的詳細資訊。
- **診斷**︰使用補救步驟來進行解決問題的技術程序。

每個安全性警示都提供可用來進一步了解攻擊本質及建議可能補救措施的資訊。有些警示也會提供更多資訊或 Azure 中其他資訊來源的連結。您可以使用所提供的資訊進一步研究並開始補救。

下列範例說明正在發生的可疑 RDP 活動︰

![可疑的活動](./media/security-center-planning-and-operations-guide/security-center-planning-and-operations-guide-fig5-ga.png)

如您所見，此刀鋒視窗會顯示有關攻擊發生時間、來源主機名稱、目標 VM 的詳細資訊，也會提供建議步驟。在某些情況下，攻擊的來源資訊可能是空的。如需有關這類行為的詳細資訊，請閱讀 [Azure 資訊安全中心警示中的缺少來源資訊](https://blogs.msdn.microsoft.com/azuresecurity/2016/03/25/missing-source-information-in-azure-security-center-alerts/)。

在[如何利用 Azure 資訊安全中心和 Microsoft Operations Management Suite 進行事件回應](https://channel9.msdn.com/Blogs/Taste-of-Premier/ToP1703)影片中，您可以看到一些示範，有助於您了解如何在每個階段使用資訊安全中心。

> [AZURE.NOTE] 如需有關如何使用資訊安全中心功能協助進行事件回應程序的詳細資訊，請參閱[善用 Azure 資訊安全中心進行事件回應](security-center-incident-response.md)。

## 另請參閱
在本文件中，您已了解如何規劃資訊安全中心的採用。如要深入了解資訊安全中心，請參閱下列主題：

- [管理及回應 Azure 資訊安全中心的安全性警示](security-center-managing-and-responding-alerts.md)
- [Azure 資訊安全中心的安全性健全狀況監視](security-center-monitoring.md) — 了解如何監視 Azure 資源的健全狀況。
- [使用 Azure 資訊安全中心監視合作夥伴解決方案](security-center-partner-solutions.md) — 了解如何監視合作夥伴解決方案的健全狀況。
- [Azure 資訊安全中心常見問題集](security-center-faq.md) — 尋找有關使用服務的常見問題。
- [Azure 安全性部落格](http://blogs.msdn.com/b/azuresecurity/) — 尋找有關 Azure 安全性與相容性的部落格文章。

<!---HONumber=AcomDC_0928_2016-->