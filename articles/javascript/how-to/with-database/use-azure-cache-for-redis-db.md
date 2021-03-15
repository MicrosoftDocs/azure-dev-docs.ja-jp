---
title: Azure の Redis で JavaScript を使用する
description: Azure で Redis データベースを作成または移動するには、Azure Cache for Redis リソースが必要です。
ms.topic: how-to
ms.date: 03/04/2021
ms.custom: devx-track-js
ms.openlocfilehash: cae27bea4326cb43e2f4c65590f9f2de46f02f7e
ms.sourcegitcommit: 737d95fe31e9db55c2d42a93f194a3f3e4bd3c7d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102622270"
---
# <a name="develop-a-javascript-application-with-azure-cache-for-redis"></a>Azure Cache for Redis を使用して JavaScript アプリケーションを開発する


Azure で Redis データベースを作成、移動、または使用するには、**Azure Cache for Redis** リソースが必要です。 リソースを作成し、データベースを使用する方法について説明します。

## <a name="create-a-resource-for-a-redis-database"></a>Redis データベース用のリソースを作成する

次のものを使用してリソースを作成できます。

* Azure CLI
* [Azure Portal](https://ms.portal.azure.com/#create/Microsoft.Cache)

[!INCLUDE [Azure CLI commands](../../includes/azure-cli-cache-for-redis-db.md)]

## <a name="view-and-use-your-redis-database"></a>Redis データベースを表示して使用する

JavaScript で Redis データベースを開発するときは、Azure portal から [Redis コンソール](/azure/azure-cache-for-redis/cache-configure#redis-console)を使用してデータベースを操作します。

:::image type="content" source="../../media/howto-database/azure-cache-for-redis-console-button.png" alt-text="JavaScript で Redis データベースを開発するときは、Azure portal から Redis コンソールを使用してデータベースを操作します。":::

このコンソールには、[Redis CLI](https://redis.io/topics/rediscli) 機能が用意されています。 [一部のコマンドがサポートされていない](/azure/azure-cache-for-redis/cache-configure#redis-commands-not-supported-in-azure-cache-for-redis)ことに注意してください。

リソースを作成したら、Azure portal を使用して Azure Storage から Redis リソースに[データをインポート](/azure/azure-cache-for-redis/cache-how-to-import-export-data)します。 

## <a name="use-native-sdk-packages-to-connect-to-redis-on-azure"></a>ネイティブ SDK パッケージを使用して Azure 上の Redis に接続する

Redis データベースでは、次のような npm パッケージが使用されます。

* [redis](https://www.npmjs.com/package/redis)
* [ioredis](https://www.npmjs.com/package/ioredis)

## <a name="install-ioredis-sdk"></a>ioredis SDK のインストール 

`ioredis` パッケージをインストールし、プロジェクトを初期化するには、次の手順に従います。

1. Node.js と npm がインストールされていることを確認します。
1. 新しいフォルダーに Node.js プロジェクトを作成します。

    ```bash
    mkdir DataDemo && \
        cd DataDemo && \
        npm init -y && \
        npm install ioredis \
        code .
    ```

    コマンドの動作:
    * `DataDemo` という名前のプロジェクト フォルダーを作成します
    * Bash ターミナルをそのフォルダーに変更します
    * プロジェクトを初期化します (これにより、`package.json` ファイルが作成されます)
    * ioredis npm SDK をプロジェクトに追加します
    * Visual Studio Code でプロジェクトを開きます

## <a name="create-javascript-file-to-bulk-insert-data-into-redis"></a>Redis にデータを一括挿入する JavaScript ファイルを作成する

1. Visual Studio Code で、`bulk_insert.js` ファイルを作成します。

1. [MOCK_DATA.csv](https://github.com/Azure-Samples/js-e2e/blob/main/database/redis/MOCK_DATA.csv) ファイルをダウンロードし、`bulk_insert.js` と同じディレクトリに配置します。

1. 次の JavaScript コードを `bulk_insert.js` にコピーします。

    ```nodejs
    const Redis = require('ioredis');
    const fs = require('fs');
    const parse = require('csv-parser')
    const { finished } = require('stream/promises');
    
    const config = {
        "HOST": "YOUR-RESOURCE-NAME.redis.cache.windows.net",
        "KEY": "YOUR-RESOURCE-PASSWORD",
    }
    
    // Create Redis config object
    const configuration = {
        host: config.HOST,
        port: 6380,
        password: config.KEY,
        tls: {
            servername: config.HOST
        },
        database: 0,
        keyPrefix: config.prefix
    }
    var redis = new Redis(configuration);
    
    // insert each row into Redis
    async function insertData(readable) {
        for await (const row of readable) {
            await redis.set(`bar2:${row.id}`, JSON.stringify(row))
        }
    }
    
    // read file, parse CSV, each row is a chunk
    const readable = fs
        .createReadStream('./MOCK_DATA.csv')
        .pipe(parse());
    
    // Pipe rows to insert function
    insertData(readable)
    .then(() => {
        console.log('succeeded');
        redis.disconnect();
    })
    .catch(console.error);
    ```

1. スクリプト内の次のものを Redis リソース情報に置き換えます。

    * YOUR-RESOURCE-NAME
    * YOUR-AZURE-REDIS-RESOURCE-KEY

1. スクリプトを実行します。

    ```bash
    node bulk_insert.js
    ```
    
## <a name="create-javascript-code-to-use-redis"></a>Redis を使用する JavaScript コードを作成する

1. Visual Studio Code で、`index.js` ファイルを作成します。


1. 次の JavaScript コードを `index.js` にコピーします。

    ```nodejs
    const redis = require('ioredis');
    
    const config = {
        "HOST": "YOUR-RESOURCE-NAME.redis.cache.windows.net",
        "KEY": "YOUR-RESOURCE-PASSWORD",
        "TIMEOUT": 300,
        "KEY_PREFIX": "demoExample:"
    }
    
    // Create Redis config object
    const configuration = {
        host: config.HOST,
        port: 6380,
        password: config.KEY,
        timeout: config.TIMEOUT,
        tls: {
            servername: config.HOST
        },
        database: 0,
        keyPrefix: config.KEY_PREFIX
    }
    
    const connect = () => {
        return redis.createClient(configuration);
    }
    
    const set = async (client, key, expiresInSeconds=configuration.timeout, stringify=true, data) => {
        return await client.setex(key, expiresInSeconds, stringify? JSON.stringify(data): data);
    }
    
    const get = async (client, key, stringParse=true) => {
        const value = await client.get(key);
        return stringParse ? JSON.parse(value) : value;
    }
    
    const remove = async (client, key) => {
          return await client.del(key);
    }
    
    const disconnect = (client) => {
        client.disconnect();
    }
    
    const test = async () => {
        
        // connect
        const dbConnection = await connect();
        
        // set
        const setResult1 = await set(dbConnection, "r1", "1000000", false, "record 1");
        const setResult2 = await set(dbConnection, "r2", "1000000", false, "record 2");
        const setResult3 = await set(dbConnection, "r3", "1000000", false, "record 3");
    
        // get
        const val2 = await get(dbConnection, "r2", false);
        console.log(val2);
        
        // delete
        const remove2 = await remove(dbConnection, "r2");
        
        // get again = won't be there
        const val2Again = await get(dbConnection, "r2", false);
        console.log(val2Again);
        
        // done
        disconnect(dbConnection)
    }
    
    test()
    .then(() => console.log("done"))
    .catch(err => console.log(err))
    ```
 
1. スクリプト内の次のものを Redis リソース情報に置き換えます。

    * YOUR-RESOURCE-NAME
    * YOUR-RESOURCE-PASSWORD

1. スクリプトを実行します。

    ```bash
    node index.js
    ```
    
    このスクリプトは、3 つのキーを挿入し、中央のキーを削除します。 コンソールの結果は次のとおりです。

    ```console
    record 2
    null
    done
    ```

## <a name="use-redis-console-in-azure-portal-to-view-data"></a>Azure portal で Redis コンソールを使用してデータを表示する

Azure portal で、コマンド `SCAN 0 COUNT 1000 MATCH *` を使用してリソースのコンソールを表示します。 

:::image type="content" source="../../media/howto-database/azure-cache-for-redis-azure-portal-console-scan.png" alt-text="Azure portal で、コマンド `SCAN 0 COUNT 1000 MATCH *` を使用してリソースのコンソールを表示します。":::

## <a name="next-steps"></a>次のステップ

* [JavaScript Web アプリをデプロイする](../deploy-web-app.md)方法
* [Azure Cache for Redis のドキュメント](/azure/azure-cache-for-redis)
* [Azure Cache for Redis のクイックスタート](/azure/azure-cache-for-redis/cache-nodejs-get-started)
* [Azure アーキテクチャ センター - キャッシュに関するベスト プラクティス](/azure/architecture/best-practices/caching)
* [Azure Cache for Redis のベスト プラクティス](/azure/azure-cache-for-redis/cache-best-practices#client-library-specific-guidance)