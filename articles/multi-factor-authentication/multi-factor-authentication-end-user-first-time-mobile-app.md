<properties
	pageTitle="搭配 Azure Multi-Factor Authentication 使用行動裝置應用程式作為連絡方式 | Microsoft Azure"
	description="本文說明如何使用行動裝置應用程式作為 Azure Multi-Factor Authentication 的主要連絡方式。"
	services="multi-factor-authentication"
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
	ms.date="08/30/2016"
	ms.author="kgremban"/>

# 搭配 Azure Multi-Factor Authentication 使用行動裝置應用程式作為連絡方式

如果您想要使用 Microsoft Authenticator 應用程式作為主要連絡方式，您可以使用這份文件。本文會引導您設定 Azure Multi-Factor Authentication，以使用行動裝置應用程式作為您的主要連絡方式。

Microsoft Authenticator 應用程式適用於 [Windows Phone](http://go.microsoft.com/fwlink/?Linkid=825071)、[Android](http://go.microsoft.com/fwlink/?Linkid=825072) 和 [iOS](http://go.microsoft.com/fwlink/?Linkid=825073)。

## 使用 Microsoft Authenticator 作為連絡方式


1. 在 [其他安全性驗證] 畫面上，從下拉式清單中選取 [行動裝置應用程式]。
2. 選取 [通知] 或 [單次密碼]，然後選取 [設定]。

	![[其他安全性驗證] 畫面](./media/multi-factor-authentication-end-user-first-time-mobile-app/mobileapp.png)

3. 在已安裝 Microsoft Authenticator 應用程式的手機上，開啟該應用程式，然後按一下 **+** 來新增帳戶。
4. 指定您想要新增的是工作帳戶還是學校帳戶。這會開啟 QR 代碼掃描器。如果您的相機未正常運作，您可以選取手動輸入公司資訊。如需詳細資訊，請參閱[手動新增帳戶](#add-an-account-manually)。

	![選取帳戶的畫面](./media/multi-factor-authentication-end-user-first-time-mobile-app/scan.png)

	![選取要掃描 QR 代碼還是手動輸入帳戶的畫面](./media/multi-factor-authentication-end-user-first-time-mobile-app/scan4.png)

5. 掃描隨畫面顯示的 QR 代碼圖片來設定行動裝置應用程式。按一下 [完成] 以關閉 QR 代碼畫面。

	![QR 代碼畫面](./media/multi-factor-authentication-end-user-first-time-mobile-app/scan2.png)

	如果您無法取得要掃描的 QR 代碼，可以手動輸入資訊。

	![手動輸入資訊的畫面](./media/multi-factor-authentication-end-user-first-time-mobile-app/barcode.png)

6. 在手機上完成啟動時，選取 [連絡我]。此步驟會將通知或驗證碼傳送到您的手機。請選取 [驗證]。

	![您選取 [驗證] 來登入的畫面](./media/multi-factor-authentication-end-user-first-time-mobile-app/verify.png)

7. 如果貴公司需要 PIN 來核准登入驗證，請輸入它。

	![輸入 PIN 的方塊](./media/multi-factor-authentication-end-user-first-time-mobile-app/scan3.png)

8. 輸入完 PIN 之後，請選取 [關閉]。此時，您的驗證應會成功。
9. 建議您輸入您的行動電話號碼，以防萬一您無法存取行動裝置應用程式。請從下拉式清單中指定您的國家/地區，然後在國家/地區名稱旁邊的方塊中輸入您的行動電話號碼。選取 [下一步]。
10. 此時，您已設定好您的連絡方式。現在即可設定非瀏覽器應用程式 (例如 Outlook 2010 或更舊版本) 的應用程式密碼。如果您不使用這些應用程式，請選取 [完成]。否則，請繼續下一個步驟。

	![建立應用程式密碼的畫面](./media/multi-factor-authentication-end-user-first-time-mobile-app/step4.png)

11. 如果您使用非瀏覽器應用程式，請將提供的應用程式密碼複製並貼到您的應用程式中。如需有關個別應用程式 (例如 Outlook 和 Lync) 的步驟，請參閱＜如何將您電子郵件中的密碼變更為應用程式密碼＞及＜如何將您應用程式中的密碼變更為應用程式密碼＞。
12. 選取 [完成]。


## 手動新增帳戶
如果您想要手動新增 Azure 帳戶：

1. 選取 [手動輸入帳戶] 按鈕。

	![輸入代碼和 URL 的畫面](./media/multi-factor-authentication-end-user-first-time-mobile-app/addaccount.png)

	![輸入代碼和 URL 的畫面](./media/multi-factor-authentication-end-user-first-time-mobile-app/addaccount2.png)

2. 如果您有一個已有 Azure Multi-Factor Authentication 的帳戶，請輸入顯示條碼的同一頁上所提供的代碼和 URL。此資料會填入行動裝置應用程式上的 [代碼] 和 [URL] 方塊中。

	![設定](./media/multi-factor-authentication-end-user-first-time-mobile-app/barcode2.png)

3. 當啟用完成時，請選取 [連絡我]。此步驟會將通知或驗證碼傳送到您的手機。請選取 [驗證]。

<!---HONumber=AcomDC_0921_2016-->