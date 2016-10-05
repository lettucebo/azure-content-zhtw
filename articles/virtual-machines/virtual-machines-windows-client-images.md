<properties
   pageTitle="使用 Windows 用戶端映像進行開發/測試案例 | Microsoft Azure"
   description="如何使用 Visual Studio 訂用帳戶權益在 Azure 中部署 Windows 7/8/10，以進行開發/測試案例"
   services="virtual-machines-windowse"
   documentationCenter=""
   authors="iainfoulds"
   manager="timlt"
   editor=""/>

<tags
   ms.service="virtual-machines-windows"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="vm-windows"
   ms.workload="infrastructure-services"
   ms.date="08/31/2016"
   ms.author="iainfou"/>

# 在 Azure 中使用 Windows 用戶端進行開發/測試案例

假設您有適當的 Visual Studio (先前稱為 MSDN) 訂用帳戶，您可以在 Azure 中使用 Windows 7、Windows 8 或 Windows 10 進行開發/測試案例。本文概述在 Azure 中執行 Windows 用戶端和使用 Azure 資源庫映像的資格需求。


## 訂用帳戶資格
有效的 Visual Studio 訂閱者 (已取得 Visual Studio 訂用帳戶授權的人) 可以使用 Windows 用戶端進行開發和測試。Windows 用戶端可以用在您自己的硬體及以任何類型的 Azure 訂用帳戶執行的 Azure 虛擬機器。Windows 用戶端無法部署至或用於 Azure 來從事一般實際執行用途，或由不是有效 Visual Studio 訂閱者的人使用。

為方便您使用，我們在 Azure 資源庫的[符合資格者的開發/測試優惠](#eligible-offers)內提供特定的 Windows 10 映像。屬於任一優惠類型的 Visual Studio 訂閱者也可以[適當地準備和建立](virtual-machines-windows-prepare-for-upload-vhd-image.md) 64 位元的 Windows 7、Windows 8 或 Windows 10 映像，然後[上傳至 Azure](virtual-machines-windows-upload-image.md)。僅限有效的 Visual Studio 訂閱者用於開發/測試用途。


## 符合資格者的優惠
下表詳細說明可透過 Azure 資源庫部署 Windows 10 的優惠識別碼。只有在下列優惠中才能看到 Windows 10 映像。需以其他優惠類型執行 Windows 用戶端的 Visual Studio 訂閱者則需要[適當地準備和建立](virtual-machines-windows-prepare-for-upload-vhd-image.md) 64 位元的 Windows 7、Windows 8 或 Windows 10 映像，然後 [上傳至 Azure](virtual-machines-windows-upload-image.md)。

| 優惠名稱 | 優惠號碼 | 可用的用戶端映像 |
|:-----------|:------------:|:-----------------------:|
| [隨用隨付開發/測試](https://azure.microsoft.com/offers/ms-azr-0023p/) | 0023P | Windows 10 |
| [Visual Studio Enterprise (MPN) 訂閱者](https://azure.microsoft.com/offers/ms-azr-0029p/) | 0029P | Windows 10 |
| [Visual Studio Professional 訂閱者](https://azure.microsoft.com/offers/ms-azr-0059p/) | 0059P | Windows 10 |
| [Visual Studio Test Professional 訂閱者](https://azure.microsoft.com/offers/ms-azr-0060p/) | 0060P | Windows 10 |
| [Visual Studio Premium 含 MSDN (權益)](https://azure.microsoft.com/offers/ms-azr-0061p/) | 0061P | Windows 10 |
| [Visual Studio Enterprise 訂閱者](https://azure.microsoft.com/offers/ms-azr-0063p/) | 0063P | Windows 10 |
| [Visual Studio Enterprise (BizSpark) 訂閱者](https://azure.microsoft.com/offers/ms-azr-0064p/) | 0064P | Windows 10 |
| [Enterprise 開發/測試](https://azure.microsoft.com/ofers/ms-azr-0148p/) | 0148P | Windows 10 |


## 檢查您的 Azure 訂用帳戶
如果您不知道您的優惠識別碼，您可以透過 Azure 入口網站或帳戶入口網站取得。

訂用帳戶優惠識別碼顯示在 Azure 入口網站的 [訂用帳戶] 刀鋒視窗︰

![Azure 入口網站中的優惠識別碼詳細資料](./media/virtual-machines-windows-client-images/offer_id_azure_portal.png)

您也可以從 Azure 帳戶入口網站的 [[訂用帳戶] 索引標籤](http://account.windowsazure.com/Subscriptions)檢視優惠識別碼︰

![Azure 帳戶入口網站中的優惠識別碼詳細資料](./media/virtual-machines-windows-client-images/offer_id_azure_account_portal.png)


## 後續步驟
您現在可以使用 [PowerShell](virtual-machines-windows-ps-create.md)、[Resource Manager 範本](virtual-machines-windows-ps-template.md) 或 [Visual Studio](../vs-azure-tools-resource-groups-deployment-projects-create-deploy.md) 來部署您的 VM。

<!---HONumber=AcomDC_0921_2016-->