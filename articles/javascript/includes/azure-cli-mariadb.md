---
ms.custom: devx-track-js
ms.topic: include
ms.date: 02/08/2021
ms.openlocfilehash: 0c441254e2c053198cc8e41e0d2457adbf3079af
ms.sourcegitcommit: 98a7e855206ff463c1d95f93c23dd665b26a0aa1
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/09/2021
ms.locfileid: "100004743"
---
## <a name="create-a-mariadb-resource-with-azure-cli"></a>Azure CLI を使用して MariaDB リソースを作成する

[Azure Cloud Shell](https://shell.azure.com) で次の Azure CLI [az mariadb server create](/cli/azure/mariadb/server#az_mariadb_server_create) コマンドを使用して、お使いのデータベースに新しい MariaDB リソースを作成します。 

```azurecli
az mariadb server create \
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
  "administratorLogin": "mariaAdmin",
  "connectionString": "mysql defaultdb --host mariadb-2.mariadb.database.azure.com --user mariaAdmin@mariadb-2 --password=123456789",
  "databaseName": "defaultdb",
  "earliestRestoreDate": "2021-02-08T19:14:17.317000+00:00",
  "fullyQualifiedDomainName": "mariadb-2.mariadb.database.azure.com",
  "id": "/subscriptions/bb881e62-cf77-4d5d-89fb-29d71e930b66/resourceGroups/my-resource-group/providers/Microsoft.DBforMariaDB/servers/mariadb-2",
  "location": "westus",
  "masterServerId": "",
  "name": "mariadb-2",
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
  "type": "Microsoft.DBforMariaDB/servers",
  "userVisibleState": "Ready",
  "version": "10.2"
}
```

プログラムを使用してサーバーに接続する前に、お使いのクライアント IP アドレスの通過を許可するようにファイアウォール規則を構成する必要があります。 

## <a name="add-firewall-rule-for-your-client-ip-address-to-mariadb-resource"></a>クライアント IP アドレスのファイアウォール規則を MariaDB リソースに追加する

既定では、ファイアウォール規則は構成されていません。 JavaScript を使用したサーバーへのクライアント接続が成功するように、お使いのクライアント IP アドレスを追加する必要があります。

```azurecli
az mariadb server firewall-rule create \
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

[Azure Cloud Shell](https://shell.azure.com) で次の Azure CLI [az mariadb db create](/cli/azure/mariadb/db#az_mariadb_db_create) コマンドを使用して、お使いのサーバーに新しい MariaDB データベースを作成します。 

```azurecli
az mariadb db create \
    --subscription YOUR-SUBSCRIPTION-ID-OR-NAME \
    --resource-group YOUR-RESOURCE-GROUP \
    --server-name YOURRESOURCENAME \
    --name YOURDATABASENAME
```

## <a name="get-the-mariadb-connection-string-with-azure-cli"></a>Azure CLI を使用して MariaDB 接続文字列を取得する

[az mariadb server show-connection-string](/cli/azure/mariadb/server#az_mariadb_server_show_connection_string) コマンドを使用して、このインスタンスの MariaDB 接続文字列を取得します。

```azurecli
az mariadb server show-connection-string \
    --subscription YOUR-SUBSCRIPTION-ID-OR-NAME \
    --server-name YOURRESOURCENAME \
    --database-name YOURDATABASENAME \
    --admin-user YOUR-ADMIN-NAME \
    --admin-password YOUR-ADMIN-PASSWORD 
```

これにより、一般的な言語用の接続文字列が JSON オブジェクトとして返されます。

```json
{
  "connectionStrings": {
    "ado.net": "Server=YOURRESOURCENAME.mariadb.database.azure.com; Port=3306; Database=YOURDATABASENAME; Uid=YOUR-ADMIN-NAME@YOURRESOURCENAME; Pwd=YOUR-ADMIN-PASSWORD",
    "jdbc": "jdbc:mariadb://YOURRESOURCENAME.mariadb.database.azure.com:3306/YOURDATABASENAME?user=YOUR-ADMIN-NAME@YOURRESOURCENAME&password=YOUR-ADMIN-PASSWORD",
    "node.js": "var conn = mysql.createConnection({host: 'YOURRESOURCENAME.mariadb.database.azure.com', user: 'YOUR-ADMIN-NAME@YOURRESOURCENAME',password: YOUR-ADMIN-PASSWORD, database: YOURDATABASENAME, port: 3306});",
    "php": "host=YOURRESOURCENAME.mariadb.database.azure.com port=3306 dbname=YOURDATABASENAME user=YOUR-ADMIN-NAME@YOURRESOURCENAME password=YOUR-ADMIN-PASSWORD",
    "python": "cnx = mysql.connector.connect(user='YOUR-ADMIN-NAME@YOURRESOURCENAME', password='YOUR-ADMIN-PASSWORD', host='YOURRESOURCENAME.mariadb.database.azure.com', port=3306, database='YOURDATABASENAME')",
    "ruby": "client = Mysql2::Client.new(username: 'YOUR-ADMIN-NAME@YOURRESOURCENAME', password: 'YOUR-ADMIN-PASSWORD', database: 'YOURDATABASENAME', host: 'YOURRESOURCENAME.mariadb.database.azure.com', port: 3306)"
  }
}
``` 




