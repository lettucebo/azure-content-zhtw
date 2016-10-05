<properties
	pageTitle="Azure Mobile Engagement iOS SDK 版本資訊 | Microsoft Azure"
	description="Azure Mobile Engagement iOS SDK 的最新更新與程序"
	services="mobile-engagement"
	documentationCenter="mobile"
	authors="piyushjo"
	manager="erikre"
	editor="" />

<tags
	ms.service="mobile-engagement"
	ms.workload="mobile"
	ms.tgt_pltfrm="mobile-ios"
	ms.devlang="objective-c"
	ms.topic="article"
	ms.date="09/12/2016"
	ms.author="piyushjo" />

#Azure Mobile Engagement iOS SDK 版本資訊

##4\.0.0 (09/12/2016)

-   iOS 10 裝置上有固定的通知不會採取動作。
-   取代 XCode 7。

##3\.2.4 (06/30/2016)

-   修正技術記錄和其他記錄之間的彙總。

##3\.2.3 (06/07/2016)

-   修正當應用程式在背景時不回報傳遞回饋的錯誤。
-   將技術記錄的傳送最佳化。

##3\.2.2 (04/07/2016)

-   修正 HTTP 要求取消有時會導致損毀的錯誤。

##3\.2.1 (12/11/2015)

-   修正當新應用程式執行個體由具有深度連結的通知觸發時造成的延遲。

##3\.2.0 (10/08/2015)

-   啟用 SDK 中的 Bitcode 以便使用 **Xcode 7**。
-   已修正與應用程式內通知相關的錯誤。
-   讓應用程式內通知在發生電池電力不足與其他這類案例時更可靠。
-   移除第三方程式庫所產生的額外主控台記錄檔。

##3\.1.0 (2015/08/26)

-   搭配協力廠商程式庫修正 iOS 9 相容性錯誤。當傳送投票結果、應用程式資訊或是額外的資料時會造成當機。

##3\.0.0 (2015/06/19)

-   Mobile Engagement 使用無聲推播通知。
-   停止支援 iOS 4.X。從此版本開始，您的應用程式部署目標必須至少為 iOS 6。

##2\.2.0 (05/21/2015)

-   針對 < iOS 6 之裝置的 Mobile Engagement 裝置識別碼現在是根據在安裝時產生的 GUID。

##2\.1.0 (2015/04/24)

-   新增 Swift 相容性。
-   現在按一下通知時，動作 URL 會在應用程式開啟後立即執行。
-   在 SDK 封裝中加入缺少的標頭檔案。
-   已修正 Mobile Engagement 當機報告程式停用時的問題。

##2\.0.0 (2015/02/17)

-   Azure Mobile Engagement 的最初發行版本
-   以連接字串組態取代 appId/sdkKey 組態。
-   已移除從任意 XMPP 實體傳送與接收任意 XMPP 訊息的 API。
-   已移除在裝置之間傳送與接收訊息的 API。
-   增強安全性。
-   已移除 SmartAd 追蹤。

<!---HONumber=AcomDC_0921_2016-->