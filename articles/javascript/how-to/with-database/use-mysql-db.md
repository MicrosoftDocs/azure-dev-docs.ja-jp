---
title: Azure MySQL で JavaScript を使用する
description: Azure で MySQL データベースを作成または移動するには、MySQL リソースが必要です。
ms.topic: how-to
ms.date: 02/8/2021
ms.custom: devx-track-js
ms.openlocfilehash: caab884c0a943f00ccc1701a7d364bb0825bce78
ms.sourcegitcommit: 98a7e855206ff463c1d95f93c23dd665b26a0aa1
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/09/2021
ms.locfileid: "100004993"
---
# <a name="develop-a-javascript-application-with-mysql-on-azure"></a>Azure で MySQL を使用して JavaScript アプリケーションを開発する

Azure で MySQL データベースを作成、移動、または使用するには、**Azure Database for MySQL** リソースが必要です。 リソースを作成し、データベースを使用する方法について説明します。

## <a name="create-an-azure-database-for-mysql-resource"></a>Azure Database for MySQL リソースを作成する 

次のものを使用してリソースを作成できます。

* Azure CLI
* [Azure Portal](https://ms.portal.azure.com/#create/Microsoft.MySQLServer)
* [@azure/arm-mysql](https://www.npmjs.com/package/@azure/arm-mysql)

[!INCLUDE [Azure CLI commands](../../includes/azure-cli-mysql-db.md)]

## <a name="view-and-use-your-mysql-on-azure"></a>Azure で MySQL を表示および使用する
JavaScript を使用して MySQL データベースを開発するときは、次のいずれかのツールを使用します。

* [Azure Cloud Shell](https://shell.azure.com/) の _mysql_ CLI
* [MySQL Workbench](https://www.mysql.com/products/workbench/)
* Visual Studio Code [拡張機能](https://marketplace.visualstudio.com/items?itemName=mtxr.sqltools-driver-mysql)

## <a name="use-sdk-packages-to-develop-your-mysql-on-azure"></a>SDK パッケージを使用して Azure で MySQL を開発する

Azure MySQL では、次に示すような既に使用可能な npm パッケージが使用されます。

* [MySQL](https://www.npmjs.com/package/MySQL)
* [Promise-mysql](https://www.npmjs.com/package/promise-mysql)

## <a name="use-promise-mysql-sdk-to-connect-to-mysql-on-azure"></a>Promise-mysql SDK を使用して Azure 上の MySQL に接続する

JavaScript を使用して Azure 上の MySQL に接続して使用するには、次の手順に従います。

1. Node.js と npm がインストールされていることを確認します。
1. 新しいフォルダーに Node.js プロジェクトを作成します。

    ```bash
    mkdir MySQLDemo && \
        cd MySQLDemo && \
        npm init -y && \
        npm install promise-mysql && \
        touch index.js && \
        code .
    ```

    コマンドの動作:
    * `MySQLDemo` という名前のプロジェクト フォルダーを作成します
    * Bash ターミナルをそのフォルダーに変更します
    * プロジェクトを初期化します (これにより、`package.json` ファイルが作成されます)
    * promise-mysql npm パッケージをインストールします (async/await を使用するため)
    * `index.js` スクリプト ファイル作成します
    * Visual Studio Code でプロジェクトを開きます

1. 次の JavaScript コードを `index.js` にコピーします。

    ```nodejs
    // To install npm package,
    // run following command at terminal
    // npm install promise-mysql

    // get MySQL SDK
    const MySQL = require('promise-mysql');

    // query server and close connection
    const query = async (config) => {
      // creation connection
      const connection = await MySQL.createConnection(config);

      // show databases on server
      const databases = await connection.query('SHOW DATABASES;');
      console.log(databases);

      // show tables in the mysql database
      const tables = await connection.query('SHOW TABLES FROM mysql;');
      console.log(tables);

      // show users configured for the server
      const rows = await connection.query('select User from mysql.user;');
      console.log(rows);

      // close connection
      connection.end();
    };

    const config = {
      host: 'YOUR-RESOURCE_NAME.mysql.database.azure.com',
      user: 'YOUR-ADMIN-NAME@YOUR-RESOURCE_NAME',
      password: 'YOUR-ADMIN-PASSWORD',
      port: 3306,
    };

    query(config)
      .then(() => console.log('done'))
      .catch((err) => console.log(err));
    ```

1. ホスト、ユーザー、およびパスワードを、お使いの接続構成オブジェクト `config` のスクリプトの値に置き換えます。 

1. スクリプトを実行します。

    ```bash
    [
      RowDataPacket { Database: 'information_schema' },
      RowDataPacket { Database: 'defaultdb' },
      RowDataPacket { Database: 'dbproducts' },
      RowDataPacket { Database: 'mysql' },
      RowDataPacket { Database: 'performance_schema' },
      RowDataPacket { Database: 'sys' }
    ]
    [
      RowDataPacket { Tables_in_mysql: '__az_action_history__' },
      RowDataPacket { Tables_in_mysql: '__az_changed_static_configs__' },
      RowDataPacket { Tables_in_mysql: '__az_replica_information__' },
      RowDataPacket { Tables_in_mysql: '__az_replication_current_state__' },
      RowDataPacket { Tables_in_mysql: '__firewall_rules__' },
      RowDataPacket { Tables_in_mysql: '__querystore_event_wait__' },
      RowDataPacket { Tables_in_mysql: '__querystore_query_metrics__' },
      RowDataPacket { Tables_in_mysql: '__querystore_query_text__' },
      RowDataPacket {
        Tables_in_mysql: '__querystore_wait_stats_procedure_errors__'
      },
      RowDataPacket {
        Tables_in_mysql: '__querystore_wait_stats_procedure_status__'
      },
      RowDataPacket { Tables_in_mysql: '__recommendation__' },
      RowDataPacket { Tables_in_mysql: '__recommendation_session__' },
      RowDataPacket { Tables_in_mysql: '__script_version__' },
      RowDataPacket { Tables_in_mysql: 'columns_priv' },
      RowDataPacket { Tables_in_mysql: 'db' },
      RowDataPacket { Tables_in_mysql: 'engine_cost' },
      RowDataPacket { Tables_in_mysql: 'event' },
      RowDataPacket { Tables_in_mysql: 'func' },
      RowDataPacket { Tables_in_mysql: 'general_log' },
      RowDataPacket { Tables_in_mysql: 'gtid_executed' },
      RowDataPacket { Tables_in_mysql: 'help_category' },
      RowDataPacket { Tables_in_mysql: 'help_keyword' },
      RowDataPacket { Tables_in_mysql: 'help_relation' },
      RowDataPacket { Tables_in_mysql: 'help_topic' },
      RowDataPacket { Tables_in_mysql: 'innodb_index_stats' },
      RowDataPacket { Tables_in_mysql: 'innodb_table_stats' },
      RowDataPacket { Tables_in_mysql: 'ndb_binlog_index' },
      RowDataPacket { Tables_in_mysql: 'plugin' },
      RowDataPacket { Tables_in_mysql: 'proc' },
      RowDataPacket { Tables_in_mysql: 'procs_priv' },
      RowDataPacket { Tables_in_mysql: 'proxies_priv' },
      RowDataPacket { Tables_in_mysql: 'query_store' },
      RowDataPacket { Tables_in_mysql: 'query_store_wait_stats' },
      RowDataPacket { Tables_in_mysql: 'recommendation' },
      RowDataPacket { Tables_in_mysql: 'server_cost' },
      RowDataPacket { Tables_in_mysql: 'servers' },
      RowDataPacket { Tables_in_mysql: 'slave_master_info' },
      RowDataPacket { Tables_in_mysql: 'slave_relay_log_info' },
      RowDataPacket { Tables_in_mysql: 'slave_worker_info' },
      RowDataPacket { Tables_in_mysql: 'slow_log' },
      RowDataPacket { Tables_in_mysql: 'tables_priv' },
      RowDataPacket { Tables_in_mysql: 'time_zone' },
      RowDataPacket { Tables_in_mysql: 'time_zone_leap_second' },
      RowDataPacket { Tables_in_mysql: 'time_zone_name' },
      RowDataPacket { Tables_in_mysql: 'time_zone_transition' },
      RowDataPacket { Tables_in_mysql: 'time_zone_transition_type' },
      RowDataPacket { Tables_in_mysql: 'user' }
    ]
    [
      RowDataPacket { User: 'mySqlAdmin' },
      RowDataPacket { User: 'azure_superuser' },
      RowDataPacket { User: 'azure_superuser' },
      RowDataPacket { User: 'mysql.session' },
      RowDataPacket { User: 'mysql.sys' }
    ]
    done
    ```

## <a name="next-steps"></a>次のステップ

* [JavaScript Web アプリをデプロイする](../deploy-web-app.md)方法
* [Azure Database for MySQL](/azure/mysql/)
* [ダンプと復元を使用した移行](/azure/mysql/concepts-migrate-dump-restore)
* [MySQL Workbench を使用した移行](/azure/mysql/concepts-migrate-import-export)