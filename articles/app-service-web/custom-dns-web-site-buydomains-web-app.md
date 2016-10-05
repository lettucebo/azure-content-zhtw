<properties
	pageTitle="如何在 Azure App Service Web Apps 中購買自訂網域名稱"
	description="了解如何在 Azure App Service 中購買搭配 Web 應用程式的自訂網域名稱。"
	services="app-service\web"
	documentationCenter=""
	authors="rmcmurray"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="app-service-web"
	ms.workload="web"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/11/2016"
	ms.author="robmcm"/>

# 在 Azure App Service 中購買並設定自訂網域名稱

[AZURE.INCLUDE [web-selector](../../includes/websites-custom-domain-selector.md)]

當您建立 Web 應用程式時，Azure 會將它指派給 azurewebsites.net 的子網域。例如，如果您的 Web 應用程式名稱為 **contoso**，則 URL 會是 **contoso.azurewebsites.net**。Azure 也會指派虛擬 IP 位址。

若是實際執行的 Web 應用程式，您可能想讓使用者看到自訂網域名稱。本文說明如何購買並使用 [App Service Web Apps](http://go.microsoft.com/fwlink/?LinkId=529714) 設定自訂網域。

[AZURE.INCLUDE [introfooter](../../includes/custom-dns-web-site-intro-notes.md)]


## Overview

如果您的 Web 應用程式沒有網域名稱，您可以輕鬆地在 [Azure 入口網站](https://portal.azure.com/)購買一個。在購買程序期間，您可以選擇將 WWW 和根網域的 DNS 記錄自動對應到您的 Web 應用程式。您也可以在 Azure 入口網站管理您的網域權限。


使用下列步驟購買網域名稱，並將其指派給您的 Web 應用程式。

1. 在瀏覽器中，開啟 [Azure 入口網站](https://portal.azure.com/)。

2. 在 [Web Apps] 索引標籤中，按一下您 Web 應用程式的名稱，並選取 [設定]，然後選取 [自訂網域]

	![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-6.png)

3. 在 [自訂網域] 刀鋒視窗中，按一下 [購買網域]。

	![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-1.png)

4. 在 [**購買網域**] 刀鋒視窗中，請使用文字方塊輸入您想要購買的網域名稱，然後按下 Enter 鍵。建議的可用網域將會顯示在文字方塊下方。選取您想要購買的網域。您可以選擇一次購買多個網域。

  ![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-2.png)

5. 按一下 [**連絡資訊**] 然後填寫網域連絡資訊表單。

  ![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-3.png)

    > [AZURE.NOTE] It is very important that you fill out all required fields with as much accuracy as possible, especially the email address. In case of purchasing the domain without "Privacy protection", you might be asked to verify your email before the domain becomes active. In some cases, incorrect data for contact information will result in failure to purchase domains. 

6. 現在您可以選擇，

	a) 每年「自動續訂」您的網域
	
	b) 選擇加入購買時免費隨附的「隱私權保護」(其登錄不支援隱私權的 TLD 除外。例如：.co.in、.co.uk 等等)
	
	c) 為 WWW 和根網域「指派預設主機名稱」到目前的 Web 應用程式。

  ![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-2.5.png)
  
    > [AZURE.NOTE] Option C configures DNS bindings and Hostname bindings automatically for you.  This way, your Web App can be accessed using custom domain as soon as the purchase is complete (baring DNS propagation delays in few cases). In case, your Web App is behind Azure Traffic Manager, you will not see an option to assign root domain, as A-Records do not work with the Traffic Manager. 
    You can always assign the domains/sub-domains purchased through one Web App to another Web App and vice-versa. See step 8 for more details. 
	
7. 按一下 [**購買網域**] 刀鋒視窗中的 [**選取**]，您就會在 [**購買確認**] 刀鋒視窗看到購買資訊。如果您接受法律條款並按下 [**購買**]，您的訂單將會送出，而您可以在 [**通知**] 中監控購買程序。網域訂單可能需要幾分鐘的時間才能完成。

  ![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-4.png)

  ![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-5.png)

8. 如果您已成功下訂一個網域，您就可以管理網域並指派給您的 Web 應用程式。在您網域的右側按一下 [**...**]。您就可以 [**取消訂單**] 或 [**管理網域**]。按一下 [**管理網域**]，我們就可以在 [**管理網域**] 刀鋒視窗將**子網域**繫結至我們的 Web 應用程式。如果您想要繫結一個**子網域**到不同的 Web 應用程式，那麼請從個別 Web 應用程式的內容中執行此步驟。在這裡，您可以從下拉式功能表選取流量管理員名稱，將網域指派至流量管理員端點 (如果 Web 應用程式在流量管理員後面)。如此一來，網域/子網域會自動指派給該流量管理員端點後面的所有 Web 應用程式。

	![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-6.png)

    > [AZURE.NOTE] 您可以在 5 天內「取消購買」以獲取完整退款。5 天後您將無法再「取消購買」，取而代之的是您會看到「刪除」網域的選項。刪除該網域不會退款，並將導致網域從您訂用帳戶釋放變成可用的網域。

完成設定後，自訂網域名稱將列在 Web 應用程式的 [**主機名稱繫結**] 區段中。

此時，您應該能夠在瀏覽器中輸入自訂網域名稱，並且能成功移至您的 Web 應用程式。
 
## 您購買的自訂網域會發生什麼事

您在 [自訂網域及 SSL] 刀鋒視窗中購買的自訂網域，會繫結至 Azure 訂用帳戶。雖然這個自訂網域是 Azure 資源，但它是獨立的，與一開始需要您購買網域的那個 App Service 應用程式並沒有關聯。這表示：

- 在 Azure 入口網站中，您可以把您購買的自訂網域提供給多個 App Service 應用程式使用，而不是只能提供給一開始需要您購買自訂網域的那個應用程式。
- 您可以在 Azure 訂用帳戶中管理您購買的所有自訂網域，方法是前往該訂用帳戶中「任何」App Service 應用程式的 [自訂網域及 SSL] 刀鋒視窗。
- 您可以在相同的 Azure 訂用帳戶中，把任何 App Service 應用程式指派該自訂網域中的子網域。
- 如果您決定要刪除某個 App Service 應用程式，可以選擇不要刪除與該應用程式繫結的自訂網誌，並把該自訂網域提供給其他應用程式使用。

## 如果您沒看到自己購買的自訂網域

如果您已經在 [自訂網域及 SSL] 刀鋒視窗中購買自訂網域，但卻沒有在 [管理網域] 下方看到該自訂網域，請確認下列事項：

- 自訂網域的建立程序可能還沒有完成。請查看 Azure 入口網站頂端的通知鈴鐺來檢視進度。
- 自訂網域的建立可能因某種原因而失敗。請查看 Azure 入口網站頂端的通知鈴鐺來檢視進度。
- 自訂的網域可能已建立成功，但刀鋒視窗可能還沒重新整理。請嘗試重新開啟 [自訂網域及 SSL] 刀鋒視窗。
- 您可能已經在某個時間點刪除了自訂網域。請查看稽核記錄，方法是前往您應用程式的主刀鋒視窗，然後按一下 [設定] > [稽核記錄]。
- 您正在查看的 [自訂網域及 SSL] 刀鋒視窗，可能屬於不同 Azure 訂用帳戶所建立的應用程式。請切換到不同訂用帳戶的另一個應用程式，並查看該應用程式的 [自訂網域及 SSL] 刀鋒視窗。在入口網站中，除了應用程式之外，您無法查看或管理不同的 Azure 訂用帳戶所建立的自訂網域。不過，如果您在網域的 [管理網域] 刀鋒視窗中按一下 [進階管理]，您會被重新導向到網域提供者的網站，讓您能夠在那裡為不同 Azure 訂用帳戶所建立的應用程式[手動設定您的自訂網域，就跟設定所有外部自訂網域一樣](web-sites-custom-domain-name.md)。

<!---HONumber=AcomDC_0921_2016-->