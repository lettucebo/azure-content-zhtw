<properties 
	pageTitle="使用 Panda 建立 Azure blob 儲存體資料功能 | Microsoft Azure" 
	description="如何使用 Panda Python 封裝對儲存在 Azure blob 容器的資料建立功能。" 
	services="machine-learning,storage" 
	documentationCenter="" 
	authors="bradsev" 
	manager="jhubbard" 
	editor="cgronlun" />

<tags 
	ms.service="machine-learning" 
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="09/19/2016" 
	ms.author="bradsev;garye" />

#使用 Panda 建立 Azure blob 儲存體資料功能

本文件說明如何使用 [Pandas](http://pandas.pydata.org/) Python 封裝，對儲存在 Azure blob 容器的資料建立特徵。概述如何將資料載入 Panda 資料框架後，接著會示範如何使用 Python 指令碼，搭配指標值和分類收納特徵，以產生分類特徵。

[AZURE.INCLUDE [cap-create-features-data-selector](../../includes/cap-create-features-selector.md)]
這個**功能表**所連結的主題會說明如何在各種環境中建立資料的特徵。此工作是 [Team Data Science Process (TDSP)](https://azure.microsoft.com/documentation/learning-paths/cortana-analytics-process/) 中的一個步驟。


## 必要條件
本文假設您已建立 Azure Blob 儲存體帳戶，並將您的資料儲存在該處。如需設定帳戶的指示，請參閱[建立 Azure 儲存體帳戶](../hdinsight-get-started.md#storage)


## 將資料載入至 Pandas 資料框架
若要進行探索和操作資料集，必須從 Blob 來源將資料集下載至本機檔案，然後將其載入 Pandas 資料框架中。以下是此程序的遵循步驟：

1. 使用 Blob 服務，透過下列 Python 程式碼範例，從 Azure Blob 下載資料。使用您的特定值來取代下列程式碼中的變數：

	    from azure.storage.blob import BlobService
    	import tables
    	
		STORAGEACCOUNTNAME= <storage_account_name>
		STORAGEACCOUNTKEY= <storage_account_key>
		LOCALFILENAME= <local_file_name>		
		CONTAINERNAME= <container_name>
		BLOBNAME= <blob_name>

    	#download from blob
    	t1=time.time()
    	blob_service=BlobService(account_name=STORAGEACCOUNTNAME,account_key=STORAGEACCOUNTKEY)
    	blob_service.get_blob_to_path(CONTAINERNAME,BLOBNAME,LOCALFILENAME)
    	t2=time.time()
    	print(("It takes %s seconds to download "+blobname) % (t2 - t1))


2. 從下載的檔案中，將資料讀取至 Pandas 資料框架。

	    #LOCALFILE is the file path	
    	dataframe_blobdata = pd.read_csv(LOCALFILE)

現在您已經準備好探索資料並在此資料集上產生功能。
	
##<a name="blob-featuregen"></a>功能產生
	
接下兩節會說明如何使用 Python 指令碼，產生帶有指標值和分類收納功能的分類功能。

###<a name="blob-countfeature"></a>以指標值為基礎的功能產生

類別功能可使用如下的方式來建立：

1. 檢查類別資料行的分佈：
	
		dataframe_blobdata['<categorical_column>'].value_counts()

2. 產生每個資料行值的指標值

		#generate the indicator column
		dataframe_blobdata_identity = pd.get_dummies(dataframe_blobdata['<categorical_column>'], prefix='<categorical_column>_identity')

3. 將指標資料行與原始資料框架聯結在一起
 
			#Join the dummy variables back to the original data frame
			dataframe_blobdata_with_identity = dataframe_blobdata.join(dataframe_blobdata_identity)

4. 移除原始變數本身：

		#Remove the original column rate_code in df1_with_dummy
		dataframe_blobdata_with_identity.drop('<categorical_column>', axis=1, inplace=True)
	
###<a name="blob-binningfeature"></a>分類收納功能產生

若要產生分類收納功能，我們可使用如下的方式繼續進行：

1. 新增一系列的資料行，以分類收納數值資料行
 
		bins = [0, 1, 2, 4, 10, 40]
		dataframe_blobdata_bin_id = pd.cut(dataframe_blobdata['<numeric_column>'], bins)
		
2. 將分類收納轉換為一系列的布林值變數

		dataframe_blobdata_bin_bool = pd.get_dummies(dataframe_blobdata_bin_id, prefix='<numeric_column>')
	
3. 最後，將虛擬變數新增回原始資料框架中

		dataframe_blobdata_with_bin_bool = dataframe_blobdata.join(dataframe_blobdata_bin_bool)	

##<a name="sql-featuregen"></a>將資料寫回 Azure Blob 並在 AzureMachine Learning 中取用

在您探索資料和建立必要功能後，可以上傳資料 (取樣性或功能性) 至 Azure Blob，並在 Azure Machine Learning 中透過下列步驟取用資料：請注意，您也可以在 Azure Machine Learning Studio 中建立其他功能。
1. 將資料框架寫入本機檔案中

		dataframe.to_csv(os.path.join(os.getcwd(),LOCALFILENAME), sep='\t', encoding='utf-8', index=False)

2. 將資料上傳至 Azure Blob，如下所示：

		from azure.storage.blob import BlobService
    	import tables

		STORAGEACCOUNTNAME= <storage_account_name>
		LOCALFILENAME= <local_file_name>
		STORAGEACCOUNTKEY= <storage_account_key>
		CONTAINERNAME= <container_name>
		BLOBNAME= <blob_name>

	    output_blob_service=BlobService(account_name=STORAGEACCOUNTNAME,account_key=STORAGEACCOUNTKEY)    
	    localfileprocessed = os.path.join(os.getcwd(),LOCALFILENAME) #assuming file is in current working directory
	    
	    try:
	   
	    #perform upload
	    output_blob_service.put_block_blob_from_path(CONTAINERNAME,BLOBNAME,localfileprocessed)
	    
	    except:	        
		    print ("Something went wrong with uploading blob:"+BLOBNAME)

3. 現在您可以使用 Azure Machine Learning [匯入資料](https://msdn.microsoft.com/library/azure/4e1b0fe6-aded-4b3f-a36f-39b8862b9004/)模組從 Blob 讀取資料，如以下畫面所示：
 
![讀取器 Blob](./media/machine-learning-data-science-process-data-blob/reader_blob.png)


 

<!---HONumber=AcomDC_0921_2016--->