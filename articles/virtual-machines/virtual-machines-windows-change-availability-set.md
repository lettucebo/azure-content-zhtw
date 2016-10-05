<properties
	pageTitle="變更 VM 可用性設定組 | Microsoft Azure"
	description="了解如何使用 Azure PowerShell 和 Resource Manager 部署模型來變更虛擬機器的可用性設定組。"
	keywords=""
	services="virtual-machines-windows"
	documentationCenter=""
	authors="Drewm3"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>
<tags
	ms.service="virtual-machines-windows"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/15/2016"
	ms.author="drewm"/>



# 變更 Windows VM 的可用性設定組

下列步驟說明如何使用 Azure PowerShell 來變更 VM 的可用性設定組。只有在建立 VM 時，才能將 VM 新增到可用性設定組中。若要變更可用性設定組，您必須將虛擬機器刪除後再重新建立。

## 使用 PowerShell 來變更可用性設定組

1. 從要修改的 VM 中擷取下列主要詳細資料。

	VM 的名稱
	
	```powershell
	$vm = Get-AzureRmVM -ResourceGroupName <Name-of-resource-group> -Name <name-of-VM>
	$vm.Name
	```
 
	VM 大小
	
	```powershell
	$vm.HardwareProfile.VmSize
	```

	網路主要網路介面和選擇性網路介面 (如果它們存在於 VM 上)
	
	```powershell
	$vm.NetworkProfile.NetworkInterfaces[0].Id
	```

	OS 磁碟設定檔

	```powershell
	$vm.StorageProfile.OsDisk.OsType
	$vm.StorageProfile.OsDisk.Name
	$vm.StorageProfile.OsDisk.Vhd.Uri
	```

	每個資料磁碟的磁碟設定檔
	
	```powershell
	$vm.StorageProfile.DataDisks[<index>].Lun
	$vm.StorageProfile.DataDisks[<index>].Vhd.Uri
	```

	已安裝的 VM 延伸模組
	
	```powershell
	$vm.Extensions
	```

2. 刪除 VM 而不刪除任何磁碟或網路介面。

	```powershell
	Remove-AzureRmVM -ResourceGroupName <resourceGroupName> -Name <vmName> 
	```

3. 建立可用性設定組 (如果可用性設定組尚未存在)

	```powershell
	New-AzureRmAvailabilitySet -ResourceGroupName <resourceGroupName> -Name <availabilitySetName> -Location "<location>" 
	```

4. 使用新的可用性設定組來重新建立 VM

	```powershell
	$vm2 = New-AzureRmVMConfig -VMName <VM-name> -VMSize <vm-size> -AvailabilitySetId <availability-set-id>

	Set-AzureRmVMOSDisk -CreateOption "Attach" -VM <vmConfig> -VhdUri <osDiskURI> -Name <osDiskName> [-Windows | -Linux]

	Add-AzureRmVMNetworkInterface -VM <vmConfig> -Id  <nicId> 

	New-AzureRmVM -ResourceGroupName <resourceGroupName> -Location <location> -VM <vmConfig>
	``` 

5. 新增資料磁碟和延伸模組。如需詳細資訊，請參閱[將資料磁碟連結到 VM](virtual-machines-windows-attach-disk-portal.md) 和[延伸模組組態範例](virtual-machines-windows-extensions-configuration-samples.md)。您可以使用 PowerShell 或 Azure CLI 將資料磁碟和延伸模組新增到 VM 中。

## 範例指令碼

下列指令碼提供一個範例，此範例會收集必要資訊、刪除原始 VM，然後在新的可用性設定組中重新建立 VM。

```powershell
	#set variables
	$rg = "demo-resource-group"
	$vmName = "demo-vm"
	$newAvailSetName = "demo-as"
	$outFile = "C:\temp\outfile.txt"

	#Get VM Details
	$OriginalVM = get-azurermvm -ResourceGroupName $rg -Name $vmName

	#Output VM details to file
	"VM Name: " | Out-File -FilePath $outFile 
	$OriginalVM.Name | Out-File -FilePath $outFile -Append

	"Extensions: " | Out-File -FilePath $outFile -Append
	$OriginalVM.Extensions | Out-File -FilePath $outFile -Append

	"VMSize: " | Out-File -FilePath $outFile -Append
	$OriginalVM.HardwareProfile.VmSize | Out-File -FilePath $outFile -Append

	"NIC: " | Out-File -FilePath $outFile -Append
	$OriginalVM.NetworkProfile.NetworkInterfaces[0].Id | Out-File -FilePath $outFile -Append

	"OSType: " | Out-File -FilePath $outFile -Append
	$OriginalVM.StorageProfile.OsDisk.OsType | Out-File -FilePath $outFile -Append

	"OS Disk: " | Out-File -FilePath $outFile -Append
	$OriginalVM.StorageProfile.OsDisk.Vhd.Uri | Out-File -FilePath $outFile -Append

	if ($OriginalVM.StorageProfile.DataDisks) {
    "Data Disk(s): " | Out-File -FilePath $outFile -Append
    $OriginalVM.StorageProfile.DataDisks | Out-File -FilePath $outFile -Append
	}

	#Remove the original VM
	Remove-AzureRmVM -ResourceGroupName $rg -Name $vmName

	#Create new availability set if it does not exist
	$availSet = Get-AzureRmAvailabilitySet -ResourceGroupName $rg -Name $newAvailSetName -ErrorAction Ignore
	if (-Not $availSet) {
    $availset = New-AzureRmAvailabilitySet -ResourceGroupName $rg -Name $newAvailSetName -Location $OriginalVM.Location
	}

	#Create the basic configuration for the replacement VM
	$newVM = New-AzureRmVMConfig -VMName $OriginalVM.Name -VMSize $OriginalVM.HardwareProfile.VmSize -AvailabilitySetId $availSet.Id
	Set-AzureRmVMOSDisk -VM $NewVM -VhdUri $OriginalVM.StorageProfile.OsDisk.Vhd.Uri  -Name $OriginalVM.Name -CreateOption Attach -Windows

	#Add Data Disks
	foreach ($disk in $OriginalVM.StorageProfile.DataDisks ) { 
    Add-AzureRmVMDataDisk -VM $newVM -Name $disk.Name -VhdUri $disk.Vhd.Uri -Caching $disk.Caching -Lun $disk.Lun -CreateOption Attach -DiskSizeInGB $disk.DiskSizeGB
	}

	#Add NIC(s)
	foreach ($nic in $OriginalVM.NetworkInterfaceIDs) {
		Add-AzureRmVMNetworkInterface -VM $NewVM -Id $nic
	}

	#Create the VM
	New-AzureRmVM -ResourceGroupName $rg -Location $OriginalVM.Location -VM $NewVM -DisableBginfoExtension
```

## 後續步驟

透過新增額外的[資料磁碟](virtual-machines-windows-attach-disk-portal.md)，將額外的儲存體新增到您的 VM。

<!---HONumber=AcomDC_0921_2016-->