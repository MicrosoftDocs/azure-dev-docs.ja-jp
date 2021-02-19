---
title: Azure PostgreSQL で JavaScript を使用する
description: Azure で PostgreSQL データベースを作成または移動するには、PostgreSQL リソースが必要です。
ms.topic: how-to
ms.date: 02/8/2021
ms.custom: devx-track-js
ms.openlocfilehash: 2633a8b9e8f120a23af7840097c2930127e294ec
ms.sourcegitcommit: 98a7e855206ff463c1d95f93c23dd665b26a0aa1
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/09/2021
ms.locfileid: "100004986"
---
# <a name="develop-a-javascript-application-with-postgresql-on-azure"></a>Azure で PostgreSQL を使用して JavaScript アプリケーションを開発する

Azure で PostgreSQL データベースを作成、移動、または使用するには、**Azure Database for PostgreSQL サーバー** リソースが必要です。 リソースを作成し、データベースを使用する方法について説明します。

## <a name="create-an-azure-database-for-postgresql-resource"></a>Azure Database for PostgreSQL リソースを作成する 

リソースを作成するには、以下を使用します。

* [Azure CLI](../with-azure-cli/create-postgresql-server-resource.md)
* [Visual Studio Code](../with-visual-studio-code/create-azure-database.md#create-a-postgresql-database)
* [Azure Portal](https://ms.portal.azure.com/#create/Microsoft.PostgreSQLServer)
* [@azure/arm-postgresql](https://www.npmjs.com/package/@azure/arm-postgresql)

## <a name="view-and-use-your-postgresql-server-on-azure"></a>Azure で PostgreSQL サーバーを表示して使用する
JavaScript で PostgreSQL データベースを開発するときは、次のいずれかのツールを使用します。

* [Azure Cloud Shell](https://shell.azure.com/) -psql CLI を使用できます
* [pgAdmin](https://www.pgadmin.org/)
* Visual Studio Code [拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-cosmosdb)

## <a name="use-sdk-packages-to-develop-your-postgresql-server-on-azure"></a>SDK パッケージを使用して Azure で PostgreSQL サーバーを開発する

Azure PostgreSQL では、次に示すような既に使用可能な npm パッケージが使用されます。

* [pg](https://www.npmjs.com/package/pg)

## <a name="use-pg-sdk-to-connect-to-postgresql-on-azure"></a>pg SDK を使用して Azure 上の PostgreSQL に接続する

JavaScript を使用して Azure 上の PostgreSQL に接続して使用するには、次の手順に従います。

1. Node.js と npm がインストールされていることを確認します。
1. 新しいフォルダーに Node.js プロジェクトを作成します。

    ```bash
    mkdir DbDemo && \
        cd DbDemo && \
        npm init -y && \
        npm install pg && \
        touch index.js && \
        code .
    ```

    コマンドの動作:
    * `DbDemo` という名前のプロジェクト フォルダーを作成します
    * Bash ターミナルをそのフォルダーに変更します
    * プロジェクトを初期化します (これにより、`package.json` ファイルが作成されます)
    * pg npm パッケージをインストールします (async/await を使用するため)
    * `index.js` スクリプト ファイル作成します
    * Visual Studio Code でプロジェクトを開きます

1. 次の JavaScript コードを `index.js` にコピーします。

    ```nodejs
    const { Client } = require('pg')
    
    const query = async (connectionString) => {
        
        // create connection
        const connection = new Client(connectionString);
        connection.connect();
        
        // show tables in the postgres database
        const tables = await connection.query('SELECT table_name FROM information_schema.tables where table_type=\'BASE TABLE\';');
        console.log(tables.rows);
    
        // show users configured for the server
        const users = await connection.query('select pg_user.usename FROM pg_catalog.pg_user;');
        console.log(users.rows);
        
        // close connection
        connection.end();
    }
    
    const server='YOURRESOURCENAME';
    const user='YOUR-ADMIN-USER';
    const password='YOUR-PASSWORD';
    const database='postgres';

    const connectionString = `postgres://${user}@${server}:${password}@${server}.postgres.database.azure.com:5432/${database}`;
    
    query(connectionString)
    .then(() => console.log('done'))
    .catch((err) => console.log(err));
    ```

1. `YOUR-ADMIN-USER`、`YOURRESOURCENAME`、および `YOUR-PASSWORD` を、お使いの接続文字列のスクリプトの値に置き換えます。 

1. スクリプトを実行して `postgres` サーバーに接続し、ベース テーブルとユーザーを表示します。

    ```bash
    node index.js
    ```

1. 結果を表示します。 

    ```bash
    [
      { table_name: 'pg_statistic' },
      { table_name: 'pg_type' },
      { table_name: 'pg_authid' },
      { table_name: 'pg_user_mapping' },
      { table_name: 'pg_attribute' },
      { table_name: 'pg_proc' },
      { table_name: 'pg_class' },
      { table_name: 'pg_attrdef' },
      { table_name: 'pg_constraint' },
      { table_name: 'pg_inherits' },
      { table_name: 'pg_index' },
      { table_name: 'pg_operator' },
      { table_name: 'pg_opfamily' },
      { table_name: 'pg_opclass' },
      { table_name: 'pg_am' },
      { table_name: 'pg_amop' },
      { table_name: 'pg_amproc' },
      { table_name: 'pg_language' },
      { table_name: 'pg_largeobject_metadata' },
      { table_name: 'pg_aggregate' },
      { table_name: 'pg_rewrite' },
      { table_name: 'pg_largeobject' },
      { table_name: 'pg_trigger' },
      { table_name: 'pg_event_trigger' },
      { table_name: 'pg_description' },
      { table_name: 'pg_cast' },
      { table_name: 'pg_enum' },
      { table_name: 'pg_namespace' },
      { table_name: 'pg_conversion' },
      { table_name: 'pg_depend' },
      { table_name: 'pg_database' },
      { table_name: 'pg_db_role_setting' },
      { table_name: 'pg_tablespace' },
      { table_name: 'pg_pltemplate' },
      { table_name: 'pg_auth_members' },
      { table_name: 'pg_shdepend' },
      { table_name: 'pg_shdescription' },
      { table_name: 'pg_ts_config' },
      { table_name: 'pg_ts_config_map' },
      { table_name: 'pg_ts_dict' },
      { table_name: 'pg_ts_parser' },
      { table_name: 'pg_ts_template' },
      { table_name: 'pg_extension' },
      { table_name: 'pg_foreign_data_wrapper' },
      { table_name: 'pg_foreign_server' },
      { table_name: 'pg_foreign_table' },
      { table_name: 'pg_policy' },
      { table_name: 'pg_replication_origin' },
      { table_name: 'pg_default_acl' },
      { table_name: 'pg_init_privs' },
      { table_name: 'pg_seclabel' },
      { table_name: 'pg_shseclabel' },
      { table_name: 'pg_collation' },
      { table_name: 'pg_range' },
      { table_name: 'pg_transform' },
      { table_name: 'sql_features' },
      { table_name: 'sql_implementation_info' },
      { table_name: 'sql_languages' },
      { table_name: 'sql_packages' },
      { table_name: 'sql_parts' },
      { table_name: 'sql_sizing' },
      { table_name: 'sql_sizing_profiles' }
    ]
    [ { usename: 'azure_superuser' }, { usename: 'YOUR-ADMIN-USER' } ]
    done
    ```

## <a name="next-steps"></a>次のステップ

* [JavaScript Web アプリをデプロイする](../deploy-web-app.md)方法
* [Azure Database for PostgreSQL サーバー](/azure/postgresql/)