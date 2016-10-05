<properties
	pageTitle="可以搭配 HDInsight 叢集使用的不同元件有哪些？ | Microsoft Azure"
	description="HDInsight 支援多個可部署的 Hadoop 叢集元件和版本。請參閱支援的 Hadoop 和 HortonWorks Data Platform (HDP) 配送版本。"
	services="hdinsight"
	editor="cgronlun"
	manager="jhubbard"
	authors="mumian"
	tags="azure-portal"
	documentationCenter=""/>

<tags
	ms.service="hdinsight"
	ms.workload="big-data"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/21/2016"
	ms.author="jgao"/>


# 可以搭配 HDInsight 使用的不同 Hadoop 元件有哪些？

了解 HDInsight 提供的不同服務層級，以及 HDInsight 隨附的不同 hadoop 元件版本。

## HDInsight Standard 和 HDInsight Premium

Azure HDInsight 提供兩種類型的巨量資料雲端提供項目：**Standard** 和 **Premium**。區段下的資料表列出**只**有在「進階」部分可用的功能。資料表中未明確標註的功能都是「標準」的部分。

>[AZURE.NOTE] HDInsight Premium 提供項目目前為預覽版，僅適用於 Linux 叢集。

| HDInsight Premium 功能 | 說明 |
|--------------|---------------|
| Microsoft R 伺服器 (預覽) | Microsoft R 伺服器是針對可調整 R 最廣泛部署的企業級分析平台。R 語言支援各種巨量資料統計資料、預測模型及機器學習功能。做為 HDInsight Premium 的一部分，您現在可以建立具有 R 伺服器的 HDInsight 叢集，以供大量資料集和模型使用。這項新功能提供資料科學家和統計學家熟悉的 R 介面，並且可以隨選調整 HDInsight，沒有叢集設定和維護的額外負荷。<br> <br>如需詳細資訊，請參閱[在 HDInsight 上開始使用 R 伺服器](hdinsight-hadoop-r-server-get-started.md)。

### 進階的支援叢集類型

下表列出 HDInsight 叢集類型和進階支援矩陣。

| 叢集類型 | 標準 | 高級 |
|--------------|---------------|--------------|
| Hadoop | 是 | 是 |
| Spark | 是 | 是 |
| HBase | 是 | 否 |
| Storm | 是 | 否 |

因為 HDInsight Premium 中包含更多的叢集類型，所以此資料表將會更新。

### 價格和 SLA

如需 HDInsight Premium 的價格和 SLA 的詳細資訊，請參閱 [HDInsight 價格](https://azure.microsoft.com/pricing/details/hdinsight/)。

## 可以搭配不同 HDInsight 版本使用的 Hadoop 元件

Azure HDInsight 支援多個可隨時部署的 Hadoop 叢集版本。每一個版本選擇都會建立特定版本的 Hortonworks Data Platform (HDP) 散發，以及該散發內包含的一組元件。下表列舉了與 HDInsight 叢集版本相關聯的元件版本。請注意，Azure HDInsight 目前所使用的預設叢集版本為 3.4 版，採用 HDP 2.4 (截至 09/14/2016 為止)。

> [AZURE.NOTE] 服務的預設版本可能會變更，恕不另行通知。如果您有版本相依性，我們建議您在使用 .NET SDK/Azure PowerShell 和 Azure CLI 建立叢集時指定版本。

元件|HDInsight 3.4 版 (預設) | HDInsight 3.3 版 | HDInsight 3.2 版 |HDInsight 3.1 版 |HDInsight 3.0 版|
---|---|---|---|---|---
Hortonworks Data Platform|2\.4|2\.3|2\.2|2\.1.7|2\.0|
Apache Hadoop & YARN|2\.7.1|2\.7.1|2\.6.0|2\.4.0|2\.2.0|
Apache Tez|0\.7.0|0\.7.0 | 0\.5.2|0\.4.0||
Apache Pig|0\.15.0|0\.15.0|0\.14.0|0\.12.1|0\.12.0|
Apache Hive & HCatalog|1\.2.1|1\.2.1|0\.14.0|0\.13.1|0\.12.0|
Apache HBase (英文) |1\.1.2|1\.1.1|0\.98.4|0\.98.0||
Apache Sqoop|1\.4.6|1\.4.6|1\.4.5|1\.4.4|1\.4.4|1\.4.3
Apache Oozie|4\.2.0|4\.2.0|4\.1.0|4\.0.0|4\.0.0|
Apache Zookeeper|3\.4.6|3\.4.6|3\.4.6|3\.4.5|3\.4.5|
Apache Storm|0\.10.0|0\.10.0|0\.9.3|0\.9.1||
Apache Mahout|0\.9.0+|0\.9.0+|0\.9.0|0\.9.0||
Apache Phoenix|4\.4.0|4\.4.0|4\.2.0|4\.0.0.2.1.7.0-2162||
Apache Spark|1\.6.0 (僅限 Linux)|1\.5.2 (僅限 Linux/實驗性組建)|1\.3.1 (僅限 Windows)|||


**取得目前的元件版本資訊**

在 HDInsight 的未來更新中，可能會變更 HDInsight 叢集版本相關的元件版本。若要判斷可用的元件和驗證叢集所使用的版本，有一個辦法就是使用 Ambari REST API。**GetComponentInformation** 命令可用來擷取服務元件的相關資訊。如需詳細資訊，請參閱＜[Ambari 文件][ambari-docs]＞(英文)。另一種取得此資訊的作法就是使用遠端桌面登入叢集，然後直接檢查 "C:\\apps\\dist" 目錄的內容。


**版本資訊**

請參閱 [HDInsight 版本資訊](hdinsight-release-notes.md)，以取得 HDInsight 最新版本的其他版本資訊。


## 支援的 HDInsight 辦本
下表列出目前可用的 HDInsight 版本、它們使用的相對應 Hortonworks Data Platform 版本及其發行日期。另外也會提供其支援到期日和淘汰日期 (已知道的話)。請注意：

* 依預設，系統會為 HDInsight 2.1 和更新版本部署搭配兩個前端節點的高可用性叢集。HDInsight 1.6 叢集並不適用。
* 在特定版本的支援到期後，您可能無法透過 Azure 入口網站取得。下表指出可在 Azure 傳統入口網站上取得的版本。您可透過 Windows PowerShell [New-AzureRmHDInsightCluster](https://msdn.microsoft.com/library/mt619331.aspx) 命令中的 `Version` 參數和 .NET SDK 持續取得叢集版本，直到其淘汰日期為止。

HDInsight 版本|HDP 版本|VM OS|高可用性|發行日期|可在 Azure 入口網站上取得|支援到期日|淘汰日期
---|---|---|---|---|---|---|---
HDI 3.4|HDP 2.4|Ubuntu 14.0.4 LTS|是|03/29/2016|是||
HDI 3.3|HDP 2.3|Ubuntu 14.0.4 LTS 或 Windows Server 2012R2|是|12/02/2015|是|06/27/2016|07/31/2017
HDI 3.2|HDP 2.2|Ubuntu 12.04 LTS 或 Windows Server 2012R2|是|2015/2/18|是|3/1/2016|04/01/2017
HDI 3.1|HDP 2.1|Windows Server 2012R2|是|2014/6/24|否|05/18/2015|06/30/2016
HDI 3.0|HDP 2.0|Windows Server 2012R2|是|02/11/2014|否|09/17/2014|06/30/2015
HDI 2.1|HDP 1.3|Windows Server 2012R2|是|10/28/2013|否|05/12/2014|05/31/2015
HDI 1.6|HDP 1.1||否|10/28/2013|否|04/26/2014|05/31/2015

**非預設叢集的部署**

### HDInsight 叢集版本的服務等級協定

SLA 是根據「支援期間」來定義。「支援期間」是指 Microsoft 客戶服務與支援中心支援 HDInsight 叢集版本的一段時間。如果 HDInsight 叢集版本的 [支援到期日] 超過目前日期，則表示該叢集不在「支援期間」。上表中可找到支援的 HDInsight 叢集版本清單。特定 HDInsight X (在較新的 X+1 版本推出後) 版本的支援到期日計算方式會以下列較晚的時間為準：

- 公式 1：將 HDInsight 叢集 X 版發行日期加上 180 天。
- 公式 2：將 HDInsight 叢集 X+1 版日期加上 90 天 (X 之後的後續版本) 可用於入口網站。

[**淘汰日期**] 是指在此日期之後便無法在 HDInsight 上建立叢集版本。

> [AZURE.NOTE] Windows 型 HDInsight 叢集 (包括 2.1、3.0、3.1、3.2 和 3.3 版) 可在 Azure 客體 OS 系列 4 上執行，此系列使用 64 位元版的 Windows Server 2012 R2 並支援 .NET Framework 4.0、4.5、4.5.1 和 4.5.2。

## 與 HDInsight 版本相關聯的 Hortonworks 版本資訊##

* HDInsight 叢集 3.4 版採用以 [Hortonworks Data Platform 2.4](http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.4.0/bk_HDP_RelNotes/content/ch_relnotes_v240.html) 為基礎的 Hadoop 散發套件。這是使用入口網站時所建立的**預設** Hadoop 叢集。



* HDInsight 叢集 3.3 版採用以 [Hortonworks Data Platform 2.3](http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.3.0/bk_HDP_RelNotes/content/ch_relnotes_v230.html) 為基礎的 Hadoop 散發。
	* Apache Storm 版本資訊可從[這裡](https://storm.apache.org/2015/11/05/storm0100-released.html)取得。
	* Apache Hive 版本資訊可從[這裡](https://issues.apache.org/jira/secure/ReleaseNote.jspa?version=12332384&styleName=Text&projectId=12310843)取得。

* HDInsight 叢集 3.2 版採用以 [Hortonworks Data Platform 2.2][hdp-2-2] 為基礎的 Hadoop 散發。

	* 特定 Apache 元件的版本資訊 - [Hive 0.14](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12310843&version=12326450)、[Pig 0.14](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12310730&version=12326954)、[HBase 0.98.4](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12310753&version=12326810)、[Phoenix 4.2.0](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12315120&version=12327581)、[M/R 2.6](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12310941&version=12327180)、[HDFS 2.6](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12310942&version=12327181)、[YARN 2.6](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12313722&version=12327197)、[Common](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12310240&version=12327179)、[Tez 0.5.2](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12314426&version=12328742)、[Ambari 2.0](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12312020&version=12327486)、[Storm 0.9.3](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12314820&version=12327112)、[Oozie 4.1.0](https://issues.apache.org/jira/secure/ReleaseNote.jspa?version=12324960&projectId=12311620)。


* HDInsight 叢集 3.1 版採用以 [Hortonworks Data Platform 2.1.7][hdp-2-1-7] 為基礎的 Hadoop 散發。在 2014/11/7 之前建立的 HDInsight 3.1 叢集是以 [Hortonworks Data Platform 2.1.1][hdp-2-1-1] 為基礎。

* HDInsight 叢集 3.0 版採用以 [Hortonworks Data Platform 2.0][hdp-2-0-8] 為基礎的 Hadoop 散發。

* HDInsight 叢集 2.1 版採用以 [Hortonworks Data Platform 1.3][hdp-1-3-0] 為基礎的 Hadoop 散發。

* HDInsight 叢集 1.6 版採用以 [Hortonworks Data Platform 1.1][hdp-1-1-0] 為基礎的 Hadoop 散發。


[image-hdi-versioning-versionscreen]: ./media/hdinsight-component-versioning/hdi-versioning-version-screen.png

[wa-forums]: http://azure.microsoft.com/support/forums/

[connect-excel-with-hive-ODBC]: hdinsight-connect-excel-hive-ODBC-driver.md

[hdp-2-2]: http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.2.0/HDP_2.2.0_Release_Notes_20141202_version/index.html

[hdp-2-1-7]: http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.1.7-Win/bk_releasenotes_HDP-Win/content/ch_relnotes-HDP-2.1.7.html

[hdp-2-1-1]: http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.1.1/bk_releasenotes_hdp_2.1/content/ch_relnotes-hdp-2.1.1.html

[hdp-2-0-8]: http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.0.8.0/bk_releasenotes_hdp_2.0/content/ch_relnotes-hdp2.0.8.0.html

[hdp-1-3-0]: http://docs.hortonworks.com/HDPDocuments/HDP1/HDP-1.3.0/bk_releasenotes_hdp_1.x/content/ch_relnotes-hdp1.3.0_1.html

[hdp-1-1-0]: http://docs.hortonworks.com/HDPDocuments/HDP1/HDP-Win-1.1/bk_releasenotes_HDP-Win/content/ch_relnotes-hdp-win-1.1.0_1.html

[ambari-docs]: https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md

[zookeeper]: http://zookeeper.apache.org/

<!---HONumber=AcomDC_0921_2016-->