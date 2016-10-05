<properties
   pageTitle="在 Mac OS X 上設定開發環境 | Microsoft Azure"
   description="安裝執行階段、SDK 和工具，並建立本機開發叢集。完成此設定之後，您就可以開始在 Mac OS X 上建置應用程式。"
   services="service-fabric"
   documentationCenter=".net"
   authors="seanmck"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotNet"
   ms.topic="get-started-article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="09/25/2016"
   ms.author="seanmck"/>

# 在 Mac OS X 上設定開發環境

> [AZURE.SELECTOR]
-[ Windows](service-fabric-get-started.md)
- [Linux](service-fabric-get-started-linux.md)
- [OSX](service-fabric-get-started-mac.md)

您可以建置 Service Fabric 應用程式以在使用 Mac OS X 的 Linux 叢集上執行。本文涵蓋如何設定您的 Mac 進行開發。

## 必要條件

Service Fabric 不會在 OS X 上以原生方式執行。若要執行本機 Service Fabric 叢集，我們提供使用 Vagrant 和 VirtualBox 的預先設定 Ubuntu 虛擬機器。開始之前，您需要：

- [Vagrant (v1.8.4 或更新版本)](http://wwww.vagrantup.com/downloads)
- [VirtualBox](http://www.virtualbox.org/wiki/Downloads)

## 建立本機 VM

若要建立包含 5 個節點 Service Fabric 叢集的本機 VM，請執行下列作業︰

1. 複製 Vagrantfile 儲存機制

    ```bash
    git clone https://github.com/azure/service-fabric-linux-vagrant-onebox.git
    ```

2. 瀏覽至儲存機制的本機複本

    ```bash
    cd service-fabric-linux-vagrant-onebox
    ```

3. (選擇性) 修改預設 VM 設定

    根據預設，本機 VM 的設定如下所示︰

    - 配置 3 GB 的記憶體
    - 在 IP 192.168.50.50 設定且能夠傳遞 Mac 主機流量的私用主機網路

    您可以變更上述任何一項設定或將其他組態新增至 Vagrantfile 中的 VM。如需設定選項的完整清單，請參閱 [Vagrant 文件](http://www.vagrantup.com/docs)。

4. 建立 VM

    ```bash
    vagrant up
    ```

    這個步驟可下載預先設定的 VM 映像、讓它在本機開機，然後在其中設定一個本機 Service Fabric 叢集。您預計需花幾分鐘的時間。如果安裝程式順利完成，您會在輸出中看到一則訊息，表示叢集正在啟動中。

    ![在 VM 佈建後啟動的叢集安裝程式][cluster-setup-script]

5. 瀏覽至位於 http://192.168.50.50:19080/Explorer 的 Service Fabric Explorer (假設您保留預設的私人網路 IP)，測試是否已正確設定叢集。

    ![從主機 Mac 檢視的 Service Fabric Explorer][sfx-mac]


## 安裝適用於 Eclipse Neon 的 Service Fabric 外掛程式 (選擇性)

Service Fabric 提供 Eclipse Neon IDE 的外掛程式，可簡化建置和部署 Java 服務的程序。

1. 在 Eclipse 中，請確定已安裝 Buildship 1.0.17 版或更新版本。您可以選擇 [說明] > [安裝詳細資料]，檢查已安裝的元件版本。您可以使用[這裡][buildship-update]的指示更新 Buildship。

2. 若要安裝 Service Fabric 外掛程式，請選擇 [說明] > [安裝新軟體...]

3. 在 [使用] 文字方塊中，輸入︰http://dl.windowsazure.com/eclipse/servicefabric。

4. 按一下 [新增]。

    ![Service Fabric 的 Eclipse Neon 外掛程式][sf-eclipse-plugin-install]

5. 選擇 Service Fabric 外掛程式，然後按 [下一步]。

6. 繼續進行安裝並接受使用者授權合約。

## 後續步驟

- [建立適用於 Linux 的第一個 Service Fabric 應用程式](service-fabric-create-your-first-linux-application-with-java.md)

<!-- Links -->

- [在 Azure 入口網站中建立 Service Fabric 叢集](service-fabric-cluster-creation-via-portal.md)
- [使用 Azure Resource Manager 建立 Service Fabric 叢集](service-fabric-cluster-creation-via-arm.md)
- [了解 Service Fabric 應用程式模型](service-fabric-application-model.md)

<!-- Images -->
[cluster-setup-script]: ./media/service-fabric-get-started-mac/cluster-setup-mac.png
[sfx-mac]: ./media/service-fabric-get-started-mac/sfx-mac.png
[sf-eclipse-plugin-install]: ./media/service-fabric-get-started-mac/sf-eclipse-plugin-install.png
[buildship-update]: https://projects.eclipse.org/projects/tools.buildship

<!---HONumber=AcomDC_0928_2016-->