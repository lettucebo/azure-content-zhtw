<properties
	pageTitle="使用 Hadoop 沙箱來了解 Hadoop | Microsoft Azure"
	description="若要開始了解 Hadoop 生態系統，您可以在 Azure 虛擬機器上從 Hortonworks 設定 Hadoop 沙箱。"
	keywords="hadoop 模擬器, hadoop 沙箱"
	editor="cgronlun"
	manager="jhubbard"
	services="hdinsight"
	authors="nitinme"
	documentationCenter=""
	tags="azure-portal"/>

<tags
	ms.service="hdinsight"
	ms.workload="big-data"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/24/2016"
	ms.author="nitinme"/>

# 透過虛擬機器的 Hadoop 沙箱開始使用 Hadoop 生態系統

了解如何在虛擬機器上從 Hortonworks 安裝 Hadoop 沙箱，以了解 Hadoop 生態系統。沙箱提供本機開發環境，讓您了解 Hadoop、Hadoop 分散式檔案系統 (HDFS)，以及作業提交。

## 必要條件

* [Oracle VirtualBox](https://www.virtualbox.org/)

熟悉 Hadoop 之後，您就可以開始在 Azure 中使用 Hadoop 建立 HDInsight 叢集。有關如何開始使用的詳細資訊，請參閱[開始在 HDInsight 中使用 Hadoop](hdinsight-hadoop-linux-tutorial-get-started.md)。

## 下載並安裝虛擬機器

1. 從 [http://hortonworks.com/downloads/#sandbox](http://hortonworks.com/downloads/#sandbox) 在 Hortonworks 沙箱選取 HDP 2.4 的__為 VirtualBox 下載__項目。系統會提示您向 Hortonworks 註冊，然後才能開始下載。

    ![用於下載 VirtualBox 之 Hortonworks 沙箱的連結影像](./media/hdinsight-hadoop-emulator-get-started/download-sandbox.png)

2. 從相同的網頁，在 Hortonworks 沙箱選取 HDP 2.4 的 __VirtualBox 安裝手冊__。這會下載內含虛擬機器安裝指示的 PDF。

    ![檢視安裝指南](./media/hdinsight-hadoop-emulator-get-started/view-install-guide.png)

## 啟動虛擬機器

1. 啟動 VirtualBox、選取 [Hortonworks 沙箱]、選取 [啟動]，然後選取 [正常啟動]。

    ![正常啟動](./media/hdinsight-hadoop-emulator-get-started/normal-start.png)

2. 虛擬機器完成開機程序後會顯示登入指示。開啟網頁瀏覽器，並瀏覽至顯示的 URL (通常是 http://127.0.0.1:8888)。

## 設定密碼

1. 從 [Hortonworks 沙箱] 頁面的 [入門] 步驟選取 [檢視進階選項]。在此頁面上使用該資訊，以透過 SSH 登入沙箱。使用提供的名稱和密碼。

    > [AZURE.NOTE] 如果您未安裝 SSH 用戶端，您可以使用虛擬機器在 \_\_http://localhost:4200/__所提供的網頁型 SSH。

    第一次使用 SSH 連線時，系統會提示您變更根帳戶的密碼。請輸入新密碼，以供日後使用 SSH 登入時使用。

2. 登入之後，輸入下列命令：

        ambari-admin-password-reset
    
    出現提示時，提供 Ambari 系統管理員帳戶的密碼。當您存取 Ambari Web UI 時將會用到。

## 使用 Hive 命令

1. 在連往沙箱的 SSH 連線中，使用下列命令啟動 Hive 殼層：

        hive

2. 殼層啟動之後，使用下列命令檢視沙箱所提供的資料表︰

        show tables;

3. 使用下列命令從 `sample_07` 資料表擷取 10 個資料列︰

        select * from sample_07 limit 10;

## 後續步驟

* [了解如何搭配使用 Visual Studio 與 Hortonworks 沙箱](hdinsight-hadoop-emulator-visual-studio.md)
* [了解 Hortonworks 沙箱的訣竅](http://hortonworks.com/hadoop-tutorial/learning-the-ropes-of-the-hortonworks-sandbox/)
* [Hadoop 教學課程 - 開始使用 HDP](http://hortonworks.com/hadoop-tutorial/hello-world-an-introduction-to-hadoop-hcatalog-hive-and-pig/)

<!---HONumber=AcomDC_0921_2016-->