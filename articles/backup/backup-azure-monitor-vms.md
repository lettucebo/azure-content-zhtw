<properties
   pageTitle="監視 Resource Manager 部署的虛擬機器備份 | Microsoft Azure"
   description="監視 Resource Manager 部署的虛擬機器備份中的事件和警示。根據警示傳送電子郵件。"
   services="backup"
   documentationCenter="dev-center-name"
   authors="markgalioto"
   manager="cfreeman"
   editor=""/>

<tags
ms.service="backup"
ms.workload="storage-backup-recovery"
ms.tgt_pltfrm="na"
ms.devlang="na"
ms.topic="article"
ms.date="08/25/2016"
ms.author="trinadhk; giridham;"/>

# 監視 Azure 虛擬機器備份的警示

警示是來自已達到或超過事件閾值之服務的回應。了解何時出現問題，可能對於維持低商務成本很重要。警示通常不會依照排程發生，因此在警示發生之後盡早得知將會有所幫助。例如，當備份或還原作業失敗時，警示會在失敗後五分鐘內發生。在保存庫儀表板中，[備份警示] 圖格會顯示嚴重和警告層級的事件。在 [備份警示] 設定中，您可以檢視所有事件。但是，如果在您處理不同問題時發生警示，您該怎麼辦？ 如果您不知道何時發生警示，可能會有點不便，或可能危及資料。若要確定正確的人員會留意警示 (當它發生時)，請設定服務以透過電子郵件傳送警示通知。如需設定電子郵件通知的詳細資訊，請參閱[設定通知](backup-azure-monitor-vms.md#configure-notifications)。

## 如何找到警示的相關資訊？

若要檢視擲回警示的事件相關資訊，您必須開啟 [備份警示] 刀鋒視窗。開啟 [備份警示] 刀鋒視窗的方法有兩種︰從保存庫儀表板中的 [備份警示] 圖格，或從 [警示和事件] 刀鋒視窗。

若要從 [備份警示] 圖格開啟 [備份警示] 刀鋒視窗︰

- 在保存庫儀表板的 [備份警示] 圖格上，按一下 [嚴重] 或 [警告] 以檢視該嚴重性層級的作業事件。

    ![備份警示圖格](./media/backup-azure-monitor-vms/backup-alerts-tile.png)


若要從 [警示和事件] 刀鋒視窗開啟 [備份警示] 刀鋒視窗︰

1. 在保存庫儀表板中，按一下 [所有設定]。![所有設定按鈕](./media/backup-azure-monitor-vms/all-settings-button.png)

2. 在 [設定] 刀鋒視窗上，按一下 [警示和事件]。![警示和事件按鈕](./media/backup-azure-monitor-vms/alerts-and-events-button.png)

3. 在 [警示與事件] 刀鋒視窗上，按一下 [備份警示]。![備份警示按鈕](./media/backup-azure-monitor-vms/backup-alerts.png)

    [備份警示] 刀鋒視窗會開啟並顯示篩選後的警示。

    ![備份警示圖格](./media/backup-azure-monitor-vms/backup-alerts-critical.png)

4. 若要檢視特定警示的詳細資訊，請從事件清單中按一下警示，以開啟其 [詳細資料] 刀鋒視窗。

    ![事件詳細資料](./media/backup-azure-monitor-vms/audit-logs-event-detail.png)

    若要自訂清單中顯示的屬性，請參閱[檢視其他事件屬性](backup-azure-monitor-vms.md#view-additional-event-attributes)

## 設定通知

 您可以設定服務以針對過去一小時發生的警示，或在特定類型的事件發生時，傳送電子郵件通知。

設定警示的電子郵件通知

1. 在 [備份警示] 功能表上，按一下 [設定通知]

    ![備份警示功能表](./media/backup-azure-monitor-vms/backup-alerts-menu.png)

    [設定通知] 刀鋒視窗隨即開啟。

    ![設定通知刀鋒視窗](./media/backup-azure-monitor-vms/configure-notifications.png)

2. 在 [設定通知] 刀鋒視窗上，針對 [電子郵件通知]，按一下 [開啟]。

    [收件者] 和 [嚴重性] 對話方塊旁邊有星號，因為該資訊是必要的。提供至少一個電子郵件地址，然後選取至少一個嚴重性。

3. 在 [收件者 (電子郵件)] 對話方塊中，輸入接收通知者的電子郵件地址。使用格式︰username@domainname.com。用分號分 (;) 分隔多個電子郵件位址。

4. 在 [通知] 區域中，選擇 [每個警示] 以在指定的警示發生時傳送通知，或選擇 [每小時摘要] 以傳送過去一小時的摘要。

5. 在 [嚴重性] 對話方塊中，選擇您要觸發電子郵件通知的一或多個層級。

6. 按一下 [儲存]。
### 有哪些警示類型可供 Azure IaaS VM 備份使用？
| 警示層級 | 傳送的警示 |
| ------------- | ------------- |
| 重要 | 備份失敗、復原失敗 |
| 警告 | None |
| 資訊 | None | 

### 會有即使已設定通知卻不寄送電子郵件的情況嗎？

即使已正確設定通知，仍會有不寄送警示的情況。在下列情況下將不會寄送電子郵件通知，以避免警示雜訊︰

- 如果通知設定為 [每小時摘要]，而且在一小時內引發警示並加以解決，
- 作業便會取消。
- 備份作業會觸發然後失敗，且另一個備份作業正在進行中。
- 啟用 Resource Manager 功能之 VM 的排程備份作業會啟動，但 VM 將不再存在。

## 自訂事件的檢視

[稽核記錄檔] 設定隨附一組預先定義的篩選器和資料行，以顯示作業事件資訊。您可以自訂檢視，因此當 [事件] 刀鋒視窗開啟時，它會顯示您所需的資訊。

1. 在[保存庫儀表板](./backup-azure-manage-vms.md#open-a-recovery-services-vault-in-the-dashboard)中，瀏覽並按一下 [稽核記錄檔]，以開啟 [事件] 刀鋒視窗。

    ![稽核記錄檔](./media/backup-azure-monitor-vms/audit-logs-1606-1.png)

    開啟的 [事件] 刀鋒視窗會顯示針對目前保存庫而篩選的作業事件。

    ![稽核記錄檔篩選器](./media/backup-azure-monitor-vms/audit-logs-filter.png)

    此刀鋒視窗會顯示過去一週發生的嚴重、錯誤、警告和資訊事件清單。時間範圍是在 [篩選] 中設定的預設值。[事件] 刀鋒視窗也會顯示橫條圖來追蹤事件發生的時間。如果您不想看到橫條圖，請在 [事件] 功能表中按一下 [隱藏圖表] 以關閉圖表。[事件] 的預設檢視會顯示 [作業]、[層級]、[狀態]、[資源] 和 [時間] 資訊。如需公開其他事件屬性的相關資訊，請參閱[展開事件資訊](backup-azure-monitor-vms.md#view-additional-event-attributes)一節。

2. 如需作業事件的其他資訊，請在 [作業] 資料行中，按一下作業事件以開啟其刀鋒視窗。此刀鋒視窗包含事件的詳細資訊。事件會依其相互關聯識別碼以及在時間範圍內發生的事件清單分組。

    ![Operation Details](./media/backup-azure-monitor-vms/audit-logs-details-window.png)

3. 若要檢視特定事件的詳細資訊，請從事件清單中按一下事件，以開啟其 [詳細資料] 刀鋒視窗。

    ![事件詳細資料](./media/backup-azure-monitor-vms/audit-logs-details-window-deep.png)

    事件層級資訊隨著資訊越多而越詳細。如果您想查看有關每個事件的這麼多資訊，而且想要將這麼多詳細資料加入至 [事件] 刀鋒視窗，請參閱[展開事件資訊](backup-azure-monitor-vms.md#view-additional-event-attributes)一節。


## 自訂事件篩選器
使用 [篩選] 進行調整，或選擇特定刀鋒視窗中顯示的資訊。若要篩選事件資訊︰

1. 在[保存庫儀表板](./backup-azure-manage-vms.md#open-a-recovery-services-vault-in-the-dashboard)中，瀏覽並按一下 [稽核記錄檔]，以開啟 [事件] 刀鋒視窗。

    ![稽核記錄檔](./media/backup-azure-monitor-vms/audit-logs-1606-1.png)

    開啟的 [事件] 刀鋒視窗會顯示針對目前保存庫而篩選的作業事件。

    ![稽核記錄檔篩選器](./media/backup-azure-monitor-vms/audit-logs-filter.png)

2. 在 [事件] 功能表上，按一下 [篩選] 以開啟該刀鋒視窗。

    ![開放篩選刀鋒視窗](./media/backup-azure-monitor-vms/audit-logs-filter-button.png)

3. 在 [篩選] 刀鋒視窗上，調整 [層級]、[時間範圍] 和 [呼叫者] 篩選器。其他篩選器已設定為提供復原服務保存庫的目前資訊，所以無法使用。

    ![稽核記錄檔查詢詳細資料](./media/backup-azure-monitor-vms/filter-blade.png)

    您可以指定事件的**層級**︰嚴重、錯誤、警告或資訊。您可以選擇任何事件層級組合，但必須選取至少一個層級。開啟或關閉層級。[時間範圍] 篩選器可讓您指定時間長度來擷取事件。如果您使用自訂的時間範圍，您可以設定開始和結束時間。

4. 當您準備好使用篩選器來查詢作業記錄時，請按一下 [更新]。結果會顯示在 [事件] 刀鋒視窗中。

    ![Operation Details](./media/backup-azure-monitor-vms/edited-list-of-events.png)


### 檢視其他事件屬性
使用 [資料行] 按鈕，您可以讓其他事件屬性出現在 [事件] 刀鋒視窗上的清單中。事件的預設清單會顯示 [作業]、[層級]、[狀態]、[資源] 和 [時間] 資訊。若要啟用其他屬性︰

1. 在 [事件] 刀鋒視窗上，按一下 [資料行]。

    ![開啟資料行](./media/backup-azure-monitor-vms/audi-logs-column-button.png)

    [選擇資料行] 刀鋒視窗隨即開啟。

    ![資料行刀鋒視窗](./media/backup-azure-monitor-vms/columns-blade.png)

2. 若要選取屬性，請按一下核取方塊。屬性核取方塊可進行開啟和關閉切換。

3. 按一下 [重設] 以在 [事件] 刀鋒視窗中重設屬性清單。從清單中新增或移除屬性之後，使用 [重設] 來檢視新的事件屬性清單。

4. 按一下 [更新] 以更新事件屬性的資料。下表提供每個屬性的相關資訊。

| 資料行名稱 |說明|
| -----------------|-----------|
| 作業|作業的名稱|
| 等級|作業的層級，其值可以是︰資訊、警告、錯誤或嚴重|
|狀態|作業的描述性狀態|
|資源|可識別資源的 URI；也稱為資源識別碼|
|時間|事件發生時的時間 (從目前時間開始測量)|
|呼叫者|何者或何物呼叫或觸發事件；可以是系統或使用者|
|Timestamp|觸發事件時的時間|
|資源群組|相關聯的資源群組|
|資源類型|Resource Manager 所使用的內部資源類型|
|訂用帳戶識別碼|相關聯的訂用帳戶識別碼|
|類別|事件的類別|
|相互關連識別碼|相關事件的通用識別碼|



## 使用 PowerShell 自訂警示
您可以在入口網站取得作業的自訂警示通知。若要取得這些作業，請在作業記錄事件中定義以 PowerShell 為基礎的警示規則。使用 PowerShell 1.3.0 版或更新版本。

若要定義自訂通知以警示備份失敗，請使用如同下列指令碼的命令：

```
PS C:\> $actionEmail = New-AzureRmAlertRuleEmail -CustomEmail contoso@microsoft.com
PS C:\> Add-AzureRmLogAlertRule -Name backupFailedAlert -Location "East US" -ResourceGroup RecoveryServices-DP2RCXUGWS3MLJF4LKPI3A3OMJ2DI4SRJK6HIJH22HFIHZVVELRQ-East-US -OperationName Microsoft.Backup/RecoveryServicesVault/Backup -Status Failed -TargetResourceId /subscriptions/86eeac34-eth9a-4de3-84db-7a27d121967e/resourceGroups/RecoveryServices-DP2RCXUGWS3MLJF4LKPI3A3OMJ2DI4SRJK6HIJH22HFIHZVVELRQ-East-US/providers/microsoft.backupbvtd2/RecoveryServicesVault/trinadhVault -Actions $actionEmail
```

**ResourceId**︰您可以從稽核記錄檔中取得 ResourceId。ResourceId 是作業記錄檔的 [資源] 資料行中提供的 URL。

**OperationName**：OperationName 的格式為 "Microsoft.RecoveryServices/recoveryServicesVault/*EventName*"，其中 *EventName* 可以是：<br/>
- Register <br/>
- Unregister <br/>
- ConfigureProtection <br/>
- Backup <br/>
- Restore <br/>
- StopProtection <br/>
- DeleteBackupData <br/>
- CreateProtectionPolicy <br/>
- DeleteProtectionPolicy <br/>
- UpdateProtectionPolicy <br/>

**Status**：支援的值為 [已開始]、[成功] 或 [失敗]。

**ResourceGroup**︰這是資源所屬的資源群組。您可以將 [資源群組] 資料行加入至產生的記錄檔。資源群組是其中一個可用的事件資訊類型。

**Name**：警示規則的名稱。

**CustomEmail**：指定您要傳送警示通知的自訂電子郵件地址

**SendToServiceOwners**：此選項會將警示通知傳送給訂用帳戶的所有系統管理員和共同管理員。它可以用於 **New-AzureRmAlertRuleEmail** Cmdlet 中

### 警示的限制
以事件為基礎的警示受到下列限制：

1. 在復原服務保存庫中的所有虛擬機器上觸發警示。您無法針對復原服務保存庫中的部份虛擬機器自訂警示。
2. 這項功能處於預覽狀態。[深入了解](../azure-portal/insights-powershell-samples.md#create-alert-rules)
3. 警示會從 "alerts-noreply@mail.windowsazure.com" 寄出。目前您無法修改電子郵件寄件者。


## 後續步驟

事件記錄檔會啟用備份作業的絕佳事後剖析和稽核支援。系統會記錄下列作業：

- 註冊
- 取消註冊
- 設定保護
- 備份 (排程和隨選備份兩者)
- 還原
- 停止保護
- 刪除備份資料
- Add policy
- 刪除原則
- 更新原則
- 取消工作

如需各項 Azure 服務的事件、作業和稽核記錄檔的廣泛說明，請參閱[檢視事件和稽核記錄檔](../azure-portal/insights-debugging-with-events.md)一文。

如需從復原點重新建立虛擬機器的詳細資訊，請參閱[還原 Azure VM](backup-azure-restore-vms.md)。如需保護虛擬機器的詳細資訊，請參閱[搶先目睹︰將 VM 備份至復原服務保存庫](backup-azure-vms-first-look-arm.md)。深入了解[管理 Azure 虛擬機器備份](backup-azure-manage-vms.md)一文中 VM 備份的管理工作。

<!---HONumber=AcomDC_0921_2016-->