<properties
	pageTitle="修正 SQL 連接錯誤、暫時性錯誤 |Microsoft Azure"
	description="了解如何在 Azure SQL Database 中排解、診斷和防止 SQL 連接錯誤或暫時性錯誤。"
	keywords="sql 連接, 連接字串, 連接問題, 暫時性錯誤, 連接錯誤"
	services="sql-database"
	documentationCenter=""
	authors="dalechen"
	manager="felixwu"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.workload="sql-database"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/20/2016"
	ms.author="daleche"/>


# 排解、診斷和防止 SQL Database 的 SQL 連接錯誤和暫時性錯誤

本文描述如何防止、排解、診斷和減少您的用戶端應用程式在與 Azure SQL Database 互動時發生的連接錯誤和暫時性錯誤。了解如何設定重試邏輯、建置連接字串和調整其他連接設定。

<a id="i-transient-faults" name="i-transient-faults"></a>

## 暫時性錯誤 (暫時性故障)

暫時性錯誤 (又稱暫時性故障) 具有很快就會自行解決的根本原因。當 Azure 系統快速地將硬體資源轉移到負載平衡更好的各種工作負載時，偶爾會發生暫時性錯誤。其中大部分重新設定事件通常會在不到 60 秒內完成。在此重新設定時間範圍期間，與 Azure SQL Database 的連接可能會有問題。建置連接到 Azure SQL Database 的應用程式時，應該預期這些暫時性錯誤，並實作重試邏輯來處理它們，而不是當做應用程式錯誤呈現給使用者。

如果用戶端程式使用 ADO.NET，系統會擲回 **SqlException**，告知您的程式發生暫時性錯誤。**Number** 屬性可以與下列主題中靠近上半部的暫時性錯誤清單進行比較：
[SQL Database 用戶端應用程式的 SQL 錯誤碼](sql-database-develop-error-messages.md)。

<a id="connection-versus-command" name="connection-versus-command"></a>

### 連接與命令

根據下列建立情況，您將重試 SQL 連接或再次建立 SQL 連接：

* **嘗試連接期間發生暫時性錯誤**：應該在延遲數秒後重試連接。

* **執行 SQL 查詢命令期間發生暫時性錯誤**：不應該立即重試命令。而是在延遲之後，應該建立全新的連接。然後，可以重試命令。


<a id="j-retry-logic-transient-faults" name="j-retry-logic-transient-faults"></a>

### 暫時性錯誤的重試邏輯


用戶端程式若包含重試邏輯，在偶爾遇到暫時性錯誤時就會更可靠。


當您的程式透過第三方中介軟體與 Azure SQL Database 進行通訊時，請洽詢廠商中介軟體是否包含暫時性錯誤的重試邏輯。

<a id="principles-for-retry" name="principles-for-retry"></a>

#### 重試原則


- 如果是暫時性錯誤，則應該重新嘗試開啟連接。


- 不應該直接重試由於暫時性錯誤而失敗的 SQL SELECT 陳述式。
 - 而是建立全新的連線，然後重試 SELECT。


- SQL UPDATE 陳述式由於暫時性錯誤而失敗時，應該先建立全新的連接，再重試 UPDATE。
 - 重試邏輯必須確保整個資料庫交易完成，或整個交易已復原。


#### 其他重試考量


- 下班後自動啟動的批次程式，以及將在早上前完成的批次程式，可以進行長時間間隔的重試。


- 使用者介面程式應該說明了人類將在長時間等候後放棄的傾向。
 - 不過，方案不得每隔幾秒鐘重試，因為該原則可能讓系統充斥要求。


#### 增加重試之間的間隔



我們建議您在您第一次重試前延遲 5 秒鐘。在少於 5 秒的延遲後重試，雲端服務會有超過負荷的風險。對於後續每次重試，延遲應以指數方式成長，最大值為 60 秒。

在 [SQL Server 連接集區 (ADO.NET)](http://msdn.microsoft.com/library/8xx3tyca.aspx) 中可找使用 ADO.NET 之用戶端的封鎖期間討論。

您也可能想要設定程式在自行終止之前的重試次數上限。


#### 具有重試邏輯的程式碼範例


各種程式設計語言中具有重試邏輯的程式碼範例位於：

- [SQL Database 和 SQL Server 的連線庫](sql-database-libraries.md)


<a id="k-test-retry-logic" name="k-test-retry-logic"></a>

#### 測試您的重試邏輯


若要測試您的重試邏輯，您必須在程式仍在執行時模擬或產生可以更正的錯誤。


##### 中斷與網路連接以進行測試


您可以測試重試邏輯的方法，就是在程式執行時中斷用戶端電腦與網路的連接。錯誤為：

- **SqlException.Number** = 11001
- 訊息：「未知的主機」


第一次重試時，您的程式可以更正拼字錯誤，然後嘗試連接。


若要使這個動作可行，請從網路拔除電腦，再啟動您的程式。然後，您的程式會辨識執行階段參數讓程式：

1. 暫時將 11001 加入至其錯誤清單，視為暫時性。
2. 如往常般嘗試其第一個連接。
3. 在攔截到錯誤之後，請從清單中移除 11001。
4. 顯示訊息告訴使用者將電腦連線到網路。
 - 使用 **Console.ReadLine** 方法或含 [確定] 按鈕的對話方塊，暫停進一步執行。將電腦插入網路中之後。使用者按下 Enter 鍵。
5. 重新嘗試連接，預期成功。


##### 連接時拼錯資料庫名稱以進行測試


在第一次連接嘗試之前，您的程式可以故意拼錯使用者名稱。錯誤為：

- **SqlException.Number** = 18456
- 錯誤將為：「使用者 'WRONG\_MyUserName' 登入失敗。」


第一次重試時，您的程式可以更正拼字錯誤，然後嘗試連接。


若要使這個動作可行，您的程式可以辨識執行階段參數讓程式：

1. 暫時將 18456 加入至其錯誤清單，視為暫時性。
2. 故意將 'WRONG\_' 加入至使用者名稱。
3. 在攔截到錯誤之後，請從清單中移除 18456。
4. 從使用者名稱中移除 'WRONG\_'。
5. 重新嘗試連接，預期成功。

<a id="net-sqlconnection-parameters-for-connection-retry" name="net-sqlconnection-parameters-for-connection-retry"></a>

### 進行連線重試的.NET SqlConnection 參數


如果您的用戶端程式利用 .NET Framework 類別 **System.Data.SqlClient.SqlConnection** 連接到 Azure SQL Database，您應該使用 .NET 4.6.1 或更新版本，以利用它的連線重試功能。此功能的詳細資料在[這裡](http://go.microsoft.com/fwlink/?linkid=393996)。


<!--
2015-11-30, FwLink 393996 points to dn632678.aspx, which links to a downloadable .docx related to SqlClient and SQL Server 2014.
-->


當您為 **SqlConnection** 物件建立[連接字串](http://msdn.microsoft.com/library/System.Data.SqlClient.SqlConnection.connectionstring.aspx)時，您應該調整下列參數的值：

- ConnectRetryCount &nbsp;&nbsp;*(預設值為 1。範圍是 0 到 255)。*
- ConnectRetryInterval &nbsp;&nbsp;*(預設值為 1 秒。範圍是 1 到 60)。*
- Connection Timeout &nbsp;&nbsp;*(預設值為 15 秒。範圍是 0 到 2147483647)*


具體來說，您選擇的值應該會讓下列等式成立：

- Connection Timeout = ConnectRetryCount * ConnectionRetryInterval

例如，如果計數 = 3，且間隔 = 10 秒，則連線時，僅 29 秒的逾時將無法給予系統充足的時間以進行第三次及最後一次的重試：29 < 3 * 10。

<a id="connection-versus-command" name="connection-versus-command"></a>

### 連接與命令


**ConnectRetryCount** 和 **ConnectRetryInterval** 參數讓您的 **SqlConnection** 物件可重試連接作業，而不需告知或中斷您的程式，例如將控制權傳回您的程式。可能會在以下情況中發生重試：

- mySqlConnection.Open 方法呼叫
- mySqlConnection.Execute 方法呼叫

有一些微妙的差異。當執行「查詢」時若發生暫時性錯誤，您的 **SqlConnection** 物件將不會重試連接作業，且絕對不會重試查詢。不過，在傳送您的查詢以供執行之前，**SqlConnection** 會先快速檢查連接。如果快速檢查偵測到連接問題，**SqlConnection** 會重試連接作業。如果重試成功，就會傳送您的查詢以供執行。


#### 是否應該將 ConnectRetryCount 與應用程式重試邏輯結合？

假設您的應用程式有健全的自訂重試邏輯。它可能會重試連接作業 4 次。如果您將 **ConnectRetryInterval** 和 **ConnectRetryCount** =3 加入到連接字串，您會將重試次數增加為 4 * 3 = 12 次重試。您可能不會想要這麼多的重試次數。

<a id="a-connection-connection-string" name="a-connection-connection-string"></a>

## 連接 Azure SQL Database

<a id="c-connection-string" name="c-connection-string"></a>

### 連接：連接字串


連接到 Azure SQL Database 所需的連接字串和連接到 Microsoft SQL Server 的字串稍有不同。您可以從 [Azure 入口網站](https://portal.azure.com/)複製資料庫的連接字串。


[AZURE.INCLUDE [sql-database-include-connection-string-20-portalshots](../../includes/sql-database-include-connection-string-20-portalshots.md)]


<a id="b-connection-ip-address" name="b-connection-ip-address"></a>

### 連接：IP 位址


您必須設定 SQL Database 伺服器，以接受來自裝載您的用戶端程式之電腦的通訊。您可以透過 [Azure 入口網站](https://portal.azure.com/)編輯防火牆設定來執行此動作。


如果您忘了設定 IP 位址，您的程式將會失敗並出現一個好用的錯誤訊息，陳述必要的 IP 位址。


[AZURE.INCLUDE [sql-database-include-ip-address-22-v12portal](../../includes/sql-database-include-ip-address-22-v12portal.md)]


如需詳細資訊，請參閱：
[作法：在 SQL Database 上進行防火牆設定](sql-database-configure-firewall-settings.md)


<a id="c-connection-ports" name="c-connection-ports"></a>

### 連接：連接埠


通常，您只需要在裝載用戶端程式的電腦上確定已開啟連接埠 1433 進行輸出通訊。


例如，當用戶端程式裝載在 Windows 電腦上時，主機上的 Windows 防火牆可讓您開啟通訊埠 1433：


1. 開啟 [控制台]
2. &gt; 所有控制台項目
3. &gt; Windows 防火牆
4. &gt; 進階設定
5. &gt; 輸出規則
6. &gt; 動作
7. &gt; 新增規則


如果您的用戶端程式裝載在 Azure 虛擬機器 (VM) 上，您應該閱讀：<br/>[1433 以外供 ADO.NET 4.5 和 SQL Database V12 使用的連接埠](sql-database-develop-direct-route-ports-adonet-v12.md)。


如需設定連接埠及 IP 位址的背景資訊，請參閱：
[Azure SQL Database 防火牆](sql-database-firewall-configure.md)


<a id="d-connection-ado-net-4-5" name="d-connection-ado-net-4-5"></a>

### 連接：ADO.NET 4.6.1


如果您的程式使用類似 **System.Data.SqlClient.SqlConnection** 的 ADO.NET 類別來連接到 Azure SQL Database，建議您使用 .NET Framework 4.6.1 版或更新的版本。


ADO.NET 4.6.1：

- 對於 Azure SQL Database，使用 **SqlConnection.Open** 方法來開啟連接時，可靠性更高。針對連接逾時期間內的特定錯誤，**Open** 方法現在包含最佳重試機制來因應暫時性錯誤。
- 支援連接共用。這包括提供給您的程式的連接物件是否運作的有效驗證。



當您從連接集區使用連接物件時，建議您的程式若未立即使用連接，則暫時關閉它。重新開啟連接比建立新的連接更便宜。


如果您使用 ADO.NET 4.0 或更舊版本，建議您升級到最新的 ADO.NET。

- 從 2015 年 11 月開始，您可以[下載 ADO.NET 4.6.1](http://blogs.msdn.com/b/dotnet/archive/2015/11/30/net-framework-4-6-1-is-now-available.aspx)。


<a id="e-diagnostics-test-utilities-connect" name="e-diagnostics-test-utilities-connect"></a>

## 診斷

<a id="d-test-whether-utilities-can-connect" name="d-test-whether-utilities-can-connect"></a>

### 診斷：測試公用程式是否可以連接


如果您的程式無法連接到 Azure SQL Database，有一個診斷選項將嘗試與公用程式連接。理想的情況下，此公用程式會使用您的程式使用的同一程式庫來連接。


您可以在任何 Windows 電腦上，嘗試這些公用程式：

- SQL Server Management Studio (SSMS.exe)，連接時使用 ADO.NET。
- sqlcmd.exe，連接時使用 [ODBC](http://msdn.microsoft.com/library/jj730308.aspx)。


一旦完成連接，就會測試簡短的 SQL SELECT 查詢是否可以運作。


<a id="f-diagnostics-check-open-ports" name="f-diagnostics-check-open-ports"></a>

### 診斷：檢查開啟的連接埠


假設您懷疑連接嘗試由於連接埠問題而失敗。在您的電腦上，您可以執行報告連接埠組態的公用程式。


下列公用程式在 Linux 上可能很有用：

- `netstat -nap`
- `nmap -sS -O 127.0.0.1`
 - (將範例值變更為您的 IP 位址)。


在 Windows 上，[PortQry.exe](http://www.microsoft.com/download/details.aspx?id=17148) 公用程式可能很有用。以下是在 Azure SQL Database 伺服器上查詢連接埠情況，以及在膝上型電腦上執行的的範例執行：


```
[C:\Users\johndoe]
>> portqry.exe -n johndoesvr9.database.windows.net -p tcp -e 1433

Querying target system called:
 johndoesvr9.database.windows.net

Attempting to resolve name to IP address...
Name resolved to 23.100.117.95

querying...
TCP port 1433 (ms-sql-s service): LISTENING

[C:\Users\johndoe]
>>
```


<a id="g-diagnostics-log-your-errors" name="g-diagnostics-log-your-errors"></a>

### 診斷：記錄您的錯誤


有時診斷間歇問題的最好方式，就是數天或數週偵測一般模式。


您的用戶端可以記錄其遇到的所有錯誤來協助診斷。您可以使記錄項目與 Azure SQL Database 本身內部記錄的錯誤資料相互關聯。


Enterprise Library 6 (EntLib60) 提供 .NET Managed 類別來協助記錄：

- [5 - 輕而易舉：使用記錄應用程式區塊](http://msdn.microsoft.com/library/dn440731.aspx)


<a id="h-diagnostics-examine-logs-errors" name="h-diagnostics-examine-logs-errors"></a>

### 診斷：檢查系統記錄找出錯誤


以下是一些查詢錯誤記錄和其他資訊的 Transact-SQL SELECT 陳述式。


| 記錄查詢 | 說明 |
| :-- | :-- |
| `SELECT e.*`<br/>`FROM sys.event_log AS e`<br/>`WHERE e.database_name = 'myDbName'`<br/>`AND e.event_category = 'connectivity'`<br/>`AND 2 >= DateDiff`<br/>&nbsp;&nbsp;`(hour, e.end_time, GetUtcDate())`<br/>`ORDER BY e.event_category,`<br/>&nbsp;&nbsp;`e.event_type, e.end_time;` | [Sys.event\_log](http://msdn.microsoft.com/library/dn270018.aspx) 檢視提供個別事件的相關資訊，包括可能導致暫時性錯誤或連線失敗的一些事件。<br/><br/>理想的情況下，您可以使 **start\_time** 或 **end\_time** 值與用戶端應用程式何時遇到問題的相關資訊相互關聯。<br/><br/>**秘訣：**您必須連接到 **master** 資料庫才能執行此動作。 |
| `SELECT c.*`<br/>`FROM sys.database_connection_stats AS c`<br/>`WHERE c.database_name = 'myDbName'`<br/>`AND 24 >= DateDiff`<br/>&nbsp;&nbsp;`(hour, c.end_time, GetUtcDate())`<br/>`ORDER BY c.end_time;` | [Sys.database\_connection\_stats](http://msdn.microsoft.com/library/dn269986.aspx) 檢視針對其他診斷提供事件類型的彙總計數。<br/><br/>**秘訣：**您必須連接到 **master** 資料庫才能執行此動作。 |

<a id="d-search-for-problem-events-in-the-sql-database-log" name="d-search-for-problem-events-in-the-sql-database-log"></a>

### 診斷：在 SQL Database 記錄中搜尋問題事件


您可以在 Azure SQL Database 的記錄中搜尋有關問題事件的項目。在 **master** 資料庫中嘗試下列 Transact-SQL SELECT 陳述式：


```
SELECT
   object_name
  ,CAST(f.event_data as XML).value
      ('(/event/@timestamp)[1]', 'datetime2')                      AS [timestamp]
  ,CAST(f.event_data as XML).value
      ('(/event/data[@name="error"]/value)[1]', 'int')             AS [error]
  ,CAST(f.event_data as XML).value
      ('(/event/data[@name="state"]/value)[1]', 'int')             AS [state]
  ,CAST(f.event_data as XML).value
      ('(/event/data[@name="is_success"]/value)[1]', 'bit')        AS [is_success]
  ,CAST(f.event_data as XML).value
      ('(/event/data[@name="database_name"]/value)[1]', 'sysname') AS [database_name]
FROM
  sys.fn_xe_telemetry_blob_target_read_file('el', null, null, null) AS f
WHERE
  object_name != 'login_event'  -- Login events are numerous.
  and
  '2015-06-21' < CAST(f.event_data as XML).value
        ('(/event/@timestamp)[1]', 'datetime2')
ORDER BY
  [timestamp] DESC
;
```


#### 數個從 sys.fn\_xe\_telemetry\_blob\_target\_read\_file 傳回的資料列


接下來是傳回的資料列可能的樣子。顯示的 null 值通常在其他資料列不是 null。


```
object_name                   timestamp                    error  state  is_success  database_name

database_xml_deadlock_report  2015-10-16 20:28:01.0090000  NULL   NULL   NULL        AdventureWorks
```


<a id="l-enterprise-library-6" name="l-enterprise-library-6"></a>

## Enterprise Library 6


Enterprise Library 6 (EntLib60) 是 .NET 類別的架構，可協助您實作雲端服務的健全用戶端，其中之一就是 Azure SQL Database 服務。您可以先造訪下列網址來找出 EntLib60 所能協助之每個領域的專用主題：

- [Enterprise Library 6 – 2013 年 4 月](http://msdn.microsoft.com/library/dn169621%28v=pandp.60%29.aspx)


在 EntLib60 可以協助的一個領域中用於處理暫時性錯誤的重試邏輯：

- [4 - 堅持是所有成功的秘方：使用暫時性錯誤處理應用程式區塊](http://msdn.microsoft.com/library/dn440719%28v=pandp.60%29.aspx)


> [AZURE.NOTE] EntLib60 的原始程式碼已可公開[下載](http://go.microsoft.com/fwlink/p/?LinkID=290898)。Microsoft 沒有計劃進一步更新或維護 EntLib 的功能。

<a id="entlib60-classes-for-transient-errors-and-retry" name="entlib60-classes-for-transient-errors-and-retry"></a>

### 用於暫時性錯誤和重試的 EntLib60 類別


下列 EntLib60 類別特別有助於重試邏輯。這些全部都在命名空間 **Microsoft.Practices.EnterpriseLibrary.TransientFaultHandling** 中，或進一步位於其下：

在命名空間 Microsoft.Practices.EnterpriseLibrary.TransientFaultHandling 中：

- **RetryPolicy** 類別
 - **ExecuteAction** 方法


- **ExponentialBackoff** 類別


- **SqlDatabaseTransientErrorDetectionStrategy** 類別


- **ReliableSqlConnection** 類別
 - **ExecuteCommand** 方法


在命名空間 **Microsoft.Practices.EnterpriseLibrary.TransientFaultHandling.TestSupport** 中：

- **AlwaysTransientErrorDetectionStrategy** 類別

- **NeverTransientErrorDetectionStrategy** 類別


以下是 EntLib60 相關資訊的連結：

- 免費的[書籍下載：Microsoft Enterprise Library 開發人員指南第 2 版](http://www.microsoft.com/download/details.aspx?id=41145)

- 最佳作法：[重試一般指引](../best-practices-retry-general.md)深入探討重試邏輯。

- 可在 NuGet 下載 [Enterprise Library - 暫時性錯誤處理應用程式區塊 6.0](http://www.nuget.org/packages/EnterpriseLibrary.TransientFaultHandling/)

<a id="entlib60-the-logging-block" name="entlib60-the-logging-block"></a>

### EntLib60：記錄區塊


- 記錄區塊是高度彈性且可設定的解決方案，可讓您：
 - 建立記錄訊息，並儲存在各種不同的位置中。
 - 分類與篩選訊息。
 - 收集有助於偵錯和追蹤的內容資訊，以及用於稽核和一般記錄需求的內容資訊。


- 記錄區塊可彙總來自記錄目的地的記錄功能，使應用程式程式碼能夠一致，而不必理會目標記錄存放區的的位置和類型。


如需詳細資料，請參閱：
[5 - 輕而易舉：使用記錄應用程式區塊](https://msdn.microsoft.com/library/dn440731%28v=pandp.60%29.aspx)

<a id="entlib60-istransient-method-source-code" name="entlib60-istransient-method-source-code"></a>

### EntLib60 IsTransient 方法的原始程式碼


接下來，**IsTransient** 方法的 C# 原始程式碼來自 **SqlDatabaseTransientErrorDetectionStrategy** 類別。原始程式碼將釐清哪些錯誤會被視為暫時性並值得重試 (從 2013 年 4 月起)。

為了強調可讀性，已從此副本移除許多 **//comment** 行。


```
public bool IsTransient(Exception ex)
{
  if (ex != null)
  {
    SqlException sqlException;
    if ((sqlException = ex as SqlException) != null)
    {
      // Enumerate through all errors found in the exception.
      foreach (SqlError err in sqlException.Errors)
      {
        switch (err.Number)
        {
            // SQL Error Code: 40501
            // The service is currently busy. Retry the request after 10 seconds.
            // Code: (reason code to be decoded).
          case ThrottlingCondition.ThrottlingErrorNumber:
            // Decode the reason code from the error message to
            // determine the grounds for throttling.
            var condition = ThrottlingCondition.FromError(err);

            // Attach the decoded values as additional attributes to
            // the original SQL exception.
            sqlException.Data[condition.ThrottlingMode.GetType().Name] =
              condition.ThrottlingMode.ToString();
            sqlException.Data[condition.GetType().Name] = condition;

            return true;

          case 10928:
          case 10929:
          case 10053:
          case 10054:
          case 10060:
          case 40197:
          case 40540:
          case 40613:
          case 40143:
          case 233:
          case 64:
            // DBNETLIB Error Code: 20
            // The instance of SQL Server you attempted to connect to
            // does not support encryption.
          case (int)ProcessNetLibErrorCode.EncryptionNotSupported:
            return true;
        }
      }
    }
    else if (ex is TimeoutException)
    {
      return true;
    }
    else
    {
      EntityException entityException;
      if ((entityException = ex as EntityException) != null)
      {
        return this.IsTransient(entityException.InnerException);
      }
    }
  }

  return false;
}
```


## 後續步驟

- 如需針對其他常見的 Azure SQL Database 連接問題進行疑難排解，請造訪[針對 Azure SQL Database 連線問題進行疑難排解](sql-database-troubleshoot-common-connection-issues.md)。

- [SQL Server 連接集區 (ADO.NET)](http://msdn.microsoft.com/library/8xx3tyca.aspx)


- [*重試*是 Apache 2.0 授權的一般用途重試文件庫，以 **Python** 撰寫，幾乎可對任何案例加入重試作業。](https://pypi.python.org/pypi/retrying)

<!---HONumber=AcomDC_0921_2016-->