<properties
    pageTitle="我在 Azure 自動化中的第一個圖形化 Runbook | Microsoft Azure"
    description="教學課程將逐步引導您建立、測試和發佈簡單的圖形化 Runbook。"
    services="automation"
    documentationCenter=""
    authors="mgoedtel"
    manager="jwhit"
    editor=""
	keywords="runbook, runbook 範本, runbook 自動化, azure runbook"/>
<tags
    ms.service="automation"
    ms.workload="tbd"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.date="07/06/2016"
    ms.author="magoedte;bwren"/>

# 我的第一個圖形化 Runbook

> [AZURE.SELECTOR] - [Graphical](automation-first-runbook-graphical.md) - [PowerShell](automation-first-runbook-textual-PowerShell.md) - [PowerShell Workflow](automation-first-runbook-textual.md)

本教學課程將逐步引導您在 Azure 自動化中建立[圖形化 Runbook](automation-runbook-types.md#graphical-runbooks)。讓我們先從將測試和發佈的簡單 Runbook 開始，同時說明如何追蹤 Runbook 工作的狀態。然後我們要修改 Runbook 以實際上管理 Azure 資源，在此情況下是啟動 Azure 虛擬機器。接著我們要藉由加入 Runbook 參數和條件式連結，讓 Runbook 更穩固。

## 必要條件

若要完成本教學課程，您需要下列項目。

-	Azure 訂用帳戶。如果您沒有這類帳戶，可以[啟用自己的 MSDN 訂戶權益](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/)或<a href="/pricing/free-account/" target="_blank">[註冊免費帳戶](https://azure.microsoft.com/free/)。
-	[Azure 執行身分帳戶](automation-sec-configure-azure-runas-account.md)，用來保存 Runbook 以及向 Azure 資源驗證。此帳戶必須擁有啟動和停止虛擬機器的權限。
-	Azure 虛擬機器。我們將會停止並啟動這台電腦，因此它不應該是生產環境。


## 步驟 1 - 建立新的 Runbook

我們將藉由建立一個輸出文字「Hello World」的簡單 Runbook 開始。

1.	在 Azure 入口網站中，開啟您的自動化帳戶。[自動化帳戶] 頁面提供這個帳戶中資源的快速檢視。您應該已經有一些資產。其中大部分是會自動包含在新自動化帳戶的模組。您應該也擁有[必要條件](#prerequisites)中所述的認證資產。
2.	按一下 [Runbook] 磚以開啟 Runbook 的清單。<br> ![Runbook 控制項](media/automation-first-runbook-graphical/runbooks-control.png)
3.	按一下 [加入 Runbook] 按鈕，然後按一下 [建立新的 Runbook] 來建立新的 Runbook。
4.	為 Runbook 提供名稱「MyFirstRunbook-Graphical」。
5.	在此情況下，我們要建立[圖形化 Runbook](automation-graphical-authoring-intro.md)，因此請選取 [圖形] 作為 [Runbook 類型]。<br> ![新的 Runbook](media/automation-first-runbook-graphical/create-new-runbook.png)<br>
6.	按一下 [建立] 來建立 Runbook 並開啟圖形化編輯器。

## 步驟 2 - 將活動加入至 Runbook

編輯器左邊的 [程式庫] 控制項可讓您選取要加入到 Runbook 的活動。我們將會加入 **Write-Output** Cmdlet 以從 Runbook 輸出文字。

1.	在 [程式庫] 控制項中，按一下搜尋文字方塊並輸入 **Write-Output**。搜尋結果將會顯示在下面。<br> ![Microsoft.PowerShell.Utility](media/automation-first-runbook-graphical/search-powershell-cmdlet-writeoutput.png)
2.	向下捲動到清單底部。您可以用滑鼠右鍵按一下 [Write-Output] 並選取 [加入至畫布]，或按一下 Cmdlet 旁邊的省略符號，然後選取 [加入至畫布]。
3.	按一下畫布上的 **Write-Output** 活動。這會開啟可讓您設定活動的 [組態] 控制項刀鋒視窗。
4.	[標籤] 會預設為 Cmdlet 的名稱，但我們可以將它變更為比較好記的名稱。將它變更為「Write Hello World to output」。
5.	按一下 [參數] 來提供 Cmdlet 的參數值。某些 Cmdlet 有多個參數集，並且您必須選取您要使用的部分。在此情況下，**Write-Output** 有只有一個參數集，因此您不需要選取其中一個。<br> ![Write-Output 屬性](media/automation-first-runbook-graphical/write-output-properties-b.png)
6.	選取 **InputObject** 參數。這是我們將在其中指定要傳送至輸出資料流的文字的參數。
7.	在 [資料來源] 下拉式清單中，選取 [PowerShell 運算式]。[資料來源] 下拉式清單提供您用來填入參數值的不同來源。您可以使用來自這類來源的輸出，例如另一個活動、自動化資產或 PowerShell 運算式。在此情況下，我們只想要輸出文字「Hello World」。我們可以使用 PowerShell 運算式，並指定字串。
8.	在 [運算式] 方塊中，輸入「"Hello World"」，然後按一下 [確定] 兩次以返回畫布。<br> ![PowerShell 運算式](media/automation-first-runbook-graphical/expression-hello-world.png)
9.	按一下 [**儲存**] 來儲存 Runbook。<br> ![儲存 Runbook](media/automation-first-runbook-graphical/runbook-toolbar-save-revised20165.png)

## 步驟 3 - 測試 Runbook

在我們發佈 Runbook 之前，為了使其可用於生產環境，我們想要測試以確定它可以正常運作。測試 Runbook 時，您會執行其**草稿**版本，並以互動方式檢視其輸出。

1.	按一下 [測試窗格] 來開啟 [測試] 刀鋒視窗。<br> ![測試窗格](media/automation-first-runbook-graphical/runbook-toolbar-test-revised20165.png)
2.	按一下 [開始] 以開始測試。這應該是唯一啟用的選項。
3.	隨即會建立 [Runbook 工作](automation-runbook-execution.md)，並在窗格中顯示其狀態。作業狀態一開始會顯示為 [已排入佇列]，表示其正在等候雲端中的 Runbook 背景工作變為可用狀態。然後當背景工作宣告該工作時，狀態將變更為「正在開始」，然後 Runbook 實際開始執行時再變更為「執行中」。
4.	Runbook 工作完成時，會顯示其輸出。在我們的情況中，我們應該會看到「Hello World」。<br> ![Hello World](media/automation-first-runbook-graphical/runbook-test-results.png)
5.	關閉 [測試] 刀鋒視窗返回畫布。

## 步驟 4 - 發佈和啟動 Runbook

我們剛剛建立的 Runbook 仍處於草稿模式。我們需要將它發佈，才能在生產環境中執行它。當您發佈 Runbook 時，您會使用草稿版本覆寫現有的已發佈版本。在我們的情況中，因為我們剛剛建立 Runbook，因此還沒有已發佈版本。

1.	按一下 [**發佈**] 來發佈 Runbook，然後出現提示時按一下 [**是**]。<br> ![發佈](media/automation-first-runbook-graphical/runbook-toolbar-publish-revised20166.png)
2.	如果您向左捲動以檢視 [Runbook] 刀鋒視窗中的 Runbook，它會顯示**已發佈**的**撰寫狀態**。
3.	捲動回右方以檢視 **MyFirstRunbook** 的刀鋒視窗。在頂端的選項可讓我們啟動 Runbook、排程它在未來的某個時間點啟動，或建立 [webhook](automation-webhooks.md)，使得可以透過 HTTP 呼叫啟動它。
4.	我們只想要啟動 Runbook，因此按一下 [**開始**] 然後出現提示時按一下 [**是**]。<br> ![啟動 Runbook](media/automation-first-runbook-graphical/runbook-controls-start-revised20165.png)
5.	作業刀鋒視窗會開啟我們剛剛建立的 Runbook 作業。我們可以關閉此刀鋒視窗，但在此情況下，我們要將它開啟，使得我們可以觀看作業的進度。
6.	[工作摘要] 中會顯示工作狀態，且符合當我們測試 Runbook 時看到的狀態。<br> ![工作摘要](media/automation-first-runbook-graphical/runbook-job-summary.png)
7.	一旦 Runbook 狀態顯示「已完成」，請按一下 [輸出]。[輸出] 刀鋒視窗會開啟，而且可以在窗格中看到我們的「Hello World」。<br> ![工作摘要](media/automation-first-runbook-graphical/runbook-job-output.png)
8.	關閉 [輸出] 刀鋒視窗。
9.	按一下 [所有記錄檔] 以開啟 Runbook 作業的 [資料流] 刀鋒視窗。我們應該只會在輸出資料流中看到「Hello World」，但可能也會顯示 Runbook 工作的其他資料流，例如 Runbook 寫入時發生的詳細資訊和錯誤。<br> ![工作摘要](media/automation-first-runbook-graphical/runbook-job-AllLogs.png)
10.	關閉 [所有記錄檔] 刀鋒視窗和 [作業] 刀鋒視窗以返回 [MyFirstRunbook] 刀鋒視窗。
11.	按一下 [作業] 以開啟此 Runbook 的 [作業] 刀鋒視窗。這樣會列出此 Runbook 所建立的所有工作。由於我們只執行一次作業，因此應該只會看到列出一項作業。<br> ![作業](media/automation-first-runbook-graphical/runbook-control-jobs.png)
12.	您可以按一下此工作以開啟我們啟動 Runbook 時所檢視的相同 [工作] 窗格。這可讓您回到過去的時間並檢視針對特定 Runbook 所建立的任何工作的詳細資料。

## 步驟 5 - 建立變數資產

我們已經測試並發行我們 Runbook，但是到目前為止，它似乎並不實用。我們想要讓它管理 Azure 資源。在設定要驗證的 Runbook 之前，我們會建立變數來保存訂用帳戶識別碼，並在設定要在下面的步驟 6 進行驗證的活動後參考此變數。包括訂用帳戶內容的參考，可讓您輕鬆地處理多個訂用帳戶。在繼續之前，請從 [導覽] 窗格上的 [訂用帳戶] 選項複製您的訂用帳戶識別碼。

1. 在 [自動化帳戶] 刀鋒視窗中按一下 [資產] 圖格，隨即會開啟 [資產] 刀鋒視窗。
2. 在 [資產] 刀鋒視窗中，按一下 [變數] 圖格。
3. 在 [變數] 刀鋒視窗中，按一下 [加入變數]。<br>![自動化變數](media/automation-first-runbook-graphical/create-new-subscriptionid-variable.png)
4. 在 [新增變數] 刀鋒視窗中，於 [名稱] 方塊中輸入 **AzureSubscriptionId**，並於 [值] 方塊中輸入您的訂用帳戶識別碼。讓 [類型] 保持是 [字串]，並讓 [加密] 保持預設值。
5. 按一下 [建立] 來建立變數。


## 步驟 6 - 加入驗證來管理 Azure 資源

現在我們已經擁有用來保存訂用帳戶識別碼的變數，接下來我們可以設定 Runbook，以使用[必要條件](#prerequisites)中提到的「執行身分」認證來進行驗證。方法是將 [Azure 執行身分] 連線 [資產] 和 [Add-AzureRMAccount] Cmdlet可加入至畫布。

1.	按一下 [MyFirstRunbook] 刀鋒視窗上的 [編輯] 來開啟圖形化編輯器。<br> ![編輯 Runbook](media/automation-first-runbook-graphical/runbook-controls-edit-revised20165.png)
2.	我們不再需要 [撰寫 Hello World 到輸出]，因此以滑鼠右鍵按一下它並選取 [刪除]。
3.	在 [程式庫] 控制項中，展開 [連線]，並選取 [加入至畫布] 以將 [AzureRunAsConnection] 加入至畫布。
4.	在畫布上選取 [Get-AutomationVariable]，然後在 [組態] 控制項窗格的 [標籤] 文字方塊中輸入 [取得執行身分連線]。這是連線
5.	在 [程式庫] 控制項中，於搜尋文字方塊中輸入 **Add-AzureRmAccount**。
6.	將 [Add-AzureRmAccount] 加入至畫布。<br> ![Add-AzureRMAccount](media/automation-first-runbook-graphical/search-powershell-cmdlet-addazurermaccount.png)
7.	將滑鼠停留在 [取得執行身分連線]，直到圖形的底端出現圓形。按一下圓形，並將箭頭拖曳到 [Add-AzureRmAccount]。您剛才建立的箭號是*連結*。Runbook 將會開始使用 [取得執行身分連線]，然後執行 [Add-AzureRmAccount]。<br> ![建立活動之間的連結](media/automation-first-runbook-graphical/runbook-link-auth-activities.png)
8.	在畫布上選取 [Add-AzureRmAccount]，然後在 [組態] 控制項窗格的 [標籤] 文字方塊中輸入 [登入 Azure]。
9.	按一下 [參數]，隨即會顯示 [活動參數組態] 刀鋒視窗。
10.  [Add-AzureRmAccount] 有多個參數集，因此我們必須先選取一個參數，我們才可以提供參數值。按一下 [參數集]，然後選取 [ServicePrincipalCertificate] 參數集。
11.  一旦您選取參數集，參數會在 [活動參數組態] 刀鋒視窗中顯示。按一下 [APPLICATIONID]。<br> ![加入 Azure RM 帳戶參數](media/automation-first-runbook-graphical/add-azurermaccount-parameterset.png)
12.  在 [參數值] 刀鋒視窗中，為 [資料來源] 選取 [活動輸出] 並從清單中選取 [取得執行身分連線]，在 [欄位路徑] 文字方塊中輸入 **ApplicationId**，然後按一下 [確定]。我們會指定欄位路徑之屬性的名稱，因為活動會輸出具有多個屬性的物件。
13.  按一下 [CERTIFICATETHUMBPRINT]，然後在 [參數值] 刀鋒視窗中，為 [資料來源] 選取 [活動輸出]。從清單中選取 [取得執行身分連線]，在 [欄位路徑] 文字方塊中輸入 **CertificateThumbprint**，然後按一下 [確定]。
14.  按一下 [SERVICEPRINCIPAL] 並在 [參數值] 刀鋒視窗中，為 [資料來源] 選取 [ConstantValue]，按一下選項 [True]，然後按一下 [確定]。
15.  按一下 [TENANTID]，然後在 [參數值] 刀鋒視窗中，為 [資料來源] 選取 [活動輸出]。從清單中選取 [取得執行身分連線]，在 [欄位路徑] 文字方塊中輸入 **TenantId**，然後按兩次 [確定]。
16.  在 [程式庫] 控制項中，於搜尋文字方塊中輸入 **Set-AzureRmContext**。
17.	 將 [Set-AzureRmContext] 加入至畫布。
18.	 在畫布上選取 [Set-AzureRmContext]，然後在 [組態] 控制項窗格的 [標籤] 文字方塊中輸入 [指定訂用帳戶識別碼]。
19.	 按一下 [參數]，隨即會顯示 [活動參數組態] 刀鋒視窗。
20. [Set-AzureRmContext] 有多個參數集，因此我們必須先選取一個參數，我們才可以提供參數值。按一下 [參數集]，然後選取 [SubscriptionId] 參數集。
21.	 一旦您選取參數集，參數會在 [活動參數組態] 刀鋒視窗中顯示。按一下 [SubscriptionID]
22.	 在 [參數值] 刀鋒視窗中，為 [資料來源] 選取 [變數資產]，接著在清單中選取 [AzureSubscriptionId]，然後按兩次 [確定]。
23.  將滑鼠停留在 [登入 Azure]，直到圖形的底端出現圓形。按一下圓形，並將箭頭拖曳到 [指定訂用帳戶識別碼]。


此時您的 Runbook 看起來應該像下面這樣︰<br>![Runbook 驗證組態](media/automation-first-runbook-graphical/runbook-auth-config.png)

## 步驟 7 - 加入活動以啟動虛擬機器

現在我們要加入 **Start-AzureRmVM** 活動，以啟動虛擬機器。您可以在 Azure 訂用帳戶中挑選任何虛擬機器，而現在我們會將該名稱硬式編碼成 Cmdlet。

1. 在 [程式庫] 控制項中，於搜尋文字方塊中輸入 **Start-AzureRm**。
2. 將 [Start-AzureRmVM] 新增至畫布，然後對它按一下並拖曳到 [指定訂用帳戶識別碼] 底下。
3. 將滑鼠停留在 [指定訂用帳戶識別碼]，直到圖形的底端出現圓形。按一下圓形，並將箭頭拖曳到 [Start-AzureRmVM]。
4.	選取 [Start-AzureRmVM]。按一下 [參數] 然後按一下 [參數集] 以檢視 [Start-AzureRmVM] 的參數集。選取 [ResourceGroupNameParameterSetName] 參數集。請注意，[ResourceGroupName] 和 [名稱] 旁邊具有驚嘆號。這表示它們是必要的參數。也請注意這兩者應該是字串值。
5.	選取 [**名稱**]。在 [資料來源] 中選取 [PowerShell 運算式]，並輸入用雙引號括住的虛擬機器名稱，此為我們要用此 Runbook 啟動的虛擬機器。按一下 [確定]。<br>![Start-AzureRmVM 名稱參數值](media/automation-first-runbook-graphical/runbook-startvm-nameparameter.png)
6.	選取 [ResourceGroupName]。為 [資料來源] 使用 [PowerShell 運算式]，並輸入用雙引號括住的資源群組名稱。按一下 [**確定**]。<br> ![Start-AzureRmVM 參數](media/automation-first-runbook-graphical/startazurermvm-params.png)
8.	按一下 [測試] 窗格，如此我們可以測試 Runbook。
9.	按一下 [**開始**] 以開始測試。當它完成時，請檢查虛擬機器已啟動。

此時您的 Runbook 看起來應該像下面這樣︰<br>![Runbook 驗證組態](media/automation-first-runbook-graphical/runbook-startvm.png)

## 步驟 8 - 將其他輸入參數加入至 Runbook

我們的 Runbook 目前會啟動在 **Start-AzureRmVM** Cmdlet 中指定之資源群組內的虛擬機器，但如果可以在啟動 Runbook 時指定這兩項，我們的 Runbook 會更有用。我們現在會將輸入參數加入 Runbook，以提供該功能。

1. 按一下 **MyFirstRunbook** 窗格上的 [**編輯**] 來開啟圖形化編輯器。
2. 按一下 [**輸入和輸出**] 然後按一下 [**加入輸入**] 來開啟 [Runbook 輸入參數] 窗格。<br> ![Runbook 輸入和輸出](media/automation-first-runbook-graphical/runbook-toolbar-InputandOutput-revised20165.png)
3. 對 [**名稱**] 指定 *VMName*。將 [**類型**] 保留為*string*，但將 [**強制**] 變更為 [*是*]。按一下 [確定]。
4. 建立稱為「ResourceGroupName」的第二個必要輸入參數，然後按一下 [確定] 來關閉 [輸入和輸出] 窗格。<br> ![Runbook 輸入參數](media/automation-first-runbook-graphical/start-azurermvm-params-outputs.png)
5. 選取 [Start-AzureRmVM] 活動，然後按一下 [參數]。
6. 將 [名稱] 的 [資料來源] 變更為 [Runbook 輸入]，然後選取 [VMName]。<br>
7. 將 [ResourceGroupName] 的 [資料來源] 變更為 [Runbook 輸入]，然後選取 [ResourceGroupName]。<br> ![Start-AzureVM 參數](media/automation-first-runbook-graphical/start-azurermvm-params-runbookinput.png)
8. 儲存 Runbook 並開啟 [測試] 窗格。請注意，您現在可以提供測試中將使用的兩個輸入變數的值。
9. 關閉 [測試] 窗格。
10.	按一下 [發佈] 來發行新版本的 Runbook。
11.	停止您在上一個步驟中啟動的虛擬機器。
12.	按一下 [**開始**] 以啟動 Runbook。輸入您要啟動之虛擬機器的 [VMName] 和 [ResourceGroupName]。<br> ![啟動 Runbook](media/automation-first-runbook-graphical/runbook-start-inputparams.png)
13.	Runbook 完成時，請檢查虛擬機器已啟動。

## 步驟 9 - 建立條件式連結

我們現在將修改 Runbook，讓它只會在未啟動時嘗試啟動虛擬機器。我們透過將 **Get-AzureRmVM** Cmdlet 加入到將取得虛擬機器之執行個體層級狀態的 Runbook，來執行此動作。然後，我們將加入名為**取得狀態**的 PowerShell 工作流程程式碼模組與 PowerShell 程式碼片段來判斷虛擬機器的狀態是正在執行還是已停止。只有當目前的執行狀態為已停止時，**取得狀態**模組中的條件式連結才會執行 **Start-AzureRmVM**。最後我們會輸出訊息，以通知您 VM 已成功啟動或未使用 PowerShell Write-Output Cmdlet。

1. 在圖形化編輯器中開啟 **MyFirstRunbook**。
2. 移除 [指定訂用帳戶識別碼] 和 [Start-AzureRmVM] 之間的連結，方法是按一下該連結，然後按 [刪除] 鍵。
3. 在 [程式庫] 控制項中，於搜尋文字方塊中輸入 **Get-AzureRm**。
4. 將 [Get-AzureRmVM] 加入至畫布。
5. 選取 [Get-AzureRmVM] 然後選取 [參數集] 以檢視 [Get-AzureRmVM] 的參數集。選取 [GetVirtualMachineInResourceGroupNameParamSet] 參數集。請注意，[ResourceGroupName] 和 [名稱] 旁邊具有驚嘆號。這表示它們是必要的參數。也請注意這兩者應該是字串值。
6. 在 [名稱] 的 [資料來源] 底下，選取 [Runbook 輸入]，然後選取 [VMName]。按一下 [確定]。
7. 在 [ResourceGroupName] 的 [資料來源] 底下，選取 [Runbook 輸入]，然後選取 [ResourceGroupName]。按一下 [確定]。
8. 在 [狀態] 的 [資料來源] 底下，選取 [常數值]，然後按一下 [True]。按一下 [確定]。
9. 建立從 [指定訂用帳戶識別碼] 至 [Get-AzureRmVM] 的連結。
10. 在 [程式庫] 控制項中展開 [Runbook 控制項]，並將 [程式碼] 加入至畫布。
11. 建立從 [Get-AzureRmVM] 至 [程式碼] 的連結。
12. 按一下 [程式碼]，接著在 [組態] 窗格中將標籤變更為 [取得狀態]。
13. 選取 [程式碼] 參數，隨即出現 [程式碼編輯器] 刀鋒視窗。
14. 在程式碼編輯器中，貼上下列程式碼片段︰

     ```
     $StatusesJson = $ActivityOutput['Get-AzureRmVM'].StatusesText 
     $Statuses = ConvertFrom-Json $StatusesJson 
     $StatusOut ="" 
     foreach ($Status in $Statuses){ 
     if($Status.Code -eq "Powerstate/running"){$StatusOut = "running"} 
     elseif ($Status.Code -eq "Powerstate/deallocated") {$StatusOut = "stopped"} 
     } 
     $StatusOut 
     ```

15. 建立從 [取得狀態] 至 [Start-AzureRmVM] 的連結。<br> ![程式碼模組的 Runbook](media/automation-first-runbook-graphical/runbook-startvm-get-status.png)
16. 選取該連結，然後在 [組態] 窗格中將 [套用條件] 變更為 [是]。請注意，連結轉變為虛線表示目標活動只有在條件解析為 true 時才會執行。
17. 在 [條件運算式] 中輸入「$ActivityOutput['Get Status'] -eq "Stopped"」。**Start-AzureRmVM** 現在只會在虛擬機器停止時執行。
18.	在 [程式庫] 控制項中，展開 **Cmdlet**，然後展開 **Microsoft.PowerShell.Utility**。
19.	將 [Write-Output] 加入至畫布兩次。<br> ![Write-Output 的 Runbook](media/automation-first-runbook-graphical/runbook-startazurermvm-complete.png)
20. 在第一個 [Write-Output] 控制項中按一下 [參數]，然後將 [標籤] 值變更為 [通知 VM 已啟動]。
21. 在 [InputObject] 中將 [資料來源] 變更為 [PowerShell 運算式]，然後輸入運算式 "$VMName successfully started"。
22. 在第二個 [Write-Output] 控制項中按一下 [參數]，然後將 [標籤] 值變更為 [通知 VM 啟動失敗]
23. 在 [InputObject] 中將 [資料來源] 變更為 [PowerShell 運算式]，然後輸入運算式 "$VMName could not start"。
24. 建立從 [Start-AzureRmVM] 至 [通知 VM 已啟動] 和 [通知 VM 啟動失敗] 的連結。
25. 選取連往 [通知 VM 已啟動] 的連結並將 [套用條件] 變更為 [True]。
26. 在 [條件運算式] 中輸入 $ActivityOutput['Start-AzureRmVM'].IsSuccessStatusCode -eq $true。此 Write-Output 控制項現在只會在虛擬機器已成功啟動時執行。
27. 選取連往 [通知 VM 啟動失敗] 的連結並將 [套用條件] 變更為 [True]。
28. 在 [條件運算式] 中輸入 $ActivityOutput['Start-AzureRmVM'].IsSuccessStatusCode -ne $true。此 Write-Output 控制項現在只會在虛擬機器未成功啟動時執行。
29.	儲存 Runbook 並開啟 [測試] 窗格。
30.	在虛擬機器停止下啟動 Runbook，它應該會啟動。

## 後續步驟

-	若要深入了解圖形化編寫，請參閱 [Azure 自動化中的圖形化編寫](automation-graphical-authoring-intro.md)
-	若要開始使用 PowerShell Runbook，請參閱[我的第一個 PowerShell Runbook](automation-first-runbook-textual-powershell.md)
-	若要開始使用 PowerShell 工作流程 Runbook，請參閱[我的第一個 PowerShell 工作流程 Runbook](automation-first-runbook-textual.md)

<!---HONumber=AcomDC_0928_2016-->