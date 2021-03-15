---
ms.custom: devx-track-js
ms.topic: include
ms.date: 02/18/2021
ms.openlocfilehash: 079ffe7f07b6038a1f6e320ca37eed3aafb41ec2
ms.sourcegitcommit: 576c878c338d286060010646b96f3ad0fdbcb814
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102118780"
---
## <a name="create-an-azure-cache-for-redis-resource-with-azure-cli"></a>Azure CLI を使用して Azure Cache for Redis リソースを作成する

[Azure Cloud Shell](https://shell.azure.com) で次の Azure CLI [az redis create](/cli/azure/redis#az_redis_create) コマンドを使用して、お使いのデータベース用の新しい MariaDB リソースを作成します。 

```azurecli
az redis create \
    --subscription YOUR-SUBSCRIPTION-ID-OR-NAME \
    --resource-group YOUR-RESOURCE-GROUP \
    --name YOUR-RESOURCE-NAME \
    --location eastus \
    --sku Basic \
    --vm-size c0 \
    --enable-non-ssl-port
```

このコマンドは、完了するまでに数分かかる場合があり、`eastus` リージョンで一般公開されるリソースが作成されます。 

応答には、次のようなサーバーの構成の詳細が含まれます。 
* Redis のバージョン: `redisVersion`
* ポート: `sslPort` および `port`

```text
{
  "accessKeys": null,
  "enableNonSslPort": true,
  "hostName": "YOUR-RESOURCE-NAME.redis.cache.windows.net",
  "id": "/subscriptions/YOUR-SUBSCRIPTION-ID-OR-NAME/resourceGroups/YOUR-RESOURCE-GROUP/providers/Microsoft.Cache/Redis/YOUR-RESOURCE-NAME",
  "instances": [
    {
      "isMaster": false,
      "nonSslPort": 13000,
      "shardId": null,
      "sslPort": 15000,
      "zone": null
    }
  ],
  "linkedServers": [],
  "location": "East US",
  "minimumTlsVersion": null,
  "name": "YOUR-RESOURCE-NAME",
  "port": 6379,
  "provisioningState": "Creating",
  "redisConfiguration": {
    "maxclients": "256",
    "maxfragmentationmemory-reserved": "12",
    "maxmemory-delta": "2",
    "maxmemory-reserved": "2"
  },
  "redisVersion": "4.0.14",
  "replicasPerMaster": null,
  "resourceGroup": "YOUR-RESOURCE-GROUP",
  "shardCount": null,
  "sku": {
    "capacity": 0,
    "family": "C",
    "name": "Basic"
  },
  "sslPort": 6380,
  "staticIp": null,
  "subnetId": null,
  "tags": {},
  "tenantSettings": {},
  "type": "Microsoft.Cache/Redis",
  "zones": null
}
```

## <a name="add-firewall-rule-for-your-client-ip-address-to-redis-resource"></a>クライアント IP アドレスのファイアウォール規則を Redis リソースに追加する

[az redis firewall-rules create](/cli/azure/redis/firewall-rules#az_redis_firewall_rules_create) コマンドを使用してファイアウォール規則を追加し、お使いのクライアント IP またはアプリの IP からキャッシュへのアクセスを定義します。

```azurecli
az redis firewall-rules create \
    --subscription YOUR-SUBSCRIPTION-ID-OR-NAME \
    --resource-group YOUR-RESOURCE-GROUP \
    --name YOUR-RESOURCE-NAME \
    --rule-name AllowMyIP \
    --start-ip 123.123.123.123 \
    --end-ip 123.123.123.123
```

お使いのクライアント IP アドレスがわからない場合は、次のいずれかの方法を使用します。
* Azure portal を使用してファイアウォール規則を表示し、検出されたクライアント IP の追加を含む変更を行う。
* Node.js コードを実行すると、クライアント IP アドレスを含む、ファイアウォール規則違反に関するエラーが表示されます。 この IP アドレスをコピーし、それを使用してファイアウォール規則を設定します。

## <a name="get-the-redis-keys-with-azure-cli"></a>Azure CLI で Redis キーを取得する

[az redis list-keys](/cli/azure/redis#az_redis_list_keys) コマンドを使用して、このインスタンスの Redis 接続文字列を取得します。

```azurecli
az redis list-keys \
    --subscription YOUR-SUBSCRIPTION-ID-OR-NAME \
    --resource-group YOUR-RESOURCE-GROUP \
    --name YOUR-RESOURCE-NAME
```

次の 2 つのキーが返されます。

```json
{
    "primaryKey": "5Uey0GHWtCs9yr1FMUMcu1Sv17AJA2QMqPm9CyNKjAA=",
    "secondaryKey": "DEPr+3zWbL6d5XwxPajAriXKgoSeCqraN8SLSoiMWhM="
  }
```

## <a name="connect-azure-cache-for-redis-to-your-app-service-web-app"></a>Azure Cache for Redis を App Service Web アプリに接続する

[az webapp config appsettings set](/cli/azure/webapp/config/appsettings#az_webapp_config_appsettings_set) コマンドを使用して、App Service Web アプリへの接続情報を追加します。

```azurecli
az webapp config appsettings set \
--subscription YOUR-SUBSCRIPTION-ID-OR-NAME \
--resource-group YOUR-RESOURCE-GROUP \
--name YOUR-APP-SERVICE-RESOURCE-NAME \
--settings "REDIS_URL=YOUR-REDIS-HOST-NAME" "REDIS_PORT=YOUR-REDIS-PORT" "REDIS_KEY=YOUR-REDIS-KEY"
```

前のコードの次の設定を置き換えます。

* YOUR-SUBSCRIPTION-ID-OR-NAME
* YOUR-RESOURCE-GROUP
* YOUR-APP-SERVICE-RESOURCE-NAME
* YOUR-REDIS-HOST-NAME
* YOUR-REDIS-PORT
* YOUR-REDIS-KEY
