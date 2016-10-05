<!--author=SharS last changed: 02/04/2016-->

#### 建立磁碟區

1. 在裝置的 [快速入門] 頁面上，按一下 [新增磁碟區]。這樣會啟動 [新增磁碟區] 精靈。

2. 在 [新增磁碟區精靈] 的 [基本設定] 下，執行列動作：
   1. 輸入磁碟區的 [名稱]。
   2. 為磁碟區指定 [佈建的容量] \(GB 或 TB)。實體裝置的磁碟區容量必須介於 1 GB 到 64 TB 之間。
   3. 在下拉式清單中，為磁碟區選取 [使用類型] 。 
   4. 如果您將此磁碟區用於封存資料，請選取 [**將此磁碟區用於較不常存取的封存資料**] 核取方塊。對於所有其他使用案例，只需選取 [**階層式磁碟區**]。(階層式磁碟區之前稱為主要磁碟區)。

        ![Add volume](./media/storsimple-create-volume/ScreenshotUpdate1VolumeFlow.png)

    4. 按一下箭頭圖示 ![arrow-icon](./media/storsimple-create-volume/HCS_ArrowIcon-include.png) 以移至下一頁。

3. 在 [其他設定] 對話方塊中，加入新的存取控制記錄 (ACR)：
   1. 提供 ACR 的 [名稱]。
   2. 在 [iSCSI 啟動器名稱] 下方，提供 Windows 主機的 iSCSI 完整格式名稱 (IQN)。如果沒有 IQN，請移至 [取得 Windows Server 主機的 IQN](#get-the-iqn-of-a-windows-server-host)。
   3. 建議選取 [**啟用此磁碟區的預設備份**] 核取方塊啟用預設備份。預設備份將會建立原則，在每天的 22:30 (裝置時間) 執行，並建立此磁碟區的雲端快照。

        > [AZURE.NOTE] 在此處啟用備份之後，就無法加以還原。您必須編輯磁碟區，才能修改此設定。

        ![Add volume](./media/storsimple-create-volume/AddVolume2-include.png)

4. 按一下核取圖示 ![核取圖示](./media/storsimple-create-volume/HCS_CheckIcon-include.png)。使用指定的設定來建立磁碟區。

![提供的影片](./media/storsimple-create-volume/Video_icon.png) **提供的影片**

若要觀看影片示範如何建立 StorSimple 磁碟區，請按一下[這裡](https://azure.microsoft.com/documentation/videos/create-a-storsimple-volume/)。

<!-----HONumber=AcomDC_0211_2016--->

