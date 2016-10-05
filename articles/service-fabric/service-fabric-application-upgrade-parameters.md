
<properties
   pageTitle="應用程式升級：升級參數 | Microsoft Azure"
   description="描述升級 Service Fabric 應用程式的相關參數，包括執行健全狀況檢查和自動復原升級的原則。"
   services="service-fabric"
   documentationCenter=".net"
   authors="mani-ramaswamy"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="09/14/2016"
   ms.author="subramar"/>



# 應用程式升級參數

本文說明在 Azure Service Fabric 應用程式升級期間套用的各種參數。參數包含應用程式的名稱和版本。它們是控制逾時的旋鈕與升級時套用的健康狀態檢查，並且指定升級失敗時必須套用的原則。


<br>

| 參數 | 說明 |
| --- | --- |
| ApplicationName | 正在升級的應用程式名稱。範例：fabric:/VisualObjects、fabric:/ClusterMonitor |
| TargetApplicationTypeVersion | 升級目標的應用程式類型的版本。 |
| FailureAction | 升級失敗時，Service Fabric 所採取的動作。應用程式可能已回復到更新前版本 (回復)，或升級可能已在目前的升級網域停止。如果是後者，升級模式也會變更為手動。允許的值為回復和手動。 |
| HealthCheckWaitDurationSec | 在升級網域上完成升級之後，Service Fabric 評估應用程式健康狀態的等待時間 (以秒為單位)。這段期間也可視為應用程式在被視為健康之前應該執行的時間。如果健康狀態檢查都通過，則升級程序會繼續進行下一個升級網域。如果健康狀態檢查失敗，Service Fabric 會等候一定間隔 (UpgradeHealthCheckInterval)，再重試一次健康狀態檢查，直到達到 HealthCheckRetryTimeout 為止。預設值和建議值為 0 秒。 |
| HealthCheckRetryTimeoutSec | Service Fabric 在將升級宣告為失敗之前，繼續執行健康狀態評估的持續時間 (以秒為單位)。預設值為 600 秒。此期間會在達到 HealthCheckWaitDuration 之後開始計算。在此 HealthCheckRetryTimeout 之內，Service Fabric 可能會執行多個應用程式健康狀態的健康狀態檢查。預設值為 10 分鐘，您應該針對您的應用程式適當地加以自訂。 |
| HealthCheckStableDurationSec | 移至下一個升級網域或完成升級之前，要確認應用程式是否穩定的持續時間 (以秒為單位)。這個等候持續時間是用來在執行健康狀態檢查後，防止未偵測到的健康狀態變更。預設值為 120 秒鐘，您應該針對您的應用程式適當地加以自訂。 |
| UpgradeDomainTimeoutSec | 升級單一升級網域的時間上限 (以秒為單位)。如果達到此逾時，升級會停止，然後根據 UpgradeFailureAction 的設定繼續。預設值為永不 (無限)，您應該針對您的應用程式適當地加以自訂。 |
| UpgradeTimeout | 逾時 (以秒為單位) 適用於整個升級。如果達到此逾時，升級會停止，並且會觸發 UpgradeFailureAction。預設值為永不 (無限)，您應該針對您的應用程式適當地加以自訂。 |
| UpgradeHealthCheckInterval | 檢查健全狀況狀態的頻率。此參數是在「叢集資訊清單」 的 [ClusterManager] 區段中指定，而不是指定為升級 Cmdlet 的一部分。預設值為 60 秒。 |






<br>
## 應用程式升級期間的服務健康狀態評估

<br>健康狀態評估準則是選擇性的。如果啟動升級時未指定健康狀態評估準則，則 Service Fabric 會使用應用程式執行個體的 ApplicationManifest.xml 中指定的應用程式健康狀態原則。


<br>

| 參數 | 說明 |
| --- | --- |
| ConsiderWarningAsError | 預設值為 False。升級期間評估應用程式健康狀態時，將應用程式的警告健康狀態事件視為錯誤。根據預設，Service Fabric 不會將警告健康狀態事件評估為失敗 (錯誤)，因此，即使有警告事件，升級還是可以繼續執行。 |
| MaxPercentUnhealthyDeployedApplications | 預設值和建議值為 0。指定應用程式被視為不健康和更新失敗之前，應用程式不健康之已部署的應用程式數目上限 (請參閱[健康狀態章節](service-fabric-health-introduction.md))。這個參數定義節點上的應用程式健康狀態，協助在升級期間偵測問題。應用程式複本通常會負載平衡至其他節點，讓應用程式健康狀態良好，以便繼續升級。藉由指定嚴謹的 MaxPercentUnhealthyDeployedApplications 健康狀態，Service Fabric 可以快速偵測應用程式封裝的問題，並協助產生失敗快速升級。 |
| MaxPercentUnhealthyServices | 預設值和建議值為 0。指定應用程式被視為不健康和升級失敗之前，不健康之應用程式執行個體中服務的數目上限。 |
| MaxPercentUnhealthyPartitionsPerService | 預設值和建議值為 0。指定服務被視為不健康之前，不健康之服務中分割的數目上限。 |
| MaxPercentUnhealthyReplicasPerPartition | 預設值和建議值為 0。指定分割被視為不健康之前，不健康之分割中複本的數目上限。 |
| UpgradeReplicaSetCheckTimeout | **無狀態服務** - 在單一升級網域內，Service Fabric 會嘗試確保服務的額外執行個體可用。如果目標執行個體計數超過一個，Service Fabric 會等到有一個以上的執行個體可用，最多到逾時值上限。此逾時是使用 UpgradeReplicaSetCheckTimeout 屬性指定。如果逾時已到期，Service Fabric 會繼續進行升級，不論服務執行個體數目。如果目標執行個體計數是一個，Service Fabric 不會等待，它會立即繼續進行升級。**可設定狀態的服務** - 在單一升級網域內，Service Fabric 會嘗試確保複本集有仲裁。Service Fabric 會等待有仲裁可用，最多到逾時值上限 (由 UpgradeReplicaSetCheckTimeout 屬性指定)。如果逾時已到期，Service Fabric 會繼續進行升級，不論是否有仲裁。此設定在向前回復時是設定為永不 (無限)，向後回復時則為 900 秒。 |
| ForceRestart | 如果您更新組態或資料封裝而不更新服務程式碼，只有當 ForceRestart 屬性設為 true 時，服務才會重新啟動。更新完成時，Service Fabric 會通知服務，新的組態封裝或資料封裝已可使用。服務會負責套用變更。必要時，服務可以自行重新啟動。 |



<br><br>可以針對應用程式執行個體的各個服務類型指定 MaxPercentUnhealthyServices、MaxPercentUnhealthyPartitionsPerService 和 MaxPercentUnhealthyReplicasPerPartition 準則。為每個服務設定這些參數可讓應用程式包含有不同評估原則的不同服務類型。例如，特定應用程式執行個體的無狀態閘道器服務類型，可以有與可設定狀態的引擎服務類型不同的 MaxPercentUnhealthyPartitionsPerService。

## 後續步驟

[使用 Visual Studio 升級您的應用程式](service-fabric-application-upgrade-tutorial.md)將引導您完成使用 Visual Studio 進行應用程式升級的步驟。

[使用 PowerShell 升級您的應用程式](service-fabric-application-upgrade-tutorial-powershell.md)將引導您完成使用 PowerShell 進行應用程式升級的步驟。

了解如何使用[資料序列化](service-fabric-application-upgrade-data-serialization.md)，以讓您的應用程式升級相容。

參考[進階主題](service-fabric-application-upgrade-advanced.md)，以了解如何在升級您的應用程式時使用進階功能。

參考[疑難排解應用程式升級](service-fabric-application-upgrade-troubleshooting.md)中的步驟，以修正應用程式升級中常見的問題。
 

<!---HONumber=AcomDC_0921_2016-->