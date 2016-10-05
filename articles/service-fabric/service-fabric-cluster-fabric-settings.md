
<properties
   pageTitle="自訂 Service Fabric 叢集設定和網狀架構升級原則 | Microsoft Azure"
   description="本文說明您可以自訂的網狀架構設定和網狀架構升級原則。"
   services="service-fabric"
   documentationCenter=".net"
   authors="chackdan"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="09/20/2016"
   ms.author="chackdan"/>

# 自訂 Service Fabric 叢集設定和網狀架構升級原則

本文將告訴您如何為 Service Fabric 叢集自訂各種網狀架構設定和網狀架構升級原則。您可以在入口網站上或使用 Resource Manager 範本自訂。

## 您可以自訂的網狀架構設定


以下是您可以自訂的網狀架構設定：

### 區段名稱︰Security

|**參數**|**允許的值**|**指引或簡短描述**|
|-----------------------|--------------------------|--------------------------|
|ClusterProtectionLevel|None 或 EncryptAndSign| 不安全的叢集為 None (預設值)，安全的叢集為 EncryptAndSign。 |

### 區段名稱：Hosting

|**參數**|**允許的值**|**指引或簡短描述**|
|-----------------------|--------------------------|--------------------------|
|ServiceTypeRegistrationTimeout|時間 (秒)，預設值為 300| 允許 ServiceType 向網狀架構註冊的最大時間|
|ServiceTypeDisableFailureThreshold|整數，預設值為 1| 這是失敗次數的閾值，超過此值之後，就會通知 FailoverManager (FM) 停用該節點上的服務類型，並嘗試放置在另一個節點。|
|ActivationRetryBackoffInterval|時間 (秒)，預設值為 5|每次啟用失敗的輪詢間隔；在每次連續啟用失敗之後，系統會重試啟用最多 MaxActivationFailureCount 次。每次嘗試的重試間隔是連續啟用失敗與啟用輪詢間隔的乘積。|
|ActivationMaxRetryInterval|時間 (秒)，預設值為 300| 在每次連續啟用失敗時，系統會重試啟用最多 ActivationMaxFailureCount 次。ActivationMaxRetryInterval 指定每次啟用失敗之後在重試之前等待的時間間隔 |
|ActivationMaxFailureCount|整數，預設值為 10| 系統在放棄之前重試失敗啟用的次數 |

### 區段名稱︰FailoverManager

|**參數**|**允許的值**|**指引或簡短描述**|
|-----------------------|--------------------------|--------------------------|
|PeriodicLoadPersistInterval|時間 (秒)，預設值為 10| 這決定 FM 檢查是否有新負載報告的頻率|

### 區段名稱︰Federation

|**參數**|**允許的值**|**指引或簡短描述**|
|-----------------------|--------------------------|--------------------------|
|LeaseDuration|時間 (秒)，預設值為 30|節點與其相鄰節點之間的租用持續時間。|
|LeaseDurationAcrossFaultDomain|時間 (秒)，預設值為 30|所有容錯網域的節點與其相鄰節點之間的租用持續時間。|

### 區段名稱︰ClusterManager

|**參數**|**允許的值**|**指引或簡短描述**|
|-----------------------|--------------------------|--------------------------|
|UpgradeStatusPollInterval|時間 (秒)，預設值為 60|輪詢應用程式升級狀態的頻率。此值決定任何 GetApplicationUpgradeProgress 呼叫的更新速率|
|UpgradeHealthCheckInterval|時間 (秒)，預設值為 60|受監視應用程式升級期間的健康狀態檢查頻率|
|FabricUpgradeStatusPollInterval|時間 (秒)，預設值為 60|輪詢網狀架構升級狀態的頻率。此值決定任何 GetFabricUpgradeProgress 呼叫的更新速率 |
|FabricUpgradeHealthCheckInterval|時間 (秒)，預設值為 60|受監視網狀架構升級期間的健康狀態檢查頻率|



## 後續步驟

如需有關叢集管理的詳細資訊，請參閱下列文件︰

[新增、變換、移除 Azure 叢集的憑證](service-fabric-cluster-security-update-certs-azure.md)

<!---HONumber=AcomDC_0921_2016-->