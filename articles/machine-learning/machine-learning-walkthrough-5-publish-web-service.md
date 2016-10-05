<properties
	pageTitle="步驟 5：部署 Machine Learning Web 服務 | Microsoft Azure"
	description="開發預測解決方案逐步解說的步驟 5：在 Machine Learning Studio 中將預測實驗部署為 Web 服務。"
	services="machine-learning"
	documentationCenter=""
	authors="garyericson"
	manager="jhubbard"
	editor="cgronlun"/>

<tags
	ms.service="machine-learning"
	ms.workload="data-services"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/16/2016"
	ms.author="garye"/>


# 逐步解說步驟 5：部署 Azure Machine Learning Web 服務

這是[在 Azure Machine Learning 中為信用風險評估開發預測性分析解決方案](machine-learning-walkthrough-develop-predictive-solution.md)逐步解說的第五個步驟


1.	[建立機器學習服務工作區](machine-learning-walkthrough-1-create-ml-workspace.md)
2.	[上傳現有資料](machine-learning-walkthrough-2-upload-data.md)
3.	[建立新實驗](machine-learning-walkthrough-3-create-new-experiment.md)
4.	[訓練及評估模型](machine-learning-walkthrough-4-train-and-evaluate-models.md)
5.	**部署 Web 服務**
6.	[存取 Web 服務](machine-learning-walkthrough-6-access-web-service.md)

----------

為了讓其他人可以使用此逐步解說中開發的預測模型，我們將它部署為 Azure 上的 Web 服務。

截至目前為止，我們一直在試驗如何定型模型。但是已部署的服務不會再定型，它會根據我們的模型，將使用者的輸入計分來產生預測。所以我們要進行一些準備工作，將這項實驗從「訓練」實驗轉換為「預測」實驗。

這是一個兩步驟程序：

1. 將我們所建立的「訓練實驗」轉換成「預測實驗」
2. 將預測實驗部署為 Web 服務

但首先，我們需要更精簡這項實驗。目前我們在實驗中有兩個不同的模型，但將實驗部署為 Web 服務時，我們只要一個模型。

假設我們已判斷出推進式樹狀模型是較適合使用的模型。因此，首要之務是移除[二元支援向量機器][two-class-support-vector-machine]模組，以及用於定型該模組的其他模組。您可能需要按一下實驗畫布底部的 [**另存新檔**]，先建立一份實驗複本。

我們必須刪除下列模組：

- [二元支援向量機器][two-class-support-vector-machine]
- 連接到這個模組的[定型模型][train-model]和[計分模型][score-model]模組
- [標準化資料][normalize-data] \(兩者)
- [評估模型][evaluate-model]

只要選取模組然後按 Delete 鍵，或用右鍵按一下模組並選取 [刪除]。

現在我們已經準備好使用[二元促進式決策樹][two-class-boosted-decision-tree]部署此模型。

## 將訓練實驗轉換為預測實驗

轉換成預測實驗的作業包含三個步驟：

1. 儲存已定型的模型，並取代我們的定型模組
2. 精簡實驗，移除只有定型才需要的模組
3. 定義 Web 服務接受輸入的位置和產生輸出的位置

幸好上述三個步驟只要按一下實驗畫布底部的 [設定 Web 服務] \(選取 [預測 Web 服務] 選項)，即可完成。

當您按一下 [設定 Web 服務]，會發生幾件事：

- 我們所定型的模型會另存為實驗畫布左側模組調色盤中的單一**定型模型**模組 (您可以在 [定型模型] 下找到這個模組)。
- 用於定型的模組會遭到移除。具體而言：
  - [二元促進式決策樹][two-class-boosted-decision-tree]
  - [定型模型][train-model]
  - [資料分割][split]
  - 用於測試資料的第二個[執行 R 指令碼][execute-r-script]模組
- 儲存的定型模型會加回實驗中。
- 加入 **Web 服務輸入**和 **Web 服務輸出**模組。

> [AZURE.NOTE] 實驗已儲存在已加入實驗畫布頂端之索引標籤下的兩個部分：原始的訓練實驗位於 [訓練實驗] 索引標籤底下，新建立的預測實驗位於 [預測實驗] 底下。

我們需要對這項特別的實驗採取一個額外步驟。我們新增了兩個[執行 R 指令碼][execute-r-script]模組來提供加權函數以進行訓練和測試。在最終模型中不需這麼做。當 Machine Learning Studio 移除[分割][split]模組時，它會移除一個[執行 R 指令碼][execute-r-script]模組，如此我們現在就可以移除其他的，並將[中繼資料編輯器][metadata-editor]直接連接到[分數模型][score-model]。

實驗現在看起來如下：

![Scoring the trained model][4]


> [AZURE.NOTE] 您可能驚訝我們為什麼要在預測實驗中留下「UCI 德國信用卡資料」資料集。服務即將使用使用者的資料，而不是原始資料集，所以為何要讓原始資料集留在模型中？
>
>服務的確不需要原始信用卡資料。但卻需要該資料的結構描述，例如，有多少個資料行及哪些資料行是數值等資訊。需要此結構描述資訊才能解譯使用者的資料。我們保持連接這些元件，以便服務執行時，評分模型才會有資料集結構描述。不會使用資料，只是使用結構描述。

最後一次執行實驗 (按一下 [執行])。如果要驗證模型仍然有效，請按一下[評分模型][score-model]模組的輸出結果，並選取 [檢視結果]。您會看到原始資料出現，也會看到信用風險值 ("Scored Labels") 和評分機率值 ("Scored Probabilities").

## 部署 Web 服務

您可以根據 Azure Resource Manager 將實驗部署為傳統 Web 服務或新的 Web 服務。

### 部署為傳統 Web 服務 ###

若要部署衍生自實驗的傳統 Web 服務，請按一下畫布下方的 [部署 Web 服務]，然後選取 [部署 Web 服務 [新]]。Machine Learning Studio 會將實驗當做 Web 服務部署，並帶您前往 Web 服務的儀表板。您可以從此處返回實驗 ([檢視快照] 或 [檢視最新]) 並執行簡單的 Web 服務測試 ([測試] 按鈕 - 請參閱下面的「測試 Web 服務」)。另外這裡還有建立可存取 Web 服務之應用程式的資訊 (此逐步說明的下一個步驟中有該資訊的詳細資料)。

![Web 服務儀表板][6]

您可以按一下 [設定] 索引標籤來設定服務。您可以在這裡修改服務名稱 (預設會指定實驗名稱) 並輸入描述。您也可以為輸入和輸出資料行指定更好記的標籤。

![設定 Web 服務][5]

### 部署為新的 Web 服務

若要部署衍生自實驗的新 Web 服務，請按一下畫布下方的 [部署 Web 服務]，然後按一下 [部署 Web 服務 [新]]。Machine Learning Studio 會帶您到 Azure Machine Learning Web Services 的 [部署實驗] 頁面。

輸入 Web 服務的名稱，然後選擇定價方案。如果您有現有的定價方案，可以進行選取，否則您必須為服務建立新的定價方案。

1.	在 [價格方案] 下拉式清單中，選取現有的方案或選取 [選取新的方案] 選項。
2.	在 [方案名稱] 中，輸入將識別您帳單上方案的名稱。
3.	選取其中一個 [每月方案層]。請注意，方案層預設為您預設區域的方案，您的 Web 服務會部署到該區域。

按一下 [部署]，Web 服務的 [快速入門] 頁面就會開啟。

您可以按一下 [設定] 功能表選項來設定服務。您可以在這裡修改服務名稱 (預設會指定實驗名稱) 並輸入描述。您也可以啟用或停用錯誤記錄和可用來測試服務的範例資料。

若要測試 Web 服務，請按一下 [測試] 功能表選項 (請參閱下面的「測試 Web 服務」)。如需建立應用程式以存取 Web 服務的資訊，請按一下 [取用] 功能表選項 (此逐步說明的下一個步驟中有該資訊的詳細資料)。


> [AZURE.TIP] 您可以在部署 Web 服務之後進行更新。例如，如果想要變更模型，只要編輯訓練實驗、調整模型參數，然後按一下 [部署 Web 服務]。然後選取 [部署 Web 服務 [傳統]] 或 [部署 Web 服務 [新]]。重新部署實驗時，將會取代 Web 服務 (現在使用的是已更新的模型)。


## 測試 Web 服務

**測試傳統 Web 服務** 在 [儀表板] 頁面上，按一下 [預設端點] 下的 [測試] 按鈕。即會顯示對話方塊，要求您提供服務的輸入資料。這些就是在原始的德國信用風險資料集中出現的資料行。

輸入一組資料，然後按一下 [確定]。

在 Web 服務中，資料透過 **Web 服務輸入**模組輸入，經由[中繼資料編輯器][metadata-editor]模組，然後到計分所在的[分數模型][score-model]。接著會透過 **Web 服務輸出**從 Web 服務輸出結果。

**測試新的 Web 服務**

在 Azure Machine Learning Web Services 入口網站中，按一下頁面頂端的 [測試] 功能表選項。[測試] 頁面會開啟，您可以輸入服務的資料。顯示的輸入欄位會對應到在原始的德國信用風險資料集中出現的資料行。

輸入一組資料，然後按一下 [測試要求-回應]。

您也可以啟用範例資料，用來測試要求-回應服務。如果您是在 Machine Learning Studio 中建立 Web 服務，範例資料會取自您用來訓練模型的資料。

測試的結果將會顯示在頁面右手邊的輸出資料行。

> [AZURE.TIP] 我們設定預測性實驗的方式，會傳回[分數模型][score-model]模組的整個結果。這包括所有輸入的資料以及信用風險值和評分機率。如果您想傳回其他內容 (例如只要信用風險值)，那麼您可以在[分數模型][score-model]和 **Web 服務輸出**之間插入[專案資料行][project-columns]模組，來排除不想讓 Web 服務傳回的資料行。

## 管理 Web 服務

**管理傳統 Web 服務**

一旦部署傳統 Web 服務之後，您就可以從 [Azure 傳統入口網站](https://manage.windowsazure.com)管理它。

1. 登入 [Azure 傳統入口網站](https://manage.windowsazure.com)。
2. 在 Microsoft Azure 服務面板中，按一下 [機器學習]。
3. 按一下您的工作區。
4. 按一下 [**Web 服務**] 索引標籤。
5. 按一下剛建立的 Web 服務。
6. 按一下 [預設] 端點。

從這裡您可以進行像是監視 Web 服務的執行狀況，和透過變更服務可處理的同時呼叫數目來調整效能的作業。您甚至可以在 Azure Marketplace 中發佈您的 Web 服務。

如需詳細資訊，請參閱：

- [建立端點](machine-learning-create-endpoint.md)
- [調整 Web 服務](machine-learning-scaling-webservice.md)
- [將 Azure Machine Learning Web 服務發佈至 Azure Marketplace](machine-learning-publish-web-service-to-azure-marketplace.md)

**管理新的 Web 服務**

一旦部署新的 Web 服務之後，您就可以從 [Azure Machine Learning Web Services 入口網站](https://servics.azureml.net)管理它。

監視 Web 服務的效能：
1. 登入 [Azure Machine Learning Web Services 入口網站](https://servics.azureml.net)。
2. 按一下 [Web 服務]。
3. 按一下您的 Web 服務。
4. 按一下 [儀表板]。

----------

**下一步：[存取 Web 服務](machine-learning-walkthrough-6-access-web-service.md)**

[1]: ./media/machine-learning-walkthrough-5-publish-web-service/publish1.png
[2]: ./media/machine-learning-walkthrough-5-publish-web-service/publish2.png
[3]: ./media/machine-learning-walkthrough-5-publish-web-service/publish3.png
[4]: ./media/machine-learning-walkthrough-5-publish-web-service/publish4.png
[5]: ./media/machine-learning-walkthrough-5-publish-web-service/publish5.png
[6]: ./media/machine-learning-walkthrough-5-publish-web-service/publish6.png


<!-- Module References -->
[evaluate-model]: https://msdn.microsoft.com/library/azure/927d65ac-3b50-4694-9903-20f6c1672089/
[execute-r-script]: https://msdn.microsoft.com/library/azure/30806023-392b-42e0-94d6-6b775a6e0fd5/
[metadata-editor]: https://msdn.microsoft.com/library/azure/370b6676-c11c-486f-bf73-35349f842a66/
[normalize-data]: https://msdn.microsoft.com/library/azure/986df333-6748-4b85-923d-871df70d6aaf/
[score-model]: https://msdn.microsoft.com/library/azure/401b4f92-e724-4d5a-be81-d5b0ff9bdb33/
[split]: https://msdn.microsoft.com/library/azure/70530644-c97a-4ab6-85f7-88bf30a8be5f/
[train-model]: https://msdn.microsoft.com/library/azure/5cc7053e-aa30-450d-96c0-dae4be720977/
[two-class-boosted-decision-tree]: https://msdn.microsoft.com/library/azure/e3c522f8-53d9-4829-8ea4-5c6a6b75330c/
[two-class-support-vector-machine]: https://msdn.microsoft.com/library/azure/12d8479b-74b4-4e67-b8de-d32867380e20/
[project-columns]: https://msdn.microsoft.com/zh-TW/library/azure/1ec722fa-b623-4e26-a44e-a50c6d726223/

<!---HONumber=AcomDC_0921_2016-->