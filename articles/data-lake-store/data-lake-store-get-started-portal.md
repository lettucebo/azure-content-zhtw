<properties 
   pageTitle="開始使用資料湖存放區 | Azure" 
   description="使用入口網站來建立資料湖存放區帳戶，並在資料湖存放區中執行基本作業" 
   services="data-lake-store" 
   documentationCenter="" 
   authors="nitinme" 
   manager="jhubbard" 
   editor="cgronlun"/>
 
<tags
   ms.service="data-lake-store"
   ms.devlang="na"
   ms.topic="hero-article"
   ms.tgt_pltfrm="na"
   ms.workload="big-data" 
   ms.date="09/13/2016"
   ms.author="nitinme"/>

# 使用 Azure 入口網站開始使用 Azure 資料湖分析

> [AZURE.SELECTOR]
- [入口網站](data-lake-store-get-started-portal.md)
- [PowerShell](data-lake-store-get-started-powershell.md)
- [.NET SDK](data-lake-store-get-started-net-sdk.md)
- [Java SDK](data-lake-store-get-started-java-sdk.md)
- [REST API](data-lake-store-get-started-rest-api.md)
- [Azure CLI](data-lake-store-get-started-cli.md)
- [Node.js](data-lake-store-manage-use-nodejs.md)

了解如何使用 Azure 入口網站建立 Azure 資料湖存放區帳戶並執行基本作業，例如建立資料夾、上傳和下載資料檔案、刪除您的帳戶等等。如需有關資料湖存放區的詳細資訊，請參閱 [Azure 資料湖存放區概觀](data-lake-store-overview.md)。

## 必要條件

開始進行本教學課程之前，您必須具備下列條件：

- **Azure 訂用帳戶**。請參閱[取得 Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。

## 使用影片快速學習？

請觀看下列影片以開始使用 Data Lake Store。

* [建立 Data Lake Store 帳戶](https://mix.office.com/watch/1k1cycy4l4gen)
* [使用資料總管來管理 Azure Data Lake Store 中的資料](https://mix.office.com/watch/icletrxrh6pc)

## 建立 Azure 資料湖存放區帳戶

1. 登入新的 [Azure 入口網站](https://portal.azure.com)。

2. 按一下 [新增]，然後依序按一下 [資料 + 儲存體] 和 [Azure 資料湖存放區]。讀取 [Azure 資料湖存放區] 刀鋒視窗中的資訊，然後按一下刀鋒視窗左上角的 [建立]。

3. 在 [新增資料湖存放區] 刀鋒視窗中提供值，如以下擷取畫面所示：

	![建立新的 Azure 資料湖存放區帳戶](./media/data-lake-store-get-started-portal/ADL.Create.New.Account.png "建立新的 Azure 資料湖帳戶")

	- **訂用帳戶**。選取您想要建立新 Data Lake Store 帳戶所在的訂用帳戶。
	- **資源群組**。選取現有的資源群組，或按一下 [建立資源群組] 來建立。資源群組是保留應用程式相關資源的容器。如需詳細資訊，請參閱 [Azure 中的資源群組](resource-group-overview.md#resource-groups)。
	- **位置**：選取您要建立資料湖存放區帳戶的位置。

4. 如果您想要資料湖存放區帳戶可透過開始面板存取，請選取 [釘選到開始面板]。

5. 按一下 [建立]。如果您選擇將帳戶釘選到開始面板，您會回到開始面板，並且可以看到資料湖存放區帳戶佈建的進度。一旦佈建資料湖存放區帳戶，帳戶刀鋒視窗就會出現。

6. 展開 [Essentials] 下拉式清單以查看資料湖存放區帳戶的相關資訊，例如所屬資源群組、位置等。按一下 [快速啟動] 圖示以查看資料湖儲存區的相關其他資源連結。

	![您的 Azure 資料湖存放區帳戶](./media/data-lake-store-get-started-portal/ADL.Account.QuickStart.png "您的 Azure 資料湖帳戶")

## <a name="createfolder"></a>在 Azure 資料湖存放區帳戶中建立資料夾

您可以在您的 Azure 資料湖存放區帳戶下建立資料夾，用於管理與存放資料。

1. 開啟您剛建立的資料湖存放區帳戶。從左窗格按一下 [瀏覽]，按一下 [資料湖存放區]，然後從 [資料湖存放區] 刀鋒視窗中，按一下您要在其下建立資料夾的帳戶名稱。如果您將帳戶釘選到開始面板，請按一下該帳戶磚。

2. 在您的 [資料湖存放區帳戶] 刀鋒視窗中，按一下 [資料總管]。

	![在資料湖存放區帳戶中建立資料夾](./media/data-lake-store-get-started-portal/ADL.Create.Folder.png "在資料湖存放區帳戶中建立資料夾")

3. 在您的 [資料湖存放區帳戶] 刀鋒視窗中，按一下 [新增資料夾]，輸入新資料夾的名稱，然後按一下 [確定]。
	
	![在資料湖存放區帳戶中建立資料夾](./media/data-lake-store-get-started-portal/ADL.Folder.Name.png "在資料湖存放區帳戶中建立資料夾")
	
	新建立的資料夾將會列在 [資料總管] 刀鋒視窗中。您可以建立任何層級的巢狀資料夾。

	![在資料湖帳戶中建立資料夾](./media/data-lake-store-get-started-portal/ADL.New.Directory.png "在資料湖帳戶中建立資料夾")


## <a name="uploaddata"></a>將資料上傳至 Azure 資料湖存放區帳戶

您可以直接在根層級將資料上傳至 Azure 資料湖存放區帳戶，或上傳至您在帳戶內建立的資料夾。在下面的螢幕擷取畫面，遵循步驟從 [資料總管] 刀鋒視窗將檔案上傳至子資料夾。在這個螢幕擷取畫面中，檔案會上傳到階層連結所示的子資料夾 (以紅色方塊標示)。

如果您要尋找一些可上傳的範例資料，您可以從 [Azure 資料湖 Git 儲存機制](https://github.com/MicrosoftBigData/usql/tree/master/Examples/Samples/Data/AmbulanceData)取得 **Ambulance Data** 資料夾。

![上傳資料](./media/data-lake-store-get-started-portal/ADL.New.Upload.File.png "上傳資料")


## <a name="properties"></a>儲存資料的屬性和可用的動作

按一下新加入的檔案以開啟 [屬性] 刀鋒視窗。此刀鋒視窗提供檔案相關聯的屬性，以及您可以對檔案執行的動作。您也可以在 Azure 資料湖存放區帳戶中複製檔案的完整路徑，如以下的螢幕擷取畫面中紅色方塊的反白顯示。

![資料上的屬性](./media/data-lake-store-get-started-portal/ADL.File.Properties.png "資料上的屬性")

* 按一下 [預覽] 可直接從瀏覽器查看檔案的預覽。您也可以指定預覽的格式。按一下 [預覽]，按一下 [檔案預覽] 刀鋒視窗中的 [格式]，然後在 [檔案預覽格式] 刀鋒視窗中指定要顯示的資料列數目、使用的編碼、要使用的分隔符號等等選項。

  ![檔案預覽格式](./media/data-lake-store-get-started-portal/ADL.File.Preview.png "檔案預覽格式")

* 按一下 [下載] 將檔案下載到您的電腦。

* 按一下 [重新命名檔案] 來重新命名檔案。

* 按一下 [刪除檔案] 來刪除檔案。


## 保護您的資料

您可以在您的 Azure 資料湖存放區帳戶中使用 Azure Active Directory 和存取控制 (ACL) 保護儲存的資料。如需如何執行此工作的指示，請參閱[在 Azure 資料湖存放區中保護資料](data-lake-store-secure-data.md)。


## 刪除 Azure 資料湖存放區帳戶

若要刪除 Azure 資料湖存放區帳戶，請從 [資料湖存放區] 刀鋒視窗按一下 [刪除]。為了確認此動作，將提示您輸入您要刪除的帳戶名稱。輸入帳戶的名稱，然後按一下 [刪除]。

![刪除資料湖帳戶](./media/data-lake-store-get-started-portal/ADL.Delete.Account.png "刪除資料湖帳戶")


## 後續步驟

- [保護 Data Lake Store 中的資料](data-lake-store-secure-data.md)
- [搭配 Data Lake Store 使用 Azure Data Lake Analytics](../data-lake-analytics/data-lake-analytics-get-started-portal.md)
- [搭配資料湖存放區使用 Azure HDInsight](data-lake-store-hdinsight-hadoop-use-portal.md)
- [存取 Data Lake Store 的診斷記錄](data-lake-store-diagnostic-logs.md)

<!---HONumber=AcomDC_0914_2016--->