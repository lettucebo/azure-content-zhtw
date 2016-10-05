<properties
	pageTitle="建立媒體服務帳戶 | Microsoft Azure"
	description="說明如何在 Azure 中建立新的 Azure 媒體服務帳戶。"
	services="media-services"
	documentationCenter=""
	authors="Juliako"
	manager="erikre"
	editor=""/>

<tags
	ms.service="media-services"
	ms.workload="media"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="get-started-article"
	ms.date="09/26/2016"
	ms.author="juliako"/>


# 建立 Azure 媒體服務帳戶

> [AZURE.SELECTOR]
- [入口網站](media-services-create-account.md)
- [PowerShell](media-services-manage-with-powershell.md)
- [REST](http://msdn.microsoft.com/library/azure/dn194267.aspx)


> [AZURE.NOTE] 若要完成此教學課程，您需要 Azure 帳戶。如需詳細資訊，請參閱 [Azure 免費試用](/pricing/free-trial/?WT.mc_id=A261C142F)。
 
Azure 傳統入口網站提供能讓您快速建立 Azure 媒體服務帳戶的方式。您可以使用自己的帳戶，來存取讓您在 Azure 中儲存、加密、編碼、管理和串流播放媒體內容的媒體服務。當您建立媒體服務帳戶時，您也會在與媒體服務帳戶相同的地理區域中建立相關聯的儲存體帳戶 (或使用現有儲存體帳戶)。

本文說明如何使用「快速建立」方法，建立新的媒體服務帳戶，然後建立它與儲存體帳戶的關聯。

<a id="concepts"></a>
## 概念

存取媒體服務時需要有兩個相關聯的帳戶：

-   **媒體服務帳戶**。您的帳戶可讓您存取 Azure 中提供的一組雲端型媒體服務。媒體服務帳戶並不會儲存實際媒體內容。而是在您的帳戶中儲存媒體內容和媒體處理工作的中繼資料。當您建立帳戶時，您會選取一個可用的媒體服務區域。所選取的區域會是儲存您帳戶之中繼資料記錄的資料中心。

	Available Media Services 可用的媒體服務 (AMS) 區域如下：北歐、西歐、美國西部、美國東部、東南亞、東亞、日本西部、日本東部。媒體服務不會使用同質群組。
	
	AMS 現在也適用於下列資料中心：巴西南部、印度西部、印度南部和印度中部。您現在可以使用 Azure 傳統入口網站[來建立媒體服務帳戶](media-services-create-account.md#create-a-media-services-account-using-quick-create)，以及執行[這裡](https://azure.microsoft.com/documentation/services/media-services/)所述的各種工作。不過，這些資料中心不會啟用即時編碼。此外，並非所有類型的編碼保留單元都可用於這些資料中心。
	
	- 巴西南部：只可以使用標準和基本編碼保留單元
	- 印度西部、印度南部和印度中部：只可以使用基本編碼保留單元


-   **相關聯的儲存體帳戶**。您的儲存體帳戶就是與媒體服務帳戶相關聯的 Azure 儲存體帳戶。儲存體帳戶提供媒體檔案的 Blob 儲存體，而且必須與媒體服務帳戶位於相同的地理區域中。建立媒體服務帳戶時，可以選擇相同區域中的現有儲存體帳戶，也可以在相同區域中建立新的儲存體帳戶。如果您刪除媒體服務帳戶，並不會刪除相關儲存體帳戶中的 Blob。

<a id="quick"></a>
## 使用「快速建立」建立媒體服務帳戶

1. 在 [Azure 傳統入口網站][]中，依序按一下 [新增]、[媒體服務] 和 [快速建立]。

![Media Services Quick Create](./media/media-services-create-account/wams-QuickCreate.png)

2. 在 [名稱] 中，輸入新帳戶的名稱。媒體服務帳戶名稱為全部小寫且不含空格的數字或字母，且長度是 3 到 24 個字元。

3. 在 [區域] 中，選取將用來儲存您媒體服務帳戶之中繼資料記錄的地理區域。只有可用的媒體服務區域才會出現在下拉式清單中。

4. 在 [儲存體帳戶] 中，選取儲存體帳戶以從媒體服務帳戶提供媒體內容的 Blob 儲存體。您可以選取與媒體服務帳戶相同地理區域中的現有儲存體帳戶，也可以建立新的儲存體帳戶。新的儲存體帳戶會建立於相同的區域中。

5. 如果您已建立新的儲存體帳戶，請在 [NEW STORAGE ACCOUNT NAME] 中輸入儲存體帳戶的名稱。儲存體帳戶名稱的規則會與媒體服務帳戶相同。

6. 按一下表單底部的 [快速建立]。

您可以在視窗底端的訊息區域監視程序的狀態。

當順利建立帳戶時，狀態會變更為 [使用中]。[媒體服務] 頁面隨即開啟，其中顯示新的帳戶。

在頁面底部會顯示 [管理金鑰] 按鈕。當您按一下此按鈕時，會顯示頁面，其中具有媒體服務帳戶名稱和主要和次要金鑰。您將需要帳戶名稱和主要金鑰資訊，以便以程式設計方式存取媒體服務帳戶。

![Media Services Page](./media/media-services-create-account/wams-mediaservices-page.png)

當您按兩下帳戶名稱時，預設會顯示 [快速入門] 頁面。此頁面可讓您執行一些在入口網站的其他頁面也可以執行的管理工作。例如，您可以從此頁面上傳視訊檔案，也可以從 [內容] 頁面上傳視訊檔案。

此外，您還可以檢視使用 Azure 媒體服務 SDK 來完成下列工作的程式碼：上傳、編碼和發行影片。您可以按一下 [寫入某程式碼] 區段下的其中一個連結，並複製程式碼，然後將該程式碼用在您的應用程式中。



##媒體服務學習路徑

[AZURE.INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

##提供意見反應

[AZURE.INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]


## 後續步驟

- [開始使用提供使用.NET SDK 的點播視訊 (VoD) 內容](media-services-dotnet-get-started.md)

- [使用 .NET SDK 建立通道，以執行從單一位元速率到多位元速率串流的即時編碼](media-services-dotnet-creating-live-encoder-enabled-channel.md)

<!-- Reusable paths. -->

<!-- Anchors. -->
  [Concepts]: #concepts
  [Before you begin]: #begin
  [How to: Create a Media Services account using Quick Create]: #quick

<!-- URLs. -->
  [Web Platform Installer]: http://go.microsoft.com/fwlink/?linkid=255386

  [Azure 傳統入口網站]: http://manage.windowsazure.com/

<!---HONumber=AcomDC_0928_2016-->