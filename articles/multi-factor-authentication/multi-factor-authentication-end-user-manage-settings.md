<properties
	pageTitle="使用 Azure Multi-Factor Authentication 時碰到困難 | Microsoft Azure"
	description="本文件將提供如果使用者使用 Azure Multi-Factor Authentication 遇到問題時，該怎麼辦的資訊。"
	services="multi-factor-authentication"
	keywords = "多重要素驗證用戶端, 驗證的問題, 相互關聯識別碼"
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

# 使用 Azure Multi-Factor Authentication 時碰到困難
>[AZURE.IMPORTANT]
協助我們改進此頁面。如果您在此頁面上找不到您的問題的解答，請提供詳細的意見反應，讓我們可以新增。

下列資訊可協助您解決一些常見問題。


- [相互關聯識別碼錯誤](#correlation-id-errors)
- [我的手機遺失或遭竊](#i-have-lost-my-phone-or-it-was-stolen)
- [我想要變更我的電話號碼](#i-want-to-change-my-phone-number)
- [我有新的電話，需要變更我的電話號碼](#i-have-a-new-phone-and-need-to-change-my-phone-number)
- [我的手機沒收到驗證碼](#i-am-not-receiving-a-code-or-a-call-on-my-phone)
- [應用程式密碼無效](#app-passwords-are-not-working)
- [如何清除舊裝置的 Microsoft驗證器並移到新裝置？](#how-do-i-clean-up-microsoft-authenticator-from-my-old-device-and-move-to-a-new-one)
- [我找不到我的問題的解答](#i-didnt-find-an-answer-to-my-problem)

##相互關聯識別碼錯誤
如果您已試過下列疑難排解步驟，但仍遇到問題，您可以將問題張貼在 [Azure AD 論壇](https://social.msdn.microsoft.com/forums/azure/home?forum=WindowsAzureAD)、[搜尋 Microsoft 知識庫 (KB)](https://www.microsoft.com/zh-TW/Search/result.aspx?q=azure%20active%20directory%20connect&form=mssupport)或[連絡支援人員](https://support.microsoft.com/zh-TW)，我們將會儘快了解您的問題。

當您連絡支援人員時，建議您包含下列資訊：

 - **錯誤的一般描述** – 使用者實際看到的錯誤訊息。 如果沒有任何錯誤訊息，請詳細說明您所注意到的未預期行為。
 - **頁面** – 您看到錯誤時的所在頁面 (包含 URL)。
 - **錯誤碼** - 您接收到的特定錯誤碼。
 - **工作階段識別碼** - 您接收到的特定工作階段識別碼。
 - **相互關聯識別碼** – 當使用者看見此錯誤時所產生的相互關連識別碼。
 - **時間戳記** – 您看到錯誤時的精確日期和時間 (包含時區)。

![相互關連識別碼](./media/multi-factor-authentication-end-user-manage/correlation.png)

 - **使用者識別碼** – 看到錯誤的使用者其識別碼為何 (例如 user@contoso.com)？
 - **使用者的相關資訊** – 使用者已同盟、密碼雜湊同步處理、只在雲端嗎？ 使用者是否有 Azure AD Premium、Enterprise Mobility，或 Azure AD Basic 授權指派？ 使用者是否使用 Office 365？等等。

包含這些資訊將有助於我們儘快為您解決問題。

## 我的手機遺失或遭竊？
如果您的手機遺失或遭竊，建議讓您的管理員重設您的[應用程式密碼](multi-factor-authentication-manage-users-and-devices.md#delete-users-existing-app-passwords)，並清除任何[已記住的裝置](multi-factor-authentication-manage-users-and-devices.md#restore-mfa-on-all-suspended-devices-for-a-user)。

有兩個選項可讓您恢復使用您的帳戶。首先，如果您已設有備用驗證電話號碼，您可用它來恢復使用您的帳戶並變更安全性設定。

如果您已有指定次要驗證電話號碼，您可以使用它來登入。![設定](./media/multi-factor-authentication-end-user-manage/altphone.png) 請注意，在上方的螢幕擷取畫面中，已設有兩個電話號碼，一個的結尾是 67，另一個的結尾是 30。

若要使用備用的電話號碼登入，請依照一般方式登入，然後選擇 [使用不同的驗證選項]。![不同的驗證方式](./media/multi-factor-authentication-end-user-manage/differentverification.png)

然後選取其他電話號碼。在此案例中，您要選取 [撥號給我 +X XXXXXXXX30]

![備用手機](./media/multi-factor-authentication-end-user-manage/altphone2.png)

>[AZURE.IMPORTANT]
請務必設定次要驗證電話號碼。由於您的主要電話號碼與行動應用程式可能在同一個手機上，因此如果您的手機遺失或遭竊，您只能依靠次要電話號碼恢復使用您的帳戶。

如果您尚未設定次要驗證電話號碼，則必須連絡您的管理員清除您的設定，讓您下次登入時，會再次出現[設定多重要素驗證](multi-factor-authentication-manage-users-and-devices.md#require-selected-users-to-provide-contact-methods-again)的提示。

## 我想要變更我的電話號碼
視您使用 Multi-Factor Authentication 的方式而定，有一些地方可讓您變更如電話號碼等設定。使用下表協助您選擇最符合的狀況。

如何使用多因素驗證|說明
:------------- | :------------- |
[搭配 Office 365 使用](#changing-your-settings-with-office-365)| 這表示您會透過 Office 365 入口網站變更您的設定。
[我不知道](#changing-your-settings-with-the-myapps-portal)|這表示您會登入 [http://myapps.microsoft.com](http://myapps.microsoft.com)，並在此變更您的設定。
[搭配 Microsoft Azure 使用](#changing-your-settings-with-microsoft-azure)| 這表示您會透過 Azure 入口網站變更您的設定。



### 變更您的 Office 365 設定


如果您搭配 Office 365 使用 Multi-Factor Authentication，您會透過 Office 365 入口網站管理其他安全性驗證設定。

#### 在 Office 365 入口網站中變更您的設定

1. 登入 [Office 365 入口網站](https://login.microsoftonline.com/)。
2. 在右上角選取 Widget 並選擇 [Office 365 設定]。
3. 按一下 [其他安全性驗證]。
4. 選取右側的 [更新用於帳戶安全性的電話號碼] 連結。![O365](./media/multi-factor-authentication-end-user-manage/o365a.png)
5. 這會引導您至允許您變更設定的頁面。![O365](./media/multi-factor-authentication-end-user-manage/o365b.png)


### 變更您的 Myapps 入口網站設定

如果您不確定您使用 Multi-Factor Authentication 的方式，您一律可以透過 Myapps 入口網站變更設定。

#### 在 Myapps 入口網站變更您的設定

1. 登入 [https://myapps.microsoft.com](https://myapps.microsoft.com)
2. 在頂端，選取 [設定檔]。
3. 選取 [其他安全性驗證]。![Myapps](./media/multi-factor-authentication-end-user-manage/myapps1.png)
4. 這會引導您至允許您變更設定的頁面。

![Proofup](./media/multi-factor-authentication-end-user-manage-myapps/proofup.png)

### 變更您的 Microsoft Azure 設定

如果您搭配 Azure 使用 Multi-Factor Authentication，您會透過 Azure 入口網站變更您的設定。

#### 在 Azure 入口網站中存取其他安全性驗證設定


1. 登入 Azure 入口網站。
2. 在 Azure 入口網站頂端，按一下您的使用者名稱。這會顯示下拉式方塊。
3. 從下拉式方塊中，選取 [其他安全性驗證]。![Azure](./media/multi-factor-authentication-end-user-manage/azure1.png)
4. 這會引導您至允許您變更設定的頁面。![Proofup](./media/multi-factor-authentication-end-user-manage-azure/proofup.png)

##我有新的電話，需要變更我的電話號碼

如果您有新的電話，需要變更 MFA 使用的主要連絡電話，您可以使用下列兩種方式之一來執行這個動作。

>[AZURE.IMPORTANT]
請務必設定次要驗證電話號碼。由於您的主要電話號碼與行動應用程式可能在同一個手機上，因此如果您的手機遺失或遭竊，您只能依靠次要電話號碼恢復使用您的帳戶。

第一個方式是使用次要驗證方法。如果您已有指定次要驗證電話號碼，您可以使用它來登入。![設定](./media/multi-factor-authentication-end-user-manage/altphone.png) 請注意，在上方的螢幕擷取畫面中，已設有兩個電話號碼，一個的結尾是 67，另一個的結尾是 30。

若要使用備用的電話號碼登入，請依照一般方式登入，然後選擇 [使用不同的驗證選項]。![不同的驗證方式](./media/multi-factor-authentication-end-user-manage/differentverification.png)

然後選取其他電話號碼。在此案例中，您要選取 [撥號給我 +X XXXXXXXX30]

![備用手機](./media/multi-factor-authentication-end-user-manage/altphone2.png)

第二個方式是連絡您的系統管理員或為您設定 MFA 的人員。只有在您尚未設定次要驗證電話號碼時，才需要這麼做。否則，您必須連絡您的系統管理員或為您設定 MFA 的人員，由他們清除您的設定，讓您下次登入時，會再次出現[設定多重要素驗證](multi-factor-authentication-manage-users-and-devices.md#require-selected-users-to-provide-contact-methods-again)的提示。

##我的手機沒收到驗證碼或來電

首先，您需要確定下列事項：



- 如果您已選取在行動電話收到來電，請確定您有適當的手機訊號強度。傳遞速度以及是否可傳遞，取決於地點與服務供應商。
- 如果您選取要用簡訊將驗證碼傳送到行動電話上，請確定您的費率方案和裝置支援傳遞簡訊。傳遞速度以及是否可傳遞，取決於地點與服務供應商。另外，嘗試接收驗證碼時，請確定手機訊號強度能夠滿足需求。
- 如果您選擇透過行動應用程式接收驗證碼，請確定您的手機訊號強度夠強。請注意，傳遞速度以及是否可傳遞，取決於地點與服務供應商。

如果您擁有智慧型手機，我們建議您使用適用於 [Windows Phone](http://go.microsoft.com/fwlink/?Linkid=825071)、[Android](http://go.microsoft.com/fwlink/?Linkid=825072) 及 [iOS](http://go.microsoft.com/fwlink/?Linkid=825073) 的 Microsoft 驗證器應用程式。

在登入時選擇 [使用不同的驗證選項]，就可以切換透過簡訊或透過行動應用程式接收驗證碼。

![不同的驗證方式](./media/multi-factor-authentication-end-user-manage/differentverification.png)

有時候某一個服務的傳遞會比另一個可靠。

請注意，如果您收到多個驗證碼，只有最新的驗證碼有效。

如果您先前已設定備用電話，建議您在登入頁面出現提示時選取該電話，然後再試一次。如果您沒有設定其他方法，請要求您的管理員清除您的設定，讓您下次登入時，會再次出現[設定多重要素驗證](multi-factor-authentication-manage-users-and-devices.md#require-selected-users-to-provide-contact-methods-again) 的提示。

##應用程式密碼無效
首先，請確定您輸入的是正確的應用程式密碼。如果仍無法登入，請嘗試登入並[建立新的應用程式密碼](multi-factor-authentication-end-user-app-passwords.md)。如果還是無法登入，請要求管理員[刪除您現有的應用程式密碼](multi-factor-authentication-manage-users-and-devices.md#delete-users-existing-app-passwords)，然後建立新的密碼並使用。

##如何清除舊裝置的 Microsoft驗證器並移到新裝置？
無論您是從裝置解除安裝應用程式，或是重新整理裝置，都不會移除後端的啟動功能。您應該使用[移至新裝置](multi-factor-authentication-microsoft-authenticator.md#how-to-move-to-the-new-microsoft-authenticator-app)中所述的步驟。

##我找不到我的問題的解答
如果在此頁面上找不到您的問題的解答，您可以將問題張貼在 [Azure AD 論壇](https://social.msdn.microsoft.com/forums/azure/home?forum=WindowsAzureAD)、[搜尋 Microsoft 知識庫 (KB)](https://www.microsoft.com/zh-TW/Search/result.aspx?q=azure%20active%20directory%20connect&form=mssupport) 或[連絡支援人員](https://support.microsoft.com/zh-TW)。

此外，您可以連絡您的系統管理員或為您設定多重要素驗證的人員，看看他們是否可以協助您。

最後，請務必在此頁面上留下一些詳細的意見反應，讓我們可以更新此頁面，並藉由提供更多的資訊，持續讓它更加完善。

<!---HONumber=AcomDC_0921_2016-->