<properties
   pageTitle="在 Linux 上設定開發環境 | Microsoft Azure"
   description="在 Linux 上安裝執行階段和 SDK，並建立本機開發叢集。完成此設定之後，您就可以開始建置應用程式。"
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
   ms.date="09/26/2016"
   ms.author="seanmck"/>

# 在 Linux 上準備您的開發環境


> [AZURE.SELECTOR]
-[ Windows](service-fabric-get-started.md)
- [Linux](service-fabric-get-started-linux.md)
- [OSX](service-fabric-get-started-mac.md)

 若要在 Linux 開發機器上部署和執行 [Azure Service Fabric 應用程式](service-fabric-application-model.md)，請安裝執行階段和通用 SDK。您也可以安裝 Java 和 .NET Core 的選擇性 SDK。

## 必要條件
### 支援的作業系統版本
下列為支援開發的作業系統版本：

- Ubuntu 16.04 (Xenial Xerus)

## 更新 apt 來源

若要透過 apt-get 安裝 SDK 和相關聯的執行階段套件，您必須先更新 apt 來源。

1. 開啟終端機。
2. 將 Service Fabric 儲存機制新增至來源清單。

    ```bash
    sudo sh -c 'echo "deb [arch=amd64] http://apt-mo.trafficmanager.net/repos/servicefabric/ trusty main" > /etc/apt/sources.list.d/servicefabric.list'
    ```

3. 將新的 GPG 金鑰新增至 apt keyring。

    ```bash
    sudo apt-key adv --keyserver apt-mo.trafficmanager.net --recv-keys 417A0893
    ```

4. 根據新增的儲存機制重新整理套件清單。

    ```bash
    sudo apt-get update
    ```

## 安裝和設定 SDK

更新來源後，您可以安裝 SDK。

1. 安裝 Service Fabric SDK 套件。系統會要求確認安裝並同意授權合約。

    ```bash
    sudo apt-get install servicefabricsdkcommon
    ```

2. 執行 SDK 安裝指令碼。

    ```bash
    sudo /opt/microsoft/sdk/servicefabric/common/sdkcommonsetup.sh
    ```

## 設定 Azure 跨平台 CLI

[Azure 跨平台 CLI][azure-xplat-cli-github] 包含與 Service Fabric 實體 (包括叢集和應用程式) 進行互動的命令。它是以 Node.js 為基礎，所以先[確認您已安裝 Node][install-node]，再繼續執行下面的指示。

1. 將 github 儲存機制複製到您的開發電腦。

    ```bash
    git clone https://github.com/Azure/azure-xplat-cli.git
    ```

2. 切換至所複製的儲存機制，並使用 Node Package Manager (npm) 安裝 CLI 的相依項目。

    ```bash
    cd azure-xplat-cli
    npm install
    ```

3. 建立從所複製儲存機制的 bin/azure 資料夾至 /usr/bin/azure 的符號連結，以便將它新增至您的路徑並從任何目錄取得命令。

    ```bash
    sudo ln -s $(pwd)/bin/azure /usr/bin/azure
    ```

4. 最後，啟用自動完成 Service Fabric 命令。

    ```bash
    azure --completion >> ~/azure.completion.sh
    echo 'source ~/azure.completion.sh' >> ~/.bash_profile
    source ~/azure.completion.sh
    ```

## 設定本機叢集

如果一切都已順利安裝，您應該能夠啟動本機叢集。

1. 執行叢集安裝指令碼。

    ```bash
    sudo /opt/microsoft/sdk/servicefabric/common/clustersetup/devclustersetup.sh
    ```

2. 開啟瀏覽器並瀏覽至 http://localhost:19080/Explorer。如果叢集已經啟動，您應會看見 Service Fabric Explorer 儀表板。

    ![Linux 上的 Service Fabric Explorer][sfx-linux]

此時，您可以根據客體容器或來賓可執行檔，部署預先建置的 Service Fabric 應用程式套件或新的套件。若要使用 Java 或 .NET Core SDK 建置新的服務，請遵循下列選擇性設定步驟。

## 安裝 Java SDK 和 Eclipse Neon 外掛程式 (選擇性)

Java SDK 提供了使用 Java 建置 Service Fabric 服務所需的程式庫和範本。

1. 安裝 Java SDK 套件。

    ```bash
    sudo apt-get install servicefabricsdkjava
    ```

2. 執行 SDK 安裝指令碼。

    ```bash
    sudo /opt/microsoft/sdk/servicefabric/java/sdkjavasetup.sh
    ```

您可以從 Eclipse Neon IDE 安裝適用於 Service Fabric 的 Eclipse 外掛程式。

1. 在 Eclipse 中，請確定已安裝 Buildship 1.0.17 版或更新版本。您可以選擇 [說明] > [安裝詳細資料]，檢查已安裝的元件版本。您可以使用[這裡][buildship-update]的指示更新 Buildship。

2. 若要安裝 Service Fabric 外掛程式，請選擇 [說明] > [安裝新軟體...]

3. 在 [使用] 文字方塊中，輸入︰http://dl.windowsazure.com/eclipse/servicefabric

4. 按一下 [新增]。

    ![Eclipse 外掛程式][sf-eclipse-plugin]

5. 選擇 Service Fabric 外掛程式，然後按 [下一步]。

6. 繼續進行安裝並接受使用者授權合約。

## 安裝 .NET Core SDK (選擇性)

.NET Core SDK 提供了使用跨平台 .NET Core 建置 Service Fabric 服務所需的程式庫和範本。

1. 安裝 .NET Core SDK 套件。

    ```bash
    sudo apt-get install servicefabricsdkcsharp
    ```

2. 執行 SDK 安裝指令碼。

    ```bash
    sudo /opt/microsoft/sdk/servicefabric/csharp/sdkcsharpsetup.sh
    ```

## 後續步驟

- [在 Linux 上建立第一個 Java 應用程式](service-fabric-create-your-first-linux-application-with-java.md)

- [在 OSX 上準備您的開發環境](service-fabric-get-started-mac.md)


<!-- Links -->

[azure-xplat-cli-github]: https://github.com/Azure/azure-xplat-cli
[install-node]: https://nodejs.org/en/download/package-manager/#installing-node-js-via-package-manager
[buildship-update]: https://projects.eclipse.org/projects/tools.buildship

<!--Images -->

[sf-eclipse-plugin]: ./media/service-fabric-get-started-linux/service-fabric-eclipse-plugin.png
[sfx-linux]: ./media/service-fabric-get-started-linux/sfx-linux.png

<!---HONumber=AcomDC_0928_2016-->