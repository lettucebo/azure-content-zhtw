Azure Data Factory 支援下列可個別或與其他活動鏈結而新增至管線的轉換活動。

資料轉換活動 | 計算環境 
:----------------------- | :--------------------
[Hive](../articles/data-factory/data-factory-hive-activity.md) | HDInsight [Hadoop] [Pig](../articles/data-factory/data-factory-pig-activity.md) | HDInsight [Hadoop] [MapReduce](../articles/data-factory/data-factory-map-reduce.md) | HDInsight [Hadoop] [Hadoop Streaming](../articles/data-factory/data-factory-hadoop-streaming-activity.md) | HDInsight [Hadoop] [Machine Learning 活動︰批次執行和更新資源](../articles/data-factory/data-factory-azure-ml-batch-execution-activity.md) | Azure VM 
[預存程序](../articles/data-factory/data-factory-stored-proc-activity.md) | Azure SQL、Azure SQL 資料倉儲或 SQL Server |
[資料湖分析 U-SQL](../articles/data-factory/data-factory-usql-activity.md) | Azure 資料湖分析 
[DotNet](../articles/data-factory/data-factory-use-custom-activities.md) | HDInsight [Hadoop] 或 Azure Batch
   
> [AZURE.NOTE] 
您可以使用 MapReduce 活動，在 HDInsight Spark 叢集上執行 Spark 程式。如需詳細資訊，請參閱[從 Azure Data Factory 叫用 Spark 程式](../articles/data-factory/data-factory-spark.md)。您可以建立自訂活動，以便在已安裝 R 的 HDInsight 叢集上執行 R 指令碼。請參閱 [使用 Azure Data Factory 執行 R 指令碼](https://github.com/Azure/Azure-DataFactory/tree/master/Samples/RunRScriptUsingADFSample)。

<!---HONumber=AcomDC_0928_2016-->