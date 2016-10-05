<properties
	pageTitle="透過百度開始使用 Azure 通知中樞 | Microsoft Azure"
	description="在本教學課程中，您將了解如何透過百度使用 Azure 通知中樞，將通知推播到 Android 裝置。"
	services="notification-hubs"
	documentationCenter="android"
	authors="wesmc7777"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="notification-hubs"
	ms.devlang="java"
	ms.topic="hero-article"
	ms.tgt_pltfrm="mobile-baidu"
	ms.workload="mobile"
	ms.date="08/19/2016"
	ms.author="wesmc"/>

# 透過百度開始使用通知中樞

[AZURE.INCLUDE [notification-hubs-selector-get-started](../../includes/notification-hubs-selector-get-started.md)]

##Overview

百度雲端推播是中文的雲端服務，可用於將推播通知傳送至行動裝置。由於中國存在著不同的應用程式市集及推播服務 (除了通常不會連線到 GCM (Google Cloud Messaging) 的 Android 裝置可用性外)，要將推播通知傳送至 Android 相當複雜，因此該服務在中國特別有用。

##必要條件

本教學課程需要下列各項：

+ Android SDK (我們假設您將使用 Eclipse)，您可從 <a href="http://go.microsoft.com/fwlink/?LinkId=389797">Android 網站</a>下載該套件
+ [行動服務 Android SDK]
+ [百度推播 Android SDK]

>[AZURE.NOTE] 若要完成此教學課程，您必須具備有效的 Azure 帳戶。如果您沒有帳戶，只需要幾分鐘的時間就可以建立免費試用帳戶。如需詳細資訊，請參閱 [Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A0E0E5C02&amp;returnurl=http%3A%2F%2Fazure.microsoft.com%2Fzh-TW%2Fdocumentation%2Farticles%2Fnotification-hubs-baidu-get-started%2F)。


##建立百度帳戶

若要使用百度，您必須有百度帳戶。如果您已經有帳戶，請登入[百度入口網站]，並略過下一個步驟。否則請參閱下列指示建立百度帳戶。

1. 移至[百度入口網站]，然後按一下 [登录] \(**登入**) 連結。按一下 [立即注册] 以開始帳戶註冊程序。

   	![][1]

2. 輸入必要的詳細資料—電話/電子郵件地址、密碼和驗證碼，然後按一下 [注册]。

   	![][2]

3. 您所輸入的電子郵件地址將會收到一封電子郵件，內含啟用百度帳戶的連結。

   	![][3]

4. 登入您的電子郵件帳戶，開啟百度啟用郵件，然後按一下啟用連結以啟用您的百度帳戶。

   	![][4]

啟用百度帳戶之後，請登入[百度入口網站]。

##註冊為百度開發人員

1. 登入[百度入口網站]之後，按一下 [更多>>] \(**更多**)。

  	![][5]

2. 向下捲動至 [站长与开发者服务] **(網站管理員和開發人員服務)** 區段，然後按一下 [百度开放云平台] \(**百度開放雲平台**)。

  	![][6]

3. 在下一個頁面中，按一下右上角的 [开发者服务] \(**開發人員服務**)。

  	![][7]

4. 在下一個頁面中，按一下右上角功能表中的 [注册开发者] \(**已註冊的開發人員**)。

  	![][8]

5. 輸入您的名稱、描述和可收到驗證簡訊的行動電話號碼，然後按一下 [送验证码] \(**傳送驗證碼**)。請注意，如果是國際電話號碼，您需要使用括號括住國碼。例如，如果是美國的電話號碼，就會是 **(1)1234567890**。

  	![][9]

6. 您接著應該會收到包含驗證號碼的簡訊，如下列範例所示：

  	![][10]

7. 請在 [验证码] \(**確認碼**) 中輸入訊息內的驗證號碼。

8. 最後，接受百度合約並按一下 [提交] \(**提交**) 之後，即可完成開發人員註冊。成功完成註冊，您會看到下列頁面：

  	![][11]

##建立百度雲端推播專案

建立百度雲端推播專案時，您會收到應用程式識別碼、API 金鑰和秘密金鑰。

1. 登入[百度入口網站]之後，按一下 [更多>>] \(**更多**)。

  	![][5]

2. 向下捲動至 [站长与开发者服务] \(**網站管理員和開發人員服務**) 區段，然後按一下 [百度开放云平台] \(**百度開放雲平台**)。

  	![][6]

3. 在下一個頁面中，按一下右上角的 [开发者服务] \(**開發人員服務**)。

  	![][7]

4. 在下一個頁面中，按一下**云服务** (**雲端服務**) 區段中的 [云推播] \(**雲端推播**)。

  	![][12]

5. 成為已註冊開發人員之後，您會在最上方的功能表看到 [管理控制台] \(**管理主控台**)。按一下 [开发者服务管理] \(**開發人員服務管理**)。

  	![][13]

6. 在下一個頁面中，按一下 [创建工程] \(**建立專案**)。

  	![][14]

7. 輸入應用程式名稱，然後按一下 [创建] \(**建立**)。

  	![][15]

8. 成功建立百度雲推播專案後，您會看到包含 **AppID**、**API 金鑰**與**秘密金鑰**的頁面。記下 API 金鑰和秘密金鑰，我們會在稍後使用。

  	![][16]

9. 按一下左窗格中的 [云推送] \(**雲端推播**)，設定專案的推播通知。

  	![][31]

10. 在下一個頁面中，按一下 [推送设置] \(**推播設定**) 按鈕。

	![][32]

11. 在組態頁面中，在 [应用包名] \(**應用程式封裝**) 欄位新增要用於 Android 專案中的封裝名稱，然後按一下 [保存设置] \(**儲存**)。

	![][33]

您會看到 [保存成功！] \(**儲存成功！**) 的訊息。

##設定您的通知中樞

1. 登入 [Azure 傳統入口網站]，並按一下畫面底部的 [+新增]。

2. 依序按一下 [應用程式服務]、[服務匯流排]、[通知中樞]，然後按一下 [快速建立]。

3. 提供 [通知中樞] 的名稱，選取 [地區] 和將建立此通知中樞的 [命名空間]，然後按一下 [建立新的通知中樞]。

  	![][17]

4. 按一下您要在其中建立通知中樞的命名空間，然後按一下頂端的 [通知中樞]。

  	![][18]

5. 選取您建立的通知中樞，然後按一下上方功能表的 [設定]。

  	![][19]

6. 向下捲動至 [百度通知設定] 區段，輸入先前從百度主控台為您的百度雲端推播專案取得的 API 金鑰和秘密金鑰。按一下 [儲存]。

  	![][20]

7. 按一下通知中樞上方的 [儀表板] 索引標籤，然後按一下 [檢視連接字串]。

  	![][21]

8. 記下 [存取連接資訊] 視窗中的 **DefaultListenSharedAccessSignature** 和 **DefaultFullSharedAccessSignature**。

    ![][22]

##將您的應用程式連接到通知中樞

1. 在 Eclipse ADT 中建立新的 Android 專案 ([檔案] > [新增] > [Android 應用程式專案])。

    ![][23]

2. 輸入 [應用程式名稱]，並確定 [Minimum Required SDK] 版本已設定為 [API 16: Android 4.1]。

    ![][24]

3. 按 [下一步] 並繼續遵循精靈的指示進行，直到出現 [建立活動] 視窗為止。請確定已選取 [空白活動]，最後選取 [完成] 以建立新的 Android 應用程式。

    ![][25]

4. 請確定您已正確設定 [專案建置目標]。

    ![][26]

5. 從 [Bintray 上之 Notification-Hubs-Android-SDK](https://bintray.com/microsoftazuremobile/SDK/Notification-Hubs-Android-SDK/0.4) 的 [Files] 索引標籤下載 notification-hubs-0.4.jar 檔案。新增檔案到您 Eclipse 專案的 **libs** 資料夾中，然後重新整理 *libs* 資料夾。

6. 下載並解壓縮[百度推播 Android SDK]，開啟 **libs** 資料夾，然後將 **pushservice-x.y.z** jar 檔案以及 **armeabi** 和 **mips** 資料夾複製到 Android 應用程式的 **libs** 資料夾。

7. 開啟 Android 專案的 **AndroidManifest.xml** 檔案，並新增 Baidu SDK 所需的權限。

	    <uses-permission android:name="android.permission.INTERNET" />
	    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
	    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
	    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
	    <uses-permission android:name="android.permission.WRITE_SETTINGS" />
	    <uses-permission android:name="android.permission.VIBRATE" />
	    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
	    <uses-permission android:name="android.permission.DISABLE_KEYGUARD" />
	    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
	    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
	    <uses-permission android:name="android.permission.ACCESS_DOWNLOAD_MANAGER" />
	    <uses-permission android:name="android.permission.DOWNLOAD_WITHOUT_NOTIFICATION" />

8. 在 **AndroidManifest.xml** 中，將 **android:name** 屬性新增至 **application** 元素以取代 *yourprojectname* (例如 **com.example.BaiduTest**)。確定此專案名稱符合您在百度主控台中設定的名稱。

		<application android:name="yourprojectname.DemoApplication"

9. 在應用程式元素內，於 **.MainActivity** 活動元素之後新增下列組態以取代 *yourprojectname* (例如 **com.example.BaiduTest**)：

		<receiver android:name="yourprojectname.MyPushMessageReceiver">
		    <intent-filter>
		        <action android:name="com.baidu.android.pushservice.action.MESSAGE" />
		        <action android:name="com.baidu.android.pushservice.action.RECEIVE" />
		        <action android:name="com.baidu.android.pushservice.action.notification.CLICK" />
		    </intent-filter>
		</receiver>

		<receiver android:name="com.baidu.android.pushservice.PushServiceReceiver"
		    android:process=":bdservice_v1">
		    <intent-filter>
		        <action android:name="android.intent.action.BOOT_COMPLETED" />
		        <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
				<action android:name="com.baidu.android.pushservice.action.notification.SHOW" />
		    </intent-filter>
		</receiver>

        <receiver android:name="com.baidu.android.pushservice.RegistrationReceiver"
            android:process=":bdservice_v1">
            <intent-filter>
                <action android:name="com.baidu.android.pushservice.action.METHOD" />
                <action android:name="com.baidu.android.pushservice.action.BIND_SYNC" />
            </intent-filter>
            <intent-filter>
                <action android:name="android.intent.action.PACKAGE_REMOVED"/>
                <data android:scheme="package" />
            </intent-filter>
        </receiver>

        <service
            android:name="com.baidu.android.pushservice.PushService"
            android:exported="true"
            android:process=":bdservice_v1"  >
            <intent-filter>
                <action android:name="com.baidu.android.pushservice.action.PUSH_SERVICE" />
            </intent-filter>
        </service>

9. 將名為 **ConfigurationSettings.java** 的新類別新增至專案。

    ![][28]

    ![][29]

10. 對其新增下列程式碼：

		public class ConfigurationSettings {
		        public static String API_KEY = "...";
				public static String NotificationHubName = "...";
				public static String NotificationHubConnectionString = "...";
			}

	使用您先前從百度雲端專案所擷取的金鑰來設定 **API\_KEY** 的值，使用 Azure 傳統入口網站中的通知中樞名稱來設定 **NotificationHubName**，並使用 Azure 傳統入口網站中的 DefaultListenSharedAccessSignature 來設定 **NotificationHubConnectionString**。

11. 新增名為 **DemoApplication.java** 的類別，並在其中新增下列程式碼：

		import com.baidu.frontia.FrontiaApplication;

		public class DemoApplication extends FrontiaApplication {
		    @Override
		    public void onCreate() {
		        super.onCreate();
		    }
		}

12. 新增另一個名為 **MyPushMessageReceiver.java** 的類別，並在其中新增下列程式碼。此類別會處理接收自百度推播伺服器的推播通知。

		import java.util.List;
		import android.content.Context;
		import android.os.AsyncTask;
		import android.util.Log;
		import com.baidu.frontia.api.FrontiaPushMessageReceiver;
		import com.microsoft.windowsazure.messaging.NotificationHub;

		public class MyPushMessageReceiver extends FrontiaPushMessageReceiver {
		    /** TAG to Log */
			public static NotificationHub hub = null;
			public static String mChannelId, mUserId;
		    public static final String TAG = MyPushMessageReceiver.class
		            .getSimpleName();

			@Override
		    public void onBind(Context context, int errorCode, String appid,
		            String userId, String channelId, String requestId) {
		        String responseString = "onBind errorCode=" + errorCode + " appid="
		                + appid + " userId=" + userId + " channelId=" + channelId
		                + " requestId=" + requestId;
		        Log.d(TAG, responseString);
		        mChannelId = channelId;
		        mUserId = userId;

		        try {
		       	 if (hub == null) {
		                hub = new NotificationHub(
		                		ConfigurationSettings.NotificationHubName,
		                		ConfigurationSettings.NotificationHubConnectionString,
		                		context);
		                Log.i(TAG, "Notification hub initialized");
		            }
		        } catch (Exception e) {
		           Log.e(TAG, e.getMessage());
		        }

		        registerWithNotificationHubs();
			}

		    private void registerWithNotificationHubs() {
		       new AsyncTask<Void, Void, Void>() {
		          @Override
		          protected Void doInBackground(Void... params) {
		             try {
		            	 hub.registerBaidu(mUserId, mChannelId);
		            	 Log.i(TAG, "Registered with Notification Hub - '"
		     	    			+ ConfigurationSettings.NotificationHubName + "'"
		     	    			+ " with UserId - '"
		     	    			+ mUserId + "' and Channel Id - '"
		     	    			+ mChannelId + "'");
		             } catch (Exception e) {
		            	 Log.e(TAG, e.getMessage());
		             }
		             return null;
		         }
		       }.execute(null, null, null);
		    }

		    @Override
		    public void onSetTags(Context context, int errorCode,
		            List<String> sucessTags, List<String> failTags, String requestId) {
		        String responseString = "onSetTags errorCode=" + errorCode
		                + " sucessTags=" + sucessTags + " failTags=" + failTags
		                + " requestId=" + requestId;
		        Log.d(TAG, responseString);
		    }

		    @Override
		    public void onDelTags(Context context, int errorCode,
		            List<String> sucessTags, List<String> failTags, String requestId) {
		        String responseString = "onDelTags errorCode=" + errorCode
		                + " sucessTags=" + sucessTags + " failTags=" + failTags
		                + " requestId=" + requestId;
		        Log.d(TAG, responseString);
		    }

		    @Override
		    public void onListTags(Context context, int errorCode, List<String> tags,
		            String requestId) {
		        String responseString = "onListTags errorCode=" + errorCode + " tags="
		                + tags;
		        Log.d(TAG, responseString);
		    }

		    @Override
		    public void onUnbind(Context context, int errorCode, String requestId) {
		        String responseString = "onUnbind errorCode=" + errorCode
		                + " requestId = " + requestId;
		        Log.d(TAG, responseString);
		    }

		    @Override
		    public void onNotificationClicked(Context context, String title,
		            String description, String customContentString) {
		        String notifyString = "title="" + title + "" description=""
		                + description + "" customContent=" + customContentString;
		        Log.d(TAG, notifyString);
		    }

		    @Override
		    public void onMessage(Context context, String message,
		            String customContentString) {
		        String messageString = "message="" + message + "" customContentString=" + customContentString;
		        Log.d(TAG, messageString);
		    }
		}

13. 開啟 **MainActivity.java**，並將下列程式碼新增至 **onCreate** 方法：

	        PushManager.startWork(getApplicationContext(),
	                PushConstants.LOGIN_TYPE_API_KEY, ConfigurationSettings.API_KEY);

14. 在頂端開啟下列 import 陳述式：

			import com.baidu.android.pushservice.PushConstants;
			import com.baidu.android.pushservice.PushManager;

##將通知傳送至您的應用程式


在 [Azure 入口網站](https://portal.azure.com/)中使用通知中樞上的 [測試傳送] 案例 (如下列螢幕畫面所示) 來傳送通知，即可在應用程式中快速測試通知的接收。

![](./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-test-send-wns.png)

推播通知通常會以後端服務傳送，例如行動服務或使用相容程式庫的 ASP.NET。如果程式庫不適用於您的後端，也可以直接使用 REST API 來傳送通知訊息。

在本教學課程中，為了簡單起見，我們只會在主控台應用程式 (而非後端服務) 中使用適用於通知中樞的 .NET SDK 傳送通知，示範如何測試您的用戶端應用程式。我們建議以[使用通知中樞將通知推播給使用者](notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md)教學課程做為下一個步驟，以便從 ASP.NET 後端傳送通知。不過，下列方法可用來傳送通知：

* **REST 介面**：您可以在使用 [REST 介面](http://msdn.microsoft.com/library/windowsazure/dn223264.aspx)的任何後端平台上支援通知。

* **Microsoft Azure 通知中樞 .NET SDK**︰在適用於 Visual Studio 的 NuGet 封裝管理員中，執行 [Install-Package Microsoft.Azure.NotificationHubs](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/)。

* **Node.js**：[如何從 Node.js 使用通知中樞](notification-hubs-nodejs-push-notification-tutorial.md)。

* **Azure 行動服務**：如需如何從已與通知中樞整合的 Azure 行動服務後端傳送通知的範例，請參閱[將推播通知新增至行動服務應用程式](../mobile-services/mobile-services-javascript-backend-windows-universal-dotnet-get-started-push.md)。

* **Java/PHP**︰如需使用 REST API 傳送通知的範例，請參閱＜如何從 Java/PHP 使用通知中樞＞([Java](notification-hubs-java-push-notification-tutorial.md) | [PHP](notification-hubs-php-push-notification-tutorial.md))。

##(選擇性) 從 .NET 主控台應用程式傳送通知。

在本節中，我們會說明如何使用.NET 主控台應用程式傳送通知。

1. 建立新的 Visual C# 主控台應用程式：

	![][30]

2. 在 [封裝管理員主控台] 視窗中，將 [預設專案] 設為新的主控台應用程式專案，然後在主控台視窗中執行下列命令：

        Install-Package Microsoft.Azure.NotificationHubs

	這會使用 <a href="http://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/">Microsoft.Azure.Notification Hubs NuGet 封裝</a>加入對 Azure 通知中樞 SDK 的參考。

	![](./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-package-manager.png)

3. 開啟檔案 **Program.cs**，並新增下列 using 陳述式：

        using Microsoft.Azure.NotificationHubs;

4. 在 `Program` 類別中新增下列方法，並使用您的值取代 *DefaultFullSharedAccessSignatureSASConnectionString* 與 *NotificationHubName*。

		private static async void SendNotificationAsync()
		{
			NotificationHubClient hub = NotificationHubClient.CreateClientFromConnectionString("DefaultFullSharedAccessSignatureSASConnectionString", "NotificationHubName");
			string message = "{"title":"((Notification title))","description":"Hello from Azure"}";
			var result = await hub.SendBaiduNativeNotificationAsync(message);
		}

5. 在您的 **Main** 方法中新增下列程式碼行：

         SendNotificationAsync();
		 Console.ReadLine();

##測試應用程式

若要使用真正的手機來測試應用程式，您只需要使用 USB 纜線將其連接到電腦即可。這樣會將應用程式載入連接的手機。

若要使用模擬器測試此應用程式，請在 Eclipse 上方的工具列按一下 [執行]，然後選取您的應用程式。這樣會啟動模擬器，然後載入並執行該應用程式。

應用程式會從百度推播通知服務中擷取 'userId' 和 'channelId'，並向通知中樞註冊。

若要傳送測試通知，您可以使用 Azure 傳統入口網站的 [偵錯] 索引標籤。如果建置 Visual Studio 的 .NET 主控台應用程式時，僅須在 Visual Studio 中按 F5 鍵，即可執行該應用程式。應用程式會傳送通知，而該通知會顯示在裝置或模擬器上方的通知區域中。


<!-- Images. -->
[1]: ./media/notification-hubs-baidu-get-started/BaiduRegistration.png
[2]: ./media/notification-hubs-baidu-get-started/BaiduRegistrationInput.png
[3]: ./media/notification-hubs-baidu-get-started/BaiduConfirmation.png
[4]: ./media/notification-hubs-baidu-get-started/BaiduActivationEmail.png
[5]: ./media/notification-hubs-baidu-get-started/BaiduRegistrationMore.png
[6]: ./media/notification-hubs-baidu-get-started/BaiduOpenCloudPlatform.png
[7]: ./media/notification-hubs-baidu-get-started/BaiduDeveloperServices.png
[8]: ./media/notification-hubs-baidu-get-started/BaiduDeveloperRegistration.png
[9]: ./media/notification-hubs-baidu-get-started/BaiduDevRegistrationInput.png
[10]: ./media/notification-hubs-baidu-get-started/BaiduDevRegistrationConfirmation.png
[11]: ./media/notification-hubs-baidu-get-started/BaiduDevConfirmationFinal.png
[12]: ./media/notification-hubs-baidu-get-started/BaiduCloudPush.png
[13]: ./media/notification-hubs-baidu-get-started/BaiduDevSvcMgmt.png
[14]: ./media/notification-hubs-baidu-get-started/BaiduCreateProject.png
[15]: ./media/notification-hubs-baidu-get-started/BaiduCreateProjectInput.png
[16]: ./media/notification-hubs-baidu-get-started/BaiduProjectKeys.png
[17]: ./media/notification-hubs-baidu-get-started/AzureNHCreation.png
[18]: ./media/notification-hubs-baidu-get-started/NotificationHubs.png
[19]: ./media/notification-hubs-baidu-get-started/NotificationHubsConfigure.png
[20]: ./media/notification-hubs-baidu-get-started/NotificationHubBaiduConfigure.png
[21]: ./media/notification-hubs-baidu-get-started/NotificationHubsConnectionStringView.png
[22]: ./media/notification-hubs-baidu-get-started/NotificationHubsConnectionString.png
[23]: ./media/notification-hubs-baidu-get-started/EclipseNewProject.png
[24]: ./media/notification-hubs-baidu-get-started/EclipseProjectCreation.png
[25]: ./media/notification-hubs-baidu-get-started/EclipseBlankActivity.png
[26]: ./media/notification-hubs-baidu-get-started/EclipseProjectBuildProperty.png
[27]: ./media/notification-hubs-baidu-get-started/EclipseBaiduReferences.png
[28]: ./media/notification-hubs-baidu-get-started/EclipseNewClass.png
[29]: ./media/notification-hubs-baidu-get-started/EclipseConfigSettingsClass.png
[30]: ./media/notification-hubs-baidu-get-started/ConsoleProject.png
[31]: ./media/notification-hubs-baidu-get-started/BaiduPushConfig1.png
[32]: ./media/notification-hubs-baidu-get-started/BaiduPushConfig2.png
[33]: ./media/notification-hubs-baidu-get-started/BaiduPushConfig3.png

<!-- URLs. -->
[行動服務 Android SDK]: https://go.microsoft.com/fwLink/?LinkID=280126&clcid=0x409
[百度推播 Android SDK]: http://developer.baidu.com/wiki/index.php?title=docs/cplat/push/sdk/clientsdk
[Azure 傳統入口網站]: https://manage.windowsazure.com/
[百度入口網站]: http://www.baidu.com/

<!---HONumber=AcomDC_0824_2016-->