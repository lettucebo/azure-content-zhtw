<properties
	pageTitle="Azure Mobile Engagement iOS SDK 升級程序 | Microsoft Azure"
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

#升級程序

如果您已經整合舊版 Engagement 到您的應用程式，在升級 SDK 時您必須考慮以下幾點。

針對每個新版 SDK，您必須先取代 (在 xcode 中移除並重新匯入) EngagementSDK 與 EngagementReach 資料夾。

##從 3.0.0 到 4.0.0

### XCode 8
從 SDK 4.0.0 版開始就必須要有 XCode 8。

> [AZURE.NOTE] 如果您實際上是仰賴 XCode 7，則可以使用 [iOS Engagement SDK v3.2.4](https://aka.ms/r6oouh)。這個舊版本的觸達模組在 iOS 10 裝置上執行時有已知錯誤︰系統通知不會採取動作。若要修正此錯誤，您必須在應用程式委派中實作已被取代的 API `application:didReceiveRemoteNotification:`，方式如下︰

	- (void)application:(UIApplication*)application
	didReceiveRemoteNotification:(NSDictionary*)userInfo
	{
	    [[EngagementAgent shared] applicationDidReceiveRemoteNotification:userInfo fetchCompletionHandler:nil];
	}

> [AZURE.IMPORTANT] **我們不建議此因應措施**，因為此 iOS API 已被取代，此行為在任何即將推出的 (甚至次要的) iOS 版本升級中會有所變更。您應盡快改用 XCode 8。

### UserNotifications 架構
您需要在建置階段新增 `UserNotifications` 架構。

在專案總管中，開啟專案窗格並選取正確的目標。然後，開啟 [建置階段] 索引標籤，並在 [連結二進位檔與程式庫] 功能表中新增架構 `UserNotifications.framework``Optional`

### 應用程式推播功能
XCode 8 可能會重設您的應用程式推播功能，請在您選取的目標的 `capability` 索引標籤中再檢查一次。

### 如果您已有自己的 UNUserNotificationCenterDelegate 實作

SDK 也有它自己的 UNUserNotificationCenterDelegate 通訊協定實作。SDK 會用它來監視在 iOS 10 或更新版本上執行的裝置中的 Engagement 通知生命週期。如果 SDK 偵測到您的委派則不會使用它自己的實作，因為每個應用程式只可以有一個 UNUserNotificationCenter 委派。這表示您必須將 Engagement 邏輯加入到您自己的委派。

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

##從 2.0.0 到 3.0.0
停止支援 iOS 4.X。從此版本開始，您的應用程式部署目標必須至少為 iOS 6。

如果您在應用程式中使用 Reach，必須將`remote-notification` 值新增至 Info.plist 檔案中的 `UIBackgroundModes` 陣列，以接收遠端通知。

在您的應用程式委派中，方法 `application:didReceiveRemoteNotification:` 需由 `application:didReceiveRemoteNotification:fetchCompletionHandler:` 取代。

"AEPushDelegate.h" 是已被取代的介面，且您必須移除所有參考。這包括從您的應用程式委派移除 `[[EngagementAgent shared] setPushDelegate:self]` 以及委派方法：

	-(void)willRetrieveLaunchMessage;
	-(void)didFailToRetrieveLaunchMessage;
	-(void)didReceiveLaunchMessage:(AEPushMessage*)launchMessage;

##從 1.16.0 到 2.0.0
以下說明如何將 SDK 整合從 Capptain SAS 提供的 Capptain 服務，移轉到由 Azure Mobile Engagement 提供的應用程式內。如果您是從較早版本移轉，請參閱 Capptain 網站，先移轉到 1.16 後再套用以下程序。

>[AZURE.IMPORTANT] Capptain 和 Mobile Engagement 是不同的服務，而以下程序只適用於移轉用戶端應用程式。移轉應用程式中的 SDK「不會」將您的資料從 Capptain 伺服器移轉到 Mobile Engagement 伺服器

### 代理程式

`registerApp:` 方法已被新方法 `init:` 取代。您的應用程式委派必須隨之更新，並使用連接字串：

			- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
			{
			  [...]
			  [EngagementAgent init:@"YOUR_CONNECTION_STRING"];
			  [...]
			}

SmartAd 追蹤已從 SDK 移除，因此您必須移除 `AETrackModule` 類別的所有執行個體

### 類別名稱變更

執行品牌再造時，有幾個類別/檔案名稱需要變更。

所有首碼為 "CP" 的類別，都使用 "AE" 首碼重新命名。

範例：

-   `CPModule.h` 已重新命名為 `AEModule.h`。

所有首碼為 "Capptain" 的類別，都已使用 "Engagement" 首碼重新命名。

範例：

-   `CapptainAgent` 類別已重新命名為 `EngagementAgent`。
-   `CapptainTableViewController` 類別已重新命名為 `EngagementTableViewController`。
-   `CapptainUtils` 類別已重新命名為 `EngagementUtils`。
-   `CapptainViewController` 類別已重新命名為 `EngagementViewController`。

<!---HONumber=AcomDC_0921_2016-->