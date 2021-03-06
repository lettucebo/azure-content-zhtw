<properties
	pageTitle="Microsoft Azure 虛擬機器、雲端服務和 Web Apps 的自動調整概觀 | Microsoft Azure"
	description="Microsoft Azure 的自動調整概觀。適用於虛擬機器、雲端服務和 Web Apps。"
	authors="rboucher"
	manager=""
	editor=""
	services="monitoring-and-diagnostics"
	documentationCenter="monitoring-and-diagnostics"/>

<tags
	ms.service="monitoring-and-diagnostics"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/06/2016"
	ms.author="robb"/>

# Microsoft Azure 虛擬機器、雲端服務和 Web Apps 的自動調整概觀

本文說明何謂 Microsoft Azure 自動調整、其優點，以及如何開始使用它。

Azure Insights 自動調整僅適用於[虛擬機器擴展集](https://azure.microsoft.com/services/virtual-machine-scale-sets/)、[雲端服務](https://azure.microsoft.com/services/cloud-services/)和 [App Service - Web Apps](https://azure.microsoft.com/services/app-service/web/)。

>[AZURE.NOTE] Azure 有兩種自動調整方法。舊版自動調整適用於虛擬機器 (可用性設定組)。這項功能的支援能力有限，建議您移轉至 VM 擴展集，以獲得更快、更可靠的自動調整支援。本文包含如何使用舊版技術的連結。


## 何謂自動調整？

自動調整可讓您執行適當數量的資源來處理應用程式的負載。它可讓您新增資源來處理增加的負載，並可藉由移除閒置資源來節省成本。您必須指定要執行的執行個體數目下限和上限，並根據一組規則自動新增或移除 VM。設定下限可確保即使沒有負載應用程式也會一直執行。設定上限則可限制每小時可能產生的總成本。您可以使用您所建立的規則自動在這兩個極端值之間進行調整。

 ![說明自動調整。新增和移除 VM](./media/monitoring-autoscale-overview/AutoscaleConcept.png)

符合規則條件時，就會觸發一個或多個自動調整動作。您可以新增和移除 VM，或執行其他動作。以下概念圖會顯示此程序。

 ![概念性的自動調整流程圖](./media/monitoring-autoscale-overview/AutoscaleOverview3.png)
 

## 自動調整程序說明
以下說明適用於先前的圖表項目。

### 資源度量 
資源會發出度量，並於稍後由規則處理。度量來自不同的方法。VM 擴展集使用來自 Azure 診斷代理程式的遙測資料，而 Web 應用程式和雲端服務的遙測則直接來自 Azure 基礎結構。一些常用的統計資料包括 CPU 使用率、記憶體使用量、執行緒計數、佇列長度和磁碟使用量。如需可使用哪些遙測資料的清單，請參閱[自動調整的常用度量](insights-autoscale-common-metrics.md)。

### 時間
排程型規則是以 UTC 為基礎。設定規則時必須正確設定時區。

### 規則
圖中只顯示一個自動調整規則，但您可以擁有多個。您可以視情況需要建立複雜的重疊規則。規則類型包含
 
 - **度量型** - 例如，當 CPU 使用率高於 50% 時執行此動作。
 - **時間型** - 例如，在指定時區的每星期六上午 8 點觸發 Webhook。

度量型規則會測量應用程式負載，並根據該負載新增或移除 VM。排程型規則可讓您在觀察到負載有時間模式，因而想要在可能的負載增加或減少發生之前做出因應時進行調整。

 
### 動作與自動化

規則可以觸發一個或多個類型的動作。

- **調整** - 將 VM 相應放大或相應縮小
- **電子郵件** - 傳送電子郵件給訂用帳戶的管理員、共同管理員和 (或) 您指定的其他電子郵件地址
- **透過 Webhook 自動執行** - 呼叫 Webhook，以觸發 Azure 內部或外部的多個複雜動作。在 Azure 內部，您可以啟動 Azure 自動化 Runbook、Azure 函數或 Azure 邏輯應用程式。Azure 外部的範例第三方 URL 包含 Slack 和 Twilio 等服務。


## 自動調整設定
自動調整使用下列術語和結構。

- **自動調整設定**會被自動調整引擎讀取來判斷是要相應增加或相應減少。它包含一個或多個設定檔、關於目標資源的資訊，以及通知設定。
    - **自動調整設定檔**是容量設定、一組用來管理觸發程序的規則，以及設定檔之調整動作和循環的組合。您可以擁有多個設定檔，以便處理不同的重疊需求。
        - **容量設定**會指出執行個體數目的最小值、最大值和預設值。[使用圖 1 的適當位置]
        - **規則**包含觸發程序 (度量觸發程序或時間觸發程序) 和調整動作 (指出符合該規則時自動調整應該相應增加或減少)。
        - **循環**會指出自動調整應該在何時讓此設定檔生效。舉例來說，您可以針對一天當中的不同時間或一週當中的不同日子擁有不同的自動調整設定檔。
- **通知設定**會定義當自動調整事件發生時，根據滿足其中一個自動調整設定之設定檔的準則，所應該發生的通知。自動調整可以通知一或多個電子郵件地址，或呼叫一或多個 Webhook。
 
![Azure 自動調整設定、設定檔和規則結構](./media/monitoring-autoscale-overview/AzureResourceManagerRuleStructure3.png)

可設定之欄位和說明的完整清單可在[自動調整 REST API](https://msdn.microsoft.com/library/dn931928.aspx) 中取得。

如需程式碼範例，請參閱

* [針對 VM 擴展集使用 Resource Manager 範本進行進階自動調整設定](insights-advanced-autoscale-virtual-machine-scale-sets.md)
* [自動調整 REST API](https://msdn.microsoft.com/library/dn931953.aspx)



## 水平和垂直調整
  
自動調整只會以水平方式增加資源規模，亦即增加 (相應放大) 或減少 (相應縮小) VM 執行個體數目。在雲端的情況下，水平調整會更有彈性，因為它可讓您有機會執行數千台 VM 來處理負載。垂直調整則不同。它會保持相同數量的 VM，但會讓 VM 的能力變強 (相應增加) 或變弱 (相應減少)。能力是以記憶體、CPU 速度、磁碟空間等項目來加以測量。垂直調整有更多限制。它取決於是否可以使用較大的硬體，因此會依區域而異，而且很快就會達到上限。垂直調整通常也需要 VM 停止和啟動。如需詳細資訊，請參閱[使用 Azure 自動化垂直調整 Azure 虛擬機器大小](../virtual-machines/virtual-machines-linux-vertical-scaling-automation.md)。


## 存取方法 
您可以透過下列途徑設定自動調整

- [Azure 入口網站](insights-how-to-scale.md)
- [PowerShell](insights-powershell-samples.md#create-and-manage-autoscale-settings)
- [跨平台命令列介面 (CLI)](insights-cli-samples.md#autoscale)
- [Insights REST API](https://msdn.microsoft.com/library/azure/dn931953.aspx)

## 支援的自動調整服務


| 服務 | 結構描述與文件 |
|--------------------------------------|-----------------------------------------------------|
| Web Apps | [調整 Web Apps](insights-how-to-scale.md) |
| 雲端服務 | [自動調整雲端服務](../cloud-services/cloud-services-how-to-scale.md) |
| 虛擬機器：傳統 | [調整傳統的虛擬機器可用性設定組](https://blogs.msdn.microsoft.com/kaevans/2015/02/20/autoscaling-azurevirtual-machines/) |
| 虛擬機器：Windows 擴展集| [在 Windows 中調整 VM 擴展集](../virtual-machine-scale-sets/virtual-machine-scale-sets-windows-autoscale.md) |
| 虛擬機器：Linux 擴展集 | [在 Linux 中調整 VM 擴展集](../virtual-machine-scale-sets/virtual-machine-scale-sets-linux-autoscale.md) |
| 虛擬機器：Windows 範例 | [針對 VM 擴展集使用 Resource Manager 範本進行進階自動調整設定](insights-advanced-autoscale-virtual-machine-scale-sets.md) |

## 後續步驟

若要深入了解自動調整，請使用先前所列的＜自動調整逐步解說＞或參閱下列資源︰

- [Azure Insights 自動調整的常用度量](insights-autoscale-common-metrics.md)
- [Azure Insights 自動調整的最佳做法](insights-autoscale-best-practices.md)
- [使用自動調整動作傳送電子郵件和 Webhook 警示通知](insights-autoscale-to-webhook-email.md)
- [自動調整 REST API](https://msdn.microsoft.com/library/dn931953.aspx)
- [排解虛擬機器擴展集自動調整的問題](../virtual-machine-scale-sets/virtual-machine-scale-sets-troubleshoot.md)

<!---HONumber=AcomDC_0914_2016-->