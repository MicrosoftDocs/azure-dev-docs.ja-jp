---
ms.custom: devx-track-js
ms.topic: include
ms.date: 02/04/2021
ms.openlocfilehash: 0edd926e2acf016eb6389fe58c7f1d5e78628dbf
ms.sourcegitcommit: 98a7e855206ff463c1d95f93c23dd665b26a0aa1
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/09/2021
ms.locfileid: "100004723"
---
## <a name="create-an-azure-database-for-mysql-resource-with-azure-cli"></a>Azure CLI を使用して Azure Database for MySQL リソースを作成する

[Azure Cloud Shell](https://shell.azure.com) で次の Azure CLI [az mysql server create](/cli/azure/mysql/server#az_mysql_server_create) コマンドを使用して、お使いのデータベースに新しい MySQL リソースを作成します。 

```azurecli
az mysql server create \
    --subscription YOUR-SUBSCRIPTION-ID-OR-NAME \
    --resource-group YOUR-RESOURCE-GROUP \
    --name YOURRESOURCENAME \
    --location eastus \
    --admin-user YOUR-ADMIN-NAME \
    --ssl-enforcement Disabled \
    --enable-public-network Enabled 
```

このコマンドは、完了するまでに数分かかる場合があり、`eastus` リージョンで一般公開されるリソースが作成されます。 

応答には、次のようなサーバーの構成の詳細が含まれます。 
* 管理者アカウント用の自動生成されたパスワード
* MySQL CLI を使用してサーバーに接続するためのコマンド

```text
{
  "additionalProperties": {},
  "administratorLogin": "mySqlAdmin",
  "byokEnforcement": "Disabled",
  "connectionString": "mysql defaultdb --host mysqldb-2.mysql.database.azure.com --user mySqlAdmin@mysqldb-2 --password=123456789",
  "databaseName": "defaultdb",
  "earliestRestoreDate": "2021-02-08T16:48:01.673000+00:00",
  "fullyQualifiedDomainName": "mysqldb-2.mysql.database.azure.com",
  "id": "/subscriptions/.../resourceGroups/my-resource-group/providers/Microsoft.DBforMySQL/servers/mysqldb-2",
  "identity": null,
  "infrastructureEncryption": "Disabled",
  "location": "westus",
  "masterServerId": "",
  "minimalTlsVersion": "TLSEnforcementDisabled",
  "name": "mysqldb-2",
  "password": "123456789",
  "privateEndpointConnections": [],
  "publicNetworkAccess": "Enabled",
  "replicaCapacity": 5,
  "replicationRole": "None",
  "resourceGroup": "my-resource-group",
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
  "type": "Microsoft.DBforMySQL/servers",
  "userVisibleState": "Ready",
  "version": "5.7"
}
```

プログラムを使用してサーバーに接続する前に、お使いのクライアント IP アドレスの通過を許可するようにファイアウォール規則を構成する必要があります。 

## <a name="add-firewall-rule-for-your-client-ip-address-to-mysql-resource"></a>クライアント IP アドレスのファイアウォール規則を MySQL リソースに追加する

既定では、ファイアウォール規則は構成されていません。 JavaScript を使用したサーバーへのクライアント接続が成功するように、お使いのクライアント IP アドレスを追加する必要があります。

```azurecli
az mysql server firewall-rule create \
--resource-group YOUR-RESOURCE-GROUP \
--server YOURRESOURCENAME \
--name AllowMyIP --start-ip-address 123.123.123.123 \
--end-ip-address 123.123.123.123
```

お使いのクライアント IP アドレスがわからない場合は、次のいずれかの方法を使用します。
* Azure portal を使用してファイアウォール規則を表示し、検出されたクライアント IP の追加を含む変更を行う
* Node.js コードを実行する。ファイアウォール規則違反に関するエラーにクライアント IP アドレスが含まれます

すべてのインターネット アドレスをファイアウォール規則 (0.0.0.0-255.255.255.255) として追加できますが、これによってお使いのサーバーは攻撃に対して無防備になります。 

## <a name="create-a-database-on-the-server-with-azure-cli"></a>Azure CLI を使用してサーバーにデータベースを作成する

[Azure Cloud Shell](https://shell.azure.com) で次の Azure CLI [az mysql db create](/cli/azure/mysql/db#az_mysql_db_create) コマンドを使用して、お使いのサーバーに新しい MySQL データベースを作成します。 

```azurecli
az mysql db create \
    --subscription YOUR-SUBSCRIPTION-ID-OR-NAME \
    --resource-group YOUR-RESOURCE-GROUP \
    --server-name YOURRESOURCENAME \
    --name YOURDATABASENAME
```


## <a name="get-the-mysql-connection-string-with-azure-cli"></a>Azure CLI を使用して MySQL 接続文字列を取得する

[az mysql server show-connection-string](/cli/azure/mysql/server#az_mysql_server_show_connection_string) コマンドを使用して、このインスタンスの MySQL 接続文字列を取得します。

```azurecli
az mysql server show-connection-string \
    --subscription YOUR-SUBSCRIPTION-ID-OR-NAME \
    --server-name YOURRESOURCENAME
```

これにより、一般的な言語用の接続文字列が JSON オブジェクトとして返されます。 接続文字列を使用する前に、`{database}`、`{username}`、`{password}` を独自の値に置き換える必要があります。 

```json
{
  "connectionStrings": {
    "ado.net": "Server=YOURRESOURCENAME.mysql.database.azure.com; Port=3306; Database={database}; Uid={username}@YOURRESOURCENAME; Pwd={password}",
    "jdbc": "jdbc:mysql://YOURRESOURCENAME.mysql.database.azure.com:3306/{database}?user={username}@YOURRESOURCENAME&password={password}",
    "mysql_cmd": "mysql {database} --host YOURRESOURCENAME.mysql.database.azure.com --user {username}@YOURRESOURCENAME --password={password}",
    "node.js": "var conn = mysql.createConnection({host: 'YOURRESOURCENAME.mysql.database.azure.com', user: '{username}@YOURRESOURCENAME',password: {password}, database: {database}, port: 3306});",
    "php": "host=YOURRESOURCENAME.mysql.database.azure.com port=3306 dbname={database} user={username}@YOURRESOURCENAME password={password}",
    "python": "cnx = mysql.connector.connect(user='{username}@YOURRESOURCENAME', password='{password}', host='YOURRESOURCENAME.mysql.database.azure.com', port=3306, database='{database}')",
    "ruby": "client = Mysql2::Client.new(username: '{username}@YOURRESOURCENAME', password: '{password}', database: '{database}', host: 'YOURRESOURCENAME.mysql.database.azure.com', port: 3306)"
  }
}
``` 