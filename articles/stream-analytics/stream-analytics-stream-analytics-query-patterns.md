<properties
	pageTitle="串流分析中一般使用模式的查詢範例 | Microsoft Azure"
	description="常見的 Azure 串流分析查詢模式 "
	keywords="查詢範例"
	services="stream-analytics"
	documentationCenter=""
	authors="jeffstokes72"
	manager="jhubbard"
	editor="cgronlun"/>

<tags
	ms.service="stream-analytics"
	ms.devlang="na"
	ms.topic="article"
	ms.tgt_pltfrm="na"
	ms.workload="big-data"
	ms.date="07/27/2016"
	ms.author="jeffstok"/>


# 一般串流分析使用模式的查詢範例

## 簡介

Azure 串流分析中的查詢以類似 SQL 的查詢語言來表示，您可以參閱[串流分析查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)指南來進一步了解。本文章根據真實世界案例概述幾個常見查詢模式的解決方案。這是進行中的工作，並將繼續不間斷使用新模式進行更新。

## 查詢範例：資料類型轉換
**描述**：在輸入資料流上定義屬性類型。例如，汽車權數即將出現在輸入資料流做為字串，並且需要轉換為 INT 以執行 SUM 來加總。

**輸入**：

| Make | 時間 | Weight |
| --- | --- | --- |
| Honda | 2015-01-01T00:00:01.0000000Z | "1000" |
| Honda | 2015-01-01T00:00:02.0000000Z | "2000" |

**輸出**：

| Make | 重量 |
| --- | --- |
| Honda | 3000 |

**解決方案**：

	SELECT
    	Make,
    	SUM(CAST(Weight AS BIGINT)) AS Weight
	FROM
    	Input TIMESTAMP BY Time
	GROUP BY
		Make,
    	TumblingWindow(second, 10)

**說明**：使用權數欄位上的 CAST 陳述式指定其類型 (請參閱[這裡](https://msdn.microsoft.com/library/azure/dn835065.aspx)的受支援資料類型清單)。


## 查詢範例：使用 Like/Not like 進行模式比對
**描述**：檢查事件的欄位值是否符合某個模式，例如傳回欄位值的事件符合特定模式的核取例如傳回以 A 開頭和以 9 結尾的車牌號碼

**輸入**：

| Make | LicensePlate | 時間 |
| --- | --- | --- |
| Honda | ABC-123 | 2015-01-01T00:00:01.0000000Z |
| Toyota | AAA-999 | 2015-01-01T00:00:02.0000000Z |
| Nissan | ABC-369 | 2015-01-01T00:00:03.0000000Z |

**輸出**：

| Make | LicensePlate | 時間 |
| --- | --- | --- |
| Toyota | AAA-999 | 2015-01-01T00:00:02.0000000Z |
| Nissan | ABC-369 | 2015-01-01T00:00:03.0000000Z |

**解決方案**：

	SELECT
    	*
	FROM
    	Input TIMESTAMP BY Time
	WHERE
    	LicensePlate LIKE 'A%9'

**說明**：使用 LIKE 陳述式檢查 LicensePlate 欄位值是否以 A 開頭，有沒有任何零或多個字元的字串，並以 9 結尾。

## 查詢範例：為不同的案例/值指定邏輯 (CASE 陳述式)
**描述**：根據一些準則提供欄位的不同運算。例如，利用特殊案例 1 提供有多少相同品牌的汽車通過的字串描述。

**輸入**：

| Make | 時間 |
| --- | --- |
| Honda | 2015-01-01T00:00:01.0000000Z |
| Toyota | 2015-01-01T00:00:02.0000000Z |
| Toyota | 2015-01-01T00:00:03.0000000Z |

**輸出**：

| CarsPassed | 時間 |
| --- | --- | --- |
| 1 Honda | 2015-01-01T00:00:10.0000000Z |
| 2 Toyotas | 2015-01-01T00:00:10.0000000Z |

**解決方案**：

    SELECT
    	CASE
			WHEN COUNT(*) = 1 THEN CONCAT('1 ', Make)
			ELSE CONCAT(CAST(COUNT(*) AS NVARCHAR(MAX)), ' ', Make, 's')
		END AS CarsPassed,
		System.TimeStamp AS Time
	FROM
		Input TIMESTAMP BY Time
	GROUP BY
		Make,
		TumblingWindow(second, 10)

**說明**：CASE 子句可讓我們根據一些準則提供不同的運算 (在我們的案例中，汽車計數在彙總視窗中)。

## 查詢範例：將資料傳送至多個輸出
**描述**：將資料從單一工作傳送到多個輸出目標。例如，分析以臨界值為基礎之警示的資料，並將所有事件封存至 blob 儲存體

**輸入**：

| Make | 時間 |
| --- | --- |
| Honda | 2015-01-01T00:00:01.0000000Z |
| Honda | 2015-01-01T00:00:02.0000000Z |
| Toyota | 2015-01-01T00:00:01.0000000Z |
| Toyota | 2015-01-01T00:00:02.0000000Z |
| Toyota | 2015-01-01T00:00:03.0000000Z |

**Output1**：

| Make | 時間 |
| --- | --- |
| Honda | 2015-01-01T00:00:01.0000000Z |
| Honda | 2015-01-01T00:00:02.0000000Z |
| Toyota | 2015-01-01T00:00:01.0000000Z |
| Toyota | 2015-01-01T00:00:02.0000000Z |
| Toyota | 2015-01-01T00:00:03.0000000Z |

**Output2**：

| Make | 時間 | Count |
| --- | --- | --- |
| Toyota | 2015-01-01T00:00:10.0000000Z | 3 |

**解決方案**：

	SELECT
		*
	INTO
		ArchiveOutput
	FROM
		Input TIMESTAMP BY Time

	SELECT
		Make,
		System.TimeStamp AS Time,
		COUNT(*) AS [Count]
	INTO
		AlertOutput
	FROM
		Input TIMESTAMP BY Time
	GROUP BY
		Make,
		TumblingWindow(second, 10)
	HAVING
		[Count] >= 3

**說明**：INTO 子句會告訴串流分析要從此陳述式將資料寫入哪個輸出。第一個查詢將我們接收到的資料傳遞至我們命名為 ArchiveOutput 的輸出。第二個查詢會執行一些簡單的彙總和篩選，並將結果傳送至下游的警示系統。*注意*：您也可以在多個輸出陳述式中重複使用 CTE 的結果 (例如 WITH 陳述式) – 這對於開放輸入來源給較少讀取器有額外的好處。例如

	WITH AllRedCars AS (
		SELECT
			*
		FROM
			Input TIMESTAMP BY Time
		WHERE
			Color = 'red'
	)
	SELECT * INTO HondaOutput FROM AllRedCars WHERE Make = 'Honda'
	SELECT * INTO ToyotaOutput FROM AllRedCars WHERE Make = 'Toyota'

## 查詢範例：計算唯一值
**描述**：計算串流在某個時間範圍內出現的唯一欄位值之數目。例如，在 2 秒鐘的時間範圍內，通過收費站的唯一汽車品牌有多少個？

**輸入**：

| Make | 時間 |
| --- | --- |
| Honda | 2015-01-01T00:00:01.0000000Z |
| Honda | 2015-01-01T00:00:02.0000000Z |
| Toyota | 2015-01-01T00:00:01.0000000Z |
| Toyota | 2015-01-01T00:00:02.0000000Z |
| Toyota | 2015-01-01T00:00:03.0000000Z |

**輸出：**

| Count | 時間 |
| --- | --- |
| 2 | 2015-01-01T00:00:02.000Z |
| 1 | 2015-01-01T00:00:04.000Z |

**解決方案：**

	WITH Makes AS (
	    SELECT
	        Make,
	        COUNT(*) AS CountMake
	    FROM
	        Input TIMESTAMP BY Time
	    GROUP BY
	          Make,
	          TumblingWindow(second, 2)
	)
	SELECT
	    COUNT(*) AS Count,
	    System.TimeStamp AS Time
	FROM
	    Makes
	GROUP BY
	    TumblingWindow(second, 1)


**說明：**我們要執行第一次彙總，以取得時間範圍內的唯一品牌及其數目。然後我們會執行取得多少品牌的彙總 – 假設一個視窗中的所有唯一值都取得相同的時間戳記，則第二個彙總視窗必須是最小的，才不會從第一個步驟彙總 2 個視窗。

## 查詢範例：判斷值是否已變更#
**描述**：查看前一個值來判斷該值是否與目前的值不同。例如收費道路上前一輛汽車的品牌，是否與目前汽車的品牌相同？

**輸入**：

| Make | 時間 |
| --- | --- |
| Honda | 2015-01-01T00:00:01.0000000Z |
| Toyota | 2015-01-01T00:00:02.0000000Z |

**輸出**：

| Make | 時間 |
| --- | --- |
| Toyota | 2015-01-01T00:00:02.0000000Z |

**解決方案**：

	SELECT
		Make,
		Time
	FROM
		Input TIMESTAMP BY Time
	WHERE
		LAG(Make, 1) OVER (LIMIT DURATION(minute, 1)) <> Make

**說明**：使用 LAG 查看前一個事件的輸入串流，並取得品牌值。然後將其和目前事件中的品牌比較，並且在兩者不同時輸出事件。

## 查詢範例：時間範圍內的第一個事件
**描述**：是否要每隔 10 分鐘尋找第一輛車？

**輸入**：

| LicensePlate | Make | 時間 |
| --- | --- | --- |
| DXE 5291 | Honda | 2015-07-27T00:00:05.0000000Z |
| YZK 5704 | Ford | 2015-07-27T00:02:17.0000000Z |
| RMV 8282 | Honda | 2015-07-27T00:05:01.0000000Z |
| YHN 6970 | Toyota | 2015-07-27T00:06:00.0000000Z |
| VFE 1616 | Toyota | 2015-07-27T00:09:31.0000000Z |
| QYF 9358 | Honda | 2015-07-27T00:12:02.0000000Z |
| MDR 6128 | BMW | 2015-07-27T00:13:45.0000000Z |

**輸出**：

| LicensePlate | Make | 時間 |
| --- | --- | --- |
| DXE 5291 | Honda | 2015-07-27T00:00:05.0000000Z |
| QYF 9358 | Honda | 2015-07-27T00:12:02.0000000Z |

**解決方案**：

	SELECT 
		LicensePlate,
		Make,
		Time
	FROM 
		Input TIMESTAMP BY Time
	WHERE 
		IsFirst(minute, 10) = 1

現在讓我們來變更問題，每隔 10 分鐘尋找特定品牌的第一輛車。

| LicensePlate | Make | 時間 |
| --- | --- | --- |
| DXE 5291 | Honda | 2015-07-27T00:00:05.0000000Z |
| YZK 5704 | Ford | 2015-07-27T00:02:17.0000000Z |
| YHN 6970 | Toyota | 2015-07-27T00:06:00.0000000Z |
| QYF 9358 | Honda | 2015-07-27T00:12:02.0000000Z |
| MDR 6128 | BMW | 2015-07-27T00:13:45.0000000Z |

**解決方案**：

	SELECT 
		LicensePlate,
		Make,
		Time
	FROM 
		Input TIMESTAMP BY Time
	WHERE 
		IsFirst(minute, 10) OVER (PARTITION BY Make) = 1

## 查詢範例：時間範圍內的上一個事件
**描述**：每隔 10 分鐘尋找上一輛車。

**輸入**：

| LicensePlate | Make | 時間 |
| --- | --- | --- |
| DXE 5291 | Honda | 2015-07-27T00:00:05.0000000Z |
| YZK 5704 | Ford | 2015-07-27T00:02:17.0000000Z |
| RMV 8282 | Honda | 2015-07-27T00:05:01.0000000Z |
| YHN 6970 | Toyota | 2015-07-27T00:06:00.0000000Z |
| VFE 1616 | Toyota | 2015-07-27T00:09:31.0000000Z |
| QYF 9358 | Honda | 2015-07-27T00:12:02.0000000Z |
| MDR 6128 | BMW | 2015-07-27T00:13:45.0000000Z |

**輸出**：

| LicensePlate | Make | 時間 |
| --- | --- | --- |
| VFE 1616 | Toyota | 2015-07-27T00:09:31.0000000Z |
| MDR 6128 | BMW | 2015-07-27T00:13:45.0000000Z |

**解決方案**：

	WITH LastInWindow AS
	(
		SELECT 
			MAX(Time) AS LastEventTime
		FROM 
			Input TIMESTAMP BY Time
		GROUP BY 
			TumblingWindow(minute, 10)
	)
	SELECT 
		Input.LicensePlate,
		Input.Make,
		Input.Time
	FROM
		Input TIMESTAMP BY Time 
		INNER JOIN LastInWindow
		ON DATEDIFF(minute, Input, LastInWindow) BETWEEN 0 AND 10
		AND Input.Time = LastInWindow.LastEventTime

**說明**：查詢中有兩個步驟 – 第一個是在 10 分鐘視窗中尋找最新的時間戳記。第二個步驟會聯結第一個查詢的結果與原始串流，以在每個視窗中尋找符合最後一個時間戳記的事件。

## 查詢範例：偵測到事件不存在
**描述**：查看沒有值的串流是否符合特定準則。例如，在 90 秒鐘內，是否有連續 2 輛相同品牌的汽車進入收費道路？

**輸入**：

| Make | LicensePlate | 時間 |
| --- | --- | --- |
| Honda | ABC-123 | 2015-01-01T00:00:01.0000000Z |
| Honda | AAA-999 | 2015-01-01T00:00:02.0000000Z |
| Toyota | DEF-987 | 2015-01-01T00:00:03.0000000Z |
| Honda | GHI-345 | 2015-01-01T00:00:04.0000000Z |

**輸出**：

| Make | 時間 | CurrentCarLicensePlate | FirstCarLicensePlate | FirstCarTime |
| --- | --- | --- | --- | --- |
| Honda | 2015-01-01T00:00:02.0000000Z | AAA-999 | ABC-123 | 2015-01-01T00:00:01.0000000Z |

**解決方案**：

	SELECT
	    Make,
	    Time,
	    LicensePlate AS CurrentCarLicensePlate,
	    LAG(LicensePlate, 1) OVER (LIMIT DURATION(second, 90)) AS FirstCarLicensePlate,
	    LAG(Time, 1) OVER (LIMIT DURATION(second, 90)) AS FirstCarTime
	FROM
	    Input TIMESTAMP BY Time
	WHERE
	    LAG(Make, 1) OVER (LIMIT DURATION(second, 90)) = Make

**說明**：使用 LAG 查看前一個事件的輸入串流，並取得品牌值。然後將其和目前事件中的品牌比較，在兩者相同時輸出事件並使用 LAG 取得上一輛車的相關資料。

## 查詢範例：偵測事件與事件之間的持續時間
**描述**：尋找特定事件的時序時間。例如，假設某個 Web 點選流會判斷花在某個功能上的時間。

**輸入**：
  
| User | 功能 | Event | Time |
| --- | --- | --- | --- |
| user@location.com | RightMenu | Start | 2015-01-01T00:00:01.0000000Z |
| user@location.com | RightMenu | End | 2015-01-01T00:00:08.0000000Z |
  
**輸出**：
  
| User | Feature | Duration |
| --- | --- | --- |
| user@location.com | RightMenu | 7 |
  

**解決方案**

````
    SELECT
    	[user], feature, DATEDIFF(second, LAST(Time) OVER (PARTITION BY [user], feature LIMIT DURATION(hour, 1) WHEN Event = 'start'), Time) as duration
    FROM input TIMESTAMP BY Time
    WHERE
    	Event = 'end'
````

**說明**：使用 LAST 函式，在事件類型為「Start」時擷取最後一個時間值。請注意，LAST 函式使用 PARTITION BY [user] 來表示，應該要分別為每位獨特的使用者來計算結果。查詢的「Start」與「Stop」事件之間時間差的上限為 1 小時，但該值可以在必要時變更 (LIMIT DURATION(hour, 1)。

## 查詢範例：偵測某個情況的持續時間
**描述**：找出某個情況的持續時間。例如，假設有個錯誤導致所有車輛的重量不正確 (超過 20,000 磅)，而我們想要計算該錯誤的持續時間。

**輸入**：

| Make | 時間 | Weight |
| --- | --- | --- |
| Honda | 2015-01-01T00:00:01.0000000Z | 2000 |
| Toyota | 2015-01-01T00:00:02.0000000Z | 25000 |
| Honda | 2015-01-01T00:00:03.0000000Z | 26000 |
| Toyota | 2015-01-01T00:00:04.0000000Z | 25000 |
| Honda | 2015-01-01T00:00:05.0000000Z | 26000 |
| Toyota | 2015-01-01T00:00:06.0000000Z | 25000 |
| Honda | 2015-01-01T00:00:07.0000000Z | 26000 |
| Toyota | 2015-01-01T00:00:08.0000000Z | 2000 |

**輸出**：

| StartFault | EndFault |
| --- | --- |
| 2015-01-01T00:00:02.000Z | 2015-01-01T00:00:07.000Z |

**解決方案**：

````
	WITH SelectPreviousEvent AS
	(
	SELECT
	*,
		LAG([time]) OVER (LIMIT DURATION(hour, 24)) as previousTime,
		LAG([weight]) OVER (LIMIT DURATION(hour, 24)) as previousWeight
	FROM input TIMESTAMP BY [time]
	)

	SELECT 
    	LAG(time) OVER (LIMIT DURATION(hour, 24) WHEN previousWeight < 20000 ) [StartFault],
    	previousTime [EndFault]
	FROM SelectPreviousEvent
	WHERE
    	[weight] < 20000
	    AND previousWeight > 20000
````

**說明**：使用 LAG 來檢視 24 小時內的輸入資料流，並尋找其中的 StartFault 和 StopFault 被 weight < 20000 合併的執行個體。

## 查詢範例：填入遺漏值
**描述**：對於有遺漏值的事件串流，以固定間隔產生事件串流。例如，每隔 5 秒產生事件，報告最近所見的資料點。

**輸入**：

| t | value |
|--------------------------|-------|
| "2014-01-01T06:01:00" | 1 |
| "2014-01-01T06:01:05" | 2 |
| "2014-01-01T06:01:10" | 3 |
| "2014-01-01T06:01:15" | 4 |
| "2014-01-01T06:01:30" | 5 |
| "2014-01-01T06:01:35" | 6 |

**輸出 (前 10 個資料列)**：

| windowend | lastevent.t | lastevent.value |
|--------------------------|--------------------------|--------|
| 2014-01-01T14:01:00.000Z | 2014-01-01T14:01:00.000Z | 1 |
| 2014-01-01T14:01:05.000Z | 2014-01-01T14:01:05.000Z | 2 |
| 2014-01-01T14:01:10.000Z | 2014-01-01T14:01:10.000Z | 3 |
| 2014-01-01T14:01:15.000Z | 2014-01-01T14:01:15.000Z | 4 |
| 2014-01-01T14:01:20.000Z | 2014-01-01T14:01:15.000Z | 4 |
| 2014-01-01T14:01:25.000Z | 2014-01-01T14:01:15.000Z | 4 |
| 2014-01-01T14:01:30.000Z | 2014-01-01T14:01:30.000Z | 5 |
| 2014-01-01T14:01:35.000Z | 2014-01-01T14:01:35.000Z | 6 |
| 2014-01-01T14:01:40.000Z | 2014-01-01T14:01:35.000Z | 6 |
| 2014-01-01T14:01:45.000Z | 2014-01-01T14:01:35.000Z | 6 |

    
**解決方案**：

    SELECT
    	System.Timestamp AS windowEnd,
    	TopOne() OVER (ORDER BY t DESC) AS lastEvent
    FROM
    	input TIMESTAMP BY t
    GROUP BY HOPPINGWINDOW(second, 300, 5)


**說明**：此查詢會每隔 5 秒產生事件，並且會輸出之前收到的最後一個事件。[跳動視窗](https://msdn.microsoft.com/library/dn835041.aspx "跳動視窗 - Azure 串流分析")持續時間會決定查詢回溯到多久之前以找出最新的事件 (在此範例中為 300 秒)。


## 取得說明
如需進一步的協助，請參閱我們的 [Azure Stream Analytics 論壇](https://social.msdn.microsoft.com/Forums/zh-TW/home?forum=AzureStreamAnalytics)

## 後續步驟

- [Azure Stream Analytics 介紹](stream-analytics-introduction.md)
- [開始使用 Azure Stream Analytics](stream-analytics-get-started.md)
- [調整 Azure Stream Analytics 工作](stream-analytics-scale-jobs.md)
- [Azure Stream Analytics 查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)
- [Azure 串流分析管理 REST API 參考](https://msdn.microsoft.com/library/azure/dn835031.aspx)
 

<!---HONumber=AcomDC_0921_2016-->