<properties 
	pageTitle="如何使用 Azure 命令列介面 (Azure CLI) 建立並管理 Azure Redis 快取 | Microsoft Azure" 
	description="了解如何在任何平台上安裝 Azure CLI、如何使用它來連接到您的 Azure 帳戶，以及如何從 Azure CLI 建立和管理 Redis 快取。" 
	services="redis-cache" 
	documentationCenter="" 
	authors="steved0x" 
	manager="douge" 
	editor=""/>

<tags 
	ms.service="cache" 
	ms.workload="tbd" 
	ms.tgt_pltfrm="cache-redis" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="09/15/2016" 
	ms.author="sdanie"/>

# 如何使用 Azure 命令列介面 (Azure CLI) 建立並管理 Azure Redis 快取

> [AZURE.SELECTOR]
- [PowerShell](cache-howto-manage-redis-cache-powershell.md)
- [Azure CLI](cache-manage-cli.md)

Azure CLI 是從任何平台管理 Azure 基礎結構的一個好方法。本文將說明如何使用 Azure CLI 建立並管理 Azure Redis 快取執行個體。

## 必要條件

若要使用 Azure CLI 建立並管理 Azure Redis 快取執行個體，您必須完成下列步驟：

-	您必須具有 Azure 帳號。如果您沒有帳戶，只需要幾分鐘的時間就可以建立[免費帳戶](https://azure.microsoft.com/pricing/free-trial/)。
-	[安裝 Azure CLI](../xplat-cli-install.md)。
-	使用個人 Azure 帳戶，或是使用工作或學校的 Azure 帳戶連接 Azure CLI 安裝，並使用 `azure login` 命令從 Azure CLI 登入。若要了解其中的差異和選擇，請參閱[從 Azure 命令列介面 (Azure CLI) 連線到 Azure 訂用帳戶](../xplat-cli-connect.md)。
-	在執行以下任何命令之前，執行 `azure config mode arm` 命令將 Azure CLI 切換至資源管理員模式。如需詳細資訊，請參閱[設定 Azure Resource Manager 模式](../xplat-cli-azure-resource-manager.md#set-the-azure-resource-manager-mode)。

## Redis 快取屬性

當建立並更新 Redis 快取執行個體時會使用下列屬性。

| 屬性 | Switch | 說明 |
|---------------------|-----------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 名稱 | -n, --name | Redis 快取的名稱。 |
| 資源群組 | -g, --resource-group | 資源群組的名稱。 |
| location | -l, --location | 要建立快取的位置。 |
| size | -z, --size | Redis 快取的大小。有效的值：[C0, C1, C2, C3, C4, C5, C6, P1, P2, P3, P4] |
| sku | -x, --sku | Redis SKU。應為其中一個：[Basic, Standard, Premium] |
| EnableNonSslPort | -e, --enable-non-ssl-port | Redis 快取的 EnableNonSslPort 屬性。如果您想針對您的快取啟用非 SSL 連接埠，則加入此旗標 |
| Redis 組態 | -c, --redis-configuration | Redis 組態。在這裡輸入組態金鑰和值的 JSON 格式字串。格式："{"":"","":""}" |
| Redis 組態 | -f, --redis-configuration-file | Redis 組態。在這裡輸入包含組態金鑰和值的檔案路徑。檔案項目的格式：{"":"","":""} |
| 分區計數 | -r, --shard-count | 建立在具有叢集之進階叢集快取的分區數目。 |
| 虛擬網路 | -v, --virtual-network | 當快取是裝載在 VNET 中時，指定確切的虛擬網路 ARM 資源識別碼以將 Redis 快取部署到其中。範例格式：/subscriptions/{subid}/resourceGroups/{resourceGroupName}/Microsoft.ClassicNetwork/VirtualNetworks/vnet1 |
| 金鑰類型 | -t, --key-type | 要更新的金鑰類型。有效的值：[Primary, Secondary] |
| StaticIP | -p, --static-ip <static-ip> | 當快取是裝載在 VNET 中，為快取在子網路中指定唯一 IP 位址。如果未提供，則會從子網路中為您選擇一個。 |
| 子網路 | t, --subnet <subnet> | 當快取是裝載在 VNET 中，指定要在其中部署快取的子網路。 |
| VirtualNetwork | -v, --virtual-network <virtual-network> | 當快取是裝載在 VNET 中時，指定確切的虛擬網路 ARM 資源識別碼以將 Redis 快取部署到其中。範例格式：/subscriptions/{subid}/resourceGroups/{resourceGroupName}/Microsoft.ClassicNetwork/VirtualNetworks/vnet1 |
| 訂用帳戶 | -s, --subscription | 訂用帳戶識別碼。 |

## 查看所有的 Redis 快取命令

若要查看所有的 Redis 快取命令和參數，請使用 `azure rediscache -h` 命令。

	C:\>azure rediscache -h
	help:    Commands to manage your Azure Redis Cache(s)
	help:
	help:    Create a Redis Cache
	help:      rediscache create [--name <name> --resource-group <resource-group> --location <location> [options]]
	help:
	help:    Delete an existing Redis Cache
	help:      rediscache delete [--name <name> --resource-group <resource-group> ]
	help:
	help:    List all Redis Caches within your Subscription or Resource Group
	help:      rediscache list [options]
	help:
	help:    Show properties of an existing Redis Cache
	help:      rediscache show [--name <name> --resource-group <resource-group>]
	help:
	help:    Change settings of an existing Redis Cache
	help:      rediscache set [--name <name> --resource-group <resource-group> --redis-configuration <redis-configuration>/--redis-configuration-file <redisConfigurationFile>]
	help:
	help:    Renew the authentication key for an existing Redis Cache
	help:      rediscache renew-key [--name <name> --resource-group <resource-group> ]
	help:
	help:    Lists Primary and Secondary key of an existing Redis Cache
	help:      rediscache list-keys [--name <name> --resource-group <resource-group>]
	help:
	help:    Options:
	help:      -h, --help  output usage information
	help:
	help:    Current Mode: arm (Azure Resource Management)

## 建立 Redis 快取

若要建立 Redis 快取，請使用下列命令：

	azure rediscache create [--name <name> --resource-group <resource-group> --location <location> [options]]

如需有關此命令的詳細資訊，請執行 `azure rediscache create -h` 命令。

	C:\>azure rediscache create -h
	help:    Create a Redis Cache
	help:
	help:    Usage: rediscache create [--name <name> --resource-group <resource-group> --location <location> [options]]
	help:
	help:    Options:
	help:      -h, --help                                               output usage information
	help:      -v, --verbose                                            use verbose output
	help:      -vv                                                      more verbose with debug output
	help:      --json                                                   use json output
	help:      -n, --name <name>                                        Name of the Redis Cache.
	help:      -g, --resource-group <resource-group>                    Name of the Resource Group
	help:      -l, --location <location>                                Location to create cache.
	help:      -z, --size <size>                                        Size of the Redis Cache. Valid values: [C0, C1, C2, C3, C4, C5, C6, P1, P2, P3, P4]
	help:      -x, --sku <sku>                                          Redis SKU. Should be one of : [Basic, Standard, Premium]
	help:      -e, --enable-non-ssl-port                                EnableNonSslPort property of the Redis Cache. Add this flag if you want to enable the Non SSL Port for your cache
	help:      -c, --redis-configuration <redis-configuration>          Redis Configuration. Enter a JSON formatted string of configuration keys and values here. Format:"{"<key1>":"<value1>","<key2>":"<value2>"}"
	help:      -f, --redis-configuration-file <redisConfigurationFile>  Redis Configuration. Enter the path of a file containing configuration keys and values here. Format for the file entry: {"<key1>":"<value1>","<key2>":"<value2>"}
	help:      -r, --shard-count <shard-count>                          Number of Shards to create on a Premium Cluster Cache
	help:      -v, --virtual-network <virtual-network>                  The exact ARM resource ID of the virtual network to deploy the redis cache in. Example format: /subscriptions/{subid}/resourceGroups/{resourceGroupName}/Microsoft.ClassicNetwork/VirtualNetworks/vnet1
	help:      -t, --subnet <subnet>                                    Required when deploying a redis cache inside an existing Azure Virtual Network
	help:      -p, --static-ip <static-ip>                              Required when deploying a redis cache inside an existing Azure Virtual Network
	help:      -s, --subscription <id>                                  the subscription identifier
	help:
	help:    Current Mode: arm (Azure Resource Management)

## 刪除現有的 Redis 快取

若要刪除 Redis 快取，請使用下列命令：

	azure rediscache delete [--name <name> --resource-group <resource-group> ]

如需有關此命令的詳細資訊，請執行 `azure rediscache delete -h` 命令。

	C:\>azure rediscache delete -h
	help:    Delete an existing Redis Cache
	help:
	help:    Usage: rediscache delete [--name <name> --resource-group <resource-group> ]
	help:
	help:    Options:
	help:      -h, --help                             output usage information
	help:      -v, --verbose                          use verbose output
	help:      -vv                                    more verbose with debug output
	help:      --json                                 use json output
	help:      -n, --name <name>                      Name of the Redis Cache.
	help:      -g, --resource-group <resource-group>  Name of the Resource Group under which the cache exists
	help:      -s, --subscription <subscription>      the subscription identifier
	help:
	help:    Current Mode: arm (Azure Resource Management)

## 列出您的訂用帳戶或資源群組中所有的 Redis 快取

若要列出您的訂用帳戶或資源群組中所有的 Redis 快取，請使用下列命令：

	azure rediscache list [options]

如需有關此命令的詳細資訊，請執行 `azure rediscache list -h` 命令。

	C:\>azure rediscache list -h
	help:    List all Redis Caches within your Subscription or Resource Group
	help:
	help:    Usage: rediscache list [options]
	help:
	help:    Options:
	help:      -h, --help                             output usage information
	help:      -v, --verbose                          use verbose output
	help:      -vv                                    more verbose with debug output
	help:      --json                                 use json output
	help:      -g, --resource-group <resource-group>  Name of the Resource Group
	help:      -s, --subscription <subscription>      the subscription identifier
	help:
	help:    Current Mode: arm (Azure Resource Management)

## 顯示現有 Redis 快取的屬性

若要顯示現有的 Redis 快取屬性，請使用下列命令：

	azure rediscache show [--name <name> --resource-group <resource-group>]

如需有關此命令的詳細資訊，請執行 `azure rediscache show -h` 命令。

	C:\>azure rediscache show -h
	help:    Show properties of an existing Redis Cache
	help:
	help:    Usage: rediscache show [--name <name> --resource-group <resource-group>]
	help:
	help:    Options:
	help:      -h, --help                             output usage information
	help:      -v, --verbose                          use verbose output
	help:      -vv                                    more verbose with debug output
	help:      --json                                 use json output
	help:      -n, --name <name>                      Name of the Redis Cache.
	help:      -g, --resource-group <resource-group>  Name of the Resource Group
	help:      -s, --subscription <subscription>      the subscription identifier
	help:
	help:    Current Mode: arm (Azure Resource Management)

<a name="scale"></a>
## 變更現有 Redis 快取的設定

若要變更現有的 Redis 快取設定，請使用下列命令：

	azure rediscache set [--name <name> --resource-group <resource-group> --redis-configuration <redis-configuration>/--redis-configuration-file <redisConfigurationFile>]

如需有關此命令的詳細資訊，請執行 `azure rediscache set -h` 命令。

	C:\>azure rediscache set -h
	help:    Change settings of an existing Redis Cache
	help:
	help:    Usage: rediscache set [--name <name> --resource-group <resource-group> --redis-configuration <redis-configuration>/--redis-configuration-file <redisConfigurationFile>]
	help:
	help:    Options:
	help:      -h, --help                                               output usage information
	help:      -v, --verbose                                            use verbose output
	help:      -vv                                                      more verbose with debug output
	help:      --json                                                   use json output
	help:      -n, --name <name>                                        Name of the Redis Cache.
	help:      -g, --resource-group <resource-group>                    Name of the Resource Group
	help:      -c, --redis-configuration <redis-configuration>          Redis Configuration. Enter a JSON formatted string of configuration keys and values here.
	help:      -f, --redis-configuration-file <redisConfigurationFile>  Redis Configuration. Enter the path of a file containing configuration keys and values here.
	help:      -s, --subscription <subscription>                        the subscription identifier
	help:
	help:    Current Mode: arm (Azure Resource Management)

## 更新現有的 Redis 快取驗證金鑰

若要更新現有 Redis 快取的驗證金鑰，請使用下列命令：

	azure rediscache renew-key [--name <name> --resource-group <resource-group> --key-type <key-type>]

針對 `key-type` 指定 `Primary` 或 `Secondary`。

如需有關此命令的詳細資訊，請執行 `azure rediscache renew-key -h` 命令。

	C:\>azure rediscache renew-key -h
	help:    Renew the authentication key for an existing Redis Cache
	help:
	help:    Usage: rediscache renew-key [--name <name> --resource-group <resource-group> ]
	help:
	help:    Options:
	help:      -h, --help                             output usage information
	help:      -v, --verbose                          use verbose output
	help:      -vv                                    more verbose with debug output
	help:      --json                                 use json output
	help:      -n, --name <name>                      Name of the Redis Cache.
	help:      -g, --resource-group <resource-group>  Name of the Resource Group under which cache exists
	help:      -t, --key-type <key-type>              type of key to renew. Valid values are: 'Primary', 'Secondary'.
	help:      -s, --subscription <subscription>      the subscription identifier
	help:
	help:    Current Mode: arm (Azure Resource Management)

## 列出現有之 Redis 快取的主要和次要金鑰

若要列出現有之 Redis 快取的主要和次要金鑰，請使用下列命令：

	azure rediscache list-keys [--name <name> --resource-group <resource-group>]

如需有關此命令的詳細資訊，請執行 `azure rediscache list-keys -h` 命令。

	C:\>azure rediscache list-keys -h
	help:    Lists Primary and Secondary key of an existing Redis Cache
	help:
	help:    Usage: rediscache list-keys [--name <name> --resource-group <resource-group>]
	help:
	help:    Options:
	help:      -h, --help                             output usage information
	help:      -v, --verbose                          use verbose output
	help:      -vv                                    more verbose with debug output
	help:      --json                                 use json output
	help:      -n, --name <name>                      Name of the Redis Cache.
	help:      -g, --resource-group <resource-group>  Name of the Resource Group under which Cache exists
	help:      -s, --subscription <subscription>      the subscription identifier
	help:
	help:    Current Mode: arm (Azure Resource Management)

<!---HONumber=AcomDC_0921_2016-->