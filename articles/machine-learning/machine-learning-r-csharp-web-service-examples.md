<properties 
	pageTitle="以 R 在 Machine Learning 上建置的 Web 服務範例 | Microsoft Azure" 
	description="尋找以 R 程式碼和 Machine Learning 建立的一組實用 Web 服務範例，並發佈至 Azure Marketplace。" 
	keywords="csharp,r 程式碼,web 服務範例"
	services="machine-learning" 
	documentationCenter="" 
	authors="jaymathe" 
	manager="jhubbard" 
	editor="cgronlun"/>

<tags 
	ms.service="machine-learning" 
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="09/14/2016" 
	ms.author="jaymathe"/>


#使用 R 程式碼在 Azure Machine Learning 上建置並發佈至 Microsoft Azure Marketplace 的 Web 服務範例

本文提供使用 Azure Machine Learning 建立並發佈至 Azure Marketplace 的範例 Web 服務。每個 Web 服務範例都附加完整的文件、內嵌可用於測試服務的樣本資料集，並說明使用者如何自行建立類似的服務。

在 Azure Machine Learning Studio 中，使用者可以撰寫 R 程式碼，而且只要按幾下，就可以將其發佈為 Web 服務，以供世界各地的應用程式和裝置取用。


[AZURE.INCLUDE [電腦-學習-免費-試用](../../includes/machine-learning-free-trial.md)]


從產生可提供統計功能的簡單計算機，到建立自訂文字採礦情感分析預測工具，由於 Azure Machine Learning 的使用者可輕鬆操作 R 程式碼，因此不論是剛接觸 R 的使用者，或是有經驗的使用者都可以從中獲益。使用者可透過行動裝置應用程式或網站，來取用這些 Web 服務；不過，這些 Web 服務範例也可用來示範 Machine Learning 如何操作 R 指令碼進行分析，以及如何使用 Machine Learning，來建立採用 R 程式碼的 Web 服務。

每個範例包含取用 Web 服務的 C# 範例。


![Azure Machine Learning 中的 R 程式碼圖表：獨佔使用 R 解決方案，或發佈至 Azure Marketplace。][1]

請考慮下列案例。

##案例 1：泛型模型 
使用者會使用可套用至新使用者資料的泛型模型，例如時間序列資料的基本預測或具備進階分析的自建 R 方法。此使用者會將此模型發行為一項 Web 服務，以便他人取用其資料。



* [二元分類器](machine-learning-r-csharp-binary-classifier.md)
* [叢集模型](machine-learning-r-csharp-cluster-model.md)
* [多變量線性迴歸](machine-learning-r-csharp-multivariate-linear-regression.md)
* [預測 - 指數平滑法](machine-learning-r-csharp-forecasting-exponential-smoothing.md)
* [ETS + STL 預測](machine-learning-r-csharp-retail-demand-forecasting.md)
* [預測 - 自動迴歸整合式移動平均 (ARIMA)](machine-learning-r-csharp-arima.md)
* [存活分析](machine-learning-r-csharp-survival-analysis.md)


##案例 2：定型模型 - 特定資料 
使用者具有可透過 R 程式碼提供實用預測的資料，例如可透過 k-means 演算法叢集以預測使用者性格類型的大型性格問卷樣本，或可透過存活分析 R 封裝預測個人罹患肺癌之風險的健康調查資料。使用者會透過可預測新使用者結果的 Web 服務來發行資料。

##案例 3：定型模型 - 泛型資料 
使用者具有泛型資料 (例如文字主體)，該資料可用於建置 Web 服務，並將其廣泛套用至不同類型的使用個案和案例。

* [語彙型情感分析](machine-learning-r-csharp-lexicon-based-sentiment-analysis.md)

##案例 4：進階計算機 
使用者提供進階計算或模擬，而不需要任何定型模型或將使用者資料套入模型。

* [比例差異測試](machine-learning-r-csharp-difference-in-two-proportions.md)
* [常態分佈套件](machine-learning-r-csharp-normal-distribution.md)
* [二項分配套件](machine-learning-r-csharp-binomial-distribution.md)

##常見問題集
如需取用 Web 服務或發佈至 Marketplace 的常見問題集，請參閱[這裡](machine-learning-marketplace-faq.md)。

[1]: ./media/machine-learning-r-csharp-web-service-examples/machine-learning-r-code-options-for-using-and-sharing-cloud.png


 

<!---HONumber=AcomDC_0921_2016-->