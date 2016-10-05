<properties
	pageTitle="第一次使用 Azure Multi-Factor Authentication 登入"
	description="本頁面描述使用者第一次登入經驗的經驗。"
	services="multi-factor-authentication"
	keywords="如何使用 Azure Directory, 雲端中的 Active Directory, Active Directory 教學課程"
	documentationCenter=""
	authors="kgremban"
	manager="femila"
	editor="curtland"/>

<tags
	ms.service="multi-factor-authentication"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/22/2016"
	ms.author="kgremban"/>
# Azure Multi-factor Authentication 的設定體驗

 當系統管理員已將您的帳戶設定為要求您的密碼與來自您的電話的回應必須同時用來驗證您的身分時，會使用其他安全性驗證設定。如果系統管理員已將您的帳戶設定為需要其他安全性驗證，**您要等到完成自動註冊程序才能登入**。

## 決定您使用 Multi-Factor Authentication 的方式

 在您的帳戶設定後第一次登入時，系統將會提示您開始進行自動註冊程序。按一下 [**立即設定**]，即可開始這個程序。

![設定](./media/multi-factor-authentication-end-user-first-time/first.png)

在註冊程序中，您將可指定偏好的驗證方法。這可以是下表中的任何一種方法。如需詳細資訊 (包括逐步解說)，只要按一下其中一種方法即可。

方法|說明
:------------- | :------------- |
[行動電話通話](multi-factor-authentication-end-user-first-time-mobile-phone.md)| 撥打自動語音電話給驗證電話。使用者可接聽電話並按電話鍵盤上的 # 進行驗證。此電話號碼將不會同步到內部部署 Active Directory。
[行動電話簡訊](multi-factor-authentication-end-user-first-time-mobile-phone.md)|傳送包含驗證碼的簡訊給使用者。系統會提示使用者使用驗證碼來回覆簡訊，或在登入介面中輸入驗證碼。
[辦公室電話通話](multi-factor-authentication-end-user-first-time-office-phone.md)|撥打自動語音電話給使用者。使用者可接聽電話並按電話鍵盤上的 # 進行驗證。
[行動應用程式](multi-factor-authentication-end-user-first-time-mobile-app.md)|將通知推送至使用者的智慧型手機或平板電腦上的 Microsoft 驗證器應用程式。使用者可點選應用程式中的 [驗證] 進行驗證。或者，應用程式也可做為 OTP 權杖進行離線驗證。使用者在登入畫面中輸入此權杖進行驗證。<br><p> Microsoft 驗證器應用程式可以在 2 種模式下運作，以提供 Multi-Factor Authentication 服務可提供的額外安全性。分述如下：<li>**通知** - 在此模式中，Microsoft 驗證器應用程式可防止未經授權存取帳戶，並停止詐騙交易。使用推播通知至您的電話或已註冊的裝置即可進行。只需檢視通知，如果合法，則選取 [驗證]。否則，您可選擇 [拒絕]，或選擇拒絕並回報詐騙通知。如需回報詐騙通知的資訊，請參閱＜如何使用 Multi-Factor Authentication 的拒絕和回報詐騙功能＞。</li><p><li>**一次性密碼** - 在此模式中，Microsoft 驗證器應用程式可做為軟體權杖以產生 OATH 驗證碼。然後，可以連同使用者名稱和密碼一起輸入此驗證碼，以提供第二種形式的驗證。</li><br><p> Microsoft 驗證器應用程式適用於 [Windows Phone](http://go.microsoft.com/fwlink/?Linkid=825071)、[Android](http://go.microsoft.com/fwlink/?Linkid=825072) 和 [IOS](http://go.microsoft.com/fwlink/?Linkid=825073)。

<!---HONumber=AcomDC_0921_2016-->