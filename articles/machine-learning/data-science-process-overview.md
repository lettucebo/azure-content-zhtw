<properties
	pageTitle="何謂 Team Data Science Process？| Microsoft Azure"
	description="Team Data Science Process 是一種系統化的方法，可建置運用進階分析的智慧型應用程式。"
	keywords="資料科學程序, 資料科學小組"
	services="machine-learning"
	documentationCenter=""
	authors="bradsev"
	manager="jhubbard"
	editor="cgronlun" />

<tags
	ms.service="machine-learning"
	ms.workload="data-services"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/19/2016"
	ms.author="bradsev" />


# 何謂 Team Data Science Process (TDSP)？

Team Data Science Process (TDSP) 提供系統化的方式建置智慧型應用程式，讓資料科學家小組可在將這些應用程式轉換為產品所需的完整活動生命週期內，有效率地共同作業。

具體來說，TDSP 目前可為資料科學小組提供下列各項：

- **方法論**︰概述定義開發週期的一系列步驟，以提供如何定義問題、分析相關資料、建置及評估預測模型，然後在企業應用程式中部署這些模型的指引。
- **資源**︰可簡化設定適用於資料科學活動環境的工具和技術 (例如資料科學 VM)，以及適用於加入新技術的實用指引。

以下是 Team Data Science Process 的開發週期：

![圖表︰小組的資料科學程序](./media/data-science-process-overview/data-science-process-for-teams-diagram.png)


此程序是**反覆執行的**：對於模型能有新的和現有的了解或精確調整，可發展出、同時也有賴於先前依序完成的修改步驟。現有的組織開發和專案規劃程序**可輕易調整**，以使用 TDSP 定義的步驟順序。

程序中的步驟已在 [TDSP 學習路徑](https://azure.microsoft.com/documentation/learning-paths/data-science-process/)中以圖表和連結方式呈現，如下所述。


## 規劃與準備步驟

## P1.商務和技術規劃

定義您的商業目標和問題，是展開分析專案的第一步。這些項目會以**商業需求**的形式指定。此步驟的主要目標，是要識別分析需要哪些主要商業變數 (例如銷售預測，或訂單屬詐騙性質的機率) 進行預測以達成這些需求。接著通常需要進一步的規劃，以嘗試從分析的觀點了解達成專案目標所需的**資料來源**。舉例來說，我們常會發現現有的系統必須收集並記錄其他類型的資料，才能解決問題並達成專案目標。如需指引，請參閱[針對 Team Data Science Process 規劃您的環境](machine-learning-data-science-plan-your-environment.md)和[在 Azure 機器學習中的進階分析案例](machine-learning-data-science-plan-sample-scenarios.md)。


## P2.規劃和準備基礎結構

Team Data Science Process 的分析環境包含數個元件：

- **資料工作區**，用來暫存分析和建立模型所需的資料
- **處理基礎結構**，用於資料的前置處理、瀏覽和模型化
- **執行階段基礎結構**，用來運作分析模型，及執行取用模型的智慧型用戶端應用程式。

必須設定的分析基礎結構通常是環境中不屬於核心作業系統的部分。但它通常會運用來自企業內多個系統的資料，以及來自公司外部的資料。分析基礎結構可以純雲端架構或內部部署設定，或是兩者的混合。如需選項，請參閱[設定用於 Team Data Science Process 中的資料科學環境](machine-learning-data-science-environment-setup.md)。


## 分析步驟：  

## 1\.將資料內嵌於資料平台

第一個步驟是從各種來源 (企業內部或外部) 將相關資料導入預計用來處理資料的分析環境中。來源的資料**格式**可能不同於目的地所需的格式。因此，嵌入工具也可能必須完成某些資料轉換。如需選項，請參閱[將資料載入至儲存體環境進行分析](machine-learning-data-science-ingest-data.md)

除了初始的資料擷取以外，許多智慧型應用程式還需要在進行中的學習過程中定期重新整理資料。這可以透過設定**資料管線**或工作流程來完成。這構成了程序中反覆執行的部分，包括會重新建置和重新評估部署解決方案的智慧型應用程式所使用的分析模型。例如，請參閱[使用 Azure Data Factory 從內部部署的 SQL Server 移動資料至 SQL Azure](machine-learning-data-science-move-sql-azure-adf.md)。


## 2\.探索資料並加以視覺化

下一個步驟藉由調查資料的**摘要統計資料**、關聯性和使用**視覺效果**之類的技術，來深入了解資料。這也是處理**資料品質**及完整性問題之處，例如遺漏值、資料類型不相符和資料關聯性不一致等。必須先使用前置處理轉換來清理原始資料，才能進一步執行分析和模型化。如需說明，請參閱[工作以準備資料增強機器學習](machine-learning-data-science-prepare-data.md)。


## 3\.產生特徵並加以選取

與網域專家合作的資料科學家，必須識別可擷取資料集的顯著屬性以及最適合用來預測計劃期間所識別之重要商業變數的特性。這些新特性可以衍生自現有資料，或可能需要收集其他資料。此程序稱為**特性工程**，這是建置有效預測性分析系統的重要步驟之一。要執行此步驟，必須有創意地結合網域知識和獲取自資料瀏覽步驟的深入資訊。如需指引，請參閱 [Team Data Science Process 中的特徵工程設計](machine-learning-data-science-create-features.md)。


## 4\.建立和訓練機器學習服務模型

資料科學家會建立分析模型，使用已清理和特性化的資料來預測規劃步驟中定義的商業需求所識別的重要變數。機器學習系統支援多個適用於各種情況的**模型化演算法**。如需指引，請參閱[如何選擇 Azure Machine Learning 的演算法](machine-learning-algorithm-choice.md)。

資料科學家必須選擇最適合其預測工作的模型，而來自多個模型的結果也經常必須結合以取得最佳結果。模型化的輸入資料通常隨機分成三個部分：

- 訓練資料集、
- 驗證資料集
- 測試資料集

模型會使用**訓練資料集**來建置。模型的最佳組合 (透過參數調整) 可藉由執行模型及評估**驗證資料集**的預測誤差來選取。最後，會使用**測試資料集**來評估選擇的模型對於未用來訓練或驗證模型之獨立資料的效能。如需程序，請參閱[如何評估 Azure 機器學習中的模型效能](machine-learning-evaluate-model-performance.md)。


## 5\.在產品中部署和使用模型

在我們擁有一組妥善執行的模型後，這些模型即可**開始運作**，供其他應用程式取用。預測可根據商業需求，以**即時**或**批次**的形式執行。模型必須透過可輕易從各種不同的應用程式 (例如線上網站、試算表、儀表板或企業營運和後端應用程式) 取用的**開放式 API 介面**加以公開，才能開始運作。請參閱[部署 Azure Machine Learning Web 服務](machine-learning-publish-a-machine-learning-web-service.md)。


## 摘要和後續步驟

[Team Data Science Process](https://azure.microsoft.com/documentation/learning-paths/data-science-process/) 會模型化為一系列反覆執行的步驟，以針對使用進階分析來建置智慧型應用程式所需的工作**提供指引**。每個步驟也會提供詳細資訊，說明如何使用各種不同的 Microsoft 技術來完成工作。

雖然 TDSP 並未指定特定類型的**文件**構件，但最好能夠記錄資料瀏覽、模型化和評估的結果，並儲存相關的程式碼，使分析可以在必要時反覆執行。這也可讓您在處理牽涉到類似資料和預測工作的其他應用程式時，能夠重複使用分析工作。

此外也會提供完整的端對端逐步解說，說明**特定案例**之程序中的所有步驟。[Team Data Science Process 逐步解說](data-science-process-walkthroughs.md)主題中會列出這些逐步解說與縮圖描述。

<!---HONumber=AcomDC_0921_2016-->