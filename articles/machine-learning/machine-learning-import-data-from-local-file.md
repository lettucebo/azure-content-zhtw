<properties
	pageTitle="將本機檔案的資料匯入 Machine Learning Studio | Microsoft Azure"
	description="如何從本機檔案將訓練資料匯入 Azure Machine Learning Studio。"
	keywords="匯入資料、資料格式、資料類型、資料來源、定型資料"
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


# 從本機檔案將訓練資料匯入至 Azure Machine Learning Studio

[AZURE.INCLUDE [import-data-into-aml-studio-selector](../../includes/machine-learning-import-data-into-aml-studio.md)]


若要在 Machine Learning Studio 中使用您自己的資料，您可以事先從本機硬碟上傳資料檔案，以在工作區建立資料集模組。


## 從本機檔案匯入資料

您可以透過執行下列動作，從本機硬碟匯入資料：

1. 在 [Machine Learning Studio] 視窗底部，按一下 [**+新增**]。
2. 選取 [**資料集**] 和 [**從本機檔案**]。
3. 在 [**上傳新的資料集**] 對話方塊中，瀏覽至您要上傳的檔案。
4. 輸入名稱、識別資料類型，然後選擇性地輸入說明。建議輸入說明 - 它可以讓您記錄資料的任何特性，您會希望在未來使用資料時記住這些特性。
5. 核取方塊 [**這是現有資料集的新版本**] 可讓您使用新資料更新現有資料集。只要按一下此核取方塊然後輸入現有資料集的名稱即可。

在上傳期間，您會看見訊息，指出您的檔案正在上傳。上傳時間取決於資料大小和連接至服務的速度。如果您知道上傳檔案將耗費很長的時間，您可以在等待時在 ML Studio 中執行其他動作。但是，關閉瀏覽器會導致資料上傳失敗。

資料上傳之後，會儲存在資料集模組中，並且可供工作區中的任何實驗使用。當您在編輯實驗時，您可以在模組調色盤的 [儲存的資料集] 清單底下的 [我的資料集] 清單內，找到您所建立的資料集。當您想將資料集用於進一步的分析和機器學習時，可以在實驗畫布上拖放資料集。

<!---HONumber=AcomDC_0921_2016-->