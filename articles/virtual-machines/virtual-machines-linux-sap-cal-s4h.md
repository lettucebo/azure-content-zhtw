<properties 
pageTitle="在 Microsoft Azure 上部署 S/4 HANA 或 BW/4 HANA | Microsoft Azure" 
description="在 Microsoft Azure 上部署 S/4 HANA 或 BW/4 HANA" 
services="virtual-machines-linux" 
documentationCenter="" 
authors="hermanndms" 
manager="timlt" 
editor="" 
tags="azure-resource-manager" 
  keywords=""/>
<tags  
  ms.service="virtual-machines-linux" 
  ms.devlang="na" 
  ms.topic="article" 
  ms.tgt_pltfrm="vm-linux" 
  ms.workload="infrastructure-services" 
  ms.date="09/15/2016" 
  ms.author="hermannd"/>


# 在 Microsoft Azure 上部署 S/4 HANA 或 BW/4 HANA 

本文說明如何透過 SAP Cloud Appliance Library 3.0 在 Microsoft Azure 上部署 S/4 HANA。螢幕擷取畫面會逐步顯示此程序。從程序觀點來看，部署其他以 SAP HANA 為基礎的解決方案 (如 BW/4 HANA) 的方式相同。使用者只需選取不同的解決方案。

若要開始使用 SAP Cloud Appliance Library (SAP CAL)，請前往[這裡](https://cal.sap.com/)。SAP 提供了有關全新 [SAP Cloud Appliance Library 3.0](http://scn.sap.com/community/cloud-appliance-library/blog/2016/05/27/sap-cloud-appliance-library-30-came-with-a-new-user-experience) 的部落格。


下列螢幕擷取畫面逐步顯示如何在 Microsoft Azure 上部署 S/4 HANA。對於 BW/4 HANA 等其他解決方案，此程序的運作方式相同。


![](./media/virtual-machines-linux-sap-cal-s4h/s4h-pic-1b.jpg)

第一張圖顯示 Microsoft Azure 上所有以 SAP CAL HANA 為基礎的可用解決方案。已選擇「SAP S/4 HANA 內部部署版本」(螢幕擷取畫面底部的解決方案) 作為範例來逐步解說此程序。

![](./media/virtual-machines-linux-sap-cal-s4h/s4h-pic-2.jpg)

首先必須建立新的 SAP CAL 帳戶。Azure 目前有兩個選擇 - 標準 Azure 以及在中國大陸由合作夥伴 21Vianet 運作的 Azure。

![](./media/virtual-machines-linux-sap-cal-s4h/s4h-pic3b.jpg)

然後使用者必須輸入可在 Azure 入口網站上找到的 Azure 訂用帳戶識別碼 - 請進一步參閱底下的取得方式。之後必須下載 Azure 管理憑證。

![](./media/virtual-machines-linux-sap-cal-s4h/s4h-pic6b.jpg)

在新的 Azure 入口網站中，使用者可在左側找到 [訂用帳戶] 項目。按一下可顯示您的使用者的所有作用中訂用帳戶。

![](./media/virtual-machines-linux-sap-cal-s4h/s4h-pic7b.jpg)

選取其中一個訂用帳戶，然後選擇 [管理憑證]，表示有對新的 Azure Resource Manager 模型使用「服務主體」的新概念。SAP CAL 尚未針對這個新的模型進行調整，仍需要「傳統」模型和舊版 Azure 入口網站才能使用管理憑證。

![](./media/virtual-machines-linux-sap-cal-s4h/s4h-pic4b.jpg)

使用者可以在這裡看到舊版 Azure 入口網站。上傳管理憑證，可讓 SAP CAL 有權限在客戶訂用帳戶內建立虛擬機器。在「訂用帳戶」索引標籤之下，使用者可以找到必須在 SAP CAL 入口網站中輸入的訂用帳戶識別碼。

![](./media/virtual-machines-linux-sap-cal-s4h/s4h-pic5.jpg)

在第二個索引標籤上，可以接著上傳之前從 SAP CAL 下載的管理憑證。

![](./media/virtual-machines-linux-sap-cal-s4h/s4h-pic8.jpg)

隨即跳出小型對話方塊，以便選取下載的憑證檔案。

![](./media/virtual-machines-linux-sap-cal-s4h/s4h-pic9.jpg)

上傳憑證後，即可在 SAP CAl 內測試 SAP CAL 與客戶 Azure 訂用帳戶之間的連線。應會跳出一則小訊息，告知此連線有效。

![](./media/virtual-machines-linux-sap-cal-s4h/s4h-pic10.jpg)

設定帳戶之後，使用者必須選取應部署的解決方案並建立執行個體。在「基本」模式中，這真的微不足道。輸入執行個體名稱，選擇 Azure 區域並定義解決方案的主要密碼。

![](./media/virtual-machines-linux-sap-cal-s4h/s4h-pic11.jpg)

在一段時間之後，視解決方案的大小和複雜性 (由 SAP CAL 提供估計) 而定，它會顯示為「作用中」並可供使用。這非常簡單。

![](./media/virtual-machines-linux-sap-cal-s4h/s4h-pic12.jpg)

查看解決方案的一些詳細資料，使用者可以看到已部署的 VM 種類。在此情況下，會建立三個不同大小和用途的 Azure VM。

![](./media/virtual-machines-linux-sap-cal-s4h/s4h-pic13.jpg)

在 Azure 入口網站上，可以找到以 SAP CAL 中所提供的相同執行個體名稱開頭的虛擬機器。

![](./media/virtual-machines-linux-sap-cal-s4h/s4h-pic14b.jpg)

現在可以透過 SAP CAL 入口網站中的 [連接] 按鈕連接到解決方案。小型對話方塊包含使用者指南的連結，其中說明使用解決方案時的所有預設認證。

![](./media/virtual-machines-linux-sap-cal-s4h/s4h-pic15.jpg)

另一個選項是登入用戶端 Windows VM，並啟動預先設定的 SAP GUI (舉例來說)。

<!---HONumber=AcomDC_0921_2016-->