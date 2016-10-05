<properties
   pageTitle="在 Service Fabric 叢集中引發混亂 | Microsoft Azure"
   description="使用錯誤注射與叢集分析服務的 API 來管理叢集中的混亂。"
   services="service-fabric"
   documentationCenter=".net"
   authors="motanv"
   manager="rsinha"
   editor="toddabel"/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="09/19/2016"
   ms.author="motanv"/>

# 在 Service Fabric 叢集中引發受控制的混亂
雲端基礎結構之類的大型分散式系統本身並不可靠。Azure Service Fabric 可讓開發人員在不可靠的基礎結構上面撰寫可靠的服務。為了撰寫強健的服務，開發人員必須能夠針對這類不可靠的基礎結構引發錯誤，才能測試其服務的穩定性。

「錯誤注射與叢集分析服務」(也稱為 FAS) 讓開發人員可以引發錯誤動作來測試服務。但鎖定式的模擬錯誤就僅只於此。若要進一步測試，可以使用「混亂」。混亂會以很長的時間在整個叢集上模擬連續的交錯錯誤，包括非失誤性和失誤性錯誤。一旦設定混亂的比率和錯誤類型後，即可透過 C# API 或 PowerShell 啟動或停止，在叢集和您的服務中產生錯誤。混亂執行時，會產生不同事件來擷取目前執行的狀態。例如，ExecutingFaultsEvent 包含在該次反覆運算中正在執行的所有錯誤；ValidationFailedEvent 則包含叢集驗證期間發現的錯誤詳細資料。您可以叫用 GetChaosReportAsync API 來取得混亂執行的報告。

## 混亂中引發的錯誤
混亂會在整個 Service Fabric 叢集中產生錯誤，並將在幾個月或幾年內看到的錯誤壓縮成幾小時。交錯錯誤和高錯誤率的組合，可以找到會在其他情形下被遺漏的極端狀況。這個混亂練習可以大幅提升服務的程式碼品質。混亂會引發下列類別的錯誤︰

 - 重新啟動節點
 - 重新啟動已部署的程式碼封裝
 - 移除複本
 - 重新啟動複本
 - 移動主要複本 (可設定)
 - 移動次要複本 (可設定)

混亂會多次反覆執行；每次反覆運算都包含指定期間的錯誤和叢集驗證。讓叢集穩定和驗證成功的所需時間可以設定。如果在叢集驗證中發現失敗，則混亂會產生並保留一個 ValidationFailedEvent，包含 UTC 時間戳記與失敗詳細資料。

例如，考慮一個設為執行 1 小時且最多有 3 個並行錯誤的混亂執行個體。混亂會引發三個錯誤，然後驗證叢集健康狀態，並重複執行上一個步驟，直到透過 StopChaosAsync API 或經過一小時後就會明確停止。如果叢集健康狀態在任何反覆運算中變得不良，也就是不會在設定的時間內穩定下來，混亂會產生 ValidationFailedEvent，指出某些地方發生錯誤，可能需要進一步調查。

目前來說，混亂只會引發安全的錯誤，這表示如果沒有外部錯誤，絕不會發生仲裁遺失或資料遺失。

## 重要的組態選項
 - **TimeToRun**：混亂在成功完成前的總執行時間。混亂在執行 TimeToRun 這段時間之前可以透過 StopChaos API 停止。
 - **MaxClusterStabilizationTimeout**︰再檢查一次之前等候叢集健康狀態變得良好的最長時間，這樣的等候是為了減少復原時叢集上的負載。執行的檢查為
    - 叢集健康狀態是否正常
    - 服務健康狀態正常
    - 服務分割區達到目標複本集大小
    - 沒有 InBuild 複本存在
 - **MaxConcurrentFaults**：每個反覆運算中引發的最大並行錯誤數。數目越大，混亂會越積極，因此導致更複雜的容錯移轉和轉換組合。無論此組態的值多高，混亂都能保證在沒有外部錯誤的狀況下都不會發生仲裁或資料遺失。
 - **EnableMoveReplicaFaults**：啟用或停用造成主要或次要複本移動的錯誤。預設會停用這些錯誤。
 - **WaitTimeBetweenIterations**：反覆運算之間 (也就是在一輪的錯誤與對應的驗證後等待下一輪) 的等待時間長度。
 - **WaitTimeBetweenFaults**︰反覆運算中兩個連續錯誤之間的等候時間長度。

## 如何執行混亂
C# 範例

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using System.Fabric;

using System.Diagnostics;
using System.Fabric.Chaos.DataStructures;

class Program
{
    private class ChaosEventComparer : IEqualityComparer<ChaosEvent>
    {
        public bool Equals(ChaosEvent x, ChaosEvent y)
        {
            return x.TimeStampUtc.Equals(y.TimeStampUtc);
        }

        public int GetHashCode(ChaosEvent obj)
        {
            return obj.TimeStampUtc.GetHashCode();
        }
    }

    static void Main(string[] args)
    {
        var clusterConnectionString = "localhost:19000";
        using (var client = new FabricClient(clusterConnectionString))
        {
            var startTimeUtc = DateTime.UtcNow;
            var stabilizationTimeout = TimeSpan.FromSeconds(30.0);
            var timeToRun = TimeSpan.FromMinutes(60.0);
            var maxConcurrentFaults = 3;

            var parameters = new ChaosParameters(
                stabilizationTimeout,
                maxConcurrentFaults,
                true, /* EnableMoveReplicaFault */
                timeToRun);

            try
            {
                client.TestManager.StartChaosAsync(parameters).GetAwaiter().GetResult();
            }
            catch (FabricChaosAlreadyRunningException)
            {
                Console.WriteLine("An instance of Chaos is already running in the cluster.");
            }

            var filter = new ChaosReportFilter(startTimeUtc, DateTime.MaxValue);

            var eventSet = new HashSet<ChaosEvent>(new ChaosEventComparer());

            while (true)
            {
                var report = client.TestManager.GetChaosReportAsync(filter).GetAwaiter().GetResult();

                foreach (var chaosEvent in report.History)
                {
                    if (eventSet.add(chaosEvent))
                    {
                        Console.WriteLine(chaosEvent);
                    }
                }

                // When Chaos stops, a StoppedEvent is created.
                // If StoppedEvent is found, exit the loop.
                var lastEvent = report.History.LastOrDefault();

                if (lastEvent is StoppedEvent)
                {
                    break;
                }

                Task.Delay(TimeSpan.FromSeconds(1.0)).GetAwaiter().GetResult();
            }
        }
    }
}
```
PowerShell

```powershell
$connection = "localhost:19000"
$timeToRun = 60
$maxStabilizationTimeSecs = 180
$concurrentFaults = 3
$waitTimeBetweenIterationsSec = 60

Connect-ServiceFabricCluster $connection

$events = @{}
$now = [System.DateTime]::UtcNow

Start-ServiceFabricChaos -TimeToRunMinute $timeToRun -MaxConcurrentFaults $concurrentFaults -MaxClusterStabilizationTimeoutSec $maxStabilizationTimeSecs -EnableMoveReplicaFaults -WaitTimeBetweenIterationsSec $waitTimeBetweenIterationsSec

while($true)
{
    $stopped = $false
    $report = Get-ServiceFabricChaosReport -StartTimeUtc $now -EndTimeUtc ([System.DateTime]::MaxValue)

    foreach ($e in $report.History) {

        if(-Not ($events.Contains($e.TimeStampUtc.Ticks)))
        {
            $events.Add($e.TimeStampUtc.Ticks, $e)
            if($e -is [System.Fabric.Chaos.DataStructures.ValidationFailedEvent])
            {
                Write-Host -BackgroundColor White -ForegroundColor Red $e
            }
            else
            {
                if($e -is [System.Fabric.Chaos.DataStructures.StoppedEvent])
                {
                    $stopped = $true
                }

                Write-Host $e
            }
        }
    }

    if($stopped -eq $true)
    {
        break
    }

    Start-Sleep -Seconds 1
}

Stop-ServiceFabricChaos
```

<!---HONumber=AcomDC_0921_2016-->