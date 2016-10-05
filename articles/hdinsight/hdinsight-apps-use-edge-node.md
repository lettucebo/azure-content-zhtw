<properties
	pageTitle="在 HDInsight 中使用空白邊緣節點 | Microsoft Azure"
	description="如何將空白邊緣節點新增至可做為用戶端的 HDInsight 叢集，以及如何測試/主控 HDInsight 應用程式。"
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
	ms.date="09/14/2016"
	ms.author="jgao"/>

# 在 HDInsight 中使用空白邊緣節點

了解如何將空白邊緣節點新增至以 Linux 為基礎的 HDInsight 叢集。空白邊緣節點是一部 Linux 虛擬機器，其中已安裝及設定和前端節點相同的用戶端工具，但未執行任何 Hadoop 服務。您可以使用邊緣節點來存取叢集、測試用戶端應用程式，以及裝載用戶端應用程式。

您可以將空白邊緣節點新增至現有 HDInsight 叢集，以及在建立叢集時新增到新的叢集。空白邊緣節點是使用 Azure Resource Manager 範本來新增。下列範例示範如何使用範本來新增︰

    "resources": [
		{
			"name": "[concat(parameters('clusterName'),'/', variables('applicationName'))]",
			"type": "Microsoft.HDInsight/clusters/applications",
			"apiVersion": "[variables('clusterApiVersion')]",
			"properties": {
				"marketPlaceIdentifier": "EmptyNode",
				"computeProfile": {
					"roles": [{
						"name": "edgenode",
						"targetInstanceCount": 1,
						"hardwareProfile": {
							"vmSize": "[parameters('edgeNodeSize')]"
						}
					}]
				},
				"installScriptActions": [{
					"name": "[concat('emptynode','-' ,uniquestring(variables('applicationName')))]",
					"uri": "https://raw.githubusercontent.com/hdinsight/Iaas-Applications/master/EmptyNode/scripts/EmptyNodeSetup.sh",
					"roles": ["edgenode"]
				}],
				"uninstallScriptActions": [],
				"httpsEndpoints": [],
				"applicationType": "CustomApplication"
			}
		}
	],

如範例所示，您可以選擇性地呼叫[指令碼動作](hdinsight-hadoop-customize-cluster-linux.md)來執行其他設定，例如在邊緣節點中安裝 [Apache Hue](hdinsight-hadoop-hue-linux.md)。

建立邊緣節點之後，您可以使用 SSH 連線到邊緣節點，並執行用戶端工具來存取 HDInsight 中的 Hadoop 叢集。

## 將邊緣節點新增至現有叢集

在本節中，您會使用 Resource Manager 範本將邊緣節點新增至現有 HDInsight 叢集。Resource Manager 範本可在 [GitHub](https://github.com/hdinsight/Iaas-Applications/tree/master/EmptyNode) 中找到。Resource Manager 範本會呼叫位於 https://raw.githubusercontent.com/hdinsight/Iaas-Applications/master/EmptyNode/scripts/EmptyNodeSetup.sh 的指令碼動作指令碼。此指令碼不會執行任何動作。其目的只是為了示範從 Resource Manager 範本呼叫指令碼動作。

**將空白邊緣節點新增至現有叢集**

1. 如果您還沒有 HDInsight 叢集，請予以建立。請參閱 [Hadoop 教學課程：開始在 HDInsight 中使用以 Linux 為基礎的 Hadoop](hdinsight-hadoop-linux-tutorial-get-started.md)。
2. 按一下以下影像，在 Azure 入口網站中登入 Azure 並開啟 Azure Resource Manager 範本。

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fhdinsight%2FIaas-Applications%2Fmaster%2FEmptyNode%2Fazuredeploy.json" target="_blank"><img src="https://acom.azurecomcdn.net/80C57D/cdn/mediahandler/docarticles/dpsmedia-prod/azure.microsoft.com/en-us/documentation/articles/hdinsight-hbase-tutorial-get-started-linux/20160201111850/deploy-to-azure.png" alt="Deploy to Azure"></a>

3. 設定下列屬性：

	- CLUSTERNAME︰輸入現有 HDInsight 叢集的名稱。
	- EDGENODESIZE︰選取其中一個 VM 大小。
	- EDGENODEPREFIX︰預設值是 **new**。若使用預設值，邊緣節點名稱是 **new-edgenode**。您可以從入口網站自訂前置詞。您也可以從範本自訂完整名稱。


4. 按一下 [確定] 儲存變更。
5. 在 [資源群組] 中，選取資源群組。
6. 按一下 [檢閱法律條款]，然後按一下 [購買]。
7. 選取 [釘選到儀表板]，然後按一下 [建立]。

## 在建立叢集時新增邊緣節點

在本節中，您會使用 Resource Manager 範本對邊緣節點建立 HDInsight 叢集。Resource Manager 範本可在公用 [Azure Blob 儲存體容器](http://hditutorialdata.blob.core.windows.net/armtemplates/create-linux-based-hadoop-cluster-in-hdinsight-with-edge-node.json)中找到。Resource Manager 範本會呼叫位於 https://raw.githubusercontent.com/hdinsight/Iaas-Applications/master/EmptyNode/scripts/EmptyNodeSetup.sh 的指令碼動作指令碼。此指令碼不會執行任何動作。其目的只是為了示範從 Resource Manager 範本呼叫指令碼動作。

**將空白邊緣節點新增至現有叢集**

1. 如果您還沒有 HDInsight 叢集，請予以建立。請參閱 [Hadoop 教學課程：開始在 HDInsight 中使用以 Linux 為基礎的 Hadoop](hdinsight-hadoop-linux-tutorial-get-started.md)。
2. 按一下以下影像，在 Azure 入口網站中登入 Azure 並開啟 Azure Resource Manager 範本。

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fhditutorialdata.blob.core.windows.net%2Farmtemplates%2Fcreate-linux-based-hadoop-cluster-in-hdinsight-with-edge-node.json" target="_blank"><img src="https://acom.azurecomcdn.net/80C57D/cdn/mediahandler/docarticles/dpsmedia-prod/azure.microsoft.com/en-us/documentation/articles/hdinsight-hbase-tutorial-get-started-linux/20160201111850/deploy-to-azure.png" alt="Deploy to Azure"></a>

3. 設定下列屬性：
		
	- CLUSTERNAME︰輸入要建立之新叢集的名稱。
	- CLUSTERLOGINUSERNAME︰輸入 Hadoop HTTP 使用者名稱。預設名稱為 **admin**。
	- CLUSTERLOGINPASSWORD︰輸入 Hadoop HTTP 使用者密碼。
	- SSHUSERNAME︰輸入 SSH 使用者名稱。預設名稱為 **sshuser**。
	- SSHPASSWORD︰輸入 SSH 使用者密碼。
	- LOCATION︰輸入資源群組名稱、叢集和預設儲存體帳戶的位置。
	- CLUSTERTYPE︰預設值為 **hadoop**。
	- CLUSTERWORKERNODECOUNT︰預設值為 2。
	- EDGENODESIZE︰選取其中一個 VM 大小。
	- EDGENODEPREFIX︰預設值是 **new**。若使用預設值，邊緣節點名稱是 **new-edgenode**。您可以從入口網站自訂前置詞。您也可以從範本自訂完整名稱。

4. 按一下 [確定] 儲存變更。
5. 在 [資源群組] 中，輸入新的資源群組名稱。
6. 按一下 [檢閱法律條款]，然後按一下 [購買]。
7. 選取 [釘選到儀表板]，然後按一下 [建立]。


## 存取邊緣節點

邊緣節點 ssh 端點是 <EdgeNodeName>.<ClusterName>-ssh.azurehdinsight.net:22。例如，new-edgenode.myedgenode0914-ssh.azurehdinsight.net:22。

在 Azure 入口網站上，邊緣節點會顯示為應用程式。入口網站可提供您相關資訊，以便使用 SSH 存取邊緣節點。

**確認邊緣節點 SSH 端點**

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 開啟具有邊緣節點的 HDInsight 叢集。
3. 從叢集刀鋒視窗中，按一下 [應用程式]。您應該會看到邊緣節點。預設名稱為 **new-edgenode**。
4. 按一下邊緣節點。您應該會看到 SSH 端點。

**在邊緣節點上使用 Hive**

5. 使用 SSH 連線到邊緣節點。請參閱[從 Linux、Unix 或 OS X 在 HDInsight 上搭配使用 SSH 與以 Linux 為基礎的 Hadoop](hdinsight-hadoop-linux-use-ssh-unix.md) 或[從 Windows 在 HDInsight 上搭配使用 SSH 與以 Linux 為基礎的 Hadoop](hdinsight-hadoop-linux-use-ssh-windows.md)。
6. 在使用 SSH 連線到邊緣節點之後，使用下列命令來開啟 Hive 主控台︰

		hive
7. 執行下列命令，以顯示叢集中的 Hive 資料表︰

		show tables;

## 刪除邊緣節點

您可以從 Azure 入口網站刪除邊緣節點。

**存取邊緣節點**

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 開啟具有邊緣節點的 HDInsight 叢集。
3. 從叢集刀鋒視窗中，按一下 [應用程式]。您應該會看到邊緣節點清單。
4. 以滑鼠右鍵按一下您想要刪除的邊緣節點，然後按一下 [刪除]。
5. 按一下 [**是**] 以確認。

## 後續步驟

在本文中，您已了解如何新增邊緣節點以及如何存取邊緣節點。若要深入了解，請參閱下列文章：

- [安裝 HDInsight 應用程式](hdinsight-apps-install-applications.md)︰了解如何將 HDInsight 應用程式安裝到您的叢集。
- [安裝自訂 HDInsight 應用程式](hdinsight-apps-install-custom-applications.md)︰了解如何將未發佈的 HDInsight 應用程式部署到 HDInsight。
- [發佈 HDInsight 應用程式](hdinsight-apps-publish-applications.md)︰了解如何將自訂 HDInsight 應用程式發佈至 Azure Marketplace。
- [MSDN：安裝 HDInsight 應用程式](https://msdn.microsoft.com/library/mt706515.aspx)︰了解如何定義 HDInsight 應用程式。
- [使用指令碼動作自訂以 Linux 為基礎的 HDInsight 叢集](hdinsight-hadoop-customize-cluster-linux.md)：了解如何使用指令碼動作來安裝其他應用程式。
- [使用 Resource Manager 範本在 HDInsight 中建立以 Linux 為基礎的 Hadoop 叢集](hdinsight-hadoop-create-linux-clusters-arm-templates.md)︰了解如何呼叫 Resource Manager 範本來建立 HDInsight 叢集。

<!---HONumber=AcomDC_0914_2016-->