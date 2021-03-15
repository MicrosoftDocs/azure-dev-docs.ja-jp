---
ms.custom: devx-track-js
ms.topic: include
ms.date: 02/17/2021
ms.openlocfilehash: e37f278149f8173c3addddda085f37112929bf33
ms.sourcegitcommit: b0a119a624e9cb6b76d968951543a414bd08eaa0
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/09/2021
ms.locfileid: "102118772"
---
## <a name="create-a-cosmos-db-resource-for-cassandra-db"></a>Cassandra DB 用の Cosmos DB リソースを作成する

[Azure Cloud Shell](https://shell.azure.com) で次の Azure CLI [az cosmosdb create](/cli/azure/cosmosdb#az_cosmosdb_create) コマンドを使用して、Cassandra データベース用の新しいリソースを作成します。 

```azurecli
az cosmosdb create \
    --subscription YOUR-SUBSCRIPTION-ID-OR-NAME \
    --resource-group YOUR-RESOURCE-GROUP \
    --name YOUR-RESOURCE_NAME \
    --capabilities EnableCassandra
```

このコマンドは、完了するまでに数分かかる場合があり、一般公開されるリソースが作成されます。 お使いのクライアント IP アドレスの通過を許可するようにファイアウォール規則を構成する必要はありません。 

応答には、次のようなサーバーの構成の詳細が含まれます。 

* `location`: cassandra-drive を含む JavaScript コードの `localDataCenter` 値に使用されます 

```json
{
  "apiProperties": null,
  "capabilities": [
    {
      "name": "EnableCassandra"
    }
  ],
  "connectorOffer": null,
  "consistencyPolicy": {
    "defaultConsistencyLevel": "Session",
    "maxIntervalInSeconds": 5,
    "maxStalenessPrefix": 100
  },
  "cors": [],
  "databaseAccountOfferType": "Standard",
  "disableKeyBasedMetadataWriteAccess": false,
  "documentEndpoint": "https://YOUR-RESOURCE_NAME.documents.azure.com:443/",
  "enableAnalyticalStorage": false,
  "enableAutomaticFailover": false,
  "enableCassandraConnector": null,
  "enableFreeTier": false,
  "enableMultipleWriteLocations": false,
  "failoverPolicies": [
    {
      "failoverPriority": 0,
      "id": "YOUR-RESOURCE_NAME-centralus",
      "locationName": "Central US"
    }
  ],
  "id": "/subscriptions/YOUR-SUBSCRIPTION-ID-OR-NAME/resourceGroups/YOUR-RESOURCE-GROUP/providers/Microsoft.DocumentDB/databaseAccounts/YOUR-RESOURCE_NAME",
  "ipRules": [],
  "isVirtualNetworkFilterEnabled": false,
  "keyVaultKeyUri": null,
  "kind": "GlobalDocumentDB",
  "location": "Central US",
  "locations": [
    {
      "documentEndpoint": "https://YOUR-RESOURCE_NAME-centralus.documents.azure.com:443/",
      "failoverPriority": 0,
      "id": "YOUR-RESOURCE_NAME-centralus",
      "isZoneRedundant": false,
      "locationName": "Central US",
      "provisioningState": "Succeeded"
    }
  ],
  "name": "YOUR-RESOURCE_NAME",
  "privateEndpointConnections": null,
  "provisioningState": "Succeeded",
  "publicNetworkAccess": "Enabled",
  "readLocations": [
    {
      "documentEndpoint": "https://YOUR-RESOURCE_NAME-centralus.documents.azure.com:443/",
      "failoverPriority": 0,
      "id": "YOUR-RESOURCE_NAME-centralus",
      "isZoneRedundant": false,
      "locationName": "Central US",
      "provisioningState": "Succeeded"
    }
  ],
  "resourceGroup": "YOUR-RESOURCE-GROUP",
  "systemData": {
    "createdAt": "2021-02-16T23:00:23.0375775Z"
  },
  "tags": {},
  "type": "Microsoft.DocumentDB/databaseAccounts",
  "virtualNetworkRules": [],
  "writeLocations": [
    {
      "documentEndpoint": "https://YOUR-RESOURCE_NAME-centralus.documents.azure.com:443/",
      "failoverPriority": 0,
      "id": "YOUR-RESOURCE_NAME-centralus",
      "isZoneRedundant": false,
      "locationName": "Central US",
      "provisioningState": "Succeeded"
    }
  ]
}
```


## <a name="create-a-keyspace-on-the-server-with-azure-cli"></a>Azure CLI を使用してサーバーにキースペースを作成する

[Azure Cloud Shell](https://shell.azure.com) で次の Azure CLI [az cosmosdb cassandra keyspace create](/cli/azure/cosmosdb/cassandra/keyspace#az_cosmosdb_cassandra_keyspace_create) コマンドを使用して、サーバーに新しい Cassandra キースペースを作成します。 

```azurecli
az cosmosdb cassandra keyspace create \
    --subscription YOUR-SUBSCRIPTION-ID-OR-NAME \
    --resource-group YOUR-RESOURCE-GROUP \
    --account-name YOUR-RESOURCE_NAME \
    --name YOUR-KEYSPACE-NAME
```

## <a name="create-a-table-on-the-keyspace-with-azure-cli"></a>Azure CLI を使用してキースペースにテーブルを作成する

[Azure Cloud Shell](https://shell.azure.com) で次の Azure CLI [az cosmosdb cassandra table create](/cli/azure/cosmosdb/cassandra/table#az_cosmosdb_cassandra_table_create) コマンドを使用して、サーバーに新しい Cassandra キースペースを作成します。 

```azurecli
az cosmosdb cassandra table create \
    --subscription YOUR-SUBSCRIPTION-ID-OR-NAME \
    --resource-group YOUR-RESOURCE-GROUP \
    --account-name YOUR-RESOURCE_NAME \
    --keyspace-name YOUR-KEYSPACE-NAME \
    --name YOUR-TABLE-NAME \
    --schema @schema.json
```

スキーマ ファイルの JSON では、テーブルの列、データ型、パーティション キーが定義されています。

```json
{
    "columns": [
        {
            "name": "Name",
            "type": "Ascii"
        },
        {
            "name": "Alias",
            "type": "Ascii"
        },
        {
            "name": "Region",
            "type": "Ascii"
        }        
    ],
    "partitionKeys": [
        {
            "name": "Region"
        }
    ]
}
```

## <a name="get-the-cassandra-connection-string-with-azure-cli"></a>Azure CLI を使用して Cassandra 接続文字列を取得する
[az cosmosdb keys list](/cli/azure/cosmosdb/keys#az_cosmosdb_keys_list) コマンドを使用して、このインスタンスの MongoDB 接続文字列を取得します。

```azurecli
az cosmosdb keys list \
    --subscription YOUR-SUBSCRIPTION-ID-OR-NAME \
    --resource-group YOUR-RESOURCE-GROUP \
    --name YOUR-RESOURCE-NAME \
    --type connection-strings 
```

これにより、接続文字列が返されます。 次の JSON は、セキュリティ情報が次のように置き換えられたサンプルの結果です。

* `YOUR-RESOURCE-NAME`: UserName とホスト名またはアカウント エンドポイントの一部に使用されます
* `YOUR-USERNAME`: `YOUR-RESOURCE-NAME` と同じ値
* `PASSWORD-1`
* `PASSWORD-2`
* `ACCOUNT-KEY-1`
* `ACCOUNT-KEY-2`

```json
{
    "connectionStrings": [
      {
        "connectionString": "AccountEndpoint=https://YOUR-RESOURCE-NAME.documents.azure.com:443/;AccountKey=PASSWORD-1;",
        "description": "Primary SQL Connection String"
      },
      {
        "connectionString": "AccountEndpoint=https://YOUR-RESOURCE-NAME.documents.azure.com:443/;AccountKey=PASSWORD-2;",
        "description": "Secondary SQL Connection String"
      },
      {
        "connectionString": "AccountEndpoint=https://YOUR-RESOURCE-NAME.documents.azure.com:443/;AccountKey=ACCOUNT-KEY-1;",
        "description": "Primary Read-Only SQL Connection String"
      },
      {
        "connectionString": "AccountEndpoint=https://YOUR-RESOURCE-NAME.documents.azure.com:443/;AccountKey=ACCOUNT-KEY-2;",
        "description": "Secondary Read-Only SQL Connection String"
      },
      {
        "connectionString": "HostName=YOUR-RESOURCE-NAME.cassandra.cosmos.azure.com;Username=YOUR-RESOURCE-NAME;Password=PASSWORD-1;Port=10350",
        "description": "Primary Cassandra Connection String"
      },
      {
        "connectionString": "HostName=YOUR-RESOURCE-NAME.cassandra.cosmos.azure.com;Username=YOUR-RESOURCE-NAME;Password=PASSWORD-2;Port=10350",
        "description": "Secondary Cassandra Connection String"
      },
      {
        "connectionString": "HostName=YOUR-RESOURCE-NAME.cassandra.cosmos.azure.com;Username=YOUR-RESOURCE-NAME;Password=ACCOUNT-KEY-1;Port=10350",
        "description": "Primary Read-Only Cassandra Connection String"
      },
      {
        "connectionString": "HostName=YOUR-RESOURCE-NAME.cassandra.cosmos.azure.com;Username=YOUR-RESOURCE-NAME;Password=ACCOUNT-KEY-2;Port=10350",
        "description": "Secondary Read-Only Cassandra Connection String"
      }
    ]
  }
```

接続文字列を使用して Cassandra データベースに接続します。 Cassandra のユーザー名はリソース名です。 