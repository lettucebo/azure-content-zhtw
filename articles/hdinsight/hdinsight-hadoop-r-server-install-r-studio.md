<properties
	pageTitle="在具有 R 伺服器的 HDInsight 叢集上安裝 RStudio (預覽) | Microsoft Azure"
	description="如何在 HDInsight (預覽) 中使用 R 伺服器安裝 RStudio。"
	services="hdinsight"
	documentationCenter=""
	authors="jeffstokes72"
	manager="jhubbard"
	editor="cgronlun"/>

<tags
   ms.service="hdinsight"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="big-data"
   ms.date="09/16/2016"
   ms.author="jeffstok"/>


# 在 HDInsight (預覽) 中使用 R 伺服器安裝 RStudio

現今有多個整合的開發環境 (IDE) 適用於 R，包括 Microsoft 最近推出的 [R Tools for Visual Studio](https://www.visualstudio.com/zh-TW/features/rtvs-vs.aspx) (RTVS)、[RStudio](https://www.rstudio.com/products/rstudio-server/) 的桌面和伺服器工具系列，或 Walware 的 Eclipse 型 [StatET](http://www.walware.de/goto/statet)。在 Linux 上最熱門的是使用 [RStudio 伺服器](https://www.rstudio.com/products/rstudio-server/)，其提供可讓遠端用戶端使用的瀏覽器型 IDE。在 HDInsight Premium 叢集的邊緣節點上安裝 RStudio 伺服器時，可提供完整的 IDE 體驗以搭配叢集上的 R 伺服器開發和執行 R 指令碼；和預設使用 R 主控台相比，可以大幅提高生產力。

在這篇文章中，您將學習如何使用自訂指令碼，在叢集的邊緣節點上安裝 RStudio Server 的社群 (免費) 版本。如果您偏好 RStudio 伺服器的商業授權 Pro 版本，您必須依照 [RStudio 伺服器](https://www.rstudio.com/products/rstudio/download-server/)的安裝指示。

> [AZURE.NOTE] 這份文件中的步驟需要 HDInsight 叢集中的 R 伺服器，且如果您使用的 HDInsight 叢集上，當初是使用[安裝 R 指令碼動作](hdinsight-hadoop-r-scripts-linux.md)來安裝 R，將無法正常運作。

## 必要條件

* 具有 R 伺服器的 Azure HDInsight 叢集已安裝。如需相關指示，請參閱[開始在 HDInsight 叢集上使用 R 伺服器](hdinsight-hadoop-r-server-get-started.md)。
* SSH 用戶端。若為 Linux 和 Unix 發佈或 Macintosh OS X，`ssh` 命令會隨作業系統提供。對於 Windows，我們建議使用 [Cygwin](http://www.redhat.com/services/custom/cygwin/) 與 [OpenSSH 選項](https://www.youtube.com/watch?v=CwYSvvGaiWU)，或 [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)。


## 使用自訂指令碼在叢集上安裝 RStudio

1. 識別叢集的邊緣節點。若為具有 R 伺服器的 HDInsight 叢集，以下是前端節點和邊緣節點的命名慣例。

	* 前端節點 - `CLUSTERNAME-ssh.azurehdinsight.net`
	* 邊緣節點 - `R-Server.CLUSTERNAME-ssh.azurehdinsight.net`

2. 使用上述的命名模式將 SSH 連接至叢集的邊緣節點。
 
	* 如果您是從 Linux 用戶端連接，請參閱[連接至以 Linux 為基礎的 HDInsight 叢集](hdinsight-hadoop-linux-use-ssh-unix.md#connect-to-a-linux-based-hdinsight-cluster)。
	* 如果您是從 Windows 用戶端連接，請參閱[使用 PuTTY 連接至以 Linux 為基礎的 HDInsight 叢集](hdinsight-hadoop-linux-use-ssh-windows.md#connect-to-a-linux-based-hdinsight-cluster)。

3. 一旦連接，就會變成叢集上的根使用者。在 SSH 工作階段中，輸入下列命令。

		sudo su -

4. 下載自訂指令碼以安裝 RStudio。使用下列命令。

		wget http://mrsactionscripts.blob.core.windows.net/rstudio-server-community-v01/InstallRStudio.sh

5. 變更自訂指令碼檔案的權限，然後執行指令碼。使用下列命令。

		chmod 755 InstallRStudio.sh
		./InstallRStudio.sh

6. 如果您在建立具有 R 伺服器的 HDInsight 叢集時使用 SSH 密碼，您可以略過此步驟，並繼續進行下一步。如果您是使用 SSH 金鑰建立叢集，您必須為您的 SSH 使用者設定密碼。連接至 Rstudio 時，您會需要此密碼。執行下列命令。當系統提示您輸入**目前的 Kerberos 密碼**時，只要按 **Enter**。請注意，您必須將 `USERNAME` 取代為您的 HDInsight 叢集的 SSH 使用者。

		passwd USERNAME
		Current Kerberos password:
		New password:
		Retype new password:
		Current Kerberos password:
		
	如果已成功設定您的密碼，您應該會看到如下的訊息。

		passwd: password updated successfully


	結束 SSH 工作階段。

7. 建立到叢集的 SSH 通道，方法是將 HDInsight 叢集上的 `localhost:8787` 對應至用戶端電腦。您必須在開啟新的瀏覽器工作階段之前，建立 SSH 通道。

	* 在使用 [Cygwin](http://www.redhat.com/services/custom/cygwin/) 的 Linux 用戶端或 Windows 用戶端上，開啟終端機工作階段並且使用下列命令。

			ssh -L localhost:8787:localhost:8787 USERNAME@R-Server.CLUSTERNAME-ssh.azurehdinsight.net
			
		以您的 HDInsight 叢集的 SSH 使用者取代 **USERNAME**，並以您的 HDInsight 叢集的名稱取代 **CLUSTERNAME**。您也可以新增 `-i id_rsa_key` 以使用 SSH 金鑰而非密碼。

	* 如果使用 Windows 用戶端與 PuTTY，則

		1.  開啟 PuTTY，並輸入連線資訊。如果您不熟悉 PuTTY，請參閱[從 Windows 在 HDInsight 上搭配使用 SSH 與以 Linux 為基礎的 Hadoop](hdinsight-hadoop-linux-use-ssh-windows.md)，以取得如何搭配 HDInsight 使用 PuTTY 的資訊。
		2.  在對話方塊左側的 [**類別**] 區段中，依序展開 [**連接**] 和 [**SSH**]，最後選取 [**通道**]。
		3.  在 [**控制 SSH 連接埠轉送的選項**] 表單中提供下列資訊：

			* **來源連接埠** - 您想要轉送之用戶端上的連接埠。例如，**8787**。
			* **目的地** - 必須對應至本機用戶端電腦的目的地。例如，**localhost:8787**。

			![建立 SSH 通道](./media/hdinsight-hadoop-r-server-install-r-studio/createsshtunnel.png "建立 SSH 通道")

		4. 按一下 [**新增**] 以新增設定，然後按一下 [**開啟**] 開啟 SSH 連線。
		5. 出現提示時，登入伺服器。這會建立 SSH 工作階段，並啟用通道。

8. 開啟網頁瀏覽器，並且根據您針對通道輸入的連接埠輸入下列 URL。

		http://localhost:8787/ 

9. 系統將會提示您輸入 SSH 使用者名稱和密碼以連接至叢集。如果您在建立叢集時使用 SSH 金鑰，您必須輸入在上述的步驟 5 中所建立的密碼。

	![連接至 R Studio](./media/hdinsight-hadoop-r-server-install-r-studio/connecttostudio.png "建立 SSH 通道")

10. 若要測試 RStudio 安裝是否成功，您可以執行測試指令碼，該指令碼會在叢集上執行以 R 為基礎的 MapReduce 和 Spark 作業。返回 [SSH] 主控台，並且輸入下列命令來下載測試指令碼，在 RStudio 中執行。

	* 如果您建立具有 R 的 Hadoop 叢集，請使用此命令。
		
			wget http://mrsactionscripts.blob.core.windows.net/rstudio-server-community-v01/testhdi.r

	* 如果您建立具有 R 的 Spark 叢集，請使用此命令。

			wget http://mrsactionscripts.blob.core.windows.net/rstudio-server-community-v01/testhdi_spark.r

11. 在 RStudio 中，您會看到您所下載的測試指令碼。按兩下以開啟該檔案，選取檔案的內容，然後按一下 [執行]。您應該可以在 [主控台] 窗格中看到輸出。
 
	![測試安裝](./media/hdinsight-hadoop-r-server-install-r-studio/test-r-script.png "測試安裝")

另一個做法是輸入 `source(testhdi.r)` 或 `source(testhdi_spark.r)` 執行指令碼。

## 另請參閱

- [適用於 HDInsight 叢集上的 R 伺服器的計算內容選項](hdinsight-hadoop-r-server-compute-contexts.md)

- [適用於 HDInsight Premium 上的 R 伺服器的 Azure 儲存體選項](hdinsight-hadoop-r-server-storage.md)


 

<!---HONumber=AcomDC_0921_2016-->