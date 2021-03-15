---
ms.custom: devx-track-js
ms.topic: include
ms.date: 02/08/2021
ms.openlocfilehash: e820cb17038a5251e658c9b7286cec65ddc40fca
ms.sourcegitcommit: b0a119a624e9cb6b76d968951543a414bd08eaa0
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/09/2021
ms.locfileid: "102118270"
---
## <a name="create-a-cosmos-db-resource-for-mongodb"></a>MongoDB の Cosmos DB リソースを作成する

[Azure Cloud Shell](https://shell.azure.com) で次の Azure CLI [az cosmosdb create](/cli/azure/cosmosdb#az_cosmosdb_create) コマンドを使用して、mongoDB データベース用の新しい Cosmos DB リソースを作成します。 

```azurecli
az cosmosdb create \
    --subscription YOUR-SUBSCRIPTION-ID-OR-NAME \
    --resource-group YOUR-RESOURCE-GROUP \
    --name YOUR-RESOURCE_NAME \
    --locations regionName=eastus \
    --kind MongoDB \
    --enable-public-network true \
    --ip-range-filter 123.123.123.123 
```

`123.123.123.123` を独自のクライアント IP に置き換えるか、パラメーター全体を削除します。 

このコマンドは、完了するまでに数分かかる場合があり、`eastus` リージョンで一般公開されるリソースが作成されます。 

```text
{
  "apiProperties": {
    "serverVersion": "3.6"
  },
  "capabilities": [
    {
      "name": "EnableMongo"
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
  "documentEndpoint": "https://mongo-2.documents.azure.com:443/",
  "enableAnalyticalStorage": false,
  "enableAutomaticFailover": false,
  "enableCassandraConnector": null,
  "enableFreeTier": false,
  "enableMultipleWriteLocations": false,
  "failoverPolicies": [
    {
      "failoverPriority": 0,
      "id": "mongodb-2",
      "locationName": "East US"
    }
  ],
  "id": "/subscriptions/.../resourceGroups/my-resource-group/providers/Microsoft.DocumentDB/databaseAccounts/mongo-2",
  "ipRules": [
    {
      "ipAddressOrRange": "123.123.123.123"
    }
  ],
  "isVirtualNetworkFilterEnabled": false,
  "keyVaultKeyUri": null,
  "kind": "MongoDB",
  "location": "Central US",
  "locations": [
    {
      "documentEndpoint": "https://mongodb-2.documents.azure.com:443/",
      "failoverPriority": 0,
      "id": "mongodb-2",
      "isZoneRedundant": false,
      "locationName": "East US",
      "provisioningState": "Succeeded"
    }
  ],
  "name": "mongo-2",
  "privateEndpointConnections": null,
  "provisioningState": "Succeeded",
  "publicNetworkAccess": "Enabled",
  "readLocations": [
    {
      "documentEndpoint": "https://mongodb-2.documents.azure.com:443/",
      "failoverPriority": 0,
      "id": "mongodb-2",
      "isZoneRedundant": false,
      "locationName": "East US",
      "provisioningState": "Succeeded"
    }
  ],
  "resourceGroup": "my-resource-group",
  "systemData": {
    "createdAt": "2021-02-08T20:21:05.9519342Z"
  },
  "tags": {},
  "type": "Microsoft.DocumentDB/databaseAccounts",
  "virtualNetworkRules": [],
  "writeLocations": [
    {
      "documentEndpoint": "https://mongodb-2.documents.azure.com:443/",
      "failoverPriority": 0,
      "id": "mongodb-2",
      "isZoneRedundant": false,
      "locationName": "East US",
      "provisioningState": "Succeeded"
    }
  ]
}
```

## <a name="add-firewall-rule-for-your-client-ip-address"></a>クライアント IP アドレスのファイアウォール規則を追加する

既定では、ファイアウォール規則は構成されていません。 JavaScript を使用したサーバーへのクライアント接続が成功するように、お使いのクライアント IP アドレスを追加する必要があります。

ファイアウォール規則を更新するには、[az cosmosdb update](/cli/azure/cosmosdb#az_cosmosdb_update) コマンドを使用します。

```azurecli
az cosmosdb update \
    --subscription YOUR-SUBSCRIPTION-ID-OR-NAME \
    --resource-group YOUR-RESOURCE-GROUP \
    --name YOUR-RESOURCE_NAME \
    --ip-range-filter 123.123.123.123
```

複数の IP アドレスを構成するには、コンマ区切りのリストを使用します。

```azurecli
az cosmosdb update \
    --subscription YOUR-SUBSCRIPTION-ID-OR-NAME \
    --resource-group YOUR-RESOURCE-GROUP \
    --name YOUR-RESOURCE_NAME \
    --ip-range-filter 123.123.123.123,456.456.456.456
```

## <a name="get-the-mongodb-connection-string-for-your-resource"></a>リソース用の MongoDB 接続文字列を取得する

[az cosmosdb keys list](/cli/azure/cosmosdb/keys#az_cosmosdb_keys_list) コマンドを使用して、このインスタンスの MongoDB 接続文字列を取得します。

```azurecli
az cosmosdb keys list \
    --subscription YOUR-SUBSCRIPTION-ID-OR-NAME \
    --resource-group YOUR-RESOURCE-GROUP \
    --name YOUR-RESOURCE-NAME \
    --type connection-strings 
```

これにより、4 つの接続文字列 (2 つの読み取り/書き込みと 2 つの読み取り専用) が返されます。 2 つずつあるため、2 つの異なるシステムまたは開発者に対して、個別に使用する接続文字列を提供できます。 

接続文字列を使用して mongoDB データベースに接続します。 次のいずれかによって、サービスが使用可能であることを確認します。

* 一般公開されている
* クライアント IP アドレスに対するファイアウォール設定

## <a name="configure-your-azure-web-app-with-the-connection-string"></a>接続文字列を使用して Azure Web アプリを構成する

[az webapp config appsettings set](/cli/azure/webapp/config/appsettings#az_webapp_config_appsettings_set) を使用して Azure Web アプリに **MONGODB_URL** 環境変数を追加し、Web アプリが Cosmos DB リソースに接続できるようにします。

```azurecli
az webapp config appsettings set \
    --subscription YOUR-SUBSCRIPTION-ID-OR-NAME \
    --resource-group YOUR-RESOURCE-GROUP \
    --name YOUR-RESOURCE_NAME \
    --settings MONGODB_URL=YOUR-CONNECTION-STRING
```
