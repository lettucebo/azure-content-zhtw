<properties
	pageTitle="從另一個實驗匯入資料到 Machine Learning Studio | Microsoft Azure"
	description="如何將訓練資料儲存在 Azure Machine Learning Studio 並將它使用於另一個實驗中。"
	keywords="匯入資料,資料,資料來源,定型資料"
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
	ms.author="garye;bradsev" />


# 從另一個實驗匯入資料到 Azure Machine Learning Studio。

[AZURE.INCLUDE [import-data-into-aml-studio-selector](../../includes/machine-learning-import-data-into-aml-studio.md)]


有時候您會想要從實驗取得中繼結果，並且使用做為其他實驗的一部分。若要這樣做，您可以將模組另存為資料集：

1. 按一下您要儲存為資料集的模組輸出。

2. 按一下 [**另存為資料集**]。

3. 系統提示時，請輸入可讓您輕易識別資料集的名稱和說明。

4. 按一下 [**確定**] 核取記號。

儲存完成時，資料集可用於工作區中的任何實驗。您可以在模組調色盤的 [**儲存的資料集**] 清單中找到它。

<!---HONumber=AcomDC_0921_2016-->