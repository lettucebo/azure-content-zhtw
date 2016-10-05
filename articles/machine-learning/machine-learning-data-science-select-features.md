<properties
	pageTitle="Team Data Science Process 中的特徵選取 | Microsoft Azure" 
	description="說明機器學習服務的資料增強程序中功能選取的目的，並提供其角色的範例。"
	services="machine-learning"
	documentationCenter=""
	authors="bradsev"
	manager="jhubbard"
	editor="cgronlun"/>

<tags
	ms.service="machine-learning"
	ms.workload="data-services"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/19/2016"
	ms.author="zhangya;bradsev" />


# Team Data Science Process (TDSP) 中的特徵選取

本文說明機器學習服務的資料增強程序中特徵選取的目的，並提供其角色的範例。這些範例是根據 Azure Machine Learning Studio 繪製。

[AZURE.INCLUDE [machine-learning-free-trial](../../includes/machine-learning-free-trial.md)]


本主題說明機器學習服務的資料增強程序中特徵選取的目的，並提供其角色的範例。這些範例是根據 Azure Machine Learning Studio 繪製。

特徵的工程設計與選取是[何謂 Team Data Science Process？](data-science-process-overview.md)中所概述 TDSP 程序的其中一部分。特徵工程設計和選取屬於 TDSP 的**開發特徵**步驟。

* **特性工程設計**：此程序嘗試從資料中的現有原始特性建立其他相關特性，以及增加學習演算法的預測功效。

* **特性選取**：此程序嘗試選取主要的原始資料特性子集，以縮小定型問題的維度。

通常會先套用**功能工程設計**以產生其他功能，然後執行**功能選取**步驟以排除不相關、多餘或高度相關的功能。


## 從您的資料篩選特性 - 特性選取 

特性選取程序通常適用於定型資料集的建構，以便進行預測性建模工作，例如分類或迴歸工作。其目的在於從原始資料集中選取一小組特性，使用極小一組的特性來代表資料中的最大變異量，藉此縮小其維度。因此，此特性子集是要用於定型模型的唯一特性。特性選取有兩個主要目的。

* 第一，特性選取通常會排除不相關、多餘或高度相關的特性，進而提高分類正確性。
* 第二，減少特性數目，讓模型定型程序更有效率。對於定型代價昂貴的學習者 (例如支援向量機器) 而言，這格外重要。

雖然特性選取設法要減少資料集中用於定型模型的特性數目 ，但通常不是指「維度縮減」。特性選取方法會擷取資料中的原始特性子集，但不會加以變更。維度縮減方法會運用經過工程設計的特性，轉換原始特性並加以修改。維度縮減方法的範例包含主成分分析、典型相關分析和奇異值分解。

監督環境中有一個廣泛應用的特性選取方法類別，稱之為「以篩選為基礎的特性選取」。這些方法會評估每個特性與目標屬性之間的相關性，套用統計量值以將評分指派給每個特性。接著會依分數將特性排名，而分數可用來設定保留或排除特定特性的臨界值。這些方法中使用的統計量值範例包含皮耳森相關、相互資訊和卡方檢定。

Azure Machine Learning Studio 中有針對特性選取而提供的模組。如下圖所示，這些模組包含[以篩選為基礎的特性選取][filter-based-feature-selection]和[費雪線性判別分析][fisher-linear-discriminant-analysis]。

![特性選取範例](./media/machine-learning-data-science-select-features/feature-Selection.png)


例如，請考慮使用[以篩選為基礎的特性選取][filter-based-feature-selection]模組。為了方便起見，我們繼續使用上述的文字採礦範例。假設在透過[特性雜湊][feature-hashing]模組建立一組 256 個特性之後，我們想要建立一個迴歸模型，其應變數為 "Col1" 並代表 1 至 5 的書籍評論評比。將 [特性評分方法] 設定為 [皮耳森相關]，則 [目標欄] 會是 "Col1"，而 [所需的特性數] 會是 50。然後，[以篩選為基礎的特性選取][filter-based-feature-selection]模組會產生一個包含 50 個特性且目標屬性為 "Col1" 的資料集。下圖顯示此實驗的流程以及我們剛才描述的輸入參數。

![特性選取範例](./media/machine-learning-data-science-select-features/feature-Selection1.png)

下圖顯示結果產生的資料集。每個特性都是根據本身與目標屬性 "Col1" 之間的皮耳森相關進行評分。系統會保留最高分的特性。

![特性選取範例](./media/machine-learning-data-science-select-features/feature-Selection2.png)

下圖顯示所選特性的對應評分。

![特性選取範例](./media/machine-learning-data-science-select-features/feature-Selection3.png)

套用[以篩選為基礎的特性選取][filter-based-feature-selection]模組，以選取 256 個特性中的 50 個特性，因為根據「皮耳森相關」評分方法，其具有目標變數 "Col1" 的最相關特性。

## 結論
功能工程設計和選取是兩種常見的工程設計和選取的功能，可提高下列定型程序的效率：嘗試擷取資料中內含的重要資訊。此外，還可改善這些模型的功效，正確地分類輸入資料以及更精確地預測感興趣的結果。特性工程設計和選取也可結合起來，讓學習更易於以運算方式處理。其作法是提高而後減少校正或定型模型所需的特性數量。從數學的角度來看，選取用來定型模型的特性是極小的一組獨立變數，可供解釋資料中的模式，然後成功地預測結果。

請注意，不一定要執行特性工程設計和特性選取。需要與否取決於我們所擁有或收集的資料、我們挑選的演算法，以及實驗的目標。

<!-- Module References -->
[feature-hashing]: https://msdn.microsoft.com/library/azure/c9a82660-2d9c-411d-8122-4d9e0b3ce92a/
[filter-based-feature-selection]: https://msdn.microsoft.com/library/azure/918b356b-045c-412b-aa12-94a1d2dad90f/
[fisher-linear-discriminant-analysis]: https://msdn.microsoft.com/library/azure/dcaab0b2-59ca-4bec-bb66-79fd23540080/
 

<!---HONumber=AcomDC_0921_2016-->