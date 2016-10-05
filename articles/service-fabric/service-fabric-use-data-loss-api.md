<properties
   pageTitle="如何在 Service Fabric 服務上叫用資料遺失 | Microsoft Azure"
   description="說明如何使用資料遺失 API"
   services="service-fabric"
   documentationCenter=".net"
   authors="LMWF"
   manager="rsinha"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="09/19/2016"
   ms.author="lemai"/>
   
# 如何在服務上叫用資料遺失

>[AZURE.WARNING] 本文件說明如何在您的服務中造成資料遺失，而且應小心使用。

## 簡介
您可以在 Service Fabric 服務的分割區上，藉由呼叫 StartPartitionDataLossAsync() 來叫用資料遺失。這個 API 會使用錯誤插入和分析服務來執行工作以造成資料遺失的狀況。

## 使用錯誤插入和分析服務

錯誤插入和分析服務目前支援下列圖表中的 API。圖表右側顯示對應的 PowerShell Cmdlet。如需每個 API 的詳細資訊，請參閱關於每個 API 的 MSDN 文件。

| C# API | PowerShell Cmdlet |
|-------------------------------------|-----------------------------------------------:|
|[StartPartitionDataLossAsync][dl] |[Start-ServiceFabricPartitionDataLoss][psdl] |
|[StartPartitionQuorumLossAsync][ql] |[Start-ServiceFabricPartitionQuorumLoss][psql] |
|[StartPartitionRestartAsync][rp] |[Start-ServiceFabricPartitionRestart][psrp] |

## 執行命令的概念概觀

錯誤插入和分析服務會使用非同步模型，您會在其中使用一個本文件中稱為 “Start” API 的 API 來啟動命令，然後使用 “GetProgress” API 檢查此命令的進度，直到其觸達終止狀態或您取消該命令為止。若要啟動命令，請針對對應的 API 呼叫 “Start” API。這個 API 會在錯誤插入和分析服務已接受要求時傳回。不過，它不會指出命令已執行多久時間，或者甚至尚未啟動。若要檢查命令的進度，可呼叫對應到先前呼叫的 “Start” API 的 “GetProgress” API。“GetProgress” API 將傳回物件，指出命令在其 State 屬性內目前的狀態。在符合下列條件之前，無限期執行命令：

1.	順利完成。如果您在此情況下於其上呼叫 “GetProgress”，進度物件的 State 會是 Completed。
2.	遇到嚴重錯誤。如果您在此情況下於其上呼叫 “GetProgress”，進度物件的 State 會是 Faulted
3.	您可以透過 [CancelTestCommandAsync][cancel] API 或 [Stop-ServiceFabricTestCommand][cancelps] PowerShell Cmdlet 來取消它。如果您在此情況下於其上呼叫 “GetProgress”，根據該 API 的引數而定，進度物件的狀態會是 Cancelled 或 ForceCancelled。如需詳細資訊，請參閱適用於 [CancelTestCommandAsync][cancel] 的文件。


## 執行命令的詳細資料

若要啟動命令，可搭配預期的引數呼叫 Start API。所有的 Start API 都具有名為 operationId 的 Guid 引數。您應該持續追蹤 operationId 引數，因為它可用來追蹤此命令的進度。這必須傳遞至 “GetProgress” API，以便追蹤命令的進度。OperationId 必須是唯一的。

成功呼叫 Start API 之後，應該在迴圈中呼叫 GetProgress API，直到所傳回進度物件的 State 屬性為 Completed 為止。所有的 [FabricTransientException][fte] 和 OperationCanceledException 都應重試。當命令觸達終止狀態 (Completed、Faulted 或 Cancelled) 時，所傳回進度物件的 Result 屬性將具有額外的資訊。如果狀態為 Completed，Result.SelectedPartition.PartitionId 將包含所選取的分割識別碼。Result.Exception 會是 null。如果狀態為 Faulted，Result.Exception 將產生錯誤插入和分析服務無法執行該命令的理由。Result.SelectedPartition.PartitionId 將具有所選取的分割識別碼。在某些情況下，會因為命令執行的程度還不夠而無法選擇分割區。在此情況下，PartitionId 會是 0。如果狀態為 Cancelled，則 Result.Exception 會是 null。與 Faulted 情況類似，Result.SelectedPartition.PartitionId 將具有所選擇的分割識別碼，但是如果因為命令執行的程度還不夠而無法執行此動作，則為 0。另請參閱下面的範例。

下列範例程式碼示範如何啟動然後檢查命令上的進度以造成特定分割區遺失資料。

```csharp
    static async Task PerformDataLossSample()
    {
        // Create a unique operation id for the command below
        Guid operationId = Guid.NewGuid();

        // Note: Use the appropriate overload for your configuration
        FabricClient fabricClient = new FabricClient();

        // The name of the target service
        Uri targetServiceName = new Uri("fabric:/MyService");

        // The id of the target partition inside the target service
        Guid targetPartitionId = new Guid("00000000-0000-0000-0000-000002233445");

        PartitionSelector partitionSelector = PartitionSelector.PartitionIdOf(targetServiceName, targetPartitionId);

        // Start the command.  Retry OperationCanceledException and all FabricTransientException's.  Note when StartPartitionDataLossAsync completes
        // successfully it only means the Fault Injection and Analysis Service has saved the intent to perform this work.  It does not say anything about the progress
        // of the command.
        while (true)
        {
            try
            {
                await fabricClient.TestManager.StartPartitionDataLossAsync(operationId, partitionSelector, DataLossMode.FullDataLoss).ConfigureAwait(false);
                break;
            }
            catch (OperationCanceledException)
            {
            }
            catch (FabricTransientException)
            {
            }

            await Task.Delay(TimeSpan.FromSeconds(1.0d)).ConfigureAwait(false);
        }

        PartitionDataLossProgress progress = null;

        // Poll the progress using GetPartitionDataLossProgressAsync until it is either Completed or Faulted.  In this example, we're assuming
        // the command won't be cancelled.        

        while (true)
        {
            try
            {
                progress = await fabricClient.TestManager.GetPartitionDataLossProgressAsync(operationId).ConfigureAwait(false);
            }
            catch (OperationCanceledException)
            {
                continue;
            }
            catch (FabricTransientException)
            {
                continue;
            }

            if (progress.State == TestCommandProgressState.Completed)
            {
                Console.WriteLine("Command '{0}' completed successfully", operationId);

                // In a terminal state .Result.SelectedPartition.PartitionId will have the chosen partition
                Console.WriteLine("  Printing selected partition='{0}'", progress.Result.SelectedPartition.PartitionId);
                break;
            }
            else if (progress.State == TestCommandProgressState.Faulted)
            {
                // If State is Faulted, the progress object's Result property's Exception property will have the reason why.
                Console.WriteLine("Command '{0}' failed with '{1}'", operationId, progress.Result.Exception);
                break;
            }
            else
            {
                Console.WriteLine("Command '{0}' is currently Running", operationId);
            }

            await Task.Delay(TimeSpan.FromSeconds(5.0d)).ConfigureAwait(false);
        }
    }
```

下列範例示範如何使用 PartitionSelector 來選擇指定服務的隨機分割區：

```csharp
    static async Task PerformDataLossUseSelectorSample()
    {
        // Create a unique operation id for the command below
        Guid operationId = Guid.NewGuid();

        // Note: Use the appropriate overload for your configuration
        FabricClient fabricClient = new FabricClient();

        // The name of the target service
        Uri targetServiceName = new Uri("fabric:/SampleService ");

        // Use a PartitionSelector that will have the Fault Injection and Analysis Service choose a random partition of “targetServiceName”
        PartitionSelector partitionSelector = PartitionSelector.RandomOf(targetServiceName);

        // Start the command.  Retry OperationCanceledException and all FabricTransientException's.  Note when StartPartitionDataLossAsync completes
        // successfully it only means the Fault Injection and Analysis Service has saved the intent to perform this work.  It does not say anything about the progress
        // of the command.
        while (true)
        {
            try
            {
                await fabricClient.TestManager.StartPartitionDataLossAsync(operationId, partitionSelector, DataLossMode.FullDataLoss).ConfigureAwait(false);
                break;
            }
            catch (OperationCanceledException)
            {
            }
            catch (FabricTransientException)
            {
            }
            catch (Exception e)
            {
                Console.WriteLine("Unexpected exception '{0}'", e);
                throw;
            }

            await Task.Delay(TimeSpan.FromSeconds(1.0d)).ConfigureAwait(false);
        }

        PartitionDataLossProgress progress = null;

        // Poll the progress using GetPartitionDataLossProgressAsync until it is either Completed or Faulted.  In this example, we're assuming
        // the command won't be cancelled.

        while (true)
        {
            try
            {
                progress = await fabricClient.TestManager.GetPartitionDataLossProgressAsync(operationId).ConfigureAwait(false);
            }
            catch (OperationCanceledException)
            {
                continue;
            }
            catch (FabricTransientException)
            {
                continue;
            }

            if (progress.State == TestCommandProgressState.Completed)
            {
                Console.WriteLine("Command '{0}' completed successfully", operationId);

                Console.WriteLine("Printing progress.Result:");
                Console.WriteLine("  Printing selected partition='{0}'", progress.Result.SelectedPartition.PartitionId);

                break;
            }
            else if (progress.State == TestCommandProgressState.Faulted)
            {
                // If State is Faulted, the progress object's Result property's Exception property will have the reason why.
                Console.WriteLine("Command '{0}' failed with '{1}', SelectedPartition {2}", operationId, progress.Result.Exception, progress.Result.SelectedPartition);
                break;
            }
            else
            {
                Console.WriteLine("Command '{0}' is currently Running", operationId);
            }

            await Task.Delay(TimeSpan.FromSeconds(1.0d)).ConfigureAwait(false);
        }
    }
```

## 歷程記錄與截斷

當命令已觸達終止狀態之後，它的中繼資料將會在錯誤插入和分析服務中保留一段時間，之後才會將它移除以節省空間。如果在移除命令之後使用該命令的 operationId 來呼叫 “GetProgress”，將會傳回 FabricException 且 ErrorCode 為 KeyNotFound。

[dl]: https://msdn.microsoft.com/library/azure/mt693569.aspx
[ql]: https://msdn.microsoft.com/library/azure/mt693558.aspx
[rp]: https://msdn.microsoft.com/library/azure/mt645056.aspx
[psdl]: https://msdn.microsoft.com/library/mt697573.aspx
[psql]: https://msdn.microsoft.com/library/mt697557.aspx
[psrp]: https://msdn.microsoft.com/library/mt697560.aspx
[cancel]: https://msdn.microsoft.com/library/azure/mt668910.aspx
[cancelps]: https://msdn.microsoft.com/library/mt697566.aspx
[fte]: https://msdn.microsoft.com/library/azure/system.fabric.fabrictransientexception.aspx

<!---HONumber=AcomDC_0921_2016-->