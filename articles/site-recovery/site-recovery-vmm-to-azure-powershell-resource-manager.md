<properties
	pageTitle="使用 Azure Site Recovery 及 PowerShell (Resource Manager) 複寫 VMM 雲端中的 HYPER-V 虛擬機器 | Microsoft Azure"
	description="使用 Azure Site Recovery 及 PowerShell 複寫 VMM 雲端中的 HYPER-V 虛擬機器"
	services="site-recovery"
	documentationCenter=""
	authors="Rajani-Janaki-Ram"
	manager="rochakm"
	editor="raynew"/>

<tags
	ms.service="site-recovery"
	ms.workload="backup-recovery"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/16/2016"
	ms.author="rajanaki"/>

# 使用 PowerShell 和 Azure Resource Manager 將 Hyper-V 虛擬機器 (位於 VMM 雲端中) 複寫至 Azure

> [AZURE.SELECTOR]
- [Azure 入口網站](site-recovery-vmm-to-azure.md)
- [PowerShell - ARM](site-recovery-vmm-to-azure-powershell-resource-manager.md)
- [傳統入口網站](site-recovery-vmm-to-azure-classic.md)
- [PowerShell - 傳統](site-recovery-deploy-with-powershell.md)



## Overview

Azure Site Recovery 可在一些部署案例中協調虛擬機器的複寫、容錯移轉及復原，為您的商務持續性與災害復原 (BCDR) 做出貢獻。如需完整的部署案例清單，請參閱 [Azure Site Recovery 概觀](site-recovery-overview.md)。

本文說明如何使用 PowerShell 自動化您在設定 Azure Site Recovery 將 System Center VMM 雲端中的 HYPER-V 虛擬機器，複寫到 Azure 儲存體時常需要執行的工作。

本文包含案例的必要條件，並對您示範

- 如何設定復原服務保存庫
- 您會在來源 VMM 伺服器上安裝 Azure Site Recovery 提供者。
- 在保存庫中註冊伺服器、新增 Azure 儲存體帳戶
- 在 Hyper-V 主機伺服器上安裝 Azure 復原服務代理程式
- 設定將套用到所有受保護虛擬機器之 VMM 雲端的保護設定
- 為那些虛擬機器啟用保護。
- 測試容錯移轉，確認一切如預期般運作。

您在設定此案例如有任何問題，可將問題張貼到 [Azure 復原服務論壇](https://social.msdn.microsoft.com/forums/azure/home?forum=hypervrecovmgr)。

> [AZURE.NOTE] Azure 建立和處理資源的部署模型有二種：[Resource Manager 和傳統](../resource-manager-deployment-model.md)。本文涵蓋內容包括如何使用 Resource Manager 部署模型。

## 開始之前

確認您已備妥這些必要條件：

### Azure 必要條件

- 您將需要 [Microsoft Azure](https://azure.microsoft.com/) 帳戶。如果您沒有此帳戶，請先前往[免費帳戶](https://azure.microsoft.com/free)。此外，您可以參閱 [Azure Site Recovery 管理員價格](https://azure.microsoft.com/pricing/details/site-recovery/)。
- 如果您正在嘗試複寫至 CSP 訂用帳戶的案例，您將需要 CSP 訂用帳戶。若要深入了解 CSP 程式，請參閱[如何註冊 CSP 程式](https://msdn.microsoft.com/library/partnercenter/mt156995.aspx)。
- 您將需要 Azure v2 儲存體 (ARM) 帳戶才能將複寫的資料儲存至 Azure。此帳戶必須啟用異地複寫。它應該與 Azure Site Recovery 服務位於相同的區或，且與相同的訂用帳戶或 CSP 訂用帳戶關聯。若要深入了解設定 Azure 儲存體，請參閱 [Microsoft Azure 儲存體簡介](../storage/storage-introduction.md)。
- 您必須確定您要保護的虛擬機器符合 [Azure 虛擬機器必要條件](site-recovery-best-practices.md#azure-virtual-machine-requirements)。

> [AZURE.NOTE] 目前，只有 VM 層級作業可以通過 Powershell。即將推出對復原計劃層級作業的支援。現在，您僅限在「受保護 VM」資料粒度才能執行容錯移轉，而不能在復原計劃層級。

### VMM 必要條件
- 您將需要在在 System Center 2012 R2 上執行的 VMM 伺服器。
- 任何包含您想要保護之虛擬機器的 VMM 伺服器都必須執行 Azure Site Recovery 提供者。它會在部署 Azure Site Recovery 期間安裝。
- 在您想要保護的 VMM 伺服器上，您至少需要一個雲端。這個雲端應該包含：
	- 一或多個 VMM 主機群組。
	- 每個主機群組中的一或多個 Hyper-V 主機伺服器或叢集。
	- 來源 Hyper-V 伺服器上的一或多部虛擬機器。
- 深入了解設定 VMM 雲端：
	- 在[使用 System Center 2012 R2 VMM 的私人雲端新功能](http://go.microsoft.com/fwlink/?LinkId=324952)和 [VMM 2012 和雲端](http://go.microsoft.com/fwlink/?LinkId=324956)中進一步了解私人 VMM 雲端。
	- 深入了解[設定 VMM 雲端網狀架構](https://msdn.microsoft.com/library/azure/dn469075.aspx#BKMK_Fabric)
	- 在您備妥雲端網狀架構元素之後，[在 VMM 中建立私人雲端](http://go.microsoft.com/fwlink/p/?LinkId=324953)與[逐步解說：使用 System Center 2012 SP1 VMM 建立私人雲端](http://go.microsoft.com/fwlink/p/?LinkId=324954)中瞭解建立私人雲端。


### Hyper-V 的必要條件

- 主機 Hyper-V 伺服器至少必須執行附帶 Hyper-V 角色的 Windows Server 2012，並且已安裝最新更新。
- 如果您在叢集中執行 Hyper-V，請注意，如果您具有靜態 IP 位址叢集，並不會自動建立叢集代理。您必須手動設定叢集代理。如需
- 如需指示，請參閱[如何設定 Hyper-V 複本代理人](http://blogs.technet.com/b/haroldwong/archive/2013/03/27/server-virtualization-series-hyper-v-replica-broker-explained-part-15-of-20-by-yung-chou.aspx)。
- 您想要管理保護的任何 Hyper-V 主機伺服計或叢集都必須包含在 VMM 雲端中。

### 網路對應的必要條件
當您在 Azure 中保護虛擬機器，網路對應會對應來源 VMM 伺服器上的虛擬機器網路和目標 Azure 網路，以啟用下列項目：

- 在相同網路上容錯移轉的所有機器都可以彼此連接，無論它們隸屬於哪個復原計畫都一樣。
- 如果目標 Azure 網路上已設定網路閘道，則虛擬機器可以連接到其他內部部署虛擬機器。
- 如果您未設定網路對應，則只有在相同復原計畫中容錯移轉的虛擬機器才能在容錯移轉到 Azure 之後彼此連接。

如果您想要部署網路對應，您需要下列項目：

- 您想要在來源 VMM 伺服器上保護的虛擬機器應該連線到 VM 網路。該網路應該連結到與雲端相關聯的邏輯網路。
- 複寫的虛擬機器可在容錯移轉之後連線的 Azure 網路。您將在容錯移轉時選取此網路。此網路應該和您的 Azure Site Recovery 訂用帳戶在相同的區域中。

在下列內容中深入了解網路對應

- [如何在 VMM 中設定邏輯網路](http://go.microsoft.com/fwlink/p/?LinkId=386307)
- [如何在 VMM 中設定 VM 網路和閘道](http://go.microsoft.com/fwlink/p/?LinkId=386308)
- [如何在 Azure 中設定和監視虛擬網路](https://azure.microsoft.com/documentation/services/virtual-network/)


###PowerShell 必要條件
確定 Azure PowerShell 已經準備就緒。如果您已經使用 PowerShell，您必須升級至 0.8.10 版或更新版本。如需設定 PowerShell 的資訊，請參閱[安裝和設定 Azure PowerShell 指南](../powershell-install-configure.md)。一旦已安裝並設定 PowerShell，您可以檢視[這裡](https://msdn.microsoft.com/library/dn850420.aspx)之服務的所有可用的 Cmdlet。

如需了解可協助您使用這些 Cmdlet 的提示 (例如參數值、輸入及輸出在 Azure PowerShell 中的處理方式)，請參閱 [Azure Cmdlet 使用者入門](https://msdn.microsoft.com/library/azure/jj554332.aspx)。

## 步驟 1：設定訂用帳戶 

1. 從 Azure powershell，登入您的 Azure 帳戶︰使用下列 Cmdlet
 
		$UserName = "<user@live.com>"
		$Password = "<password>"
		$SecurePassword = ConvertTo-SecureString -AsPlainText $Password -Force
		$Cred = New-Object System.Management.Automation.PSCredential -ArgumentList $UserName, $SecurePassword
		Login-AzureRmAccount #-Credential $Cred 
	

2. 取得您的訂用帳戶清單。這樣也會列出每個訂用帳戶的 subscriptionID。記下您要建立復原服務保存庫之訂用帳戶的 subscriptionID

		Get-AzureRmSubscription 

3. 設定藉由提及訂用帳戶識別碼在其中建立復原服務保存庫的訂用帳戶

		Set-AzureRmContext –SubscriptionID <subscriptionId>


## 步驟 2：建立復原服務保存庫 

1. 如果您還沒有 ARM 資源群組，請加以建立

		New-AzureRmResourceGroup -Name #ResourceGroupName -Location #location

2. 建立新的復原服務保存庫，並儲存在變數 (稍後使用) 中建立的 ASR 保存庫物件。您也可以使用 Get-AzureRMRecoveryServicesVault Cmdlet 擷取建立後的 ASR 保存庫物件：-

		$vault = New-AzureRmRecoveryServicesVault -Name #vaultname -ResouceGroupName #ResourceGroupName -Location #location 

## 步驟 3：設定復原服務保存庫內容

1.  執行下面的命令來設定保存庫內容。

		Set-AzureRmSiteRecoveryVaultSettings -ARSVault $vault

## 步驟 4：安裝 Azure Site Recovery 提供者

1.	在 VMM 機器上，執行下列命令來建立目錄：
	
		New-Item c:\ASR -type directory
		
2.	執行下列命令，使用已下載的提供者將檔案解壓縮
	
		pushd C:\ASR\
		.\AzureSiteRecoveryProvider.exe /x:. /q

	
3.	使用下列命令安裝提供者：
	
		.\SetupDr.exe /i
		$installationRegPath = "hklm:\software\Microsoft\Microsoft System Center Virtual Machine Manager Server\DRAdapter"
		do
		{
		                $isNotInstalled = $true;
		                if(Test-Path $installationRegPath)
		                {
		                                $isNotInstalled = $false;
		                }
		}While($isNotInstalled)

    等待安裝完成。
	
4.	使用下列命令在保存庫中註冊伺服器：
	
		$BinPath = $env:SystemDrive+"\Program Files\Microsoft System Center 2012 R2\Virtual Machine Manager\bin"
		pushd $BinPath
		$encryptionFilePath = "C:\temp".\DRConfigurator.exe /r /Credentials $VaultSettingFilePath /vmmfriendlyname $env:COMPUTERNAME /dataencryptionenabled $encryptionFilePath /startvmmservice

	
## 步驟 5：建立 Azure 儲存體帳戶

1. 如果您沒有 Azure 儲存體帳戶，請執行下列命令，在相同地區建立啟用帳戶的異地複寫：

		$StorageAccountName = "teststorageacc1"	#StorageAccountname
		$StorageAccountGeo  = "Southeast Asia" 	
		$ResourceGroupName =  “myRG” 			#ResourceGroupName 
		$RecoveryStorageAccount = New-AzureRmStorageAccount -ResourceGroupName $ResourceGroupName -Name $StorageAccountName -Type “StandardGRS” -Location $StorageAccountGeo

請注意，儲存體帳戶必須與 Azure Site Recovery 服務位於相同的區或，且與相同的訂用帳戶關聯。

## 步驟 6：安裝 Azure 復原服務代理程式

1. 在 [http:/aka.ms/latestmarsagent](http://aka.ms/latestmarsagent) 下載 Azure 復原服務代理程式，並在 VMM 雲端中您要保護的每一個 Hyper-V 主機伺服器上將其安裝。

2. 在所有 VMM 主機上執行下列命令：

	marsagentinstaller.exe /q /nu

## 步驟 7：設定雲端保護設定

1.	執行下列命令來建立 Azure 的複寫原則：

	
		$ReplicationFrequencyInSeconds = "300";    	#options are 30,300,900
		$PolicyName = “replicapolicy”
		$Recoverypoints = 6            		#specify the number of recovery points 

		$policryresult = New-AzureRmSiteRecoveryPolicy -Name $policyname -ReplicationProvider HyperVReplicaAzure -ReplicationFrequencyInSeconds $replicationfrequencyinseconds -RecoveryPoints $recoverypoints -ApplicationConsistentSnapshotFrequencyInHours 1 -RecoveryAzureStorageAccountId "/subscriptions/q1345667/resourceGroups/test/providers/Microsoft.Storage/storageAccounts/teststorageacc1"

2.	執行下列命令以取得保護容器：
	
		$PrimaryCloud = "testcloud"
		$protectionContainer = Get-AzureRmSiteRecoveryProtectionContainer -friendlyName $PrimaryCloud;  
  
3.	取得原則詳細資料給使用已建立作業並且提及易記原則名稱的變數︰

		$policy = Get-AzureRmSiteRecoveryPolicy -FriendlyName $policyname

4.	啟動保護容器與複寫原則的關聯：

		$associationJob  = Start-AzureRmSiteRecoveryPolicyAssociationJob -Policy     $Policy -PrimaryProtectionContainer $protectionContainer  

5.	完成工作之後，執行下列命令：
   
		$job = Get-AzureRmSiteRecoveryJob -Job $associationJob
   		if($job -eq $null -or $job.StateDescription -ne "Completed")
   		 {
        $isJobLeftForProcessing = $true;
    	}

6.	完成工作處理之後，執行下列命令：

		if($isJobLeftForProcessing)
    	{
    	Start-Sleep -Seconds 60
    	}
        }While($isJobLeftForProcessing)

若要檢查作業是否完成，請執行[監視活動](#monitor)中的步驟。

## 步驟 8：設定網路對應

開始網路對應之前，請確認來源 VMM 伺服器上的虛擬機器已連線到 VM 網路。此外，請建立一或多個 Azure 虛擬網路。

在[使用 Azure Resource Manager 和 PowerShell 建立具有站對站 VPN 連線的虛擬網路](../vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md)中，深入了解如何使用 Azure Resource Manager 和 PowerShell 建立虛擬網路

請注意，多個虛擬機器網路可對應至單一 Azure 網路。如果目標網路具有多個子網路，且其中一個子網路的名稱和來源虛擬機器所在之子網路名稱相同，複本虛擬機器將會在容錯移轉之後連線到該目標子網路。如果沒有目標子網路具有相符的名稱，虛擬機器將會連線到網路中的第一個子網路。

1. 第一個命令會取得目前的 Azure Site Recovery 保存庫的伺服器。命令會將 Microsoft Azure Site Recovery 伺服器儲存在 $Servers 陣列變數。

		$Servers = Get-AzureRmSiteRecoveryServer

2. 第二個命令會取得 $Servers 陣列中第一部伺服器的站台復原網路。此命令會在 $Networks 變數中儲存網路。


    	$Networks = Get-AzureRmSiteRecoveryNetwork -Server $Servers[0]

3. 第三個命令會取得 Azure 虛擬網路，然後取得 $AzureVmNetworks 變數中的該值。
   
		$AzureVmNetworks =  Get-AzureRmVirtualNetwork

4. 最終的 Cmdlet 會在主要網路與 Azure 虛擬機器網路之間建立對應。Cmdlet 會將主要網路指定為 $Networks 的第一個元素。Cmdlet 會將虛擬機器網路指定為 $AzureVmNetworks 的第一個元素。

		New-AzureRmSiteRecoveryNetworkMapping -PrimaryNetwork $Networks[0] -AzureVMNetworkId $AzureVmNetworks[0]


## 步驟 9：對虛擬機器啟用保護

正確設定伺服器、雲端和網路後，您就可以對雲端中的虛擬機器啟用保護。

 請注意：

 - 虛擬機器必須符合 Azure 需求。請在《規劃指南》的[必要條件和支援](site-recovery-best-practices.md)中查看這些需求。

 - 若要啟用保護功能，您必須為虛擬機器設定作業系統和作業系統磁碟屬性。當您使用虛擬機器範本在 VMM 中建立虛擬機器時，您可以設定屬性。您也可以在虛擬機器屬性的 [一般] 及 [硬體設定] 索引標籤上，為現有的虛擬機器設定這些屬性。若未在 VMM 中設定這些屬性，您將可在 Azure 站台復原入口網站中加以設定。


  1. 若要啟用保護，請執行下列命令以取得保護容器：
	
			$ProtectionContainer = Get-AzureRmSiteRecoveryProtectionContainer -friendlyName $CloudName
	
  2. 執行下列命令以取得保護實體 (VM)：
	
	 		$protectionEntity = Get-AzureRmSiteRecoveryProtectionEntity -friendlyName $VMName -ProtectionContainer $protectionContainer
		
  3. 執行下列命令以啟用 VM 的 DR：

			$jobResult = Set-AzureRmSiteRecoveryProtectionEntity -ProtectionEntity $protectionentity -Protection Enable –Force -Policy $policy -RecoveryAzureStorageAccountId  $storageID "/subscriptions/217653172865hcvkchgvd/resourceGroups/rajanirgps/providers/Microsoft.Storage/storageAccounts/teststorageacc1



## 測試您的部署

若要測試部署，您可以對單一虛擬機器執行測試容錯移轉，或者建立包含多部虛擬機器的復原方案，再對這個方案執行測試容錯移轉。測試容錯移轉會在隔離的網路中模擬您的容錯移轉與復原機制。請注意：

- 如果您在容錯移轉之後要使用遠端桌面連接到 Azure 中的虛擬機器，請先在虛擬機器上啟用遠端桌面連線，再執行測試容錯移轉。
- 容錯移轉之後，您將透過遠端桌面使用公用 IP 位址連接到 Azure 中的虛擬機器。如果想要這樣做，請確定沒有任何網域原則禁止您使用公用位址連接到虛擬機器。

若要檢查作業是否完成，請執行[監視活動](#monitor)中的步驟。


### 執行測試容錯移轉

1.	執行下列命令來啟動測試容錯移轉：
	
		$protectionEntity = Get-AzureRmSiteRecoveryProtectionEntity -Name $VMName -ProtectionContainer $protectionContainer

		$jobIDResult =  Start-AzureRmSiteRecoveryTestFailoverJob -Direction PrimaryToRecovery -ProtectionEntity $protectionEntity -AzureVMNetworkId <string>  

### 執行計劃性容錯移轉

1. 執行下列命令來啟動計劃性容錯移轉：
	
		$protectionEntity = Get-AzureRmSiteRecoveryProtectionEntity -Name $VMName -ProtectionContainer $protectionContainer

		$jobIDResult =  Start-AzureRmSiteRecoveryPlannedFailoverJob -Direction PrimaryToRecovery -ProtectionEntity $protectionEntity -AzureVMNetworkId <string>  

### 執行非計劃性容錯移轉

1. 執行下列命令來啟動非計劃性容錯移轉：
		
		$protectionEntity = Get-AzureRmSiteRecoveryProtectionEntity -Name $VMName -ProtectionContainer $protectionContainer

		$jobIDResult =  Start-AzureRmSiteRecoveryUnPlannedFailoverJob -Direction PrimaryToRecovery -ProtectionEntity $protectionEntity -AzureVMNetworkId <string>  

	
## <a name=monitor></a>監視活動

使用下列命令來監視活動。請注意，您必須在工作之間等候處理程序完成。

	Do
	{
        $job = Get-AzureSiteRecoveryJob -Id $associationJob.JobId;
        Write-Host "Job State:{0}, StateDescription:{1}" -f Job.State, $job.StateDescription;
        if($job -eq $null -or $job.StateDescription -ne "Completed")
        {
        	$isJobLeftForProcessing = $true;
        }

	if($isJobLeftForProcessing)
        {
        	Start-Sleep -Seconds 60
        }
	}While($isJobLeftForProcessing)



## 後續步驟

[閱讀更多](https://msdn.microsoft.com/library/azure/mt637930.aspx) Azure Site Recovery PowerShell Cmdlet 的相關資訊。

<!---HONumber=AcomDC_0921_2016-->