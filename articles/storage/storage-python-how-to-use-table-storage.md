<properties
	pageTitle="如何使用 Python 的資料表儲存體 | Microsoft Azure"
	description="使用 Azure 表格儲存體 (NoSQL 資料存放區) 將結構化的資料儲存在雲端。"
	services="storage"
	documentationCenter="python"
	authors="tamram"
	manager="carmonm"
	editor="tysonn"/>

<tags
	ms.service="storage"
	ms.workload="storage"
	ms.tgt_pltfrm="na"
	ms.devlang="python"
	ms.topic="article"
	ms.date="09/20/2016"
	ms.author="gusapost;tamram"/>


# 如何使用 Python 的資料表儲存體

[AZURE.INCLUDE [storage-selector-table-include](../../includes/storage-selector-table-include.md)]
<br/>
[AZURE.INCLUDE [storage-try-azure-tools-tables](../../includes/storage-try-azure-tools-tables.md)]

## Overview

本指南說明如何使用 Azure 資料表儲存體服務執行一般案例。這些範例是以 Python 所撰寫，並使用 [Microsoft Azure Storage SDK for Python (適用於 Python 的 Microsoft Azure 儲存體 SDK)]。除了「在資料表中插入及查詢實體」外，所涵蓋的案例還包括「建立和刪除資料表」。

[AZURE.INCLUDE [storage-table-concepts-include](../../includes/storage-table-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../../includes/storage-create-account-include.md)]

## 建立資料表

**TableService** 物件可讓您操作資料表服務。下列程式碼將建立 **TableService** 物件。將程式碼新增至您想要在其中以程式設計方式存取 Azure 儲存體之任何 Python 檔案內的頂端附近：

	from azure.storage.table import TableService, Entity

下列程式碼會使用儲存體帳戶名稱和帳戶金鑰來建立 **TableService** 物件。將 'myaccount' 和 'mykey' 取代為您的帳戶名稱和金鑰。

	table_service = TableService(account_name='myaccount', account_key='mykey')

	table_service.create_table('tasktable')

## 將實體加入至資料表

若要新增實體，請先建立字典或定義實體屬性名稱和值的實體。請注意，您必須為每個實體指定 **PartitionKey** 及 **RowKey**。這些是您的實體的唯一識別碼。比起查詢其他屬性，您可以使用這些值更快地查詢。系統使用 **PartitionKey** 自動將資料表的實體分散在許多儲存體節點上。具有相同 **PartitionKey** 的實體會儲存在相同節點上。**RowKey** 是實體在其所屬資料分割內的唯一識別碼。

若要將實體新增至資料表，請將字典物件傳給 **insert\_entity** 方法。

	task = {'PartitionKey': 'tasksSeattle', 'RowKey': '1', 'description' : 'Take out the trash', 'priority' : 200}
	table_service.insert_entity('tasktable', task)

您也可以將 **Entity** 類別的執行個體傳給 **insert\_entity** 方法。

	task = Entity()
	task.PartitionKey = 'tasksSeattle'
	task.RowKey = '2'
	task.description = 'Wash the car'
	task.priority = 100
	table_service.insert_entity('tasktable', task)

## 更新實體

此程式碼說明如何以更新版本取代舊版本的現有實體。

	task = {'PartitionKey': 'tasksSeattle', 'RowKey': '1', 'description' : 'Take out the garbage', 'priority' : 250}
	table_service.update_entity('tasktable', task)

如果正在更新的實體不存在，則更新操作便會失敗。如果您想要儲存實體 (不論它是否已存在)，請使用 **insert\_or\_replace\_entity**。在下列範例中，第一個呼叫將取代現有實體。第二個呼叫將插入新實體，因為資料表中沒有具有指定 **PartitionKey** 和 **RowKey** 的實體存在。

	task = {'PartitionKey': 'tasksSeattle', 'RowKey': '1', 'description' : 'Take out the garbage again', 'priority' : 250}
	table_service.insert_or_replace_entity('tasktable', task)

	task = {'PartitionKey': 'tasksSeattle', 'RowKey': '3', 'description' : 'Buy detergent', 'priority' : 300}
	table_service.insert_or_replace_entity('tasktable', task)

## 變更一組實體

有時候批次提交多個操作是有意義的，可以確保伺服器會進行不可部分完成的處理。若要完成此操作，您可以使用 **TableBatch** 類別。當您真的想提交批次時，請呼叫 **commit\_batch**。請注意，所有實體都必須位於相同的資料分割中，才能以批次方式進行變更。以下範例在一個批次中同時新增兩個實體。

	from azure.storage.table import TableBatch
	batch = TableBatch()
	task10 = {'PartitionKey': 'tasksSeattle', 'RowKey': '10', 'description' : 'Go grocery shopping', 'priority' : 400}
	task11 = {'PartitionKey': 'tasksSeattle', 'RowKey': '11', 'description' : 'Clean the bathroom', 'priority' : 100}
	batch.insert_entity(task10)
	batch.insert_entity(task11)
	table_service.commit_batch('tasktable', batch)

Batch 也可以與內容管理員語法搭配使用：

	task12 = {'PartitionKey': 'tasksSeattle', 'RowKey': '12', 'description' : 'Go grocery shopping', 'priority' : 400}
	task13 = {'PartitionKey': 'tasksSeattle', 'RowKey': '13', 'description' : 'Clean the bathroom', 'priority' : 100}

	with table_service.batch('tasktable') as batch:
		batch.insert_entity(task12)
		batch.insert_entity(task13)


## 查詢實體

若要查詢資料表中的實體，請使用 **get\_entity** 方法，傳遞 **PartitionKey** 和 **RowKey**。

	task = table_service.get_entity('tasktable', 'tasksSeattle', '1')
	print(task.description)
	print(task.priority)

## 查詢實體集合

此範例會根據 **PartitionKey** 找到在西雅圖的所有工作。

	tasks = table_service.query_entities('tasktable', filter="PartitionKey eq 'tasksSeattle'")
	for task in tasks:
		print(task.description)
		print(task.priority)

## 查詢實體屬性的子集

一項資料表查詢可以只擷取實體的少數屬性。這項稱為「投射」的技術可減少頻寬並提高查詢效能 (尤其是對大型實體而言)。使用 **select** 參數並傳遞您要帶到用戶端的屬性名稱。

下列程式碼中的查詢只會傳回資料表中各實體的說明。

[AZURE.NOTE] 下列程式碼片段只針對雲端儲存體服務運作。儲存體模擬器並不支援此程式碼片段。

	tasks = table_service.query_entities('tasktable', filter="PartitionKey eq 'tasksSeattle'", select='description')
	for task in tasks:
		print(task.description)

## 刪除實體

您可以使用實體的資料分割和資料列索引鍵來刪除實體。

	table_service.delete_entity('tasktable', 'tasksSeattle', '1')

## 刪除資料表

下列程式碼會從儲存體帳戶刪除資料表。

	table_service.delete_table('tasktable')

## 後續步驟

了解表格儲存體的基本概念之後，請使用下列連結深入了解。

- [Python 開發人員中心](/develop/python/)
- [Azure 儲存體服務 REST API](http://msdn.microsoft.com/library/azure/dd179355)
- [Azure 儲存體團隊部落格]
- [Microsoft Azure Storage SDK for Python]

[Azure 儲存體團隊部落格]: http://blogs.msdn.com/b/windowsazurestorage/
[Microsoft Azure Storage SDK for Python]: https://github.com/Azure/azure-storage-python
[Microsoft Azure Storage SDK for Python (適用於 Python 的 Microsoft Azure 儲存體 SDK)]: https://github.com/Azure/azure-storage-python

<!---HONumber=AcomDC_0921_2016-->