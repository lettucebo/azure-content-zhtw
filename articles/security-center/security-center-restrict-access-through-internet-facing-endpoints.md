<properties
   pageTitle="在 Azure 資訊安全中心限制透過網際網路面向端點的存取 | Microsoft Azure"
   description="本文件說明了如何實作 Azure 資訊安全中心建議的「限制透過網際網路面向端點的存取」。"
   services="security-center"
   documentationCenter="na"
   authors="TerryLanfear"
   manager="MBaldwin"
   editor=""/>

<tags
   ms.service="security-center"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="07/26/2016"
   ms.author="terrylan"/>

# 在 Azure 資訊安全中心限制透過網際網路面向端點的存取

如果您的任一網路安全性群組 (NSG) 有一或多個輸入規則允許來自任何來源 IP 位址的存取，Azure 資訊安全中心會建議您限制透過網際網路面向端點的存取。開放任一來源 IP 位址的存取可能會讓攻擊者存取您的資源。資訊安全中心會建議您編輯這些輸入規則，以限制只有實際上需要存取權的來源 IP 位址才能存取。

有「任何」來源的任何非 Web 連接埠，就會產生這項建議。

> [AZURE.NOTE] 本文件將使用範例部署來介紹服務。這不是逐步指南。

## 實作建議

1. 在 [建議] 刀鋒視窗中，選取 [限制透過網際網路面向端點的存取]。![限制透過網際網路面向端點的存取][1]

2. 這樣會開啟 [限制透過網際網路面向端點的存取] 刀鋒視窗。此刀鋒視窗會列出虛擬機器 (VM) 與導致潛在安全性問題的輸入規則。選取 VM。![選取 VM][2]

3. [NSG] 刀鋒視窗會顯示網路安全性群組資訊、相關的輸入規則和關聯的 VM。選取 [編輯輸入規則] 繼續編輯輸入規則。![[網路安全性群組] 刀鋒視窗][3]

4. 在 [輸入安全性規則] 刀鋒視窗中選取要編輯的輸入規則。在此範例中，我們選取 [允許 Web]。![輸入安全性規則][4]

  注意，您也可以選取 [預設規則] 以查看所有 NSG 包含的預設規則集。預設規則無法刪除，但因為其會指派為較低優先權，因此可以由您所建立的規則覆寫預設規則。深入了解 [預設規則](../virtual-network/ virtual-networks-nsg.md#default-rules)。![預設規則][5]

5. 在 [允許 Web] 刀鋒視窗中編輯輸入規則的內容，讓**來源**是 IP 位址或 IP 位址區塊。若要深入了解輸入規則的內容，請參閱 [NSG 規則](../virtual-network/virtual-networks-nsg.md#nsg-rules)。

  ![編輯輸入規則][6]

## 另請參閱

本文說明了如何實作資訊安全中心建議的「限制透過網際網路面向端點的存取」。 若要深入了解啟用 NSG 與規則，請參閱下列項目：

- [什麼是網路安全性群組 (NSG)？](../virtual-network/virtual-networks-nsg.md)
- [如何使用 Azure 入口網站管理 NSG](../virtual-network/virtual-networks-create-nsg-arm-pportal.md)

如要深入了解資訊安全中心，請參閱下列主題：

- [在 Azure 資訊安全中心設定安全性原則](security-center-policies.md) -- 了解如何為您的 Azure 訂用帳戶及資源群組設定安全性原則。
- [管理 Azure 資訊安全中心的安全性建議](security-center-recommendations.md) -- 了解建議如何協助保護您的 Azure 資源。
- [Azure 資訊安全中心的安全性健康情況監視](security-center-monitoring.md) -- 了解如何監視 Azure 資源的健康情況。
- [管理及回應 Azure 資訊安全中心的安全性警示](security-center-managing-and-responding-alerts.md) -- 了解如何管理及回應安全性警示。
- [使用 Azure 資訊安全中心監視合作夥伴解決方案](security-center-partner-solutions.md) -- 了解如何監視合作夥伴解決方案的健康情況。
- [Azure 資訊安全中心常見問題集 (FAQ)](security-center-faq.md) -- 尋找有關使用服務的常見問題。
- [Azure 安全性部落格](http://blogs.msdn.com/b/azuresecurity/) -- 取得最新的 Azure 安全性新聞和資訊。

<!--Image references-->
[1]: ./media/security-center-restrict-access-thru-internet-facing-endpoint/restrict-access-thru-internet-facing-endpoint.png
[2]: ./media/security-center-restrict-access-thru-internet-facing-endpoint/select-a-vm.png
[3]: ./media/security-center-restrict-access-thru-internet-facing-endpoint/network-security-group-blade.png
[4]: ./media/security-center-restrict-access-thru-internet-facing-endpoint/inbound-security-rules.png
[5]: ./media/security-center-restrict-access-thru-internet-facing-endpoint/default-rules.png
[6]: ./media/security-center-restrict-access-thru-internet-facing-endpoint/edit-inbound-rule.png

<!---HONumber=AcomDC_0727_2016-->