<properties 
	pageTitle="如何設定進階 Azure Redis 快取的資料永續性" 
	description="了解如何設定和管理進階層 Azure Redis 快取執行個體的資料永續性" 
	services="redis-cache" 
	documentationCenter="" 
	authors="steved0x" 
	manager="douge" 
	editor=""/>

<tags 
	ms.service="cache" 
	ms.workload="tbd" 
	ms.tgt_pltfrm="cache-redis" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="09/15/2016" 
	ms.author="sdanie"/>

# 如何設定進階 Azure Redis 快取的資料永續性

Azure Redis 快取有不同的快取服務，在快取大小和功能 (包括新的進階層) 的選擇上提供了彈性。

Azure Redis 快取進階層包括叢集、永續性及虛擬網路支援之類的功能。本文說明如何在進階 Azure Redis 快取執行個體中設定永續性。

如需其他進階快取功能的相關資訊，請參閱 [Azure Redis 快取進階層簡介](cache-premium-tier-intro.md)。

## 資料永續性是什麼？
Redis 永續性可讓您保存儲存在 Redis 中的資料。您也可以擷取快照和備份資料，以在硬體失敗時載入。這是優於基本或標準層的重大優勢，基本或標準層的所有資料是儲存在記憶體中，若發生快取節點故障的失敗，資料可能會遺失。

Azure Redis 快取使用 [RDB 模型](http://redis.io/topics/persistence) (其資料儲存在 Azure 儲存體帳戶中) 提供 Redis 永續性。設定永續性後，Azure Redis 快取會依據可設定的備份頻率，在磁碟中保存一份 Redis 二進位格式的 Redis 快取快照。如果發生同時停用主要和複本快取的災難性事件，即可使用最新的快照重新建構快取。

若要設定永續性，可在建立快取期間從 [**新的 Redis 快取**] 刀鋒視窗設定，也可在現有進階快取的 [**設定**] 刀鋒視窗中設定。

## 建立進階快取

若要建立快取並設定永續性，請登入 [Azure 入口網站](https://portal.azure.com)，然後按一下 [新增] -> [資料 + 儲存體] > [Redis 快取]。

![建立 Redis 快取][redis-cache-new-cache-menu]

若要設定永續性，請先在 [**選擇定價層**] 刀鋒視窗中選取其中一個 [**進階**] 快取。

![選擇定價層][redis-cache-premium-pricing-tier]

選取進階定價層後，按一下 [Redis 永續性]。

![Redis 永續性][redis-cache-persistence]

下節的步驟說明如何在您的新進階快取上設定 Redis 永續性。Redis 永續性設定後，按一下 [**建立**] 以建立具有 Redis 永續性的新進階快取。

## 設定 Redis 永續性

Redis 永續性是在 [**Redis 資料永續性**] 刀鋒視窗中所設定。若為新的快取，則在快取建立程序期間存取此刀鋒視窗，如上節所述。若為現有快取，則從快取的 [**設定**] 刀鋒視窗存取 [**Redis 資料永續性**] 刀鋒視窗。

![Redis 設定][redis-cache-settings]

若要啟用 Redis 永續性，請按一下 [**已啟用**] 來啟用 RDB (Redis 資料庫) 備份。若要停用先前所啟用進階快取的 Redis 永續性，請按一下 [**已停用**]。

若要設定備份間隔，請選取下拉式清單中的 [**備份頻率**]。選項包括 [15 分鐘]、[30 分鐘]、[60 分鐘]、[6 小時]、[12 小時] 及 [24 小]時。在先前的備份作業成功完成後，此間隔便會開始倒數計時，時間過後就會起始新的備份。

按一下 [儲存體帳戶] 選取要使用的儲存體帳戶，然後從 [儲存體金鑰] 下拉式清單中選擇 [主要金鑰] 或 [次要金鑰]。您必須選擇與快取相同區域的儲存體帳戶，建議選取 [進階儲存體] 帳戶，因為進儲存體的輸送量較高。

>[AZURE.IMPORTANT] 如果重新產生了永續性帳戶的儲存體金鑰，您必須從 [儲存體金鑰] 下拉式清單中重新選擇所需的金鑰。

![Redis 永續性][redis-cache-persistence-selected]

按一下 [**確定**] 以儲存永續性組態。

備份頻率間隔過後，會啟動下一個備份 (或新快取的第一個備份)。



## 永續性常見問題集

下列清單包含 Azure Redis 快取永續性常見問題的解答。

-	[可以對先前建立的快取啟用永續性嗎？](#can-i-enable-persistence-on-a-previously-created-cache)
-	[在建立快取之後，可以變更備份頻率嗎？](#can-i-change-the-backup-frequency-after-i-create-the-cache)
-	[為什麼我的備份頻率是 60 分鐘，備份的間隔卻超過 60 分鐘？](#why-if-i-have-a-backup-frequency-of-60-minutes-there-is-more-than-60-minutes-between-backups)
-	[建立新的備份時，舊的備份會發生什麼事？](#what-happens-to-the-old-backups-when-a-new-backup-is-made)
-	[如果我調整為不同大小，並還原為調整作業之前製作的備份時，會發生什麼事？](#what-happens-if-i-have-scaled-to-a-different-size-and-a-backup-is-restored-that-was-made-before-the-scaling-operation)

### 可以對先前建立的快取啟用永續性嗎？

可以，可在建立快取時以及現有進階快取中設定 Redis 永續性。

### 在建立快取之後，可以變更備份頻率嗎？

可以，您可以在 [**Redis 資料永續性**] 刀鋒視窗上變更備份頻率。如需相關指示，請參閱[設定 Redis 永續性](#configure-redis-persistence)。

### 為什麼我的備份頻率是 60 分鐘，備份的間隔卻超過 60 分鐘？

在前一個備份程序順利完成後，備份頻率間隔才會開始計算。如果備份頻率是 60 分鐘，而備份程序要 15 分鐘才能順利完成，則下一次備份要在先前的備份開始的 75 分鐘後才會開始。

### 建立新的備份時，舊的備份會發生什麼事？

除了最新的備份外，所有備份都會自動刪除。這項刪除作業可能不會立即發生，但較舊的備份不會無限期保存。

### 如果我調整為不同大小，並還原為調整作業之前製作的備份時，會發生什麼事？

-	如果您已調整為較大的大小則沒有任何影響。
-	如果已調整為較小的大小，而且您的自訂[資料庫](cache-configure.md#databases)設定大於新大小的[資料庫限制](cache-configure.md#databases)，則不會還原這些資料庫中的資料。如需詳細資訊，請參閱[我的自訂資料庫設定在調整期間會受到影響嗎？](#is-my-custom-databases-setting-affected-during-scaling)
-	如果已調整為較小的大小，而且較小的大小中沒有足夠的空間可保存來自最近備份的所有資料，將會在還原程序中收回金鑰，通常是使用 [allkeys-lru](http://redis.io/topics/lru-cache) 收回原則。

## 後續步驟
了解如何使用更多進階快取功能。

-	[Azure Redis Cache 高階層簡介](cache-premium-tier-intro.md)
  
<!-- IMAGES -->

[redis-cache-new-cache-menu]: ./media/cache-how-to-premium-persistence/redis-cache-new-cache-menu.png

[redis-cache-premium-pricing-tier]: ./media/cache-how-to-premium-persistence/redis-cache-premium-pricing-tier.png

[redis-cache-persistence]: ./media/cache-how-to-premium-persistence/redis-cache-persistence.png

[redis-cache-persistence-selected]: ./media/cache-how-to-premium-persistence/redis-cache-persistence-selected.png

[redis-cache-settings]: ./media/cache-how-to-premium-persistence/redis-cache-settings.png

<!---HONumber=AcomDC_0921_2016-->