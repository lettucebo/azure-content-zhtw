<properties
	pageTitle="變更為 Azure AD v2.0 端點 | Microsoft Azure"
	description="對應用程式模型 v2.0 公用預覽通訊協定所進行的變更的描述。"
	services="active-directory"
	documentationCenter=""
	authors="dstrockis"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/16/2016"
	ms.author="dastrock"/>

# v2.0 驗證通訊協定的重要更新
開發人員請注意！ 在接下來兩週，我們會對 v2.0 驗證通訊協定進行一些更新，這些更新對於您在我們的預覽期間撰寫的任何應用程式可能是重大變更。

## 那些人會受到影響？
任何已撰寫使用 v2.0 整合驗證端點的任何應用程式，

```
https://login.microsoftonline.com/common/oauth2/v2.0/authorize
```

更多關於 v2.0 端點的詳細資訊可以在[這裡](active-directory-appmodel-v2-overview.md)找到。

如果您已經藉由直接編碼至 v2.0 通訊協定，使用 v2.0 端點來建置應用程式，使用我們的任何 OpenID Connect 或 OAuth Web 中繼軟體，或使用其他協力廠商程式庫來執行驗證，您應該準備好測試您的專案，並且視需要進行變更。

## 那些人不會受到影響？
任何已根據保護 Azure AD 驗證端點所撰寫的任何應用程式，

```
https://login.microsoftonline.com/common/oauth2/authorize
```

此通訊協定一定都是如此，並且不會發生任何變更。

此外，如果您的應用程式**只**使用我們的 ADAL 程式庫來執行驗證，您不必變更任何項目。ADAL 已防護您的應用程式免於變更。

## 所做的變更有哪些？
### 從 JWT 標頭移除 x5t 值
V2.0 端點大量使用 JWT 權杖，其中包含標頭參數區段與權杖的相關中繼資料。如果您解碼其中一個目前 JWT 的標頭，您會發現類似以下的情形：

```
{ 
	"type": "JWT",
	"alg": "RS256",
	"x5t": "MnC_VZcATfM5pOYiJHMba9goEKY",
	"kid": "MnC_VZcATfM5pOYiJHMba9goEKY"
}
```

「x5t」和「kid」屬性都會識別從 OpenID Connect 中繼資料端點擷取，應該用於驗證權杖簽章的公開金鑰。

我們在這裡進行的變更是要移除「x5t」屬性。您可以繼續使用相同的機制來驗證權杖，但應該只依賴「kid」屬性來擷取正確的公用金鑰，如 OpenID Connect 通訊協定中所指定。

> [AZURE.IMPORTANT] **您的作業：請確定您的應用程式不依賴 x5t 值是否存在。**

### 移除 profile\_info
先前，v2.0 端點在稱為 `profile_info` 的權杖回應中已傳回 base64 編碼的 JSON 物件。藉由傳送要求至下列項目，從 v2.0 端點要求存取權杖時：

```
https://login.microsoftonline.com/common/oauth2/v2.0/token
```

回應看起來類似下列 JSON 物件：
```
{ 
	"token_type": "Bearer",
	"expires_in": 3599,
	"scope": "https://outlook.office.com/mail.read",
	"access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsI...",
	"refresh_token": "OAAABAAAAiL9Kn2Z27UubvWFPbm0gL...",
	"profile_info": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsI...",
}
```

包含 `profile_info` 值的資訊是關於登入應用程式的使用者 - 其顯示名稱、名字、姓氏、電子郵件地址、 識別碼等等。`profile_info` 主要是用於權杖快取和顯示用途。

我們現在移除 `profile_info` 值 - 不過別擔心，我們仍然會在稍微不同的地方為開發人員提供這項資訊。不是 `profile_info`，v2.0 端點現在會在每個權杖回應中傳回 `id_token`：

```
{ 
	"token_type": "Bearer",
	"expires_in": 3599,
	"scope": "https://outlook.office.com/mail.read",
	"access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsI...",
	"refresh_token": "OAAABAAAAiL9Kn2Z27UubvWFPbm0gL...",
	"id_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsI...",
}
```

您可以解碼並剖析的 id\_token 以擷取您從 profile\_info 所收到的相同資訊。Id\_token 是 JSON Web 權杖 (JWT)，其內容由 OpenID Connect 所指定。要執行這項操作的程式碼應該非常類似 – 您只需要擷取 id\_token 的中間區段 (主體)，而且 base64 會將其解碼以在 JSON 物件中存取。

兩週過後，您應該撰寫程式碼以從 `id_token` 或 `profile_info` (存在的其中一個) 擷取使用者資訊。如此一來，當變更時，您的應用程式可以順暢地處理從 `profile_info` 至 `id_token` 的轉換而不會中斷。

> [AZURE.IMPORTANT] **您的作業：請確定您的應用程式不依賴 `profile_info` 值是否存在。**

### 移除 id\_token\_expires\_in
類似於 `profile_info`，我們同時也從回應中移除 `id_token_expires_in` 參數。先前，v2.0 端點會傳回 `id_token_expires_in` 的值以及每個 id\_token 回應，例如在授權回應中：

```
https://myapp.com?id_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsI...&id_token_expires_in=3599...
```

或權杖回應中：

```
{ 
	"token_type": "Bearer",
	"id_token_expires_in": 3599,
	"scope": "openid",
	"id_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsI...",
	"refresh_token": "OAAABAAAAiL9Kn2Z27UubvWFPbm0gL...",
	"profile_info": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsI...",
}
```

`id_token_expires_in` 值會指出 id\_token 保持有效的秒數。現在，我們完全移除 `id_token_expires_in` 值。相反地，您可以使用 OpenID Connect 標準 `nbf` 和 `exp` 宣告來檢查 id\_token 的有效性。請參閱 [v2.0 權杖參考](active-directory-v2-tokens.md)以取得這些宣告的詳細資訊。

> [AZURE.IMPORTANT] **您的作業：請確定您的應用程式不依賴 `id_token_expires_in` 值是否存在。**


### 變更 scope=openid 傳回的宣告
這項變更將是最重要的 – 事實上，它將會影響使用 v2.0 端點的幾乎每個應用程式。許多應用程式使用 `openid` 範圍將要求傳送至 v2.0 端點，例如：

```
https://login.microsoftonline.com/common/oauth2/v2.0/authorize?
client_id=...
&redirect_uri=...
&response_mode=form_post
&response_type=id_token
&scope=openid offline_access https://outlook.office.com/mail.read
```

今天，當使用者同意 `openid` 範圍，您的應用程式會在產生的 id\_token 中收到豐富的使用者相關資訊。這些宣告可以包含其名稱、慣用的使用者名稱、電子郵件地址、物件識別碼等等。

在此更新中，我們會變更 `openid` 範圍給予您的應用程式存取權的資訊，使其更符合 OpenID Connect 規格。`openid` 範圍只允許您的應用程式將使用者登入，在 id\_token 的 `sub` 宣告中接收應用程式特定識別碼。只被授與 `openid` 範圍的 id\_token 中的宣告會缺乏任何個人識別資訊。範例 id\_token 宣告為：

```
{ 
	"aud": "580e250c-8f26-49d0-bee8-1c078add1609",
	"iss": "https://login.microsoftonline.com/b9410318-09af-49c2-b0c3-653adc1f376e/v2.0 ",
	"iat": 1449520283,
	"nbf": 1449520283,
	"exp": 1449524183,
	"nonce": "12345",
	"sub": "MF4f-ggWMEji12KynJUNQZphaUTvLcQug5jdF2nl01Q",
	"tid": "b9410318-09af-49c2-b0c3-653adc1f376e",
	"ver": "2.0",
}
```

如果您想要取得有關您的應用程式中的使用者的個人識別資訊 (PII)，您的應用程式必須向使用者要求其他權限。我們從 OpenID Connect 規格引進兩個新領域的支援 – `email` 和 `profile` 範圍 – 可讓您執行這項操作。

- `email` 範圍非常簡單，它可讓您的應用程式透過 id\_token 中的 `email` 宣告存取使用者的主要電子郵件地址。請注意，`email` 宣告不一定會出現在 id\_tokens 中 – 只有在使用者的設定檔中可用時才會包含。
- `profile` 範圍可以讓您的應用程式存取使用者的所有其他基本資訊 – 其名稱、慣用的使用者名稱、物件識別碼等等。

這可讓您以最低洩漏的方式編碼應用程式 – 您可以只向使用者要求應用程式執行其作業所需的資訊集。如果您想要繼續取得您的應用程式目前接收的使用者資訊的完整集合，您應該在授權要求中包含所有三個範圍：

```
https://login.microsoftonline.com/common/oauth2/v2.0/authorize?
client_id=...
&redirect_uri=...
&response_mode=form_post
&response_type=id_token
&scope=openid profile email offline_access https://outlook.office.com/mail.read
```

您的應用程式可以立即開始傳送 `email` 和 `profile`，v2.0 端點會接受這兩個範圍，並且視需要開始向使用者要求權限。不過，`openid` 範圍的轉譯中的變更幾週之後才會生效。

> [AZURE.IMPORTANT] **您的作業：如果您的應用程式需要使用者的相關資訊，新增 `profile` 和 `email` 範圍。** 請注意，ADAL 預設會在要求中同時包含這些權限。

### 移除簽發者結尾斜線。
先前，v2.0 端點的權杖中的簽發者值採用此格式

```
https://login.microsoftonline.com/{some-guid}/v2.0/
```

其中 guid 是發行權杖的 Azure AD 租用戶的 tenantId。進行這些變更之後，簽發者值會改變

```
https://login.microsoftonline.com/{some-guid}/v2.0 
```

在這兩個權杖和 OpenID Connect 探索文件中。

> [AZURE.IMPORTANT] **您的作業：確定您的應用程式在簽發者驗證期間接受包含或不含結尾斜線的簽發者值。**

## 為何變更？
導入這些變更的主要動機是要與 OpenID Connect 標準規格相容。與 OpenID Connect 相容，我們希望將與 Microsoft 識別服務以及業界其他識別服務整合的差異降到最低。我們想要讓開發人員使用他們最愛的開放原始碼驗證程式庫而不必變更程式庫，以配合 Microsoft 的差異。

## 您該怎麼辦？
目前，您可以開始進行上述的所有變更。您應該立即：

1.	**移除 `x5t` 標頭參數的任何相依性。**
2.	**正常處理權杖回應中從 `profile_info` 至 `id_token` 的轉換。**
3.  **移除 `id_token_expires_in` 回應參數的任何相依性。**
3.	**如果您的應用程式需要基本使用者資訊，將 `profile` 和 `email` 範圍新增至您的應用程式。**
4.	**在權杖中接受包含或不含結尾斜線的簽發者值。**

我們的 [v2.0 通訊協定文件](active-directory-v2-protocols.md)已更新以反映這些變更，因此您可能會使用它做為協助更新程式碼的參考。

如果您對於變更的範圍有任何進一步的問題，歡迎在我們的 Twitter (@AzureAD) 與我們連絡。

## 通訊協定變更發生頻率為何？
我們無法預見驗證通訊協定的任何進一步重大變更。我們刻意將這些變更統合至一個發行版本，這樣，您就不需要馬上再經歷這種類型的更新程序。當然，我們將會繼續將功能新增至您可以充分利用的 v2.0 驗證服務，但是這些變更應該只是附加的，而不是中斷現有的程式碼。

最後，感謝您在預覽期間試用功能。至此，我們早期採用者的深入資訊和體驗非常寶貴，而且我們希望您將會繼續共用您的意見與想法。

祝各位編碼程式愉快！

Microsoft 身分識別部門

<!---HONumber=AcomDC_0921_2016-->