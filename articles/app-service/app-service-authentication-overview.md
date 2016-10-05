<properties
	pageTitle="Azure App Service 中的驗證和授權 | Microsoft Azure"
	description="Azure App Service 的驗證/授權功能的概念性參考和概觀"
	services="app-service"
	documentationCenter=""
	authors="mattchenderson"
	manager="erikre"
	editor=""/>

<tags
	ms.service="app-service"
	ms.workload="mobile"
	ms.tgt_pltfrm="na"
	ms.devlang="multiple"
	ms.topic="article"
	ms.date="08/29/2016"
	ms.author="mahender"/>

# Azure App Service 中的驗證與授權

## 什麼是 App Service 驗證 / 授權？

App Service 驗證/授權是可讓您的應用程式接受使用者登入的一項功能，而不需要您在應用程式後端變更程式碼。它提供簡單的方法來保護您的應用程式，以及使用每位使用者的資料。

App Service 使用同盟身分識別，由第三方識別提供者儲存帳戶並驗證使用者。應用程式依賴提供者的身分識別資訊，所以應用程式本身不需要儲存該資訊。App Service 支援五個現成的識別提供者：Azure Active Directory、Facebook、Google、Microsoft 帳戶及 Twitter。您的應用程式可以隨意搭配這幾種身分識別提供者，讓使用者選擇如何登入。若要擴充內建的支援，您可以整合其他識別提供者或[您自己的自訂身分識別解決方案][custom-auth]。

如果您想要立即開始，請參閱下列其中一個教學課程︰

- [將驗證新增至 iOS 應用程式][iOS] \(或 [Android]、[Windows]、[Xamarin.iOS]、[Xamarin.Android]、[Xamarin.Forms] 或 [Cordova])
- [Azure App Service 中 API Apps 的使用者驗證][apia-user]
- [開始使用 Azure App Service - 第 2 部分][web-getstarted]

## App Service 中驗證的運作方式

若要使用其中一個識別提供者進行驗證，您首先需要設定讓識別提供者了解您的應用程式。識別提供者接著會提供識別碼和密碼，讓您提供給 App Service。這樣就完成信任關係，讓 App Service 能夠驗證來自識別提供者的使用者判斷提示，例如驗證權杖。

若要使用其中一個提供者讓使用者登入，必須將使用者重新導向至該提供者用於讓使用者登入的端點。如果客戶使用網頁瀏覽器，您可以讓 App Service 自動將所有未驗證的使用者引導至讓使用者登入的端點。否則，您需要將客戶引導至 `{your App Service base URL}/.auth/login/<provider>`，其中 `<provider>` 是下列其中一個值：aad、facebook、google、microsoft 或 twitter。在本文稍後各節說明行動和 API 的案例。

透過網頁瀏覽器來與您的應用程式互動的使用者會設定 Cookie，以便在瀏覽您的應用程式時，能夠維持在已驗證的狀態。對於其他用戶端類型，例如行動，將會核發 JSON Web Token (JWT) (應該出現在 `X-ZUMO-AUTH` 標頭中) 給用戶端。Mobile Apps 用戶端 SDK 將會為您處理這項工作。或者，Azure Active Directory 身分識別權杖或存取權杖可能直接包含在 `Authorization` 標頭中，做為[持有人權杖](https://tools.ietf.org/html/rfc6750)。

App Service 會驗證您的應用程式核發來驗證使用者的任何 Cookie 或權杖。若要限制誰可以存取您的應用程式，請參閱本文稍後的[授權](#authorization)一節。

### 使用提供者 SDK 的行動驗證

在後端完成一切設定後，您就可以修改讓行動用戶端使用 App Service 進行登入。有以下兩種方法：

- 使用指定的識別提供者發佈的 SDK 來建立身分識別，然後取得 App Service 的存取權。
- 使用一行程式碼讓 Mobile Apps 用戶端 SDK 能夠將使用者登入。

>[AZURE.TIP] 大部分應用程式都應該使用提供者 SDK，讓使用者登入時有更一致的體驗、使用重新整理支援，以及享受提供者指定的其他好處。

當您使用提供者 SDK 時，使用者的登入體驗可以與執行應用程式的作業系統更緊密地整合。這也會提供您一個提供者權杖以及用戶端上的一些使用者資訊，讓它更容易取用圖表 API 和自訂使用者體驗。在部落格和論壇上，您有時會看到這稱為「用戶端流程」或「用戶端導向流程」，因為用戶端的程式碼會將使用者登入，而且用戶端程式碼可以存取提供者權杖。

取得提供者權杖之後，必須傳送給 App Service 進行驗證。當 App Service 驗證權杖之後，App Service 會建立新的 App Service 權杖來傳回給用戶端。Mobile Apps 用戶端 SDK 提供 Helper 方法來管理此交換，並自動將權杖附加至應用程式後端的所有要求。開發人員也可以選擇持續參考提供者權杖。

### 不使用提供者 SDK 的行動驗證

如果您不想設定提供者 SDK，您可以允許 Azure App Service 的 Mobile Apps 功能替您登入。Mobile Apps 用戶端 SDK 將為您選擇的提供者開啟網頁檢視，並將使用者登入。在部落格和論壇上，您有時會看到這稱為「伺服器流程」或「伺服器導向流程」，因為伺服器會管理使用者登入程序，而用戶端 SDK 永遠不會收到提供者權杖。

每個平台的驗證教學課程都包含啟動此流程的程式碼。在流程結束時，用戶端 SDK 會擁有 App Service 權杖，而該權杖會自動附加至對應用程式後端的所有要求。

### 服務對服務驗證

您除了可以授權使用者存取您的應用程式，還可以信任另一個應用程式來呼叫您自己的 API。例如，您可以讓一個 Web 應用程式呼叫另一個 Web 應用程式中的 API。在此情況下，您可以使用服務帳戶的認證 (而非使用者認證) 來取得權杖。在 Azure Active Directory 用語中，服務帳戶亦稱為*服務主體*，而使用這類帳戶的驗證又稱為「服務對服務案例」。

>[AZURE.IMPORTANT] 由於行動應用程式是在客戶裝置上執行，因此行動應用程式_不算_是信任的應用程式，不應該使用服務主體流程。反之，行動應用程式應該使用稍早詳述的使用者流程。

在服務對服務案例下，App Service 可以使用 Azure Active Directory 保護您的應用程式。呼叫端應用程式只需要提供 Azure Active Directory 服務主體授權權杖，而此權杖是經由提供用戶端識別碼和用戶端密碼而從 Azure Active Directory 取得。[API Apps 的服務主體驗證][apia-service]教學課程中說明這個使用 ASP.NET API 應用程式的案例範例。

如果您想要使用 App Service 驗證來處理服務對服務案例，您可以使用用戶端憑證或基本驗證。如需 Azure 中用戶端憑證的詳細資訊，請參閱[如何設定 Web Apps 的 TLS 相互驗證](../app-service-web/app-service-web-configure-tls-mutual-auth.md)。如需 ASP.NET 中基本驗證的詳細資訊，請參閱[ASP.NET Web API 2 中的驗證篩選](http://www.asp.net/web-api/overview/security/authentication-filters)。

App Service 邏輯應用程式至 API 應用程式的服務帳戶驗證屬於特殊案例，在[將您裝載在 App Service 上的自訂 API 與邏輯應用程式一起使用](../app-service-logic/app-service-logic-custom-hosted-api.md)中有詳細說明。

## <a name="authorization"></a>App Service 中授權的運作方式

您可以完整控制哪些要求可存取您的應用程式。可以使用下列任何一個行為對 App Service 驗證 / 授權進行設定︰

- 只允許通過驗證的要求進入您的應用程式。

	如果瀏覽器收到匿名要求，App Service 會重新導向至您選擇的識別提供者頁面，讓使用者能夠登入。如果要求來自行動裝置，則傳回的回應是 HTTP _401 未經授權_回應。

	使用此選項，您完全不需要在應用程式撰寫任何驗證程式碼。如果需要更細部的授權，您的程式碼可取得使用者的相關資訊。

- 允許所有要求抵達您的應用程式，但只接受通過驗證的要求，並在 HTTP 標頭中傳遞驗證資訊。

	此選項會將您的應用程式程式碼的授權決策延後。這提供更大的彈性來處理匿名要求，但您需要撰寫程式碼。

- 允許所有要求進入您的應用程式，不對要求中的驗證資訊採取任何動作。

	在此情況下，會關閉驗證/授權功能。將驗證和授權工作全部交由應用程式程式碼來處理。

上述行為是由 Azure 入口網站中的 [當要求未經驗證時所要採取的動作] 選項所控制。如果您選擇 [使用提供者名稱登入]，則必須驗證所有要求。[允許要求 (無動作)] 會將授權決策交由您的程式碼決定，但仍會提供驗證資訊。如果您想要程式碼能夠處理所有要求，可以停用驗證/授權功能。

## 在應用程式中使用您的使用者身分識別

App Service 會使用特殊標頭，將某些使用者資訊傳遞至您的應用程式。外部要求會禁止這些標頭，必須由 App Service 驗證/授權設定才會出現。某些範例標頭包括︰

* X-MS-CLIENT-PRINCIPAL-NAME
* X-MS-CLIENT-PRINCIPAL-ID
* X-MS-TOKEN-FACEBOOK-ACCESS-TOKEN
* X-MS-TOKEN-FACEBOOK-EXPIRES-ON

以任何語言或架構撰寫的程式碼可以從這些標頭中取得所需的資訊。針對 ASP.NET 4.6 應用程式，**ClaimsPrincipal** 會自動設定適當的值。

您的應用程式也可透過應用程式的 `/.auth/me` 端點上的 HTTP GET，取得使用者的其他詳細資料。要求隨附的有效權杖會傳回 JSON 承載，其中包含有關所使用的提供者、基礎提供者權杖的詳細資料，以及其他一些使用者資訊。Mobile Apps 伺服器 SDK 提供 Helper 方法來處理此資料。如需詳細資訊，請參閱[如何使用 Azure Mobile Apps Node.js SDK](../app-service-mobile/app-service-mobile-node-backend-how-to-use-server-sdk.md#howto-tables-getidentity) 和[使用適用於 Azure 行動應用程式的 .NET 後端伺服器 SDK](../app-service-mobile/app-service-mobile-dotnet-backend-how-to-use-server-sdk.md#user-info)。

## 文件和其他資源

### 識別提供者
下列教學課程示範如何設定 App Service 來使用不同的驗證提供者：

- [如何設定您的 App 以使用 Azure Active Directory 登入][AAD]
- [如何設定 App 以使用 Facebook 登入][Facebook]
- [如何設定 App 以使用 Google 登入][Google]
- [如何設定 App 以使用 Microsoft 帳戶登入][MSA]
- [如何設定 App 以使用 Twitter 登入][Twitter]

如果您想要使用不同於這裡提供的身分識別系統，則您也可以利用 [Mobile Apps .NET 伺服器 SDK 中的預覽自訂驗證支援][custom-auth]，此支援可用於 Web 應用程式、行動應用程式或 API 應用程式。

### Web 應用程式
下列教學課程示範如何將驗證新增至 Web 應用程式：

- [開始使用 Azure App Service - 第 2 部分][web-getstarted]

### 行動應用程式
下列教學課程示範如何使用伺服器導向流程，將驗證新增至您的行動用戶端︰

- [將驗證新增至您的 iOS 應用程式][iOS]
- [將驗證新增至您的 Android 應用程式][Android]
- [將驗證新增至您的 Windows App][Windows]
- [將驗證新增至 Xamarin.iOS 應用程式][Xamarin.iOS]
- [將驗證新增至 Xamarin.Android 應用程式][Xamarin.Android]
- [將驗證加入 Xamarin.Forms 應用程式中][Xamarin.Forms]
- [將驗證新增至您的 Cordova 應用程式][Cordova]

如果您想要使用適用於 Azure Active Directory 的用戶端導向流程，請參考下列資源︰

- [使用 Active Directory Authentication Library for iOS][ADAL-iOS]
- [使用 Active Directory Authentication Library for Android][ADAL-Android]
- [使用 Active Directory Authentication Library for Windows 和 Xamarin][ADAL-dotnet]

如果您想要使用適用於 Facebook 的用戶端導向流程，請參考下列資源︰

- [使用 Facebook SDK for iOS](../app-service-mobile/app-service-mobile-ios-how-to-use-client-library.md#facebook-sdk)

如果您想要使用適用於 Twitter 的用戶端導向流程，請參考下列資源︰

- [使用 Twitter Fabric for iOS](../app-service-mobile/app-service-mobile-ios-how-to-use-client-library.md#twitter-fabric)

如果您想要使用適用於 Google 的用戶端導向流程，請參考下列資源︰

- [使用 Google Sign-In SDK for iOS](../app-service-mobile/app-service-mobile-ios-how-to-use-client-library.md#google-sdk)

### API 應用程式
下列教學課程示範如何保護您的 API 應用程式︰

- [Azure App Service 中 API Apps 的使用者驗證][apia-user]
- [在 Azure App Service 中 API Apps 的服務主體驗證][apia-service]









[apia-user]: ../app-service-api/app-service-api-dotnet-user-principal-auth.md
[apia-service]: ../app-service-api/app-service-api-dotnet-service-principal-auth.md

[web-getstarted]: ../app-service-web/app-service-web-get-started-2.md#authenticate-your-users

[iOS]: ../app-service-mobile/app-service-mobile-ios-get-started-users.md
[Android]: ../app-service-mobile/app-service-mobile-android-get-started-users.md
[Xamarin.iOS]: ../app-service-mobile/app-service-mobile-xamarin-ios-get-started-users.md
[Xamarin.Android]: ../app-service-mobile/app-service-mobile-xamarin-android-get-started-users.md
[Xamarin.Forms]: ../app-service-mobile/app-service-mobile-xamarin-forms-get-started-users.md
[Windows]: ../app-service-mobile/app-service-mobile-windows-store-dotnet-get-started-users.md
[Cordova]: ../app-service-mobile/app-service-mobile-cordova-get-started-users.md

[AAD]: ../app-service-mobile/app-service-mobile-how-to-configure-active-directory-authentication.md
[Facebook]: ../app-service-mobile/app-service-mobile-how-to-configure-facebook-authentication.md
[Google]: ../app-service-mobile/app-service-mobile-how-to-configure-google-authentication.md
[MSA]: ../app-service-mobile/app-service-mobile-how-to-configure-microsoft-authentication.md
[Twitter]: ../app-service-mobile/app-service-mobile-how-to-configure-twitter-authentication.md

[custom-auth]: ../app-service-mobile/app-service-mobile-dotnet-backend-how-to-use-server-sdk.md#custom-auth

[ADAL-Android]: ../app-service-mobile/app-service-mobile-android-how-to-use-client-library.md#adal
[ADAL-iOS]: ../app-service-mobile/app-service-mobile-ios-how-to-use-client-library.md#adal
[ADAL-dotnet]: ../app-service-mobile/app-service-mobile-dotnet-how-to-use-client-library.md#adal

<!---HONumber=AcomDC_0831_2016-->