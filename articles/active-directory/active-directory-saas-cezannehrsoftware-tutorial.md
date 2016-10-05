<properties
	pageTitle="教學課程：Azure Active Directory 與 Cezanne HR Software 整合 | Microsoft Azure"
	description="了解如何設定 Azure Active Directory 與 Cezanne HR Software 之間的單一登入。"
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
	ms.date="09/15/2016"
	ms.author="jeedes"/>


# 教學課程：Azure Active Directory 與 Cezanne HR Software 整合

本教學課程旨在說明如何整合 Cezanne HR Software 與 Azure Active Directory (Azure AD)。

Cezanne HR Software 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Cezanne HR Software 的人員
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Cezanne HR Software (單一登入)
- 您可以在 Azure 傳統入口網站中集中管理您的帳戶

若您想了解 SaaS app 與 Azure AD 整合的更多詳細資訊，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](active-directory-appssoaccess-whatis.md)。

## 必要條件

若要設定 Azure AD 與 Cezanne HR Software 整合，您需要以下項目：

- Azure AD 訂用帳戶
- 已啟用 Cezanne HR Software 單一登入的訂用帳戶


> [AZURE.NOTE] 若要測試本教學課程中的步驟，我們不建議使用生產環境。


若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則您不應使用生產環境，。
- 如果您沒有 Azure AD 試用環境，您可以在[這裡](https://azure.microsoft.com/pricing/free-trial/)取得一個月試用。


## 案例描述
此教學課程的目標是讓您在測試環境中測試 Azure AD 單一登入。

本教學課程中說明的案例由二個主要建置組塊組成：

1. 從資源庫加入 Cezanne HR Software
2. 設定並測試 Azure AD 單一登入


## 從資源庫加入 Cezanne HR Software
若要設定 Cezanne HR Software 與 Azure AD 整合，您需要從資源庫將 Cezanne HR Software 新增到受管理的 SaaS 應用程式清單。

**若要從資源庫新增 Cezanne HR Software，請執行下列步驟：**

1. 在「Azure 傳統入口網站」中，按一下左方瀏覽窗格上的 [Active Directory]。

	![Active Directory][1]

2. 從 [目錄] 清單中，選取要啟用目錄整合的目錄。

3. 若要開啟應用程式檢視，請在目錄檢視中，按一下頂端功能表中的 [應用程式]。
	
	![應用程式][2]

4. 按一下頁面底部的 [新增]。
	
	![應用程式][3]

5. 在 [欲執行動作] 對話方塊中，按一下 [從資源庫加入應用程式]。

	![應用程式][4]

6. 在 [搜尋] 方塊中，輸入 **Cezanne HR Software**。

	![建立 Azure AD 測試使用者](./media/active-directory-saas-cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_01.png)

7. 在結果窗格中，選取 [Cezanne HR Software]，然後按一下 [完成] 以新增應用程式。

	![選取資源庫中的應用程式](./media/active-directory-saas-cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_0001.png)

##  設定並測試 Azure AD 單一登入
本節的目標是說明如何以名為 "Britta Simon" 的測試使用者為基礎，使用 Cezanne HR Software 來設定及測試 Azure AD 單一登入。

單一登入若要運作，Azure AD 必須知道 Cezanne HR Software 與 Azure AD 中互相對應的使用者。換句話說，Azure AD 使用者和 Cezanne HR Software 中的相關使用者之間必須建立連結關聯性。

建立此連結關聯性的方法是將 Azure AD 中的 **user name** 的值指定為 Cezanne HR Software 中 **Username** 的值。

若要使用 Cezanne HR Software 設定並測試 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-single-sign-on)** - 讓您的使用者能夠使用此功能。
2. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
3. **[建立 Cezanne HR Software 測試使用者](#creating-a-cezanne-hr-software-test-user)** - 使 Cezanne HR Software 中對應的 Britta Simon 連結到她在 Azure AD 中的代表項目。
4. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
5. **[測試單一登入](#testing-single-sign-on)** - 驗證組態是否能運作。

### 設定 Azure AD 單一登入

在本節中，您會在傳統入口網站中啟用 Azure AD 單一登入，並在您的 Cezanne HR Software 應用程式中設定單一登入。

**若要使用 Cezanne HR Software 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在傳統入口網站的 [Cezanne HR Software] 應用程式整合頁面上，按一下 [設定單一登入] 來開啟 [設定單一登入] 對話方塊。
	 
	![設定單一登入][6]

2. 在 [要如何讓使用者登入 Cezanne HR Software] 頁面上，選取 [Azure AD 單一登入]，然後按 [下一步]。
    
	![設定單一登入](./media/active-directory-saas-cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_03.png)

3. 在 [設定應用程式設定] 對話方塊頁面上，執行下列步驟，然後按 [下一步]：

    ![設定單一登入](./media/active-directory-saas-cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_04.png)

	a.在 [登入 URL] 文字方塊中，以下列模式輸入 URL︰`https://w3.cezanneondemand.com/cezannehr/-/<tenant id>`。

    b.在 [識別碼] 文字方塊中，輸入︰`https://w3.cezanneondemand.com/CezanneOnDemand/`。

	c.在 [回覆 URL] 文字方塊中，以下列模式輸入 URL：`https://w3.cezanneondemand.com:443/CezanneOnDemand/-/<tenant id>/Saml/samlp`。

	d.按 [**下一步**]

	> [AZURE.NOTE] 請注意，您必須使用實際的「登入 URL」、「識別碼」及「回覆 URL」來更新這些值。若要取得這些值，請透過 <mailto:info@cezannehr.com> 聯絡 Cezanne HR Software 支援小組。

4. 在 [設定在 Cezanne HR Software 單一登入] 頁面上，執行下列步驟，然後按 [下一步]：

	![設定單一登入](./media/active-directory-saas-cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_05.png)

    a.按一下 [下載憑證]，然後將檔案儲存在您的電腦上。

    b.按 [下一步]。

5. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入您的 Cezanne HR Software 租用戶。

6. 在左側的導覽窗格上，按一下 [系統設定]。移至 [安全性設定]。然後瀏覽至 [單一登入設定]。

	![在應用程式端設定單一登入](./media/active-directory-saas-cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_000.png)

7. 在 [允許使用者使用下列的單一登入 (SSO) 服務來登入] 面板中檢查 [SAML 2.0] 方塊，然後選取旁邊的 [進階組態] 選項。

	![在應用程式端設定單一登入](./media/active-directory-saas-cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_001.png)

8. 按一下 [新增] 按鈕。

	![在應用程式端設定單一登入](./media/active-directory-saas-cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_002.png)

9. 在 [SAML 2.0 身分識別提供者] 區段中執行下列步驟。

	![在應用程式端設定單一登入](./media/active-directory-saas-cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_003.png)

	a.輸入身分識別提供者名稱，做為**顯示名稱**。

	b.在 [實體識別碼] 文字方塊中，放入得自 Azure AD 應用程式組態精靈的 [實體 ID] 值。

	c.將 **SAML 繫結**變更為 'POST'。

	d.在 [Security Token Service 端點] 文字方塊中，放入來自 Azure AD 應用程式組態精靈的 [單一登入服務 URL] 值。

	e.在 [使用者 ID 屬性名稱] 中輸入 'http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name'。

	f.按一下 [上傳] 圖示來上傳從 Azure AD 下載的憑證。

	g.按一下 [確定] 按鈕。

10. 按一下 [儲存] 按鈕。

	![在應用程式端設定單一登入](./media/active-directory-saas-cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_004.png)

11. 在傳統入口網站中，選取單一登入設定確認，然後按 [下一步]。
    
	![Azure AD 單一登入][10]

12. 在 [單一登入確認] 頁面上，按一下 [完成]。
    
	![Azure AD 單一登入][11]



### 建立 Azure AD 測試使用者
本節的目標是要在傳統入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][20]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在「Azure 傳統入口網站」中，按一下左方瀏覽窗格上的 [Active Directory]。

    ![建立 Azure AD 測試使用者](./media/active-directory-saas-cezannehrsoftware-tutorial/create_aaduser_09.png)

2. 從 [目錄] 清單中，選取要啟用目錄整合的目錄。

3. 若要顯示使用者清單，請按一下頂端功能表中的 [使用者]。
    
	![建立 Azure AD 測試使用者](./media/active-directory-saas-cezannehrsoftware-tutorial/create_aaduser_03.png)

4. 若要開啟 [加入使用者] 對話方塊，請按一下底部工具列中的 [加入使用者]。

    ![建立 Azure AD 測試使用者](./media/active-directory-saas-cezannehrsoftware-tutorial/create_aaduser_04.png)

5. 在 [告訴我們這位使用者] 對話方塊頁面上，執行下列步驟：

    ![建立 Azure AD 測試使用者](./media/active-directory-saas-cezannehrsoftware-tutorial/create_aaduser_05.png)

    a.針對 [使用者類型]，選取 [您組織中的新使用者]。

    b.在 [使用者名稱] 文字方塊中，輸入 **BrittaSimon**。

    c.按 [下一步]。

6.  在 [使用者設定檔]對話方塊頁面上，執行下列步驟：
    
	![建立 Azure AD 測試使用者](./media/active-directory-saas-cezannehrsoftware-tutorial/create_aaduser_06.png)

    a.在 [名字] 文字方塊中，輸入 **Britta**。

    b.在 [姓氏] 文字方塊中，輸入 **Simon**。

    c.在 [顯示名稱] 文字方塊中，輸入 **Britta Simon**。

    d.在 [角色] 清單中選取 [使用者]。

    e.按 [下一步]。

7. 在 [取得暫時密碼] 對話方塊頁面上，按一下 [建立]。
    
	![建立 Azure AD 測試使用者](./media/active-directory-saas-cezannehrsoftware-tutorial/create_aaduser_07.png)

8. 在 [取得暫時密碼] 對話方塊頁面上，執行下列步驟：
    
	![建立 Azure AD 測試使用者](./media/active-directory-saas-cezannehrsoftware-tutorial/create_aaduser_08.png)

    a.記下 [新密碼] 的值。

    b.按一下 [完成]。



### 建立 Cezanne HR Software 測試使用者

為了讓 Azure AD 使用者能夠登入 Cezanne HR Software，必須將他們佈建到 Cezanne HR Software 中。Cezanne HR Software 需以手動的方式佈建。

####若要佈建使用者帳戶，請執行下列步驟：

1.  以系統管理員身分登入您的 Cezanne HR Software 公司網站。

2.  在左側的導覽窗格上，按一下 [系統設定]。移至 [管理使用者]。然後瀏覽至 [新增使用者]。

    ![新使用者](./media/active-directory-saas-cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_005.png "新使用者")

3.  在 [人員詳細資料] 區段中，執行下列步驟︰

    ![新使用者](./media/active-directory-saas-cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_006.png "新使用者")

	a.設定 [內部使用者] 為 OFF。

	b.在 [名字] 文字方塊中，輸入 **Britta**。

    c.在 [姓氏] 文字方塊中，輸入 **Simon**。

	d.在 [電子郵件] 文字方塊中，輸入 Britta Simon 帳戶的電子郵件地址。

4.  在 [帳戶資訊] 區段中，執行下列步驟︰

    ![新使用者](./media/active-directory-saas-cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_007.png "新使用者")

	a.在 [使用者名稱] 文字方塊中，輸入 Britta Simon 的電子郵件地址。

	b.在 [密碼] 文字方塊中，輸入 Britta Simon 帳戶的密碼。

	c.選取 [HR Professional] 為 [安全性角色]。

	d. 按一下 [確定]。

5. 瀏覽至 [單一登入] 索引標籤並選取 [SAML 2.0 識別碼] 區域中的 [新增]。

	![User](./media/active-directory-saas-cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_008.png "User")

6. 為 [身分識別提供者] 選擇身分識別提供者，並在 [使用者識別碼] 文字方塊中輸入 Britta Simon 帳戶的電子郵件地址。

	![User](./media/active-directory-saas-cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_009.png "User")
	
7. 按一下 [儲存] 按鈕。

	![User](./media/active-directory-saas-cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_010.png "User")


### 指派 Azure AD 測試使用者

本節的目標是授與 Britta Simon 對 Cezanne HR Software 的存取權，讓她能夠使用 Azure 單一登入。
	
![指派使用者][200]

**若要指派 Britta Simon 到 Cezanne HR Software，請執行以下步驟：**

1. 在傳統入口網站中，若要開啟應用程式檢視，請在目錄檢視中，按一下頂端功能表中的 [應用程式]。
    
	![指派使用者][201]

2. 在應用程式清單中，選取 [Cezanne HR Software]。
    
	![設定單一登入](./media/active-directory-saas-cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_50.png)

3. 在頂端的功能表中，按一下 [使用者]。
    
	![指派使用者][203]

4. 在 [使用者] 清單中，選取 [Britta Simon]。

5. 在底部的工具列中，按一下 [指派]。
    
	![指派使用者][205]

### 測試單一登入

本節的目標是要使用存取面板來測試您的 Azure AD 單一登入組態。
 
當您在 [存取面板] 中按一下 [Cezanne HR Software] 圖格時，您應該會自動登入您的 Cezanne HR Software 應用程式。

## 其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](active-directory-saas-tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](active-directory-appssoaccess-whatis.md)



<!--Image references-->

[1]: ./media/active-directory-saas-cezannehrsoftware-tutorial/tutorial_general_01.png
[2]: ./media/active-directory-saas-cezannehrsoftware-tutorial/tutorial_general_02.png
[3]: ./media/active-directory-saas-cezannehrsoftware-tutorial/tutorial_general_03.png
[4]: ./media/active-directory-saas-cezannehrsoftware-tutorial/tutorial_general_04.png

[6]: ./media/active-directory-saas-cezannehrsoftware-tutorial/tutorial_general_05.png
[10]: ./media/active-directory-saas-cezannehrsoftware-tutorial/tutorial_general_06.png
[11]: ./media/active-directory-saas-cezannehrsoftware-tutorial/tutorial_general_07.png
[20]: ./media/active-directory-saas-cezannehrsoftware-tutorial/tutorial_general_100.png

[200]: ./media/active-directory-saas-cezannehrsoftware-tutorial/tutorial_general_200.png
[201]: ./media/active-directory-saas-cezannehrsoftware-tutorial/tutorial_general_201.png
[203]: ./media/active-directory-saas-cezannehrsoftware-tutorial/tutorial_general_203.png
[204]: ./media/active-directory-saas-cezannehrsoftware-tutorial/tutorial_general_204.png
[205]: ./media/active-directory-saas-cezannehrsoftware-tutorial/tutorial_general_205.png

<!---HONumber=AcomDC_0921_2016-->