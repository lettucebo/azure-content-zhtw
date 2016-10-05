<properties 
	pageTitle="企業整合概觀 | Microsoft Azure App Service | Microsoft Azure" 
	description="利用企業整合的功能，使用 Logic Apps 啟用商務程序和整合案例" 
	services="logic-apps" 
	documentationCenter=".net,nodejs,java"
	authors="msftman" 
	manager="erikre" 
	editor="cgronlun"/>

<tags 
	ms.service="logic-apps" 
	ms.workload="integration" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="09/08/2016" 
	ms.author="deonhe"/>

# 企業整合套件概觀

## 何謂企業整合套件？
企業整合套件是 Microsoft 的雲端解決方案，可順暢地啟用企業對企業 (B2B) 通訊。此套件會使用業界標準通訊協定，包括 [AS2](./app-service-logic-enterprise-integration-as2.md)、[X12](./app-service-logic-enterprise-integration-x12.md) 和 [EDIFACT](./app-service-logic-enterprise-integration-edifact.md)，在商業夥伴之間交換訊息。您可以選擇性地使用加密和數位簽章來保護訊息。

此套件可讓使用不同通訊協定和格式的組織，透過將不同格式轉換為雙方組織系統都可解譯並採取動作的格式，以電子方式交換訊息。

如果您熟悉 BizTalk Server 或 Microsoft Azure BizTalk 服務，您就會發現可輕鬆地使用企業整合功能，因為大部分的概念都很類似。有一個主要差異是企業整合會使用整合帳戶，來簡化對於 B2B 通訊中所使用構件的儲存與管理。

在架構上，企業整合套件是以儲存所有構件的**整合帳戶**依據，這些構件可用來設計、部署及維護您的 B2B 應用程式。整合帳戶基本上是以雲端為基礎的容器，您可以在其中儲存構件，例如結構描述、夥伴、憑證、對應和合約。這些構件接著可在 Logic Apps 中用來建置 B2B 工作流程。您需要先將整合帳戶連結到邏輯應用程式，才能在邏輯應用程式中使用構件。連結它們之後，邏輯應用程式將可存取整合帳戶的構件。

## 為什麼應該使用企業整合？
- 透過企業整合，您就能夠在同一個地方 (也就是您的整合帳戶) 使用所有構件。
- 您可以利用 Logic Apps 引擎和它的所有連接器來建置 B2B 工作流程，並整合協力廠商 SaaS 應用程式、內部部署應用程式，以及自訂應用程式
- 您也可以利用 Azure Functions

## 如何開始使用企業整合？
您可以透過 **Azure 入口網站**上的 Logic Apps 設計工具，使用邏輯應用程式來建置和管理 B2B 應用程式。

您也可以使用 [PowerShell](https://msdn.microsoft.com/library/azure/mt652195.aspx "邏輯應用程式 PowerShell 主題") 來管理邏輯應用程式。

以下是您可以在 Azure 入口網站中建立應用程式之前需採取的步驟概觀︰![概觀影像](./media/app-service-logic-enterprise-integration-overview/overview-0.png)

## 有哪些常見的案例？

企業整合支援下列產業標準：

- EDI - 電子資料交換
- EAI - 企業應用程式整合

## 若要開始，您需要：
- 具有整合帳戶的 Azure 訂用帳戶
- 可建立對應和結構描述的 Visual Studio 2015
- [Visual Studio 2015 2.0 適用的 Microsoft Azure Logic Apps 企業整合工具](https://aka.ms/vsmapsandschemas)

## 試試看
[立即試用](https://github.com/Azure/azure-quickstart-templates/tree/master/201-logic-app-as2-send-receive)來部署全功能的範例 AS2 傳送與接收邏輯應用程式，以使用 Logic Apps 的 B2B 功能。

## 深入了解：
- [合約](./app-service-logic-enterprise-integration-agreements.md "了解企業整合合約")
- [企業對企業 (B2B) 案例](./app-service-logic-enterprise-integration-b2b.md "了解如何建立具有 B2B 功能的 Logic Apps")
- [憑證](./app-service-logic-enterprise-integration-certificates.md "了解企業整合憑證")
- [一般檔案編碼/解碼](./app-service-logic-enterprise-integration-flatfile.md "了解如何將一般檔案內容編碼和解碼")
- [整合帳戶](./app-service-logic-enterprise-integration-accounts.md "了解整合帳戶")
- [對應](./app-service-logic-enterprise-integration-maps.md "了解企業整合對應")
- [合作夥伴](./app-service-logic-enterprise-integration-partners.md "了解企業整合夥伴")
- [結構描述](./app-service-logic-enterprise-integration-schemas.md "了解企業整合結構描述")
- [XML 訊息驗證](./app-service-logic-enterprise-integration-xml.md "了解如何使用 Logic Apps 驗證 XML 訊息")
- [XML 轉換](./app-service-logic-enterprise-integration-transform.md "了解企業整合對應")
- [企業整合連接器](../connectors/apis-list.md "了解企業整合套件連接器")

<!---HONumber=AcomDC_0921_2016-->