<properties
pageTitle="在 HDInsight 叢集建立期間新增 Hive 程式庫 | Azure"
description="了解如何在叢集建立期間將 Hive 程式庫 (jar 檔案) 新增至 HDInsight 叢集。"
services="hdinsight"
documentationCenter=""
authors="Blackmist"
manager="jhubbard"
editor="cgronlun"/>

<tags
ms.service="hdinsight"
ms.devlang="na"
ms.topic="article"
ms.tgt_pltfrm="na"
ms.workload="big-data"
ms.date="09/20/2016"
ms.author="larryfr"/>

#在 HDInsight 叢集建立期間新增 Hive 程式庫

如果您有與 HDInsight 上的 Hive 經常一起使用的程式庫，本文件包含在叢集建立期間使用指令碼動作來預先載入程式庫的詳細資訊。這樣可讓程式庫供整個 Hive 使用 (不需要使用 [ADD JAR](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Cli) 來載入它們。)

##運作方式

建立叢集時，您可以選擇性地指定指令碼動作，以便在建立叢集節點時在節點上執行指令碼。這份文件中的指令碼接受單一參數，也就是將預先載入的程式庫 (儲存為 jar 檔案) 所在的 WASB 位置。

叢集建立期間，指令碼會列舉檔案、將它們複製到前端和背景工作節點上的 `/usr/lib/customhivelibs/` 目錄，然後將它們加入至 `core-site.xml` 檔案中的 `hive.aux.jars.path` 屬性。在以 Linux 為基礎的叢集上，它也會以檔案的位置來更新 `hive-env.sh` 檔案。

> [AZURE.NOTE] 使用本文中的指令碼動作可讓程式庫在下列案例中可供使用：
>
> * __以 Linux 為基礎的 HDInsight__ - 使用 __Hive 命令列__、__WebHCat__ (Templeton) 和 __HiveServer2__ 時。
> * __以 Windows 為基礎的 HDInsight__ - 使用 __Hive 命令列__和 __WebHCat__ (Templeton) 時。

##指令碼

__指令碼位置__

__以 Linux 為基礎的叢集__：[https://hdiconfigactions.blob.core.windows.net/linuxsetupcustomhivelibsv01/setup-customhivelibs-v01.sh](https://hdiconfigactions.blob.core.windows.net/linuxsetupcustomhivelibsv01/setup-customhivelibs-v01.sh)

__以 Windows 為基礎的叢集__：[https://hdiconfigactions.blob.core.windows.net/setupcustomhivelibsv01/setup-customhivelibs-v01.ps1](https://hdiconfigactions.blob.core.windows.net/setupcustomhivelibsv01/setup-customhivelibs-v01.ps1)

__需求__

* 指令碼必須同時套用至__前端節點__和__背景工作節點__。

* 您想要安裝的 jar 必須儲存在__單一容器__中的 Azure Blob 儲存體。

* 包含 jar 檔案程式庫的儲存體帳戶__必須__在建立期間連結至 HDInsight 叢集。有兩種方式可以完成這項作業：

    * 放在叢集的預設儲存體帳戶上的容器中。
    
    * 放在連結儲存體容器上的容器中。例如，在入口網站中，您可以使用 [選擇性組態]、[連結的儲存體帳戶] 新增額外的儲存體。

* 必須指定容器的 WASB 路徑做為指令碼動作的參數。例如，如果 jar 儲存在名為 __mystorage__ 的儲存體帳戶上稱為 __libs__ 的容器中，則這個參數會是 \_\_wasbs://libs@mystorage.blob.core.windows.net/__。

    > [AZURE.NOTE] 本文件假設您已建立儲存體帳戶、blob 容器，也已將檔案上傳給它。
    >
    > 如果您尚未建立儲存體帳戶，您可以透過 [Azure 入口網站](https://portal.azure.com)這樣做。接著，您可以使用 [Azure 儲存體總管](http://storageexplorer.com/)之類的公用程式，在帳戶中建立新的容器，並將檔案上傳給它。

##使用指令碼建立叢集

> [AZURE.NOTE] 下列步驟會建立以 Linux 為基礎的 HDInsight 新叢集。若要建立以 Windows 為基礎的新叢集，請在建立叢集時選取 __Windows__ 作為叢集作業系統，並使用 Windows (PowerShell) 指令碼，而不是 bash 指令碼。
> 
> 您也可以使用 Azure PowerShell 或 HDInsight .NET SDK，以使用此指令碼建立叢集。如需使用這些方法的詳細資訊，請參閱[使用指令碼動作自訂 HDInsight 叢集](hdinsight-hadoop-customize-cluster-linux.md)。

1. 使用[在 Linux 上佈建 HDInsight 叢集](hdinsight-hadoop-provision-linux-clusters.md#portal)中的步驟開始佈建叢集，但是不完成佈建。

2. 在 [選用組態] 刀鋒視窗中，選取 [指令碼動作]，並提供如下所示的資訊：

    * __名稱__：輸入指令碼動作的易記名稱。
    * __指令碼 URI__：https://hdiconfigactions.blob.core.windows.net/linuxsetupcustomhivelibsv01/setup-customhivelibs-v01.sh
    * __HEAD__：勾選此選項
    * __背景工作角色__：勾選此選項。
    * __ZOOKEEPER__：將此選項保留空白。
    * __參數__：輸入包含 jar 的容器和儲存體帳戶的 WASB 位址。例如，\_\_wasbs://libs@mystorage.blob.core.windows.net/__。

3. 在 [指令碼動作] 底部，使用 [選取] 按鈕以儲存組態。

4. 在 [選擇性組態] 刀鋒視窗中，選取 [連結的儲存體帳戶]，然後選取 [新增儲存體金鑰] 連結。選取包含 jar 的儲存體帳戶，然後使用 [選取] 按鈕來儲存設定，並回到 [選擇性組態] 刀鋒視窗。

5. 使用 [選擇性組態] 刀鋒視窗底部的 [選取] 按鈕，儲存選擇性組態資訊。

6. 繼續如[在 Linux 上佈建 HDInsight 叢集](hdinsight-hadoop-provision-linux-clusters.md#portal)中所述佈建叢集。

完成建立叢集時，您將能夠從 Hive 使用透過此指令碼加入的 jar，而不需要使用 `ADD JAR` 陳述式。

##後續步驟

在這份文件中，您已經學會如何在叢集建立期間將 jar 檔案包含的 Hive 程式庫新增至 HDInsight 叢集。如需有關使用 Hive 的詳細資訊，請參閱[搭配使用 Hive 與 HDInsight](hdinsight-use-hive.md)

<!---HONumber=AcomDC_0921_2016-->