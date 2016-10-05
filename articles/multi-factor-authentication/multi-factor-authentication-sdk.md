<properties 
	pageTitle="整合內部部署身分識別與 Azure Active Directory。"
	description="這就是 Azure AD Connect，說明它是什麼及使用的理由。"
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

# 在自訂應用程式中建置 Multi-Factor Authentication (SDK)

> [AZURE.IMPORTANT]  如果您想要下載 SDK，即使您有 Azure MFA、AAD Premium 或 EMS 授權，還是必須建立 Azure Multi-Factor Auth Provider。如果您基於此目的建立 Azure Multi-Factor Auth Provider 並已擁有授權，則必須建立採用 [每個啟用的使用者] 模型的 Provider，並將 Provider 連結至包含 Azure MFA、Azure AD Premium 或 EMS 授權的目錄。這可確保您不須付費，除非使用 SDK 的唯一使用者超過您所擁有的授權數目。

Azure Multi-Factor Authentication 軟體開發套件 (SDK) 可讓您將通話和簡訊驗證直接內建到 Azure AD 租用戶中的應用程式登入或交易程序。

Multi-Factor Authentication SDK 適用於 C#、Visual Basic (.NET)、Java、Perl、PHP 和 Ruby。SDK 為多因素驗證加上一層精簡的包裝函式。它包含您撰寫程式碼所需要的一切，包括加註的原始程式碼檔案、範例檔案，以及詳細的 ReadMe 檔案。每一套 SDK 也包含 Multi-Factor Authentication 提供者獨有的憑證和私密金鑰，用來加密交易。只要您有提供者，您就可以視需要下載許多種語言和格式的 SDK。

Multi-Factor Authentication SDK 中的 API 結構相當簡單。您可以使用多因素選項參數 (例如驗證模式) 和使用者資料 (例如要撥打的電話號碼或要驗證的 PIN 碼)，透過單一函式來呼叫 API。API 會將函式呼叫轉換成 Web 服務要求，再提交到以雲端為基礎的 Azure Multi-Factor Authentication Service。所有呼叫都必須參考每個 SDK 所包含的私人憑證。

因為 API 無權存取 Azure Active Directory 中註冊的使用者，您必須在檔案或資料庫中提供使用者資訊，例如電話號碼和 PIN 碼。API 也沒有提供註冊或使用者管理功能，所以您需要將這些程序內建到您的應用程式中。






## 下載 Azure Multi-Factor Authentication SDK

下載 Azure Multi-factor SDK 需要 [Azure Multi-Factor Auth Provider](multi-factor-authentication-get-started-auth-provider.md)。即使您擁有 Azure MFA、Azure AD Premium 或 Enterprise Mobility Suite 授權，這還是需要完整的 Azure 訂用帳戶。若要下載 SDK，您必須瀏覽至 Multi-Factor 管理入口網站，方法是直接管理 Multi-Factor Auth Provider，或按一下 MFA 服務設定頁面上的 [移至入口網站] 連結。


### 從 Azure 入口網站下載 Azure Multi-Factor Authentication SDK


1. 以系統管理員身分登入 Azure 入口網站。
2. 在左側選取 [Active Directory]。
3. 在 [Active Directory] 頁面頂端，按一下 [**多因素驗證提供者**]
4. 在底部按一下 [**管理**]
5. 這會開啟新的頁面。在左下方按一下 [SDK]。
<center>![下載](./media/multi-factor-authentication-sdk/download.png)</center>
6. 選取您想要的語言，然後按一下其中一個相關聯的下載連結。
7. 儲存下載內容。



### 透過服務設定下載 Azure Multi-Factor Authentication SDK


1. 以系統管理員身分登入 Azure 入口網站。
2. 在左側選取 [Active Directory]。
3. 按兩下您的 Azure AD 的執行個體。
4. 在頂端按一下 [**設定**]
5. 在 Multi-Factor Authentication 底下選取 [**管理服務設定**] ![下載](./media/multi-factor-authentication-sdk/download2.png)
6. 在 [服務設定] 頁面上，於畫面底部按一下 [**移至入口網站**]。![下載](./media/multi-factor-authentication-sdk/download3a.png)
7. 這會開啟新的頁面。在左下方按一下 [SDK]。
8. 選取您想要的語言，然後按一下其中一個相關聯的下載連結。
9. 儲存下載內容。

## Azure Multi-Factor Authentication Server SDK 的內容
在 SDK 內，您會找到下列項目：

- **讀我檔案**。說明如何在新的或現有的應用程式中使用 Multi-Factor Authentication API。
- Multi-Factor Authentication 的**原始程式檔**
- 您用來與 Multi-Factor Authentication 服務進行通訊的**用戶端憑證**
- 憑證的**私密金鑰**
- **呼叫結果。** 呼叫結果碼的清單。若要開啟此檔案，請使用可設定文字格式的應用程式，例如 WordPad。使用呼叫結果碼來測試及疑難排解您在應用程式中的 Multi-Factor Authentication 實作。它們不是驗證狀態碼。
- **範例。** Multi-Factor Authentication 基本工作實作的範例程式碼。


>[AZURE.WARNING]用戶端憑證是特別為您產生的唯一私人憑證。請勿分享或遺失此檔案。它是您與 Multi-Factor Authentication 通訊時確保安全性的關鍵。

## 程式碼範例：標準模式電話驗證

此程式碼範例示範如何使用 Azure Multi-Factor Authentication SDK 中的 API，將標準模式語音通話驗證加入至您的應用程式。標準模式是指使用者按下 # 鍵來回應通話。

此範例在含有 C# 伺服器端邏輯的基本 ASP.NET 應用程式中使用 C# .NET 2.0 Multi-Factor Authentication SDK，但程序非常類似其他語言的簡單實作。因為 SDK 包含原始程式檔，而不是可執行檔，您可以建置檔案並參考它們，或直接將它們包含在您的應用程式中。

>[AZURE.NOTE]實作 Multi-Factor Authentication 時，請使用其他因素做為第二或第三驗證，以補充您的主要驗證方法。這些方法並非設計來做為主要驗證方法。

### 程式碼範例概觀
這是非常簡單的 Web 示範應用程式的範例程式碼，使用 # 鍵回應通話來完成使用者驗證。此通話因素在 Multi-Factor Authentication 中稱為標準模式。

用戶端程式碼不包含任何 Multi-Factor Authentication 特定項目。因為其他驗證因素與主要驗證無關，您可以新增它們而不必變更現有登入介面。Multi-Factor SDK 中的 API 可讓您自訂使用者經驗，但您可能完全不需要變更任何項目。

伺服器端程式碼在步驟 2 新增標準模式驗證。它使用標準模式驗證所需的參數建立 PfAuthParams 物件：使用者名稱、電話號碼、模式，以及每個呼叫都需要的用戶端憑證的路徑 (CertFilePath)。如需 PfAuthParams 中的所有參數的示範，請參閱 SDK 中的範例檔案。

接下來，程式碼將 PfAuthParams 物件傳遞至 pf\_authenticate() 函式。傳回值指出驗證成功或失敗。Out 參數 callStatus 和 errorID 包含其他通話結果資訊。通話結果碼會記錄在 SDK 中的通話結果檔案。

只需要幾行就能撰寫這個最小實作。不過，在實際執行的程式碼中，將會包含更複雜的錯誤處理、其他資料庫程式碼和強化的使用者經驗。

### Web 用戶端程式碼

以下是示範頁面的 Web 用戶端程式碼。


	<%@ Page Language="C#" AutoEventWireup="true" CodeFile="Default.aspx.cs" Inherits="_Default" %>

	<!DOCTYPE html>

	<html xmlns="http://www.w3.org/1999/xhtml">
	<head runat="server">
	<title>Multi-Factor Authentication Demo</title>
	</head>
	<body>
	<h1>Azure Multi-Factor Authentication Demo</h1>
	<form id="form1" runat="server">

	<div style="width:auto; float:left">
	Username:&nbsp;<br />
	Password:&nbsp;<br />
	</div>

	<div">
	<asp:TextBox id="username" runat="server" width="100px"/><br />
	<asp:Textbox id="password" runat="server" width="100px" TextMode="password" /><br />
	</div>

	<asp:Button id="btnSubmit" runat="server" Text="Log in" onClick="btnSubmit_Click"/>

	<p><asp:Label ID="lblResult" runat="server"></asp:Label></p>

	</form>
	</body>
	</html>


### 伺服器端程式碼

在下列伺服器端程式碼中，步驟 2 中設定並執行 Multi-Factor Authentication。標準模式 (MODE\_STANDARD) 是指使用者按下 # 鍵來回應通話。

	using System;
	using System.Collections.Generic;
	using System.Linq;
	using System.Web;
	using System.Web.UI;
	using System.Web.UI.WebControls;

	public partial class _Default : System.Web.UI.Page
	{
	    protected void Page_Load(object sender, EventArgs e)
	    {
	    }

	    protected void btnSubmit_Click(object sender, EventArgs e)
	    {
	        // Step 1: Validate the username and password
	        if (username.Text != "Contoso" || password.Text != "password")
	        {
	            lblResult.ForeColor = System.Drawing.Color.Red;
	            lblResult.Text = "Username or password incorrect.";
	        }
	        else
	        {
	            // Step 2: Perform multi-factor authentication

	            // Add call details from the user database.
	            PfAuthParams pfAuthParams = new PfAuthParams();
	            pfAuthParams.Username = username.Text;
	            pfAuthParams.Phone = "9134884271";
	            pfAuthParams.Mode = pf_auth.MODE_STANDARD;

	            // Specify a client certificate
	            // NOTE: This file contains the private key for the client
	            // certificate. It must be stored with appropriate file
	            // permissions.
	            pfAuthParams.CertFilePath = "c:\\cert_key.p12";

	            // Perform phone-based authentication
	            int callStatus;
	            int errorId;

	            if(pf_auth.pf_authenticate(pfAuthParams, out callStatus, out errorId))
	            {
	                lblResult.ForeColor = System.Drawing.Color.Green;
	                lblResult.Text = "Multi-Factor Authentication succeeded.";
	            }
	            else
	            {
	                lblResult.ForeColor = System.Drawing.Color.Red;
	                lblResult.Text = " Multi-Factor Authentication failed.";
	            }
	        }

	    }
	}

<!---HONumber=AcomDC_0921_2016-->