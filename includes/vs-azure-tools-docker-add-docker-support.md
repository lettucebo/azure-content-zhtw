1. 在 Visual Studio [方案總管] 中，用滑鼠右鍵按一下專案，然後從內容功能表中選取 [新增] > [Docker 支援]。

    ![新增 Docker 支援內容功能表](media/vs-azure-tools-docker-add-docker-support/docker-support-context-menu.png)

1. 將 Docker 支援新增至 ASP.NET Core Web 專案會導致將數個 Docker 相關檔案新增至專案，包括 Docker-Compose 檔案、部署 Windows PowerShell 指令碼以及 Docker 屬性檔。

    ![Docker 檔案已新增至專案](media/vs-azure-tools-docker-add-docker-support/docker-files-added.png)
    
> [AZURE.NOTE] 如果使用 [Docker for Windows Beta](https://beta.docker.com)，請開啟 [屬性\\Docker.props]，移除預設值，然後重新啟動 Visaul Studio 來讓值生效。
> 
> ```
> <DockerMachineName Condition="'$(DockerMachineName)'=='' "></DockerMachineName>
> ```

<!---HONumber=AcomDC_0921_2016-->