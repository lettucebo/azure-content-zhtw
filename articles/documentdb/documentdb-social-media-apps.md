<properties 
	pageTitle="DocumentDB 的設計模式：社交媒體應用程式 | Microsoft Azure" 
	description="了解具有 DocumentDB 與其他 Azure 服務之儲存體彈性的社交網路設計模式。" 
	keywords="社交媒體應用程式"
	services="documentdb" 
	authors="ealsur" 
	manager="" 
	editor="" 
	documentationCenter=""/>

<tags 
	ms.service="documentdb" 
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="06/29/2016" 
	ms.author="mimig"/>

# 使用 DocumentDB 進行社交活動

在當今大幅互連的社會當中，我們的生活或多或少都成為**社交網路**的一部分。我們會使用社交網路與朋友、同事、家人保持聯絡，有時候還可以跟擁有共同興趣的人交流這份愛好。

身為工程師或開發人員的我們，可能會好奇這些網路如何儲存我們的資料以及與其互連，或可能已針對本身的特定利基市場，建立或建構新的社交網路。這時候，最大的問題出現了︰該如何儲存這些所有資料？

好比說，我們要建立脫穎而出的全新社交網路，讓使用者可以張貼文章並內含相關媒體，例如圖片、影片，甚至音樂。使用者可以評論貼文，並給予評等的點數。他們可以看到貼文摘要，並可在主要的網站登陸頁面上進行互動。乍聽之下似乎不複雜，但為了保持簡潔，我們先把話題停留在此 (畢竟若要深入探討受關聯性影響的自訂使用者摘要，已超過本文的主題了)。

那麼，這些資料到底要如何儲存，以及儲存在哪呢？

許多人可能有使用 SQL 資料庫的經驗，或至少具有[關聯式資料模型化](https://en.wikipedia.org/wiki/Relational_model)的概念，因此可能會著手繪製類似如下的內容︰

![說明相對關聯式模型的圖表](./media/documentdb-social-media-apps/social-media-apps-sql.png)

但這麼正規且漂亮的資料結構，卻無法擴充。

別誤會，我已經使用 SQL 資料庫很久了，知道它的確很棒，但它也跟其他的模式、做法和軟體平台一樣，不可能是萬能的。

為什麼 SQL 不是此案例的最佳選擇？ 我們來看看一篇貼文的結構，假設我想要在網站或應用程式中顯示這篇貼文，就必須執行... 8 個資料表聯結的查詢 (您沒看錯！)，而這只為了顯示單一貼文！現在，試想一下，以動態方式載入並出現在螢幕上的貼文串流，您就懂我的意思了。

當然，我們還是可以使用一個龐大的 SQL 執行個體，來求解含有眾多聯結的數千個查詢並提供內容，但如果有一個比較簡單的解決方案，為什麼還要這麼大費周章？

## NoSQL 的方法

您可以[在 Azure 上執行](http://neo4j.com/developer/guide-cloud-deployment/#_windows_azure)一種特殊的圖表資料庫，但這種資料庫很昂貴，而且需要 IaaS 服務 (基礎結構即服務，主要是虛擬機器) 以及進行維護。因此，我會將這篇文章的重點放在適用於大多數案例且成本低廉，同時可在 Azure 的 NoSQL 資料庫 [DocumentDB](https://azure.microsoft.com/services/documentdb/) 上執行的解決方案。如果使用 [NoSQL](https://en.wikipedia.org/wiki/NoSQL) 方法，將資料以 JSON 格式儲存，並套用[反正規化](https://en.wikipedia.org/wiki/Denormalization)，即可將之前複雜的貼文轉換成單一[文件](https://en.wikipedia.org/wiki/Document-oriented_database)：

    {
        "id":"ew12-res2-234e-544f",
        "title":"post title",
        "date":"2016-01-01",
        "body":"this is an awesome post stored on NoSQL",
        "createdBy":User,
        "images":["http://myfirstimage.png","http://mysecondimage.png"],
        "videos":[
            {"url":"http://myfirstvideo.mp4", "title":"The first video"},
            {"url":"http://mysecondvideo.mp4", "title":"The second video"}
        ],
        "audios":[
            {"url":"http://myfirstaudio.mp3", "title":"The first audio"},
            {"url":"http://mysecondaudio.mp3", "title":"The second audio"}
        ]
    }

如此一來，即可利用單一查詢而不需要任何聯結，即可取得該文件。這麼做不只更簡單、直接也節省成本，而且只需較少的資源，即可達到更佳的結果。

Azure DocumentDB 可以其[自動索引編製作業](documentdb-indexing.md)來確保所有屬性都已編製成索引，甚至還可以[自訂](documentdb-indexing-policies.md)這項作業。無結構描述的方法可讓我們使用不同的動態結構來儲存文件，日後若我們希望貼文具有類別或已與其建立關聯的主題標籤清單時，不需進行任何額外的工作，即可由 DocumentDB 來處理新的文件與新增的屬性。

某篇文章的回應可視為含父屬性的其他貼文 (如此可簡化物件的對應)。

    {
        "id":"1234-asd3-54ts-199a",
        "title":"Awesome post!",
        "date":"2016-01-02",
        "createdBy":User2,
        "parent":"ew12-res2-234e-544f"
    }

    {
        "id":"asd2-fee4-23gc-jh67",
        "title":"Ditto!",
        "date":"2016-01-03",
        "createdBy":User3,
        "parent":"ew12-res2-234e-544f"
    }

接著，即可將所有社交互動儲存為個別物件上的計數器︰

    {
        "id":"dfe3-thf5-232s-dse4",
        "post":"ew12-res2-234e-544f",
        "comments":2,
        "likes":10,
        "points":200
    }

若要建立摘要，只需建立文件，包含具有特定相關性順序的貼文識別碼清單即可︰

    [
        {"relevance":9, "post":"ew12-res2-234e-544f"},
        {"relevance":8, "post":"fer7-mnb6-fgh9-2344"},
        {"relevance":7, "post":"w34r-qeg6-ref6-8565"}
    ]

例如，我們可以設定依建立日期的「最新」貼文串流、過去 24 小時內獲得較多讚的「最熱門」貼文串流，甚至可以依據邏輯 (例如關注者與興趣) 為每位使用者實作自訂串流，而這仍屬於文章清單。關鍵在於如何建立這些清單，而且讀取效能不會受到影響。在取得其中一份清單之後，我們可以使用 [IN 運算子](documentdb-sql-query.md#where-clause)向 DocumentDB 發出單一查詢，一次取得貼文的頁面。

您可以使用 [Azure App Service](https://azure.microsoft.com/services/app-service/) 的背景處理序 [Webjobs](../app-service-web/web-sites-create-web-jobs.md) 來建置摘要串流。建立貼文之後就會觸發背景處理，方法是使用 [Azure 儲存體](https://azure.microsoft.com/services/storage/)[佇列](../storage/storage-dotnet-how-to-use-queues.md)以及使用 [Azure Webjobs SDK](../app-service-web/websites-dotnet-webjobs-sdk.md) 觸發的 Webjobs，根據我們的自訂邏輯，在串流內進行貼文的傳播。

您也可以使用相同的技術，以延後方式來處理貼文的點數和按讚數，建立最終一致的環境。

至於粉絲，則需要有更多的技巧來處理。DocumentDB 的文件大小限制為 512 Kb，因此您可能會考慮使用下列結構，以文件形式儲存粉絲：

    {
    	"id":"234d-sd23-rrf2-552d",
    	"followersOf": "dse4-qwe2-ert4-aad2",
    	"followers":[
    		"ewr5-232d-tyrg-iuo2",
    		"qejh-2345-sdf1-ytg5",
    		//...
    		"uie0-4tyg-3456-rwjh"
    	]
    }

這可能適用於有數千位粉絲的使用者，但是，如果有一些名人加入我們的行列，這個處理方法最終將會觸達文件大小上限。

為了解決這個問題，我們可以使用混合的處理方法。我們可以在使用者統計資料文件中儲存粉絲人數：

    {
    	"id":"234d-sd23-rrf2-552d",
    	"user": "dse4-qwe2-ert4-aad2",
    	"followers":55230,
    	"totalPosts":452,
    	"totalPoints":11342
    }

接著使用[擴充功能](https://github.com/richorama/AzureStorageExtensions#azuregraphstore)，將實際的粉絲圖表儲存於 Azure 儲存體資料表，以允許進行簡單的「A-跟隨-B」儲存和擷取。如此一來，我們就可將確切粉絲清單的擷取程序 (當我們需要它時) 委派給 Azure 儲存體資料表，但對於快速號碼查閱，我們仍會繼續使用 DocumentDB。

## 「階梯」模式與資料複製

您可能已經注意到，參考貼文的 JSON 文件中，有一位使用者出現多次。您猜的沒錯，這意謂著在使用這種反正規化的情況下，代表使用者的資訊可能會出現在多個位置。

為了能更快速地查詢，所以產生了資料重複的情況。這個副作用產生的問題是，如果某些動作導致使用者的資料變更，我們就得尋找該使用者進行過的所有活動，並全數更新。聽起來不太實際，對吧？

圖表資料庫有另一套解決方式，而我們必須找出在應用程式中顯示之每個活動的使用者索引鍵屬性，來解決此問題。如果我們會在應用程式中看到某篇貼文，且只會顯示建立者的名稱與圖片，那為什麼要將所有的使用者資料都儲存在 “createdBy” 屬性中呢？ 如果我們只要針對每個回應顯示使用者的圖片，就不需要使用者資訊的其餘部分。這時就可以用到我所謂的「階梯模式」。

讓我們以使用者資訊當作範例︰

    {
        "id":"dse4-qwe2-ert4-aad2",
        "name":"John",
        "surname":"Doe",
        "address":"742 Evergreen Terrace",
        "birthday":"1983-05-07",
        "email":"john@doe.com",
        "twitterHandle":"@john",
        "username":"johndoe",
        "password":"some_encrypted_phrase",
        "totalPoints":100,
        "totalPosts":24
    }
    
查看此資訊可以快速偵測出哪些是重要資訊而哪些不是，以建立出一道「階梯」：

![階梯模式的圖表](./media/documentdb-social-media-apps/social-media-apps-ladder.png)

最小的一階稱為「使用者區塊」，其為可識別使用者並為資料重複使用的最基本部分。只要將重複資料的大小縮減為僅限我們「顯示」的資訊，即可降低大量更新的可能性。

中間的階梯稱為「使用者」，其為在 DocumentDB 中大多數與效能相依之查詢會用到的完整資料，也是最重要與最常存取的資料。它包含「使用者區塊」所代表的資訊。

最大的一階為「擴充使用者」。它包含所有重要的使用者資訊，加上其他並不需要快速讀取或最終使用情況的相關資料 (例如登入過程)。這些資料可以儲存在 DocumentDB 外部、Azure SQL Database 中或 Azure 儲存體資料表中。

為什麼我們要分割使用者，甚至在不同位置儲存這些資訊？ 因為 DocumentDB 的儲存空間[有限](documentdb-limits.md)，而且從效能方面來看，文件愈大，查詢也就愈昂貴。讓文件保持精簡，只包含適當資訊以針對社交網路進行所有與效能相依的查詢，並將用於最終情況 (例如完整的設定檔編輯、登入，甚至是針對資料採礦進行的使用情況分析和巨量資料計劃) 所需的其他額外資訊加以儲存。我們不用擔心針對資料採礦進行的資料收集作業是否比較慢，因為這項作業執行於 Azure SQL Database 上，但我們必須保障使用者擁有快速且精簡的體驗。在 DocumentDB 中儲存的「使用者」，應類似如下︰

    {
        "id":"dse4-qwe2-ert4-aad2",
        "name":"John",
        "surname":"Doe",
        "username":"johndoe"
        "email":"john@doe.com",
        "twitterHandle":"@john"
    }

而貼文應該看起來像這樣：

    {
        "id":"1234-asd3-54ts-199a",
        "title":"Awesome post!",
        "date":"2016-01-02",
        "createdBy":{
        	"id":"dse4-qwe2-ert4-aad2",
    		"username":"johndoe"
        }
    }

如果影響到其中一個區塊屬性之處發生編輯，您也可以使用指向索引屬性的查詢 (SELECT * FROM posts p WHERE p.createdBy.id == “edited\_user\_id”)，然後更新區塊，即可輕鬆找到受影響的文件。

## 搜尋方塊

一般來說，使用者都能很幸運地產生許多內容。即使內容不是直接在本身的內容串流中，我們也應該可以提供搜尋及尋找這些內容的功能，因為有時我們可能不想關注建立者，或只是想要尋找 6 個月前的舊貼文。

幸好，我們使用的是 Azure DocumentDB，因此可以透過 [Azure 搜尋服務](https://azure.microsoft.com/services/search/)，輕鬆地在幾分鐘的時間內實作搜尋引擎，而不需要輸入任何一行程式碼 (當然除了搜尋程序和 UI 以外)。

為什麼可以這麼輕鬆？

Azure 搜尋服務會實作[索引子](https://msdn.microsoft.com/library/azure/dn946891.aspx) (這是與您資料儲存機制連結的背景處理序)，並會自動新增、更新或移除您在索引中的物件。它們支援 [Azure SQL Database 索引子](https://blogs.msdn.microsoft.com/kaevans/2015/03/06/indexing-azure-sql-database-with-azure-search/)、[Azure Blob 索引子](../search/search-howto-indexing-azure-blob-storage.md)，很幸運地也支援 [Azure DocumentDB 索引子](../documentdb/documentdb-search-indexer.md)。從 DocumentDB 到 Azure 搜尋服務的資訊轉換相當簡單，因為兩者皆是以 JSON 格式儲存，我們只需要[建立索引](../search/search-create-index-portal.md)，並從我們要編製索引的文件來對應屬性即可。短短幾分鐘內 (取決於資料大小而定)，即可透過雲端基礎結構中的最佳搜尋即服務解決方案來搜尋所有內容。

如需 Azure 搜尋服務的詳細資訊，請瀏覽[搜尋漫遊指南](https://blogs.msdn.microsoft.com/mvpawardprogram/2016/02/02/a-hitchhikers-guide-to-search/)。

## 基礎知識

儲存這些所有內容之後，隨著內容每天不斷增長，我們可能會考慮到該怎麼運用這些使用者的資訊的串流？

答案很簡單：讓這些內容發揮效益，並從中學習。

但是，我們可以學習到什麼呢？ 其中幾個簡單的範例包括[情感分析](https://en.wikipedia.org/wiki/Sentiment_analysis)、根據使用者喜好設定的內容建議，甚至還包括自動化內容仲裁者，其可確保社交網路所發行的所有內容都適合闔家瀏覽。

現在，大家一定更感興趣了吧？您一定以為要有數學的博士學位，才能從簡單的資料庫和檔案中擷取這些模式和資訊，但您錯了。

[Azure Machine Learning](https://azure.microsoft.com/services/machine-learning/) 是一項完全受管理的雲端服務，其隨附於 [Cortana Intelligence Suite](https://www.microsoft.com/en/server-cloud/cortana-analytics-suite/overview.aspx)，可讓您透過簡單的拖放介面使用演算法來建立工作流程、以 [R](https://en.wikipedia.org/wiki/R_(programming_language)) 撰寫自己的演算法程式碼，或使用一些內建和現成的 API，例如︰[文字分析](https://gallery.cortanaanalytics.com/MachineLearningAPI/Text-Analytics-2)、[內容仲裁者](https://www.microsoft.com/moderator)或[建議](https://gallery.cortanaanalytics.com/MachineLearningAPI/Recommendations-2)。

為了達成上述任一個機器學習服務案例，我們可以使用 [Azure Data Lake](https://azure.microsoft.com/services/data-lake-store/) 內嵌不同來源的資訊，並使用 [U-SQL](https://azure.microsoft.com/documentation/videos/data-lake-u-sql-query-execution/) 來處理資訊，並產生可由 Azure Machine Learning 處理的輸出。

## 結論

本篇文章嘗試探討以低成本的服務在 Azure 上完整建立社交網路的替代方案，並鼓勵使用多層次儲存體解決方案和稱為「階梯」的資料分散方式，提供更好的結果。

![Azure 服務之間社交網路互動的圖表](./media/documentdb-social-media-apps/social-media-apps-azure-solution.png)

事實上，這類案例並沒有萬靈丹，而是靠下列強大服務組合所建立的綜效，它們可讓我們建立絕佳體驗：Azure DocumentDB 的高速和自由性，可提供強大的社交應用程式；Azure 搜尋服務這類頂級搜尋解決方案背後蘊藏的智慧；靈活的 Azure App Service，不只能主控不限語言的應用程式，還可主控功能強大的背景處理序；可儲存大量資料的可擴充 Azure 儲存體和 Azure SQL Database；具分析功能的 Azure Machine Learning，可建立知識與智慧，以提供處理序的意見反應並協助我們將正確的內容傳遞給適當的使用者。

## 後續步驟

請閱讀[在 DocumentDB 中模型化資料](documentdb-modeling-data.md)文章，以深入了解資料模型化。如果您想了解 DocumentDB 的其他使用案例，請參閱[常見的 DocumentDB 使用案例](documentdb-use-cases.md)。

或者，您可以遵循 [DocumentDB 的學習路徑](https://azure.microsoft.com/documentation/learning-paths/documentdb/)以深入了解 DocumentDB。

<!---HONumber=AcomDC_0706_2016-->