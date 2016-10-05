<properties
   pageTitle="設定開發環境 | Microsoft Azure"
   description="安裝執行階段、SDK 和工具，並建立本機開發叢集。完成此設定之後，您就可以開始建置應用程式。"
   services="service-fabric"
   documentationCenter=".net"
   authors="rwike77"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotNet"
   ms.topic="get-started-article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="09/13/2016"
   ms.author="ryanwi"/>

# 準備您的開發環境

> [AZURE.SELECTOR]
-[ Windows](service-fabric-get-started.md)
- [Linux](service-fabric-get-started-linux.md)
- [OSX](service-fabric-get-started-mac.md)

 若要在您的開發機器上建置並執行 [Azure Service Fabric 應用程式][1]，請安裝執行階段、SDK 和工具。您也必須執行 SDK 中包含的 Windows PowerShell 指令碼。

## 必要條件
### 支援的作業系統版本
下列為支援開發的作業系統版本：

- Windows 7
- Windows 8/Windows 8.1
- Windows Server 2012 R2
- Windows 10

>[AZURE.NOTE] 根據預設，Windows 7 只包含 Windows PowerShell 2.0。Service Fabric PowerShell Cmdlet 需要 PowerShell 3.0 或更新版本。您可以從 Microsoft 下載中心[下載 Windows PowerShell 5.0][powershell5-download]。

## 安裝執行階段、SDK 和工具

Web Platform Installer 針對 Service Fabric 開發提供兩個組態︰

- [安裝適用於 Visual Studio 2015 的 Service Fabric 執行階段、SDK 和工具 (需要 Visual Studio 2015 Update 2 或更新版本)][full-bundle-vs2015]
- [僅安裝 Service Fabric 執行階段和 SDK (不安裝 Visual Studio 工具)][core-sdk]

## 啟用 PowerShell 指令碼執行

Service Fabric 會使用 Windows PowerShell 指令碼，以便建立本機開發叢集，以及從 Visual Studio 部署應用程式。根據預設，Windows 會封鎖這些指令碼的執行。若要啟用其，您必須修改 PowerShell 執行原則。以系統管理員身分開啟 PowerShell 並輸入下列命令：

```powershell
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Force -Scope CurrentUser
```

## 後續步驟
現在您的開發環境已完成設定，您可以開始建置和執行應用程式。

- [在 Visual Studio 中建立第一個 Service Fabric 應用程式](service-fabric-create-your-first-application-in-visual-studio.md)
- [了解如何在本機叢集上部署和管理應用程式](service-fabric-get-started-with-a-local-cluster.md)
- [深入了解程式設計模型：Reliable Services 和 Reliable Actors](service-fabric-choose-framework.md)
- [請查看 GitHub 上的 Service Fabric 程式碼範例](https://aka.ms/servicefabricsamples)
- [使用 Service Fabric 總管將叢集視覺化](service-fabric-visualizing-your-cluster.md)
- [遵循 Service Fabric 學習路徑來取得廣泛的平台簡介](https://azure.microsoft.com/documentation/learning-paths/service-fabric/)

[1]: http://azure.microsoft.com/campaigns/service-fabric/ "Service Fabric 活動頁面"
[2]: http://go.microsoft.com/fwlink/?LinkId=517106 "VS RC"
[full-bundle-vs2015]: http://www.microsoft.com/web/handlers/webpi.ashx?command=getinstallerredirect&appid=MicrosoftAzure-ServiceFabric-VS2015 "VS 2015 WebPI 連結"
[full-bundle-dev15]: http://www.microsoft.com/web/handlers/webpi.ashx?command=getinstallerredirect&appid=MicrosoftAzure-ServiceFabric-Dev15 "Dev15 WebPI 連結"
[core-sdk]: http://www.microsoft.com/web/handlers/webpi.ashx?command=getinstallerredirect&appid=MicrosoftAzure-ServiceFabric-CoreSDK "Core SDK WebPI 連結"
[powershell5-download]: https://www.microsoft.com/zh-TW/download/details.aspx?id=50395

<!---HONumber=AcomDC_0928_2016-->