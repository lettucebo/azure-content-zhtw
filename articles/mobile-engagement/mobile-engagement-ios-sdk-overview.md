<properties
	pageTitle="Azure Mobile Engagement iOS SDK 概觀 | Microsoft Azure"
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
	ms.date="09/14/2016"
	ms.author="piyushjo" />

#Azure Mobile Engagement iOS SDK

從這裡開始取得有關如何在 iOS 應用程式中整合 Azure Mobile Engagement 的所有詳細資料。如果您想要先試用一下，請務必先完成我們的 [15 分鐘教學課程](mobile-engagement-ios-get-started.md)。

按一下以查看 [SDK 內容](mobile-engagement-ios-sdk-content.md)

##整合程序
1. 從這裡開始：[如何在 iOS 應用程式中整合 Mobile Engagement](mobile-engagement-ios-integrate-engagement.md)

2. 通知：[如何在 iOS 應用程式中整合 Reach (通知)](mobile-engagement-ios-integrate-engagement-reach.md)

3. 標記計劃實作：[如何在 iOS 應用程式中使用進階的 Mobile Engagement 標記 API](mobile-engagement-ios-use-engagement-api.md)


##版本資訊

### 4\.0.0 (09/12/2016)

-   iOS 10 裝置上有固定的通知不會採取動作。
-   取代 XCode 7。

如需較早版本，請參閱[完整版本資訊](mobile-engagement-ios-release-notes.md)

##升級程序

如果您已經整合舊版 Engagement 到您的應用程式，在升級 SDK 時您必須考慮以下幾點。

如果您錯過數個版本的 SDK，可能必須遵循幾個程序步驟，請參閱完整的[升級程序](mobile-engagement-ios-upgrade-procedure.md)。

針對每個新版 SDK，您必須先取代 (在 xcode 中移除並重新匯入) EngagementSDK 與 EngagementReach 資料夾。

###從 3.0.0 到 4.0.0

### XCode 8
從 SDK 4.0.0 版開始就必須要有 XCode 8。

> [AZURE.NOTE] 如果您實際上是仰賴 XCode 7，則可以使用 [iOS Engagement SDK v3.2.4](https://aka.ms/r6oouh)。這個舊版本的觸達模組在 iOS 10 裝置上執行時有已知錯誤︰系統通知不會採取動作。若要修正此錯誤，您必須在應用程式委派中實作已被取代的 API `application:didReceiveRemoteNotification:`，方式如下︰

	- (void)application:(UIApplication*)application
	didReceiveRemoteNotification:(NSDictionary*)userInfo
	{
	    [[EngagementAgent shared] applicationDidReceiveRemoteNotification:userInfo fetchCompletionHandler:nil];
	}

> [AZURE.IMPORTANT] **我們不建議此因應措施**，因為此 iOS API 已被取代，此行為在任何即將推出的 (甚至次要的) iOS 版本升級中會有所變更。您應盡快改用 XCode 8。

#### UserNotifications 架構
您需要在建置階段新增 `UserNotifications` 架構。

在專案總管中，開啟專案窗格並選取正確的目標。然後，開啟 [建置階段] 索引標籤，並在 [連結二進位檔與程式庫] 功能表中新增架構 `UserNotifications.framework``Optional`

#### 應用程式推播功能
XCode 8 可能會重設您的應用程式推播功能，請在您選取的目標的 `capability` 索引標籤中再檢查一次。

#### 如果您已有自己的 UNUserNotificationCenterDelegate 實作

SDK 有它自己的 UNUserNotificationCenterDelegate 通訊協定實作。SDK 會用它來監視在 iOS 10 或更新版本上執行的裝置中的 Engagement 通知生命週期。如果 SDK 偵測到您的委派則不會使用它自己的實作，因為每個應用程式只可以有一個 UNUserNotificationCenter 委派。這表示您必須將 Engagement 邏輯加入到您自己的委派。

有兩種方式可以達到這個目的。

藉由直接將委派呼叫轉送給 SDK：

	#import <UIKit/UIKit.h>
	#import "EngagementAgent.h"
	#import <UserNotifications/UserNotifications.h>


	@interface MyAppDelegate : NSObject <UIApplicationDelegate, UNUserNotificationCenterDelegate>
	@end

	@implementation MyAppDelegate

	- (void)userNotificationCenter:(UNUserNotificationCenter *)center willPresentNotification:(UNNotification *)notification withCompletionHandler:(void (^)(UNNotificationPresentationOptions options))completionHandler
	{
	  // Your own logic.

	  [[EngagementAgent shared] userNotificationCenterWillPresentNotification:notification withCompletionHandler:completionHandler]
	}

	- (void)userNotificationCenter:(UNUserNotificationCenter *)center didReceiveNotificationResponse:(UNNotificationResponse *)response withCompletionHandler:(void(^)())completionHandler
	{
	  // Your own logic.

	  [[EngagementAgent shared] userNotificationCenterDidReceiveNotificationResponse:response withCompletionHandler:completionHandler]
	}
	@end

或藉由繼承自 `AEUserNotificationHandler` 類別

	#import "AEUserNotificationHandler.h"
	#import "EngagementAgent.h"

	@interface CustomUserNotificationHandler :AEUserNotificationHandler
	@end

	@implementation CustomUserNotificationHandler

	- (void)userNotificationCenter:(UNUserNotificationCenter *)center willPresentNotification:(UNNotification *)notification withCompletionHandler:(void (^)(UNNotificationPresentationOptions options))completionHandler
	{
	  // Your own logic.

	  [super userNotificationCenter:center willPresentNotification:notification withCompletionHandler:completionHandler];
	}

	- (void)userNotificationCenter:(UNUserNotificationCenter *)center didReceiveNotificationResponse: UNNotificationResponse *)response withCompletionHandler:(void(^)())completionHandler
	{
	  // Your own logic.

	  [super userNotificationCenter:center didReceiveNotificationResponse:response withCompletionHandler:completionHandler];
	}

	@end

> [AZURE.NOTE] 您可以藉由將通知的 `userInfo` 字典傳遞給代理程式的 `isEngagementPushPayload:` 類別方法，來決定通知是否來自 Engagement。

<!---HONumber=AcomDC_0921_2016-->