<properties
	pageTitle="在登入和存取面板頁面加上公司商標"
	description="了解如何將公司商標新增至 Azure 登入頁面和存取面板頁面"
	services="active-directory"
	documentationCenter=""
	authors="curtand"
	manager="femila"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="get-started-article"
	ms.date="09/22/2016"
	ms.author="curtand"/>

# 在登入和存取面板頁面加上公司商標


為了避免混淆，許多公司都想對其管理的所有網站和服務套用一致的外觀與風格。Azure Active Directory 提供這項功能，讓您利用公司標誌和自訂色彩配置來自訂下列網頁的外觀：

- **登入頁面** - 當您登入 Office 365 或其他使用 Azure AD 作為識別提供者的 Web 型應用程式時，便會出現此頁面。在進行主領域探索期間或要輸入認證時，您就會與此頁面互動。主領域探索可讓系統將同盟使用者重新導向至其內部部署 STS (例如 AD FS)。

- **存取面板頁面** - 存取面板是網頁型入口網站，可讓您檢視並啟動 Azure AD 系統管理員已授與您存取權的雲端式應用程式。若要存取「存取面板」，請使用下列 URL：[https://myapps.microsoft.com](https://myapps.microsoft.com)。

本主題說明如何自訂登入頁面和存取面板頁面。

> [AZURE.NOTE]
>
- 公司商標是您升級至 Premium 或 Basic 版本的 Azure Active Directory 後，或是 Office 365 使用者時才能使用的功能。如需詳細資訊，請參閱 [Azure Active Directory 版本](active-directory-editions.md)。
- Azure Active Directory Premium 和 Basic 版本適用於使用全球 Azure Active Directory 執行個體的中國客戶。由 21Vianet 在中國提供的 Microsoft Azure 服務目前不支援 Azure Active Directory Premium 和 Basic 版本。如需詳細資訊，請透過 [Azure Active Directory 論壇](https://feedback.azure.com/forums/169401-azure-active-directory/)與我們連絡。



## 自訂登入頁面

一般而言，如果您需要透過瀏覽器存取貴組織訂閱的雲端應用程式和服務，您可使用登入頁面。

如果您已對登入頁面套用變更，最多需要一小時變更才會出現。

只有當您使用租用戶特定 URL (例如 https://outlook.com/**contoso**.com 或 https://mail.**contoso**.com) 來造訪服務時，才會顯示加上商標的登入頁面。

當您使用非租用戶特定 URL (例如 https://mail.office365.com) 造訪服務時，則會出現沒有商標的登入頁面。在此情況下，在您輸入使用者識別碼或選取使用者圖格之後，您的商標就會出現。

> [AZURE.NOTE]
>
- 在已設定商標之 Azure 傳統入口網站的 [Active Directory] > [目錄] > [網域] 區段中，您的網域名稱必須為 [作用中]。
- 登入頁面商標不會延續到 Microsoft 的消費者登入頁面。如果您使用個人 Microsoft 帳戶進行登入，可能會看到 Azure AD 所呈現並加上商標的使用者圖格清單，但是您組織的商標不會套用到 Microsoft 帳戶登入頁面。


如果您想要在此頁面上顯示您的公司商標、色彩和其他可自訂的元素，請參閱下列影像以了解這兩種做法的差異。

下列螢幕擷取畫面顯示桌上型電腦上 Office 365 登入頁面的自訂「前」範例︰

![自訂前的 Office 365 登入頁面][1]

下列螢幕擷取畫面顯示桌上型電腦上 Office 365 登入頁面的自訂「後」範例︰

![自訂後的 Office 365 登入頁面][2]

下列螢幕擷取畫面顯示自訂**前**在行動裝置上的 Office 365 登入頁面範例︰

![自訂前的 Office 365 登入頁面][3]


下列螢幕擷取畫面顯示自訂**後**在行動裝置上的 Office 365 登入頁面範例︰

![自訂後的 Office 365 登入頁面][4]


當您調整瀏覽器視窗大小時，大型圖例 (例如先前所示的圖例) 通常會裁剪成符合不同的螢幕外觀比例。請記住，您應該嘗試保持圖例中的主要視覺元素，讓它們永遠顯示在左上角 (從右至左的語言，則顯示在右上角)。這十分重要，因為調整大小通常會從右下角往上/往左，或從下方往上方。

下圖顯示將瀏覽器調整到左方時如何裁剪圖例：

![][6]

以下是將瀏覽器調整到上方之後，如何顯示圖例：

![][7]

## 我們可以自訂頁面上的哪些元素？

您可以自訂登入頁面上的下列元素：

![][5]

 頁面元素 | 頁面上的位置
	------------- | -------------
橫幅標誌 | 顯示於頁面的右上方。取代您所登入之目的地網站要顯示的標誌 (例如 Office 365 或 Azure)。
大型圖例/背景色彩 | 顯示於頁面的左方。取代您所登入之目的地網站要顯示的影像。可能會顯示「背景色彩」，來替代低頻寬連線或窄畫面上的「大型圖例」。
登入頁面文字 | 在使用工作或學校帳戶登入之前需要傳達有用資訊時，會顯示於頁尾上方。例如，您可能想要包括支援人員的電話號碼或法律聲明。

> [AZURE.NOTE]
所有元素都是選用的。例如，如果您指定 [橫幅標誌]，但未指定 [大型圖例]，則登入頁面會顯示您的標誌以及目的地網站的圖例 (即 Office 365 加州高速公路影像)。

您也可以將此頁面上的所有元素都翻成當地使用語。設定一組「預設」自訂元素之後，就可以設定不同地區設定的其他版本。您也可以混合使用並符合各種元素。例如，您可以：

- 建立適用於所有文化的「預設」大型圖例，然後建立英文和法文的特定版本。當您將瀏覽器設定為這兩種語言之一時，會出現特定的影像，至於其他所有語言則會出現預設圖例。

- 為您的組織設定不同的標誌 (例如日文或希伯來文版本)。



## 存取面板頁面自訂

[存取面板] 頁面基本上是入口網站頁面，可供快速存取系統管理員已授與您存取權的雲端應用程式。在此頁面上，您的應用程式會顯示為可點選的應用程式圖格。


下列螢幕擷取畫面顯示自訂後的存取面板頁面範例：

![][8]

## 使用公司商標來設定目錄

您可以針對 Azure 傳統入口網站中的每個目錄，設定一組預設的可自訂元素。儲存預設值之後，系統管理員可以針對不同的語言/地區設定新增每個元素的當地語系化版本。所有可自訂元素都是選用的。

例如，如果您設定預設 [橫幅標誌]，但未設定 [大型圖例]，則登入頁面會將您的標誌顯示在右上角。不過，會顯示網站的預設圖例。

假設有下列組態︰

- 預設的橫幅標誌和英文登入頁面文字
- 語言特定的德文登入頁面文字

如果您的語言喜好設定是德文，您會得到預設的橫幅標誌但為德文文字。

技術上，雖然您可以針對 Azure AD 所支援的每種語言設定不同的一組，但是基於維護和效能考量，建議您保持低變化數目。

**若要將公司商標新增至您的目錄，請執行下列步驟：**

1. 以想要自訂之目錄的管理員身分，登入 [Azure 傳統入口網站](https://manage.windowsazure.com)。
2. 選取您想要自訂的目錄。
3. 在頂端的工具列中，按一下 [設定]。
4. 按一下 [自訂商標]。
4. 修改您想要自訂的元素。所有欄位都是選用的。
5. 按一下 [儲存]。

您對登入頁面商標進行的新變更最多需要一個小時才會出現。

**若要新增語言特定公司商標，請執行下列步驟：**

1. 以想要自訂之目錄的管理員身分，登入 [Azure 傳統入口網站](https://manage.windowsazure.com)。
2. 選取您想要自訂的目錄。
3. 在頂端的工具列中，按一下 [設定]。
4. 按一下 [自訂商標]。
2. 按一下 [新增特定語言的商標]。
3. 選取您要自訂標誌的語言，然後按 [下一步]。
3. 僅編輯您想要設定語言特定覆寫的元素。所有欄位都是選用的。如果欄位空白，則會改為顯示自訂預設值 (或者，如果未設定自訂預設值，則為 Microsoft 預設值)。
4. 按一下 [儲存]。

**若要從您的目錄中移除公司商標，請執行下列步驟：**

1. 以想要自訂之目錄的管理員身分，登入 [Azure 傳統入口網站](https://manage.windowsazure.com)。
2. 選取您想要自訂的目錄。
3. 在頂端的工具列中，按一下 [設定]。
4. 按一下 [自訂商標]。
5. 在 [自訂商標] 頁面上，選取 [編輯現有商標設定]，然後移至下一頁。
3. 根據您想要移除的元素，執行下列一或多項動作：

	a.在 [橫幅標誌] 之下，選取 [移除上傳的標誌]。

    b.在 [圖格標誌] 之下，選取 [移除上傳的標誌]。

    c.移除所有文字方塊中的文字。

    d.按 [下一步]。

    e.移除所有文字方塊中的文字。

4. 按一下 [儲存] 移除元素。
5. 如有必要，請再按一下 [自訂商標]，並針對需要移除的所有語言特定商標重複這些步驟。按一下 [自訂商標] 並看到未設定現有設定的 [自訂預設商標] 表單時，已移除所有商標設定。

## 測試和範例

建議您先使用測試租用戶進行試驗，再於生產環境中進行變更。

**若要確認商標是否已套用︰**

1. 開啟 InPrivate 或 Incognito 瀏覽器工作階段。
2. 瀏覽 https://outlook.com/contoso.com，並將 contoso.com 取代為您自訂的網域。

這也適用於類似 contoso.onmicrosoft.com 的網域。

為了協助您建立有效自訂集，我們已自訂下列兩個虛構的登入頁面：

- [http://aka.ms/aaddemo001](http://aka.ms/aaddemo001)
- [http://aka.ms/aaddemo002](http://aka.ms/aaddemo002)

若要測試語言特定設定，您需要將網頁瀏覽器中的預設語言喜好設定修改為已在自訂中所設定的語言。在 Internet Explorer 中，您可在 [網際網路選項] 功能表中進行此設定。

## 可自訂元素

Azure AD 中的部分可自訂元素有多個使用案例。您可以在每個目錄設定一次公司標誌並同時用於 [登入] 頁面和 [存取面板] 頁面。有些可自訂元素為登入頁面所特有。下表提供不同可自訂元素的詳細資料。

名稱 | 說明 | 條件約束 | 建議
	------------- | ------------- | ------------- | -------------
橫幅標誌 | [橫幅標誌] 會顯示在 [登入] 頁面和 [存取面板] 上。 | <p>JPG 或 PNG</p><p>60x280 像素</p><p>10 KB</p> | <p>使用您組織的完整標誌 (包含 pictogram 和 logotype)</p><p>高度保持低於 30 個像素，避免行動裝置上出現捲軸</p><p>保持低於 4 KB</p><p>使用透明 PNG (不假設 [登入] 頁面的背景永遠是白色)</p>
磚標誌 | (目前未用於 [登入] 頁面中) 未來，這段文字可能用來取代不同體驗位置中的泛用「工作或學校帳戶」pictogram。 | <p>JPG 或 PNG</p><p>120x120 像素</p><p>10 KB</p> | <p>保持簡單 (無小型文字)，因為此影像大小可能會調整為 50%
</p> |
登入頁面使用者名稱標籤 | (目前未用於 [登入] 頁面中) 未來，這段文字可能用來取代不同體驗位置中的泛用「工作或學校帳戶」字串。您可以將它設定為「Contoso 帳戶」或「Contoso 識別碼」這類項目。 | <p>Unicode 文字，最多 50 個字元</p><p>僅純文字 (沒有連結或 HTML 標記)</p> | <p>保持簡短和簡單</p><p>詢問使用者通常如何參照您提供給他們的工作或學校帳戶。</p>
登入頁面文字 | 此「重複使用」文字會出現在 [登入] 頁面表單下方，並且可以用來傳達其他指示或可在何處取得說明和支援。 | <p>Unicode 文字，最多 256 個字元</p><p>僅純文字 (沒有連結或 HTML 標記)</p> | 保持低於 250 個字元 (約 3 行文字)
登入頁面圖例 | 圖例是顯示在 [登入] 頁面表單左邊之 [登入] 頁面中的大型影像。 | <p>JPG 或 PNG</p><p>1420x1200</p><p>500 KB</p> | <p>1420x1200 像素</p><p>重要事項：保持越小越好，最好低於 200 KB。如果此影像太大，則會在未快取影像時影響 [登入] 頁面的效能</p><p>此影像通常會進行剪裁，以符合不同的螢幕外觀比例。將主要視覺元素保持在左上角 (RTL 語言，則顯示在右上角)，因為隨著瀏覽器視窗的縮小，調整大小會從右下角往左上方。</p>
登入頁面背景色彩 | 登入頁面背景色彩用於 [登入] 頁面表單左方的區域。 | 必須是十六進位格式的 RGB 色彩 (範例: #FFFFFF) | <p>可能會顯示背景色彩，來替代低頻寬連線上的「大型圖例」</p><p>建議挑選 [橫幅標誌] 的主要色彩</p>


## 後續步驟

- [開始使用 Azure Active Directory Premium](active-directory-get-started-premium.md)
- [檢視存取和使用情況報告](active-directory-view-access-usage-reports.md)

<!--Image references-->
[1]: ./media/active-directory-add-company-branding/SignInPage_beforecustomization.png
[2]: ./media/active-directory-add-company-branding/SignInPage_aftercustomization.png
[3]: ./media/active-directory-add-company-branding/SignInPage_mobile_beforecustomization.png
[4]: ./media/active-directory-add-company-branding/SignInPage_mobile_aftercustomization.png
[5]: ./media/active-directory-add-company-branding/SignInPage_aftercustomization_elements.png
[6]: ./media/active-directory-add-company-branding/SignInPage_aftercustomization_croppedleft.png
[7]: ./media/active-directory-add-company-branding/SignInPage_aftercustomization_croppedtop.png
[8]: ./media/active-directory-add-company-branding/APBranding.png

<!---HONumber=AcomDC_0928_2016-->