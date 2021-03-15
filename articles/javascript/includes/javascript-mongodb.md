---
ms.custom: devx-track-js
ms.topic: include
ms.date: 03/04/2021
ms.openlocfilehash: a16cdd0846485622d6f61a049c3dfc2d848812f5
ms.sourcegitcommit: 737d95fe31e9db55c2d42a93f194a3f3e4bd3c7d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102622294"
---
## <a name="install-mongodb-sdk"></a>mongodb SDK のインストール 

JavaScript と mongodb を使用して Azure Cosmos DB 上の mongoDB に接続して使用するには、次の手順に従います。

1. Node.js と npm がインストールされていることを確認します。
1. 新しいフォルダーに Node.js プロジェクトを作成します。

    ```bash
    mkdir DataDemo && \
        cd DataDemo && \
        npm init -y && \
        npm install mongodb &&
        code .
    ```

    コマンドの動作:
    * `DataDemo` という名前のプロジェクト フォルダーを作成します
    * Bash ターミナルをそのフォルダーに変更します
    * プロジェクトを初期化します (これにより、`package.json` ファイルが作成されます)
    * SDK をインストールします
    * Visual Studio Code でプロジェクトを開きます

## <a name="create-javascript-file-to-bulk-insert-data-into-mongodb-database"></a>MongoDB データベースにデータを一括挿入する JavaScript ファイルを作成する

1. Visual Studio Code で、`bulk_insert.js` ファイルを作成します。

1. [MOCK_DATA.csv](https://github.com/Azure-Samples/js-e2e/blob/main/database/redis/MOCK_DATA.csv) ファイルをダウンロードし、`bulk_insert.js` と同じディレクトリに配置します。

1. 次の JavaScript コードを `bulk_insert.js` にコピーします。

    :::code language="javascript" source="~/../js-e2e//database/mongodb/bulk_insert_mongodb.js" :::

1. スクリプト内の次のものをリソース情報に置き換えます。

    * YOUR_RESOURCE_PRIMARY_CONNECTION_STRING

1. スクリプトを実行します。

    ```bash
    node bulk_insert.js
    ```

## <a name="create-javascript-code-to-use-mongodb"></a>MongoDB を使用する JavaScript コードを作成する

1. Visual Studio Code で、`index.js` ファイルを作成します。

1. 次の JavaScript コードを `index.js` にコピーします。

    :::code language="javascript" source="~/../js-e2e//database/mongodb/index_mongodb.js" :::

1. スクリプト内の次のものをリソース情報に置き換えます。

    * YOUR_RESOURCE_PRIMARY_CONNECTION_STRING

1. スクリプトを実行します。

    ```bash
    node index.js
    ```