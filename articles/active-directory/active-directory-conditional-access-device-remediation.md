<properties
	pageTitle="疑難排解︰您無法從這裡完成 | Microsoft Azure"
	description="本主題可協助您找出可供依循的補救步驟來取得應用程式的存取權。"
	services="active-directory"
	keywords="裝置型條件式存取、裝置註冊、啟用裝置註冊、裝置註冊和 MDM"
	documentationCenter=""
	authors="markusvi"
	manager="femila"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="get-started-article"
	ms.date="08/23/2016"
	ms.author="markvi"/>


# 疑難排解︰您無法從這裡完成

您在存取應用程式 (例如 SharePoint Online) 時遭到拒絕存取。現在，該怎麼辦？

本指南可協助您找出可供依循的補救步驟來取得應用程式的存取權。



您的裝置在哪個裝置平台上執行？ 此問題的答案可為您判斷本主題中的適當區段︰


-	Windows 裝置
-	iOS 裝置 (iPhone 或 iPad)
-	Android 裝置

## 從 Windows 裝置存取

如果您的裝置執行 Windows 10、Windows 8.1、Windows 8.0、Windows 7、Windows Server 2016、Windows Server 2012 R2、Windows Server 2012 或 Windows Server 2008 R2，請識別您在嘗試存取應用程式時所到達的頁面以選擇適當的原因。

### 裝置未註冊

如果未向 Azure Active Directory (Azure AD) 應用程式註冊您的裝置且應用程式受裝置原則保護，您可能會看到含有下列內容的頁面︰

![未註冊的裝置的「您無法從這裡完成」訊息](./media/active-directory-conditional-access-device-remediation/01.png "案例")



如果您的裝置已加入您組織中的 Active Directory 網域，您可以嘗試下列作業︰

1.	確定您已使用工作帳戶 (Active Directory 帳戶) 登入 Windows。
2.	透過 VPN 或 DirectAccess 連接到公司網路。
3.	連線後，請使用 Windows 鍵 + L 鍵來鎖定您的 Windows 工作階段。
4.	輸入您的工作帳戶認證，以解除鎖定 Windows 工作階段。
5.	等候一分鐘，然後再次嘗試存取應用程式。
6.	如果您看到相同的頁面，請連絡您的系統管理員，按一下 [更多詳細資料] 連結，然後提供詳細資料。

如果您的裝置未加入網域並執行 Windows 10，您有兩個選項︰

- 執行 Azure AD Join。
- 將您的工作或學校帳戶新增至 Windows。

如需有關兩者之間差異的資訊，請參閱[在您的工作場所中使用 Windows 10 裝置](active-directory-azureadjoin-windows10-devices.md)。

若要執行 Azure AD Join，請執行下列步驟 (不適用於 Windows Phone)︰

**Windows 10 年度更新版**

1.	開啟 [Settings] 應用程式。
2.	按一下 [帳戶] > [取工作或學校]。
3.	按一下 [連接]。
4.	按一下頁面底部的 [將此裝置加入至 Azure AD]。
5.	向您的組織驗證、視需要提供 Multi-Factor Authentication 證明，然後遵循相關步驟，直到完成為止。
6.	登出，然後使用您的工作帳戶登入。
7.	再次嘗試存取應用程式。




**Windows 10 2015 年 11 月更新**


1.	開啟 [Settings] 應用程式。
2.	按一下 [系統] > [關於]。
3.	按一下 [加入 Azure AD]。
4.	向您的組織驗證、視需要提供 Multi-Factor Authentication 證明，然後遵循相關步驟，直到完成為止。
5.	登出，然後使用您的工作帳戶 (Azure AD 帳戶) 登入。
6.	再次嘗試存取應用程式。

若要新增您的工作或學校帳戶，請執行下列步驟︰

**Windows 10 年度更新版**

1.	開啟 [Settings] 應用程式。
2.	按一下 [帳戶] > [取工作或學校]。
3.	按一下 [連接]。
4.	向您的組織驗證、視需要提供 Multi-Factor Authentication 證明，然後遵循相關步驟，直到完成為止。
5.	再次嘗試存取應用程式。


**Windows 10 2015 年 11 月更新**

1.	開啟 [Settings] 應用程式。
2.	按一下 [帳戶] > [您的帳戶]。
3.	按一下 [新增工作或學校帳戶]。
4.	向您的組織驗證、視需要提供 Multi-Factor Authentication 證明，然後遵循相關步驟，直到完成為止。
5.	再次嘗試存取應用程式。

如果您的裝置未加入網域並執行 Windows 8.1，您可以執行下列步驟，進行 [加入工作場所] 並註冊 Microsoft Intune︰

1.	開啟 [電腦設定]。
2.	按一下 [網路] > [工作場所]。
3.	按一下 [**加入**]。
4.	向您的組織驗證、視需要提供 Multi-Factor Authentication 證明，然後遵循相關步驟，直到完成為止。
5.	按一下 [開啟]。
6.	等到完成為止。
7.	再次嘗試存取應用程式。


## 不支援的瀏覽器

如果您是從下列瀏覽器存取應用程式，您將會看到類似先前所示的頁面︰

- Chrome、Firefox 或不是 Windows 10 或 Windows Server 2016 中 Microsoft Edge 或 Microsoft Internet Explorer 的其他瀏覽器。
- Windows 8.1、Windows 7、Windows Server 2012 R2、Windows Server 2012 或 Windows Server 2008 R2 中的 Firefox。

![不支援的瀏覽器的「您無法從這裡完成」訊息](./media/active-directory-conditional-access-device-remediation/02.png "案例")


唯一的補救方式是使用應用程式針對您的裝置平台支援的瀏覽器。

## 從 iOS 裝置存取
稍後再查看 iPhone 或 iPad 的指示。

## 從 Android 裝置存取
稍後再查看 Android 電話或平板電腦的指示。

## 後續步驟

[Azure Active Directory 條件式存取](active-directory-conditional-access.md)

<!---HONumber=AcomDC_0831_2016--->