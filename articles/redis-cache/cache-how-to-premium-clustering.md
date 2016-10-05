<properties 
	pageTitle="如何設定進階 Azure Redis 快取的 Redis 叢集 | Microsoft Azure" 
	description="了解如何建立和管理進階層 Azure Redis 快取執行個體的 Redis 叢集" 
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

# 如何設定進階 Azure Redis 快取的 Redis 叢集
Azure Redis 快取有不同的快取服務，在快取大小和功能 (包括新的進階層) 的選擇上提供了彈性。

Azure Redis 快取進階層包括叢集、永續性及虛擬網路支援之類的功能。本文說明如何在進階 Azure Redis 快取執行個體中設定叢集。

如需其他進階快取功能的相關資訊，請參閱 [Azure Redis 快取進階層簡介](cache-premium-tier-intro.md)。

## Redis 叢集是什麼？
Azure Redis 快取提供 Redis 叢集的方式，就像[實作於 Redis](http://redis.io/topics/cluster-tutorial) 一樣。使用 Redis 叢集可以獲得下列好處：

-	能夠自動分割您在多個節點之間的資料集。
-	當節點的子集發生故障或無法與叢集的其餘部分通訊時，可以繼續作業。
-	更多輸送量：當您增加分區數目時，輸送量會呈線性增加。
-	更多記憶體大小：當您增加分區數目時，會呈線性增加。

如需高階快取的大小、輸送量和頻寬等方面的詳細資訊，請參閱 [Azure Redis 快取常見問題集](cache-faq.md#what-redis-cache-offering-and-size-should-i-use)。

在 Azure 中，Redis 叢集以主要/複本模型方式提供，其中的每個分區都有一個具複寫功能的主要/複本組，而複寫是由 Azure Redis 快取服務管理。

## 叢集
叢集是在快取建立期間於 [新的 Redis 快取] 刀鋒視窗中所啟用。

[AZURE.INCLUDE [redis-cache-create](../../includes/redis-cache-premium-create.md)]

叢集是在 [**Redis 叢集**] 刀鋒視窗中所設定。

![叢集][redis-cache-clustering]

叢集中最多可包含 10 個分區。按一下 [**已啟用**]，針對 [**分區計數**]，滑動滑桿或鍵入一個 1 到 10 之間的數字，然後按一下 [**確定**]。

每個分區都是一個由 Azure 管理的主要/複本快取組，快取總大小的計算方式，是將分區數目乘以在定價層中選取的快取大小。

![叢集][redis-cache-clustering-selected]

建立快取後，您可以連接並使用它，就像非叢集化快取一樣，而且 Redis 將會在整個快取分區散發資料。如果[已啟用](cache-how-to-monitor.md#enable-cache-diagnostics)診斷，則會針對每個分區個別擷取度量，而且可以在 [Redis 快取] 刀鋒視窗中[檢視](cache-how-to-monitor.md)。

如需搭配 StackExchange.Redis 用戶端使用叢集的範例程式碼，請參閱 [Hello World](https://github.com/rustd/RedisSamples/tree/master/HelloWorld) 範例的 [clustering.cs](https://github.com/rustd/RedisSamples/blob/master/HelloWorld/Clustering.cs) 部分。

<a name="cluster-size"></a>
## 在執行中的進階快取上變更叢集大小

若要從已啟用叢集的執行中進階快取變更叢集大小，可從 [設定] 刀鋒視窗中按一下 [(預覽) Redis 叢集大小]。

>[AZURE.NOTE] 請注意，雖然 Azure Redis 快取進階層已發行正式上市版，但 Redis 叢集大小功能目前為預覽狀態。

![Redis 叢集大小][redis-cache-redis-cluster-size]

若要變更叢集大小，請使用滑桿，或在 [分區計數] 文字方塊中輸入 1 到 10 之間的數字，然後按一下 [確定] 加以儲存。

## 叢集常見問題集

下列清單包含 Azure Redis 快取叢集常見問題的解答。

-	[我需要對我的用戶端應用程式進行任何變更才能使用叢集嗎？](#do-i-need-to-make-any-changes-to-my-client-application-to-use-clustering)
-	[如何在叢集中散發索引鍵？](#how-are-keys-distributed-in-a-cluster)
-	[我可以建立的最大快取大小為何？](#what-is-the-largest-cache-size-i-can-create)
-	[所有 Redis 用戶端都支援叢集嗎？ ](#do-all-redis-clients-support-clustering)
-	[啟用叢集後，要如何連接到我的快取？](#how-do-i-connect-to-my-cache-when-clustering-is-enabled)
-	[我可以直接連接到我的快取的個別分區嗎？](#can-i-directly-connect-to-the-individual-shards-of-my-cache)
-	[我可以為先前建立的快取設定叢集嗎？](#can-i-configure-clustering-for-a-previously-created-cache)
-	[我可以設定基本或標準快取的叢集嗎？](#can-i-configure-clustering-for-a-basic-or-standard-cache)
-	[我可以將叢集使用於 Redis ASP.NET 工作階段狀態和輸出快取提供者嗎？](#can-i-use-clustering-with-the-redis-aspnet-session-state-and-output-caching-providers)
-	[當我使用 StackExchange.Redis 和叢集時收到 MOVE 例外狀況，該怎麼辦？](#i-am-getting-move-exceptions-whzh-TWing-stackexchangeredis-and-clustering-what-should-i-do)

### 我需要對我的用戶端應用程式進行任何變更才能使用叢集嗎？

-	啟用叢集時，只可以使用資料庫 0。如果用戶端應用程式使用多個資料庫，並嘗試讀取或寫入至 0 以外的資料庫，就會擲回下列例外狀況。`Unhandled Exception: StackExchange.Redis.RedisConnectionException: ProtocolFailure on GET --->` `StackExchange.Redis.RedisCommandException: Multiple databases are not supported on this server; cannot switch to database: 6`

    如需詳細資訊，請參閱 [Redis 叢集規格 - 實作的子集](http://redis.io/topics/cluster-spec#implemented-subset)。

-	如果您使用 [StackExchange.Redis](https://www.nuget.org/packages/StackExchange.Redis/)，則必須使用 1.0.481 或更新版本。您可以使用與連接未啟用叢集的快取時所用的相同[端點、連接埠和金鑰](cache-configure.md#properties)來連接快取。唯一的差別在於必須在資料庫 0 上完成所有的讀取和寫入。
	-	其他用戶端可能有不同的需求。請參閱[所有 Redis 用戶端都支援叢集嗎？](#do-all-redis-clients-support-clustering)
-	如果您的應用程式使用分成單一命令的多個索引鍵作業，則所有索引鍵都必須位於相同的分區。若要完成此動作，請參閱[如何在叢集中散發金鑰？](#how-are-keys-distributed-in-a-cluster)
-	如果您使用 Redis ASP.NET 工作階段狀態提供者，則必須使用 2.0.1 或更高版本。請參閱[我可以將叢集使用於 Redis ASP.NET 工作階段狀態和輸出快取提供者嗎？](#can-i-use-clustering-with-the-redis-aspnet-session-state-and-output-caching-providers)

### 如何在叢集中散發索引鍵？

依據 Redis [金鑰散發模型](http://redis.io/topics/cluster-spec#keys-distribution-model)文件︰金鑰空間會分割成 16384 個位置。每個索引鍵都會雜湊並指派給上述的其中一個位置，而這些位置散發於叢集的各個節點。您可以設定哪個部分的索引鍵會雜湊，以確保多個索引鍵位於使用雜湊標記的相同分區中。

-	具有雜湊標記的金鑰 - 如果金鑰的任何部分被括在 `{` 和 `}` 中，則只有該部分的金鑰會為了判斷金鑰的雜湊位置而進行雜湊。例如，下列 3 個金鑰會位於相同的分區︰`{key}1`、`{key}2` 和 `{key}3`，因為只會雜湊名稱的 `key` 部分。如需索引鍵雜湊標記規格的完整清單，請參閱[索引鍵雜湊標記](http://redis.io/topics/cluster-spec#keys-hash-tags)。
-	沒有雜湊標記的索引鍵 - 整個索引鍵名稱都用於雜湊。這會導致以統計方式平均散發於快取的各個分區。

如需最佳的效能和輸送量，我們建議平均散發索引鍵。如果您使用具有雜湊標記的索引鍵，則應用程式必須負責確保平均散發索引鍵。

如需詳細資訊，請參閱[金鑰散發模型](http://redis.io/topics/cluster-spec#keys-distribution-model)、[Redis 叢集資料分區化](http://redis.io/topics/cluster-tutorial#redis-cluster-data-sharding)和[金鑰雜湊標記](http://redis.io/topics/cluster-spec#keys-hash-tags)。

如需搭配 StackExchange.Redis 用戶端使用叢集，並尋找相同分區中之金鑰的範例程式碼，請參閱 [Hello World](https://github.com/rustd/RedisSamples/tree/master/HelloWorld) 範例的 [clustering.cs](https://github.com/rustd/RedisSamples/blob/master/HelloWorld/Clustering.cs) 部分。

### 我可以建立的最大快取大小為何？

最大的進階快取大小為 53 GB。您最多可以建立 10 個分區，等於最大大小為 530 GB。如果您需要較大的大小，可以[要求更多](mailto:wapteams@microsoft.com?subject=Redis%20Cache%20quota%20increase)。如需詳細資訊，請參閱 [Azure Redis 快取價格](https://azure.microsoft.com/pricing/details/cache/)。

### 所有 Redis 用戶端都支援叢集嗎？

現階段，並非所有用戶端都支援 Redis 叢集。StackExchange.Redis 就是不支援的其中一例。如需其他用戶端的詳細資訊，請參閱 [Redis 叢集教學課程](http://redis.io/topics/cluster-tutorial)的[試用叢集](http://redis.io/topics/cluster-tutorial#playing-with-the-cluster)一節。

>[AZURE.NOTE] 如果您使用 StackExchange.Redis 做為您的用戶端，請確定您使用的是最新版的 [StackExchange.Redis](https://www.nuget.org/packages/StackExchange.Redis/) (1.0.481) 或更新版本，叢集才能正常運作。如果您有任何關於 Move 例外狀況的問題，請參閱 [Move 例外狀況](#move-exceptions)，以取得詳細資訊。

### 啟用叢集後，要如何連接到我的快取？

您可以使用與連接未啟用叢集的快取時所用的相同[端點、連接埠和金鑰](cache-configure.md#properties)來連接快取。Redis 會管理後端上的叢集，因此您不需從用戶端進行管理。

### 我可以直接連接到我的快取的個別分區嗎？

目前官方尚未提供支援。如前所述，每個分區都包含一個主要/複本快取組，統稱為快取執行個體。您可以使用 GitHub 中 Redis 存放庫[不穩定](http://redis.io/download)分支內的 redis-cli 公用程式，連接到這些快取執行個體。使用 `-c` 參數啟用這個版本時，會實作基本支援。如需詳細資訊，請參閱 [Redis 叢集教學課程](http://redis.io/topics/cluster-tutorial)中 [http://redis.io](http://redis.io) 上的[試用叢集](http://redis.io/topics/cluster-tutorial#playing-with-the-cluster)。

如為非 SSL，請使用下列命令。

	Redis-cli.exe –h <<cachename>> -p 13000 (to connect to instance 0)
	Redis-cli.exe –h <<cachename>> -p 13001 (to connect to instance 1)
	Redis-cli.exe –h <<cachename>> -p 13002 (to connect to instance 2)
	...
	Redis-cli.exe –h <<cachename>> -p 1300N (to connect to instance N)

如為 SSL，將 `1300N` 取代為 `1500N`。

### 我可以為先前建立的快取設定叢集嗎？

目前您只能在建立快取時啟用叢集。您可以在建立快取之後變更叢集大小，但無法在建立快取之後將叢集新增至進階快取，或從進階快取中移除叢集。已啟用叢集且只有一個分區的進階快取，與相同大小且沒有叢集的進階快取不同。

### 我可以設定基本或標準快取的叢集嗎？

叢集僅適用於進階快取。

### 我可以將叢集使用於 Redis ASP.NET 工作階段狀態和輸出快取提供者嗎？

-	**Redis 輸出快取提供者** - 不需要變更。
-	**Redis 工作階段狀態提供者** - 若要使用叢集，您必須使用 [RedisSessionStateProvider](https://www.nuget.org/packages/Microsoft.Web.RedisSessionStateProvider) 2.0.1 或更高版本，否則會擲回例外狀況。這是一項重大變更。如需詳細資訊，請參閱 [v2.0.0 重大變更詳細資料](https://github.com/Azure/aspnet-redis-providers/wiki/v2.0.0-Breaking-Change-Details)。

<a name="move-exceptions"></a>
### 當我使用 StackExchange.Redis 和叢集時收到 MOVE 例外狀況，該怎麼辦？

如果您正在使用 StackExchange.Redis，並且在使用叢集時收到 `MOVE` 例外狀況，請確定您使用的是 [StackExchange.Redis 1.1.603](https://www.nuget.org/packages/StackExchange.Redis/) 或更新版本。如需設定 .NET 應用程式以使用 StackExchange.Redis 的指示，請參閱[設定快取用戶端](cache-dotnet-how-to-use-azure-redis-cache.md#configure-the-cache-clients)。

## 後續步驟
了解如何使用更多進階快取功能。

-	[Azure Redis Cache 高階層簡介](cache-premium-tier-intro.md)
  
<!-- IMAGES -->

[redis-cache-clustering]: ./media/cache-how-to-premium-clustering/redis-cache-clustering.png

[redis-cache-clustering-selected]: ./media/cache-how-to-premium-clustering/redis-cache-clustering-selected.png

[redis-cache-redis-cluster-size]: ./media/cache-how-to-premium-clustering/redis-cache-redis-cluster-size.png

<!---HONumber=AcomDC_0921_2016-->