<properties
pageTitle="使用適用於 Visual Studio 的 Microsoft Azure Data Lake 工具搭配 Hortonworks 沙箱 | Microsoft Azure"
description="了解如何使用適用於 Visual Studio 的 Azure Data Lake 工具搭配 Hortonworks 沙箱 (在本機 VM 中執行)。 您可以使用這些工具，在沙箱上建立和執行 Hive 和 Pig 作業，以及檢視作業輸出和歷程記錄。"
services="hdinsight"
documentationCenter=""
authors="Blackmist"
manager="paulettm"
editor="cgronlun"/>

<tags
ms.service="hdinsight"
ms.devlang="na"
ms.topic="article"
ms.tgt_pltfrm="na"
ms.workload="big-data"
ms.date="08/26/2016"
ms.author="larryfr"/>

# 使用適用於 Visual Studio 的 Azure Data Lake 工具搭配 Hortonworks 沙箱

除了用於 Azure Data Lake 和 HDInsight 的工具以外，適用於 Visual Studio 的 Azure Data Lake 工具還包括用於一般 Hadoop 叢集的工具。本文件提供使用 Azure Data Lake 工具搭配在本機虛擬機器中執行的 Hortonworks 沙箱所需的步驟。

使用 Hortonworks 沙箱，可讓您在本機開發環境上使用 Hadoop。一旦您已開發解決方案並想要進行大規模部署時，您可以接著移至 HDInsight 叢集。

## 必要條件

* 在您的開發環境上的虛擬機器中執行的 Hortonworks 沙箱。本文件是針對在 Oracle VirtualBox 中執行的沙箱進行撰寫和測試，而該沙箱是使用[在 Hadoop 生態系統中開始使用](hdinsight-hadoop-emulator-get-started.md)文件中的資訊進行設定。

* Visual Studio 2013 或 2015，或任何版本。

* [Azure SDK for.NET](https://azure.microsoft.com/downloads/) 2.7.1 或更新版本

* [適用於 Visual Studio 的 Azure Data Lake 工具](https://www.microsoft.com/download/details.aspx?id=49504)

## 設定沙箱的密碼

請確定 Hortonworks 沙箱執行中，然後依照[在 Hadoop 生態系統中開始使用](hdinsight-hadoop-emulator-get-started.md#set-passwords)中的步驟來設定 SSH `root` 帳戶及 Ambari `admin` 帳戶的密碼。從 Visual Studio 連接至沙箱時，將使用這些密碼。

## 將工具連接至沙箱

1. 開啟 Visual Studio，然後依序選取 [檢視] 和 [伺服器總管]。

2. 在 [伺服器總管] 中，以滑鼠右鍵按一下 [HDInsight] 項目，然後選取 [連接到 HDInsight Emulator]。

    ![連接到 HDInsight Emulator](./media/hdinsight-hadoop-emulator-visual-studio/connect-emulator.png)

3. 在 [連接到 HDInsight Emulator] 對話方塊中，輸入您為 Ambari 設定的密碼。

    ![輸入 Ambari 密碼](./media/hdinsight-hadoop-emulator-visual-studio/enter-ambari-password.png)

    選取 [下一步] 以繼續操作。

4. 使用 [密碼] 欄位來輸入您為 `root` 帳戶所設定的密碼。讓其他欄位保持預設值。

    ![輸入根密碼](./media/hdinsight-hadoop-emulator-visual-studio/enter-root-password.png)

    選取 [下一步] 以繼續操作。

5. 等候服務的驗證完成。在某些情況下，驗證可能會失敗並提示您更新組態。發生這種情況時，選取 [更新] 按鈕並等候服務的設定和驗證完成。

    ![錯誤和更新按鈕](./media/hdinsight-hadoop-emulator-visual-studio/fail-and-update.png)

    > [AZURE.NOTE] 更新程序會使用 Ambari，將 Hortonworks 沙箱組態修改成適用於 Visual Studio 的 Azure Data Lake 工具所預期的組態。

    驗證完成後，選取 [完成] 以完成設定。

    ![完成連接](./media/hdinsight-hadoop-emulator-visual-studio/finished-connect.png)

    > [AZURE.NOTE] 視您的開發環境的速度，以及配置給虛擬機器的記憶體數量而定，可能需要幾分鐘的時間來設定及驗證服務。

完成這些步驟後，在 [伺服器總管] 的 HDInsight 區段下立即會有 [HDInsight 本機叢集] 項目。

## 撰寫 Hive 查詢

Hive 會提供類似 SQL 的查詢語言 (HiveQL)，以便處理結構化資料。使用下列步驟，了解如何針對本機叢集執行特定查詢。

1. 在 [伺服器總管] 中，以滑鼠右鍵按一下您先前新增的本機叢集項目，然後選取 [撰寫 Hive 查詢]。

    ![撰寫 Hive 查詢](./media/hdinsight-hadoop-emulator-visual-studio/write-hive-query.png)

    這會開啟新的查詢視窗，讓您快速輸入查詢並提交到本機叢集。

2. 在新的查詢視窗中，輸入以下資訊︰

        select count(*) from sample_08;
    
    在查詢視窗的頂端，確定已選取本機叢集的組態，然後選取 [提交]。讓其他值 ([批次] 和伺服器名稱) 保持預設值。

    ![查詢視窗和提交按鈕](./media/hdinsight-hadoop-emulator-visual-studio/submit-hive.png)

    請注意，您也可以使用 [提交] 旁邊的下拉式功能表來選取 [進階]。這會開啟一個對話方塊，讓您在提交作業時提供其他選項。

    ![進階提交](./media/hdinsight-hadoop-emulator-visual-studio/advanced-hive.png)

3. 提交查詢後，將會顯示作業狀態。這可提供 Hadoop 處理作業時的相關資訊。[作業狀態] 項目會提供作業的目前狀態。系統會定期更新此狀態，您也可以使用重新整理圖示來手動重新整理此狀態。

    ![作業狀態](./media/hdinsight-hadoop-emulator-visual-studio/job-state.png)

    [作業狀態] 變成 [已完成] 後，有向非循環圖 (DAG) 隨即顯示。此圖說明 Tez 所決定的執行路徑 (本機叢集上 Hive 的預設執行引擎)。
    
    > [AZURE.NOTE] Tez 也是使用以 Linux 為基礎的 HDInsight 叢集時的預設值。但不是以 Windows 為基礎的 HDInsight 上的預設值；若要在那裡使用它，您必須將 `set hive.execution.engine = tez;` 一行加入至 Hive 查詢的開頭。

    使用 [作業輸出] 連結來檢視輸出。在此情況下，這是 __823__；sample\_08 資料表中的資料列數目。您可以使用 [作業記錄檔] 和 [下載 YARN 記錄檔] 連結，檢視作業的相關診斷資訊。

4. 您也可以將 [批次] 欄位變更為 [互動式]，以互動方式執行 Hive 作業，然後選取 [執行]。

    ![互動式查詢](./media/hdinsight-hadoop-emulator-visual-studio/interactive-query.png)

    這可串流處理至 [HiveServer2 輸出] 視窗期間所產生的輸出記錄檔。
    
    > [AZURE.NOTE] 這是作業完成後 [作業記錄檔] 連結中可用的相同資訊。

    ![HiveServer2 輸出](./media/hdinsight-hadoop-emulator-visual-studio/hiveserver2-output.png)

## 建立 Hive 專案

您也可以建立包含多個 Hive 指令碼的專案。當您有需要留在一起或使用版本控制系統維護的相關指令碼時，這非常有用。

1. 在 Visual Studio 中，依序選取 [檔案]、[新增] 和 [專案]。

2. 從專案清單，展開 [範本]、[Azure Data Lake]，然後選取 [HIVE (HDInsight)]。從範本清單中，選取 [Hive 範例]。輸入名稱和位置，然後選取 [確定]。

    ![HIVE (HDInsight) 範本](./media/hdinsight-hadoop-emulator-visual-studio/new-hive-project.png)

[Hive 範例] 專案包含兩個指令碼：__WebLogAnalysis.hql__ 和 __SensorDataAnalysis.hql__。您可以使用視窗頂端的同一個 [提交] 按鈕進行提交。

## 建立 Pig 專案

當 Hive 提供類似 SQL 的語言來處理結構化資料時，Pig 會提供語言 (Pig Latin)，讓您為套用至您的資料的轉換開發管線。使用下列步驟來搭配使用 Pig 與本機叢集。

1. 開啟 Visual Studio，並依序選取 [檔案] > [新增] 和 [專案]。從專案清單，展開 [範本]、[Azure Data Lake]，然後選取 [Pig (HDInsight)]。從範本清單中，選取 [Pig 應用程式]。輸入名稱、位置，然後選取 [確定]。

    ![Pig (HDInsight) 專案](./media/hdinsight-hadoop-emulator-visual-studio/new-pig.png)

2. 輸入下列內容做為與此專案一起建立的檔案 __script.pig__ 的內容。

        a = LOAD '/demo/data/Website/Website-Logs' AS (
            log_id:int, 
            ip_address:chararray, 
            date:chararray, 
            time:chararray, 
            landing_page:chararray, 
            source:chararray);
        b = FILTER a BY (log_id > 100);
        c = GROUP b BY ip_address;
        DUMP c;

    雖然 Pig 使用與 Hive 不同的語言，但您透過 [提交] 按鈕在這兩種語言之間執行作業的方式一致。選取 [提交] 旁邊的下拉式清單，即可顯示 Pig 的進階提交對話方塊。

    ![Pig 進階提交](./media/hdinsight-hadoop-emulator-visual-studio/advanced-pig.png)
    
3. 如同 Hive 查詢，也會顯示作業狀態和輸出。

    ![已完成 Pig 作業的影像](./media/hdinsight-hadoop-emulator-visual-studio/completed-pig.png)

## 檢視作業

Azure Data Lake 工具也可讓您輕鬆地檢視已在 Hadoop 上執行的作業相關資訊。使用下列步驟來查看已在本機叢集上執行的作業。

1. 從 [伺服器總管] 中，於本機叢集上按一下滑鼠右鍵，然後選取 [檢視作業]。這會顯示已提交至叢集的作業清單。

    ![檢視作業](./media/hdinsight-hadoop-emulator-visual-studio/view-jobs.png)

2. 選取作業清單中的作業，以檢視作業詳細資料。

    ![選取作業](./media/hdinsight-hadoop-emulator-visual-studio/view-job-details.png)

    顯示的資訊類似您在執行 Pig 或 Hive 查詢後看到的資訊，再加上用來檢視輸出和記錄檔資訊的連結。

3. 您也可以從這裡修改和重新提交作業。

## 檢視 Hive 資料庫

1. 在 [伺服器總管] 中，展開 [HDInsight 本機叢集] 項目，然後展開 [Hive 資料庫]。這將顯示本機叢集上的 [預設] 和 [xademo] 資料庫。展開資料庫即可顯示資料庫內的資料表。

    ![擴充的資料庫](./media/hdinsight-hadoop-emulator-visual-studio/expanded-databases.png)

2. 展開資料表即可顯示該資料表的資料行。您可以以滑鼠右鍵按一下資料表並選取 [檢視前 100 個資料列]，即可快速檢視資料。

    ![Hive 資料庫檢視](./media/hdinsight-hadoop-emulator-visual-studio/view-100.png)

### 資料庫和資料表屬性

您可能已注意到，您可以選擇檢視資料庫或資料表的 [屬性]。這會在 [屬性] 視窗中顯示所選項目的詳細資料。

![屬性](./media/hdinsight-hadoop-emulator-visual-studio/properties.png)

### 建立資料表

若要建立新的資料表，以滑鼠右鍵按一下資料庫，然後按一下 [建立資料表]。

![建立資料表](./media/hdinsight-hadoop-emulator-visual-studio/create-table.png)

然後，您可以使用表單建立資料表。您可以看到原始 HiveQL，其將用於建立此頁面底部的資料表。

![建立資料表](./media/hdinsight-hadoop-emulator-visual-studio/create-table-form.png)

## 後續步驟

* [了解 Hortonworks 沙箱的訣竅](http://hortonworks.com/hadoop-tutorial/learning-the-ropes-of-the-hortonworks-sandbox/)
* [Hadoop 教學課程 - 開始使用 HDP](http://hortonworks.com/hadoop-tutorial/hello-world-an-introduction-to-hadoop-hcatalog-hive-and-pig/)

<!---HONumber=AcomDC_0921_2016-->