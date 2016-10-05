**基本資料湖分析程序：**
	
![Azure 資料湖分析程序流程圖](./media/data-lake-analytics-basic-process-include/data-lake-analytics-process.png)
	
1. 建立資料湖分析帳戶。
2. 準備來源資料。資料湖分析工作可讀取 Azure 資料湖存放區帳戶或 Azure Blob 儲存體帳戶資料中的資料。
3. 開發 U SQL 指令碼。
4. 將工作 (U-SQL 指令碼) 提交至資料湖分析帳戶。此工作會讀取來源資料、依照 U-SQL 指令碼中的指示處理資料，然後將輸出儲存至資料湖存放區帳戶或 Blob 儲存體帳戶。

<!---HONumber=AcomDC_0921_2016-->