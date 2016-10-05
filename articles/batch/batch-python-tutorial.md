<properties
	pageTitle="教學課程 - 開始使用 Azure Batch Python 用戶端 | Microsoft Azure"
	description="了解 Azure Batch 的基本概念，以及利用簡單的案例了解如何開發 Batch 服務"
	services="batch"
	documentationCenter="python"
	authors="mmacy"
	manager="timlt"
	editor=""/>

<tags
	ms.service="batch"
	ms.devlang="python"
	ms.topic="hero-article"
	ms.tgt_pltfrm="na"
	ms.workload="big-compute"
	ms.date="09/27/2016"
	ms.author="marsma"/>

# 開始使用 Azure Batch Python 用戶端

> [AZURE.SELECTOR]
- [.NET](batch-dotnet-get-started.md)
- [Python](batch-python-tutorial.md)

在我們討論以 Python 撰寫的小型 Batch 應用程式時，了解 [Azure Batch][azure_batch] 和 [Batch Python][py_azure_sdk] 用戶端的基本概念。我們看看這兩個範例指令碼如何使用 Batch 服務來處理雲端中 Linux 虛擬機器上的平行工作負載，以及如何與 [Azure 儲存體](./../storage/storage-introduction.md)互動來預備和擷取檔案。您將了解常見的 Batch 應用程式工作流程，並取得 Batch 的主要元件，例如作業、工作、集區和計算節點。

![Batch 方案工作流程 (基本)][11]<br/>

## 必要條件

本文假設您已具備 Python 的使用知識並熟悉 Linux。而且假設您可以滿足針對 Azure Batch 和儲存體服務所指定的帳戶建立需求。

### 帳戶

- **Azure 帳戶**：如果您沒有 Azure 訂用帳戶，請[建立免費的 Azure 帳戶][azure_free_account]。
- **Batch 帳戶**：擁有 Azure 訂用帳戶後，請[建立 Azure Batch 帳戶](batch-account-create-portal.md)。
- **儲存體帳戶**：請參閱[關於 Azure 儲存體帳戶](../storage/storage-create-storage-account.md)中的[建立儲存體帳戶](../storage/storage-create-storage-account.md#create-a-storage-account)。

### 程式碼範例

Python 教學課程[程式碼範例][github_article_samples]是在 GitHub 上 [azure-batch-samples][github_samples] 儲存機制中找到的許多 Batch 程式碼範例之一。按一下儲存機制首頁上的 [複製或下載] > [下載 ZIP]，或按一下 [azure-batch-samples-master.zip][github_samples_zip] 直接下載連結，即可下載所有範例。解壓縮 ZIP 檔案的內容後，在 `article_samples` 目錄中可找到本教學課程的兩個指令碼︰

`/azure-batch-samples/Python/Batch/article_samples/python_tutorial_client.py`<br/> `/azure-batch-samples/Python/Batch/article_samples/python_tutorial_task.py`

### Python 環境

若要在本機工作站上執行「python\_tutorial\_client.py」範例指令碼，您需要與版本 **2.7** 或 **3.3+** 相容的 **Python 解譯器**。此指令碼已在 Linux 和 Windows 上測試。

### 密碼編譯相依項目

您必須為[密碼編譯][crypto]程式庫安裝 `azure-batch` 和 `azure-storage` Python 套件所需的相依項目。針對您的平台執行下列其中一個適用作業，或參閱[密碼編譯安裝][crypto_install]詳細資料以取得詳細資訊︰

* Ubuntu

    `apt-get update && apt-get install -y build-essential libssl-dev libffi-dev libpython-dev python-dev`

* CentOS

    `yum update && yum install -y gcc openssl-dev libffi-devel python-devel`

* SLES/OpenSUSE

    `zypper ref && zypper -n in libopenssl-dev libffi48-devel python-devel`

* Windows

    `pip install cryptography`

>[AZURE.NOTE] 如果要在 Linux 上安裝 Python 3.3+，請使用適用於 Python 相依項目的 python3 對等項目。例如，在 Ubuntu 上︰`apt-get update && apt-get install -y build-essential libssl-dev libffi-dev libpython3-dev python3-dev`

### Azure 套件

接下來請安裝 **Azure Batch** 和 **Azure 儲存體** Python 套件。使用此處找到的 **pip** 和 requirements.txt 即可完成此作業：

`/azure-batch-samples/Python/Batch/requirements.txt`

發出下列 **pip** 命令以安裝 Batch 和儲存體封裝：

`pip install -r requirements.txt`

或者，您可以手動方式安裝 [azure-batch][pypi_batch] 和 [azure-storage][pypi_storage] Python 封裝：

`pip install azure-batch`<br/> `pip install azure-storage`

> [AZURE.TIP] 如果您使用無特殊權限的帳戶，則可能需要在您的命令前面加上 `sudo`。例如，`sudo pip install -r requirements.txt`。如需有關如何安裝 Python 封裝的詳細資訊，請參閱 readthedocs.io 上的[安裝封裝][pypi_install]。

## Batch Python 教學課程程式碼範例

Batch Python 教學課程程式碼範例是由兩個 Python 指令碼和幾個資料檔案所組成。

- **python\_tutorial\_client.py**：與 Batch 和儲存體服務進行互動，以在計算節點 (虛擬機器) 上執行平行工作負載的用戶端應用程式。python\_tutorial\_client.py 指令碼會在本機工作站上執行。

- **python\_tutorial\_task.py**：在 Azure 中的計算節點上執行進而執行實際工作的指令碼。在範例中，python\_tutorial\_task.py 會剖析從 Azure 儲存體下載的檔案 (輸入檔) 中的文字。然後產生文字檔 (輸出檔)，其中包含出現在輸入檔中的前三個單字清單。在它建立輸出檔之後，python\_tutorial\_task.py 會將此檔案上傳至 Azure 儲存體。這讓檔案可供下載至您的工作站上執行的用戶端指令碼。python\_tutorial\_task.py 指令碼會在 Batch 服務中的多個計算節點上平行執行。

- **./data/taskdata*.txt**︰這三個文字檔會對在計算節點上執行的工作提供輸入。

下圖說明用戶端和工作指令碼所執行的主要作業。此基本工作流程通常由使用 Batch 建立的許多計算方案所組成。雖然不會示範 Batch 服務中可用的每項功能，但幾乎每個 Batch 案例都包含此工作流程的某些部分。

![Batch 範例工作流程][8]<br/>

[**步驟 1.**](#step-1-create-storage-containers) 在 Azure Blob 儲存體中建立**容器**。<br/> [**步驟 2.**](#step-2-upload-task-script-and-data-files) 將工作指令碼和輸入檔案上傳至容器。<br/> [**步驟 3.**](#step-3-create-batch-pool) 建立 Batch **集區**。<br/> &nbsp;&nbsp;&nbsp;&nbsp;**3a.** 集區 **StartTask** 會在節點加入集區時將工作指令碼 (python\_tutorial\_task.py) 下載至這些節點。<br/> [**步驟 4.**](#step-4-create-batch-job) 建立 Batch **作業**。<br/> [**步驟 5.**](#step-5-add-tasks-to-job) 將**工作**新增至作業。<br/> &nbsp;&nbsp;&nbsp;&nbsp;**5a.** 工作會排程在節點上執行。<br/> &nbsp;&nbsp;&nbsp;&nbsp;**5b.** 每項工作會從 Azure 儲存體下載其輸入資料，然後開始執行。<br/> [**步驟 6.**](#step-6-monitor-tasks) 監視工作。<br/> &nbsp;&nbsp;&nbsp;&nbsp;**6a.** 當工作完成時，它們會將其輸出資料上傳至 Azure 儲存體。<br/> [**步驟 7.**](#step-7-download-task-output) 從儲存體下載工作輸出。

如上所述，並非每個 Batch 方案都會執行這些確切步驟，並且可能包含更多步驟，但此範例會示範在 Batch 方案中找到的一般程序。

## 準備用戶端指令碼

執行範例之前，將您的 Batch 和儲存體帳戶認證加入至 python\_tutorial\_client.py。如果您尚未這麼做，請在您喜愛的編輯器中開啟此檔案，並使用您的認證更新下列程式碼行。

```python
# Update the Batch and Storage account credential strings below with the values
# unique to your accounts. These are used when constructing connection strings
# for the Batch and Storage client objects.

# Batch account credentials
batch_account_name = "";
batch_account_key  = "";
batch_account_url  = "";

# Storage account credentials
storage_account_name = "";
storage_account_key  = "";
```

您可以在 [Azure 入口網站][azure_portal]中每項服務的帳戶刀鋒視窗中尋找您的 Batch 和儲存體帳戶認證：

![入口網站中的 Batch 認證][9] ![入口網站中的 Storage 認證][10]<br/>

在下列各節中，我們會分析指令碼用來處理 Batch 服務中工作負載的步驟。鼓勵您在進行本文的其餘部分時，定期在編輯器中參考指令碼。

瀏覽至 **python\_tutorial\_client.py** 中的下列一行以開始進行步驟 1：

```python
if __name__ == '__main__':
```

## 步驟 1：建立儲存體容器

![在 Azure 儲存體中建立容器][1] <br/>

Batch 包含與 Azure 儲存體進行互動的內建支援。儲存體帳戶內的容器會提供在您的 Batch 帳戶中執行的工作所需的檔案。容器也會提供一個位置來儲存工作所產生的輸出資料。python\_tutorial\_client.py 首先會在 [Azure Blob 儲存體](../storage/storage-introduction.md#blob-storage)中建立三個容器：

- **應用程式**︰此容器會儲存工作所執行的 Python 指令碼 python\_tutorial\_task.py。
- **輸入**：工作會從「輸入」容器下載所要處理的資料檔案。
- **輸出**：當工作完成輸入檔案的處理時，它們會將結果上傳至「輸出」容器。

為了與儲存體帳戶進行互動並建立容器，我們使用 [azure-storage][pypi_storage] 封裝來建立 [BlockBlobService][py_blockblobservice] 物件 (Blob 用戶端)。 然後，我們使用 Blob 用戶端在儲存體帳戶中建立三個容器。

```python
 # Create the blob client, for use in obtaining references to
 # blob storage containers and uploading files to containers.
 blob_client = azureblob.BlockBlobService(
     account_name=_STORAGE_ACCOUNT_NAME,
     account_key=_STORAGE_ACCOUNT_KEY)

 # Use the blob client to create the containers in Azure Storage if they
 # don't yet exist.
 app_container_name = 'application'
 input_container_name = 'input'
 output_container_name = 'output'
 blob_client.create_container(app_container_name, fail_on_exist=False)
 blob_client.create_container(input_container_name, fail_on_exist=False)
 blob_client.create_container(output_container_name, fail_on_exist=False)
```

建立容器之後，應用程式現在即可上傳工作所要使用的檔案。

> [AZURE.TIP] [How to use Azure Blob storage from Python](../storage/storage-python-how-to-use-blob-storage.md) 提供使用 Azure 儲存體容器和 Blob 的概觀。當您開始使用 Batch 時，此概觀應在您的閱讀清單的頂端附近。

## 步驟 2：上傳工作指令碼和資料檔案

![將工作應用程式和輸入 (資料) 檔案上傳至容器][2] <br/>

在檔案上傳作業中，python\_tutorial\_client.py 會先定義**應用程式**和**輸入**檔案路徑的集合 (因為其存在於本機電腦上)。然後將這些檔案上傳到在上一個步驟中建立的容器。

```python
 # Paths to the task script. This script will be executed by the tasks that
 # run on the compute nodes.
 application_file_paths = [os.path.realpath('python_tutorial_task.py')]

 # The collection of data files that are to be processed by the tasks.
 input_file_paths = [os.path.realpath('./data/taskdata1.txt'),
                     os.path.realpath('./data/taskdata2.txt'),
                     os.path.realpath('./data/taskdata3.txt')]

 # Upload the application script to Azure Storage. This is the script that
 # will process the data files, and is executed by each of the tasks on the
 # compute nodes.
 application_files = [
     upload_file_to_container(blob_client, app_container_name, file_path)
     for file_path in application_file_paths]

 # Upload the data files. This is the data that will be processed by each of
 # the tasks executed on the compute nodes in the pool.
 input_files = [
     upload_file_to_container(blob_client, input_container_name, file_path)
     for file_path in input_file_paths]
```

使用清單理解功能，針對集合中的每個檔案呼叫 `upload_file_to_container` 函式並填入兩個 [ResourceFile][py_resource_file] 集合。`upload_file_to_container` 函式會如下所示：

```
def upload_file_to_container(block_blob_client, container_name, file_path):
    """
    Uploads a local file to an Azure Blob storage container.

    :param block_blob_client: A blob service client.
    :type block_blob_client: `azure.storage.blob.BlockBlobService`
    :param str container_name: The name of the Azure Blob storage container.
    :param str file_path: The local path to the file.
    :rtype: `azure.batch.models.ResourceFile`
    :return: A ResourceFile initialized with a SAS URL appropriate for Batch
    tasks.
    """
    blob_name = os.path.basename(file_path)

    print('Uploading file {} to container [{}]...'.format(file_path,
                                                          container_name))

    block_blob_client.create_blob_from_path(container_name,
                                            blob_name,
                                            file_path)

    sas_token = block_blob_client.generate_blob_shared_access_signature(
        container_name,
        blob_name,
        permission=azureblob.BlobPermissions.READ,
        expiry=datetime.datetime.utcnow() + datetime.timedelta(hours=2))

    sas_url = block_blob_client.make_blob_url(container_name,
                                              blob_name,
                                              sas_token=sas_token)

    return batchmodels.ResourceFile(file_path=blob_name,
                                    blob_source=sas_url)
```

### ResourceFiles

[ResourceFile][py_resource_file] 提供 Batch 中的工作，以及 Azure 儲存體中會在工作執行前下載到計算節點之檔案的 URL。[ResourceFile][py_resource_file].**blob\_source** 屬性會指定 Azure 儲存體中現有檔案的完整 URL。此 URL 也可能包含可供安全存取檔案的共用存取簽章 (SAS)。Batch 中的大部分工作類型都包含 ResourceFiles 屬性，包括：

- [CloudTask][py_task]
- [StartTask][py_starttask]
- [JobPreparationTask][py_jobpreptask]
- [JobReleaseTask][py_jobreltask]

此範例不會使用 JobPreparationTask 或 JobReleaseTask 工作類型，但您可以在[在 Azure Batch 計算節點上執行作業準備和完成工作](batch-job-prep-release.md)中深入了解。

### 共用存取簽章 (SAS)

共用存取簽章是可供安全存取 Azure 儲存體中容器和 Blob 的字串。python\_tutorial\_client.py 指令碼會使用 Blob 和容器共用存取簽章，並示範如何從儲存體服務取得這些共用存取簽章字串。

- **Blob 共用存取簽章**：集區的 StartTask 會在從儲存體下載工作指令碼和輸入資料檔案時，使用 Blob 共用存取簽章 (請參閱下面[步驟 3](#step-3-create-batch-pool))。python\_tutorial\_client.py 中的 `upload_file_to_container` 函式包含可取得各 Blob 共用存取簽章的程式碼。在儲存體模組中呼叫 [BlockBlobService.make\_blob\_url][py_make_blob_url] 即可完成。

- **容器共用存取簽章**：每個工作在計算節點上完成其工作時，便會將其輸出檔案上傳至 Azure 儲存體中的「輸出」容器。若要這樣做，python\_tutorial\_task.py 會使用容器共用存取簽章，其可提供容器的寫入存取權。python\_tutorial\_client.py 中的 `get_container_sas_token` 函式會取得容器的共用存取簽章，然後該簽章會以命令列引數的形式傳遞至工作。步驟 5 [將工作新增至作業](#step-5-add-tasks-to-job)討論容器 SAS 的使用方式。

> [AZURE.TIP] 查看有關共用存取簽章的兩部分系列[第 1 部分：了解 SAS 模型](../storage/storage-dotnet-shared-access-signature-part-1.md)和[第 2 部分：建立和使用 SAS 與 Blob 服務](../storage/storage-dotnet-shared-access-signature-part-2.md)，進一步了解如何提供您儲存體帳戶中資料的安全存取。

## 步驟 3：建立 Batch 集區

![建立 Batch 集區][3] <br/>

Batch **集區** 是 Batch 執行作業工作所在的計算節點 (虛擬機器) 集合。

將工作指令碼和資料檔案上傳至儲存體帳戶之後，python\_tutorial\_client.py 會使用 Batch Python 模組開始與 Batch 服務互動。為了這麼做，會建立 [BatchServiceClient][py_batchserviceclient]：

```python
 # Create a Batch service client. We'll now be interacting with the Batch
 # service in addition to Storage.
 credentials = batchauth.SharedKeyCredentials(_BATCH_ACCOUNT_NAME,
                                              _BATCH_ACCOUNT_KEY)

 batch_client = batch.BatchServiceClient(
     credentials,
     base_url=_BATCH_ACCOUNT_URL)
```

接下來，呼叫 `create_pool` 以在 Batch 帳戶中建立計算節點的集區。

```python
def create_pool(batch_service_client, pool_id,
                resource_files, publisher, offer, sku):
    """
    Creates a pool of compute nodes with the specified OS settings.

    :param batch_service_client: A Batch service client.
    :type batch_service_client: `azure.batch.BatchServiceClient`
    :param str pool_id: An ID for the new pool.
    :param list resource_files: A collection of resource files for the pool's
    start task.
    :param str publisher: Marketplace image publisher
    :param str offer: Marketplace image offer
    :param str sku: Marketplace image sku
    """
    print('Creating pool [{}]...'.format(pool_id))

    # Create a new pool of Linux compute nodes using an Azure Virtual Machines
    # Marketplace image. For more information about creating pools of Linux
    # nodes, see:
    # https://azure.microsoft.com/documentation/articles/batch-linux-nodes/

    # Specify the commands for the pool's start task. The start task is run
    # on each node as it joins the pool, and when it's rebooted or re-imaged.
    # We use the start task to prep the node for running our task script.
    task_commands = [
        # Copy the python_tutorial_task.py script to the "shared" directory
        # that all tasks that run on the node have access to.
        'cp -r $AZ_BATCH_TASK_WORKING_DIR/* $AZ_BATCH_NODE_SHARED_DIR',
        # Install pip and the dependencies for cryptography
        'apt-get update',
        'apt-get -y install python-pip',
        'apt-get -y install build-essential libssl-dev libffi-dev python-dev',
        # Install the azure-storage module so that the task script can access
        # Azure Blob storage
        'pip install azure-storage']

    # Get the node agent SKU and image reference for the virtual machine
    # configuration.
    # For more information about the virtual machine configuration, see:
    # https://azure.microsoft.com/documentation/articles/batch-linux-nodes/
    sku_to_use, image_ref_to_use = \
        common.helpers.select_latest_verified_vm_image_with_node_agent_sku(
            batch_service_client, publisher, offer, sku)

    new_pool = batch.models.PoolAddParameter(
        id=pool_id,
        virtual_machine_configuration=batchmodels.VirtualMachineConfiguration(
            image_reference=image_ref_to_use,
            node_agent_sku_id=sku_to_use),
        vm_size=_POOL_VM_SIZE,
        target_dedicated=_POOL_NODE_COUNT,
        start_task=batch.models.StartTask(
            command_line=
            common.helpers.wrap_commands_in_shell('linux', task_commands),
            run_elevated=True,
            wait_for_success=True,
            resource_files=resource_files),
        )

    try:
        batch_service_client.pool.add(new_pool)
    except batchmodels.batch_error.BatchErrorException as err:
        print_batch_exception(err)
        raise
```

當您建立集區時，您會定義 [PoolAddParameter][py_pooladdparam] 以指定集區的數個屬性：

- 集區的 [識別碼] (id - 必要)<p/>如同 Batch 中的大部分實體，新的集區必須具有 Batch 帳戶內的唯一識別碼。您的程式碼會使用其識別碼參考此集區，而這就是您在 Azure [入口網站][azure_portal]中識別集區的方式。

- **計算節點數目** (*target\_dedicated* - 必要)<p/>會指定應在集區中部署多少 VM。請務必注意，所有的 Batch 帳戶都具有預設**配額**，以限制 Batch 帳戶中的**核心** (因而限制計算節點) 數目。您可在 [Azure Batch 服務的配額和限制](batch-quota-limit.md)中發現預設配額以及如何[增加配額](batch-quota-limit.md#increase-a-quota) (例如 Batch 帳戶中的核心數目上限) 的說明。如果您發現自問「為什麼我的集區不會觸達 X 個以上的節點？」，此核心配額可能是原因。

- 節點的**作業系統** (virtual\_machine\_configuration **或** cloud\_service\_configuration - 必要)<p/>在 python\_tutorial\_client.py 中，我們會使用 [VirtualMachineConfiguration][py_vm_config] 來建立 Linux 節點的集區。`common.helpers`中的 `select_latest_verified_vm_image_with_node_agent_sku` 函式可簡化 [Azure 虛擬機器 Marketplace][vm_marketplace] 映像的使用方式。如需使用 Marketplace 映像的詳細資訊，請參閱[在 Azure Batch 集區中佈建 Linux 計算節點](batch-linux-nodes.md)。

- **計算節點的大小** (vm\_size - 必要)<p/>因為我們要針對 [VirtualMachineConfiguration][py_vm_config] 指定 Linux 節點，所以我們會從 [Azure 中的虛擬機器大小](../virtual-machines/virtual-machines-linux-sizes.md)指定 VM 大小 (在此範例中為 `STANDARD_A1`)。同樣地，如需詳細資訊，請參閱[在 Azure Batch 集區中佈建 Linux 計算節點](batch-linux-nodes.md)。

- **啟動工作** (start\_task - 非必要)<p/>透過上述實體節點屬性，您也可以指定集區的 [StartTask][py_starttask] (非必要)。StartTask 會在每個節點加入集區以及每次重新啟動節點時，於該節點上執行。StartTask 特別適合用於準備計算節點以便執行工作，例如安裝您的工作會執行的應用程式。<p/>在此範例應用程式中，StartTask 會將它從儲存體下載的檔案 (使用 StartTask 的 **resource\_files** 屬性所指定)，從 StartTask「工作目錄」複製到在節點上執行的所有工作可以存取的「共用」目錄。基本上，這會在節點加入集區時將 `python_tutorial_task.py` 複製到每個節點上的共用目錄，以便在節點上執行的任何工作都能存取它。

您可能會注意到對 `wrap_commands_in_shell` 協助程式函式的呼叫。此函式會採用不同命令的集合，並針對工作的命令列屬性建立合適的單一命令列。

此外，在上述程式碼片段中值得注意的是在 StartTask 的 **command\_line** 屬性中使用的兩個環境變數：`AZ_BATCH_TASK_WORKING_DIR` 和 `AZ_BATCH_NODE_SHARED_DIR`。Batch 集區中的每個計算節點都會自動以 Batch 特有的數個環境變數進行設定。工作所執行的任何程序都可以存取這些環境變數。

> [AZURE.TIP] 若要深入了解 Batch 集區中計算節點上可用的環境變數，以及有關工作的工作目錄資訊，請參閱 [Azure Batch 功能概觀](batch-api-basics.md)中的**工作的環境設定**和**檔案和目錄**。

## 步驟 4：建立 Batch 作業

![建立 Batch 作業][4]<br/>

Batch **作業**是與計算節點集區相關聯的工作集合。作業中的工作會在相關聯集區的計算節點上執行。

您不僅可使用作業來組織及追蹤相關工作負載中的工作，也可以強加特定條件約束，例如作業 (並延伸至其工作) 的最大執行階段，以及相對於 Batch 帳戶中其他作業的作業優先順序。不過，在此範例中，作業只與在步驟 3 建立的集區相關聯。不會設定任何其他屬性。

所有 Batch 作業都會與特定集區相關聯。此關聯表示會在哪些節點上執行作業的工作。您可使用 [PoolInformation][py_poolinfo] 屬性來指定此集區，如下列程式碼片段所示。

```python
def create_job(batch_service_client, job_id, pool_id):
    """
    Creates a job with the specified ID, associated with the specified pool.

    :param batch_service_client: A Batch service client.
    :type batch_service_client: `azure.batch.BatchServiceClient`
    :param str job_id: The ID for the job.
    :param str pool_id: The ID for the pool.
    """
    print('Creating job [{}]...'.format(job_id))

    job = batch.models.JobAddParameter(
        job_id,
        batch.models.PoolInformation(pool_id=pool_id))

    try:
        batch_service_client.job.add(job)
    except batchmodels.batch_error.BatchErrorException as err:
        print_batch_exception(err)
        raise
```

現已建立一個作業，便會加入工作來進行工作。

## 步驟 5：將工作加入至作業

![將工作加入至作業][5]<br/> (1) 工作已加入至作業，(2) 工作已排定在節點上執行，以及 (3) 工作會下載要處理的資料檔案

Batch **工作**是在計算節點上執行的個別工作單位。工作有一個命令列，可執行您在該命令列中指定的指令碼或可執行檔。

若要實際進行工作，必須將工作加入至作業。每個 [CloudTask][py_task] 都是透過命令列屬性以及工作在其命令列自動執行前下載至節點的 [ResourceFiles][py_resource_file] (如同集區的 StartTask) 進行設定。在此範例中，每個工作只會處理一個檔案。因此其 ResourceFiles 集合只包含單一元素。

```python
def add_tasks(batch_service_client, job_id, input_files,
              output_container_name, output_container_sas_token):
    """
    Adds a task for each input file in the collection to the specified job.

    :param batch_service_client: A Batch service client.
    :type batch_service_client: `azure.batch.BatchServiceClient`
    :param str job_id: The ID of the job to which to add the tasks.
    :param list input_files: A collection of input files. One task will be
     created for each input file.
    :param output_container_name: The ID of an Azure Blob storage container to
    which the tasks will upload their results.
    :param output_container_sas_token: A SAS token granting write access to
    the specified Azure Blob storage container.
    """

    print('Adding {} tasks to job [{}]...'.format(len(input_files), job_id))

    tasks = list()

    for input_file in input_files:

        command = ['python $AZ_BATCH_NODE_SHARED_DIR/python_tutorial_task.py '
                   '--filepath {} --numwords {} --storageaccount {} '
                   '--storagecontainer {} --sastoken "{}"'.format(
                    input_file.file_path,
                    '3',
                    _STORAGE_ACCOUNT_NAME,
                    output_container_name,
                    output_container_sas_token)]

        tasks.append(batch.models.TaskAddParameter(
                'topNtask{}'.format(input_files.index(input_file)),
                wrap_commands_in_shell('linux', command),
                resource_files=[input_file]
                )
        )

    batch_service_client.task.add_collection(job_id, tasks)
```

> [AZURE.IMPORTANT] 當工作存取環境變數 (例如 `$AZ_BATCH_NODE_SHARED_DIR`) 或執行在節點的 `PATH` 中找不到的應用程式時，工作命令列必須明確地叫用 Shell，例如透過 `/bin/sh -c MyTaskApplication $MY_ENV_VAR`。如果您的工作在節點的 `PATH` 中執行應用程式且未參考任何環境變數，這就不是必要條件。

在上述程式碼片段中的 `for` 迴圈內，您可以看到已建構工作的命令列，其中有五個命令列引數傳遞至 python\_tutorial\_task.py：

1. **filepath**：這是節點上現有檔案的本機路徑。在上述步驟 2 的 `upload_file_to_container` 中建立 ResourceFile 物件時，檔案名稱會用於此屬性 (ResourceFile 建構函式中的 `file_path` 參數)。這表示可以在節點上與 python\_tutorial\_task.py 相同的目錄中找到檔案。

2. **numwords**：最前面 N 個單字應該寫入輸出檔案。

3. **storageaccount**︰儲存體帳戶的名稱，其擁有工作輸出應上傳至的容器。

4. **storagecontainer**︰輸出檔應上傳至的儲存體容器名稱。

5. **sastoken**：共用存取簽章 (SAS)，可提供 Azure 儲存體中**輸出**容器的寫入存取權。Python\_tutorial\_task.py 指令碼會在建立其 BlockBlobService 參考時使用此共用存取簽章。這可提供容器的寫入存取權，而不需要儲存體帳戶的存取金鑰。

```python
# NOTE: Taken from python_tutorial_task.py

# Create the blob client using the container's SAS token.
# This allows us to create a client that provides write
# access only to the container.
blob_client = azureblob.BlockBlobService(account_name=args.storageaccount,
                                         sas_token=args.sastoken)
```

## 步驟 6：監視工作

![監視工作][6]<br/> *指令碼 (1) 會監視工作的完成狀態，以及 (2) 將結果資料上傳至 Azure 儲存體的工作*

工作新增至作業時，會自動排入佇列及排程，以便在與作業相關聯的集區中的計算節點上執行。根據您指定的設定，Batch 會為您處理所有工作佇列、排程、重試和其他工作管理責任。

監視工作執行的方法有許多種。python\_tutorial\_client.py 中的 `wait_for_tasks_to_complete` 函式會提供監視特定工作狀態的簡單範例，在此例中為[已完成][py_taskstate]狀態。

```python
def wait_for_tasks_to_complete(batch_service_client, job_id, timeout):
    """
    Returns when all tasks in the specified job reach the Completed state.

    :param batch_service_client: A Batch service client.
    :type batch_service_client: `azure.batch.BatchServiceClient`
    :param str job_id: The id of the job whose tasks should be to monitored.
    :param timedelta timeout: The duration to wait for task completion. If all
    tasks in the specified job do not reach Completed state within this time
    period, an exception will be raised.
    """
    timeout_expiration = datetime.datetime.now() + timeout

    print("Monitoring all tasks for 'Completed' state, timeout in {}..."
          .format(timeout), end='')

    while datetime.datetime.now() < timeout_expiration:
        print('.', end='')
        sys.stdout.flush()
        tasks = batch_service_client.task.list(job_id)

        incomplete_tasks = [task for task in tasks if
                            task.state != batchmodels.TaskState.completed]
        if not incomplete_tasks:
            print()
            return True
        else:
            time.sleep(1)

    print()
    raise RuntimeError("ERROR: Tasks did not reach 'Completed' state within "
                       "timeout period of " + str(timeout))
```

## 步驟 7：下載工作輸出

![從儲存體下載工作輸出][7]<br/>

現已完成作業，可以從 Azure 儲存體下載工作的輸出。在 python\_tutorial\_client.py 中呼叫 `download_blobs_from_container` 即可完成此操作：

```python
def download_blobs_from_container(block_blob_client,
                                  container_name, directory_path):
    """
    Downloads all blobs from the specified Azure Blob storage container.

    :param block_blob_client: A blob service client.
    :type block_blob_client: `azure.storage.blob.BlockBlobService`
    :param container_name: The Azure Blob storage container from which to
     download files.
    :param directory_path: The local directory to which to download the files.
    """
    print('Downloading all files from container [{}]...'.format(
        container_name))

    container_blobs = block_blob_client.list_blobs(container_name)

    for blob in container_blobs.items:
        destination_file_path = os.path.join(directory_path, blob.name)

        block_blob_client.get_blob_to_path(container_name,
                                           blob.name,
                                           destination_file_path)

        print('  Downloaded blob [{}] from container [{}] to {}'.format(
            blob.name,
            container_name,
            destination_file_path))

    print('  Download complete!')
```

> [AZURE.NOTE] 在 python\_tutorial\_client.py 中呼叫 `download_blobs_from_container`可指定檔案應下載到您的主目錄。您可隨意修改此輸出位置。

## 步驟 8：刪除容器

因為您需對位於 Azure 儲存體中的資料付費，所以建議您移除您的 Batch 作業不再需要的所有 Blob。在 python\_tutorial\_client.py 中，做法對 [BlockBlobService.delete\_container][py_delete_container] 進行三次呼叫：

```
# Clean up storage resources
print('Deleting containers...')
blob_client.delete_container(app_container_name)
blob_client.delete_container(input_container_name)
blob_client.delete_container(output_container_name)
```

## 步驟 9：刪除作業和集區

在最後一個步驟中，系統會提示您刪除 python\_tutorial\_client.py 指令碼所建立的作業和集區。雖然您不需支付作業和工作的費用，但您「需」支付計算節點的費用。因此，我們建議您只在必要時配置節點。刪除未使用的集區可成為您維護程序的一部分。

BatchServiceClient 的 [JobOperations][py_job] 和 [PoolOperations][py_pool] 兩者都有對應的刪除方法 (在您確認刪除時呼叫)：

```python
# Clean up Batch resources (if the user so chooses).
if query_yes_no('Delete job?') == 'yes':
    batch_client.job.delete(_JOB_ID)

if query_yes_no('Delete pool?') == 'yes':
    batch_client.pool.delete(_POOL_ID)
```

> [AZURE.IMPORTANT] 請記住，您需支付計算資源的費用，而刪除未使用的集區會將成本降到最低。另外請注意，刪除集區也會刪除該集區內的所有計算節點，而刪除集區後，將無法復原節點上的任何資料。

## 執行範例指令碼

當您執行教學課程[程式碼範例][github_article_samples]中的 python\_tutorial\_client.py 指令碼時，主控台輸出大致如下。在 `Monitoring all tasks for 'Completed' state, timeout in 0:20:00...` 會暫停執行，然而會建立、啟動集區的計算節點，以及執行集區的啟動工作中的命令。在執行期間和之後，使用 [Azure 入口網站][azure_portal]來監視集區、計算節點、作業和工作。使用 [Azure 入口網站][azure_portal]或 [Microsoft Azure 儲存體總管][storage_explorer]來檢視應用程式所建立的儲存體資源 (容器和 Blob)。

>[AZURE.TIP] 從 `azure-batch-samples/Python/Batch/article_samples` 目錄內執行 *python\_tutorial\_client.py* 指令碼。它會使用相對路徑來匯入 `common.helpers` 模組，因此如果您沒有從此目錄內執行指令碼，您可能會看到 `ImportError: No module named 'common'`。

以預設組態執行範例時，一般的執行時間**大約 5-7 分鐘**。

```
Sample start: 2016-05-20 22:47:10

Uploading file /home/user/py_tutorial/python_tutorial_task.py to container [application]...
Uploading file /home/user/py_tutorial/data/taskdata1.txt to container [input]...
Uploading file /home/user/py_tutorial/data/taskdata2.txt to container [input]...
Uploading file /home/user/py_tutorial/data/taskdata3.txt to container [input]...
Creating pool [PythonTutorialPool]...
Creating job [PythonTutorialJob]...
Adding 3 tasks to job [PythonTutorialJob]...
Monitoring all tasks for 'Completed' state, timeout in 0:20:00..........................................................................
  Success! All tasks reached the 'Completed' state within the specified timeout period.
Downloading all files from container [output]...
  Downloaded blob [taskdata1_OUTPUT.txt] from container [output] to /home/user/taskdata1_OUTPUT.txt
  Downloaded blob [taskdata2_OUTPUT.txt] from container [output] to /home/user/taskdata2_OUTPUT.txt
  Downloaded blob [taskdata3_OUTPUT.txt] from container [output] to /home/user/taskdata3_OUTPUT.txt
  Download complete!
Deleting containers...

Sample end: 2016-05-20 22:53:12
Elapsed time: 0:06:02

Delete job? [Y/n]
Delete pool? [Y/n]

Press ENTER to exit...
```

## 後續步驟

您可隨意變更 python\_tutorial\_client.py 和 python\_tutorial\_task.py，以試驗不同的計算案例。例如，嘗試將執行延遲新增至 python\_tutorial\_task.py，以模擬長時間執行的工作並在入口網站中監視這些工作。嘗試新增更多工作，或調整計算節點的數目。新增邏輯來檢查並允許使用現有的集區，以加速執行時間。

既然您已熟悉 Batch 方案的基本工作流程，現在可以深入了解 Batch 服務的其他功能。

- 如果您不熟悉這項服務，我們建議檢閱 [Azure Batch 功能概觀](batch-api-basics.md)一文。
- 從 [Batch 學習路徑][batch_learning_path]中的**深入開發**之下的其他 Batch 開發文章著手。
- 在 [TopNWords][github_topnwords] 範例中，查看利用 Batch 處理「前 N 個單字」工作負載的不同實作方式。

[azure_batch]: https://azure.microsoft.com/services/batch/
[azure_free_account]: https://azure.microsoft.com/free/
[azure_portal]: https://portal.azure.com
[batch_learning_path]: https://azure.microsoft.com/documentation/learning-paths/batch/
[blog_linux]: http://blogs.technet.com/b/windowshpc/archive/2016/03/30/introducing-linux-support-on-azure-batch.aspx
[crypto]: https://cryptography.io/en/latest/
[crypto_install]: https://cryptography.io/en/latest/installation/
[github_samples]: https://github.com/Azure/azure-batch-samples
[github_samples_zip]: https://github.com/Azure/azure-batch-samples/archive/master.zip
[github_topnwords]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/TopNWords
[github_article_samples]: https://github.com/Azure/azure-batch-samples/tree/master/Python/Batch/article_samples

[nuget_packagemgr]: https://visualstudiogallery.msdn.microsoft.com/27077b70-9dad-4c64-adcf-c7cf6bc9970c
[nuget_restore]: https://docs.nuget.org/consume/package-restore/msbuild-integrated#enabling-package-restore-during-build

[py_account_ops]: http://azure-sdk-for-python.readthedocs.org/en/latest/ref/azure.batch.operations.html#azure.batch.operations.AccountOperations
[py_azure_sdk]: https://pypi.python.org/pypi/azure
[py_batch_docs]: http://azure-sdk-for-python.readthedocs.org/en/latest/ref/azure.batch.html
[py_batch_package]: https://pypi.python.org/pypi/azure-batch
[py_batchserviceclient]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.html#azure.batch.BatchServiceClient
[py_blockblobservice]: http://azure.github.io/azure-storage-python/ref/azure.storage.blob.blockblobservice.html#azure.storage.blob.blockblobservice.BlockBlobService
[py_cloudtask]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.models.html#azure.batch.models.CloudTask
[py_computenodeuser]: http://azure-sdk-for-python.readthedocs.org/en/latest/ref/azure.batch.models.html#azure.batch.models.ComputeNodeUser
[py_cs_config]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.models.html#azure.batch.models.CloudServiceConfiguration
[py_delete_container]: http://azure.github.io/azure-storage-python/ref/azure.storage.blob.baseblobservice.html#azure.storage.blob.baseblobservice.BaseBlobService.delete_container
[py_gen_blob_sas]: http://azure.github.io/azure-storage-python/ref/azure.storage.blob.baseblobservice.html#azure.storage.blob.baseblobservice.BaseBlobService.generate_blob_shared_access_signature
[py_gen_container_sas]: http://azure.github.io/azure-storage-python/ref/azure.storage.blob.baseblobservice.html#azure.storage.blob.baseblobservice.BaseBlobService.generate_container_shared_access_signature
[py_image_ref]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.models.html#azure.batch.models.ImageReference
[py_imagereference]: http://azure-sdk-for-python.readthedocs.org/en/latest/ref/azure.batch.models.html#azure.batch.models.ImageReference
[py_job]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.operations.html#azure.batch.operations.JobOperations
[py_jobpreptask]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.models.html#azure.batch.models.JobPreparationTask
[py_jobreltask]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.models.html#azure.batch.models.JobReleaseTask
[py_list_skus]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.operations.html#azure.batch.operations.AccountOperations.list_node_agent_skus
[py_make_blob_url]: http://azure.github.io/azure-storage-python/ref/azure.storage.blob.baseblobservice.html#azure.storage.blob.baseblobservice.BaseBlobService.make_blob_url
[py_pool]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.operations.html#azure.batch.operations.PoolOperations
[py_pooladdparam]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.models.html#azure.batch.models.PoolAddParameter
[py_poolinfo]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.models.html#azure.batch.models.PoolInformation
[py_resource_file]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.models.html#azure.batch.models.ResourceFile
[py_samples_github]: https://github.com/Azure/azure-batch-samples/tree/master/Python/Batch/
[py_starttask]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.models.html#azure.batch.models.StartTask
[py_starttask]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.models.html#azure.batch.models.StartTask
[py_task]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.models.html#azure.batch.models.CloudTask
[py_taskstate]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.models.html#azure.batch.models.TaskState
[py_vm_config]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.models.html#azure.batch.models.VirtualMachineConfiguration
[pypi_batch]: https://pypi.python.org/pypi/azure-batch
[pypi_storage]: https://pypi.python.org/pypi/azure-storage

[pypi_install]: http://python-packaging-user-guide.readthedocs.io/en/latest/installing/
[storage_explorer]: http://storageexplorer.com/
[visual_studio]: https://www.visualstudio.com/products/vs-2015-product-editions
[vm_marketplace]: https://azure.microsoft.com/marketplace/virtual-machines/

[1]: ./media/batch-python-tutorial/batch_workflow_01_sm.png "在 Azure 儲存體中建立容器"
[2]: ./media/batch-python-tutorial/batch_workflow_02_sm.png "將工作應用程式和輸入 (資料) 檔案上傳至容器"
[3]: ./media/batch-python-tutorial/batch_workflow_03_sm.png "建立 Batch 集區"
[4]: ./media/batch-python-tutorial/batch_workflow_04_sm.png "建立 Batch 作業"
[5]: ./media/batch-python-tutorial/batch_workflow_05_sm.png "將工作加入至作業"
[6]: ./media/batch-python-tutorial/batch_workflow_06_sm.png "監視工作"
[7]: ./media/batch-python-tutorial/batch_workflow_07_sm.png "從儲存體下載工作輸出"
[8]: ./media/batch-python-tutorial/batch_workflow_sm.png "Batch 方案工作流程 (完整圖表)"
[9]: ./media/batch-python-tutorial/credentials_batch_sm.png "入口網站中的 Batch 認證"
[10]: ./media/batch-python-tutorial/credentials_storage_sm.png "入口網站中的儲存體認證"
[11]: ./media/batch-python-tutorial/batch_workflow_minimal_sm.png "Batch 方案工作流程 (最小圖表)"

<!---HONumber=AcomDC_0928_2016-->