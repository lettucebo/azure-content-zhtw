<properties
	pageTitle="教學課程：Azure Active Directory 與 HackerOne 整合 | Microsoft Azure"
	description="了解如何設定 Azure Active Directory 與 HackerOne 之間的單一登入。"
	services="active-directory"
	documentationCenter=""
	authors="jeevansd"
	manager="femila"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="07/09/2016"
	ms.author="jeedes"/>


# 教學課程：Azure Active Directory 與 HackerOne 整合

在本教學課程中，您要整合 HackerOne 與 Azure Active Directory (Azure AD)。

HackerOne 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 HackerOne 的人員
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 HackerOne (單一登入)
- 您可以在 Azure 傳統入口網站中集中管理您的帳戶

若您想了解 SaaS app 與 Azure AD 整合的更多詳細資訊，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](active-directory-appssoaccess-whatis.md)。

## 必要條件

若要設定 Azure AD 與 HackerOne 整合，您需要下列項目：

- Azure 訂用帳戶
- 啟用 HackerOne 單一登入的訂用帳戶


> [AZURE.NOTE] 若要測試本教學課程中的步驟，我們不建議使用生產環境。


若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則您不應使用生產環境，。
- 如果您沒有 Azure AD 試用環境，您可以在[這裡](https://azure.microsoft.com/pricing/free-trial/)取得一個月試用。


## 案例描述
在本教學課程中，您會在測試環境中設定和測試 Azure AD 單一登入。本教學課程中說明的案例由二個主要建置組塊組成：

1. 從資源庫中加入 HackerOne
2. 設定並測試 Azure AD 單一登入


## 從資源庫中加入 HackerOne
若要將 HackerOne 整合到 Azure AD 中，您需要從資源庫將 HackerOne 加入受管理的 SaaS 應用程式清單中。

**若要從資源庫加入 HackerOne，請執行下列步驟：**

1. 在 **Azure 傳統入口網站**中，按一下左方瀏覽窗格的 [Active Directory]。

	![Active Directory][1]

2. 從 [目錄] 清單中，選取要啟用目錄整合的目錄。

3. 若要開啟應用程式檢視，請在目錄檢視中，按一下頂端功能表中的 [應用程式]。

	![應用程式][2]

4. 按一下頁面底部的 [新增]。

	![應用程式][3]

5. 在 [欲執行動作] 對話方塊中，按一下 [從資源庫中新增應用程式]。

![應用程式][4]

6. 在搜尋方塊中輸入 **HackerOne**。

![建立 Azure AD 測試使用者](./media/active-directory-saas-hackerone-tutorial/tutorial_hackerone_01.png)

7. 在結果窗格中選取 [HackerOne]，然後按一下 [完成] 加入應用程式。


##  設定並測試 Azure AD 單一登入
接下來，您要使用 "Britta Simon" 的測試使用者身分，以 HackerOne 設定及測試 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 HackerOne 與 Azure AD 中互相對應的使用者。換句話說，必須在 Azure AD 使用者和 HackerOne 中的相關使用者之間，建立連結關聯性。建立此連結關聯性的方法，就是將 Azure AD 中 [使用者名稱] 的值指定為 HackerOne 中 [Username] 的值。

若要以 HackerOne 設定及測試 Azure AD 單一登入，您需要完成下列構成要素：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-single-sign-on)** - 讓您的使用者能夠使用此功能。
2. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
3. **[建立 HackerOne 測試使用者](#creating-a-hackerone-test-user)** - 在 Certify 中建立一個 Britta Simon 對應項目，其要與 Azure AD 中代表她的項目連結。
4. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
5. **[測試單一登入](#testing-single-sign-on)** - 驗證組態是否能運作。

### 設定 Azure AD 單一登入

接下來，您會在傳統入口網站中啟用 Azure AD 單一登入，並在您的 HackerOne 中設定單一登入。

在此程序中，您必須建立 Base-64 編碼的憑證檔案。如果您不熟悉此程序，請參閱[如何將二進位憑證轉換成文字檔](http://youtu.be/PlgrzUZ-Y1o)。

**若要使用 HackerOne 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 傳統入口網站的 [HackerOne] 應用程式整合頁面上，按一下 [設定單一登入] 來開啟 [設定單一登入] 對話方塊。

	![設定單一登入][6]

2. 在 [要如何讓使用者登入 HackerOne] 頁面上，選取 [Azure AD 單一登入]，然後按一下 [下一步]。

	![設定單一登入](./media/active-directory-saas-hackerone-tutorial/tutorial_hackerone_03.png)

3. 在 [設定應用程式設定] 對話方塊頁面上，執行下列步驟，然後按一下 [下一步]：

	![設定單一登入](./media/active-directory-saas-hackerone-tutorial/tutorial_hackerone_04.png)


    a.在 [登入 URL] 文字方塊中，使用下列模式輸入使用者用來登入 HackerOne 應用程式的 URL：**“https://hackerone.com/<公司名稱>/authentication”**。

    b.如果您不知道租用戶 URL，請透過 [support@hackerone.com](mailto:support@hackerone.com) 連絡 HackerOne 支援小組來取得。

	c.在 [識別碼] 文字方塊中，輸入租用戶 URL。

	d.按 [下一步]。


4. 在 [設定在 HackerOne 單一登入] 頁面上，執行下列步驟，然後按一下 [下一步]：

	![設定單一登入](./media/active-directory-saas-hackerone-tutorial/tutorial_hackerone_05.png)

    a.按一下 [下載憑證]，然後將檔案儲存在您的電腦上。

    b.按 [下一步]。


1. 以系統管理員身分登入 HackerOne 租用戶。

1. 在頂端的功能表中，按一下 [設定]。

	![設定單一登入](./media/active-directory-saas-hackerone-tutorial/tutorial_hackerone_001.png)

1. 瀏覽至 [驗證] 並按一下 [Add SAML settings] \(加入 SAML 設定)。

	![設定單一登入](./media/active-directory-saas-hackerone-tutorial/tutorial_hackerone_003.png)


1. 在 [SAML Settings] \(SAML設定) 頁面上，執行下列步驟：

	![設定單一登入](./media/active-directory-saas-hackerone-tutorial/tutorial_hackerone_004.png)

    a.在 [電子郵件網域] 文字方塊中，輸入註冊的網域。

	b.在 Azure 傳統入口網站上，複製 [單一登入服務 URL]，然後將它貼至 [單一登入 URL] 文字方塊中。

    c.從您下載的憑證建立「Base-64 編碼」檔案。

       >[AZURE.TIP] 如需詳細資訊，請參閱[如何將二進位憑證轉換成文字檔](http://youtu.be/PlgrzUZ-Y1o)。
	
    d.在記事本中開啟 Base-64 編碼的憑證，將其內容複製到剪貼簿，然後貼到 [X509 憑證] 文字方塊。

    e.按一下 [儲存]


1. 在 [驗證設定] 對話方塊上，執行下列步驟：

	![設定單一登入](./media/active-directory-saas-hackerone-tutorial/tutorial_hackerone_005.png)

    a.按一下 [執行測試]。

    b.如果 [狀態] 欄位的值等於 [上次測試狀態: 已建立]，請透過 [support@hackerone.com](mailto:support@hackerone.com) 連絡您的 HackerOne 支援小組，要求檢閱您的設定。


6. 在 Azure 傳統入口網站中，選取單一登入設定確認，然後按一下 [下一步]。

	![Azure AD 單一登入][10]

7. 在 [單一登入確認] 頁面上，按一下 [完成]。
 
	![Azure AD 單一登入][11]




### 建立 Azure AD 測試使用者

接下來，您會在傳統入口網站中建立名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][20]

**若要在 Azure AD 中建立 SECURE DELIVER 測試使用者，請執行下列步驟：**

1. 在 **Azure 傳統入口網站**中，按一下左方瀏覽窗格的 [Active Directory]。

	![建立 Azure AD 測試使用者](./media/active-directory-saas-hackerone-tutorial/create_aaduser_09.png)

2. 從 [目錄] 清單中，選取要啟用目錄整合的目錄。

3. 若要顯示使用者清單，請在頂端的功能表中，按一下 [使用者]。

	![建立 Azure AD 測試使用者](./media/active-directory-saas-hackerone-tutorial/create_aaduser_03.png)

4. 若要開啟 [加入使用者] 對話方塊，請按一下底部工具列上的 [加入使用者]。![建立 Azure AD 測試使用者](./media/active-directory-saas-hackerone-tutorial/create_aaduser_04.png)

5. 在 [告訴我們這位使用者] 對話方塊頁面上，執行下列步驟：

	![建立 Azure AD 測試使用者](./media/active-directory-saas-hackerone-tutorial/create_aaduser_05.png)

    a.針對 [使用者類型]，選取 [您組織中的新使用者]。

    b.在 [使用者名稱] 文字方塊中，輸入 **BrittaSimon**。

    c.按 [下一步]。

6.  在 [使用者設定檔] 對話方塊頁面上，執行下列步驟：

	![建立 Azure AD 測試使用者](./media/active-directory-saas-hackerone-tutorial/create_aaduser_06.png)

    a.在 [名字] 文字方塊中，輸入 **Britta**。

    b.在 [姓氏] 文字方塊中，輸入 **Simon**。

    c.在 [顯示名稱] 文字方塊中，輸入 **Britta Simon**。

    d.在 [角色] 清單中選取 [使用者]。

    e.按 [下一步]。

7. 在 [取得暫時密碼] 對話方塊頁面上，按一下 [建立]。

	![建立 Azure AD 測試使用者](./media/active-directory-saas-hackerone-tutorial/create_aaduser_07.png)

8. 在 [取得暫時密碼] 對話方塊頁面上，執行下列步驟：

	![建立 Azure AD 測試使用者](./media/active-directory-saas-hackerone-tutorial/create_aaduser_08.png)

    a.記下 [新密碼] 的值。

    b.按一下 [完成]。


### 建立 HackerOne 測試使用者

接下來，您要在 HackerOne 建立名為 Britta Simon 的使用者。HackerOne 支援預設啟用的 Just-In-Time 佈建。

在這一節沒有您需要進行的動作項目。當您存取 HackerOne 時，如果還沒有新的使用者，就會建立一個。[設定 Azure AD 單一登入](#configuring-azure-ad-single-single-sign-on)。

> [AZURE.NOTE] 如果您需要手動建立使用者，您需要連絡 Certify 支援小組。




### 指派 Azure AD 測試使用者

接下來，您要把 HackerOne 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200]

**若要將 Britta Simon 指派給 HackerOne，請執行下列步驟：**

1. 在 Azure 傳統入口網站中，若要開啟應用程式檢視，請在目錄檢視中，按一下頂端功能表中的 [應用程式]。

	![指派使用者][201]

2. 在應用程式清單中，選取 [HackerOne]。

	![設定單一登入](./media/active-directory-saas-hackerone-tutorial/tutorial_hackerone_50.png)

1. 在頂端的功能表中，按一下 [使用者]。

	![指派使用者][203]

1. 在 [使用者] 清單中，選取 [Britta Simon]。

2. 在底部的工具列中，按一下 [指派]。

	![指派使用者][205]



### 測試單一登入

最後，您會使用存取面板來測試您的 Azure AD 單一登入設定。當您在存取面板中按一下 [HackerOne] 圖格時，應該會自動登入您的 HackerOne 應用程式。


## 其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](active-directory-saas-tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](active-directory-appssoaccess-whatis.md)


<!--Image references-->

[1]: ./media/active-directory-saas-hackerone-tutorial/tutorial_general_01.png
[2]: ./media/active-directory-saas-hackerone-tutorial/tutorial_general_02.png
[3]: ./media/active-directory-saas-hackerone-tutorial/tutorial_general_03.png
[4]: ./media/active-directory-saas-hackerone-tutorial/tutorial_general_04.png

[6]: ./media/active-directory-saas-hackerone-tutorial/tutorial_general_05.png
[10]: ./media/active-directory-saas-hackerone-tutorial/tutorial_general_06.png
[11]: ./media/active-directory-saas-hackerone-tutorial/tutorial_general_07.png
[20]: ./media/active-directory-saas-hackerone-tutorial/tutorial_general_100.png

[200]: ./media/active-directory-saas-hackerone-tutorial/tutorial_general_200.png
[201]: ./media/active-directory-saas-hackerone-tutorial/tutorial_general_201.png
[203]: ./media/active-directory-saas-hackerone-tutorial/tutorial_general_203.png
[204]: ./media/active-directory-saas-hackerone-tutorial/tutorial_general_204.png
[205]: ./media/active-directory-saas-hackerone-tutorial/tutorial_general_205.png

<!---HONumber=AcomDC_0713_2016-->