<properties
   pageTitle="適用於 HDInsight (預覽) 中 R 伺服器的計算內容選項 | Microsoft Azure"
   description="了解 HDInsight (預覽) 中 R 伺服器之使用者可用的不同計算內容選項"
   services="HDInsight"
   documentationCenter=""
   authors="jeffstokes72"
   manager="jhubbard"
   editor="cgronlun"
/>

<tags
   ms.service="HDInsight"
   ms.devlang="R"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="data-services"
   ms.date="07/21/2016"
   ms.author="jeffstok"
/>

# 適用於 HDInsight (預覽) 中 R 伺服器的計算內容選項

Azure HDInsight (預覽) 上的 Microsoft R 伺服器可提供最新的 R 型分析功能。它會在您的 [Azure Blob](../storage/storage-introduction.md "Azure Blob 儲存體") 儲存體帳戶或本機 Linux 檔案系統中，使用儲存在容器的 HDFS 中的資料。R 伺服器是根據開放原始碼 R 所建置，因此您建置的 R 型應用程式可以利用 8000 多個開放原始碼 R 封裝的任何一個。它們也可以利用 [ScaleR](http://www.revolutionanalytics.com/revolution-r-enterprise-scaler "Revolution Analytics ScaleR") (R 伺服器隨附的 Microsoft 巨量資料分析封裝) 中的常式。

Premium 叢集的邊緣節點提供便利的地方，以便連接到叢集並執行 R 指令碼。有了邊緣節點之後，即可選擇跨邊緣節點伺服器的核心，執行 ScaleR 的平行分散式函數。也可以選擇透過使用 ScaleR 的 Hadoop Map Reduce 或 Spark 計算內容，跨叢集的節點執行這些函數。

## 邊緣節點的計算內容

一般而言，在邊緣節點上 R 伺服器中執行的 R 指令碼會在該節點上的 R 解譯器內執行。但呼叫 ScaleR 函式的步驟則屬例外。ScaleR 呼叫會在計算環境中執行，該環境是由您設定 ScaleR 計算內容的方式所決定。當您從邊緣節點執行您的 R 指令碼時，計算內容的可能值為本機循序 (‘local’)、本機平行 (‘localpar’)、Map Reduce 和 Spark，如下所示：

‘local’ 和 ‘localpar’ 選項的差別只在於執行 rxExec 呼叫的方式。它們都會在所有可用的核心之間，以平行方式執行其他的 rx 函式呼叫，除非已指定，否則皆使用 ScaleR numCoresToUse 選項，例如 rxOptions(numCoresToUse=6)。以下摘要說明不同的計算內容選項。

| 計算內容 | 設定方式 | 執行內容 |
|------------------|---------------------------------|---------------------------------------------------------------------------------------|
| 本機循序 | rxSetComputeContext(‘local’) | 跨邊緣節點伺服器的核心平行執行，只有 rxExec 為循序執行。 |
| 本機平行 | rxSetComputeContext(‘localpar’) | 跨邊緣節點伺服器的核心平行執行 |
| Spark | RxSpark() | 跨 HDI 叢集的節點透過 Spark 平行處理分散式執行 |
| Map Reduce | RxHadoopMR() | 跨 HDI 叢集的節點透過 Map Reduce 平行處理分散式執行 |


假設您想要針對效能目的進行平行處理執行，則有三個選項。您會選擇哪個選項，取決於您的分析工作本質以及資料的大小和位置。

## 決定計算內容的指導方針

目前沒有任何公式可告訴您該使用哪個計算內容。不過，有一些指導原則可協助您做出正確的選擇，或至少幫助您縮小選擇範圍，然後再執行基準測試。這些指導原則包括︰

1.	本機 Linux 檔案系統比 HDFS 還快。
2.	如果資料位於本機，且是 XDF 格式，則重複分析會比較快。
3.	最好是從文字資料來源串流小量的資料；如果資料量較大，請將其轉換為 XDF 格式再進行分析。
4.	對於非常大量的資料，將資料複製或串流至邊緣節點以進行分析的額外負荷會變得難以管理。
5.	Spark 在 Hadoop 中的分析速度較 Map Reduce 快。

對於這些原則，用來選取計算內容的一般規則是︰

### 本機

- 如果要分析的資料量很小，而且不需要重複分析，請直接串流至分析常式並且使用 'local' 或 'localpar'。
- 如果要分析的資料量很小或是中等大小，並且需要重複分析，請將它複製到本機檔案系統、匯入至 XDF，然後透過 'local' 或 'localpar' 進行分析。

### Hadoop Spark

- 如果要分析的資料量很大，請將它匯入至 HDFS 中的 XDF (除非儲存上有問題)，然後透過 'Spark' 進行分析。

### Hadoop Map Reduce

- 請只在您使用 Spark 計算內容時發生無法克服的問題時才使用，因為它的速度通常會比較慢。

## rxSetComputeContext 的內嵌說明

如需 ScaleR 計算內容的詳細資訊和範例，請參閱 rxSetComputeContext 方法上 R 中的內嵌說明，例如︰

    > ?rxSetComputeContext

您也可以參閱《ScaleR 分散式計算指南》，可以從 [R 伺服器 MSDN](https://msdn.microsoft.com/library/mt674634.aspx "MSDN 上的 R 伺服器") 文件庫取得。


## 後續步驟

在本文中，您已學會如何建立包含 R 伺服器的新 HDInsight 叢集。您也學會了從 SSH 工作階段使用 R 主控台的基本概念。現在您可以閱讀下列文章，探索在 HDInsight 上使用 R 伺服器的其他方法︰

- [適用於 Hadoop 的 R 伺服器概觀](hdinsight-hadoop-r-server-overview.md)
- [開始使用適用於 Hadoop 的 R 伺服器](hdinsight-hadoop-r-server-get-started.md)
- [將 RStudio 伺服器加入 HDInsight Premium 中](hdinsight-hadoop-r-server-install-r-studio.md)
- [適用於 HDInsight Premium R 伺服器的 Azure 儲存體選項](hdinsight-hadoop-r-server-storage.md)

<!---HONumber=AcomDC_0921_2016-->