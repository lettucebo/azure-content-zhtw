<properties 
	pageTitle="DocumentDB 效能秘訣 | Microsoft Azure" 
	description="了解用以改善 Azure DocumentDB 資料庫效能的用戶端設定選項"
	keywords="如何改善資料庫效能"
	services="documentdb" 
	authors="mimig1" 
	manager="jhubbard" 
	editor="" 
	documentationCenter=""/>

<tags 
	ms.service="documentdb" 
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="09/19/2016" 
	ms.author="mimig"/>

# DocumentDB 的效能秘訣

Azure DocumentDB 是一個既快速又彈性的分散式資料庫，可在獲得延遲與輸送量保證的情況下順暢地調整。使用 DocumentDB 時，您不須進行主要的架構變更，或是撰寫複雜的程式碼來調整您的資料庫。相應增加和減少就像進行單一 API 呼叫或 [SDK 方法呼叫](documentdb-performance-levels.md#changing-performance-levels-using-the-net-sdk)一樣簡單。不過，由於 DocumentDB 是透過網路呼叫存取，所以您可以進行用戶端最佳化以達到最高效能。

如果您詢問「如何改善我的資料庫效能？」，請考慮下列選項：

## 網路
<a id="direct-connection"></a>

1. **原則︰使用直接連接模式**
    
    用戶端連接到 Azure DocumentDB 的方式，對於效能有重大影響 (尤其對觀察到的用戶端延遲而言)。有兩個重要組態設定可用來設定用戶端連接原則 - 連接模式和[連接通訊協定](#connection-protocol)。兩個可用的模式︰

    1. 閘道模式 (預設值)
    2. 直接模式

    由於 DocumentDB 是分散式儲存體系統，DocumentDB 資源 (如集合) 會分割於多部電腦，並且會複寫每個資料分割以提供高可用性。同時以資源形式在內部提供使用的路由表，會保存將邏輯位址轉譯為實體位址的方法。

    在閘道模式中，DocumentDB 閘道機器會執行此路由，因此可讓用戶端程式碼變得簡單且精簡。用戶端應用程式會對 DocumentDB 閘道機器發出要求，這類機器可將要求中的邏輯 URI 轉譯成後端節點的實體位址，並適當地轉送要求。相反地，直接模式用戶端必須維護 (且定期重新整理) 這一份路由表，並直接連接到後端 DocumentDB 節點。

    所有 SDK 平台都支援閘道模式並設為預設值。如果您的應用程式在有嚴格防火牆限制的公司網路中執行，則閘道模式會是最佳的選擇，因為它會使用標準 HTTPS 連接埠與單一端點。不過，對於效能的影響是每次讀取或寫入 DocumentDB 資料時，閘道模式都會涉及額外的網路躍點。因此，直接模式因為網路躍點較少，所以可提供較佳的效能。

2. **連接原則︰使用 TCP 通訊協定**

    運用直接模式時，有兩個可用的通訊協定選項︰

    - TCP
    - HTTPS

    DocumentDB 提供透過 HTTPS 的簡單且開放 RESTful 程式設計模型。此外，它可提供有效率的 TCP 通訊協定，此 TCP 通訊協定在通訊模型中也符合 REST 限制，並且可以透過 .NET 用戶端 SDK 取得。為了達到最佳效能，儘可能使用 TCP 通訊協定。

    連接模式設定於使用 ConnectionPolicy 參數建構 DocumentClient 執行個體期間。如果使用直接模式，也可以在 ConnectionPolicy 參數內設定 Protocol。

        var serviceEndpoint = new Uri("https://contoso.documents.net");
        var authKey = new "your authKey from Azure Mngt Portal";
        DocumentClient client = new DocumentClient(serviceEndpoint, authKey, 
        new ConnectionPolicy
        {
            ConnectionMode = ConnectionMode.Direct,
            ConnectionProtocol = Protocol.Tcp
        });

    因為只有直接模式支援 TCP，所以如果使用閘道模式，則 HTTPS 通訊協定一律用來與閘道通訊，並忽略 ConnectionPolicy 中的 Protocol 值。

    ![DocumentDB 連接原則的圖例](./media/documentdb-performance-tips/azure-documentdb-connection-policy.png)

3. **呼叫 OpenAsync 以避免第一次要求的啟動延遲**

    根據預設，第一個要求會因為必須擷取位址路由表而有較高的延遲。若要避免此第一次要求的啟動延遲，您應該在初始化期間呼叫 OpenAsync() 一次，如下所示。

        await client.OpenAsync();
<a id="same-region"></a>
4. **為了效能在相同 Azure 區域中共置用戶端**

    可能的話，請將任何呼叫 DocumentDB 的應用程式放在與 DocumentDB 資料庫相同的區域中。以約略的比較來說，在相同區域內對 DocumentDB 進行的呼叫會在 1-2 毫秒內完成，但美國西岸和美國東岸之間的延遲則會大於 50 毫秒。視要求所採用的路由而定，各項要求從用戶端傳遞至 Azure 資料中心界限時的這類延遲可能有所不同。確保呼叫端應用程式位於與佈建的 DocumentDB 端點相同的 Azure 區域中，將可能達到最低的延遲。如需可用區域的清單，請參閱 [Azure 區域](https://azure.microsoft.com/regions/#services)。

    ![DocumentDB 連接原則的圖例](./media/documentdb-performance-tips/azure-documentdb-same-region.png) <a id="increase-threads"></a>
5. **增加執行緒/工作數目**

    由於對 DocumentDB 的呼叫是透過網路進行，因此您可能需要改變要求的平行處理原則程度，以便讓用戶端應用程式在不同要求之間只需等待很短的時間。例如，如果您使用 .NET 的[工作平行程式庫](https://msdn.microsoft.com//library/dd460717.aspx)，請建立大約數百個讀取或寫入 DocumentDB 的工作。

## SDK 的使用方式

1. **安裝最新的 SDK**

    DocumentDB SDK 會不斷改良，以提供最佳效能。請參閱 [DocumentDB SDK](documentdb-sdk-dotnet.md) 頁面來判斷最新的 SDK 並檢閱改進項目。

2. **在應用程式存留期內使用單一 DocumentDB 用戶端**
  
    請注意，每個 DocumentClient 執行個體都具備執行緒安全，並且在直接模式中運作時執行有效率的連接管理和位址快取。若要藉由 DocumentClient 獲得有效率的連接管理和更佳的效能，建議在應用程式存留期內對每個 AppDomain 使用單一 DocumentClient 執行個體。<a id="max-connection"></a>
3. **增加每部主機的 System.Net MaxConnections**

    DocumentDB 要求預設是透過 HTTPS/REST 發出，並受制於每個主機名稱或 IP 位址的預設連線限制。您可能必須將 MaxConnections 設定成較高的值 (100-1000)，以便讓用戶端程式庫能夠利用多個連到 DocumentDB 的同時連線。在 .NET SDK 1.8.0 和更新版本中，[ServicePointManager.DefaultConnectionLimit](https://msdn.microsoft.com/library/system.net.servicepointmanager.defaultconnectionlimit.aspx) 的預設值為 50，而若要變更此值，您可以將 [Documents.Client.ConnectionPolicy.MaxConnectionLimit](https://msdn.microsoft.com/zh-TW/library/azure/microsoft.azure.documents.client.connectionpolicy.maxconnectionlimit.aspx) 設定為更高的值。

4. **微調分割之集合的平行查詢**

    DocumentDB .NET SDK 1.9.0 版和更新版本支援平行查詢，可讓您平行查詢分割的集合。如需詳細資訊，請參閱＜使用 SDK＞和相關的程式碼範例。它們的設計目的是要改善查詢延遲和輸送量。平行查詢提供兩個可供使用者微調以符合其需求的參數：(a) MaxDegreeOfParallelism：用來控制可平行查詢的分割數目上限，以及 (b) MaxBufferedItemCount：用來控制預先擷取的結果數目。
    
    (a) **微調 MaxDegreeOfParallelism：**平行查詢的運作方式是以平行方式查詢多個分割。不過，對於查詢會以循序方式擷取來自個別分割集合的資料。因此，將 MaxDegreeOfParallelism 設定為分割數目會最有機會達到最高效能的查詢，但前提是其他所有系統條件皆維持不變。如果您不知道分割數目，您可以將 MaxDegreeOfParallelism 設定為較高的數字，然後系統會選擇最小值 (資料分割數目、使用者提供的輸入值) 做為 MaxDegreeOfParallelism。
    
    請務必注意，若對於查詢是以平均方式將資料分佈於所有分割，平行查詢便會產生最佳效益。如果分割之集合的分割方式是查詢所傳回的所有或大多數資料集中在少數幾個分割中 (最差的情況是集中在一個分割)，則這些分割會成為查詢效能的瓶頸。
    
    (b) **微調 MaxBufferedItemCount：**平行查詢的設計目的是要在用戶端處理目前的結果批次時預先擷取結果。預先擷取有助於改善查詢的整體延遲。MaxBufferedItemCount 是用來限制預先擷取之結果數量的參數。將 MaxBufferedItemCount 設定為預期傳回的結果數目 (或更高的數目)，可讓查詢透過預先擷取獲得最大效益。
    
    請注意，預先擷取會以相同方式運作，不受 MaxDegreeOfParallelism 的影響，而且來自所有分割的資料會有單一緩衝區。

5. **開啟伺服器端 GC**
    
    在某些情況下，降低廢棄項目收集頻率可能會有幫助。在 .NET 中，請將 [gcServer](https://msdn.microsoft.com/library/ms229357.aspx) 設定為 true。

6. **在 RetryAfter 間隔實作降速**
 
    在進行效能測試期間，您應該增加負載，直到系統對小部分要求進行節流處理為止。如果進行節流處理，用戶端應用程式應該在節流時降速，且持續時間達伺服器指定的重試間隔。採用降速可確保您在重試之間花費最少的等待時間。重試原則支援包含於 DocumentDB [.NET](documentdb-sdk-dotnet.md) 和 [Java](documentdb-sdk-java.md) 的 1.8.0 版和更新版本中，以及 [Node.js](documentdb-sdk-nodejs.md) 和 [Python](documentdb-sdk-python.md) 的 1.9.0 版或更新版本中。如需詳細資訊，請參閱[超過保留的輸送量限制](documentdb-request-units.md#exceeding-reserved-throughput-limits)和 [RetryAfter](https://msdn.microsoft.com/library/microsoft.azure.documents.documentclientexception.retryafter.aspx)。

7. **相應放大用戶端工作負載**

    如果您是以高輸送量層級 (> 50,000 RU/秒) 進行測試，用戶端應用程式可能會成為瓶頸，因為電腦對 CPU 或網路的使用率將達到上限。如果您達到這個點，您可以將用戶端應用程式向外延展至多部伺服器，以繼續將 DocumentDB 帳戶再往前推進一步。

8. **快取較低讀取延遲的文件 URI**

    盡可能快取文件 URI 以達到最佳讀取效能。<a id="tune-page-size"></a>
9. **調整查詢/讀取摘要的頁面大小以獲得更好的效能**

    使用讀取摘要功能 (亦即 ReadDocumentFeedAsync) 執行大量文件讀取時，或發出 DocumentDB SQL 查詢時，如果結果集太大，則會以分段方式傳回結果。根據預設，會以 100 個項目或 1 MB 的區塊傳回結果 (以先達到的限制為準)。

    若要減少擷取所有適用結果所需的網路來回行程次數，您可以使用 x-ms-max-item-count 要求標頭將頁面大小最多增加至 1000。在您只需要顯示幾個結果的情況下 (例如，您的使用者介面或應用程式 API 一次只傳回 10 筆結果)，您也可以將頁面大小縮小為 10，以降低讀取和查詢所耗用的輸送量。

    您也可以使用可用的 DocumentDB SDK 設定頁面大小。例如：
    
        IQueryable<dynamic> authorResults = client.CreateDocumentQuery(documentCollection.SelfLink, "SELECT p.Author FROM Pages p WHERE p.Title = 'About Seattle'", new FeedOptions { MaxItemCount = 1000 });

10. **增加執行緒/工作數目**

	請參閱＜網路＞一節中的[增加執行緒/工作數目](increase-threads.md)。

## 索引原則

1. **使用延遲索引加快尖峰時間擷取速率**

    DocumentDB 可讓您在集合層級指定索引編製原則，該原則可讓您選擇是否要對集合中的文件自動編製索引。此外，您也可以選擇同步 (一致) 和非同步 (延遲) 索引更新。每次在集合中插入、取代或刪除文件時，預設會同步更新索引。同步模式可讓查詢遵守與文件讀取操作相同的[一致性層級](documentdb-consistency-levels.md)，而不會有任何需要索引趕上的延遲。
    
    當發生資料寫入暴增，而您想要拉長時間來分攤編製內容索引所需的工作時，可以考慮延遲索引編製。延遲索引編製也可讓您有效地使用您佈建的輸送量，並在尖峰時間，以最少的延遲為寫入要求提供服務。不過，請特別注意，啟用延遲索引編製功能後，不論為 DocumentDB 帳戶設定的一致性層級為何，查詢結果最終仍是會保持一致。

    因此，一致索引編製模式 (IndexingPolicy.IndexingMode 設定為 [一致]) 會產生每次寫入的最高要求單位費用，而延遲索引編製模式 (IndexingPolicy.IndexingMode 設定為 [延遲]) 和不編製索引 (IndexingPolicy.Automatic 設定為 False) 在寫入時的編製索引成本為零。

2. **從索引編製中排除未使用的路徑以加快寫入速度**

    DocumentDB 的索引編製原則也可讓您利用檢索路徑 (IndexingPolicy.IncludedPaths 和 IndexingPolicy.ExcludedPaths)，指定要在索引編製中包含或排除的文件路徑。在事先知道查詢模式的案例中，使用檢索路徑可改善寫入效能並降低索引儲存空間，因為檢索成本與檢索的唯一路徑數目直接相互關聯。例如，下列程式碼示範如何使用 "*" 萬用字元，將文件 (也稱為樹狀子目錄) 的整個區段自索引編製作業中排除。

        var collection = new DocumentCollection { Id = "excludedPathCollection" };
        collection.IndexingPolicy.IncludedPaths.Add(new IncludedPath { Path = "/*" });
        collection.IndexingPolicy.ExcludedPaths.Add(new ExcludedPath { Path = "/nonIndexedContent/*");
        collection = await client.CreateDocumentCollectionAsync(UriFactory.CreateDatabaseUri("db"), excluded);

    如需詳細資訊，請參閱 [DocumentDB 索引編製原則](documentdb-indexing-policies.md)。

## 輸送量
<a id="measure-rus"></a>

1. **測量和調整較低的要求單位/秒使用量**

    DocumentDB 提供許多的資料庫作業，包括使用 UDF、預存程序和觸發程序進行關聯式和階層式查詢，而這些作業全都是對資料庫集合內的文件來進行。與上述各項作業相關聯的成本，會因為完成作業所需的 CPU、IO 和記憶體而不同。您不需要考慮和管理硬體資源，您可以將要求單位 (RU) 想成是執行各種資料庫作業以及服務應用程式要求時所需的資源數量。

    系統會依據您購買的容量單位數目，為每個資料庫帳戶佈建[要求單位](documentdb-request-units.md)。要求單位消耗量是以每秒的速率來計算。如果應用程式的速率超過為其帳戶佈建的要求單位速率，便會受到限制，直到該速率降到帳戶的預留層級以下。如果應用程式需要較高層級的輸送量，您可以另外購買容量單位。

    查詢的複雜性會影響針對作業所耗用的要求單位數量。述詞數目、述詞性質、UDF 數目，以及來源資料集的大小，全都會影響查詢作業的成本。

    若要測量任何作業 (建立、更新或刪除) 的負荷，請檢查 x-ms-request-charge 標頭 (或 .NET SDK 的 ResourceResponse<T> 或 FeedResponse<T> 中同等的 RequestCharge 屬性) 來測量這些作業所耗用的要求單位數量。

        // Measure the performance (request units) of writes
        ResourceResponse<Document> response = await client.CreateDocumentAsync(collectionSelfLink, myDocument);
        Console.WriteLine("Insert of document consumed {0} request units", response.RequestCharge);
        // Measure the performance (request units) of queries
        IDocumentQuery<dynamic> queryable = client.CreateDocumentQuery(collectionSelfLink, queryString).AsDocumentQuery();
        while (queryable.HasMoreResults)
             {
                  FeedResponse<dynamic> queryResponse = await queryable.ExecuteNextAsync<dynamic>();
                  Console.WriteLine("Query batch consumed {0} request units", queryResponse.RequestCharge);
             }
        
    在此標頭中傳回的要求費用是佈建輸送量的一小部分 (也就是 2000 RU / 秒)。例如，如果上述查詢傳回 1000 份 1KB 文件，則作業成本會是 1000。因此在一秒內，伺服器在節流後續要求前，只會接受兩個這類要求。如需詳細資訊，請參閱[要求單位](documentdb-request-units.md)和[要求單位計算機](https://www.documentdb.com/capacityplanner)。

2. **處理速率限制/要求速率太大**

    當用戶端嘗試超過帳戶保留的輸送量時，伺服器的效能不會降低，而且不會使用超過保留層級的輸送量容量。伺服器將預先使用 RequestRateTooLarge (HTTP 狀態碼 429) 來結束要求，並傳回 x-ms-retry-after-ms 標頭，以指出使用者重試要求之前必須等候的時間量 (毫秒)。
 
        HTTP Status 429,
        Status Line: RequestRateTooLarge
        x-ms-retry-after-ms :100

    SDK 全都隱含地攔截這個回應，採用伺服器指定的 retry-after 標頭，並重試此要求。除非有多個用戶端同時存取您的帳戶，否則下次重試將會成功。

    如果您有多個用戶端都以高於要求速率的方式累積運作，則用戶端在內部設定為 9 的預設重試計數可能會不敷使用；在此情況下，用戶端會擲回 DocumentClientException (狀態碼 429) 到應用程式。在 ConnectionPolicy 執行個體上設定 RetryOptions，即可變更預設重試次數。根據預設，如果要求繼續以高於要求速率的方式運作，則會在 30 秒的累計等候時間後傳回 DocumentClientException (狀態碼 429)。即使目前的重試計數小於最大重試計數 (預設值 9 或使用者定義的值)，也會發生這種情況。

    雖然自動重試行為有助於改善大部分應用程式的恢復功能和可用性，但是在進行效能基準測試時可能會有所歧異 (尤其是在測量延遲時)。如果實驗達到伺服器節流並導致用戶端 SDK 以無訊息模式重試，則用戶端觀察到的延遲將會突然增加。若要避免效能實驗期間的延遲尖峰，測量每個作業所傳回的費用，並確保要求是以低於保留要求速率的方式運作。如需詳細資訊，請參閱[要求單位](documentdb-request-units.md)。
   
3. **輸送量較高之少量文件的設計**

    指定之作業的要求費用 (也就是要求處理成本) 與文件大小直接相互關聯。大型文件的作業成本高於小型文件的作業成本。

## 一致性層級

1. **使用較差的一致性層級以取得更好的讀取延遲**

    在調整 DocumentDB 應用程式效能時，另一個需要考量的重要因素是一致性層級。一致性層級的選擇會同時影響讀取和寫入的效能。您可以設定資料庫帳戶的預設一致性層級，而所選的一致性層級會套用至 DocumentDB 帳戶中的所有集合 (橫跨所有資料庫)。就寫入作業而言，變更一致性層級的影響層面為要求延遲。使用更強的一致性層級時，會增加寫入延遲。相反地，一致性層級對於讀取作業的影響層面則為輸送量。較差的一致性層級可讓用戶端實現較高的讀取輸送量。

    所有對使用者定義的資源發出的讀取和查詢，預設都會使用資料庫帳戶上所指定的預設一致性層級。不過，您可以指定 x-ms-consistency-level 要求標頭，來降低特定讀取/查詢要求的一致性層級。如需詳細資訊，請參閱 [DocumentDB 的一致性層級](documentdb-consistency-levels.md)。

## 後續步驟

如需用來評估 DocumentDB 以利用少數用戶端電腦達到高效能的範例應用程式，請參閱 [Azure DocumentDB 的效能和規模測試](documentdb-performance-testing.md)。

此外，若要深入了解如何針對規模和高效能設計您的應用程式，請參閱 [Azure DocumentDB 的資料分割與調整規模](documentdb-partition-data.md)。

<!---HONumber=AcomDC_0921_2016-->