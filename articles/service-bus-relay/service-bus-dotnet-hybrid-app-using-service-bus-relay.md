<properties
	pageTitle="混合式內部部署/雲端應用程式 (.NET) | Microsoft Azure"
	description="了解如何使用 Azure 服務匯流排轉送建立 .NET 內部部署/雲端混合式應用程式。"
	services="service-bus-relay"
	documentationCenter=".net"
	authors="sethmanheim"
	manager="timlt"
	editor=""/>

<tags
	ms.service="service-bus-relay"
	ms.workload="tbd"
	ms.tgt_pltfrm="na"
	ms.devlang="dotnet"
	ms.topic="hero-article"
	ms.date="09/16/2016"
	ms.author="sethm"/>

# 使用 Azure 服務匯流排轉送的 .NET 內部部署/雲端混合式應用程式

## 簡介

本文說明如何使用 Microsoft Azure 和 Visual Studio 建置混合式雲端應用程式。本教學課程假設您先前沒有使用 Azure 的經驗。不用 30 分鐘，您就會獲得一個使用多個 Azure 資源，於雲端上啟動並執行的應用程式。

您將了解：

-   如何建立 Web 服務或調整現有的 Web 服務，以供 Web 方案使用。
-   如何使用 Azure 服務匯流排轉送服務，在 Azure 應用程式與其他位置託管的 Web 服務之間共用資料。

[AZURE.INCLUDE [create-account-note](../../includes/create-account-note.md)]

## 服務匯流排轉送如何協助混合式方案

商業方案通常由下列項目組成：為了應付全新的獨特商業需求而撰寫的自訂程式碼，以及既有方案和系統提供的現有功能。

眾多方案架構爭相開始使用雲端，以期能夠更輕鬆地處理擴充需求並降低操作成本。在這麼做之後，它們發現想要運用作為其方案建置組塊的現有服務資產是在公司防火牆內，無法供雲端方案輕易存取。許多內部服務並不是以可輕易在公司網路邊緣公開的方式建置或主控。

「服務匯流排轉送」是針對採取現有 Windows Communication Foundation (WCF) Web 服務，並使那些服務可供公司週邊外之解決方案安全存取的使用案例而設計的，不需要進行會干擾公司網路基礎結構的變更。這類服務匯流排轉送服務仍然裝載在其現有的環境，但它們會委派接聽連入工作階段和雲端裝載的服務匯流排的要求。服務匯流排也可使用[共用存取簽章](../service-bus/service-bus-sas-overview.md) (SAS) 驗證，保護這些服務免受未經授權的存取。

## 解決方案案例

在本教學課程中，您將建立 ASP.NET 網站，讓您可在產品庫存頁面上看到產品清單。

![][0]

本教學課程假設您有現有內部部署系統中的產品資訊，並使用服務匯流排轉送來連接該系統。這項模擬是由簡單主控台應用程式中執行的 Web 服務來進行，並由記憶體內的產品集支援。您將能夠在自己的電腦上執行這個主控台應用程式，並將 Web 角色部署至 Azure。在這麼做之後，您將看到在 Azure 資料中心執行的 Web 角色將確實呼叫至電腦，即使電腦幾乎確定會位於至少一個防火牆和網路位址轉譯 (NAT) 層後面也一樣。

下列是已完成之 Web 應用程式的開始頁面螢幕擷取畫面。

![][1]

## 設定開發環境

在開始開發 Azure 應用程式之前，請取得工具，並設定開發環境：

1.  從[取得工具和 SDK][] 頁面安裝 Azure SDK for .NET。

2. 	按一下您所使用 Visual Studio 版本的 [安裝 SDK]。本教學課程中的步驟使用 Visual Studio 2015。

4.  當系統提示您執行或儲存安裝程式時，按一下 [執行]。

5.  在 **Web Platform Installer** 中，按一下 [安裝] 並繼續進行安裝。

6.  完成安裝後，您將具有開始進行開發所需的一切。SDK 包含可讓您在 Visual Studio 輕易開發 Azure 應用程式的工具。如果您未安裝 Visual Studio，則 SDK 也會安裝免費的 Visual Studio Express。

## 建立命名空間

若要開始在 Azure 中使用服務匯流排功能，首先必須建立服務命名空間。命名空間提供範圍容器，可在應用程式內定址服務匯流排資源。

[AZURE.INCLUDE [service-bus-create-namespace-portal](../../includes/service-bus-create-namespace-portal.md)]

## 建立內部部署伺服器

首先，您將建置 (模擬) 內部部署產品目錄系統。此作業相當簡單；您可將它看成是呈現實際內部部署產品目錄系統，此系統具有我們正在嘗試整合的完整服務面。

此專案是 Visual Studio 主控台應用程式，會使用 [Azure 服務匯流排 NuGet 封裝](https://www.nuget.org/packages/WindowsAzure.ServiceBus/)來納入服務匯流排程式庫和組態設定。

### 建立專案

1.  使用系統管理員權限，啟動 Microsoft Visual Studio。若要以系統管理員權限啟動 Visual Studio，請在 **Visual Studio** 程式圖示上按一下滑鼠右鍵，然後按一下 [以系統管理員身分執行]。

2.  在 Visual Studio 的 [檔案] 功能表，按一下 [新增]，然後按一下 [專案]。

3.  從 [已安裝的範本] 的 [Visual C#] 下，按一下 [主控台應用程式]。在 [名稱] 方塊中，鍵入名稱 **ProductsServer**：

    ![][11]

4.  按一下 [確定] 以建立 **ProductsServer** 專案。

7.  如果已安裝 Visual Studio 的 NuGet 封裝管理員，請跳至下一個步驟。否則，請造訪 [NuGet][] \(英文)，然後按一下 [[安裝 NuGet]](http://visualstudiogallery.msdn.microsoft.com/27077b70-9dad-4c64-adcf-c7cf6bc9970c) (英文)。按照提示安裝 NuGet 封裝管理員，然後重新啟動 Visual Studio。

7.  在 [方案總管] 中，以滑鼠右鍵按一下 **ProductsServer** 專案，然後按一下 [管理 NuGet 封裝]。

8.  按一下 [瀏覽] 索引標籤，然後搜尋 `Microsoft Azure Service Bus`。按一下 [安裝] 並接受使用條款。

    ![][13]

    請注意，現在會參考必要的用戶端組件。

9.  新增產品連絡人的新類別。在 [方案總管] 的 **ProductsServer** 專案上按一下滑鼠右鍵、按一下 [加入]，然後按一下 [類別]。

10. 在 [名稱] 方塊中，鍵入名稱 **ProductsContract.cs**。然後按一下 [新增]。

11. 在 **ProductsContract.cs** 中，將命名空間定義取代為下列程式碼，以定義服務的連絡人。

	```
	namespace ProductsServer
	{
	    using System.Collections.Generic;
	    using System.Runtime.Serialization;
	    using System.ServiceModel;
	
	    // Define the data contract for the service
	    [DataContract]
	    // Declare the serializable properties.
	    public class ProductData
	    {
	        [DataMember]
	        public string Id { get; set; }
	        [DataMember]
	        public string Name { get; set; }
	        [DataMember]
	        public string Quantity { get; set; }
	    }
	
	    // Define the service contract.
	    [ServiceContract]
	    interface IProducts
	    {
	        [OperationContract]
	        IList<ProductData> GetProducts();
	
	    }
	
	    interface IProductsChannel : IProducts, IClientChannel
	    {
	    }
	}
	```

12. 在 Program.cs 中，將命名空間定義取代為下列程式碼，為它新增設定檔服務和主機。

	```
	namespace ProductsServer
	{
	    using System;
	    using System.Linq;
	    using System.Collections.Generic;
	    using System.ServiceModel;
	
	    // Implement the IProducts interface.
	    class ProductsService : IProducts
	    {
	
	        // Populate array of products for display on website
	        ProductData[] products =
	            new []
	                {
	                    new ProductData{ Id = "1", Name = "Rock",
	                                     Quantity = "1"},
	                    new ProductData{ Id = "2", Name = "Paper",
	                                     Quantity = "3"},
	                    new ProductData{ Id = "3", Name = "Scissors",
	                                     Quantity = "5"},
	                    new ProductData{ Id = "4", Name = "Well",
	                                     Quantity = "2500"},
	                };
	
	        // Display a message in the service console application
	        // when the list of products is retrieved.
	        public IList<ProductData> GetProducts()
	        {
	            Console.WriteLine("GetProducts called.");
	            return products;
	        }
	
	    }
	
	    class Program
	    {
	        // Define the Main() function in the service application.
	        static void Main(string[] args)
	        {
	            var sh = new ServiceHost(typeof(ProductsService));
	            sh.Open();
	
	            Console.WriteLine("Press ENTER to close");
	            Console.ReadLine();
	
	            sh.Close();
	        }
	    }
	}
	```

13. 在 [方案總管] 中，按兩下 **App.config** 檔案，以在 Visual Studio 編輯器中開啟它。在 **&lt;system.ServiceModel&gt;** 元素的底部 (但仍在 &lt;system.ServiceModel&gt; 內)，加入下列 XML 程式碼。請務必以您的命名空間名稱取代 yourServiceNamespace，並以您先前從入口網站擷取到的 SAS 金鑰取代 yourKey：

    ```
    <system.serviceModel>
	...
      <services>
         <service name="ProductsServer.ProductsService">
           <endpoint address="sb://yourServiceNamespace.servicebus.windows.net/products" binding="netTcpRelayBinding" contract="ProductsServer.IProducts" behaviorConfiguration="products"/>
         </service>
      </services>
      <behaviors>
         <endpointBehaviors>
           <behavior name="products">
             <transportClientEndpointBehavior>
                <tokenProvider>
                   <sharedAccessSignature keyName="RootManageSharedAccessKey" key="yourKey" />
                </tokenProvider>
             </transportClientEndpointBehavior>
           </behavior>
         </endpointBehaviors>
      </behaviors>
    </system.serviceModel>
    ```
14. 仍是在 App.config 中，請以您先前從入口網站取得的連接字串取代 **&lt;appSettings&gt;** 元素中的連接字串值。

	```
	<appSettings>
   	<!-- Service Bus specific app settings for messaging connections -->
   	<add key="Microsoft.ServiceBus.ConnectionString"
	       value="Endpoint=sb://yourNamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=yourKey"/>
	</appSettings>
	```

14. 按 **Ctrl+Shift+B**，或從 [建置] 功能表中按一下 [建置方案] 來建置應用程式並驗證您的工作到目前為止是否正確無誤。

## 建立 ASP.NET 應用程式

在本節中，您將建置簡單的 ASP.NET 應用程式，來顯示從產品服務擷取的資料。

### 建立專案

1.  確定 Visual Studio 是以系統管理員權限來執行。

2.  在 Visual Studio 的 [檔案] 功能表，按一下 [新增]，然後按一下 [專案]。

3.  從 [已安裝的範本] 的 [Visual C#] 下，按一下 [ASP.NET Web 應用程式]。將專案命名為 **ProductsPortal**。然後按一下 [確定]。

    ![][15]

4.  從 [選取範本] 清單，按一下 [MVC]。

6.  勾選 [雲端中的主機] 方塊。

    ![][16]

5. 按一下 [變更驗證] 按鈕。在 [變更驗證] 對話方塊中，按一下 [不需要驗證]，然後按一下 [確定]。在本教學課程中，您會部署不需要使用者登入的應用程式。

	![][18]

6. 	在 [新建 ASP.NET 專案] 核取方塊中的 [Microsoft Azure] 區段，確認已選取 [在雲端託管]，並在下拉式清單中選取 [App Service]。

	![][19]

7. 按一下 [確定]。

8. 您現在必須設定新 Web 應用程式的 Azure 資源。遵循[設定新 Web 應用程式的 Azure 資源](../app-service-web/web-sites-dotnet-get-started.md#configure-azure-resources-for-a-new-web-app)一節中的所有步驟。然後，回到本教學課程並繼續進行下一個步驟。

5.  在 [方案總管] 中，於 [模型] 上按一下滑鼠右鍵、按一下 [加入]，再按一下 [類別]。在 [名稱] 方塊中，鍵入名稱 **Product.cs**。然後按一下 [新增]。

    ![][17]

### 修改 Web 應用程式

1.  在 Visual Studio 的 Product.cs 檔案中，將現有的命名空間定義取代為下列程式碼。

	```
	// Declare properties for the products inventory.
 	namespace ProductsWeb.Models
	{
    	public class Product
    	{
    	    public string Id { get; set; }
    	    public string Name { get; set; }
    	    public string Quantity { get; set; }
    	}
	}
	```

2.  在 [方案總管] 中展開 **Controllers** 資料夾，然後按兩下 **HomeController.cs** 檔案以在 Visual Studio 中開啟。

3. 在 **HomeController.cs** 檔案中，以下列程式碼取代現有的命名空間定義。

	```
	namespace ProductsWeb.Controllers
	{
	    using System.Collections.Generic;
	    using System.Web.Mvc;
	    using Models;
	
	    public class HomeController : Controller
	    {
	        // Return a view of the products inventory.
	        public ActionResult Index(string Identifier, string ProductName)
	        {
	            var products = new List<Product>
	                {new Product {Id = Identifier, Name = ProductName}};
	            return View(products);
	        }
	     }
	}
	```

3.  在 [方案總管] 中，展開 Views\\Shared 資料夾，然後按兩下 **\_Layout.cshtml** 檔案以在 Visual Studio 編輯器中開啟。

5.  將所有出現的 **My ASP.NET Application** 變更為 **LITWARE's Products**。

6. 移除 [首頁]、[關於] 和 [連絡人] 連結。在下列範例中，刪除反白顯示的程式碼。

	![][41]

7.  在 [方案總管] 中，展開 Views\\Home 資料夾，然後按兩下 **Index.cshtml** 以在 Visual Studio 編輯器中開啟。將檔案的整個內容取代為下列程式碼。

	```
	@model IEnumerable<ProductsWeb.Models.Product>
	
	@{
	 		ViewBag.Title = "Index";
	}
	
	<h2>Prod Inventory</h2>
	
	<table>
	  		<tr>
	      		<th>
	          		@Html.DisplayNameFor(model => model.Name)
	      		</th>
	              <th></th>
	      		<th>
	          		@Html.DisplayNameFor(model => model.Quantity)
	      		</th>
	  		</tr>
	
	@foreach (var item in Model) {
	  		<tr>
	      		<td>
	          		@Html.DisplayFor(modelItem => item.Name)
	      		</td>
	      		<td>
	          		@Html.DisplayFor(modelItem => item.Quantity)
	      		</td>
	  		</tr>
	}
	
	</table>
	```

9.  若要驗證您的工作到目前為止是否正確無誤，您可以按 **Ctrl+Shift+B** 來建置專案。


### 在本機執行應用程式

執行應用程式以驗證它是否能正常運作。

1.  確定 **ProductsPortal** 為作用中專案。在 [方案總管] 的專案名稱上按一下滑鼠右鍵，然後選取 [設定為啟始專案]。
2.  在 Visual Studio 中按 F5。
3.  您的應用程式應該就會出現在瀏覽器中並正在執行。

    ![][21]

## 組合在一起

下一步是利用 ASP.NET 應用程式連接內部部署產品伺服器。

1.  請在 Visual Studio 重新開啟您在[建立 ASP.NET 應用程式](#create-an-aspnet-application)一節中建立的 **ProductsPortal** 專案 (如果尚未開啟)。

2.  類似於＜建立內部部署伺服器＞一節中的步驟，將 NuGet 封裝新增至專案參考。在 [方案總管] 中，以滑鼠右鍵按一下 **ProductsPortal** 專案，然後按一下 [管理 NuGet 封裝]。

3.  搜尋「服務匯流排」並選取 [Microsoft Azure 服務匯流排] 項目。然後完成安裝並關閉此對話方塊。

4.  在 [方案總管] 的 **ProductsPortal** 專案上按一下滑鼠右鍵，再按一下 [新增]，然後按一下 [現有項目]。

5.  從 **ProductsServer** 主控台專案導覽至 **ProductsContract.cs** 檔案。按一下以反白顯示 ProductsContract.cs。按一下 [加入] 旁邊的向下箭頭，然後按一下 [加入做為連結]。

	![][24]

6.  現在會在 Visual Studio 編輯器中開啟 **HomeController.cs** 檔案，並將命名空間定義取代為下列程式碼。請務必以服務命名空間名稱取代 *yourServiceNamespace*，並以 SAS 金鑰取代 *yourKey*。這可讓用戶端呼叫內部部署服務，並傳回呼叫結果。

	```
	namespace ProductsWeb.Controllers
	{
	    using System.Linq;
	    using System.ServiceModel;
	    using System.Web.Mvc;
	    using Microsoft.ServiceBus;
	    using Models;
	    using ProductsServer;
	
	    public class HomeController : Controller
	    {
	        // Declare the channel factory.
	        static ChannelFactory<IProductsChannel> channelFactory;
	
	        static HomeController()
	        {
	            // Create shared access signature token credentials for authentication.
	            channelFactory = new ChannelFactory<IProductsChannel>(new NetTcpRelayBinding(),
	                "sb://yourServiceNamespace.servicebus.windows.net/products");
	            channelFactory.Endpoint.Behaviors.Add(new TransportClientEndpointBehavior {
	                TokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider(
	                    "RootManageSharedAccessKey", "yourKey") });
	        }
	
	        public ActionResult Index()
	        {
	            using (IProductsChannel channel = channelFactory.CreateChannel())
	            {
	                // Return a view of the products inventory.
	                return this.View(from prod in channel.GetProducts()
	                                 select
	                                     new Product { Id = prod.Id, Name = prod.Name,
	                                         Quantity = prod.Quantity });
	            }
	        }
	    }
	}
	```

7.  在 [方案總管] 中，以滑鼠右鍵按一下 **ProductsPortal** 方案 (務必以滑鼠右鍵按一下方案，而非專案)。按一下 [新增]，然後按一下 [現有專案]。

8.  導覽至 **ProductsServer** 專案，然後按兩下 **ProductsServer.csproj** 方案檔來新增它。

9.  **ProductsServer** 必須在執行中，才能在 **ProductsPortal** 上顯示資料。在 [方案總管] 的 **ProductsPortal** 方案上按一下滑鼠右鍵，然後按一下 [屬性]。[屬性頁面] 對話方塊隨即出現。

10. 在左側，按一下 [啟始專案]。在右側，按一下 [多個啟始專案]。確定 **ProductsServer** 和 **ProductsPortal** 依序出現，且 [啟動] 設為兩者的動作。

      ![][25]

11. 仍是在 [屬性] 對話方塊中，按一下左側的 [專案相依性]。

12. 在 [專案] 清單中，按一下 **ProductsServer**。確定**未**選取 [ProductsPortal]。

14. 在 [專案] 清單中，按一下 **ProductsPortal**。確定已選取 [ProductsServer]。

    ![][26]

15. 按一下 [屬性頁面] 對話方塊中的 [確定]。

## 在本機執行專案

若要在本機測試應用程式，請在 Visual Studio 中按 **F5**。內部部署伺服器 (**ProductsServer**) 應該會先啟動，然後 **ProductsPortal** 應用程式應在瀏覽器視窗中啟動。此時，您將看到從產品服務內部部署系統擷取的產品庫存清單資料。

![][10]

按 **ProductsPortal** 頁面上的 [重新整理]。每次重新整理頁面時，您會看到伺服器應用程式在呼叫來自 **ProductsServer** 的 `GetProducts()` 時顯示一則訊息。

先關閉這兩個應用程式，才能繼續下一個步驟。

## 將 ProductsPortal 專案部署至 Azure Web 應用程式

下一個步驟是將 **ProductsPortal** 前端轉換為 Azure Web 應用程式。首先，部署 **ProductsPortal** 專案，遵循[將 Web 專案部署至 Azure Web 應用程式](../app-service-web/web-sites-dotnet-get-started.md#deploy-the-web-project-to-the-azure-web-app)一節中的所有步驟。部署完成後，回到本教學課程並繼續進行下一個步驟。

> [AZURE.NOTE] 在部署後自動啟動 **ProductsPortal** Web 專案時，您可在瀏覽器視窗中看到錯誤訊息。這是預期的，且因為 **ProductsServer** 應用程式尚未執行而出現。

請複製已部署 Web 應用程式的 URL，印為您在下一個步驟中需要用到 URL。您也可以從 Visual Studio 中的 [Azure App Service 活動] 視窗中取得此 URL：

![][9]

### 將 ProductsPortal 設定為 Web 應用程式

在雲端執行此應用程式之前，您必須確定 **ProductsPortal** 是從 Visual Studio 內以 Web 應用程式的形式啟動。

1. 在 Visual Studio 中，以滑鼠右鍵按一下 **ProjectsPortal** 專案，然後按一下 [屬性]。

3. 在左側欄中，按一下 [Web]。

5. 在 [起始動作] 區段中，按一下 [起始 URL] 按鈕，然後在文字方塊中輸入您先前部署的 Web 應用程式的 URL，例如 `http://productsportal1234567890.azurewebsites.net/`。

	![][27]

6. 從 Visual Studio 的 [檔案] 功能表中，按一下 [全部儲存]。

7. 在 Visual Studio 的 [建置] 功能表中，按一下 [重建方案]。

## 執行應用程式

2.  按 F5 以建置並執行應用程式。內部部署伺服器 (**ProductsServer** 主控台應用程式) 應該會第一個啟動，然後 **ProductsPortal** 應用程式應該會在瀏覽器視窗中啟動，如下面的螢幕擷取畫面所示。再次注意，產品庫存清單會列出從產品服務內部部署系統擷取的資料，並在 Web 應用程式中顯示該資訊。檢查 URL，確定 **ProductsPortal** 正在雲端中以 Azure Web 應用程式的形式執行。

    ![][1]

	> [AZURE.IMPORTANT] **ProductsServer** 主控台應用程式必須正在執行中，而且能夠提供資料給 **ProductsPortal** 應用程式。如果瀏覽器顯示錯誤，請等候幾秒，讓 **ProductsServer** 載入並顯示下列訊息。然後按瀏覽器中的 [重新整理]。

	![][37]

3. 回到瀏覽器中，按 **ProductsPortal** 頁面上的 [重新整理]。每次重新整理頁面時，您會看到伺服器應用程式在呼叫來自 **ProductsServer** 的 `GetProducts()` 時顯示一則訊息。

	![][38]

## 後續步驟  

若要深入了解服務匯流排，請參閱下列資源：

* [Azure 服務匯流排][sbwacom]
* [如何使用服務匯流排佇列][sbwacomqhowto]


  [0]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hybrid.png
  [1]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/App2.png
  [取得工具和 SDK]: http://go.microsoft.com/fwlink/?LinkId=271920
  [NuGet]: http://nuget.org
  
  [11]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-con-1.png
  [13]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-multi-tier-13.png
  [15]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-2.png
  [16]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-4.png
  [17]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-7.png
  [18]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-5.png
  [19]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-6.png
  [9]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-9.png
  [10]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/App3.png

  [21]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/App1.png
  [24]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-12.png
  [25]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-13.png
  [26]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-14.png
  [27]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-8.png
  
  [36]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/App2.png
  [37]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-service1.png
  [38]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-service2.png
  [41]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-multi-tier-40.png
  [43]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-hybrid-43.png


  [sbwacom]: /documentation/services/service-bus/
  [sbwacomqhowto]: ../service-bus-messaging/service-bus-dotnet-get-started-with-queues.md

<!---HONumber=AcomDC_0928_2016-->