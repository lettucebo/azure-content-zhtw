<properties
   pageTitle="使用 Visual Studio 疑難排解 Windows 上的 Docker 用戶端錯誤 | Microsoft Azure"
   description="疑難排解使用 Visual Studio 在 Windows 上建立及部署 Web 應用程式到 Docker 時您會遇到的問題。"
   services="azure-container-service"
   documentationCenter="na"
   authors="mlearned"
   manager="douge"
   editor="" />
<tags
   ms.service="multiple"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="multiple"
   ms.date="06/08/2016"
   ms.author="allclark" />

# 疑難排解 Visual Studio Docker 開發

使用 Visual Studio Tools for Docker Preview 時，可能會因預覽本質而發生一些問題。以下是一些常見問題和解決方法。


## 無法驗證磁碟區對應
需要磁碟區對應才能與容器中的應用程式資料夾共用應用程式的原始程式碼和二進位檔。docker-compose.dev.debug.yml 和 docker-compose.dev.release.yml 檔案中包含特定的磁碟區對應。當主機電腦上的檔案變更時，容器就會在類似的資料夾結構中反映這些變更。

若要啟用磁碟區對應，請從 Docker For Windows「白鯨」系統匣圖示開啟 [設定...]，然後選取 [共用磁碟機] 索引標籤。檢查裝載專案的磁碟機代號以及 %USERPROFILE% 所在的磁碟機代號，然後按一下 [套用]，以確保它們可共用。

若要測試磁碟區對應功能是否正常，請在磁碟機共用之後，從 Visual Studio 內重建並按 F5，或從命令提示字元嘗試下列命令︰

在 Windows 命令提示字元

[注意︰這邊假設您的「使用者」資料夾位於已共用的 "C" 磁碟機上。如果您共用的是不同的磁碟機，請視需要更新]
```
docker run -it -v /c/Users/Public:/wormhole busybox
```

在 Linux 容器

```
/ # ls
```

您應該會看到「使用者/公用」資料夾中的目錄清單。如果未顯示任何檔案，而且「/c/使用者/公用」資料夾不是空的，則未正確地設定磁碟區對應。

```
bin       etc       proc      sys       usr       wormhole
dev       home      root      tmp       var
```

切換至 wormhole 目錄，以查看 `/c/Users/Public` 目錄的內容：

```
/ # cd wormhole/
/wormhole # ls
AccountPictures  Downloads        Music            Videos
Desktop          Host             NuGet.Config     desktop.ini
Documents        Libraries        Pictures
/wormhole #
```

**注意：**使用 Linux VM 時，容器檔案系統區分大小寫。

##建置：「PrepareForBuild」工作意外失敗。

Microsoft.DotNet.Docker.CommandLine.ClientException︰嘗試連線時發生錯誤︰

請確認預設的 Docker 主機正在執行。開啟命令提示字元並執行︰

```
docker info
```

如果此命令傳回錯誤，則請嘗試啟動 **Docker For Windows** 桌面應用程式。如果桌面應用程式正在執行，則應該可以在系統匣中看到 [白鯨] 圖示。以滑鼠右鍵按一下系統匣圖示，然後開啟 [設定]。依序按一下 [重設] 索引標籤和 [重新啟動 Docker...]。

##從 0.31 版手動升級至 0.40 版


1. 備份專案
1. 刪除專案中的下列檔案︰

    ```
      Dockerfile
      Dockerfile.debug
      DockerTask.ps1
      docker-compose-yml
      docker-compose.debug.yml
      .dockerignore
      Properties\Docker.props
      Properties\Docker.targets
    ```

1. 關閉方案，並從 .xproj 檔案中移除下列幾行︰

    ```
      <DockerToolsMinVersion>0.xx</DockerToolsMinVersion>
      <Import Project="Properties\Docker.props" />
      <Import Project="Properties\Docker.targets" />
    ```

1. 重新開啟方案
1. 從 Properties\\launchSettings.json 檔案移除下列幾行︰

    ```
      "Docker": {
        "executablePath": "%WINDIR%\\System32\\WindowsPowerShell\\v1.0\\powershell.exe",
        "commandLineArgs": "-ExecutionPolicy RemoteSigned .\\DockerTask.ps1 -Run -Environment $(Configuration) -Machine '$(DockerMachineName)'"
      }
    ```

1. 在 publishOptions 中從 project.json 移除下列與 Docker 相關的檔案︰

    ```
    "publishOptions": {
      "include": [
        ...
        "docker-compose.yml",
        "docker-compose.debug.yml",
        "Dockerfile.debug",
        "Dockerfile",
        ".dockerignore"
      ]
    },
    ```

1. 解除安裝舊版本並安裝 Docker 工具 0.40，然後從 ASP.Net Core Web 或主控台應用程式的內容功能表再次。這會將新的必要 Docker 構件新增回您的專案。

## 嘗試選取 [新增] -> [Docker 支援] 或偵錯 (F5) 容器中的 ASP.NET 核心應用程式時出現錯誤對話方塊

我們偶爾會發現，先解除安裝再重新安裝擴充功能之後，Visual Studio 的 MEF (Managed Extensibility Framework) 快取會損毀。若發生這種情況，就會導致在新增 Docker 支援和 (或) 嘗試執行或偵錯 (F5) ASP.NET 核心應用程式時出現各種錯誤對話方塊。暫時的解決方法是執行下列步驟來刪除和重新產生 MEF 快取。

1. 關閉所有 Visual Studio 執行個體
1. 開啟 %USERPROFILE%\\AppData\\Local\\Microsoft\\VisualStudio\\14.0\\
1. 刪除下列資料夾
     ```
       ComponentModelCache
       Extensions
       MEFCacheBackup
    ```
1. 開啟 Visual Studio
1. 再次嘗試案例

<!---HONumber=AcomDC_0921_2016-->