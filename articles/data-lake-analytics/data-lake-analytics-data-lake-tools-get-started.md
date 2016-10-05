<properties
   pageTitle="使用適用於 Visual Studio 的資料湖工具開發 U-SQL 指令碼 | Azure"
   description="了解如何安裝適用於 Visual Studio 的資料湖工具，如何開發和測試 U-SQL 指令碼。"
   services="data-lake-analytics"
   documentationCenter=""
   authors="edmacauley"
   manager="jhubbard"
   editor="cgronlun"/>

<tags
   ms.service="data-lake-analytics"
   ms.devlang="na"
   ms.topic="get-started-article"
   ms.tgt_pltfrm="na"
   ms.workload="big-data"
   ms.date="05/16/2016"
   ms.author="edmaca"/>

# 教學課程：使用適用於 Visual Studio 的資料湖工具開發 U-SQL 指令碼

[AZURE.INCLUDE [get-started-selector](../../includes/data-lake-analytics-selector-get-started.md)]


了解如何安裝適用於 Visual Studio 的資料湖工具，和使用適用於 Visual Studio 的資料湖工具來撰寫和測試 U-SQL 指令碼。

U-SQL 是高度可擴充、高度可延伸的語言，用來準備、轉換和分析在資料湖的所有資料。如需詳細資訊，請參閱 [U-SQL 參考](http://go.microsoft.com/fwlink/p/?LinkId=691348)。


###必要條件

- **Visual Studio 2015、Visual Studio 2013 update 4 或 Visual Studio 2012。支援 Enterprise (Ultimate/Premium)、Professional、Community 版本；不支援 Express 版本。目前不支援 Visual Studio "15"，我們正在開發中。**
- **Microsoft Azure SDK for .NET 2.7.1 版或更新版本**。請使用 [Web Platform Installer](http://www.microsoft.com/web/downloads/platform.aspx) 來安裝。
- **[Visual Studio 適用的資料湖工具](http://aka.ms/adltoolsvs)**。

    安裝 Visual Studio 適用的 Data Lake 工具之後，您會在 [伺服器總管] 中的 [Azure] 節點下看到 [Data Lake Analytics] 節點 (您可以按 Ctrl+Alt+S 開啟 [伺服器總管])。

- **完成[使用 Azure 入口網站開始使用 Azure Data Lake Analytics](data-lake-analytics-get-started-portal.md) 一文中的下列兩個小節**。

	- [建立 Azure 資料湖分析帳戶](data-lake-analytics-get-started-portal.md#create_adl_analytics_account)。
	- [將 SearchLog.tsv 上傳到預設資料湖儲存體帳戶](data-lake-analytics-get-started-portal.md#update-data-to-the-default-adl-storage-account)。

    為了方便起見，您可以在＜[附錄 A：準備教學課程所需的 PowerShell 範例](data-lake-analytics-data-lake-tools-get-started.md#appx-a-powershell-sample-for-preparing-the-tutorial)＞一節中，找到用來建立資料湖分析服務及上傳來源資料檔的 PowerShell 範例指令碼。

	資料湖工具不支援建立資料湖分析帳戶。因此您必須使用 Azure 入口網站、Azure PowerShell、.NET SDK 或 Azure CLI 建立帳戶。若要執行資料湖分析工作，您需要一些資料。即使資料湖工具支援上傳資料，您將使用入口網站來上傳範例資料，以方便遵循本教學課程。

## 連接到 Azure

**連接到資料湖分析**

1. 開啟 Visual Studio。
2. 按一下 [檢視] 功能表的 [伺服器總管] 來開啟伺服器總管。或是按下 **[CTRL] + [ALT] + S**。
3. 對 [Azure] 按一下滑鼠右鍵、按一下 [連接到 Microsoft Azure 訂用帳戶]，然後依照指示進行。
4. 在 [伺服器總管] 中展開 [Azure]，然後展開 [資料湖分析]。如果有資料湖分析帳戶，您就會看到其清單。您無法從 Visual Studio 建立資料湖分析帳戶。若要建立帳戶，請參閱[使用 Azure 入口網站開始使用 Azure Data Lake Analytics](data-lake-analytics-get-started-portal.md) 或[使用 Azure PowerShell 開始使用 Azure Data Lake Analytics](data-lake-analytics-get-started-powershell.md)。

## 上傳來源資料檔案

您已在本教學課程稍早的＜**必要條件**＞一節中上傳部分資料。

萬一您想要使用您自己的資料，以下是從資料湖工具上傳資料的程序。

**將檔案上傳至相依的 Azure 資料湖帳戶**

1. 在 [伺服器總管] 中依序展開 [Azure]、[資料湖分析]、您的資料湖分析帳戶，以及 [儲存體帳戶]。您應該會看到預設資料湖儲存體帳戶，和連結的資料湖儲存體帳戶，和連結的 Azure 儲存體帳戶。預設資料湖帳戶具有「預設儲存體帳戶」的標籤。
2. 以滑鼠右鍵按一下預設資料湖儲存體帳戶，然後按一下 [總管]。它會開啟 [適用於 Visual Studio 的資料湖工具總管] 窗格：它會在左邊顯示樹狀檢視，在右邊顯示內容檢視。
3. 瀏覽至您要上傳檔案的資料夾，
4. 以滑鼠右鍵按一下任何空白區域，然後按一下 [上傳]。

	![U-SQL Visual Studio 專案 U-SQL](./media/data-lake-analytics-data-lake-tools-get-started/data-lake-analytics-data-lake-tools-upload-files.png)

**上傳檔案至連結的 Azure Blob 儲存體帳戶**

1. 在 [伺服器總管] 中依序展開 [Azure]、[資料湖分析]、您的資料湖分析帳戶，以及 [儲存體帳戶]。您應該會看到預設資料湖儲存體帳戶，和連結的資料湖儲存體帳戶，和連結的 Azure 儲存體帳戶。
2. 展開 Azure 儲存體帳戶。
3. 以滑鼠右鍵按一下您想要用來上傳檔案的容器，然後按一下 [總管]。如果您沒有容器，您必須先使用 Azure 入口網站、Azure PowerShell 或其他工具建立一個容器。
4. 瀏覽至您要上傳檔案的資料夾，
5. 以滑鼠右鍵按一下任何空白區域，然後按一下 [上傳]。

## 開發 U-SQL 指令碼

資料湖分析工作是以 U-SQL 語言撰寫。若要深入了解 U-SQL，請參閱[開始使用 U-SQL 語言](data-lake-analytics-u-sql-get-started.md)和[U-SQL 語言參考](http://go.microsoft.com/fwlink/?LinkId=691348)。

**建立並提交資料湖分析工作**

1. 從 [檔案] 功能表中，按一下 [新增]，再按 [專案]。
2. 選取 [U-SQL 專案] 類型。

	![新的 U-SQL Visual Studio 專案](./media/data-lake-analytics-data-lake-tools-get-started/data-lake-analytics-data-lake-tools-new-project.png)

3. 按一下 [確定]。Visual studio 會建立擁有 **Script.usql** 檔案的解決方案。
4. 將下列指令碼輸入 **Script.usql** 檔案：

        @searchlog =
            EXTRACT UserId          int,
                    Start           DateTime,
                    Region          string,
                    Query           string,
                    Duration        int?,
                    Urls            string,
                    ClickedUrls     string
            FROM "/Samples/Data/SearchLog.tsv"
            USING Extractors.Tsv();

		@res =
		    SELECT *
		    FROM @searchlog;        

        OUTPUT @res   
            TO "/Output/SearchLog-from-Data-Lake.csv"
        USING Outputters.Csv();

	此 U-SQL 指令碼會使用 **Extractors.Tsv()** 讀取來源資料檔案，然後使用 **Outputters.Csv()** 建立 csv 檔案。

    除非您將來源檔案複製到不同的位置，否則請勿修改這兩個路徑。資料湖分析將建立輸出資料夾 (若尚未建立)。

	使用儲存在預設資料湖帳戶中檔案的相對路徑，是比較容易的方法。您也可以使用絕對路徑。例如

        adl://<Data LakeStorageAccountName>.azuredatalakestore.net:443/Samples/Data/SearchLog.tsv

    您必須使用絕對路徑存取連結儲存體帳戶中的檔案。儲存在連結 Azure 儲存體帳戶中之檔案的語法是：

        wasb://<BlobContainerName>@<StorageAccountName>.blob.core.windows.net/Samples/Data/SearchLog.tsv

    >[AZURE.NOTE] 目前不支援具有公用 Blob 或公用容器存取權限的 Azure Blob 容器。

	請注意下列功能：

	- **IntelliSense**

		名稱自動完成和成員會針對資料列集、類別、資料庫、結構描述和使用者定義物件 (UDO) 顯示。

		目錄實體的 IntelliSense (資料庫、結構描述、資料表、UDO 等等) 與您的計算帳戶相關。您可以在頂端工具列中檢查目前作用中的計算帳戶、資料庫和結構描述，並且透過下拉式清單進行切換。

    - **展開 *資料行**

        按一下 * 的右邊，您應該會看到 * 下方的藍色底線。將滑鼠游標移到藍色底線上，然後按一下向下箭號。![展開 Data Lake Visual Studio 工具 *](./media/data-lake-analytics-data-lake-tools-get-started/data-lake-analytics-data-lake-tools-expand-asterisk.png)

        按一下 [展開資料行]，然後工具將會用資料行名稱取代 *。

	- **自動格式化**

		使用者可以根據程式碼結構，在 [編輯] -> [進階] 底下變更 U-SQL 指令碼的縮排：

		- 格式化文件 (Ctrl+E、D)：格式化整份文件
		- 格式化選取範圍 (Ctrl+K、Ctrl+F)：格式化選取範圍。如果未進行選取，這個快速鍵會格式化游標所在的行。

		所有格式化規則可在 [工具] -> [選項] -> [文字編輯器] -> [SIP] -> [格式化] 底下進行設定。
	- **智慧縮排**

		適用於 Visual Studio 的資料湖工具能夠在您撰寫指令碼時自動縮排運算式。這項功能預設會停用，使用者需要透過檢查 [U-SQL] -> [選項和設定] -> [切換] -> [啟用智慧縮排] 來啟用它。

	- **移至定義並尋找所有參考**

		以滑鼠右鍵按一下資料列集/參數/資料行/UDO 等的名稱，然後按一下 [移至定義] \(F12) 可讓您瀏覽至其定義。按一下 [尋找所有參考] \(Shift + F12) 會顯示所有參考。

	- **插入 Azure 路徑**

		並非在撰寫指令碼時記住 Azure 檔案路徑並且手動輸入，適用於 Visual Studio 的 Data Lake 工具提供簡單的方法：在編輯器中按一下滑鼠右鍵，按一下 [插入 Azure 路徑]。瀏覽至 Azure Blob 瀏覽器對話方塊中的檔案。按一下 [確定]，檔案路徑就會插入您的程式碼。

5. 指定資料湖分析帳戶、資料庫和結構描述。為了測試，您可以選取 [(本機)] 以便在本機執行指令碼。如需詳細資訊，請參閱＜[在本機執行 U-SQL](#run-u-sql-locally)＞一節。

	![提交 U-SQL Visual Studio 專案](./media/data-lake-analytics-data-lake-tools-get-started/data-lake-analytics-data-lake-tools-submit-job.png)

    如需詳細資訊，請參閱[使用 U-SQL 目錄](data-lake-analytics-use-u-sql-catalog.md)。

5. 在 [方案總管] 中，用滑鼠右鍵按一下 **Script.usql**，然後按一下 [建置指令碼]。確認 [輸出] 窗格中的結果。
6. 在 [方案總管] 中，用滑鼠右鍵按一下 **Script.usql**，然後按一下 [提交指令碼]。(選擇性) 您也可以從 Script.usql 窗格按一下 [提交]。請參閱上一個螢幕擷取畫面。按一下 [提交] 按鈕旁的向下箭號，使用進階選項提交：
7. 指定 [工作名稱]、驗證 [分析帳戶]，然後按一下 [提交]。提交作業完成時，[適用於 Visual Studio 的資料湖工具結果] 視窗中便會出現提交結果和工作連結。

	![提交 U-SQL Visual Studio 專案](./media/data-lake-analytics-data-lake-tools-get-started/data-lake-analytics-data-lake-tools-submit-job-advanced.png)

8. 您必須按一下 [重新整理] 按鈕，才能查看最新的工作狀態並重新整理畫面。當工作成功時，它將會顯示**工作圖形**、**中繼資料作業**、**狀態記錄**、**診斷**：

	![U SQL Visual Studio 資料湖分析工作效能圖表](./media/data-lake-analytics-data-lake-tools-get-started/data-lake-analytics-data-lake-tools-performance-graph.png)

	* 工作摘要。顯示目前工作的摘要資訊，例如：狀態、進度、執行時間、執行階段名稱、傳送者等。
	* 工作詳細資料。提供這項工作的詳細資訊，包括指令碼、資源、頂點執行檢視。
	* 工作圖形。提供四個圖形以視覺化呈現作業的資訊：進度、資料讀取、資料寫入、執行時間、每個節點平均執行時間、輸入輸送量、輸出輸送量。
	* 中繼資料作業。它會顯示所有中繼資料作業。
	* 狀態記錄。
	* 診斷。適用於 Visual Studio 的資料湖工具會自動診斷工作執行。當他們的工作中有一些錯誤或效能問題時您會收到警示。如需詳細資訊，請參閱「工作診斷」(連結 TBD)。

**檢查工作狀態**

1. 在 [伺服器總管] 中，依序展開 [Azure]、[資料湖分析]，以及資料湖分析帳戶名稱。
2. 按兩下 [工作] 以列出工作。
2. 按一下工作以查看狀態。

**查看工作輸出**

1. 在 [伺服器總管] 中，依序展開 [Azure]、[資料湖分析]、您的資料湖分析帳戶，以及 [儲存體帳戶]，然後用滑鼠右鍵按一下預設的資料湖儲存體帳戶，再按一下 [總管]。
2.  按兩下 [輸出] 以開啟資料夾。
3.  按兩下 [SearchLog-From-adltools.csv]。


###工作播放

工作播放可讓您觀看工作執行進度和以視覺化方式偵測出效能異常和瓶頸。這項功能可以在工作完成執行 (亦即在工作正在執行的期間) 之前及完成執行之後使用。工作執行期間進行播放可讓使用者播放截至目前時間為止的進度。

**檢視工作執行進度**

1. 按一下右上角的 [載入設定檔]。請參閱上一個螢幕擷取畫面。
2. 按一下左下角的 [播放] 按鈕以檢閱工作執行進度。
3. 在播放期間，按一下 [暫停] 即可暫停播放；您也可以直接把進度列拖曳至特定的位置。


###熱度圖

適用於 Visual Studio 的資料湖工具提供使用者可選取的工作檢視色彩覆疊，以表示每個階段的進度、資料 I/O、執行時間、I/O 輸送量。透過此方法，使用者可以直接且直覺地找出潛在的問題和工作屬性的分佈。您可以選擇要從下拉式清單中顯示的資料來源。

## 在本機執行 U-SQL

在 Visual Studio 中使用 U-SQL 本機執行體驗，您可以：

- 在本機執行 U-SQL 指令碼以及 C# 組件。
- 在本機偵錯 C# 組件。
- 在伺服器總管中建立/刪除/檢視本機資料庫、組件、結構描述及資料表，就跟您能為 Azure 資料湖分析服務做的一樣。

您將會在 Visual Studio 中看到「本機」帳戶，且安裝程式會建立 *DataRoot* 資料夾 (位於 *C:\\LocalRunRoot*)。DataRoot 資料夾將用於：

- 儲存中繼資料 (包括資料表、資料庫、TVF 等)。
- 針對特定指令碼：如果會在輸入/輸出路徑中參考相對路徑，則我們將查閱 DataRoot (以及指令碼的路徑 (如果它是輸入))
- 如果您嘗試註冊組件並使用相對路徑，就不會參考 DataRoot 資料夾 (如需詳細資訊，請參閱＜進行本機執行時使用組件＞的內容)

下列視訊示範 U-SQL 本機執行功能：

>[AZURE.VIDEO usql-localrun]

### 已知問題與限制

- U-SQL 本機執行不支援在本機查詢檔案組。請參閱 [U-SQL 檔案組](https://msdn.microsoft.com/library/azure/mt621294.aspx)。未來將會解決此問題。
- 效能會因為低平行處理原則而變慢，原因在於工作計劃會在單一處理序中循序執行。
- 本機執行無法在 Visual Studio 中顯示工作圖形。未來將會處理此問題。
- 無法在 [伺服器總管] 中為本機帳戶建立資料表/DB 等項目。
- 參考相對路徑時：

    - 在指令碼輸入 (EXTRACT * FROM “/path/abc”) 中：將同時搜尋 DataRoot 路徑和指令碼路徑。
    - 在指令碼輸出 (OUTPUT TO “path/abc”) 中：將使用 DataRoot 路徑做為輸出資料夾。
    - 在組件註冊 (CREATE ASSEMBLY xyz FROM “/path/abc”) 中：將搜尋指令碼路徑，但不會搜尋 DataRoot。
    - 在註冊的 TVF/檢視或其他中繼資料項目中：將搜尋 DataRoot 路徑，但不會搜尋指令碼路徑。

    針對在資料湖服務上執行的指令碼，預設儲存體帳戶將用來做為根資料夾，並據以進行搜尋。

### 在本機測試 U-SQL 指令碼
如需開發 U-SQL 指令碼的指示，請參閱[開發 U-SQL 指令碼](#develop-and-test-u-sql-scripts)。若要在本機建置並執行 U-SQL 指令碼，請在叢集下拉式清單中選取 [(本機)]，然後按一下 [提交]。請確定您已參考正確的資料：參考絕對路徑，或是將資料放置於 DataRoot 資料夾下。

![在本機提交 U-SQL Visual Studio 專案](./media/data-lake-analytics-data-lake-tools-get-started/data-lake-analytics-data-lake-tools-submit-job-local-run.png)

您也可以用滑鼠右鍵按一下指令碼，然後按一下內容功能表的 [執行本機方案]，或是按下 **CTRL+F5** 來觸發本機執行。

### 在本機執行中使用組件

有兩種方式可以執行自訂的 C# 檔案：

- 在程式碼後置檔案中撰寫組件，而組件將會自動註冊，並在指令碼完成之後卸除。
- 建立 C# 組件專案，並透過類似如下的指令碼，向本機帳戶註冊輸出 dll。請注意，路徑是相對於指令碼，而不是 DataRoot 資料夾。

![在 U-SQL 本機執行中使用組件](./media/data-lake-analytics-data-lake-tools-get-started/data-lake-analytics-data-lake-tools-local-run-assembly.png)

### 在本機偵錯指令碼和 C# 組件

您可以偵錯 C# 組件，而不需提交並向 Azure 資料湖分析服務註冊。您可以在這兩個程式碼後置檔案和參考的 C# 專案中設定中斷點。

**如何為程式碼後置檔案中的本機程式碼偵錯**
1.	在程式碼後置檔案中設定中斷點。
2.	按下 **F5** 以便在本機為指令碼偵錯。

下列程序僅適用於 Visual Studio 2015。在舊版 Visual Studio 中，您可能需要手動加入 pdb 檔案。

**如何為參考的 C# 專案中的本機程式碼偵錯**
1.	建立 C# 組件專案，並建置它來產生輸出 dll。
2.	使用 U-SQL 陳述式來註冊 dll：

        CREATE ASSEMBLY assemblyname FROM @"..\..\path\to\output\.dll";
3.	在 C# 程式碼中設定中斷點。
4.	按下 **F5** 以便在本機為參考 C# dll 的指令碼偵錯。

##另請參閱

若要使用不同的工具開始使用資料湖分析，請參閱：

- [使用 Azure 入口網站開始使用 Data Lake Analytics](data-lake-analytics-get-started-portal.md)
- [使用 Azure PowerShell 開始使用資料湖分析](data-lake-analytics-get-started-powershell.md)
- [使用 .NET SDK 開始使用資料湖分析](data-lake-analytics-get-started-net-sdk.md)
- [在 U-SQL 作業中進行 C# 程式碼偵錯](data-lake-analytics-debug-u-sql-jobs.md)

若要查看更多開發主題：

- [使用資料湖分析來分析 weblog](data-lake-analytics-analyze-weblogs.md)
- [使用適用於 Visual Studio 的資料湖工具開發 U-SQL 指令碼](data-lake-analytics-data-lake-tools-get-started.md)
- [開始使用 Azure 資料湖分析 U-SQL 語言](data-lake-analytics-u-sql-get-started.md)
- [針對資料湖分析工作開發 U-SQL 使用者定義運算子](data-lake-analytics-u-sql-develop-user-defined-operators.md)

##附錄 A：準備教學課程所需的 PowerShell 範例

下列 PowerShell 指令碼會為您準備 Azure Data Lake Analytics 帳戶和來源資料，讓您可以跳到[開發 U-SQL 指令碼](data-lake-analytics-data-lake-tools-get-started.md#develop-u-sql-scripts)一節。

    #region - used for creating Azure service names
    $nameToken = "<Enter an alias>"
    $namePrefix = $nameToken.ToLower() + (Get-Date -Format "MMdd")
    #endregion

    #region - service names
    $resourceGroupName = $namePrefix + "rg"
    $dataLakeStoreName = $namePrefix + "adas"
    $dataLakeAnalyticsName = $namePrefix + "adla"
    $location = "East US 2"
    #endregion


    # Treat all errors as terminating
    $ErrorActionPreference = "Stop"

    #region - Connect to Azure subscription
    Write-Host "`nConnecting to your Azure subscription ..." -ForegroundColor Green
    try{Get-AzureRmContext}
    catch{Login-AzureRmAccount}
    #endregion

    #region - Create an Azure Data Lake Analytics service account
    Write-Host "Create a resource group ..." -ForegroundColor Green
    New-AzureRmResourceGroup `
        -Name  $resourceGroupName `
        -Location $location

    Write-Host "Create a Data Lake account ..."  -ForegroundColor Green
    New-AzureRmDataLakeStoreAccount `
        -ResourceGroupName $resourceGroupName `
        -Name $dataLakeStoreName `
        -Location $location

    Write-Host "Create a Data Lake Analytics account ..."  -ForegroundColor Green
    New-AzureRmDataLakeAnalyticsAccount `
        -Name $dataLakeAnalyticsName `
        -ResourceGroupName $resourceGroupName `
        -Location $location `
        -DefaultDataLake $dataLakeStoreName

    Write-Host "The newly created Data Lake Analytics account ..."  -ForegroundColor Green
    Get-AzureRmDataLakeAnalyticsAccount `
        -ResourceGroupName $resourceGroupName `
        -Name $dataLakeAnalyticsName  
    #endregion

    #region - prepare the source data
    Write-Host "Import the source data ..."  -ForegroundColor Green
    $localFolder = "C:\Tutorials\Downloads" # A temp location for the file.
    $storageAccount = "adltutorials"  # Don't modify this value.
    $container = "adls-sample-data"  #Don't modify this value.

    # Create the temp location  
    New-Item -Path $localFolder -ItemType Directory -Force

    # Download the sample file from Azure Blob storage
    $context = New-AzureStorageContext -StorageAccountName $storageAccount -Anonymous
    $blobs = Azure\Get-AzureStorageBlob -Container $container -Context $context
    $blobs | Get-AzureStorageBlobContent -Context $context -Destination $localFolder

    # Upload the file to the default Data Lake Store account    
    Import-AzureRmDataLakeStoreItem -AccountName $dataLakeStoreName -Path $localFolder"SearchLog.tsv" -Destination "/Samples/Data/SearchLog.tsv"

    Write-Host "List the source data ..."  -ForegroundColor Green
    Get-AzureRmDataLakeStoreChildItem -Account $dataLakeStoreName -Path  "/Samples/Data/"
    #endregion

<!---HONumber=AcomDC_0914_2016--->