<properties
	pageTitle="管理 Azure 復原服務保存庫與伺服器 | Microsoft Azure"
	description="使用本教學課程了解如何管理 Azure 復原服務保存庫與伺服器。"
	services="backup"
	documentationCenter=""
	authors="Jim-Parker"
	manager="jwhit"
	editor="tysonn"/>

<tags
	ms.service="backup"
	ms.workload="storage-backup-recovery"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="07/19/2016"
	ms.author="jimpark; markgal"/>


# 監視和管理 Windows 電腦的 Azure 復原服務保存庫和伺服器

> [AZURE.SELECTOR]
- [資源管理員](backup-azure-manage-windows-server.md)
- [傳統](backup-azure-manage-windows-server-classic.md)

在本文中，您將了解透過 Azure 管理入口網站和 Microsoft Azure 備份代理程式提供之備份管理工作的概觀。

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-rm-include.md)] 傳統部署模型。

## 管理入口網站工作

### 存取復原服務保存庫

1. 使用 Azure 訂用帳戶登入 [Azure 入口網站](https://portal.azure.com/)。

2. 在 [中樞] 功能表上按一下 [瀏覽]，然後在資源清單中輸入**復原服務**。當您開始輸入時，清單將會根據您輸入的文字進行篩選。按一下 [復原服務保存庫]。

    ![建立復原服務保存庫的步驟 1](./media/backup-azure-manage-windows-server/browse-to-rs-vaults.png) <br/>

2. 從清單中選取您要檢視的保存庫名稱，以開啟復原服務保存庫儀表板刀鋒視窗。

    ![復原服務保存庫儀表板](./media/backup-azure-manage-windows-server/rs-vault-dashboard.png) <br/>

## 監視作業和警示
您可以從復原服務保存庫儀表板監視作業和警示，您可以看到︰

- 備份警示詳細資料
- 檔案和資料夾，以及雲端中受保護的 Azure 虛擬機器
- Azure 中已使用的儲存體總計
- 備份作業狀態

![備份儀表板工作](./media/backup-azure-manage-windows-server/dashboard-tiles.png)

按一下上述每個圖格中的資訊，將會開啟相關聯的刀鋒視窗以便管理相關工作。

在儀表板的頂端：

- [設定] 可供存取可用的備份工作。
- 備份 - 協助您將新的檔案和資料夾 (或 Azure VM) 備份至復原服務保存庫。
- 刪除 - 如果不再使用復原服務保存庫，您可予以刪除來釋出儲存空間。只有在從保存庫中刪除了所有受保護的伺服器之後，[刪除] 才會啟用。

![備份儀表板工作](./media/backup-azure-manage-windows-server/dashboard-tasks.png)
## 針對使用 Azure 備份代理程式之備份的警示：
| 警示層級 | 傳送的警示 |
| ------------- | ------------- |
| 重要 | 備份失敗、復原失敗 |
| 警告 | 備份完成，但有警告 (當不到一百個檔案因為損毀問題而未備份，以及超過一百萬個檔案成功備份時) |
| 資訊 | None |
## 管理備份警示
按一下 [備份警示] 圖格以開啟 [備份警示] 刀鋒視窗及管理警示。

![備份警示](./media/backup-azure-manage-windows-server/manage-backup-alerts.png)

[備份警示] 圖格會顯示下列各項的數目︰

- 過去 24 小時內未解決的重大警示
- 過去 24 小時內未解決的警告警示

按一下每個連結會帶您前往 [備份警示] 刀鋒視窗，其中包含這些警示 (重大或警告) 的篩選檢視。

在 [備份警示] 刀鋒視窗中，您可︰

- 選擇要與您的警示一起納入的適當資訊。

    ![選擇資料行](./media/backup-azure-manage-windows-server/choose-alerts-colunms.png)

- 針對嚴重性、狀態和開始/結束時間篩選警示。

    ![篩選警示](./media/backup-azure-manage-windows-server/filter-alerts.png)

- 針對嚴重性、頻率和接收者設定通知，以及開啟或關閉警示。

    ![篩選警示](./media/backup-azure-manage-windows-server/configure-notifications.png)

如果選取 [每個警示] 做為 [通知] 頻率，則電子郵件不會分組或減少。每個警示會產生 1 則通知。這是預設設定，而且也會立即送出解決方式電子郵件。

如果選取 [每小時摘要] 做為 [通知] 頻率，則會傳送一封電子郵件給使用者，告知過去一小時有未解決的新警示產生。解決方式電子郵件會在每小時結束時送出。

可以針對下列嚴重性等級傳送警示︰

- 重大
- 警告
- 資訊

您可使用 [作業詳細資料] 刀鋒視窗中的 [停用] 按鈕來停用警示。當您按一下 [停用] 時，您可以提供解決方式資訊。

您可以使用 [選擇資料行] 按鈕，選擇您要顯示在警示中的資料行。

>[AZURE.NOTE] 在 [設定] 刀鋒視窗中，選取 [監視和報告] > [警示和事件] > [備份警示]，然後按一下 [篩選] 或 [設定通知]，以管理備份警示。

## 管理備份項目
管理內部部署備份現在已可在管理入口網站中使用。在儀表板的 [備份] 區段中，[備份項目] 圖格會顯示保存庫中受保護的備份項目數。

按一下 [備份項目] 圖格中的 [檔案-資料夾]。

![備份項目圖格](./media/backup-azure-manage-windows-server/backup-items-tile.png)

隨即開啟篩選器設定為 [檔案-資料夾] 的 [備份項目] 刀鋒視窗，其中列出每個特定備份項目。

![備份項目](./media/backup-azure-manage-windows-server/backup-item-list.png)

如果您按一下清單中的特定備份項目，您會看到該項目的基本詳細資料。

>[AZURE.NOTE] 在 [設定] 刀鋒視窗中，您可選取 [受保護項目] > [備份項目]，然後從下拉式功能表中選取 [檔案-資料夾]，以管理檔案和資料夾。

![從設定備份項目](./media/backup-azure-manage-windows-server/backup-files-and-folders.png)

## 管理備份作業
內部部署 (當內部部署伺服器備份至 Azure 時) 和 Azure 備份的備份作業均會顯示在儀表板中。

在儀表板的 [備份] 區段中，[備份作業] 圖格會顯示作業數目：

- 進行中
- 過去 24 小時內失敗。

若要管理您的備份作業，請按一下 [備份作業] 圖格，即會開啟 [備份作業] 刀鋒視窗。

![從設定備份項目](./media/backup-azure-manage-windows-server/backup-jobs.png)

您可使用頁面頂端的 [選擇資料行] 按鈕，修改 [備份作業] 刀鋒視窗中可用的資訊。

使用 [篩選] 按鈕來選取檔案和資料夾以及 Azure 虛擬機器備份。

如果您未看到已備份的檔案和資料夾，按一下頁面頂端的 [篩選] 按鈕，然後從 [項目類型] 功能表中選取 [檔案和資料夾]。

>[AZURE.NOTE] 在 [設定] 刀鋒視窗中，您可選取 [監視和報告] > [作業] > [備份作業]，然後從下拉式功能表中選取 [檔案-資料夾]，以管理備份作業。

## 監視備份使用量
在儀表板的 [備份] 區段中，[備份使用量] 圖格會顯示 Azure 中耗用的儲存體。提供下列各項的儲存體使用量︰
- 與保存庫相關聯的雲端 LRS 儲存體使用量
- 與保存庫相關聯的雲端 GRS 儲存體使用量

## 生產伺服器
若要管理您的生產伺服器，請按一下 [設定]。按一下 [管理] 之下的 [備份基礎結構] > [生產伺服器]。

[生產伺服器] 刀鋒視窗會列出所有可用的生產伺服器。按一下清單中的伺服器以開啟伺服器詳細資料。

![受保護項目](./media/backup-azure-manage-windows-server/production-server-list.png)

## Microsoft Azure 備份代理程式工作

## 開啟備份代理程式

開啟 **Microsoft Azure 備份**代理程式 (透過在電腦中搜尋「Microsoft Azure 備份」即可找到)。

![Windows Server 備份排程](./media/backup-azure-manage-windows-server/snap-in-search.png)

從備份代理程式主控台右側可用的 [動作]，您可執行下列管理工作︰

- 註冊伺服器
- 排程備份
- 立即備份
- 變更屬性

![Microsoft Azure 備份代理程式主控台動作](./media/backup-azure-manage-windows-server/console-actions.png)

>[AZURE.NOTE] 若要**復原資料**，請參閱[將檔案還原到 Windows Server 或 Windows 用戶端電腦](backup-azure-restore-windows-server.md)。

## 修改現有備份

1. 在 Microsoft Azure 備份代理程式中，按一下 [排程備份]。

    ![Windows Server 備份排程](./media/backup-azure-manage-windows-server/schedule-backup.png)

2. 在 [排程備份精靈] 中，讓 [變更備份項目或時間] 選項保留選取狀態，然後按 [下一步]。

    ![Windows Server 備份排程](./media/backup-azure-manage-windows-server/modify-or-stop-a-scheduled-backup.png)

3. 如果您想要新增或變更項目，在 [選取要備份的項目] 畫面中按一下 [新增項目]。

    您也可以在這個精靈頁面中設定 [排除設定]。如果您想要排除檔案或檔案類型，請參閱新增[排除設定](#exclusion-settings)的程序。

4. 選取要備份的檔案和資料夾，然後按一下 [確定]。

    ![Windows Server 備份排程](./media/backup-azure-manage-windows-server/add-items-modify.png)

5. 指定 [備份排程]，然後按 [下一步]。

    您可以排程每日 (一天最多 3 次) 或每週備份。

    ![Windows Server 備份項目](./media/backup-azure-manage-windows-server/specify-backup-schedule-modify-close.png)

    >[AZURE.NOTE] 這篇[文章](backup-azure-backup-cloud-as-tape.md)中會詳細說明指定備份排程。

6. 選取備份複本的 [保留原則]，然後按 [下一步]。

    ![Windows Server 備份項目](./media/backup-azure-manage-windows-server/select-retention-policy-modify.png)

7. 在 [確認] 畫面上檢閱資訊，然後按一下 [完成]。

8. 當精靈完成 [備份排程] 的建立之後，按一下 [關閉]。

    修改保護之後，您可以藉由移至 [工作] 索引標籤並確認變更反映於備份工作中，來確定可正確觸發備份。

## 啟用網路節流  
Azure 備份代理程式提供 [節流] 索引標籤，可讓您控制在資料傳輸期間使用網路頻寬的方式。如果您需要在上班時間內備份資料，但不希望備份程序干擾其他網際網路流量，這樣的控制會很有幫助。資料傳輸的節流適用於備份和還原活動。

啟用節流︰

1. 在 [備份代理程式] 中，按一下 [變更屬性]。

2. 選取 [啟用備份作業的網際網路頻寬使用節流功能] 核取圖格。

    ![網路節流](./media/backup-azure-manage-windows-server/throttling-dialog.png)

3. 一旦啟用節流之後，請指定允許在 [工作時間] 和 [非工作時間] 進行備份資料傳輸的頻寬。

    頻寬值從每秒 512 KB (Kbps) 開始，並可高達每秒 1023 MB (Mbps)。您也可以指定 [工作時間] 的開始和完成時間，以及一週中有哪幾天視為工作天。指定工作時間以外的時間會視為非工作時間。

4. 按一下 [確定]。

## 管理排除設定

1. 開啟 **Microsoft Azure 備份代理程式** (透過在電腦中搜尋「Microsoft Azure 備份」即可找到)。

    ![Windows Server 備份排程](./media/backup-azure-manage-windows-server/snap-in-search.png)

2. 在 Microsoft Azure 備份代理程式中，按一下 [排程備份]。

    ![Windows Server 備份排程](./media/backup-azure-manage-windows-server/schedule-backup.png)

3. 在排程備份精靈中，讓 [變更備份項目或時間] 選項保留選取狀態，然後按 [下一步]。

    ![Windows Server 備份排程](./media/backup-azure-manage-windows-server/modify-or-stop-a-scheduled-backup.png)

4. 按一下 [排除設定]。

    ![Windows Server 備份排程](./media/backup-azure-manage-windows-server/exclusion-settings.png)

5. 按一下 [新增排除]。

    ![Windows Server 備份排程](./media/backup-azure-manage-windows-server/add-exclusion.png)

6. 選取位置，然後按一下 [確定]。

    ![Windows Server 備份排程](./media/backup-azure-manage-windows-server/exclusion-location.png)

7. 在 [檔案類型] 欄位中新增副檔名。

    ![Windows Server 備份排程](./media/backup-azure-manage-windows-server/exclude-file-type.png)

    新增 .mp3 副檔名

    ![Windows Server 備份排程](./media/backup-azure-manage-windows-server/exclude-mp3.png)

    若要新增其他副檔名，可按一下 [新增排除]，然後輸入另一個副檔名 (新增 .jpeg 副檔名)。

    ![Windows Server 備份排程](./media/backup-azure-manage-windows-server/exclude-jpg.png)

8. 當您已新增所有副檔名之後，按一下 [確定]。

9. 按 [下一步] 繼續執行排程備份精靈， 直到出現 [確認] 頁面，然後按一下 [完成]。

    ![Windows Server 備份排程](./media/backup-azure-manage-windows-server/finish-exclusions.png)

## 常見問題集
**Q1.備份作業狀態在 Azure 備份代理程式中顯示為已完成，但為何不會立即反映在入口網站中？**

A1.Azure 備份代理程式與 Azure 入口網站中反映的備份作業狀態之間最多有 15 分鐘的延遲。

**Q.2 當備份作業失敗時，需要多久的時間才會引發警示？**

A.2 在 Azure 備份失敗的 5 分鐘內就會引發警示。

**Q3.是否會有已設定通知但不會傳送電子郵件的情況？**

A3.以下是不傳送通知以便減少警示雜訊的案例︰

   - 如果通知設為每小時，而且在一小時內引發警示並加以解決，
   - 作業便會取消。
   - 第二個備份作業會失敗，因為原始的備份作業正在進行中。

## 疑難排解監視問題<br>
#### 問題︰來自 Azure 備份代理程式的作業與警示未出現在入口網站中。
##### 疑難排解步驟：
「OBRecoveryServicesManagementAgent」可用來將作業和警示的資料傳送至 Azure 備份服務。開啟工作管理員，然後檢視「OBRecoveryServicesManagementAgent」處理序是否正在執行。此處理序有時會停止或關閉。如果處理序並未執行，請從控制台瀏覽服務清單、啟動或重新啟動「Microsoft Azure 復原服務管理代理程式」。如需進一步資訊，請瀏覽「Azure 備份代理程式安裝資料夾」\\Microsoft Azure Recovery Services Agent\\Temp\\GatewayProvider* 中的記錄。<b>例如：</b>C:\\Program Files\\Microsoft Azure Recovery Services Agent\\Temp\\GatewayProvider0.errlog

## 後續步驟
- [從 Azure 還原 Windows Server 或 Windows 用戶端](backup-azure-restore-windows-server.md)
- 若要深入了解 Azure 備份，請參閱 [Azure 備份概觀](backup-introduction-to-azure-backup.md)
- 瀏覽 [Azure 備份論壇](http://go.microsoft.com/fwlink/p/?LinkId=290933)

<!---HONumber=AcomDC_0921_2016-->