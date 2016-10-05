<properties
   pageTitle="Microsoft Azure 記錄整合簡介 (預覽) | Microsoft Azure"
   description="了解 Azure 記錄整合、其主要功能及運作方式。"
   services="security"
   documentationCenter="na"
   authors="TomShinder"
   manager="MBaldwin"
   editor="TerryLanfear"/>

<tags
   ms.service="security"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="08/24/2016"
   ms.author="TomSh"/>

# Microsoft Azure 記錄整合簡介 (預覽)

了解 Azure 記錄整合、其主要功能及運作方式。

## Overview

裝載於 Azure 中的「平台即服務」(PaaS) 和「基礎結構即服務」(IaaS) 會在安全性記錄檔中產生大量資料。這些記錄檔包含重要的資訊，可提供關於下列項目有智慧且有力的深入了解：違反原則、內部和外部威脅、法規遵循問題，以及網路、主機及使用者活動中的異常。

Azure 記錄整合可讓您將來自 Azure 資源的未經處理記錄，整合到內部部署安全性資訊及事件管理 (SIEM) 系統內。Azure 記錄整合會從您的 Windows *(WAD)* 虛擬機器收集 Azure 診斷資料，也會從合作夥伴解決方案 (例如「Web 應用程式防火牆」(WAF)) 收集診斷資料。這項整合提供您內部部署或在雲端中所有資產統一的儀表板，以便您彙總、相互關聯、分析和警示安全性事件。

![Azure 記錄整合][1]

## 可以整合哪些記錄檔？

Azure 會為每項 Azure 服務產生大量記錄。這些記錄檔會分類為兩種主要類型︰

- **控制/管理記錄檔**：可讓您看到 Azure Resource Manager 的 CREATE、UPDATE 和 DELETE 作業。Azure 稽核記錄檔是本類型的一個範例。
- **資料平面記錄檔**：可讓您看到使用 Azure 資源時所引發的事件。本類型的範例是虛擬機器中的 Windows 事件系統記錄檔、安全性和應用程式記錄檔。

Azure 記錄整合目前支援 Azure 稽核記錄檔、虛擬機器記錄檔 和 Azure 資訊安全中心警示的整合。

如果您有關於「Azure 記錄整合」的問題，請傳送電子郵件給 [AzSIEMteam@microsoft.com](mailto:AzSIEMteam@microsoft.com)

## 後續步驟

在本文中為您介紹了 Azure 記錄整合。若要深入了解 Azure 記錄整合和支援的記錄檔的類型，請參閱下列各項︰

- [Azure 記錄檔的 Microsoft Azure 記錄整合 (預覽)](https://www.microsoft.com/download/details.aspx?id=53324) – Azure 記錄整合詳細資料、系統需求和安裝指示的下載中心。
- [開始使用 Azure 記錄整合](security-azure-log-integration-get-started.md) - 本教學課程將逐步引導您安裝 Azure 記錄整合，和來自 Azure 儲存體、Azure 稽核記錄檔以及 Azure 資訊安全中心警示的整合記錄檔。
- [合作夥伴設定步驟](https://blogs.msdn.microsoft.com/azuresecurity/2016/08/23/azure-log-siem-configuration-steps/) – 此部落格文章說明如何設定 Azure 記錄整合，以搭配使用合作夥伴解決方案 Splunk、HP ArcSight 和 IBM QRadar。
- [Azure 記錄整合常見問題集 (FAQ)](security-azure-log-integration-faq.md) - 此常見問題集會回答有關 Azure 記錄整合的問題。
- [以 Azure 記錄整合來整合資訊安全中心警示](../security-center/security-center-integrating-alerts-with-log-integration.md) - 這份文件說明如何將資訊安全中心警示以及 Azure 診斷和 Azure 稽核記錄檔所收集的虛擬機器安全性事件，與您的 Log Analytics 或 SIEM 方案進行同步處理。
- [Azure 診斷和 Azure 稽核記錄檔的新功能](https://azure.microsoft.com/blog/new-features-for-azure-diagnostics-and-azure-audit-logs/) – 此部落格文章為您介紹 Azure 稽核記錄檔和其他功能，協助您深入了解您的 Azure 資源的作業。

<!--Image references-->
[1]: ./media/security-azure-log-integration-overview/azure-log-integration.png

<!---HONumber=AcomDC_0921_2016-->