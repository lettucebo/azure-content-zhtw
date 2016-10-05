<properties
	pageTitle="Azure Active Directory v2.0 Android 應用程式 | Microsoft Azure"
	description="如何建置可使用個人 Microsoft 帳戶及公司或學校帳戶登入使用者，並使用協力廠商程式庫呼叫圖形 API 的 Android 應用程式。"
	services="active-directory"
	documentationCenter=""
	authors="brandwe"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/16/2016"
	ms.author="brandwe"/>

#  使用 v2.0 端點透過圖形 API 將登入新增至使用協力廠商程式庫的 Android 應用程式

Microsoft 身分識別平台會使用開放式標準，例如 OAuth2 和 OpenID Connect。開發人員可以使用任何想要的程式庫，來與我們的服務整合。為了協助開發人員使用我們的平台搭配其他程式庫，我們撰寫了數篇逐步解說，示範如何設定協力廠商程式庫以連接到 Microsoft 身分識別平台。大部分實作 [RFC6749 OAuth2 規格](https://tools.ietf.org/html/rfc6749)的程式庫都能連接到 Microsoft 身分識別平台。

使用此篇逐步解說建立的應用程式，使用者可以使用圖形 API 來登入其組織，然後在組織中搜尋自己。

如果您是 OAuth2 或 OpenID Connect 新手，此範例組態可能不太適合您。建議您閱讀 [2\.0 通訊協定 - OAuth 2.0 授權碼流程](active-directory-v2-protocols-oauth-code.md)背景。

> [AZURE.NOTE] 我們的平台中有些功能沒有採用 OAuth2 或 OpenID Connect 標準的運算式 (例如條件式存取和 Intune 原則管理)，所以會要求您使用開放原始碼 Microsoft Azure 身分識別程式庫。

v2.0 端點並未支援所有的 Azure Active Directory 案例和功能。

> [AZURE.NOTE] 若要判斷是否應該使用 v2.0 端點，請閱讀相關的 [v2.0 限制](active-directory-v2-limitations.md)。


## 從 GitHub 下載程式碼
本教學課程的程式碼保留在 [GitHub](git@github.com:Azure-Samples/active-directory-android-native-oidcandroidlib-v2.git)。若要遵循執行，您可以[用 .zip 格式下載應用程式的基本架構](git@github.com:Azure-Samples/active-directory-android-native-oidcandroidlib-v2.git/archive/skeleton.zip)，或複製基本架構：

```
git clone --branch skeleton git@github.com:Azure-Samples/active-directory-android-native-oidcandroidlib-v2.git
```

您也可以下載範例，並立即開始著手使用︰

```
git@github.com:Azure-Samples/active-directory-android-native-oidcandroidlib-v2.git
```

## 註冊應用程式
在[應用程式註冊入口網站](https://apps.dev.microsoft.com)建立新的應用程式，或遵循[如何註冊應用程式與 v2.0 端點](active-directory-v2-app-registration.md)的詳細步驟進行。請確定：

- 複製所指派給您的「應用程式識別碼」，因為您很快就會用到。
- 為您的應用程式新增**行動**平台。

> 注意︰應用程式註冊入口網站會提供 [重新導向 URI] 值。不過，在此範例中，您必須使用 `https://login.microsoftonline.com/common/oauth2/nativeclient` 的預設值。


## 下載 NXOAuth2 協力廠商程式庫並建立工作區

在此逐步解說中，您將使用 GitHub 中的 OIDCAndroidLib，這個以 Google 的 OpenID Connect 程式碼為基礎的 OAuth2 程式庫。它會實作原生應用程式設定檔，並支援使用者的授權端點。您需要上述各項，才能與 Microsoft 身分識別平台整合。

將 OIDCAndroidLib 儲存機制複製到您的電腦。

```
git@github.com:kalemontes/OIDCAndroidLib.git
```

![androidStudio](media/active-directory-android-native-oidcandroidlib-v2/emotes-url.png)

## 設定您的 Android Studio 環境

1. 建立新的 Android Studio 專案，並接受精靈中的預設值。

	![在 Android Studio 中建立新專案](media/active-directory-android-native-oidcandroidlib-v2/SetUpSample1.PNG)

	![目標 Android 裝置](media/active-directory-android-native-oidcandroidlib-v2/SetUpSample2.PNG)

	![將活動加入行動裝置](media/active-directory-android-native-oidcandroidlib-v2/SetUpSample3.PNG)

2. 若要設定您的專案模組，請將複製的儲存機制移至專案位置。您也可以先建立專案，然後將專案直接複製到專案位置。

	![專案模組](media/active-directory-android-native-oidcandroidlib-v2/SetUpSample4_1.PNG)

3. 使用內容功能表，或使用 Ctrl+Alt+Maj+S 快速鍵，來開啟專案模組設定。

	![專案模組設定](media/active-directory-android-native-oidcandroidlib-v2/SetUpSample4.PNG)

4. 因為您只想要專案容器設定，所以請移除預設的應用程式模組。

	![預設應用程式模組](media/active-directory-android-native-oidcandroidlib-v2/SetUpSample5.PNG)

5. 從複製的儲存機制將模組匯入目前的專案。

	![匯入 gradle 專案](media/active-directory-android-native-oidcandroidlib-v2/SetUpSample6.PNG) ![建立新的模組頁面](media/active-directory-android-native-oidcandroidlib-v2/SetUpSample7.PNG)

6. 對 `oidlib-sample` 模組重複執行這些步驟。

7. 檢查 `oidlib-sample` 模組上的 oidclib 相依性。

	![oidlib-sample 模組的 oidclib 相依性](media/active-directory-android-native-oidcandroidlib-v2/SetUpSample8.PNG)

8. 按一下 [確定] 並等候 gradle 同步處理。

	您的 settings.gradle 應如下所示：

	![settings.gradle 的螢幕擷取畫面](media/active-directory-android-native-oidcandroidlib-v2/SetUpSample8_1.PNG)

9. 建置範例應用程式，確定此範例可正確運作。

	您尚未能夠將此範例使用於 Azure Active Directory。我們需要先設定一些端點。這是為了確保在我們開始自訂範例應用程式前，Android Studio 沒有問題。

10. 在 Android Studio 中建置 `oidlib-sample` 並當作目標執行。

	![oidlib-sample 建置進度](media/active-directory-android-native-oidcandroidlib-v2/SetUpSample9.png)

11. 刪除從專案中移除模組時所留下的 `app ` 目錄，因為 Android Studio 會基於安全考量而不刪除它。

	![包含應用程式目錄的檔案結構](media/active-directory-android-native-oidcandroidlib-v2/SetUpSample12.PNG)

12. 開啟 [編輯設定] 功能表，移除從專案中移除模組時所留下的執行組態。

	![編輯設定功能表](media/active-directory-android-native-oidcandroidlib-v2/SetUpSample10.PNG) ![執行應用程式設定](media/active-directory-android-native-oidcandroidlib-v2/SetUpSample11.PNG)

## 設定範例的端點

您現在已順利執行 `oidlib-sample`，讓我們編輯某些端點以便使用於 Azure Active Directory。

### 編輯 oidc\_clientconf.xml 檔案來設定您的用戶端

1. 因為您目前只使用 OAuth2 流量來取得權杖及呼叫圖形 API，所以將用戶端設定成只進行 OAuth2。後面的範例中會示範 OIDC。

	```xml
	    <bool name="oidc_oauth2only">true</bool>
	```

2. 設定您從註冊入口網站收到的用戶端識別碼。

	```xml
	    <string name="oidc_clientId">86172f9d-a1ae-4348-aafa-7b3e5d1b36f5</string>
	    <string name="oidc_clientSecret"></string>
	```

3. 如下所示來設定您的重新導向 URI。

	```xml
	    <string name="oidc_redirectUrl">https://login.microsoftonline.com/common/oauth2/nativeclient</string>
	```

4. 設定您需要的範圍，以便存取 圖形 API。

	```xml
	    <string-array name="oidc_scopes">
	        <item>openid</item>
	        <item>https://graph.microsoft.com/User.Read</item>
	        <item>offline_access</item>
	    </string-array>
	```

`oidc_scopes` 中的 `User.Read` 值可讓您讀取已登入使用者的基本設定檔。您可以在 [Microsoft Graph 權限範圍](https://graph.microsoft.io/docs/authorization/permission_scopes)，深入了解所有可用範圍。

如果您想要 OpenID Connect 中有關 `openid` 或 `offline_access` 範圍的說明，請參閱 [2\.0 通訊協定 - OAuth 2.0 授權碼流程](active-directory-v2-protocols-oauth-code.md)。

### 編輯 oidc\_endpoints.xml 檔案來設定用戶端端點

- 開啟 `oidc_endpoints.xml` 檔案並進行下列變更：

	```xml
	<!-- Stores OpenID Connect provider endpoints. -->
	<resources>
	    <string name="op_authorizationEnpoint">https://login.microsoftonline.com/common/oauth2/v2.0/authorize</string>
	    <string name="op_tokenEndpoint">https://login.microsoftonline.com/common/oauth2/v2.0/token</string>
	    <string name="op_userInfoEndpoint">https://www.example.com/oauth2/userinfo</string>
	    <string name="op_revocationEndpoint">https://www.example.com/oauth2/revoketoken</string>
	</resources>
	```

如果您使用 OAuth2 做為通訊協定，則不得變更這些端點。

> [AZURE.NOTE]
Azure Active Directory 目前不支援 `userInfoEndpoint` 和 `revocationEndpoint` 的端點。如果您保留這些預設的 example.com 值，您將會不時看到提醒，說明無法在此範例中使用這些值 :-)


## 設定圖形 API 呼叫

- 開啟 `HomeActivity.java` 檔案並進行下列變更：

	```Java
	   //TODO: set your protected resource url
	    private static final String protectedResUrl = "https://graph.microsoft.com/v1.0/me/";
	```

以下的簡單圖形 API 呼叫會傳回我們的資訊。

這些全都是您要進行的變更。執行 `oidlib-sample` 應用程式，然後按一下 [登入]。

順利通過驗證後，請選取 [要求受保護的資源] 按鈕，以測試您對圖形 API 的呼叫。

## 取得產品的安全性更新

我們鼓勵您造訪[安全性 TechCenter](https://technet.microsoft.com/security/dd252948) 並訂閱資訊安全摘要報告警示，以收到有關安全性事件的通知。

<!---HONumber=AcomDC_0921_2016-->