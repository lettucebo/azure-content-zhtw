<properties 
    pageTitle="事件中樞常見問題集 (FAQ) | Microsoft Azure"
    description="事件中樞常見問題集。"
    services="event-hubs"
    documentationCenter="na"
    authors="sethmanheim"
    manager="timlt"
    editor="" />
<tags 
    ms.service="event-hubs"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="09/01/2016"
    ms.author="sethm" />

# 事件中樞常見問題集

事件中樞提供大規模攝取量、持續性，以及從高輸送量資料來源和/或數百萬台裝置處理資料事件的能力。與服務匯流排佇列和主題配對後，事件中樞可在[物聯網 (IoT)](https://azure.microsoft.com/services/iot-hub/) 案例中實現持續的命令和控制部署。

本文討論有關事件中樞的價格資訊，並提供一些常見問題的解答：

## 定價資訊

如需事件中樞定價的完整資訊，請參閱[事件中樞定價詳細資料](https://azure.microsoft.com/pricing/details/event-hubs/)。

## 事件中樞輸入事件的計算方式為何？

每個傳送到事件中樞的事件都算是可計費訊息。*輸入事件*的定義為小於或等於 64 KB 的資料單位。任何大小小於或等於 64 KB 的事件均視為一個可計費事件。如果事件大於 64 KB，可計費事件的數目乃根據事件大小來計算 (64 KB 的倍數)。例如，一個傳送到事件中樞的 8KB 事件將視為一個事件來計費，不過一則傳送到事件中樞的 96KB 訊息則視為兩個事件來計費。

自事件中樞取用的事件，以及管理作業和控制呼叫 (如檢查點)，不會計入可計費輸入事件，但會累積在輸送量單位額度內。

## 事件中樞輸送量單位是什麼？

您明確選取事件中樞輸送量單位，可以者透過 Azure 入口網站或事件中樞 Resource Manager 範本。輸送量單位會套用到事件中樞命名空間內的所有事件中樞，以及賦予命名空間下列功能的每個輸送量單位：

- 最高每秒 1MB 的輸入事件 (傳送到事件中樞的事件)，但不超過每秒 1000 個輸入事件、管理作業或控制 API 呼叫。

- 最高每秒 2MB 的輸出事件 (從事件中樞取用的事件)。

- 最多 84GB 的事件儲存體 (足以應付預設的 24 小時保留期限)。

事件中樞輸送量單位以小時計費，基準為在指定時段內選取之單位的最大數目。

## 事件中樞的輸送量單位限制如何實施？

如果命名空間內所有事件中樞的輸入輸送量總計或輸入事件率總計超過彙總輸送量單位額度，傳送者將遭受節流處置並會收到錯誤，指出已超過輸入配額。

如果命名空間內所有事件中樞的輸出輸送量總計或輸出事件率總計超過彙總輸送量單位額度，接收者將遭受節流處置並會收到錯誤，指出已超過輸出配額。輸入和輸出配額採單獨實施，如此一來，傳送者無法使事件取用速率變慢，接收者也不能阻止事件傳送到事件中樞。

請注意，輸送量單位的選擇與事件中樞的資料分割數目無關。雖然每個資料分割提供每秒 1MB 的最大輸入輸送量 (每秒最多 1000 個事件)，以及每秒 2MB 的最大輸出輸送量，不過資料分割本身並沒有固定的費用。需要付費的是事件中樞命名空間內所有事件中樞的彙總輸送量單位。透過此模式，除非系統上的事件負載確實需要更高額的輸送量，否則您可以建立足夠的資料分割來支援系統的預期最大負載，而不會產生任何輸送量單位費用，也不需要隨著系統上的負載增加來變更系統結構和架構。

## 可供選取的輸送量單位數目是否有限制？

每個命名空間的預設配額為 20 個輸送量單位。您可以藉由提出支援票證來要求較大的輸送量單位配額。除了 20 個輸送量單位的限制之外，還有 20 和 100 個輸送量單位的組合。請注意，使用 20 個以上的輸送量單位會排除不需要提出支援票證即可變更輸送量單位數目的能力。

## 保留事件中樞事件超過 24 小時需要計費嗎？

事件中樞標準層允許 24 小時以上的訊息保留期間，最多達 30 天。如果儲存之事件總數的大小超過選定輸送量單位數目的儲存額度 (每個輸送量單位 84 GB)，超過額度的大小將以已發佈的 Azure Blob 儲存體費率計費。即使您已將輸送量單位的最大輸入額度用盡，每個輸送量單位的儲存額度依然涵蓋 24 小時保留期間 (預設值) 的所有儲存費用。

## 最大保留期間是什麼？

事件中樞標準層目前支援的最大保留期間為 7 天。請注意，事件中樞的立意並非做為永久的資料存放區。大於 24 小時的保留期間乃專為方便地在同一系統上重新執行事件串流的案例而設計。例如，根據現有資料來訓練或驗證新機器學習模型。

## 事件中樞儲存空間大小如何計算及收費？

儲存在所有事件中樞內之事件，包括事件標頭的所有內部負荷或磁碟儲存結構的所有內部負荷，是以全天為單位來測量。我們會在一天結束時計算儲存空間大小峰值。每日儲存額度的計算乃以當天選定之輸送量單位的最小數目為基準 (每個輸送量單位提供 84GB 的額度)。如果總大小超過計算出來的每日儲存額度，我們會採用 Azure Blob 儲存體費率來計算超出的儲存空間 (依照**本地備援儲存體**費率)。

## 我可以使用一個 AMQP 連線在事件中樞與服務匯流排佇列/主題之間傳送及接收嗎？

可以。只要所有事件中樞、佇列和主題都位於相同的命名空間內即可。因此，您可以在延遲時間少於 1 秒的情況下，使用符合成本效益且具有高可調整性的方式實作雙向代理連線來連接許多裝置。

## 代理連線費用適用於事件中樞嗎？

對傳送者來說，只有在使用 AMQP 通訊協定時才需要支付連線費用。不論傳送系統或裝置的數目多寡，使用 HTTP 來傳送事件不需要支付連線費用。如果您打算使用 AMQP (例如，為了實現更有效率的事件串流，或針對 IoT 命令和控制案例啟用雙向通訊)，請參閱[服務匯流排定價資訊](https://azure.microsoft.com/pricing/details/service-bus/)頁面，以取得構成代理連線之元素和其計量方式的資訊。

## 事件中樞基本層和標準層之間的差異為何？

事件中樞標準層提供事件中樞基本層和一些競爭系統未提供的功能。這些功能包括超過 24 小時的保留期間，以及在延遲時間少於一秒的情況下使用單一 AMQP 連線將命令傳送到大量裝置，外加從這些裝置將遙測傳送到事件中樞等能力。如需功能清單，請參閱[事件中樞定價詳細資訊](https://azure.microsoft.com/pricing/details/event-hubs/)。

## 各地區上市情況

事件中樞可在以下區域取得：

|地理區域|區域|
|---|---|
|美國|美國中部、美國東部、美國東部 2、美國中南部、美國西部|
|歐洲|北歐、西歐|
|亞太地區|東亞、東南亞|
|日本|日本東部、日本西部|
|巴西|巴西南部|
|澳大利亞|澳大利亞東部、澳大利亞東南部|

## 支援與 SLA

事件中樞的技術支援可透過[社群論壇](https://social.msdn.microsoft.com/forums/azure/home)取得。計費及訂用帳戶管理支援均為免費提供。

若要深入了解 SLA，請參閱[服務等級協定](https://azure.microsoft.com/support/legal/sla/)頁面。

## 後續步驟

若要深入了解事件中樞，請參閱下列文章：

- [事件中樞概觀][]。
- [使用事件中樞的完整範例應用程式][]。
- 使用服務匯流排佇列的[佇列訊息解決方案][]。

[事件中樞概觀]: event-hubs-overview.md
[使用事件中樞的完整範例應用程式]: https://code.msdn.microsoft.com/Service-Bus-Event-Hub-286fd097
[佇列訊息解決方案]: ../service-bus/service-bus-dotnet-multi-tier-app-using-service-bus-queues.md

<!----HONumber=AcomDC_0907_2016-->