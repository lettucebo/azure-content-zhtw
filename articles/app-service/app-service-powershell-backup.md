<properties
	pageTitle="使用 PowerShell 來備份及還原 App Service 應用程式"
	description="了解如何使用 PowerShell 來備份和還原 Azure App Service 中的應用程式"
	services="app-service"
	documentationCenter=""
	authors="NKing92"
	manager="wpickett"
    editor="" />

<tags
	ms.service="app-service"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/10/2016"
	ms.author="nicking"/>
# 使用 PowerShell 來備份及還原 App Service 應用程式

> [AZURE.SELECTOR]
- [PowerShell](app-service-powershell-backup.md)
- [REST API](../app-service-web/websites-csm-backup.md)

了解如何使用 Azure PowerShell 來備份及還原 [App Service 應用程式](https://azure.microsoft.com/services/app-service/web/)。如需有關 Web 應用程式備份的詳細資訊，包括需求和限制，請參閱[在 Azure App Service 中備份 Web 應用程式](../app-service-web/web-sites-backup.md)。

## 必要條件
若要使用 PowerShell 來管理您的應用程式備份，您需要下列項目︰

- **SAS URL**，提供 Azure 儲存體容器的讀取和寫入存取權。如需 SAS URL 的說明，請參閱[了解 SAS 模型](../storage/storage-dotnet-shared-access-signature-part-1.md)。如需使用 PowerShell 管理 Azure 儲存體的範例，請參閱[搭配使用 Azure PowerShell 與 Azure 儲存體](../storage/storage-powershell-guide-full.md)。
- **資料庫連接字串**，如果您想要備份資料庫和 Web 應用程式。

### 如何產生 SAS URL 以搭配 Web 應用程式備份 Cmdlet 使用
您可以使用 PowerShell 產生 SAS URL。以下是如何產生一個 SAS URL 以搭配本文所討論的 Cmdlet 使用的範例。

		$storageAccountName = "<your storage account's name>"
		$storageAccountRg = "<your storage account's resource group>"

		# This returns an array of keys for your storage account. Be sure to select the appropriate key. Here we select the first key as a default.
		$storageAccountKey = Get-AzureRmStorageAccountKey -ResourceGroupName $storageAccountRg -Name $storageAccountName
		$context = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey[0].Value

		$blobContainerName = "<name of blob container for app backups>"
		$sasUrl = New-AzureStorageContainerSASToken -Name $blobContainerName -Permission rwdl -Context $context -ExpiryTime (Get-Date).AddMonths(1) -FullUri

## 安裝 Azure PowerShell 1.3.2 或更新版本

如需安裝和使用 Azure PowerShell 的指示，請參閱[搭配使用 Azure PowerShell 與 Azure Resource Manager](../powershell-install-configure.md)。

## 建立備份

使用 New-AzureRmWebAppBackup Cmdlet 來建立 Web 應用程式的備份。

		$sasUrl = "<your SAS URL>"
		$resourceGroupName = "Default-Web-WestUS"
		$appName = "ContosoApp"

		$backup = New-AzureRmWebAppBackup -ResourceGroupName $resourceGroupName -Name $appName -StorageAccountUrl $sasUrl

這會以自動產生的名稱建立備份。如果您要為您的備份提供名稱，請使用 BackupName 選用參數。

		$backup = New-AzureRmWebAppBackup -ResourceGroupName $resourceGroupName -Name $appName -StorageAccountUrl $sasUrl -BackupName MyBackup

若要將資料庫包含在備份中，請先使用 New-AzureRmWebAppDatabaseBackupSetting Cmdlet 建立資料庫備份設定，然後在 New-AzureRmWebAppBackup Cmdlet 的 Databases 參數中提供該設定。Databases 參數可接受資料庫設定陣列，讓您備份一個以上的資料庫。

		$dbSetting1 = New-AzureRmWebAppDatabaseBackupSetting -Name DB1 -DatabaseType SqlAzure -ConnectionString "<connection_string>"
		$dbSetting2 = New-AzureRmWebAppDatabaseBackupSetting -Name DB2 -DatabaseType SqlAzure -ConnectionString "<connection_string>"
		$dbBackup = New-AzureRmWebAppBackup -ResourceGroupName $resourceGroupName -Name $appName -BackupName MyBackup -StorageAccountUrl $sasUrl -Databases $dbSetting1,$dbSetting2

## 取得備份

Get-AzureRmWebAppBackupList Cmdlet 會傳回 Web 應用程式的所有備份陣列。您必須提供 Web 應用程式與其資源群組的名稱。

		$resourceGroupName = "Default-Web-WestUS"
		$appName = "ContosoApp"
		$backups = Get-AzureRmWebAppBackupList -Name $appName -ResourceGroupName $resourceGroupName

若要取得特定備份，請使用 Get-AzureRmWebAppBackup Cmdlet。

		$backup = Get-AzureRmWebAppBackup -Name $appName -ResourceGroupName $resourceGroupName -BackupId 10102

為了方便，您也可以使用管線將 Web 應用程式物件傳送至任何備份管理 Cmdlet。

		$app = Get-AzureRmWebApp -Name ContosoApp -ResourceGroupName Default-Web-WestUS
		$backupList = $app | Get-AzureRmWebAppBackupList
		$backup = $app | Get-AzureRmWebAppBackup -BackupId 10102

## 排程自動備份

您可以排程備份以在指定的間隔自動進行。若要設定備份排程，請使用 Edit-AzureRmWebAppBackupConfiguration Cmdlet。此 Cmdlet 會採用數個參數︰

- **Name** - Web 應用程式的名稱。
- **ResourceGroupName** - 包含 Web 應用程式之資源群組的名稱。
- **Slot** - 選用。Web 應用程式位置的名稱。
- **StorageAccountUrl** - 用來儲存備份的 Azure 儲存體容器的 SAS URL。
- **FrequencyInterval** - 應多久進行一次備份的數值。必須是正整數。
- **FrequencyUnit** - 應多久進行一次備份的時間單位。選項為 [小時] 和 [天]。
- **RetentionPeriodInDays** - 在自動刪除自動備份前所應儲存的天數。
- **StartTime** - 選用。應開始自動備份的時間。如果這是 null，將立即開始備份。必須是 DateTime。
- **Databases** - 選用。要備份資料庫的 DatabaseBackupSettings 陣列。
- **KeepAtLeastOneBackup** - 選擇性切換參數。如有一個備份應該永遠留在儲存體帳戶中 (無論有多舊)，請提供此參數值。

以下是如何使用此 Cmdlet 的範例。

		$resourceGroupName = "Default-Web-WestUS"
		$appName = "ContosoApp"
		$slotName = "StagingSlot"
		$dbSetting1 = New-AzureRmWebAppDatabaseBackupSetting -Name DB1 -DatabaseType SqlAzure -ConnectionString "<connection_string>"
		$dbSetting2 = New-AzureRmWebAppDatabaseBackupSetting -Name DB2 -DatabaseType SqlAzure -ConnectionString "<connection_string>"
		Edit-AzureRmWebAppBackupConfiguration -Name $appName -ResourceGroupName $resourceGroupName -Slot $slotName `
		  -StorageAccountUrl "<your SAS URL>" -FrequencyInterval 6 -FrequencyUnit Hour -Databases $dbSetting1,$dbSetting2 `
		  -KeepAtLeastOneBackup -StartTime (Get-Date).AddHours(1)

若要取得目前的備份排程，請使用 Get-AzureRmWebAppBackupConfiguration Cmdlet。這很適合用於修改已設定的排程。

		$configuration = Get-AzureRmWebAppBackupConfiguration -Name $appName -ResourceGroupName $resourceGroupName

		# Modify the configuration slightly
		$configuration.FrequencyInterval = 2
		$configuration.FrequencyUnit = "Day"

		# Apply the new configuration by piping it into the Edit-AzureRmWebAppBackupConfiguration cmdlet
		$configuration | Edit-AzureRmWebAppBackupConfiguration

## 從備份還原 Web 應用程式

若要從備份中還原 Web 應用程式，請使用 Restore-AzureRmWebAppBackup Cmdlet。使用這個 Cmdlet 的最簡單方式就是透過管線傳入從 Get-AzureRmWebAppBackup Cmdlet 或 Get-AzureRmWebAppBackupList Cmdlet 擷取的備份物件。

一旦有備份物件，即可透過管線將該物件送至 Restore-AzureRmWebAppBackup Cmdlet。指定 Overwrite 切換參數，以指出您打算以備份的內容覆寫 Web 應用程式的內容。如果備份包含資料庫，也將會還原這些資料庫。

		$backup | Restore-AzureRmWebAppBackup -Overwrite

以下是如何藉由指定所有參數來使用 Restore-AzureRmWebAppBackup 的範例。

		$resourceGroupName = "Default-Web-WestUS"
		$appName = "ContosoApp"
		$slotName = "StagingSlot"
		$blobName = "ContosoBackup.zip"
		$dbSetting1 = New-AzureRmWebAppDatabaseBackupSetting -Name DB1 -DatabaseType SqlAzure -ConnectionString "<connection_string>"
		$dbSetting2 = New-AzureRmWebAppDatabaseBackupSetting -Name DB2 -DatabaseType SqlAzure -ConnectionString "<connection_string>"
		Restore-AzureRmWebAppBackup -ResourceGroupName $resourceGroupName -Name $appName -Slot $slotName -StorageAccountUrl "<your SAS URL>" -BlobName $blobName -Databases $dbSetting1,$dbSetting2 -Overwrite

## 刪除備份

若要刪除備份，請使用 Remove-AzureRmWebAppBackup Cmdlet。這會從您的儲存體帳戶移除備份。指定您的應用程式名稱、其資源群組和您想要刪除之備份的識別碼。

		$resourceGroupName = "Default-Web-WestUS"
		$appName = "ContosoApp"
		Remove-AzureRmWebAppBackup -ResourceGroupName $resourceGroupName -Name $appName -BackupId 10102

您也可以透過管線將備份物件送至 Remove-AzureRmWebAppBackup Cmdlet 加以刪除。

		$backup = Get-AzureRmWebAppBackup -Name $appName -ResourceGroupName $resourceGroupName -BackupId 10102
		$backup | Remove-AzureRmWebAppBackup -Overwrite

<!---HONumber=AcomDC_0921_2016-->