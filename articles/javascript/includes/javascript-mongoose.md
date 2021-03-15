---
ms.custom: devx-track-js
ms.topic: include
ms.date: 03/04/2021
ms.openlocfilehash: 6121b17a6155799c5a6d4d43e5870f7613ab2f0b
ms.sourcegitcommit: 737d95fe31e9db55c2d42a93f194a3f3e4bd3c7d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102622292"
---
## <a name="install-mongoose-sdk"></a>mongoose SDK のインストール 

JavaScript と mongoose を使用して Azure Cosmos DB 上の mongoDB に接続して使用するには、次の手順に従います。

1. Node.js と npm がインストールされていることを確認します。
1. 新しいフォルダーに Node.js プロジェクトを作成します。

    ```bash
    mkdir DataDemo && \
        cd DataDemo && \
        npm init -y && \
        npm install mongoose &&
        code .
    ```

    コマンドの動作:
    * `DataDemo` という名前のプロジェクト フォルダーを作成します
    * Bash ターミナルをそのフォルダーに変更します
    * プロジェクトを初期化します (これにより、`package.json` ファイルが作成されます)
    * SDK をインストールします
    * Visual Studio Code でプロジェクトを開きます

## <a name="use-mongoose-sdk-to-connect-to-mongodb-on-azure"></a>mongoose SDK を使用して Azure 上の MongoDB に接続する

1. 次の JavaScript コードを `index.js` にコピーします。
1. 
    :::code language="javascript" source="~/../js-e2e//database/mongodb/index_mongoose.js" :::
 
1. スクリプト内の `YOUR-CONNECTION-STRING` をリソースの接続文字列に置き換えます。 
1. スクリプトを実行します。

    ```bash
    node index.js
    ```

    結果は次のようになります。

    ```console
    find all
    loop {"_id":"6019a68a6ecddc35d536c92c","name":"Joan Smith","job":"Developer","__v":0}
    loop {"_id":"6019a68e6ecddc35d536c92d","name":"Bob Jones","job":"Quality Assurance","__v":0}
    loop {"_id":"6019a6916ecddc35d536c92e","name":"Michelle Roberts","job":"Program Manager","__v":0}
    succeeded
    ```