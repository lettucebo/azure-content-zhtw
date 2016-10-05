<properties
	pageTitle="如何使用 ADAL 在 Android 上啟用跨應用程式的 SSO | Microsoft Azure"
	description="如何使用 ADAL SDK 的功能來啟用跨應用程式的單一登入。"
	services="active-directory"
	documentationCenter=""
	authors="brandwe"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="android"
	ms.devlang="java"
	ms.topic="article"
	ms.date="09/16/2016"
	ms.author="brandwe"/>


# 如何使用 ADAL 在 Android 上啟用跨應用程式的 SSO


提供單一登入 (SSO)，讓使用者只需要輸入一次認證，就可以讓認證自動跨應用程式運作，這是客戶目前的期待。在小型螢幕上輸入其使用者名稱和密碼的困難，通常伴隨著其他因素 (2FA)，例如撥打電話或簡訊傳送程式碼，如果使用者必須對您的產品執行這個動作多次，會容易讓使用者不滿。

此外，如果您利用其他應用程式可能使用的身分識別平台，例如 Microsoft 帳戶或來自 Office365 的工作帳戶，客戶會預期這些認證可以跨所有應用程式使用，無論是什麼廠商。

Microsoft 身分識別平台以及我們的 Microsoft 身分識別 SDK 會為您執行所有繁重的工作，並讓您符合客戶的期待，在整個裝置上使用本身應用程式中的 SSO，或利用我們的訊息代理程式功能和 Authenticator 應用程式。

本逐步解說會告訴您如何在應用程式中設定我們的 SDK 以提供這個優勢給客戶。

本逐步解說適用於：

* Azure Active Directory
* Azure Active Directory B2C
* Azure Active Directory B2B
* Azure Active Directory 條件式存取

請注意，下列文件假設您已了解如何[在 Azure Active directory 的舊版入口網站中佈建應用程式](active-directory-how-to-integrate.md)，並且已整合應用程式和 [Microsoft Identity Android SDK](https://github.com/AzureAD/azure-activedirectory-library-for-android)。

## Microsoft 身分識別平台中的 SSO 概念

### Microsoft 身分識別代理人

Microsoft 提供應用程式給每個行動平台，可跨不同廠商的應用程式橋接認證，並且允許需要單一安全地方來驗證認證的特殊增強功能。我們稱它們為**訊息代理程式**。在 iOS 和 Android 上，會透過可下載的應用程式提供訊息代理程式，這些應用程式由客戶獨立安裝，或由為員工管理部分或全部裝置的公司推送至裝置。這些訊息代理程式支援只管理某些應用程式或整個裝置的安全性，取決於 IT 系統管理員的需求。在 Windows 中，內建於作業系統的帳戶選擇器會提供此功能，在技術上稱為 Web 驗證訊息代理程式。

若要了解我們如何使用這些訊息代理程式，以及客戶可能如何在其 Microsoft 身分識別平台的登入流程中看見它們，請參閱詳細資訊。

### 在行動裝置上登入的模式

如需在裝置上存取認證，請遵循 Microsoft 身分識別平台的兩個基本模式︰

* 非訊息代理程式協助登入
* 訊息代理程式協助登入

#### 非訊息代理程式協助登入

非訊息代理程式協助登入是在應用程式內發生的登入體驗，並且使用該應用程式之裝置上的本機儲存體。儲存體可能會跨應用程式共用，但認證會緊密繫結至使用該認證的應用程式或應用程式套件。這是您在許多行動應用程式中最可能經歷的體驗，您會在應用程式本身內部輸入使用者名稱和密碼。

這些登入具備下列優點︰

-  使用者體驗完全存在於應用程式內部。
-  認證可以跨由相同憑證登入的應用程式共用，提供單一登入體驗給您的應用程式套件。
-  在登入前後，會提供登入體驗的控制項給應用程式。

這些登入具備下列缺點︰

- 使用者無法跨所有使用 Microsoft 身分識別的應用程式體驗單一登入，只會跨應用程式擁有且已設定的 Microsoft 身分識別。
- 您的應用程式無法搭配更進階的商務功能使用，例如條件式存取，或使用產品的 InTune 套件。
- 您的應用程式無法支援商務使用者之以憑證為基礎的驗證。

以下說明 Microsoft Identity SDK 如何使用應用程式的共用儲存體來啟用 SSO：

```
+------------+ +------------+  +-------------+
|            | |            |  |             |
|   App 1    | |   App 2    |  |   App 3     |
|            | |            |  |             |
|            | |            |  |             |
+------------+ +------------+  +-------------+
| Azure SDK  | | Azure SDK  |  | Azure SDK   |
+------------+-+------------+--+-------------+
|                                            |
|            App Shared Storage              |
+--------------------------------------------+
```

#### 訊息代理程式協助登入

訊息代理程式協助登入在訊息代理程式應用程式中發生的登入體驗，並且使用訊息代理程式的儲存體和安全性，跨利用 Microsoft 身分識別平台之裝置上的所有應用程式共用認證。這表示您的應用程式將會依賴訊息代理程式才能讓使用者登入。在 iOS 和 Android 上，會透過可下載的應用程式提供訊息代理程式，這些應用程式由客戶獨立安裝，或由為使用者管理裝置的公司推送至裝置。此類型應用程式的範例為 iOS 上的 Azure Authenticator 應用程式。在 Windows 中，內建於作業系統的帳戶選擇器會提供此功能，在技術上稱為 Web 驗證訊息代理程式。體驗會依平台而有所不同，如果未正確管理，有時可能會干擾使用者。如果您已安裝 Facebook 應用程式，並在另一個應用程式中使用 Facebook 登入功能，您可能最熟悉這種模式。Microsoft 身分識別平台會利用相同的模式。

對於 iOS 而言，這會前往「轉換」動畫，您的應用程式已在其中傳送到背景工作，而 Azure Authenticator 應用程式則來到前景讓使用者選取他們想要登入的帳戶。

對於 Android 和 Windows 而言，帳戶選擇器會顯在比較不干擾使用者的應用程式頂端。

#### 如何叫用訊息代理程式

如果在裝置上安裝相容的訊息代理程式，例如 Azure Authenticator 應用程式，當使用者指出他們想要使用 Microsoft 身分識別平台的任何帳戶登入時，Microsoft 身分識別 SDK 會自動為您叫用訊息代理程式。這可以是個人的 Microsoft 帳戶、工作或學校帳戶，或您提供的帳戶，以及在 Azure 中使用 B2C 和 B2B 產品的主機。藉由使用非常安全的演算法和加密，我們確定會以安全的方式要求這些認證並將其傳送回您的應用程式。這些機制的確切技術詳細資料並為發佈，但已透過 Apple 和 Google 的共同作業開發。

**如果 Microsoft Identity SDK 呼叫訊息代理程式或使用非訊息代理程式協助流程，開發人員就有這個選項。** 不過，如果開發人員選擇不使用訊息代理程式協助流程，他們會失去利用 SSO 認證的優點，使用者可能已在裝置上新增這些認證，並防止 Microsoft 提供給客戶的商務功能使用其應用程式，例如條件式存取、Intune 管理功能和以憑證為基礎的驗證。

這些登入具備下列優點︰

-  無論廠商為何，使用者都可以跨所有應用程式體驗 SSO。
-  您的應用程式可利用更進階的商務功能，例如條件式存取，或使用產品的 InTune 套件。
-  您的應用程式可支援商務使用者之以憑證為基礎的驗證。
- 由訊息代理程式利用額外的安全性演算法和加密來驗證應用程式和使用者身分識別，可享有更多安全的登入體驗。

這些登入具備下列缺點︰

- 在 iOS 中，使用者可在選擇認證時轉換出您的應用程式體驗。
- 喪失在應用程式內為客戶管理登入體驗的功能。



以下說明 Microsoft Identity SDK 如何使用訊息代理程式應用程式來啟用 SSO：

```
+------------+ +------------+   +-------------+
|            | |            |   |             |
|   App 1    | |   App 2    |   |   Someone   |
|            | |            |   |    Else's   |
|            | |            |   |     App     |
+------------+ +------------+   +-------------+
| Azure SDK  | | Azure SDK  |   | Azure SDK   |
+-----+------+-+-----+------+-  +-------+-----+
      |              |                  |
      |       +------v------+           |
      |       |             |           |
      |       | Microsoft   |           |
      +-------> Broker      |^----------+
              | Application
              |             |
              +-------------+
              |             |
              |   Broker    |
              |   Storage   |
              |             |
              +-------------+

```
              
有了這個背景資訊，您應該能夠使用 Microsoft 身分識別平台和 SDK 在應用程式內進一步了解和實作 SSO。


## 使用 ADAL 啟用跨應用程式的 SSO

我們將在此使用 ADAL iOS SDK 執行以下動作︰

- 開啟應用程式套件的非訊息代理程式協助 SSO
- 開啟訊息代理程式協助 SSO 的支援


### 開啟非訊息代理程式協助 SSO 的 SSO

對於跨應用程式的非訊息代理程式協助 SSO，Microsoft Identity SDK 會為您管理 SSO 的大部分複雜內容。這包括在快取中尋找適當的使用者，以及維護登入使用者的清單以便您查詢。

若要跨您擁有的應用程式啟用 SSO，您需要執行下列動作︰

1. 請確定您所有的應用程式使用相同的用戶端識別碼或應用程式識別碼。
* 請確定您所有的應用程式具有相同的 SharedUserID 集合。
* 請確定您所有的應用程式共用來自 Google Play 商店的相同簽署憑證，以便您可以共用儲存體。

#### 步驟 1：將相同的用戶端識別碼 / 應用程式識別碼使用於應用程式套件中的所有應用程式

為了讓 Microsoft 身分識別平台知道它可以跨應用程式共用權杖，您的每個應用程式必須共用相同的用戶端識別碼或應用程式識別碼。這是您在入口網站中註冊第一個應用程式時提供給您的唯一識別碼。

如果 Microsoft 識別服務使用相同的應用程式識別碼，您可能想知道如何識別它的不同應用程式。答案是使用**重新導向 URI**。每個應用程式可以在上架的入口網站中註冊多個重新導向 URI。組件中的每個應用程式將會有不同的重新導向 URI。其外觀的範例如下：

App1 重新導向 URI：`msauth://com.example.userapp/IcB5PxIyvbLkbFVtBI%2FitkW%2Fejk%3D`

App2 重新導向 URI：`msauth://com.example.userapp1/KmB7PxIytyLkbGHuI%2UitkW%2Fejk%4E`

App3 重新導向 URI：`msauth://com.example.userapp2/Pt85PxIyvbLkbKUtBI%2SitkW%2Fejk%9F`

....

它們在相同的用戶端識別碼 / 應用程式識別碼之下是巢狀的，並且可根據您在 SDK 組態中傳回給我們的重新導向 URI 查看。

```
+-------------------+
|                   |
|  Client ID        |
+---------+---------+
          |
          |           +-----------------------------------+
          |           |  App 1 Redirect URI               |
          +----------^+                                   |
          |           +-----------------------------------+
          |
          |           +-----------------------------------+
          +----------^+  App 2 Redirect URI               |
          |           |                                   |
          |           +-----------------------------------+
          |
          +----------^+-----------------------------------+
                      |  App 3 Redirect URI               |
                      |                                   |
                      +-----------------------------------+

```


請注意，以下將說明這些重新導向 URI 的格式。您可以使用任何重新導向 URI，除非您想要支援訊息代理程式，在此情況下，它們的外觀必須如上所示


#### 步驟 2︰在 Android 中設定共用儲存體

設定 `SharedUserID` 已超出本文件的範圍，但可藉由閱讀[資訊清單](http://developer.android.com/guide/topics/manifest/manifest-element.html)上的 Google Android 文件來學習。您必須決定 sharedUserID 的名稱，並且跨所有的應用程式使用該名稱。

一旦所有的應用程式中都有 `SharedUserID`，您即可立即使用 SSO。

> [AZURE.WARNING] 
當您跨應用程式共用儲存體時，任何應用程式都可以刪除使用者，更糟的是可以跨應用程式刪除所有權杖。如果您的應用程式依賴這些權杖來執行背景工作，這樣會造成可怕的災難。共用儲存體表示您在整個 Microsoft Identity SDK 必須非常小心進行任何移除作業。

就這麼簡單！ Microsoft Identity SDK 現在會跨所有應用程式共用認證。使用者清單也會跨應用程式執行個體共用。

### 開啟訊息代理程式協助 SSO 的 SSO

應用程式使用任何已安裝在裝置上之訊息代理程式的功能**預設為關閉**。若要搭配使用應用程式與訊息代理程式，您必須執行一些額外的設定，並將一些程式碼新增至您的應用程式。

要遵循的步驟如下：

1. 在應用程式程式碼呼叫 MS SDK 時啟用訊息代理程式模式
2. 建立新的重新導向 URI，並將其提供給應用程式與應用程式註冊
3. 在 Android 資訊清單中設定正確的權限


#### 步驟 1︰在應用程式中啟用訊息代理程式模式
當您建立「設定」或驗證執行個體的初始設定時，已開啟應用程式使用訊息代理程式的功能。您可以設定程式碼中的 ApplicationSettings 類型就可以辦到︰

```
AuthenticationSettings.Instance.setUseBroker(true);
```


#### 步驟 2︰利用 URL 配置建立新的重新導向 URI

為了確保我們永遠傳回認證權杖給正確的應用程式，我們必須確定 Android 作業系統可以確認我們回呼應用程式的方式。Android 作業系統會使用 Google Play 商店中的憑證雜湊。這樣就不會讓惡意應用程式假冒。因此，我們利用這個方法搭配訊息代理程式應用程式的 URI，以確保將權杖傳回給正確的應用程式。我們需要您在應用程式中建立此唯一重新導向 URI，並在我們的開發人員入口網站中設定為重新導向 URI。

您的重新導向 URI 必須是適當的格式︰

`msauth://packagename/Base64UrlencodedSignature`

例如：*msauth://com.example.userapp/IcB5PxIyvbLkbFVtBI%2FitkW%2Fejk%3D*

此重新導向 URI 必須在使用 [Azure 傳統入口網站](https://manage.windowsazure.com/)的應用程式註冊中指定。如需 Azure AD 應用程式註冊的詳細資訊，請參閱[與 Azure Active Directory 整合](active-directory-how-to-integrate.md)。


#### 步驟 3：在您的應用程式中設定正確的權限

我們的 Android 訊息代理程式應用程式會使用 Android 作業系統的 Accounts Manager 功能來管理所有應用程式的認證。若要在 Android 中使用此訊息代理程式，您的應用程式資訊清單必須有使用 AccountManager 帳戶的權限。在[此處適用於帳戶管理員的 Google 文件](http://developer.android.com/reference/android/accounts/AccountManager.html)中有詳細的討論

特別是以下這些權限︰

```
GET_ACCOUNTS
USE_CREDENTIALS
MANAGE_ACCOUNTS
```

### 您已設定 SSO！

現在 Microsoft Identity SDK 會自動跨應用程式共用認證，並在訊息代理程式出現在其裝置上時叫用它。

<!---HONumber=AcomDC_0921_2016-->