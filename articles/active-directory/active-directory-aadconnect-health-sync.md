
<properties
	pageTitle="使用 Azure AD Connect Health 進行同步處理 | Microsoft Azure"
	description="這是 Azure AD Connect Health 頁面，其中討論如何監視 Azure AD Connect 同步處理。"
	services="active-directory"
	documentationCenter=""
	authors="billmath"
	manager="femila"
	editor="curtand"/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="get-started-article"
	ms.date="08/08/2016"
	ms.author="billmath"/>

# 使用適用於同步處理的 Azure AD Connect Health
下列文件適用於使用 Azure AD Connect Health 來監視 Azure AD Connect (同步處理)。如需使用 Azure AD Connect Health 來監視 AD FS 的詳細資訊，請參閱[在 AD FS 使用 Azure AD Connect Health](active-directory-aadconnect-health-adfs.md)。此外，如需使用 Azure AD Connect Health 來監視 Active Directory 網域服務的詳細資訊，請參閱 [在 AD DS 使用 Azure AD Connect Health](active-directory-aadconnect-health-adds.md)。

![適用於同步處理的 Azure AD Connect Health](./media/active-directory-aadconnect-health-sync/sync.png)

## 適用於同步處理的 Azure AD Connect Health 警示
＜適用於同步處理的 Azure AD Connect Health 警示＞小節將為您提供作用中警示的清單。每個警示都包含相關資訊、解決步驟，以及相關文件的連結。選取作用中或已解決的警示，您將會看到一個包含額外資訊的新刀鋒視窗，以及解決警示可以採取的步驟，和其他文件的連結。您也可以檢視過去已解決的警示的歷史資料。

選取警示，將會為您提供其他資訊，以及解決警示可以採取的步驟，和其他文件的連結。

![Azure AD Connect 同步處理錯誤](./media/active-directory-aadconnect-health-sync/alert.png)

### 有限的警示評估
如果 Azure AD Connect 不使用預設組態 (比方說，如果 [屬性篩選] 從預設組態變更為自訂組態)，則 Azure AD Connect Health 代理程式不會上傳 Azure AD Connect 相關的錯誤事件。

這會限制服務的警示評估。您應會在 Azure 入口網站中您的服務之下，看到指出這種情況的橫幅。

![適用於同步處理的 Azure AD Connect Health](./media/active-directory-aadconnect-health-sync/banner.png)

您可以按一下 [設定] 並允許 Azure AD Connect Health 代理程式上傳所有的錯誤記錄檔，加以變更。

![適用於同步處理的 Azure AD Connect Health](./media/active-directory-aadconnect-health-sync/banner2.png)

## 同步處理深入了解
藉由適用於同步處理的 Azure AD Connect Health 的最新版本，新增了下列新功能：

- 同步處理作業的延遲
- 物件變更趨勢

### 同步處理延遲
這項功能提供連接器同步處理作業 (匯入、匯出等) 延遲的圖形化趨勢。這提供快速且輕鬆的方式，讓您了解不僅僅作業的延遲 (如果您會發生大量變更也很好)，還提供一個方式來偵測延遲中可能需要進一步調查的異常行為。

![同步處理延遲](./media/active-directory-aadconnect-health-sync/synclatency.png)

根據預設，只會顯示 Azure AD 連接器「匯出」作業的延遲。若要查看連接器上的更多作業，或檢視其他連接器的作業，請以滑鼠右鍵按一下圖表並選擇特定作業和連接器。

### 同步處理物件的變更
這項功能提供正在評估並匯出至 Azure AD 的變更數的圖形化趨勢。現在，嘗試從同步處理記錄檔收集此資訊並不容易。圖表不僅可讓您以更簡單的方式監視您的環境中發生的變更數，同時提供正在發生的失敗的視覺化檢視。

![同步處理延遲](./media/active-directory-aadconnect-health-sync/syncobjectchanges.png)

## 相關連結

* [Azure AD Connect Health](active-directory-aadconnect-health.md)
* [Azure AD Connect Health 代理程式安裝](active-directory-aadconnect-health-agent-install.md)
* [Azure AD Connect Health 操作](active-directory-aadconnect-health-operations.md)
* [使用 Azure AD Connect Health 來搭配 AD FS](active-directory-aadconnect-health-adfs.md)
* [在 AD DS 使用 Azure AD Connect Health](active-directory-aadconnect-health-adds.md)
* [Azure AD Connect Health 常見問題集](active-directory-aadconnect-health-faq.md)
* [Azure AD Connect Health 版本歷程記錄](active-directory-aadconnect-health-version-history.md)

<!---HONumber=AcomDC_0928_2016-->