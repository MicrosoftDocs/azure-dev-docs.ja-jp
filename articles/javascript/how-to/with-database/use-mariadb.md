---
title: Azure MariaDB で JavaScript を使用する
description: Azure で MariaDB データベースを作成または移動するには、MariaDB リソースが必要です。
ms.topic: how-to
ms.date: 02/04/2021
ms.custom: devx-track-js
ms.openlocfilehash: 320397584f78809e39e7eeaf7c1c6805755a82de
ms.sourcegitcommit: 98a7e855206ff463c1d95f93c23dd665b26a0aa1
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/09/2021
ms.locfileid: "100004732"
---
# <a name="develop-a-javascript-application-with-mariadb-on-azure"></a>Azure で MariaDB を使用して JavaScript アプリケーションを開発する

Azure で MariaDB データベースを作成、移動、または使用するには、**Azure Database for MariaDB** リソースが必要です。 リソースを作成し、データベースを使用する方法について説明します。

## <a name="create-an-azure-database-for-mariadb-resource"></a>Azure Database for MariaDB リソースを作成する 

次のものを使用してリソースを作成できます。

* Azure CLI
* [Azure Portal](https://ms.portal.azure.com/#create/Microsoft.MariaDBServer)
* [@azure/arm-mariadb](https://www.npmjs.com/package/@azure/arm-mariadb)

[!INCLUDE [Azure CLI commands](../../includes/azure-cli-mariadb.md)]

## <a name="view-and-use-your-mariadb-on-azure"></a>Azure で MariaDB を表示および使用する
JavaScript で MariaDB データベースを開発するときは、次のいずれかのツールを使用します。

* [Azure Cloud Shell](https://shell.azure.com/) の _mysql_ CLI
* [MySQL Workbench](https://www.mysql.com/products/workbench/)
* Visual Studio Code [拡張機能](https://marketplace.visualstudio.com/items?itemName=mtxr.sqltools-driver-mysql)

## <a name="use-sdk-packages-to-develop-your-mariadb-on-azure"></a>Azure で SDK パッケージを使用して MariaDB を開発する

Azure MariaDB では、次に示すような既に使用可能な npm パッケージが使用されます。

* [mariadb](https://www.npmjs.com/package/mariadb)

## <a name="use-mariadb-sdk-to-connect-to-mariadb-on-azure"></a>MariaDB SDK を使用して Azure 上の MariaDB に接続する

JavaScript を使用して Azure 上の MariaDB に接続して使用するには、次の手順に従います。

1. Node.js と npm がインストールされていることを確認します。
1. 新しいフォルダーに Node.js プロジェクトを作成します。

    ```bash
    mkdir mariaDbDemo && \
        cd mariaDbDemo && \
        npm init -y && \
        npm install mariadb && \
        touch index.js && \
        code .
    ```

    コマンドの動作:
    * `mariaDbDemo` という名前のプロジェクト フォルダーを作成します
    * Bash ターミナルをそのフォルダーに変更します
    * プロジェクトを初期化します (これにより、`package.json` ファイルが作成されます)
    * mariadb npm パッケージをインストールします
    * `index.js` スクリプト ファイル作成します
    * Visual Studio Code でプロジェクトを開きます

1. 次の JavaScript コードを `index.js` にコピーします。

    ```nodejs
    // To install npm package,
    // run following command at terminal
    // npm install mariadb

    // get mariadb SDK
    const mariadb = require('mariadb');

    // query server and close connection
    const query = async (config) => {
      // creation connection
      const connection = await mariadb.createConnection(config);

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
      host: 'YOUR-RESOURCE_NAME.mariadb.database.azure.com',
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
      { Database: 'information_schema' },
      { Database: 'mysql' },
      { Database: 'performance_schema' },
      { Database: 'quickstartdb' },
      { Database: 'tutorial' },
      meta: [
        ColumnDef {
          _parse: [StringParser],
          collation: [Collation],
          columnLength: 256,
          columnType: 253,
          flags: 1,
          scale: 0,
          type: 'VAR_STRING'
        }
      ]
    ]
    [
      { Tables_in_mysql: '__az_action_history__' },
      { Tables_in_mysql: '__az_changed_static_configs__' },
      { Tables_in_mysql: '__az_replica_information__' },
      { Tables_in_mysql: '__az_replication_current_state__' },
      { Tables_in_mysql: '__az_slave_relay_log_info__' },
      { Tables_in_mysql: '__firewall_rules__' },
      { Tables_in_mysql: '__script_version__' },
      { Tables_in_mysql: 'column_stats' },
      { Tables_in_mysql: 'columns_priv' },
      { Tables_in_mysql: 'db' },
      { Tables_in_mysql: 'event' },
      { Tables_in_mysql: 'func' },
      { Tables_in_mysql: 'general_log' },
      { Tables_in_mysql: 'gtid_slave_pos' },
      { Tables_in_mysql: 'help_category' },
      { Tables_in_mysql: 'help_keyword' },
      { Tables_in_mysql: 'help_relation' },
      { Tables_in_mysql: 'help_topic' },
      { Tables_in_mysql: 'host' },
      { Tables_in_mysql: 'index_stats' },
      { Tables_in_mysql: 'innodb_index_stats' },
      { Tables_in_mysql: 'innodb_table_stats' },
      { Tables_in_mysql: 'plugin' },
      { Tables_in_mysql: 'proc' },
      { Tables_in_mysql: 'procs_priv' },
      { Tables_in_mysql: 'proxies_priv' },
      { Tables_in_mysql: 'roles_mapping' },
      { Tables_in_mysql: 'servers' },
      { Tables_in_mysql: 'slow_log' },
      { Tables_in_mysql: 'table_stats' },
      { Tables_in_mysql: 'tables_priv' },
      { Tables_in_mysql: 'time_zone' },
      { Tables_in_mysql: 'time_zone_leap_second' },
      { Tables_in_mysql: 'time_zone_name' },
      { Tables_in_mysql: 'time_zone_transition' },
      { Tables_in_mysql: 'time_zone_transition_type' },
      { Tables_in_mysql: 'transaction_registry' },
      { Tables_in_mysql: 'user' },
      meta: [
        ColumnDef {
          _parse: [StringParser],
          collation: [Collation],
          columnLength: 292,
          columnType: 253,
          flags: 1,
          scale: 0,
          type: 'VAR_STRING'
        }
      ]
    ]
    [
      { User: 'azureAdmin' },
      { User: 'azure_superuser' },
      { User: 'azure_superuser' },
      { User: 'azure_superuser' },
      meta: [
        ColumnDef {
          _parse: [StringParser],
          collation: [Collation],
          columnLength: 320,
          columnType: 254,
          flags: 16515,
          scale: 0,
          type: 'STRING'
        }
      ]
    ]
    done
    ```

## <a name="next-steps"></a>次のステップ

* [JavaScript Web アプリをデプロイする](../deploy-web-app.md)方法
* [Azure Database for MariaDB](/azure/mariadb/)
* [Azure Database for MariaDB に移行するための移行ガイド](/azure/mariadb/howto-migrate-dump-restore)