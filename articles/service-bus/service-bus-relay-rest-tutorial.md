<properties
    pageTitle="使用轉送訊息的服務匯流排 REST 教學課程 |Microsoft Azure"
    description="建置簡單的服務匯流排轉送主機應用程式來公開 REST 架構介面。"
    services="service-bus"
    documentationCenter="na"
    authors="sethmanheim"
    manager="timlt"
    editor="" />
<tags
    ms.service="service-bus"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="06/03/2016"
    ms.author="sethm" />

# 服務匯流排 REST 教學課程

本教學課程描述如何建置簡單的服務匯流排主機應用程式來公開 REST 架構介面。REST 可讓 Web 用戶端 (例如 Web 瀏覽器) 透過 HTTP 要求存取服務匯流排 API。

本教學課程會使用 Windows Communication Foundation (WCF) REST 程式設計模型，在服務匯流排上建構 REST 服務。如需詳細資訊，請參閱 WCF 文件中的 [WCF REST 程式設計模型](https://msdn.microsoft.com/library/bb412169.aspx)和[設計與實作服務](https://msdn.microsoft.com/library/ms729746.aspx)。

## 步驟 1︰建立服務命名空間

第一步是建立命名空間，並取得共用存取簽章 (SAS) 金鑰。命名空間會為每個透過服務匯流排公開的應用程式提供應用程式界限。建立服務命名空間時，系統會自動產生 SAS 金鑰。服務命名空間與 SAS 金鑰的結合提供一個認證，供服務匯流排驗證對應用程式的存取權。

1. 若要建立服務命名空間，請造訪 [Azure 傳統入口網站][]。按一下左側的 [服務匯流排]，再按一下 [建立]。輸入命名空間的名稱，再按一下核取記號。

2. 在 [Azure 傳統入口網站][]的主視窗中，按一下您在上一個步驟中建立的命名空間名稱。

3. 按一下 [設定] 索引標籤。

4. 在 [**共用存取金鑰產生器**] 區段中，記下與 **RootManageSharedAccessKey** 原則相關聯的 [**主要金鑰**]，或是將它複製到剪貼簿。您稍後會在本教學課程中使用此值。

## 步驟 2：定義 REST 架構 WCF 服務合約以使用服務匯流排

使用其他服務匯流排服務時，如果您建立 REST 樣式服務，您必須定義合約。合約會指定主機支援哪些作業。服務作業可視為 Web 服務方法。合約可以透過定義 C++、C# 或 Visual Basic 介面建立。介面中的每個方法會對應一個特定服務作業。[ServiceContractAttribute](https://msdn.microsoft.com/library/system.servicemodel.servicecontractattribute.aspx) 屬性必須套用至每個介面，且 [OperationContractAttribute](https://msdn.microsoft.com/library/system.servicemodel.operationcontractattribute.aspx) 屬性必須套用至每個作業。如果介面中的方法具備沒有 [OperationContractAttribute](https://msdn.microsoft.com/library/system.servicemodel.operationcontractattribute.aspx) 的 [ServiceContractAttribute](https://msdn.microsoft.com/library/system.servicemodel.servicecontractattribute.aspx)，則不會公開該方法。用來執行這些工作的程式碼顯示在程序後面的範例中。

基本的服務匯流排合約與 REST 樣式合約之間的主要差異在於屬性會新增至 [OperationContractAttribute](https://msdn.microsoft.com/library/system.servicemodel.operationcontractattribute.aspx)：[WebGetAttribute](https://msdn.microsoft.com/library/system.servicemodel.web.webgetattribute.aspx)。這個屬性可讓您將介面的方法對應至介面另一端的方法。在此情況下，我們會使用 [WebGetAttribute](https://msdn.microsoft.com/library/system.servicemodel.web.webgetattribute.aspx) 將方法連結至 HTTP GET。這可讓服務匯流排準確擷取和解譯傳送到介面的命令。

### 使用介面建立服務匯流排合約

1. 以系統管理員身分開啟 Visual Studio：以滑鼠右鍵按一下 [開始] 功能表中的程式，然後按一下 [以系統管理員身分執行]。

2. 這會建立新的主控台應用程式專案。按一下 [檔案] 功能表，再依序選取 [新增] 及 [專案]。在 [新增專案] 對話方塊中，按一下 [Visual C#]，選取 [主控台應用程式] 範本，並將它命名為 **ImageListener**。使用預設 [位置]。按一下 [確定] 以建立專案。

3. 若為 C# 專案，Visual Studio 會建立 `Program.cs` 檔案。這個類別包含空的 `Main()` 方法，即正確建置主控台應用程式專案所需的方法。

4. 安裝服務匯流排 NuGet 封裝，以新增服務匯流排和 **System.ServiceModel.dll** 的參考。此封裝會自動將加入服務匯流排程式庫的參考，以及 WCF **System.ServiceModel**。在 [方案總管] 中，以滑鼠右鍵按一下 **ImageListener** 專案，然後按一下 [管理 NuGet 封裝]。按一下 [瀏覽] 索引標籤，然後搜尋 `Microsoft Azure Service Bus`。按一下 [安裝] 並接受使用條款。

5. 您必須明確地將 **System.ServiceModel.dll** 的參考新增至專案：

	a.在 [方案總管] 中，以滑鼠右鍵按一下專案資料夾下的**參考**資料夾，然後按一下 [加入參考]。

	b.在 [新增參考] 對話方塊中，按一下左側的 [架構] 索引標籤，然後在 [搜尋] 方塊中輸入 **System.ServiceModel.Web**。選取 [System.ServiceModel.Web] 核取方塊，然後按一下 [確定]。

6. 在 Program.cs 檔案開頭處新增下列 `using` 陳述式。

	```
	using System.ServiceModel;
	using System.ServiceModel.Channels;
	using System.ServiceModel.Web;
	using System.IO;
	```

	[System.ServiceModel](https://msdn.microsoft.com/library/system.servicemodel.aspx) 是可讓您以程式設計方式存取 WCF 基本功能的命名空間。服務匯流排會使用 WCF 的許多物件和屬性來定義服務合約。您將在大部分服務匯流排轉送應用程式中使用此命名空間。同樣地，[System.ServiceModel.Channels](https://msdn.microsoft.com/library/system.servicemodel.channels.aspx) 可協助定義通道，您可透過此物件與服務匯流排和用戶端 Web 瀏覽器通訊。最後，[System.ServiceModel.Web](https://msdn.microsoft.com/library/system.servicemodel.web.aspx) 包含可讓您建立 Web 架構應用程式的類型。

7. 將 `ImageListener` 命名空間重新命名為 **Microsoft.ServiceBus.Samples**。

 	```
	namespace Microsoft.ServiceBus.Samples
	{
		...
	```

8. 直接在命名空間宣告的左大括號後面定義名為 **IImageContract** 的新介面，並將 **ServiceContractAttribute** 屬性套用至包含 `http://samples.microsoft.com/ServiceModel/Relay/` 值的介面。命名空間值與您的整個程式碼範圍中使用的命名空間不同。命名空間值作為此合約的唯一識別碼，且應該具有版本資訊。如需詳細資訊，請參閱[服務版本控制](http://go.microsoft.com/fwlink/?LinkID=180498)。明確指定命名空間可避免將預設命名空間值新增至合約名稱。

	```
	[ServiceContract(Name = "ImageContract", Namespace = "http://samples.microsoft.com/ServiceModel/Relay/RESTTutorial1")]
	public interface IImageContract
	{
	}
	```

9. 在 `IImageContract` 介面中，針對 `IImageContract` 合約在介面中公開的單一作業宣告方法，並將 `OperationContractAttribute` 屬性套用至您想要公開為公用服務匯流排合約一部分的方法。

	```
	public interface IImageContract
	{
		[OperationContract]
		Stream GetImage();
	}
	```

10. 在 **OperationContract** 屬性中，新增 **WebGet** 值。

	```
	public interface IImageContract
	{
		[OperationContract, WebGet]
		Stream GetImage();
	}
	```

	這樣可讓服務匯流排將 HTTP GET 要求傳送至 `GetImage`，以及將 `GetImage` 的傳回值轉譯成 HTTP GETRESPONSE 回覆。稍後在教學課程中，您將使用 Web 瀏覽器來存取此方法，並可在瀏覽器中顯示映像。

11. 緊接著 `IImageContract` 定義後面，宣告從 `IImageContract` 和 `IClientChannel` 介面繼承的通道。

	```
	public interface IImageChannel : IImageContract, IClientChannel { }
	```

	通道是 WCF 物件，服務和用戶端可透過它彼此傳遞資訊。您稍後將在主機應用程式中建立通道。服務匯流排接著使用此通道，將 HTTP GET 要求從瀏覽器傳遞至您的 **GetImage** 實作。服務匯流排也會使用通道，來取得 **GetImage** 傳回值，並將其轉譯成用戶端瀏覽器的 HTTP GETRESPONSE。

12. 從 [建置] 功能表中，按一下 [建置方案] 以確認您的工作到目前為止是否正確無誤。

### 範例

下列程式碼示範定義了服務匯流排合約的基本介面。

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.ServiceModel;
using System.ServiceModel.Channels;
using System.ServiceModel.Web;
using System.IO;

namespace Microsoft.ServiceBus.Samples
{

    [ServiceContract(Name = "IImageContract", Namespace = "http://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IImageContract
    {
        [OperationContract, WebGet]
        Stream GetImage();
    }

    public interface IImageChannel : IImageContract, IClientChannel { }

    class Program
    {
        static void Main(string[] args)
        {
        }
    }
}
```

## 步驟 3：實作 REST 架構 WCF 服務合約以使用服務匯流排

建立 REST 樣式服務匯流排服務需要先建立合約，您可使用介面定義該合約。下一步是實作介面。這牽涉到建立名為 **ImageService** 的類別，該類別實作使用者定義的 **IImageContract** 介面。實作合約後，接著可使用 App.config 檔案設定介面。組態檔包含應用程式的必要資訊，例如服務名稱、合約名稱，以及用來與服務匯流排通訊的通訊協定類型。程序後面的範例提供用來執行這些工作的程式碼。

如同先前的步驟，實作 REST 樣式合約與基本服務匯流排合約之間的差異很小。

### 實作 REST 樣式服務匯流排合約

1. 在緊接著 **IImageContract** 介面定義的正後方，建立名為 **ImageService** 的新類別。**ImageService** 類別會實作 **IImageContract** 介面。

	```
	class ImageService : IImageContract
	{
	}
	```
	類似於其他介面實作，您可以在不同的檔案中實作定義。不過，在此教學課程中，實作會出現在與介面定義和 `Main()` 方法相同的檔案中。

2. 將 [ServiceBehaviorAttribute](https://msdn.microsoft.com/library/system.servicemodel.servicebehaviorattribute.aspx) 屬性套用至 **IImageService** 類別，以表示此類別為 WCF 合約的實作。

	```
	[ServiceBehavior(Name = "ImageService", Namespace = "http://samples.microsoft.com/ServiceModel/Relay/")]
	class ImageService : IImageContract
	{
	}
	```

	如先前所述，此命名空間不是傳統的命名空間。相反地，它是識別合約的 WCF 架構的一部分。如需詳細資訊，請參閱 WCF 文件中的[資料合約名稱](https://msdn.microsoft.com/library/ms731045.aspx)主題。

3. 將 .jpg 映像加入至您的專案。

	這是此服務在接收瀏覽器中顯示的圖片。以滑鼠右鍵按一下您的專案，然後按一下 [加入]。然後按一下 [現有項目]。使用 [加入現有項目] 對話方塊瀏覽至適當的 .jpg，然後按一下 [加入]。

	加入檔案時，請確定選取 [檔案名稱:] 欄位旁下拉式清單中的 [所有檔案]。本教學課程的其餘部分假設映像的名稱為 "image.jpg"。如果您有不同的檔案，您必須重新命名映像，或變更您的程式碼來彌補。

4. 為確定執行中的服務能夠找到此影像檔，請在 [方案總管] 中以滑鼠右鍵按一下該影像檔，然後按一下 [屬性]。在 [**屬性**] 窗格中，將 [**複製到輸出目錄**] 設為 [**有更新時才複製**]。

5. 將 **System.Drawing.dll** 組件的參考新增至專案，並新增下列關聯的 `using` 陳述式。

	```
	using System.Drawing;
	using System.Drawing.Imaging;
	using Microsoft.ServiceBus;
	using Microsoft.ServiceBus.Web;
	```

6. 在 **ImageService** 類別中，加入下列建構函式，以便載入點陣圖並準備將它傳送給用戶端瀏覽器。

	```
	class ImageService : IImageContract
	{
		const string imageFileName = "image.jpg";

		Image bitmap;

		public ImageService()
		{
			this.bitmap = Image.FromFile(imageFileName);
		}
	}
	```

7. 緊接先前的程式碼後方，在 **ImageService** 類別中加入下列 **GetImage** 方法，以傳回包含該影像的 HTTP 訊息。

	```
	public Stream GetImage()
	{
		MemoryStream stream = new MemoryStream();
		this.bitmap.Save(stream, ImageFormat.Jpeg);

		stream.Position = 0;
		WebOperationContext.Current.OutgoingResponse.ContentType = "image/jpeg";

		return stream;
	}
	```

	這個實作會使用 **MemoryStream** 來擷取映像，並備妥它以串流至瀏覽器。它會從零起算串流位置，將串流內容宣告為 jpeg，並且串流資訊。

8. 從 [建置] 功能表中，按一下 [建置方案]。

### 為服務匯流排上執行的 Web 服務定義組態

1. 在 [方案總管] 中按兩下 **App.config**，以在 Visual Studio 編輯器中開啟它。

	**App.config** 檔類似於 WCF 組態檔，並包含服務名稱、端點 (也就是公開的位置服務匯流排，讓用戶端與主機彼此通訊) 和繫結 (用於通訊的通訊協定類型)。此處的主要差異是設定的服務端點會參考 [WebHttpRelayBinding](https://msdn.microsoft.com/library/microsoft.servicebus.webhttprelaybinding.aspx) 繫結，這不是 .NET Framework 的一部分。

2. `<system.serviceModel>` XML 元素是定義一或多個服務的 WCF 元素。在這裡，它用來定義服務名稱和端點。在 `<system.serviceModel>` 元素底部 (但仍在 `<system.serviceModel>` 內)，新增具有以下內容的 `<bindings>` 元素。這會定義應用程式中使用的繫結。您可以定義多個繫結，但在本教學課程中您只會定義一個。

	```
	<bindings>
		<!-- Application Binding -->
		<webHttpRelayBinding>
			<binding name="default">
				<security relayClientAuthenticationType="None" />
			</binding>
		</webHttpRelayBinding>
	</bindings>
	```

	此步驟會將 **relayClientAuthenticationType** 設為 **None** 來定義服務匯流排 [WebHttpRelayBinding](https://msdn.microsoft.com/library/microsoft.servicebus.webhttprelaybinding.aspx) 繫結。這個設定表示使用此繫結的端點不需要用戶端認證。

3. 在 `<bindings>` 元素後方加入 `<services>` 元素。類似於繫結，您可以在單一組態檔中定義多個服務。不過，您在本教學課程中只會定義一個。

	```
	<services>
		<!-- Application Service -->
		<service name="Microsoft.ServiceBus.Samples.ImageService"
             behaviorConfiguration="default">
			<endpoint name="RelayEndpoint"
					contract="Microsoft.ServiceBus.Samples.IImageContract"
					binding="webHttpRelayBinding"
					bindingConfiguration="default"
					behaviorConfiguration="sbTokenProvider"
					address="" />
		</service>
	</services>
	```

	這個步驟會設定服務，該服務會使用先前定義的預設 **webHttpRelayBinding**。它也會使用在下一個步驟中定義的預設 **sbTokenProvider**。

4. 在 `<services>` 元素後，建立具有下列內容的 `<behaviors>` 元素，以步驟 1 中的 [Azure 傳統入口網站][]所取得的*共用存取簽章* (SAS) 金鑰來取代 "SAS\_KEY"。

	```
	<behaviors>
		<endpointBehaviors>
			<behavior name="sbTokenProvider">
				<transportClientEndpointBehavior>
					<tokenProvider>
						<sharedAccessSignature keyName="RootManageSharedAccessKey" key="SAS_KEY" />
					</tokenProvider>
				</transportClientEndpointBehavior>
			</behavior>
			</endpointBehaviors>
			<serviceBehaviors>
				<behavior name="default">
					<serviceDebug httpHelpPageEnabled="false" httpsHelpPageEnabled="false" />
				</behavior>
			</serviceBehaviors>
	</behaviors>
	```

5. 仍是在 App.config 中，請以您先前從入口網站取得的連接字串，取代 `<appSettings>` 元素中的整個連接字串值。

	```
	<appSettings>
   	<!-- Service Bus specific app settings for messaging connections -->
   	<add key="Microsoft.ServiceBus.ConnectionString"
	       value="Endpoint=sb://yourNamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=yourKey"/>
	</appSettings>
	```

6. 從 [建置] 功能表中，按一下 [建置方案] 以建置整個方案。

### 範例

下列程式碼會顯示使用 **WebHttpRelayBinding** 繫結，正在服務匯流排上執行之 REST 架構服務的合約和服務實作。

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.ServiceModel;
using System.ServiceModel.Channels;
using System.ServiceModel.Web;
using System.IO;
using System.Drawing;
using System.Drawing.Imaging;
using Microsoft.ServiceBus;
using Microsoft.ServiceBus.Web;

namespace Microsoft.ServiceBus.Samples
{


    [ServiceContract(Name = "ImageContract", Namespace = "http://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IImageContract
    {
        [OperationContract, WebGet]
        Stream GetImage();
    }

    public interface IImageChannel : IImageContract, IClientChannel { }

    [ServiceBehavior(Name = "ImageService", Namespace = "http://samples.microsoft.com/ServiceModel/Relay/")]
    class ImageService : IImageContract
    {
        const string imageFileName = "image.jpg";

        Image bitmap;

        public ImageService()
        {
            this.bitmap = Image.FromFile(imageFileName);
        }

        public Stream GetImage()
        {
            MemoryStream stream = new MemoryStream();
            this.bitmap.Save(stream, ImageFormat.Jpeg);

            stream.Position = 0;
            WebOperationContext.Current.OutgoingResponse.ContentType = "image/jpeg";

            return stream;
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
        }
    }
}
```

下列範例會顯示與服務相關聯的 App.config 檔案。

```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <startup> 
        <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.5.2"/>
    </startup>
    <system.serviceModel>
        <extensions>
            <!-- In this extension section we are introducing all known service bus extensions. User can remove the ones they don't need. -->
            <behaviorExtensions>
                <add name="connectionStatusBehavior"
                    type="Microsoft.ServiceBus.Configuration.ConnectionStatusElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="transportClientEndpointBehavior"
                    type="Microsoft.ServiceBus.Configuration.TransportClientEndpointBehaviorElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="serviceRegistrySettings"
                    type="Microsoft.ServiceBus.Configuration.ServiceRegistrySettingsElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
            </behaviorExtensions>
            <bindingElementExtensions>
                <add name="netMessagingTransport"
                    type="Microsoft.ServiceBus.Messaging.Configuration.NetMessagingTransportExtensionElement, Microsoft.ServiceBus,  Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="tcpRelayTransport"
                    type="Microsoft.ServiceBus.Configuration.TcpRelayTransportElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="httpRelayTransport"
                    type="Microsoft.ServiceBus.Configuration.HttpRelayTransportElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="httpsRelayTransport"
                    type="Microsoft.ServiceBus.Configuration.HttpsRelayTransportElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="onewayRelayTransport"
                    type="Microsoft.ServiceBus.Configuration.RelayedOnewayTransportElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
            </bindingElementExtensions>
            <bindingExtensions>
                <add name="basicHttpRelayBinding"
                    type="Microsoft.ServiceBus.Configuration.BasicHttpRelayBindingCollectionElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="webHttpRelayBinding"
                    type="Microsoft.ServiceBus.Configuration.WebHttpRelayBindingCollectionElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="ws2007HttpRelayBinding"
                    type="Microsoft.ServiceBus.Configuration.WS2007HttpRelayBindingCollectionElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="netTcpRelayBinding"
                    type="Microsoft.ServiceBus.Configuration.NetTcpRelayBindingCollectionElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="netOnewayRelayBinding"
                    type="Microsoft.ServiceBus.Configuration.NetOnewayRelayBindingCollectionElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="netEventRelayBinding"
                    type="Microsoft.ServiceBus.Configuration.NetEventRelayBindingCollectionElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="netMessagingBinding"
                    type="Microsoft.ServiceBus.Messaging.Configuration.NetMessagingBindingCollectionElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
            </bindingExtensions>
        </extensions>
      <bindings>
        <!-- Application Binding -->
        <webHttpRelayBinding>
          <binding name="default">
            <security relayClientAuthenticationType="None" />
          </binding>
        </webHttpRelayBinding>
      </bindings>
      <services>
        <!-- Application Service -->
        <service name="Microsoft.ServiceBus.Samples.ImageService"
             behaviorConfiguration="default">
          <endpoint name="RelayEndpoint"
                  contract="Microsoft.ServiceBus.Samples.IImageContract"
                  binding="webHttpRelayBinding"
                  bindingConfiguration="default"
                  behaviorConfiguration="sbTokenProvider"
                  address="" />
        </service>
      </services>
      <behaviors>
        <endpointBehaviors>
          <behavior name="sbTokenProvider">
            <transportClientEndpointBehavior>
              <tokenProvider>
                <sharedAccessSignature keyName="RootManageSharedAccessKey" key="[SAS_KEY]" />
              </tokenProvider>
            </transportClientEndpointBehavior>
          </behavior>
        </endpointBehaviors>
        <serviceBehaviors>
          <behavior name="default">
            <serviceDebug httpHelpPageEnabled="false" httpsHelpPageEnabled="false" />
          </behavior>
        </serviceBehaviors>
      </behaviors>
    </system.serviceModel>
    <appSettings>
        <!-- Service Bus specific app setings for messaging connections -->
        <add key="Microsoft.ServiceBus.ConnectionString"
            value="Endpoint=sb://yourNamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=yourKey"/>
    </appSettings>
</configuration>
```

## 步驟 4：裝載 REST 架構 WCF 服務以使用服務匯流排

此步驟描述如何在服務匯流排上使用主控台應用程式來執行 Web 服務。程序後面的範例提供此步驟中撰寫的完整程式碼清單。

### 建立服務的基底位址

1. 在 `Main()` 函數宣告中，建立變數來儲存服務匯流排專案的命名空間。務必以您先前建立的服務命名空間名稱取代 `yourNamespace`。

	```
	string serviceNamespace = "yourNamespace";
	```
	服務匯流排會使用您的命名空間名稱來建立唯一 URI。

2. 為根據命名空間之服務的基底位址建立 `Uri` 執行個體。

	```
	Uri address = ServiceBusEnvironment.CreateServiceUri("https", serviceNamespace, "Image");
	```

### 建立和設定 Web 服務主機

- 建立 Web 服務主機，使用本節稍早建立的 URI 位址。

	```
	WebServiceHost host = new WebServiceHost(typeof(ImageService), address);
	```
	服務主機是具現化主機應用程式的 WCF 物件。此範例會將您要建立的主機類型 (**ImageService**)，及要公開主機應用程式的位址傳給它。

### 執行 Web 服務主機

1. 開啟服務。

	```
	host.Open();
	```
	服務現在正在執行。

2. 顯示訊息，指出服務正在執行，以及如何停止服務。

	```
	Console.WriteLine("Copy the following address into a browser to see the image: ");
	Console.WriteLine(address + "GetImage");
	Console.WriteLine();
	Console.WriteLine("Press [Enter] to exit");
	Console.ReadLine();
	```

3. 完成時，關閉服務主機。

	```
	host.Close();
	```

## 範例

下列範例包括教學課程先前步驟的服務合約和實作，以及在主控台應用程式中主控服務。將下列程式碼編譯為名為 ImageListener.exe 的可執行檔。

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.ServiceModel;
using System.ServiceModel.Channels;
using System.ServiceModel.Web;
using System.IO;
using System.Drawing;
using System.Drawing.Imaging;
using Microsoft.ServiceBus;
using Microsoft.ServiceBus.Web;

namespace Microsoft.ServiceBus.Samples
{

    [ServiceContract(Name = "ImageContract", Namespace = "http://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IImageContract
    {
        [OperationContract, WebGet]
        Stream GetImage();
    }

    public interface IImageChannel : IImageContract, IClientChannel { }

    [ServiceBehavior(Name = "ImageService", Namespace = "http://samples.microsoft.com/ServiceModel/Relay/")]
    class ImageService : IImageContract
    {
        const string imageFileName = "image.jpg";

        Image bitmap;

        public ImageService()
        {
            this.bitmap = Image.FromFile(imageFileName);
        }

        public Stream GetImage()
        {
            MemoryStream stream = new MemoryStream();
            this.bitmap.Save(stream, ImageFormat.Jpeg);

            stream.Position = 0;
            WebOperationContext.Current.OutgoingResponse.ContentType = "image/jpeg";

            return stream;
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
            string serviceNamespace = "InsertServiceNamespaceHere";
            Uri address = ServiceBusEnvironment.CreateServiceUri("https", serviceNamespace, "Image");

            WebServiceHost host = new WebServiceHost(typeof(ImageService), address);
            host.Open();

            Console.WriteLine("Copy the following address into a browser to see the image: ");
            Console.WriteLine(address + "GetImage");
            Console.WriteLine();
            Console.WriteLine("Press [Enter] to exit");
            Console.ReadLine();

            host.Close();
        }
    }
}
```

### 編譯程式碼

建置解決方案後，請執行下列命令以執行應用程式：

1. 按 **F5**，或瀏覽至可執行檔的位置 (ImageListener\\bin\\Debug\\ImageListener.exe) 來執行服務。讓應用程式保持執行狀態，因為需要它來執行下一個步驟。

2. 將位址從命令提示字元複製並貼到瀏覽器以查看映像。

3. 當您完成時，請按 [命令提示字元] 視窗中的 **Enter** 來關閉應用程式。

## 後續步驟

既然您已經建立使用服務匯流排轉送服務的應用程式，請參閱下列文章以進一步了解轉送傳訊：

- [Azure 服務匯流排架構概觀](service-bus-fundamentals-hybrid-solutions.md#relays)

- [如何使用服務匯流排轉送服務](service-bus-dotnet-how-to-use-relay.md)

[Azure 傳統入口網站]: http://manage.windowsazure.com

<!---HONumber=AcomDC_0608_2016-->