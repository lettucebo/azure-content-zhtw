<properties
	pageTitle="如何使用 Xamarin 的 Blob 儲存體 | Microsoft Azure"
	description="適用於 Xamarin 的 Azure 儲存體用戶端程式庫可讓開發人員使用其原生的使用者介面以建立 iOS、Android 和 Windows 市集應用程式。本教學課程示範如何使用 Xamarin 來建立使用 Azure Blob 儲存體的應用程式。"
	services="storage"
	documentationCenter="xamarin"
	authors="micurd"
	manager="jahogg"
	editor="tysonn"/>

<tags
	ms.service="storage"
	ms.workload="storage"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/31/2016"
	ms.author="micurd;tamram"/>

# 如何使用 Xamarin 的 Blob 儲存體

[AZURE.INCLUDE [storage-selector-blob-include](../../includes/storage-selector-blob-include.md)]

## Overview

Xamarin 可讓開發人員使用共用的 C# 程式碼基底，使用其原生的使用者介面以建立 iOS、Android 和 Windows 市集應用程式。本教學課程會示範如何搭配 Xamarin 應用程式使用 Azure Blob 儲存體。如果您想要深入了解 Azure 儲存體，在一頭栽進程式碼之前，請先參閱 [Microsoft Azure 儲存體簡介](storage-introduction.md)。

[AZURE.INCLUDE [storage-create-account-include](../../includes/storage-create-account-include.md)]

[AZURE.INCLUDE [storage-mobile-authentication-guidance](../../includes/storage-mobile-authentication-guidance.md)]

## 建立新的 Xamarin 應用程式

針對此快速入門，我們將建立以 Android、iOS 和 Windows 為目標的應用程式。此應用程式只會建立容器，並將 blob 上傳到此容器。針對此快速入門，我們將在 Windows 上使用 Visual Studio，但是在 Mac OS 上建立使用 Xamarin Studio 的應用程式時，可以套用相同的做法。

請遵循下列步驟來建立應用程式：

1. 下載並安裝 [Xamarin for Visual Studio](https://www.xamarin.com/download) (若您尚未這麼做)。
2. 開啟 Visual Studio，建立一個空白的應用程式 (原生共用)︰[檔案] > [新增] > [專案] > [跨平台] > [空白應用程式 (原生共用)]。
3. 以滑鼠右鍵按一下 [方案總管] 窗格中的方案，然後選取 [管理方案的 NuGet 套件]。搜尋 **WindowsAzure.Storage**，將最新穩定版安裝到您方案中的所有專案。
4. 建置並執行專案。

您現在應該有一個應用程式，在您按一下按鈕時就會遞增計數器。

> [AZURE.NOTE] Xamarin 的 Azure 儲存體用戶端程式庫也適用於 Xamarin.Forms 應用程式。

## 建立容器並上傳 blob

接下來，您需要新增一些程式碼到共用類別 `MyClass.cs` 以建立一個容器，並將 blob 上傳到此容器。`MyClass.cs` 看起來應該如下所示︰

	using Microsoft.WindowsAzure.Storage;
	using Microsoft.WindowsAzure.Storage.Blob;
	using System.Threading.Tasks;

	namespace XamarinApp
	{
		public class MyClass
		{
			public MyClass ()
			{
			}

		    public static async Task createContainerAndUpload()
		    {
		        // Retrieve storage account from connection string.
		        CloudStorageAccount storageAccount = CloudStorageAccount.Parse("DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here");

		        // Create the blob client.
		        CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

		        // Retrieve reference to a previously created container.
		        CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");

				// Create the container if it doesn't already exist.
            	await container.CreateIfNotExistsAsync();

		        // Retrieve reference to a blob named "myblob".
		        CloudBlockBlob blockBlob = container.GetBlockBlobReference("myblob");

		        // Create the "myblob" blob with the text "Hello, world!"
		        await blockBlob.UploadTextAsync("Hello, world!");
		    }
		}
	}

務必將 "your\_account\_name\_here" 和 "your\_account\_key\_here" 取代為您的實際帳戶名稱和帳戶金鑰。然後，您可以在您的 iOS、Android 和 Windows Phone 應用程式中使用此共用類別。您可以直接將 `MyClass.createContainerAndUpload()` 加入每個專案。例如：

### XamarinApp.Droid > MainActivity.cs

	using Android.App;
	using Android.Widget;
	using Android.OS;

	namespace XamarinApp.Droid
	{
		[Activity (Label = "XamarinApp.Droid", MainLauncher = true, Icon = "@drawable/icon")]
		public class MainActivity : Activity
		{
			int count = 1;

			protected override async void OnCreate (Bundle bundle)
			{
				base.OnCreate (bundle);

				// Set our view from the "main" layout resource
				SetContentView (Resource.Layout.Main);

				// Get our button from the layout resource,
				// and attach an event to it
				Button button = FindViewById<Button> (Resource.Id.myButton);

				button.Click += delegate {
					button.Text = string.Format ("{0} clicks!", count++);
				};

	      	  await MyClass.createContainerAndUpload();
			}
		}
	}

### XamarinApp.iOS > ViewController.cs

	using System;
	using UIKit;

	namespace XamarinApp.iOS
	{
		public partial class ViewController : UIViewController
		{
			int count = 1;

			public ViewController (IntPtr handle) : base (handle)
			{
			}

			public override async void ViewDidLoad ()
			{
				base.ViewDidLoad ();
				// Perform any additional setup after loading the view, typically from a nib.
				Button.AccessibilityIdentifier = "myButton";
				Button.TouchUpInside += delegate {
					var title = string.Format ("{0} clicks!", count++);
					Button.SetTitle (title, UIControlState.Normal);
				};

	            await MyClass.createContainerAndUpload();
	    	}

			public override void DidReceiveMemoryWarning ()
			{
				base.DidReceiveMemoryWarning ();
				// Release any cached data, images, etc that aren't in use.
			}
		}
	}

### XamarinApp.WinPhone > MainPage.xaml > MainPage.xaml.cs

	using Windows.UI.Xaml.Controls;
	using Windows.UI.Xaml.Navigation;

	// The Blank Page item template is documented at http://go.microsoft.com/fwlink/?LinkId=391641

	namespace XamarinApp.WinPhone
	{
	    /// <summary>
	    /// An empty page that can be used on its own or navigated to within a Frame.
	    /// </summary>
	    public sealed partial class MainPage : Page
	    {
	        int count = 1;

	        public MainPage()
	        {
	            this.InitializeComponent();

	            this.NavigationCacheMode = NavigationCacheMode.Required;
	        }

	        /// <summary>
	        /// Invoked when this page is about to be displayed in a Frame.
	        /// </summary>
	        /// <param name="e">Event data that describes how this page was reached.
	        /// This parameter is typically used to configure the page.</param>
	        protected override async void OnNavigatedTo(NavigationEventArgs e)
	        {
	            // TODO: Prepare page for display here.

	            // TODO: If your application contains multiple pages, ensure that you are
	            // handling the hardware Back button by registering for the
	            // Windows.Phone.UI.Input.HardwareButtons.BackPressed event.
	            // If you are using the NavigationHelper provided by some templates,
	            // this event is handled for you.
	            Button.Click += delegate {
	                var title = string.Format("{0} clicks!", count++);
	                Button.Content = title;
	            };

	            await MyClass.createContainerAndUpload();
	        }
	    }
	}


## 執行應用程式

您現在可以在 Android 或 Windows Phone 模擬器中執行這個應用程式。您也可以在 iOS 模擬器中執行此應用程式，但這需要 Mac 作業系統。如需具體的做法指示，請閱讀[將 Visual Studio 連接到 Mac](https://developer.xamarin.com/guides/ios/getting_started/installation/windows/connecting-to-mac/) 的文件。

一旦您執行應用程式，它會在您的儲存體帳戶中建立 `mycontainer` 容器。它應該包含 blob `myblob`，其具有文字 `Hello, world!`。您可以使用 [Microsoft Azure 儲存體總管](http://storageexplorer.com/)來驗證。

## 後續步驟

在此快速入門中，您已學會如何在使用 Azure 儲存體的 Xamarin 中建立跨平台的應用程式。此快速入門具體的重點是 Blob 儲存體中的一個案例。不過，除了 Blob 儲存體以外，您還可以處理更多，例如資料表、檔案和佇列儲存體。若要深入了解，請參閱下列文章：
- [以 .NET 開始使用 Azure Blob 儲存體](storage-dotnet-how-to-use-blobs.md)
- [以 .NET 開始使用 Azure 資料表儲存體](storage-dotnet-how-to-use-tables.md)
- [以 .NET 開始使用 Azure 佇列儲存體](storage-dotnet-how-to-use-queues.md)
- [在 Windows 上開始使用 Azure 檔案儲存體](storage-dotnet-how-to-use-files.md)

[AZURE.INCLUDE [storage-try-azure-tools-blobs](../../includes/storage-try-azure-tools-blobs.md)]

<!---HONumber=AcomDC_0921_2016-->