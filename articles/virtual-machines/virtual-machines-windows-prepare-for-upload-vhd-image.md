<properties
	pageTitle="準備要上傳至 Azure 的 Windows VHD| Microsoft Azure"
	description="準備 Windows VHD 在上傳至 Azure 之前所需的建議做法"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="genlin"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/18/2016"
	ms.author="glimoli;genli"/>

# 準備要上傳至 Azure 的 Windows VHD
若要從內部部署將 Windows VM 上傳至 Azure，您必須正確地準備虛擬硬碟 (VHD)。您必須完成數個建議的步驟，才能將 VHD 上傳至 Azure。本文說明如何準備要上傳至 Microsoft Azure 的 Windows VHD，同時也說明 [Sysprep 的使用時機和使用方式](#step23)。

## 準備虛擬磁碟

>[AZURE.NOTE] Azure 不支援較新的 VHDX 格式。VHD 必須是固定大小，非動態大小。如有需要，下列指示將詳細說明如何從 VHDX 或動態磁碟進行轉換。允許的 VHD 大小上限為 1023 GB。

確定 Windows VHD 目前在本機伺服器上的運作正常。先解決 VM 本身的任何錯誤，然後嘗試轉換或上傳至 Azure。

如果您需要將虛擬磁碟轉換為 Azure 所需的格式，請使用下列各節所述的其中一種方法。在執行任何虛擬磁碟轉換程序或 Sysprep 之前，請先備份 VM。

### 使用 HYPER-V 管理員進行轉換
- 開啟 Hyper-V 管理員，然後在左側選取您的本機電腦。在上方功能表中，按一下 [動作] > [編輯磁碟]。
	- 在 [尋找虛擬硬碟] 畫面上，瀏覽並選取您的虛擬磁碟。
	- 在下一個畫面中選取 [轉換]
		- 如果您需要從 VHDX 進行轉換，選取 [VHD]，然後按 [下一步]
		- 如果您需要從動態磁碟進行轉換，選取 [固定大小]，然後按 [下一步]

	- 瀏覽並選取 [新的 VHD 檔案路徑]。
	- 按一下 [完成] 以關閉。

### 使用 PowerShell 進行轉換
您可以使用 [CONVERT-VHD PowerShell Cmdlet](http://technet.microsoft.com/library/hh848454.aspx) 來轉換虛擬磁碟。在下列範例中，我們會從 VHDX 轉換為 VHD，並從動態轉換成固定類型：

```powershell
Convert-VHD –Path c:\test\MY-VM.vhdx –DestinationPath c:\test\MY-NEW-VM.vhd -VHDType Fixed
```

### 從 VMware VMDK 磁碟格式進行轉換
如果您的 Windows VM 映像是 [VMDK 檔案格式](https://en.wikipedia.org/wiki/VMDK)，使用 [Microsoft Virtual Machine Converter](https://www.microsoft.com/download/details.aspx?id=42497) 將它轉換為 VHD。如需詳細資訊，請閱讀部落格：[How to Convert a VMware VMDK to Hyper-V VHD (如何將 VMware VMDK 轉換為 Hyper-V VHD)](http://blogs.msdn.com/b/timomta/archive/2015/06/11/how-to-convert-a-vmware-vmdk-to-hyper-v-vhd.aspx)。

## 準備要上傳的 Windows 組態

> [AZURE.NOTE] 利用[系統管理權限](https://technet.microsoft.com/library/cc947813.aspx)執行下列所有命令。

1. 在路由表上移除任何靜態持續路由：

	- 若要檢視路由表，請執行 `route print`。
	- 檢查 [持續路由] 區段。如果有持續的路由，請使用 [route delete](https://technet.microsoft.com/library/cc739598.apx) 加以移除。

2. 移除 WinHTTP Proxy：

	```
	netsh winhttp reset proxy
	```

3. 將磁碟 SAN 原則設為 [Onlineall](https://technet.microsoft.com/library/gg252636.aspx)：

	```
	diskpart san policy=onlineall
	```

4. 使用適用於 Windows 的國際標準時間 (UTC)，並將 Windows 時間 (w32time) 服務的啟動類型設為 [自動]：

	```
	REG ADD HKLM\SYSTEM\CurrentControlSet\Control\TimeZoneInformation /v RealTimeIsUniversal /t REG_DWORD /d 1
	sc config w32time start= auto
	```


## 設定 Windows 服務
5. 確定以下的每個 Windows 服務都已設為 **Windows 預設值**。它們是使用下列清單所列的啟動設定來設定。您可以執行下列命令來重設啟動設定：

	```
	sc config bfe start= auto

	sc config dcomlaunch start= auto

	sc config dhcp start= auto

	sc config dnscache start= auto

	sc config IKEEXT start= auto

	sc config iphlpsvc start= auto

	sc config PolicyAgent start= demand

	sc config LSM start= auto

	sc config netlogon start= demand

	sc config netman start= demand

	sc config NcaSvc start= demand

	sc config netprofm start= demand

	sc config NlaSvc start= auto

	sc config nsi start= auto

	sc config RpcSs start= auto

	sc config RpcEptMapper start= auto

	sc config termService start= demand

	sc config MpsSvc start= auto

	sc config WinHttpAutoProxySvc start= demand

	sc config LanmanWorkstation start= auto

	sc config RemoteRegistry start= auto
	```


## 設定遠端桌面組態
6. 如果有任何自我簽署憑證繫結至遠端桌面通訊協定 (RDP) 接聽程式，請移除它們：

	```
	REG DELETE "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp\SSLCertificateSHA1Hash”
	```

	如需如何針對 RDP 接聽程式設定憑證的詳細資訊，請參閱 [Listener Certificate Configurations in Windows Server (Windows Server 中的接聽程式憑證組態)](https://blogs.technet.microsoft.com/askperf/2014/05/28/listener-certificate-configurations-in-windows-server-2012-2012-r2/)

7. 針對 RDP 服務設定 [KeepAlive](https://technet.microsoft.com/library/cc957549.aspx) 值：

	```
	REG ADD "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v KeepAliveEnable /t REG_DWORD  /d 1 /f

	REG ADD "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v KeepAliveInterval /t REG_DWORD  /d 1 /f

	REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp" /v KeepAliveTimeout /t REG_DWORD /d 1 /f
	```

8. 設定 RDP 服務的驗證模式：

	```
	REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v UserAuthentication /t REG_DWORD  /d 1 /f

	REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v SecurityLayer /t REG_DWORD  /d 1 /f

	REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v fAllowSecProtocolNegotiation /t REG_DWORD  /d 1 /f
	```

9. 將下列子機碼加入至登錄以啟用 RDP 服務：

	```
	REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD  /d 0 /f
	```


## 設定 Windows 防火牆規則
10. 允許 WinRM 透過三種防火牆設定檔 (網域、私人和公用)，並啟用 PowerShell 遠端服務：

	```
	Enable-PSRemoting -force
	```

11. 確定下列客體作業系統的防火牆規則已就緒：

	- 輸入

	```
	netsh advfirewall firewall set rule dir=in name="File and Printer Sharing (Echo Request - ICMPv4-In)" new enable=yes

	netsh advfirewall firewall set rule dir=in name="Network Discovery (LLMNR-UDP-In)" new enable=yes

	netsh advfirewall firewall set rule dir=in name="Network Discovery (NB-Datagram-In)" new enable=yes

	netsh advfirewall firewall set rule dir=in name="Network Discovery (NB-Name-In)" new enable=yes

	netsh advfirewall firewall set rule dir=in name="Network Discovery (Pub-WSD-In)" new enable=yes

	netsh advfirewall firewall set rule dir=in name="Network Discovery (SSDP-In)" new enable=yes

	netsh advfirewall firewall set rule dir=in name="Network Discovery (UPnP-In)" new enable=yes

	netsh advfirewall firewall set rule dir=in name="Network Discovery (WSD EventsSecure-In)" new enable=yes

	netsh advfirewall firewall set rule dir=in name="Windows Remote Management (HTTP-In)" new enable=yes

	netsh advfirewall firewall set rule dir=in name="Windows Remote Management (HTTP-In)" new enable=yes
	```

	- 輸入和輸出

	```
	netsh advfirewall firewall set rule group="Remote Desktop" new enable=yes

	netsh advfirewall firewall set rule group="Core Networking" new enable=yes
	```

	- 輸出

	```
	netsh advfirewall firewall set rule dir=out name="Network Discovery (LLMNR-UDP-Out)" new enable=yes

	netsh advfirewall firewall set rule dir=out name="Network Discovery (NB-Datagram-Out)" new enable=yes

	netsh advfirewall firewall set rule dir=out name="Network Discovery (NB-Name-Out)" new enable=yes

	netsh advfirewall firewall set rule dir=out name="Network Discovery (Pub-WSD-Out)" new enable=yes

	netsh advfirewall firewall set rule dir=out name="Network Discovery (SSDP-Out)" new enable=yes

	netsh advfirewall firewall set rule dir=out name="Network Discovery (UPnPHost-Out)" new enable=yes

	netsh advfirewall firewall set rule dir=out name="Network Discovery (UPnP-Out)" new enable=yes

	netsh advfirewall firewall set rule dir=out name="Network Discovery (WSD Events-Out)" new enable=yes

	netsh advfirewall firewall set rule dir=out name="Network Discovery (WSD EventsSecure-Out)" new enable=yes

	netsh advfirewall firewall set rule dir=out name="Network Discovery (WSD-Out)" new enable=yes
	```


## 其他 Windows 組態步驟
12. 執行 `winmgmt /verifyrepository` 以確認 Windows Management Instrumentation (WMI) 存放庫是一致的。如果存放庫損毀，請參閱[此部落格文章](https://blogs.technet.microsoft.com/askperf/2014/08/08/wmi-repository-corruption-or-not)。

13. 確定開機設定資料 (BCD) 設定符合下列設定：

	```
	bcdedit /set {bootmgr} integrityservices enable

	bcdedit /set {default} device partition=C:

	bcdedit /set {default} integrityservices enable

	bcdedit /set {default} recoveryenabled Off

	bcdedit /set {default} osdevice partition=C:

	bcdedit /set {default} bootstatuspolicy IgnoreAllFailures
	```

14. 移除任何額外的傳輸驅動程式介面篩選，例如分析 TCP 封包的軟體。
15. 若要確定磁碟狀況良好且一致，請執行 `CHKDSK /f` 命令。
16.	將與實體元件或任何其他虛擬化技術相關的所有其他協力廠商軟體和驅動程式解除安裝。
17. 確定協力廠商應用程式並未使用連接埠 3389。在 Azure 中，此連接埠是由 RDP 服務所使用。您可以使用 `netstat -anob` 命令來檢查應用程式正在使用的連接埠。
18.	如果您想要上傳的 Windows VHD 是一個網域控制站，請遵循[這些額外步驟](https://support.microsoft.com/kb/2904015)來準備磁碟。
19.	將 VM 重新開機，以確保 Windows 仍然狀況良好，可以使用 RDP 連線來達成。
20.	重設目前的本機系統管理員密碼，並確定您可以使用此帳戶，透過 RDP 連線登入 Windows。此存取權限會受到「允許透過遠端桌面服務登入」原則物件所控制。此物件位於 [電腦設定]\\[Windows 設定]\\[安全性設定]\\[本機原則]\\[使用者權限指派] 之下。


## 安裝 Windows 更新
22. 安裝 Windows 的最新更新。如果這不可行，請確定已安裝下列更新：

	- [KB3137061](https://support.microsoft.com/kb/3137061) Microsoft Azure VM 無法從網路中斷情況復原，並發生資料損毀問題

	- [KB3115224](https://support.microsoft.com/kb/3115224) 為在 Windows Server 2012 R2 或 Windows Server 2012 主機上執行的 VM 提供的可靠性改進

	- [KB3140410](https://support.microsoft.com/kb/3140410) MS16-031：Microsoft Windows 的安全性更新，可解決提高權限問題：2016 年 3 月 8 日

	- [KB3063075](https://support.microsoft.com/kb/3063075) 當您在 Microsoft Azure 中執行 Windows Server 2012 R2 虛擬機器時，會記錄許多識別碼 129 的事件

	- [KB3137061](https://support.microsoft.com/kb/3137061) Microsoft Azure VM 無法從網路中斷情況復原，並發生資料損毀問題

	- [KB3114025](https://support.microsoft.com/kb/3114025) 當您從 Windows 8.1 或 Server 2012 R2 存取 Azure 檔案儲存體時效能變慢

	- [KB3033930](https://support.microsoft.com/kb/3033930) Hotfix 可針對 Windows 中的 Azure 服務，提高每個處理序在 RIO 緩衝區中的 64K 限制

	- [KB3004545](https://support.microsoft.com/kb/3004545) 您無法在 Windows 中透過 VPN 連線，存取裝載於 Azure 裝載服務上的虛擬機器

	- [KB3082343](https://support.microsoft.com/kb/3082343) 當 Azure 網站間 VPN 通道使用 Windows Server 2012 R2 RRAS 時，遺失了跨單位 VPN 連線

	- [KB3140410](https://support.microsoft.com/kb/3140410) MS16-031：Microsoft Windows 的安全性更新，可解決提高權限問題：2016 年 3 月 8 日

	- [KB3146723](https://support.microsoft.com/kb/3146723) MS16-048：CSRSS 安全性更新的描述︰2016 年 4 月 12 日
	- [KB2904100](https://support.microsoft.com/kb/2904100) 在 Windows 中，系統會在磁碟 I/O 期間凍結<a id="step23"></a>
23. 如果您想要建立映像以便從中部署多部電腦，您需要先執行 `sysprep` 來將該映像一般化，然後將 VHD 上傳至 Azure。您不需要執行 `sysprep`，即可使用特製化 VHD。如需如何建立一般化映像的詳細資訊，請參閱下列文章：

	- [使用 Resource Manager 部署模型，從現有的 Azure VM 建立 VM 映像](virtual-machines-windows-capture-image.md)
	- [使用傳統部署模型，從現有的 Azure VM 建立 VM 映像](virtual-machines-windows-classic-capture-image.md)
	- [伺服器角色的 Sysprep 支援](https://msdn.microsoft.com/windows/hardware/commercialize/manufacture/desktop/sysprep-support-for-server-roles)



## 建議的額外組態

下列設定不會影響 VHD 上傳。不過，我們強烈建議您設定它們。

- 安裝 [Azure 虛擬機器代理程式](http://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409)。安裝代理程式之後，您就能啟用 VM 擴充功能。VM 擴充功能實作了您想要與 VM 搭配使用的大部分重要功能，例如重設密碼、設定 RDP，以及許多其他功能。

- 傾印記錄檔能幫助您進行 Windows 損毀問題的疑難排解。啟用傾印記錄檔收集：

	```
	REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 2 /f`

	REG ADD "HKLM\SOFTWARE\Microsoft\Windows\Windows Error Reporting\LocalDumps" /v DumpFolder /t REG_EXPAND_SZ /d "c:\CrashDumps" /f

	REG ADD "HKLM\SOFTWARE\Microsoft\Windows\Windows Error Reporting\LocalDumps" /v DumpCount /t REG_DWORD /d 10 /f

	REG ADD "HKLM\SOFTWARE\Microsoft\Windows\Windows Error Reporting\LocalDumps" /v DumpType /t REG_DWORD /d 2 /f

	sc config wer start= auto
	```

- 在 Azure 中建立 VM 之後，在磁碟機 D 上設定系統定義的大小分頁檔：

	```
	REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management" /t REG_MULTI_SZ /v PagingFiles /d "D:\pagefile.sys 0 0" /f
	```


## 後續步驟

- [將 Windows VM 映像上傳至 Azure 供 Resource Manager 部署使用](virtual-machines-windows-upload-image.md)

<!---HONumber=AcomDC_0921_2016-->