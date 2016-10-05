<properties 
	pageTitle="Microsoft Azure Multi-Factor Authentication 使用者狀態"
	description="了解 Azure MFA 中的使用者狀態"
	services="multi-factor-authentication"
	documentationCenter=""
	authors="kgremban"
	manager="femila"
	editor="curtand"/>

<tags
	ms.service="multi-factor-authentication"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/04/2016"
	ms.author="kgremban"/>

# Azure Multi-Factor Authentication 中的使用者狀態

Azure Multi-Factor Authentication 中的使用者帳戶具有下列三種不同狀態：

State | 說明 |受影響的非瀏覽器應用程式| 注意事項
:-------------: | :-------------: |:-------------: |:-------------: |
已停用 | 未註冊 Multi-Factor Authentication 之新使用者的預設狀態。|否|使用者未使用 Multi-Factor Authentication。
已啟用 |使用者已註冊 Multi-Factor Authentication。|無。它們會繼續運作，直到註冊程序完成為止。|使用者已啟用，但尚未完成註冊程序。系統會提示他們在下次登入時完成程序。
已強制|使用者已註冊，並且已完成使用 Multi-Factor Authentication 的註冊程序。|是。應用程式需要應用程式密碼。 | 使用者可能已完成或可能尚未完成註冊。如果他們已完成註冊程序，表示他們正在使用 Multi-Factor Authentication。否則，系統會提示使用者在下次登入時完成程序。

## 變更使用者狀態
使用者狀態變更取決於其是否已針對 MFA 進行設定，以及使用者是否已完成整個程序。當您為使用者開啟 MFA，使用者的狀態會從停用變更為啟用。一旦狀態已變更為啟用的使用者登入並完成了程序，其狀態將會變更為強制。

### 檢視使用者的狀態
--------------------------------------------------------------------------------
1.  以系統管理員身分登入 **Azure 傳統入口網站**。
2.  在左側按一下 [Active Directory]。
3.  在 [目錄] 下方，針對要啟用的使用者按一下目錄。![按一下目錄](./media/multi-factor-authentication-get-started-cloud/directory1.png)
4.  在頂端按一下 [使用者]。
5.  在頁面底部，按一下 [管理 Multi-Factor Auth]。![按一下目錄](./media/multi-factor-authentication-get-started-cloud/manage1.png)
6.  這會開啟新的瀏覽器索引標籤。您將能檢視使用者狀態。![按一下目錄](./media/multi-factor-authentication-get-started-user-states/userstate1.png)

###若要將狀態從停用變更為啟用
1.  以系統管理員身分登入 **Azure 傳統入口網站**。
2.  在左側按一下 [Active Directory]。
3.  在 [目錄] 下方，針對要啟用的使用者按一下目錄。![按一下目錄](./media/multi-factor-authentication-get-started-cloud/directory1.png)
4.  在頂端按一下 [使用者]。
5.  在頁面底部，按一下 [管理 Multi-Factor Auth]。![按一下目錄](./media/multi-factor-authentication-get-started-cloud/manage1.png)
6.  這會開啟新的瀏覽器索引標籤。找出要啟用 Multi-Factor Authentication 的使用者。您可能需要在頂端變更檢視。請確定狀態為 [停用]。![啟用使用者](./media/multi-factor-authentication-get-started-cloud/enable1.png)
7.  請在其名稱旁的方塊**打勾**。
7.  在右側按一下 [啟用]。![啟用使用者](./media/multi-factor-authentication-get-started-cloud/user1.png)
8.  按一下 [啟用 Multi-Factor Auth]。![啟用使用者](./media/multi-factor-authentication-get-started-cloud/enable2.png)
9.  您應該注意到使用者的狀態已經從 [停用] 變更為 [啟用]。![啟用使用者](./media/multi-factor-authentication-get-started-cloud/user.png)
10.  在您啟用您的使用者後，建議您透過電子郵件通知他們。電子郵件也應該通知他們如何使用其非瀏覽器應用程式，以避免遭到鎖定。

### 若要將狀態從啟用/強制變更為停用
1.  以系統管理員身分登入 **Azure 傳統入口網站**。
2.  在左側按一下 [Active Directory]。
3.  在 [目錄] 下方，針對要啟用的使用者按一下目錄。![按一下目錄](./media/multi-factor-authentication-get-started-cloud/directory1.png)
4.  在頂端按一下 [使用者]。
5.  在頁面底部，按一下 [管理 Multi-Factor Auth]。![按一下目錄](./media/multi-factor-authentication-get-started-cloud/manage1.png)
6.  這會開啟新的瀏覽器索引標籤。找出您想要設為停用的使用者。您可能需要在頂端變更檢視。請確定狀態為 [停用] 或 [強制]。
7.  請在其名稱旁的方塊**打勾**。
7.  在右側按一下 [停用]。![停用使用者](./media/multi-factor-authentication-get-started-user-states/userstate2.png)
8.  系統將提示您確認此動作。按一下 [是]。![停用使用者](./media/multi-factor-authentication-get-started-user-states/userstate3.png)
9.  接著您應該會看到該作業已經成功。按一下 [關閉]。![停用使用者](./media/multi-factor-authentication-get-started-user-states/userstate4.png)

<!---HONumber=AcomDC_0921_2016-->