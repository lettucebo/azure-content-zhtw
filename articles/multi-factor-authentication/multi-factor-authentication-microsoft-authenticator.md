<properties
	pageTitle="適用於行動電話的 Microsoft 驗證器應用程式 | Microsoft Azure"
	description="了解如何升級至最新版的 Azure Authenticator。"
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
	ms.date="08/22/2016"
	ms.author="kgremban"/>

# Microsoft 驗證器

Microsoft 驗證器應用程式可在 Azure 帳戶 (例如 bsimon@contoso.onmicrosoft.com)、內部部署工作帳戶 (例如 bsimon@contoso.com) 或 Microsoft 帳戶 (例如 bsimon@outlook.com) 提供額外一層安全性。

此應用程式可以下列兩種方式之一運作︰

- **通知**。此應用程式可協助防止未經授權即存取帳戶，並藉由推播通知到您的手機或平板電腦來停止詐騙交易。只需檢視通知，如果合法，則選取 [驗證]。否則，您可以選取 [拒絕]。如需拒絕通知的相關資訊，請參閱＜如何使用 Multi-Factor Authentication 的拒絕和回報詐騙功能＞。

- **密碼與驗證碼**。此應用程式可以作為軟體權杖來產生 OATH 驗證碼。您必須在登入畫面出現提示時輸入應用程式提供的驗證碼，以及使用者名稱和密碼。驗證碼提供第二種形式的驗證。

隨著 Microsoft 驗證器應用程式的推出，舊的 Azure Authenticator 應用程式將被取代。Azure 驗證器應用程式仍可繼續運作，但如果您決定改用新的 Microsoft 驗證器應用程式，本文可協助您。

## 安裝應用程式

Microsoft 驗證器應用程式適用於 [Windows Phone](http://go.microsoft.com/fwlink/?Linkid=825071)、[Android](http://go.microsoft.com/fwlink/?Linkid=825072) 和 [IOS](http://go.microsoft.com/fwlink/?Linkid=825073)。

## 將帳戶新增至應用程式

針對每個想要新增至 Microsoft 驗證器應用程式的帳戶，請使用下列其中一個程序。

### 使用 QR 代碼掃描器將帳戶新增至應用程式

1. 移至安全性驗證設定畫面。如需如何進入此畫面的詳細資訊，請參閱[變更安全性設定](multi-factor-authentication-end-user-manage-settings.md)。

2. 選取 [設定]。

	![安全性驗證設定畫面上的設定按鈕](./media/multi-factor-authentication-azure-authenticator/azureauthe.png)

	這會顯示有 QR 代碼的畫面。

	![提供 QR 代碼的畫面](./media/multi-factor-authentication-azure-authenticator/barcode2.png)

3. 開啟 Microsoft 驗證器應用程式。在 [帳戶] 畫面上，選取 [+]，然後指定您要新增工作帳戶或學校帳戶。

	![有加號的帳戶畫面](./media/multi-factor-authentication-azure-authenticator/addaccount3.png)

	![指定工作帳戶或學校帳戶的畫面](./media/multi-factor-authentication-end-user-first-time-mobile-app/scan.png)

4. 使用相機來掃描 QR 代碼，然後選取 [完成] 關閉 QR 代碼畫面。

	![掃描 QR 代碼的畫面](./media/multi-factor-authentication-end-user-first-time-mobile-app/scan2.png)

	如果相機運作不正常，您可以手動輸入 QR 代碼和 URL。如需詳細資訊，請參閱[手動新增帳戶至應用程式](#add-an-account-to-the-app-manually)。

5. 請等候帳戶啟用。啟動完成時，選取 [與我連絡]。這會將通知或驗證碼傳送到您的手機。選取 [驗證]。

	![您選取 [驗證] 來登入的畫面](./media/multi-factor-authentication-end-user-first-time-mobile-app/verify.png)

6. 如果貴公司需要 PIN 來核准登入驗證，請輸入它。

	![輸入 PIN 的方塊](./media/multi-factor-authentication-end-user-first-time-mobile-app/scan3.png)

7. 輸入完 PIN 之後，請選取 [關閉]。此時，您的驗證應會成功。
8. 建議您輸入您的行動電話號碼，以防萬一您無法存取應用程式。請從下拉式清單中指定您的國家/地區，然後在國家/地區名稱旁邊的方塊中輸入您的行動電話號碼。選取 [下一步]。
9. 此時，您已設定好您的連絡方式。現在即可設定非瀏覽器應用程式 (例如 Outlook 2010 或更舊版本) 的應用程式密碼。如果您不使用這些應用程式，請選取 [完成]。否則，請繼續下一個步驟。

	![建立應用程式密碼的畫面](./media/multi-factor-authentication-end-user-first-time-mobile-app/step4.png)

10. 如果您使用非瀏覽器應用程式，請將提供的應用程式密碼複製並貼到您的應用程式中。如需有關個別應用程式 (例如 Outlook 和 Lync) 的步驟，請參閱＜如何將您電子郵件中的密碼變更為應用程式密碼＞及＜如何將您應用程式中的密碼變更為應用程式密碼＞。
11. 選取 [完成]。

您現在應該會在 [帳戶] 畫面上看到新的帳戶。

![帳戶畫面](./media/multi-factor-authentication-azure-authenticator/accounts.png)

### 手動新增帳戶至應用程式

1. 移至安全性驗證設定畫面。如需如何進入此畫面的詳細資訊，請參閱[變更安全性設定](multi-factor-authentication-end-user-manage-settings.md)。

2. 選取 [設定]。

	![安全性驗證設定畫面上的設定按鈕](./media/multi-factor-authentication-azure-authenticator/azureauthe.png)

	這會顯示有 QR 代碼的畫面。記下代碼和 URL。

	![提供 QR 代碼與 URL 的畫面](./media/multi-factor-authentication-azure-authenticator/barcode2.png)

3. 開啟 Microsoft 驗證器應用程式。在 [帳戶] 畫面上，選取 [+]，然後指定您要新增工作帳戶或學校帳戶。

	![有加號的帳戶畫面](./media/multi-factor-authentication-azure-authenticator/addaccount3.png)

	![指定工作帳戶或學校帳戶的畫面](./media/multi-factor-authentication-end-user-first-time-mobile-app/scan.png)

4. 在掃描器上，選取 [手動輸入代碼]。

	![掃描 QR 代碼的畫面](./media/multi-factor-authentication-end-user-first-time-mobile-app/scan2.png)

5. 在應用程式的適當方塊中輸入代碼和 URL。

	![輸入代碼和 URL 的畫面](./media/multi-factor-authentication-azure-authenticator/manual.png)

	![輸入代碼和 URL 的畫面](./media/multi-factor-authentication-end-user-first-time-mobile-app/addaccount2.png)

6. 請等候帳戶啟用。啟動完成時，選取 [與我連絡]。這會將通知或驗證碼傳送到您的手機。選取 [驗證]。

您現在應該會在 [帳戶] 畫面上看到新的帳戶。

![帳戶畫面](./media/multi-factor-authentication-azure-authenticator/accounts.png)

### 使用 Touch ID 將帳戶新增至應用程式

在 iOS 上的 Microsoft 驗證器應用程式支援 Touch ID。Azure Multi-Factor Authentication 可讓組織要求裝置的 PIN 碼。使用 Touch ID 時，iOS 使用者不需要輸入 PIN 碼。相反地，他們可以掃描其指紋，並選取 [核准]。

設定 Touch ID 搭配 Azure 驗證器很簡單。您可以使用 PIN 碼完成一般驗證挑戰。如果您的裝置支援 Touch ID，Microsoft 驗證器將會自動為該帳戶進行設定。

![Touch ID 安裝程式的驗證](./media/multi-factor-authentication-azure-authenticator/touchid1.png)

往後當您被要求驗證登入時，您只要選取收到的推播通知並且掃描您的指紋即可，不需要輸入您的 PIN。

![推播通知](./media/multi-factor-authentication-azure-authenticator/touchid2.png)

## 解除安裝舊版 Azure 驗證應用程式

在將所有帳戶加入至新應用程式之後，您可以在手機中解除安裝舊版應用程式。

## 刪除帳戶

若要從 Microsoft 驗證器應用程式移除帳戶，請選取帳戶，然後選取 [刪除]。

![刪除按鈕](./media/multi-factor-authentication-azure-authenticator/remove.png)

<!---HONumber=AcomDC_0921_2016-->