<properties
	pageTitle="在 Azure Batch 中使用多重執行個體工作來執行 MPI 應用程式 | Microsoft Azure"
	description="了解如何在 Azure Batch 中使用多重執行個體工作類來執行訊息傳遞介面 (MPI) 應用程式。"
	services="batch"
	documentationCenter=".net"
	authors="mmacy"
	manager="timlt"
	editor="" />

<tags
	ms.service="batch"
	ms.devlang="multiple"
	ms.topic="article"
	ms.tgt_pltfrm="vm-windows"
	ms.workload="big-compute"
	ms.date="08/29/2016"
	ms.author="marsma" />

# 在 Azure Batch 中使用多重執行個體工作來執行訊息傳遞介面 (MPI) 應用程式

多重執行個體工作可讓您在多個計算節點上同時執行 Azure Batch 工作。這些工作可以在 Batch 中實現高效能運算案例，例如訊息傳遞介面 (MPI) 應用程式。在本文中，您將了解如何使用 [Batch .NET][api_net] 程式庫來執行多重執行個體工作。

## 多重執行個體工作概觀

在 Batch 中，每個工作通常是在單一計算節點上執行 -- 您將多個工作提交給作業，而 Batch 服務會將每個工作排定在節點上執行。不過，藉由設定工作的**多重執行個體設定**，您可以指示 Batch 將該工作分割成子工作，以便在多個節點上執行。

![多重執行個體工作概觀][1]

將具有多重執行個體設定的工作提交給作業時，Batch 會執行多重執行個體工作特有的幾個步驟：

1. Batch 服務自動將工作分割成一個**主要工作**和許多**子工作**。Batch 接著將主要工作和子工作排定在集區的計算節點上執行。
2. 這些工作 (主要工作和子工作) 會下載您在多重執行個體設定中指定的任何**一般資源檔**。
3. 下載一般資源檔之後，主要工作和子工作會執行您在多重執行個體設定中指定的**協調命令**。此協調命令通常用來啟動背景服務 (例如 [Microsoft MPI][msmpi_msdn] 的 `smpd.exe`)，也可能會確認節點已準備好處理節點間的訊息。
4. 主要工作及所有子工作順利完成協調命令之後，「只有」**主要工作**會執行多重執行個體工作的**命令列** (「應用程式命令」)。例如，在 [MS-MPI][msmpi_msdn] 架構的方案中，您會在這裡使用 `mpiexec.exe` 執行已啟用 MPI 的應用程式。

> [AZURE.NOTE] 雖然「多重執行個體工作」在功能上不同，但不是特殊的工作類型，例如 [StartTask][net_starttask] 或 [JobPreparationTask][net_jobprep]。多重執行個體工作只是已設定多重執行個體設定的 Standard Batch 工作 (Batch .NET 中的 [CloudTask][net_task])。在本文中，我們將它稱為**多重執行個體工作**。

## 多重執行個體工作的需求

多重執行個體工作需要有**已啟用節點間通訊**和**已停用並行工作執行**的集區。如果您嘗試在已停用節點間通訊，或「maxTasksPerNode」值大於 1 的集區中執行多重執行個體工作，則永遠不會排定工作--它會無限期停留在「作用中」狀態。此程式碼片段顯示如何使用 Batch .NET 程式庫建立這種集區。

```csharp
CloudPool myCloudPool =
	myBatchClient.PoolOperations.CreatePool(
		poolId: "MultiInstanceSamplePool",
		targetDedicated: 3
		virtualMachineSize: "small",
		cloudServiceConfiguration: new CloudServiceConfiguration(osFamily: "4"));

// Multi-instance tasks require inter-node communication, and those nodes
// must run only one task at a time.
myCloudPool.InterComputeNodeCommunicationEnabled = true;
myCloudPool.MaxTasksPerComputeNode = 1;
```

此外，多重執行個體工作「只」可以在 **2015 年 12 月 14 日之後建立的集區**中的節點上執行。

> [AZURE.TIP] 當您在 Batch 集區中使用大小 [A8 或 A9 計算節點](../virtual-machines/virtual-machines-windows-a8-a9-a10-a11-specs.md)時，MPI 應用程式可以利用 Azure 的高效能、低延遲的遠端直接記憶體存取 (RDMA) 網路。[雲端服務的大小](./../cloud-services/cloud-services-sizes-specs.md)提供 Batch 集區中可用的計算節點大小的完整清單。

### 使用 StartTask 來安裝 MPI 應用程式

若要使用多重執行個體工作來執行 MPI 應用程式，您首先需要將 MPI 軟體安裝到集區中的計算節點。這是使用 [StartTask][net_starttask] 的好時機，每當節點加入集區或重新啟動時，它就會執行。此程式碼片段會建立 StartTask 將 MS-MPI 安裝套件指定為[資源檔][net_resourcefile]，並指定將於資源檔下載至節點之後執行的命令列。

```csharp
// Create a StartTask for the pool which we use for installing MS-MPI on
// the nodes as they join the pool (or when they are restarted).
StartTask startTask = new StartTask
{
    CommandLine = "cmd /c MSMpiSetup.exe -unattend -force",
    ResourceFiles = new List<ResourceFile> { new ResourceFile("https://mystorageaccount.blob.core.windows.net/mycontainer/MSMpiSetup.exe", "MSMpiSetup.exe") },
    RunElevated = true,
    WaitForSuccess = true
};
myCloudPool.StartTask = startTask;

// Commit the fully configured pool to the Batch service to actually create
// the pool and its compute nodes.
await myCloudPool.CommitAsync();
```

> [AZURE.NOTE] 當您在 Batch 中使用多重執行個體工作實作 MPI 方案時，不限於只能使用 MS-MPI。任何 MPI 標準實作只要與您為集區中的計算節點指定的作業系統相容，都可使用。

## 使用 Batch .NET 建立多個執行個體工作

既然我們已討論過集區需求和 MPI 套件安裝，現在讓我們建立多重執行個體工作。在此片段中，我們會建立標準 [CloudTask][net_task]，然後設定其 [MultiInstanceSettings][net_multiinstance_prop] 屬性。如先前所述，多重執行個體工作不是獨特的工作類型，而只是已設定多重執行個體設定的標準 Batch 工作。

```csharp
// Create the multi-instance task. Its command line is the "application command"
// and will be executed *only* by the primary, and only after the primary and
// subtasks execute the CoordinationCommandLine.
CloudTask myMultiInstanceTask = new CloudTask(id: "mymultiinstancetask",
    commandline: "cmd /c mpiexec.exe -wdir %AZ_BATCH_TASK_SHARED_DIR% MyMPIApplication.exe");

// Configure the task's MultiInstanceSettings. The CoordinationCommandLine will be executed by
// the primary and all subtasks.
myMultiInstanceTask.MultiInstanceSettings =
    new MultiInstanceSettings(numberOfNodes) {
    CoordinationCommandLine = @"cmd /c start cmd /c ""%MSMPI_BIN%\smpd.exe"" -d",
    CommonResourceFiles = new List<ResourceFile> {
    new ResourceFile("https://mystorageaccount.blob.core.windows.net/mycontainer/MyMPIApplication.exe",
                     "MyMPIApplication.exe")
    }
};

// Submit the task to the job. Batch will take care of splitting it into subtasks and
// scheduling them for execution on the nodes.
await myBatchClient.JobOperations.AddTaskAsync("mybatchjob", myMultiInstanceTask);
```

## 主要工作和子工作

當您建立工作的多重執行個體設定時，您需要指定用來執行工作的計算節點數目。當您將工作提交給作業時，Batch 服務會建立一個**主要工作**和足夠的**子工作**，而且合計符合您指定的節點數。

系統會指派範圍介於 0 到 *numberOfInstances-1* 的整數識別碼給這些工作。識別碼 0 的工作是主要工作，其他所有識別碼都是子工作。比方說，如果您為工作建立下列多重執行個體設定，則主要工作的識別碼為 0，而子工作的識別碼為 1 到 9。

```csharp
int numberOfNodes = 10;
myMultiInstanceTask.MultiInstanceSettings = new MultiInstanceSettings(numberOfNodes);
```

## 協調命令

主要工作和子工作都會執行**協調命令**。

阻止叫用協調命令 -- 在所有子工作的協調命令順利傳回之前，Batch 不會執行應用程式命令。因此，協調命令應該啟動任何所需的背景服務，確認它們已準備好可供使用，然後結束。比方說，在使用 MS-MPI 第 7 版的方案中，此協調命令會在節點上啟動 SMPD 服務，然後結束：

```
cmd /c start cmd /c ""%MSMPI_BIN%\smpd.exe"" -d
```

請注意此協調命令中使用 `start`。這是必要的，因為 `smpd.exe` 應用程式不會在執行後立即傳回。如果不使用 [start][cmd_start] 命令，此協調命令就不會傳回，因此將阻止執行應用程式命令。

## 應用程式命令

主要工作及所有子工作完成執行協調命令之後，「只有」主要工作會執行多重執行個體工作的命令列。我們將此命令列稱為**應用程式命令**，以便與協調命令有所區別。

針對 MS-MPI 應用程式，使用應用程式命令以 `mpiexec.exe` 執行已啟用 MPI 的應用程式。例如，以下是使用 MS-MPI 第 7 版的方案所執行的應用程式命令：

```
cmd /c ""%MSMPI_BIN%\mpiexec.exe"" -c 1 -wdir %AZ_BATCH_TASK_SHARED_DIR% MyMPIApplication.exe
```

>[AZURE.NOTE] 因為 MS-MPI 的 `mpiexec.exe` 預設會使用 `CCP_NODES` 變數 (請參閱[環境變數](#environment-variables))，上述範例應用程式命令列會排除它。

## 環境變數

這些環境變數可在計算節點上取得，以用於主要工作的應用程式命令所執行的應用程式和指令碼︰

* `CCP_NODES`

  * **可用性**：CloudServiceConfiguration 和 VirtualMachineConfiguration 集區
  * **格式**：`numNodes<space>nodeIP<space>numCores<space>`
  * **範例**：`2 10.0.0.4 1 10.0.0.5 1`

* `AZ_BATCH_NODE_LIST`

  * **可用性**：CloudServiceConfiguration 和 VirtualMachineConfiguration 集區
  * **格式**：`nodeIP;nodeIP`
  * **範例**：`10.0.0.4;10.0.0.5`

* `AZ_BATCH_HOST_LIST`

  * **可用性**：VirtualMachineConfiguration 集區
  * **格式**：`nodeIP,nodeIP`
  * **範例**：`10.0.0.4,10.0.0.5`

## 資源檔

多重執行個體工作需要考量兩組資源檔：「所有」工作 (主要工作和子工作) 下載的**一般資源檔**，以及為多重執行個體工作本身指定的**資源檔** (「只有主要」工作會下載)。

您可以在工作的多重執行個體設定中指定一或多個**一般資源檔**。主要工作及所有子工作會從 [Azure 儲存體](./../storage/storage-introduction.md)，將這些一般資源檔下載到每個節點的工作共用目錄。您可以使用 `AZ_BATCH_TASK_SHARED_DIR` 環境變數從應用程式命令和協調命令列存取工作共用目錄。

「只有」主要工作會將您為多重執行個體工作本身指定的資源檔，下載到工作的工作目錄 `AZ_BATCH_TASK_WORKING_DIR`-- 子工作不會下載為多重執行個體工作指定的資源檔。

節點上執行的主要工作及所有子工作可存取 `AZ_BATCH_TASK_SHARED_DIR` 的內容。`tasks/mybatchjob/job-1/mymultiinstancetask/` 是工作共用目錄的一個例子。主要工作和每項子工作也有一個只有該工作才能存取的工作目錄，而且可使用環境變數 `AZ_BATCH_TASK_WORKING_DIR` 來存取。

請注意，在本文的程式碼範例中，我們未指定多重執行個體工作本身的資源檔，只有為集區的 StartTask 和多重執行個體設定的 [CommonResourceFiles][net_multiinsance_commonresfiles] 指定資源檔。

> [AZURE.IMPORTANT] 在命令列中，請一律使用環境變數 `AZ_BATCH_TASK_SHARED_DIR` 和 `AZ_BATCH_TASK_WORKING_DIR` 來參考這些目錄。請勿嘗試以手動方式建構路徑。

## 工作存留期

主要工作的存留期控制整個多重執行個體工作的存留期。當主要工作結束時，所有子工作就會終止。主要工作的結束代碼就是工作的結束代碼，因此在重試用途上用於判斷工作成功或失敗。

如果任何子工作失敗，比方說結束時傳回碼不是零，則整個多重執行個體工作會失敗。接著會終止並重試多重執行個體工作，直到觸及重試限制為止。

當您刪除多重執行個體工作時，Batch 服務也會刪除主要工作和所有子工作。所有子工作目錄及其檔案會從計算節點中刪除，如同在標準工作中一樣。

多重執行個體工作的 [TaskConstraints][net_taskconstraints] \(例如 [MaxTaskRetryCount][net_taskconstraint_maxretry]、[MaxWallClockTime][net_taskconstraint_maxwallclock] 和 [RetentionTime][net_taskconstraint_retention] 屬性) 都視為用於標準工作，並套用至主要工作和所有子工作。不過，如果您在多重執行個體工作新增到作業之後變更 [RetentionTime][net_taskconstraint_retention] 屬性，這項變更只會套用到主要工作。所有的子工作會繼續使用原始 [RetentionTime][net_taskconstraint_retention]。

如果最近的工作是多重執行個體工作的一部分，計算節點的最近工作清單會反映子工作的識別碼。

## 取得子工作的相關資訊

若要使用 Batch .NET 程式庫取得子工作的詳細資訊，請呼叫 [CloudTask.ListSubtasks][net_task_listsubtasks] 方法。這個方法會傳回所有子工作的相關資訊，以及已執行工作的計算節點的相關資訊。您可以根據這項資訊判斷每項子工作的根目錄、集區識別碼、其目前狀態、結束代碼等等。您可以使用這項資訊結合 [PoolOperations.GetNodeFile][poolops_getnodefile] 方法，以取得子工作的檔案。請注意，這個方法不會傳回主要工作 (識別碼 0) 的相關資訊。

> [AZURE.NOTE] 除非另有指明，否則在多重執行個體 [CloudTask][net_task] 本身執行的 Batch .NET 方法「只會」套用到主要工作。例如，當您在多重執行個體工作上呼叫 [CloudTask.ListNodeFiles][net_task_listnodefiles] 方法時，只會傳回主要工作的檔案。

下列程式碼片段示範如何取得子工作資訊，以及從它們執行所在的節點要求檔案的內容。

```csharp
// Obtain the job and the multi-instance task from the Batch service
CloudJob boundJob = batchClient.JobOperations.GetJob("mybatchjob");
CloudTask myMultiInstanceTask = boundJob.GetTask("mymultiinstancetask");

// Now obtain the list of subtasks for the task
IPagedEnumerable<SubtaskInformation> subtasks = myMultiInstanceTask.ListSubtasks();

// Asynchronously iterate over the subtasks and print their stdout and stderr
// output if the subtask has completed
await subtasks.ForEachAsync(async (subtask) =>
{
    Console.WriteLine("subtask: {0}", subtask.Id);
    Console.WriteLine("exit code: {0}", subtask.ExitCode);

    if (subtask.State == TaskState.Completed)
    {
        ComputeNode node =
			await batchClient.PoolOperations.GetComputeNodeAsync(subtask.ComputeNodeInformation.PoolId,
                                                                 subtask.ComputeNodeInformation.ComputeNodeId);

        NodeFile stdOutFile = await node.GetNodeFileAsync(subtask.ComputeNodeInformation.TaskRootDirectory + "\" + Constants.StandardOutFileName);
        NodeFile stdErrFile = await node.GetNodeFileAsync(subtask.ComputeNodeInformation.TaskRootDirectory + "\" + Constants.StandardErrorFileName);
        stdOut = await stdOutFile.ReadAsStringAsync();
        stdErr = await stdErrFile.ReadAsStringAsync();

        Console.WriteLine("node: {0}:", node.Id);
        Console.WriteLine("stdout.txt: {0}", stdOut);
        Console.WriteLine("stderr.txt: {0}", stdErr);
    }
    else
    {
        Console.WriteLine("\tSubtask {0} is in state {1}", subtask.Id, subtask.State);
    }
});
```

## 後續步驟

- 您可以建置簡單的 MS-MPI 應用程式，供 Batch 中測試多重執行個體工作時使用。Microsoft HPC 和 Azure Batch 小組部落格文章[如何編譯及執行簡單的 MS-MPI 程式][msmpi_howto]中，包含使用 MS-MPI 建立簡單的 MPI 應用程式的逐步解說。

- 如需 MS-MPI 的最新資訊，請參閱 MSDN 上的 [Microsoft MPI][msmpi_msdn] 頁面。

[api_net]: http://msdn.microsoft.com/library/azure/mt348682.aspx
[api_rest]: http://msdn.microsoft.com/library/azure/dn820158.aspx
[batch_explorer]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/BatchExplorer
[cmd_start]: https://technet.microsoft.com/library/cc770297.aspx
[github_samples]: https://github.com/Azure/azure-batch-samples
[msmpi_msdn]: https://msdn.microsoft.com/library/bb524831.aspx
[msmpi_sdk]: http://go.microsoft.com/FWLink/p/?LinkID=389556
[msmpi_howto]: http://blogs.technet.com/b/windowshpc/archive/2015/02/02/how-to-compile-and-run-a-simple-ms-mpi-program.aspx

[net_jobprep]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.jobpreparationtask.aspx
[net_multiinstance_class]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.multiinstancesettings.aspx
[net_multiinstance_prop]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.multiinstancesettings.aspx
[net_multiinsance_commonresfiles]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.multiinstancesettings.commonresourcefiles.aspx
[net_multiinstance_coordcmdline]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.multiinstancesettings.coordinationcommandline.aspx
[net_multiinstance_numinstances]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.multiinstancesettings.numberofinstances.aspx
[net_pool]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.aspx
[net_pool_create]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.createpool.aspx
[net_pool_starttask]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.starttask.aspx
[net_resourcefile]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.resourcefile.aspx
[net_starttask]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.starttask.aspx
[net_starttask_cmdline]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.starttask.commandline.aspx
[net_task]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.aspx
[net_taskconstraints]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.taskconstraints.aspx
[net_taskconstraint_maxretry]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.taskconstraints.maxtaskretrycount.aspx
[net_taskconstraint_maxwallclock]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.taskconstraints.maxwallclocktime.aspx
[net_taskconstraint_retention]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.taskconstraints.retentiontime.aspx
[net_task_listsubtasks]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.listsubtasks.aspx
[net_task_listnodefiles]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.listnodefiles.aspx
[poolops_getnodefile]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.getnodefile.aspx

[rest_multiinstance]: https://msdn.microsoft.com/library/azure/mt637905.aspx

[1]: ./media/batch-mpi/batch_mpi_01.png "多重執行個體概觀"

<!---HONumber=AcomDC_0831_2016-->