---
ms.custom: devx-track-js
ms.topic: include
ms.date: 02/04/2021
ms.openlocfilehash: 1fe0e6ef7147040d1a5496f6492cc1e63e2e425e
ms.sourcegitcommit: 98a7e855206ff463c1d95f93c23dd665b26a0aa1
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/09/2021
ms.locfileid: "100004990"
---
## <a name="create-an-azure-database-for-postgresql-server-resource-with-azure-cli"></a>Azure CLI を使用して Azure Database for PostgreSQL サーバー リソースを作成する

[Azure Cloud Shell](https://shell.azure.com) で次の Azure CLI [az postgres server create](/cli/azure/postgres/server#az_postgres_server_create) コマンドを使用して、お使いのデータベースに新しい PostgreSQL サーバー リソースを作成します。 

```azurecli
az postgres server create \
    --subscription YOUR-SUBSCRIPTION-ID-OR-NAME \
    --resource-group YOUR-RESOURCE-GROUP \
    --location eastus \
    --name YOURRESOURCENAME \
    --admin-user YOUR-ADMIN-USER \
    --ssl-enforcement Disabled \
    --enable-public-network Enabled 
```

このコマンドは、完了するまでに数分かかる場合があり、`eastus` リージョンで一般公開されるリソースが作成されます。 

応答には、次のようなサーバーの構成の詳細が含まれます。 
* 管理者アカウント用の自動生成されたパスワード
* 接続文字列

```text
{
  "additionalProperties": {},
  "administratorLogin": "YOUR-ADMIN-USER",
  "byokEnforcement": "Disabled",
  "connectionString": "postgres://YOUR-ADMIN-USER%40YOURRESOURCENAME:123456789@YOURRESOURCENAME.postgres.database.azure.com/postgres?sslmode=require",
  "earliestRestoreDate": "2021-02-08T21:56:20.130000+00:00",
  "fullyQualifiedDomainName": "YOURRESOURCENAME.postgres.database.azure.com",
  "id": "/subscriptions/.../resourceGroups/YOUR-RESOURCE-GROUP/providers/Microsoft.DBforPostgreSQL/servers/YOURRESOURCENAME",
  "identity": null,
  "infrastructureEncryption": "Disabled",
  "location": "eastus",
  "masterServerId": "",
  "minimalTlsVersion": "TLSEnforcementDisabled",
  "name": "YOURRESOURCENAME",
  "password": "123456789",
  "privateEndpointConnections": [],
  "publicNetworkAccess": "Enabled",
  "replicaCapacity": 5,
  "replicationRole": "None",
  "resourceGroup": "YOUR-RESOURCE-GROUP",
  "sku": {
    "additionalProperties": {},
    "capacity": 2,
    "family": "Gen5",
    "name": "GP_Gen5_2",
    "size": null,
    "tier": "GeneralPurpose"
  },
  "sslEnforcement": "Disabled",
  "storageProfile": {
    "additionalProperties": {},
    "backupRetentionDays": 7,
    "geoRedundantBackup": "Disabled",
    "storageAutogrow": "Enabled",
    "storageMb": 51200
  },
  "tags": null,
  "type": "Microsoft.DBforPostgreSQL/servers",
  "userVisibleState": "Ready",
  "version": "9.6"
}
```

サーバーに接続する前に、お使いのクライアント IP アドレスの通過を許可するようにファイアウォール規則を構成する必要があります。 

## <a name="add-a-firewall-rule-for-your-client-ip-address-to-postgresql-resource"></a>クライアント IP アドレスのファイアウォール規則を PostgreSQL リソースに追加する

既定では、ファイアウォール規則は構成されていません。 JavaScript を使用したサーバーへのクライアント接続が成功するように、お使いのクライアント IP アドレスを追加する必要があります。

[Azure Cloud Shell](https://shell.azure.com) で次の Azure CLI [az postgres server firewall-rule create](/cli/azure/postgres/server#az_postgres_server_firewall_rule_create) コマンドを使用して、お使いのデータベースに新しいファイアウォール規則を作成します。 


```azurecli
az postgres server firewall-rule create \
    --subscription YOUR-SUBSCRIPTION-ID-OR-NAME \
    --resource-group YOUR-RESOURCE-GROUP \
    --server YOURRESOURCENAME \
    --name AllowMyIP \
    --start-ip-address 123.123.123.123 \
    --end-ip-address 123.123.123.123
```

お使いのクライアント IP アドレスがわからない場合は、次のいずれかの方法を使用します。
* Azure portal を使用してファイアウォール規則を表示し、検出されたクライアント IP の追加を含む変更を行う
* Node.js コードを実行する。ファイアウォール規則違反に関するエラーにクライアント IP アドレスが含まれます

すべてのインターネット アドレスをファイアウォール規則 (0.0.0.0-255.255.255.255) として追加できますが、これによってお使いのサーバーは攻撃に対して無防備になります。 

## <a name="create-a-postgresql-database-on-the-server-with-azure-cli"></a>Azure CLI を使用してサーバーにPostgreSQL データベースを作成する

[Azure Cloud Shell](https://shell.azure.com) で次の Azure CLI [az postgres db create](/cli/azure/postgres/db#az_postgres_db_create) コマンドを使用して、お使いのサーバーに新しい PostgreSQL データベースを作成します。 

```azurecli
az postgres db create \
    --subscription YOUR-SUBSCRIPTION-ID-OR-NAME \
    --resource-group YOUR-RESOURCE-GROUP \
    --server-name YOURRESOURCENAME \
    --name YOURDATABASENAME
```

## <a name="get-the-postgresql-connection-string-with-azure-cli"></a>Azure CLI を使用して PostgreSQL 接続文字列を取得する

[az postgres server show-connection-string](/cli/azure/postgres/server#az_postgres_server_show_connection_string) コマンドを使用して、このインスタンスの PostgreSQL 接続文字列を取得します。

```azurecli
az postgres server show-connection-string \
    --subscription YOUR-SUBSCRIPTION-ID-OR-NAME \
    --server-name YOURRESOURCENAME
```

これにより、一般的な言語用の接続文字列が JSON オブジェクトとして返されます。 接続文字列を使用する前に、`{database}`、`{username}`、`{password}` を独自の値に置き換える必要があります。 `YOURRESOURCENAME` をお使いのリソース名に置き換えます。

```json
{
  "connectionStrings": {
    "C++ (libpq)": "host=YOURRESOURCENAME.postgres.database.azure.com port=5432 dbname={database} user={username}YOURRESOURCENAME password={password} sslmode=require",
    "ado.net": "Server=YOURRESOURCENAME.postgres.database.azure.com;Database={database};Port=5432;User Id={username}@YOURRESOURCENAME;Password={password};",
    "jdbc": "jdbc:postgresql://YOURRESOURCENAME.postgres.database.azure.com:5432/{database}?user={username}@YOURRESOURCENAME&password={password}",
    "node.js": "var client = new pg.Client('postgres://{username}@YOURRESOURCENAME:{password}@YOURRESOURCENAME.postgres.database.azure.com:5432/{database}');",
    "php": "host=YOURRESOURCENAME.postgres.database.azure.com port=5432 dbname={database} user={username}@YOURRESOURCENAME password={password}",
    "psql_cmd": "postgresql://{username}@YOURRESOURCENAME:{password}@YOURRESOURCENAME.postgres.database.azure.com/{database}?sslmode=require",
    "python": "cnx = psycopg2.connect(database='{database}', user='{username}@YOURRESOURCENAME', host='YOURRESOURCENAME.postgres.database.azure.com', password='{password}', port='5432')",
    "ruby": "cnx = PG::Connection.new(:host => 'YOURRESOURCENAME.postgres.database.azure.com', :user => '{username}@YOURRESOURCENAME', :dbname => '{database}', :port => '5432', :password => '{password}')"
  }
}
``` 