<properties
   pageTitle="連線到 Azure SQL 資料倉儲 | Microsoft Azure"
   description="如何尋找您的伺服器名稱和 Azure SQL 資料倉儲的連接字串"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="sonyam"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.devlang="NA"
   ms.topic="get-started-article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services"
   ms.date="09/26/2016"
   ms.author="sonyama;barbkess"/>

# 連接到 Azure SQL 資料倉儲

本文可協助您第一次連接到 SQL 資料倉儲。

## 尋找您的伺服器名稱

連接到 SQL 資料倉儲的第一個步驟是了解如何尋找您的伺服器名稱。例如，下列範例中的伺服器名稱是 sample.database.windows.net。若要尋找完整的伺服器名稱：

1. 移至 [Azure 入口網站][]。
2. 按一下 [SQL Database]
3. 按一下您想連接的資料庫。
4. 找出完整的伺服器名稱。

    ![完整伺服器名稱][1]

## 支援的驅動程式和連接字串

Azure SQL 資料倉儲支援 [ADO.NET][]、[ODBC][]、[PHP][] 和 [JDBC][]。按一下上述其中一個驅動程式，以尋找最新版本和文件。若要從 Azure 入口網站自動為您使用的驅動程式產生連接字串，您可以按一下前述範例中的 [顯示資料庫連接字串]。下列一些範例顯示每個驅動程式的連接字串。

> [AZURE.NOTE] 請考慮將連線逾時設定為 300 秒，以便在短時間無法使用時能夠維持連線。

### ADO.NET 連接字串範例

```C#
Server=tcp:{your_server}.database.windows.net,1433;Database={your_database};User ID={your_user_name};Password={your_password_here};Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;
```

### ODBC 連接字串範例

```C#
Driver={SQL Server Native Client 11.0};Server=tcp:{your_server}.database.windows.net,1433;Database={your_database};Uid={your_user_name};Pwd={your_password_here};Encrypt=yes;TrustServerCertificate=no;Connection Timeout=30;
```

### PHP 連接字串範例

```PHP
Server: {your_server}.database.windows.net,1433 \r\nSQL Database: {your_database}\r\nUser Name: {your_user_name}\r\n\r\nPHP Data Objects(PDO) Sample Code:\r\n\r\ntry {\r\n   $conn = new PDO ( "sqlsrv:server = tcp:{your_server}.database.windows.net,1433; Database = {your_database}", "{your_user_name}", "{your_password_here}");\r\n    $conn->setAttribute( PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION );\r\n}\r\ncatch ( PDOException $e ) {\r\n   print( "Error connecting to SQL Server." );\r\n   die(print_r($e));\r\n}\r\n\rSQL Server Extension Sample Code:\r\n\r\n$connectionInfo = array("UID" => "{your_user_name}", "pwd" => "{your_password_here}", "Database" => "{your_database}", "LoginTimeout" => 30, "Encrypt" => 1, "TrustServerCertificate" => 0);\r\n$serverName = "tcp:{your_server}.database.windows.net,1433";\r\n$conn = sqlsrv_connect($serverName, $connectionInfo);
```

### JDBC 連接字串範例

```Java
jdbc:sqlserver://yourserver.database.windows.net:1433;database=yourdatabase;user={your_user_name};password={your_password_here};encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.database.windows.net;loginTimeout=30;
```

## 連線設定

SQL 資料倉儲會在連線和物件建立期間將一些設定標準化。這些設定不能覆寫，其中包括︰

| 資料庫設定 | 值 |
| :--------------------- | :--------------------------- |
| [ANSI\_NULLS][] | 開啟 |
| [QUOTED\_IDENTIFIERS][] | 開啟 |
| [DATEFORMAT][] | mdy |
| [DATEFIRST][] | 7 |

## 後續步驟

若要使用 Visual Studio 連接及查詢，請參閱[使用 Visual Studio 查詢][]。若要深入了解驗證選項，請參閱[適用於 Azure SQL 資料倉儲的驗證][]。

<!--Articles-->
[使用 Visual Studio 查詢]: ./sql-data-warehouse-query-visual-studio.md
[適用於 Azure SQL 資料倉儲的驗證]: ./sql-data-warehouse-authentication.md

<!--MSDN references-->
[ADO.NET]: https://msdn.microsoft.com/library/e80y5yhx(v=vs.110).aspx
[ODBC]: https://msdn.microsoft.com/library/jj730314.aspx
[PHP]: https://msdn.microsoft.com/library/cc296172.aspx?f=255&MSPPError=-2147217396
[JDBC]: https://msdn.microsoft.com/library/mt484311(v=sql.110).aspx
[ANSI\_NULLS]: https://msdn.microsoft.com/library/ms188048.aspx
[QUOTED\_IDENTIFIERS]: https://msdn.microsoft.com/library/ms174393.aspx
[DATEFORMAT]: https://msdn.microsoft.com/library/ms189491.aspx
[DATEFIRST]: https://msdn.microsoft.com/library/ms181598.aspx

<!--Other-->
[Azure 入口網站]: https://portal.azure.com

<!--Image references-->
[1]: media/sql-data-warehouse-connect-overview/get-server-name.png

<!---HONumber=AcomDC_0928_2016-->