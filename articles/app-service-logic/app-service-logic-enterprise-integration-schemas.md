<properties 
	pageTitle="結構描述與企業整合套件的概觀 | Microsoft Azure App Service | Microsoft Azure" 
	description="了解如何使用結構描述搭配企業整合套件與 Logic Apps" 
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
	ms.date="07/29/2016" 
	ms.author="deonhe"/>

# 了解結構描述與企業整合套件  

## 為什麼要使用結構描述
您可以使用結構描述來確認您收到的 XML 文件都有效，這表示文件會以預先定義格式包含預期的資料。

## 如何新增結構描述
從 Azure 入口網站：

1. 選取 [更多服務]  
![](./media/app-service-logic-enterprise-integration-overview/overview-11.png)
2. 在篩選搜尋方塊中輸入**整合**，然後從結果清單中選取  
[整合帳戶] ![](./media/app-service-logic-enterprise-integration-overview/overview-21.png)
3. 選取您要新增結構描述的**整合帳戶**  
![](./media/app-service-logic-enterprise-integration-overview/overview-31.png)
4. 選取 [結構描述] 圖格  
![](./media/app-service-logic-enterprise-integration-schemas/schema-11.png)

#### 新增少於 2 MB 的結構描述檔案  

5. 在開啟的 [結構描述] 刀鋒視窗中選取 [新增] 按鈕 ![](./media/app-service-logic-enterprise-integration-schemas/schema-21.png)
6. 輸入結構描述的**名稱**，接著，若要上傳結構描述檔案，可選取 [結構描述] 文字方塊右邊的資料夾圖示。完成上傳程序之後，選取 [確定] 按鈕。![](./media/app-service-logic-enterprise-integration-schemas/schema-31.png)

#### 新增超過 2 MB (最大 8 MB) 的結構描述檔案  

7. 如果 Blob 安全性存取層級是**公用**  
  ![](./media/app-service-logic-enterprise-integration-schemas/blob-public.png)

  * 將結構描述上傳至儲存體，並複製 URI  
  ![](./media/app-service-logic-enterprise-integration-schemas/schema-blob.png)

  * 選取 [新增結構描述] 中的 [大型檔案]，然後在 [內容 URI] 中提供 URI  
  ![](./media/app-service-logic-enterprise-integration-schemas/schema-largefile.png)

8. 如果 Blob 安全性存取層級是**沒有匿名存取**  
  ![](./media/app-service-logic-enterprise-integration-schemas/blob-1.png)

  * 將結構描述上傳至儲存體  
  ![](./media/app-service-logic-enterprise-integration-schemas/blob-3.png)

  * 產生結構描述的共用存取簽章  
  ![](./media/app-service-logic-enterprise-integration-schemas/blob-2.png)

  * 選取 [新增結構描述] 中的 [大型檔案]，然後在 [內容 URI] 中提供共用存取簽章 URI  
  ![](./media/app-service-logic-enterprise-integration-schemas/schema-largefile.png)

9. 您應該會看到新加入的結構描述  
  ![](./media/app-service-logic-enterprise-integration-schemas/schema-41.png)

## 如何使用結構描述
- 結構描述可用來驗證在 B2B 案例中交換的訊息。

## 如何編輯結構描述
1. 選取 [結構描述] 圖格
2. 從開啟的 [結構描述] 刀鋒視窗中選取您想要編輯的結構描述
3. 選取 [結構描述] 刀鋒視窗中的 [上傳] 連結  
![](./media/app-service-logic-enterprise-integration-schemas/edit-12.png)
4. 使用開啟的檔案選擇器對話方塊，選取您想要上傳的結構描述檔案。
5. 在檔案選擇器中選取 [開啟]  
![](./media/app-service-logic-enterprise-integration-schemas/edit-31.png)
6. 您會收到通知指出上傳成功

## 如何刪除結構描述
1. 選取 [結構描述] 圖格
2. 從開啟的 [結構描述] 刀鋒視窗中，選取您想要刪除的結構描述
3. 從 [結構描述] 刀鋒視窗功能表列中選取 [刪除] 連結  
![](./media/app-service-logic-enterprise-integration-schemas/delete-12.png)
4. 如果您真的想要刪除所選取的結構描述，選擇 [刪除結構描述] 對話方塊上的 [是] 來確認您的選擇  
![](./media/app-service-logic-enterprise-integration-schemas/delete-21.png)
5. 最後，請注意，[結構描述] 刀鋒視窗中的結構描述清單重會新整理，而且不會再列出您已刪除的結構描述  
![](./media/app-service-logic-enterprise-integration-schemas/delete-31.png)

## 後續步驟

- [深入了解企業整合套件](./app-service-logic-enterprise-integration-overview.md "了解企業整合套件")

<!---HONumber=AcomDC_0921_2016-->