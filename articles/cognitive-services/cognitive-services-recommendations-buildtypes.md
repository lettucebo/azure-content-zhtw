<properties
	pageTitle="快速入門指南：Machine Learning 建議 API | Microsoft Azure"
	description="Azure Machine Learning Recommendations--快速入門指南"
	services="cognitive-services"
	documentationCenter=""
	authors="luiscabrer"
	manager="jhubbard"
	editor="cgronlun"/>

<tags
	ms.service="cognitive-services"
	ms.workload="data-services"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/20/2016"
	ms.author="luisca"/>

#  組建類型與模型品質 #

<a name="TypeofBuilds"></a>
## 支援的組建類型 ##

建議 API 目前支援兩種組建類型︰「建議」和「FBT」。這兩種類型各使用不同的演算法來建立，並各具不同優點。本文件說明這其中的每一個組建，以及用來比較所產生之模型品質的技術。

如果您尚未這樣做，我們建議您先完成[快速入門指南](cognitive-services-recommendations-quick-start.md)。

<a name="RecommendationBuild"></a>
### 建議組建類型 ###

「建議」組建類型使用矩陣分解來提供建議。它會根據交易來產生[潛伏功能](https://en.wikipedia.org/wiki/Latent_variable)向量以描述每個項目，然後使用這些潛伏的向量來比較類似的項目。

若您根據在您的電子產品商店中所做的採購來定型模型，並提供 Lumia 650 手機做為模型的輸入，則模型會傳回可能購買 Lumia 650 手機的人通常會購買的一組項目。這些項目可能不互補。在此範例中，模型可能會傳回其他手機，因為喜歡 Lumia 650 的人可能會喜歡其他手機。

建議組建有兩項功能可讓它更具吸引力︰

**建議組建支援「冷項目」放置**

沒有太大用途的項目被稱為冷項目。例如，如果您收到一批未曾銷售過的某款手機，系統無法單獨推斷此產品的交易建議。這表示系統應該了解產品本身的相關資訊。

如果您想要使用冷項目放置，則需要提供目錄中各個項目的功能資訊。以下是目錄前幾行的可能樣貌 (請注意功能的「索引鍵=值」格式)。

>6CX-00001,Surface Pro2, Surface,, Type=Hardware, Storage=128 GB, Memory=4G, Manufacturer=Microsoft

>73H-00013,Wake Xbox 360,Gaming,, Type=Software, Language=English, Rating=Mature

>WAH-0F05,Minecraft Xbox 360,Gaming,, * Type=Software, Language=Spanish, Rating=Youth

您還需要設定下列組建參數︰

| 組建參數 | 注意事項
|------------------     |-----------
|*useFeaturesInModel* | 設定為 **True**。指出是否可以使用功能以加強建議模型。
|*allowColdItemPlacement* | 設定為 **True**。指出建議是否也應該透過功能相似度推入冷項目。
| *modelingFeatureList* | 使用於建議組建中的功能名稱逗號分隔清單，可加強建議。例如，上述範例中的 “Language,Storage”。

**建議組建支援使用者建議**

建議組建支援[使用者建議](https://westus.dev.cognitive.microsoft.com/docs/services/Recommendations.V4.0/operations/56f30d77eda5650db055a3dd)。這表示它可以根據使用者的交易歷程記錄為其提供個人化的建議。對於使用者建議，您可以提供該使用者的使用者識別碼或最近的交易歷程記錄。

其中一個您可能會想要套用使用者建議的典型範例是在登入時的歡迎頁面上。您可以在那裡推銷適用於特定使用者的內容。

您可能也想要在使用者即將簽出時套用建議組建類型。這時候，您會有使用者即將購買的項目清單，而您可以根據目前的購物籃提供建議。

#### 建議組建參數

| Name | 	說明 |	 類型，<br>有效值 <br>(預設值)
|-------|-------------------|------------------
| *NumberOfModelIterations* |	整體運算時間和模型精確度會反映模型執行反覆運算的次數。數字越高，模型就越精確，但計算時間也會拉長。 |	 整數，<br>10 到 50 <br>(40)
| *NumberOfModelDimensions* |	維度的數目與功能的數目相關，模型會嘗試尋找資料中的數目。增加維度的數目將允許結果進一步微調成較小的叢集。不過，太多維度會讓模型無法尋找項目之間的關聯。 |	整數，<br>10 到 40 <br>(20) |
| *ItemCutOffLowerBound* |	定義項目最少應該位於多少個使用點，才能被視為模型的一部分。 |		整數，<br>2 或以上<br> (2) |
| *ItemCutOffUpperBound* | 	定義項目最多應該位於多少個使用點，才能被視為模型的一部分。 | 整數，<br>2 或以上<br> (2147483647) |
|*UserCutOffLowerBound* |	定義使用者最少必須已執行多少次交易，才能被視為模型的一部分。 |	整數，<br>2 或以上<br> (2)
| *UserCutOffUpperBound* |	定義使用者最多必須已執行多少次交易，才能被視為模型的一部分。 |	整數，<br>2 或以上<br> (2147483647)|
| *UseFeaturesInModel* |	指出是否可以使用功能以加強建議模型。 | 	 布林值<br> 預設值︰True
|*ModelingFeatureList* |	使用於建議組建中的功能名稱逗號分隔清單，可加強建議。它取決於重要的功能。 |	字串，最多 512 個字元
| *AllowColdItemPlacement* |	指出建議是否也應該透過功能相似度推入冷項目。 | 布林值<br> 預設值︰False
| *EnableFeatureCorrelation* | 指出功能是否可用於推論。 |	布林值<br> 預設值︰False
| *ReasoningFeatureList* |	用於推論句 (例如建議說明) 的功能名稱逗號分隔清單。它取決於對客戶而言很重要的功能。 | 字串，最多 512 個字元
| *EnableU2I* |	啟用個人化的建議，也稱為「使用者對項目 (U2I)」建議。 | 布林值<br> 預設值︰True
|*EnableModelingInsights* |	定義是否應該執行離線評估，才能收集模型深入解析 (也就是精確度和多樣性度量)。若設為 true，就不會將資料的子集用於訓練，因為要將它保留來供測試模型使用。深入了解[離線評估](#OfflineEvaluation)。 | 布林值<br> 預設值︰False
| *SplitterStrategy* | 如果將啟用模型深入解析設為「true」，則這是應基於評估目的用來分割資料的方式。 | 字串，「RandomSplitter」或「LastEventSplitter」<br>預設值︰RandomSplitter


<a name="FBTBuild"></a>
### FBT 組建類型 ###

人氣組合 (FBT) 組建會進行分析，以計算兩項或三項不同產品一起共同出現的次數。然後根據相似度函式 (**co-occurrences**、**Jaccard**、**lift**) 將集合排序。

請將 **Jaccard** 和 **lift** 視為標準化 Co-occurrences 的方法。這表示只有在項目與種子項目一起購買時才會傳回這些項目。

在 Lumia 650 手機範例中，只有在與 Lumia 650 手機相同的工作階段中購買手機 X 時，才會傳回手機 X。因為這不太可能發生，所以我們會預期傳回 Lumia 650 的補充項目；例如，螢幕保護裝置或 Lumia 650 的電源配接器。

目前，如果有兩個具有相同使用者識別碼和時間戳記的項目出現在一筆交易中，則假設會在相同的工作階段中購買這兩個項目。

FBT 組建目前不支援冷項目，因為依照定義，它會預期同一筆交易中購買了兩個項目。FBT 組建可傳回項目集 (三個一組)，但因為它們接受以單一種子項目做為輸入，所以不支援個人化建議。


#### FBT 組建參數

| Name | 	說明 |		類型，<br>有效值 <br>(預設值)
|-------|---------------|-----------------------
| *FbtSupportThreshold* | 模型的保守程度。模型化時要考量項目的共同出現次數。 | 整數，<br>3-50 <br>(6)
| *FbtMaxItemSetSize* | 頻繁集合中的項目數界限。| 整數，<br>2-3 <br>(2)
| *FbtMinimalScore* | 頻繁集合應該具有的最低分數，以包含在傳回的結果中。愈高愈好。 | 雙精準數 <br> 0 以上 <br> (0)
| *FbtSimilarityFunction* | 定義組建所使用的相似度函式。**Lift** 有利於機緣巧合、**co-occurrence** 有助於可預測性，而 **Jaccard** 可在兩者間取得一個很好的平衡。 | 字串，<br> <i>cooccurrence、lift、jaccard</i><br> 預設值︰ <i>jaccard</i>

<a name="SelectBuild"></a>
## 組件評估和選取 ##

本指引可協助您判斷是否應該使用建議組建或 FBT 組建，但是在您可以使用其中任一組建的情況下，它不會提供明確的答案。此外，即使您知道要使用 FBT 組建類型，但仍可能想要選擇使用 **Jaccard** 或 **lift** 做為相似度函式。

在這兩種組建之間進行選取的最佳方式就是在真實世界中進行測試 (線上評估版)，並追蹤不同組建的轉換率。根據建議點選次數、來自顯示建議的實際購買數目，或甚至是顯示不同建議時的實際購買量，可以測量轉換率。您可以根據您的商務目標來選取轉換率計量。

在某些情況下，建議您在模型投入生產之前，先離線評估模型。雖然離線評估無法取代線上評估，但可以做為度量。

<a name="OfflineEvaluation"></a>
## 離線評估  ##

離線評估的目的是要預測精確度 (會購買其中一個建議項目的使用者數目) 和建議的多樣性 (建議的項目數)。在精確度和多樣性度量評估中，系統會找出使用者樣本，並將這些使用者的交易分成兩個群組︰訓練資料集和測試資料集。

> [AZURE.NOTE] 若要使用離線度量，您的使用量資料中必須有時間戳記。必須有時間資料，才能正確分割訓練和測試資料集之間的使用量。

> 此外，離線評估可能不會產生小型使用量檔案的結果。為了徹底評估，測試資料集中應該至少有 1000 個使用點。

<a name="Precision"></a>
### Precision-at-k ###
下表代表 precision-at-k 離線評估的輸出。

| K | 1 | 2 | 3 | 	4 | 	5
|---|---|---|---|---|---|
|百分比 |	13\.75 |	18\.04 | 21 |	24\.31 |	26\.61
|測試的使用者 |	10,000 |	10,000 |	10,000 |	10,000 |	10,000
|考量的使用者 |	10,000 |	10,000 |	10,000 |	10,000 |	10,000
|不考量的使用者 |	0 |	0 |	0 |	0 |	0

#### K
在上表中，「K」代表要向客戶顯示的建議數目。資料表顯示的意思如下：「如果在測試期間只對客戶顯示了一個建議，則只有 13.75 的使用者會購買該項建議」。 此陳述的根據是模型是以購買資料來定型的假設。另一種說法是「K=1 時的精確度」是 13.75。

您發現，對客戶顯示的項目愈多，客戶購買建議項目的可能性愈大。在上述實驗中，建議 5 個項目時的機率幾乎倍增至 26.61%。

#### 百分比
與至少一個顯示的「k」建議互動的使用者百分比。此百分比的計算方式：將與至少一個建議互動的使用者數目除以考量的使用者總數。如需詳細資訊，請參閱考量的使用者。

#### 測試的使用者
此資料列中的資料代表測試資料集中的使用者總數。

#### 考量的使用者
只有在系統根據使用訓練資料集所產生的模型而建議至少「k」個項目時，使用者才會納入考量。

#### 不考量的使用者
此資料列中的資料代表任何不考量的使用者。未收到至少「k」個建議項目的使用者。

不考量的使用者 = 測試的使用者 – 考量的使用者

<a name="Diversity"></a>
### 多樣性 ###
多樣性計量會測量建議的項目類型。下表代表多樣性離線評估的輸出。

|百分位數值區 |	0-90| 90-99| 99-100
|------------------|--------|-------|---------
|百分比 | 34\.258 | 55\.127| 10\.615


建議的項目總數：100,000

建議的唯一項目數：954

#### 百分位數值區
每個百分位數值區都是以一個範圍 (介於 0 與 100 之間的最小值和最大值) 表示。接近 100 的項目是最受歡迎的項目，而接近 0 的項目則最不受歡迎。比方說，如果 99-100 百分位數值區的百分比值為 10.6，這表示 10.6% 的建議只傳回前 1% 最受歡迎的項目。百分位數值區最小值會包含在內，而最大值則是排除在外 (但 100 除外)。
#### 建議的唯一項目數
建議的唯一項目數度量會顯示傳回進行評估的不同項目數目。
#### 建議的項目總數
建議的項目總數度量會顯示建議的項目數目。有些項目可能會重複。

<a name="ImplementingEvaluation"></a>
### 離線評估度量 ###
精確度和多樣性離線度量可能有助於您選取要使用的組建。在建置階段，於個別的 FBT 或建議組建參數中︰

-	將「enableModelingInsights」組建參數設為 **true**。
-	(選擇性) 選取「splitterStrategy」(「RandomSplitter」或「LastEventSplitter」)。「RandomSplitter」會根據指定的「randomSplitterParameters」測試百分比和隨機種子值，分割訓練和測試集中的使用量資料。「LastEventSplitter」會根據每個使用者的最後一筆交易，分割訓練和測試集中的使用量資料。

這會觸發一個組建，該組建僅使用一部分的資料進行訓練，並使用其餘資料來計算評估度量。在組建完成之後，若要取得評估的輸出，您必須呼叫[取得組建計量 API](https://westus.dev.cognitive.microsoft.com/docs/services/Recommendations.V4.0/operations/577eaa75eda565095421666f)，並傳遞個別的「modelId」和「buildId」。

 以下是範例評估的 JSON 輸出。


    {
     "Result": {
     "precisionItemRecommend": null,
     "precisionUserRecommend": {
      "precisionMetrics": [
        {
          "k": 1,
          "percentage": 13.75,
          "usersInTest": 10000,
          "usersConsidered": 10000,
          "usersNotConsidered": 0
        },
        {
          "k": 2,
          "percentage": 18.04,
          "usersInTest": 10000,
          "usersConsidered": 10000,
          "usersNotConsidered": 0
        },
        {
          "k": 3,
          "percentage": 21.0,
          "usersInTest": 10000,
          "usersConsidered": 10000,
          "usersNotConsidered": 0
        },
        {
          "k": 4,
          "percentage": 24.31,
          "usersInTest": 10000,
          "usersConsidered": 10000,
          "usersNotConsidered": 0
        },
        {
          "k": 5,
          "percentage": 26.61,
          "usersInTest": 10000,
          "usersConsidered": 10000,
          "usersNotConsidered": 0
        }
      ],
      "error": null
    },
    "diversityItemRecommend": null,
    "diversityUserRecommend": {
      "percentileBuckets": [
        {
          "min": 0,
          "max": 90,
          "percentage": 34.258
        },
        {
          "min": 90,
          "max": 99,
          "percentage": 55.127
        },
        {
          "min": 99,
          "max": 100,
          "percentage": 10.615
        }
      ],
      "totalItemsRecommended": 100000,
      "uniqueItemsRecommended": 954,
      "uniqueItemsInTrainSet": null,
      "error": null
      }
     },
    "Id": 1,
    "Exception": null,
    "Status": 5,
    "IsCanceled": false,
    "IsCompleted": true,
    "CreationOptions": 0,
    "AsyncState": null,
    "IsFaulted": false
    }

<!---HONumber=AcomDC_0921_2016-->