<properties
   pageTitle="使用 Java 在 Linux 上建立第一個 Service Fabric 應用程式 | Microsoft Azure"
   description="使用 Java 建立和部署 Service Fabric 應用程式"
   services="service-fabric"
   documentationCenter="java"
   authors="seanmck"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.devlang="java"
   ms.topic="hero-article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="09/28/2016"
   ms.author="seanmck"/>


# 建立第一個 Azure Service Fabric 應用程式

> [AZURE.SELECTOR]
- [C Sharp](service-fabric-create-your-first-application-in-visual-studio.md)
- [Java](service-fabric-create-your-first-linux-application-with-java.md)

Service Fabric 提供了在 Linux 上建置服務的 .NET Core 和 Java SDK。在本教學課程中，我們將探討如何建立適用於 Linux 的應用程式以及使用 Java 建置服務。

## 必要條件

開始之前，請確定您已[設定 Linux 開發環境](service-fabric-get-started-linux.md)。如果您使用 Mac OS X，您可以[使用 Vagrant 在虛擬機器中設定 Linux 一整體環境](service-fabric-get-started-mac.md)。

## 建立應用程式

Service Fabric 應用程式可以包含一或多個服務，而每個服務在提供應用程式的功能時都有特定角色。適用於 Linux 的 Service Fabric SDK 包含 [Yeoman](http://yeoman.io/) 產生器，可讓您輕鬆建立第一個服務且稍後新增更多服務。讓我們使用 Yeoman 來建立具有單一服務的新應用程式。

1. 在終端機中，輸入 **yo azuresfjava**。

2. 為您的應用程式命名。

3. 選擇第一個服務的類型並加以命名。基於本教學課程的用途，我們將選擇 Reliable Actor 服務。

  ![適用於 Java 的 Service Fabric Yeoman 產生器][sf-yeoman]

>[AZURE.NOTE] 如需選項的詳細資訊，請參閱 [Service Fabric 程式設計模型概觀](service-fabric-choose-framework.md)。

## 建置應用程式

Service Fabric Yeoman 範本包含 [Gradle](https://gradle.org/) 的建置指令碼，可用來從終端機建置應用程式。

  ```bash
  gradle
  ```

## 部署應用程式

建置應用程式後，可以使用 Azure CLI 將它部署到本機叢集。

1. 連接到本機 Service Fabric 叢集。

    ```bash
    azuresfcli servicefabric cluster connect
    ```

2. 使用範本中所提供的安裝指令碼，將應用程式套件複製到叢集的映像存放區、註冊應用程式類型，以及建立應用程式的執行個體。

    ```bash
    cd myapp
    ./install.sh
    ```

3. 開啟瀏覽器並瀏覽至位於 http://localhost:19080/Explorer 的 Service Fabric Explorer (如果在 Mac OS X 上使用 Vagrant，請以 VM 的私人 IP 取代 localhost)。

4. 展開 [應用程式] 節點，請注意，您的應用程式類型現在有一個項目，而另一個則是該類型的第一個執行個體。

## 啟動測試用戶端並執行容錯移轉

動作項目專案沒有任何屬於自己的項目。它們需要其他服務或用戶端傳送訊息給它們。動作項目範本包含簡單的測試指令碼，您可以用來與動作項目服務互動。

1. 使用監看式公用程式執行指令碼，以查看動作項目服務的輸出。

    ```bash
    cd myactorsvcTestClient
    watch -n 1 ./testclient.sh
    ```

2. 在 Service Fabric Explorer 中，找出裝載動作項目服務主要複本的節點。在以下的螢幕擷取畫面中是節點 3。

    ![在 Service Fabric Explorer 中尋找主要複本][sfx-primary]

3. 按一下您在上一個步驟中找到的節點，然後從 [動作] 功能表選取 [停用 (重新啟動)]。這會重新啟動本機叢集中五個節點的其中一個，並強制容錯移轉至在另一個節點上執行的其中一個次要複本。當您這麼做時，請留意測試用戶端的輸出，並注意儘管是容錯移轉，計數器仍會繼續增加。

## 使用 Eclipse Neon 外掛程式建置和部署應用程式

如果您已安裝適用於 Eclipse Neon 的 Service Fabric 外掛程式，您可以使用它來建立、建置和部署使用 Java 建置的 Service Fabric 應用程式。

### 建立應用程式

Service Fabric 外掛程式可透過 Eclipse 擴充功能取得。

1. 在 Eclipse 中，選擇 [檔案] > [其他] > [Service Fabric]。您會看到一組選項，包括 [動作項目] 和 [容器]。

    ![Eclipse 中的 Service Fabric 範本][sf-eclipse-templates]

2. 在此情況下，選擇 [無狀態服務]。

3. 系統會要求您確認使用的 Service Fabric 層面，以最佳化搭配 Service Fabric 專案使用的 Eclipse。選擇 [是]。

### 部署應用程式

Service Fabric 範本包含一組用來建置和部署應用程式的 Gradle 工作，您可以透過 Eclipse 觸發這些工作。

1. 選擇 [執行] > [執行組態]。

2. 展開 [Gradle 專案] 並選擇 [ServiceFabricDeployer]。

3. 按一下 **[執行]**。

幾分鐘內即可建置和部署您的應用程式。您可以從 Service Fabric Explorer 監視其狀態。

## 後續步驟

- [深入了解 Reliable Actors](service-fabric-reliable-actors-introduction.md)

<!-- Images -->
[sf-yeoman]: ./media/service-fabric-create-your-first-linux-application-with-java/sf-yeoman.png
[sfx-primary]: ./media/service-fabric-create-your-first-linux-application-with-java/sfx-primary.png
[sf-eclipse-templates]: ./media/service-fabric-create-your-first-linux-application-with-java/sf-eclipse-templates.png

<!---HONumber=AcomDC_0928_2016-->