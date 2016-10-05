<properties
   pageTitle="資料列層級安全性與 Power BI Embedded"
   description="資料列層級安全性與 Power BI Embedded 的詳細資料"
   services="power-bi-embedded"
   documentationCenter=""
   authors="mgblythe"
   manager="NA"
   editor=""
   tags=""/>
<tags
   ms.service="power-bi-embedded"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="powerbi"
   ms.date="07/05/2016"
   ms.author="mblythe"/>

# Power BI Embedded 的資料列層級安全性

資料列層級安全性 (RLS) 可以用來限制使用者對於報告或資料集內特定資料的存取，讓多個不同使用者在查看不同資料的同時，能夠使用相同的報告。Power BI Embedded 現在支援使用 RLS 設定資料集。

![](media\power-bi-embedded-rls\pbi-embedded-rls-flow-1.png)

為了利用 RLS，了解三個主要概念很重要：使用者、角色和規則。讓我們仔細看看每個概念：

**使用者** – 這是檢視報告的實際使用者。在 Power BI Embedded 中，使用者是依應用程式權杖中的使用者名稱屬性來識別。

**角色** – 使用者所屬角色。角色是規則的容器，並可命名為類似「業務經理」或「業務代表」的項目。在 Power BI Embedded 中，使用者是依應用程式權杖中的角色屬性來識別。

**規則** – 角色有規則，這些規則是要套用至資料的實際篩選器。可以像 “Country = USA” 一樣簡單，或是更動態的項目。

### 範例

對於這篇文章的其餘部分，我們將提供撰寫 RLS，然後在內嵌應用程式中使用的範例。我們的範例會使用[零售分析範例](http://go.microsoft.com/fwlink/?LinkID=780547) PBIX 檔案。

![](media\power-bi-embedded-rls\pbi-embedded-rls-scenario-2.png)

我們的零售分析範例會顯示特定零售鏈中所有商店的銷售額。沒有 RLS，不論哪一個區域的經理登入及檢視報告，都會看到相同的資料。資深管理階層決定每個區域經理只能看到他們所管理的商店的銷售額，為了達成這個目的，我們可以使用 RLS。

RLS 是在 Power BI Desktop 中撰寫。當開啟資料集和報告時，我們可以切換至圖表檢視來查看結構描述︰

![](media\power-bi-embedded-rls\pbi-embedded-rls-diagram-view-3.png)

以下是一些對於此結構描述要注意的事項：

-	所有量值，例如**總銷售額**，是儲存在**銷售**事實資料表。
-	有四個額外的相關維度資料表︰**項目**、**時間**、**商店** 和 **區域**。
-	關聯線的箭號表示篩選條件可以從一個資料表流向另一個資料表的方向。例如，如果篩選條件置於目前結構描述中的**時間[日期]**，它只會往下篩選**銷售**資料表中的值。其他資料表都不會受到此篩選條件的影響，因為關聯線的所有箭號都指向銷售資料表，不會指向其他方向。
-	**區域**資料表表示誰是每個區域的經理︰

    ![](media\power-bi-embedded-rls\pbi-embedded-rls-district-table-4.png)

根據此結構描述，如果我們將篩選條件套用至區域資料表中的 [區域經理] 資料行，且如果該篩選條件符合檢視報告的使用者，則該篩選條件也會往下篩選**商店**和**銷售**資料表，只顯示該特定區域經理的資料。

方式如下：

1.	在 [模型] 索引標籤中，按一下 [管理角色]。![](media\power-bi-embedded-rls\pbi-embedded-rls-modeling-tab-5.png)

2.	建立新的角色，稱為 [經理]。![](media\power-bi-embedded-rls\pbi-embedded-rls-manager-role-6.png)

3.	在 [區域] 資料表中輸入下列 DAX 運算式︰**[District Manager] = USERNAME()** ![](media\power-bi-embedded-rls\pbi-embedded-rls-manager-role-7.png)

4.	若要確保規則都能運作，在 [模型] 索引標籤上，按一下 [以角色身分檢視]，然後輸入下列項目︰![](media\power-bi-embedded-rls\pbi-embedded-rls-view-as-roles-8.png)

    報告隨即會顯示資料，如同您已登入為 **Andrew Ma**。

像我們在這裡所做的一樣套用篩選條件，將會往下篩選**區域**、**商店**和**銷售**資料表中的所有記錄。不過，由於**銷售**和**時間**之間的關聯的篩選方向，**銷售**和**項目**與**項目**和**時間**資料表將不會向下篩選。

![](media\power-bi-embedded-rls\pbi-embedded-rls-diagram-view-9.png)

對於此需求可能沒問題，但是，如果我們不想要讓經理查看他們沒有任何銷售的項目，我們可以針對關聯性開啟雙向交叉篩選，讓安全性篩選條件同時流向兩個方向。這可以藉由編輯**銷售**和**項目**之間的關聯性來完成，如下所示：

![](media\power-bi-embedded-rls\pbi-embedded-rls-edit-relationship-10.png)

現在，篩選條件可以從銷售資料表流向**項目**資料表：

![](media\power-bi-embedded-rls\pbi-embedded-rls-diagram-view-11.png)

**注意** 如果您針對資料使用 DirectQuery 模式，您必須啟用雙向交叉篩選，方法是選取這兩個選項︰

1.	[檔案] -> [選項和設定] -> [預覽功能] -> [針對 DirectQuery 啟用兩個方向的交叉篩選].
2.	[檔案] -> [選項和設定] -> [DirectQuery] -> [允許 DirectQuery 模式中的不受限制量值].


若要深入了解雙向交叉篩選，下載 [SQL Server Analysis Services 2016 和 Power BI Desktop 中的雙向交叉篩選](http://download.microsoft.com/download/2/7/8/2782DF95-3E0D-40CD-BFC8-749A2882E109/Bidirectional cross-filtering in Analysis Services 2016 and Power BI.docx) 白皮書。

這會包裝 Power BI Desktop 中需要完成的所有工作，但還有一件工作需要完成，讓我們定義的 RLS 規則在 Power BI Embedded 中運作。使用者是由您的應用程式和應用程式權杖驗證和授權，應用程式和應用程式權杖是用來授與使用者對於特定 Power BI Embedded 報告的存取權。Power BI Embedded 對於您的使用者是誰，並沒有任何特定資訊。如果要讓 RLS 運作，您需要將一些額外的內容傳遞做為您的應用程式權杖的一部分︰
-	**username** (選擇性) – 與 RLS 搭配使用，這是字串，可以在套用 RLS 規則時用來協助識別使用者。請參閱「搭配使用資料列層級安全性和 Power BI Embedded」
-	**角色** – 字串，包含套用資料列層級安全性規則時要選取的角色。如果傳遞多個角色，應該將它們傳遞為字串陣列。

如果 username 屬性存在，則您也必須在角色中傳遞至少一個值。

完整的應用程式權杖看起來如下：

![](media\power-bi-embedded-rls\pbi-embedded-rls-app-token-string-12.png)

現在，所有項目都聚合在一起，當有人登入我們的應用程式以檢視此報告，他們將只能夠看到他們獲得允許可以看到的資料，如同我們的資料列層級安全性所定義。

![](media\power-bi-embedded-rls\pbi-embedded-rls-dashboard-13.png)

## 另請參閱
[資料列層級安全性 (RLS) 與 Power](https://powerbi.microsoft.com/zh-TW/documentation/powerbi-admin-rls/)

<!----HONumber=AcomDC_0907_2016-->