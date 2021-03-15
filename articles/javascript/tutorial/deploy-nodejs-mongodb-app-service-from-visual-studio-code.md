---
title: VSCode を使用して Express.js/MongoDB アプリをデプロイする - App Service/Cosmos DB
description: このチュートリアルでは、MongoDB ネイティブ API を使用して、MongoDB データベースで Node.js アプリを使用します。 Node.js アプリケーションを (Linux 上の) Azure App Service にデプロイし、ホストされているアプリが動作することを確認します。
ms.topic: tutorial
ms.date: 01/12/2020
ms.custom: scenarios:getting-started, languages:JavaScript, devx-track-javascript
ms.openlocfilehash: 9765078f6afa79bf42b722784401f20b8e51b4bc
ms.sourcegitcommit: 576c878c338d286060010646b96f3ad0fdbcb814
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102117906"
---
# <a name="deploy-expressjs-mongodb-app-to-app-service-from-visual-studio-code"></a>Visual Studio Code から App Service に Express.js MongoDB アプリをデプロイする

MongoDB に接続する Express.js アプリケーションを、(Linux 上で) Azure App Service および Cosmos DB にデプロイします。 

プログラミング作業は完了しており、このチュートリアルでは、Azure 拡張機能を使用して Visual Studio Code 内部からローカルおよびリモートの Azure 環境を正常に使用することに焦点を当てています。

* **[サンプル コード](https://github.com/Azure-Samples/js-e2e-express-mongo)**

## <a name="top-tasks"></a>上位のタスク

このチュートリアルには、JavaScript 開発者向けの **上位の Azure タスク** がいくつか含まれています。

* MongoDB データベースをホストするために Cosmos DB リソースを作成する
* Express.js アプリをホストするために App Service リソースを作成する
* Express.js アプリを App Service にデプロイする

## <a name="sample-application"></a>サンプル アプリケーション

[サンプルの Express.js アプリ](https://github.com/Azure-Samples/js-e2e-express-mongo)は、次の要素で構成されています。

* ポート 8080 でホストされている **Express.js サーバー**
* 単純な **React.js サーバー側ビュー** エンジン
* データを挿入、削除、検索する **MongoDB ネイティブ API** 関数

:::image type="content" source="../media/tutorial-end-to-end-app-cosmos/nodejs-app-connected-mongodb-form.png" alt-text="MongoDB データベースに接続された単純な Node.js アプリ。":::

## <a name="create-or-use-existing-azure-subscription"></a>Azure サブスクリプションを作成するか、既存のものを使用する 

* アクティブなサブスクリプションが含まれる Azure アカウント。 [無料で作成できます](https://azure.microsoft.com/free/)。

## <a name="install-software"></a>ソフトウェアをインストールする

- [Node.js 12 (LTS) と npm](https://nodejs.org/en/download) (ローカル コンピューターにインストールされている Node.js パッケージ マネージャー)。
- ローカル コンピューターにインストールされている [Visual Studio Code](https://code.visualstudio.com/)。 
- Visual Studio Code 拡張機能:
    - Visual Studio Code 用の [Azure App Service 拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureappservice) (Visual Studio Code 内からインストールされます)。
    - [Azure のデータベース](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-cosmosdb)

## <a name="create-a-cosmos-db-database-resource-for-mongodb"></a>MongoDB 用の Cosmos DB データベース リソースを作成する

最初に Cosmos リソースを作成します (これには数分かかるため)。 

1. Visual Studio Code で、左端のメニューから **Azure** アイコンを選択し、 **[データベース]** セクションを選択します。 

    **[データベース]** セクションが表示されていない場合は、上部にある Azure の **[...]** メニューでセクションをオンにしていることを確認してください。 

    :::image type="content" source="../media/tutorial-end-to-end-app-cosmos/vscode-azure-extension-select-database-section.png" alt-text="Visual Studio Code のリモート コンテナー アイコンの部分的なスクリーンショット"::: 

1. Azure エクスプローラーの **[データベース]** セクションで、右クリックでサブスクリプションを選択し、 **[サーバーの作成]** を選択します。
1. **[Create new Azure Database Server]\(新しい Azure データベース サーバーを作成する\)** コマンド パレットで、 **[Azure Cosmos DB for MongoDB API]\(MongDB 用 Azure Cosmos DB API\)** を選択します。 
1. 値がどのように使用されるかを理解するために次の表を使用しながら、プロンプトに従います。 データベースの作成には、最大 15 分かかる場合があります。

    |プロパティ|値|
    |--|--|
    |新しいリソースに対するグローバルに一意の **アカウント名** を入力します。| リソースに対して、`cosmos-mongodb-YOUR-NAME` などの値を入力します。 `YOUR-NAME` を自分の名前または一意の ID に置き換えます。 この一意の名前は、ブラウザーでリソースにアクセスするための URL の一部としても使用されます。|
    |リソース グループを選択または作成します。|リソース グループを作成する必要がある場合は、所有者、目的、およびリージョン (`westus-cosmostutorial-joesmith` など) を識別する名前付け規則を使用します。|
    |Location|リソースの場所。 このチュートリアルでは、自分の近くにあるリージョンの場所を選択します。|

    リソースの作成には、最大で 15 分かかる場合があります。 時間に制限がある場合は次のセクションをスキップすることができますが、次のセクションは数分で完了できるため、忘れずに戻ってきてください。

## <a name="get-cosmos-db-connection-string"></a>Cosmos DB の接続文字列を取得する

引き続き Azure データベース エクスプローラーでリソース名を右クリックし、 **[接続文字列のコピー]** を選択して接続文字列をコピーします。 これは後で環境変数ファイルのチュートリアルで必要になります。

:::image type="content" source="../media/tutorial-end-to-end-app-cosmos/vscode-databases-extenion-copy-connection-string.png" alt-text="Express.js Web アプリのフォームとローカル MongoDB からのデータの結果。":::

## <a name="download-and-run-the-sample-expressjs-app"></a>サンプルの Express.js アプリをダウンロードして実行する

Express.js の Web アプリが用意されています。 アプリをダウンロードし、依存関係をインストールして、アプリを実行します。 

1. ローカル コンピューターに [zip 圧縮された GitHub リポジトリをダウンロード](https://github.com/Azure-Samples/js-e2e-express-mongo.git)し、フォルダーに展開します。 
1. Visual Studio Code でそのフォルダーを開きます。 フォルダーを右クリックして **[Open with Code]\(コードで開く\)** を選択するか、フォルダー内で同等の CLI を使用することができます。

    ```bash
    code .
    ```

1. 環境ファイル `.env` を編集し、`DATABASE_URL` プロパティの値として Cosmos DB の接続文字列プロパティを追加します。 

    ```bash
    ENVIRONMENT=development
    DATABASE_NAME=
    DATABASE_COLLECTION_NAME=
    DATABASE_URL=
    ```

1. Visual Studio Code で、ターミナル ウィンドウを開き、次のコマンドを実行して、サンプルの依存関係をインストールし、Web アプリを起動します。

    ```bash
    npm install && npm start
    ```

1. ローカル コンピューターのブラウザーで Web アプリを表示します。

    ```url
    http://localhost:8080/
    ```

    :::image type="content" source="../media/tutorial-end-to-end-app-cosmos/nodejs-app-connected-mongodb-form.png" alt-text="MongoDB データベースに接続された単純な Node.js アプリ。":::

## <a name="create-web-app-resource-and-deploy-expressjs-app"></a>Web アプリ リソースを作成して Express.js アプリをデプロイする

App Service 用の Visual Studio Code 拡張機能を使用して App Service リソースを作成し、Express.js Web アプリをそのリソースにデプロイします。

1. Azure エクスプローラーに移動します。 サブスクリプションを右クリックし、[`Create new web app...`] を選択します。

    :::image type="content" source="../media/tutorial-end-to-end-app-cosmos/create-web-app-with-extension.png" alt-text="Azure App Service 拡張機能を使用して Web アプリを作成する Visual Studio Code の部分的なスクリーンショット。":::

1. 値がどのように使用されるかを理解するために次の表を使用しながら、プロンプトに従います。

    |プロパティ|値|
    |--|--|
    |新しい Web アプリのグローバルに一意の名前を入力します。| App Service リソースに `web-app-with-mongodb-YOUR-NAME` などの値を入力します。 `<YOUR-NAME>` を自分の名前または一意の ID に置き換えます。 この一意の名前は、ブラウザーでリソースにアクセスするために URL の一部としても使用されます。|
    |Linux アプリのランタイムを選択します。|[`Node 12 LTS`] を選択します。|

1. アプリの作成プロセスが完了すると、Visual Studio Code の右下隅にステータス メッセージが表示され、[`Deploy`] または [`View output`] を選択できます。 [`Deploy`] を選択します。

    :::image type="content" source="../media/tutorial-end-to-end-app-cosmos/vscode-app-extension-create-web-app-deploy-web-app.png" alt-text="Azure App Service 拡張機能を使用して Web アプリの作成直後に Web アプリをデプロイする Visual Studio Code の部分的なスクリーンショット。":::

    ステータス メッセージが表示されなくなったら、Azure エクスプローラーを選択してデプロイし、リソース名を右クリックして、 **[Web アプリにデプロイ]** を選択します。

1. デプロイ プロセス中、通知が表示され、**出力ウィンドウ** を表示することを選択できます。  これにより、デプロイのローリング状態が表示されます。 

1. デプロイが完了すると、通知が表示されます。 **[ストリーム ログ]** を選択して、ローリング ログを表示します。 

    :::image type="content" source="../media/tutorial-end-to-end-app-cosmos/vscode-app-service-deployed.png" alt-text="サービスがデプロイされています。[ストリーム ログ]。":::

    :::image type="content" source="../media/tutorial-end-to-end-app-cosmos/vscode-app-service-stream-logs.png" alt-text="デプロイが完了すると、[ストリーム ログ] を選択できる通知が表示されます。":::    

1. ブラウザーで Web サイトを開き、テキスト `YOUR-RESOURCE_NAME` を独自のリソース名 `https://YOUR-RESOURCE_NAME.azurewebsites.net` に置き換えます。
1. Web アプリを使用して、項目を追加および削除します。 

## <a name="make-changes-and-redeploy"></a>変更を加えて再デプロイする

いくつか変更を行って、App Service 拡張機能を使用してアプリを[再デプロイ](../how-to/deploy-web-app.md#deploy-or-redeploy-to-app-service-with-visual-studio-code)します。 

## <a name="clean-up-resources"></a>リソースをクリーンアップする 

このチュートリアルを完了したら、作成された 2 つのリソースを削除して、課金されないようにする必要があります。 

1. Visual Studio Code で、データベース用の Azure エクスプローラーを使用し、リソースを右クリックしてから、 **[アカウントの削除]** を選択します。
1. Visual Studio Code で、App Service 用の Azure エクスプローラーを使用し、リソースを右クリックしてから **[削除]** を選択します。

## <a name="next-steps"></a>次のステップ

App Service と Cosmos DB についての学習を続ける
* [アプリの設定を構成する方法について確認する](../how-to/configure-web-app-settings.md)
* [Azure App Service 向けの Node.js アプリを構成する](/azure/app-service/configure-language-nodejs?pivots=platform-linux)
* [SSH を使用した接続](/azure/app-service/configure-linux-open-ssh-session)
* [Cosmos DB にデータを移行する](/azure/dms/tutorial-mongodb-cosmos-db?toc=/azure/cosmos-db/toc.json?toc=/azure/cosmos-db/toc.json)
