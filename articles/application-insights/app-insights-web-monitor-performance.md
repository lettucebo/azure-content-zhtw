<properties 
	pageTitle="使用 Application Insights 來監視應用程式的健康情況和流量" 
	description="開始使用 Application Insights。分析內部部署或 Microsoft Azure 應用程式的使用情況、可用性和效能。" 
	services="application-insights" 
    documentationCenter=""
	authors="alancameronwills" 
	manager="douge"/>

<tags 
	ms.service="application-insights" 
	ms.workload="tbd" 
	ms.tgt_pltfrm="ibiza" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="11/25/2015" 
	ms.author="awills"/>
 
# 監視 Web 應用程式的效能

Application Insights 目前僅供預覽。


確認應用程式的運作狀況良好，以及迅速找出任何失敗。[Application Insights][start] 能指出任何效能問題和例外狀況，以及協助您尋找及診斷根本原因。

Application Insights 可以監視 Java 和 ASP.NET Web 應用程式與服務、WCF 服務。這些服務可以裝載在內部部署、虛擬機器上，或做為 Microsoft Azure 網站。

用戶端方面，Application Insights 可從網頁及各種裝置 (包括 iOS、Android 和 Windows 市集應用程式) 上取得遙測。


## <a name="setup"></a>設定效能監視

如果您尚未將 Application Insights 新增至專案中 (亦即專案沒有 ApplicationInsights.config)，請選擇以下任一種方法來開始使用：

* [ASP.NET Web 應用程式](app-insights-asp-net.md)
 * [加入例外狀況監視](app-insights-asp-net-exceptions.md)
 * [加入相依性監視](app-insights-monitor-performance-live-website-now.md)
* [J2EE Web 應用程式](app-insights-java-get-started.md)
 * [加入相依性監視](app-insights-java-agent.md)


## <a name="view"></a>探索效能度量

在 [Azure 入口網站](https://portal.azure.com)中，瀏覽至您為應用程式設定的 Application Insights 資源。概觀刀鋒視窗會顯示基本的效能資料：



按一下任一圖表可查看詳細資料，也能查看較長期的結果。例如，按一下 [要求] 磚，然後選取時間範圍：


![按一下詳細資料的方式來選取時間範圍](./media/app-insights-web-monitor-performance/appinsights-48metrics.png)

按一下圖表以選擇它要顯示的度量，或是加入新圖表並選取其度量：

![按一下圖形以選擇度量](./media/app-insights-web-monitor-performance/appinsights-61perfchoices.png)

> [AZURE.NOTE] **取消核取所有度量**可查看所有可供選擇的項目。度量分為多個群組；當您選取群組的任何成員時，只有該群組的其他成員會出現。


## <a name="metrics"></a>這具有哪些意義？ 效能磚和報告

您可以取用多種效能度量。我們從預設顯示在應用程式分頁中的度量開始討論。


### 要求

在指定期間內接收到的 HTTP 要求數量。將此度量與其他報告中的結果比較，可了解應用程式在各種負載下的行為。

HTTP 要求包括頁面、資料及影像的所有 GET 或 POST 要求。

按一下磚可取得特定 URL 的計數。

### 平均回應時間

測量從 Web 要求進入應用程式到傳回回應之間的時間。

資料點會指出移動平均值。在有許多要求的情況下，可能會有一些要求偏離平均值，但在圖表中沒有顯著的高低起伏。

找尋異常的高峰。一般來說，當要求增加時，回應時間也會上升。如果出現不相稱的上升跡象，表示應用程式可能遭遇到資源限制 (例如，應用程式使用之服務的 CPU 或容量)。

按一下磚可取得特定 URL 的時間。

![](./media/app-insights-web-monitor-performance/appinsights-42reqs.png)


### 最慢的要求

![](./media/app-insights-web-monitor-performance/appinsights-44slowest.png)

顯示可能需要進行效能調整的要求。


### 失敗的要求

![](./media/app-insights-web-monitor-performance/appinsights-46failed.png)

丟出無法攔截之例外狀況的要求計數。

按一下磚可查看特定失敗的詳細資料；選取個別要求可查看要求的詳細資料。

系統只會針對各項檢查保留一個具代表性的失敗範例。

### 其他度量

若要查看其他可顯示的度量，請按一下圖表，然後取消選取所有度量以查看可供使用的完整集合。按一下 (i) 可查看各項度量的定義。

![取消選取所有度量以查看完整的集合](./media/app-insights-web-monitor-performance/appinsights-62allchoices.png)


選取任何度量將會停用其他度量，使其無法顯示在同一個圖表中。

## 系統效能計數器


Windows 提供多種效能計數器，您也可以自行定義。

(若是 Azure 上裝載的應用程式，請 [將 Azure 診斷傳送至 Application Insights](app-insights-azure-diagnostics.md)。)

若要查看一組常見的[效能計數器](http://www.codeproject.com/Articles/8590/An-Introduction-To-Performance-Counters)，請開啟 [伺服器] 刀鋒視窗。您也可以編輯圖表，然後從 [效能計數器] 區段選取度量來選擇計數器︰

![](./media/app-insights-web-monitor-performance/sys-perf.png)

透過在 Windows 系統上使用 PowerShell 命令 [`Get-Counter -ListSet *`](https://technet.microsoft.com/library/hh849685.aspx)，即可決定您的系統上可用的一組完整度量。

如果您想要的計數器不在 [度量] 清單中，您可以將它們加入 SDK 收集的集合。開啟 ApplicationInsights.config，然後編輯效能收集器指示詞：

    <Add Type="Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.PerformanceCollectorModule, Microsoft.AI.PerfCounterCollector">
      <Counters>
        <Add PerformanceCounter="\Objects\Processes"/>
        <Add PerformanceCounter="\Sales(electronics)# Items Sold" ReportAs="Item sales"/>
      </Counters>
    </Add>

您可以擷取標準計數器以及您自行實作的計數器。`\Objects\Processes` 適用於所有 Windows 系統；`\Sales...` 是可在 Web 伺服器中實作的自訂計數器範例。

格式為 `\Category(instance)\Counter"`，若是沒有執行個體的類別，則為 `\Category\Counter`。


若計數器名稱含有下列項目以外的字元，則需要 `ReportAs`：字母、圓角括號、斜線、連字號、底線、空格和點。

如果您指定執行個體，系統會收集它做為報告度量的 "CounterInstanceName" 維度。

### 在程式碼中收集效能計數器

若要收集系統效能計數器並將其推送到 Application Insights，可以使用下列程式碼片段：

    var perfCollectorModule = new PerformanceCollectorModule();
    perfCollectorModule.Counters.Add(new PerformanceCounterCollectionRequest(
      @"\.NET CLR Memory([replace-with-application-process-name])# GC Handles", "GC Handles")));
    perfCollectorModule.Initialize(TelemetryConfiguration.Active);

或者，您可以透過您建立的自訂度量執行相同的作業︰

    var perfCollectorModule = new PerformanceCollectorModule();
    perfCollectorModule.Counters.Add(new PerformanceCounterCollectionRequest(
      @"\Sales(electronics)# Items Sold", "Items sold"));
    perfCollectorModule.Initialize(TelemetryConfiguration.Active);

此外，如果您想要

### 例外狀況計數

「例外狀況率和例外狀況度量之間的差異為何？」

* 例外狀況率是系統效能計數器。CLR 會計算所有擲回之已處理和未處理的例外狀況，並依據間隔的長度將總數分割為取樣間隔。Application Insights SDK 會收集此結果並將它傳送至入口網站。
* 例外狀況是在圖表的取樣間隔中由入口網站接收之 TrackException 報告的計數。它只包含您程式碼中撰寫 TrackException 呼叫所在位置的已處理例外狀況，並且不包含所有的[未處理例外狀況](app-insights-asp-net-exceptions.md)。 

## 設定警示

若要在任何度量有不尋常的值時收到電子郵件通知，請加入警示。您可以選擇將電子郵件傳送給帳戶管理員，或傳送給特定的電子郵件地址。

![](./media/app-insights-web-monitor-performance/appinsights-413setMetricAlert.png)

設定其他屬性之前的資源。如果您想要設定效能或使用度量的相關警示，請勿選擇 webtest 資源。

請小心注意系統要求您輸入臨界值時所使用的單位。

我沒有看到 [新增警示] 按鈕 - 您只擁有此群組帳戶的唯讀權限嗎？ 請洽詢帳戶管理員。

## <a name="diagnosis"></a>診斷問題

以下是幾個尋找及診斷效能問題的訣竅：

* 設定 [Web 測試][availability] \(英文)，以在網站故障、回應異常或過於緩慢時收到警示。 
* 比較要求計數與其他度量，了解失敗或回應過慢的情況是否與負載有關。
* 在程式碼中[插入及搜尋追蹤陳述式][diagnostic] \(英文) 有助於找出問題所在。

## <a name="next"></a>接續步驟

[Web 測試][availability] \(英文) - 定期從全世界傳送 Web 要求給應用程式。

[擷取及搜尋診斷追蹤][diagnostic] \(英文) - 插入追蹤呼叫並詳查結果，以便找出問題所在。

[流量追蹤][usage] \(英文) - 了解使用者使用應用程式的情況。

[疑難排解][qna] \(英文) - 和問答集

## 影片

[AZURE.VIDEO performance-monitoring-application-insights]

<!--Link references-->

[availability]: app-insights-monitor-web-app-availability.md
[diagnostic]: app-insights-diagnostic-search.md
[greenbrown]: app-insights-asp-net.md
[qna]: app-insights-troubleshoot-faq.md
[redfield]: app-insights-monitor-performance-live-website-now.md
[start]: app-insights-overview.md
[usage]: app-insights-web-track-usage.md

 

<!---HONumber=AcomDC_0330_2016-->