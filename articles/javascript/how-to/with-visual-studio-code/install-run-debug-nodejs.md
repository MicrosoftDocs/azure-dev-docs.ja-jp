---
title: Visual Studio Code を使用して Node.js を開発する
description: Visual Studio を使用して JavaScript Node.js プロジェクトを開発し、デバッグする手順について説明します。
ms.topic: how-to
ms.date: 01/28/2021
ms.custom: devx-track-js
ms.openlocfilehash: 4a0aa42a51d76a8d4aaa9ae703e0ff3aa8b15fc0
ms.sourcegitcommit: 3f8aa923e4626b31cc533584fe3b66940d384351
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/01/2021
ms.locfileid: "99230132"
---
# <a name="how-to-develop-and-debug-nodejs-with-visual-studio-code"></a>Visual Studio Code を使用して Node.js の開発とデバッグを行う方法

Visual Studio を使用して JavaScript Node.js プロジェクトを開発し、デバッグする手順について説明します。 

## <a name="prepare-your-environment"></a>環境を準備する

1. [Visual Studio Code](https://code.visualstudio.com/) をインストールします。 
1. [Git](https://git-scm.com/) をインストールします。 Visual Studio Code は Git と統合されており、"*ソース管理*" の管理を[サイド バー](https://code.visualstudio.com/docs/getstarted/userinterface)で行えます。

1. MongoDB データベース接続文字列を取得します。

    使用できる MongoDB データベースがない場合は、以下を行うことができます。
    * コンテナーのいずれかが MongoDB データベースであるマルチコンテナー構成内でこのローカル プロジェクトを実行することを選択します。 [Docker](https://www.docker.com/) と [Remote - Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) 拡張機能をインストールして、ローカル MongoDB データベースを実行しているコンテナーのいずれかを使用してマルチコンテナー構成を取得します。 
    * MongoDB データベース用の [Azure CosmosDB](/azure/cosmos-db/) リソースを作成することを選択します。 詳細については、こちらの[チュートリアル](../../tutorial/deploy-nodejs-mongodb-app-service-from-visual-studio-code.md#create-a-cosmosdb-database-resource-for-mongodb)を参照してください。

## <a name="clone-sample-project-to-local-computer"></a>サンプル プロジェクトをローカル コンピューターに複製する

最初に、次の手順でサンプル プロジェクトをダウンロードします。

1. Visual Studio Code を開きます。

1. **F1** キーを押してコマンド パレットを表示します。

1. コマンド パレットのプロンプトに「`gitcl`」と入力し、**Git: Clone** コマンドを選択して、**Enter** キーを押します。

    ![Visual Studio Code コマンド パレットのプロンプトに gitcl コマンドを入力](../../media/node-howto-e2e/visual-studio-code-git-clone.png)

1. **リポジトリの URL** の入力を求められたら、「`https://github.com/scotch-io/node-todo`」と入力し、**Enter** キーを押します。

1. プロジェクトの複製先となるローカル ディレクトリを選択 (または作成) します。

    ![Visual Studio Code エクスプローラー](../../media/node-howto-e2e/visual-studio-code-explorer.png)

## <a name="use-the-integrated-bash-terminal-to-install-dependencies"></a>統合された bash ターミナルを使用して依存関係をインストールする

この Node.js プロジェクトでは、最初に npm からプロジェクトのすべての依存関係を確実にインストールしておく必要があります。  

1. **Ctrl**+ **`** キーを押して Visual Studio Code の統合ターミナルを表示します。 

1. 「`yarn`」と入力し、**Enter** キーを押します。  

     ![Visual Studio Code 内で yarn コマンドを実行](../../media/node-howto-e2e/visual-studio-code-install-yarn.png)

## <a name="navigate-the-project-files-and-code"></a>プロジェクト ファイルおよびコードを移動する

コードベースの操作に慣れるために、Visual Studio Code に備わっているナビゲーション機能の例をいくつか紹介します。

1. **Ctrl**+**P** キーを押します。

1. 「`.js`」と入力すると、プロジェクトに含まれているすべての JavaScript/JSON ファイルが、各ファイルの親ディレクトリと共に表示されます。 

1. *server.js* (アプリのスタートアップ スクリプト) を選択します。

1. **database** 変数 (6 行目のインポート) にマウス カーソルを合わせると、該当する型が表示されます。 ファイル内の変数/モジュール/型をすぐに調べることができるので、プロジェクトの開発時に役立ちます。 

    ![ホバー ヘルプを使用して Visual Studio Code で型を検出する](../../media/node-howto-e2e/visual-studio-code-hover-help.png)

1. 変数 (**database** など) 内のどこかでマウスをクリックすると、同じファイル内でその変数が使われている箇所をすべて確認できます。 プロジェクト内である変数が使われている箇所をすべて表示するには、その変数を右クリックし、コンテキスト メニューから、 **[すべての参照の検索]** を選択します。

    ![Visual Studio Code ですべての参照を検索する](../../media/node-howto-e2e/visual-studio-code-find-all-references.png)

1. マウス カーソルを変数に合わせることによってその型を調べることに加え、変数の定義を調べることもできます。別のファイルに存在していても問題ありません。 たとえば、**database.localUrl** (12 行目) を右クリックして、コンテキスト メニューから **[定義をここに表示]** を選択します。

    ![Visual Studio Code で変数の定義を表示する](../../media/node-howto-e2e/visual-studio-code-peek-definition.png)

## <a name="use-visual-studio-code-autocompletion-with-mongodb"></a>MongoDB で Visual Studio Code オート コンプリートを使用する

`database.localUrl` プロパティの宣言には、MongoDB の接続文字列がハードコーディングされています。 このセクションでは、環境変数から接続文字列を取得するようにコードを変更します。また、Visual Studio Code のオートコンプリート機能について説明します。  

1. *server.js* ファイルを開きます

1. 次のコードを探してください。

    ```javascript
    mongoose.connect(database.localUrl);
    ```

    次のコードに置き換えます。

    ```javascript
    mongoose.connect(process.env.MONGODB_URL || database.localUrl);
    ```

コードを (コピーして貼り付けるのではなく) 手動で入力した場合、`process` の後にピリオドを入力すると、Visual Studio Code によって Node.js の process グローバル API で使用できるメンバーが表示されます。

![process.env での VS Code 環境変数](../../media/node-howto-e2e/visual-studio-code-process-env.png)

オートコンプリートは、Visual Studio Code がバックグラウンドで TypeScript (JavaScript にも対応) を使用し、型情報を提供することによって実現されています。その型情報が、ユーザーが何か入力したときに、入力候補一覧に伝えられます。 Visual Studio Code は、Node.js プロジェクトを認識し、[Node.js に使用される TypeScript の型指定ファイルを NPM から](https://www.npmjs.com/package/@types/node)自動的にダウンロードします。 この型指定ファイルによって、他の Node.js グローバル (`Buffer`、`setTimeout` など) やすべての組み込みモジュール (`fs`、`http` など) でオートコンプリートが利用できるようになります。

型指定の自動取得は、Node.js の組み込み API に対してだけでなく、React、Underscore、Express など、2,000 個を超えるサードパーティ モジュールに対しても機能します。 たとえば、構成されている MongoDB データベース インスタンスに Mongoose が接続できなかった場合に、サンプル アプリがクラッシュしないようにするためには、次のコード行を 13 行目に挿入します。

```javascript
mongoose.connection.on("error", () => { console.log("DB connection error"); });
```

先ほどのコードと同様、何もしなくてもオートコンプリートが作動することがわかります。

![オートコンプリートによって API のメンバーが自動的に表示される](../../media/node-howto-e2e/visual-studio-code-autocomplete-mongoose.png)

このオートコンプリート機能がどのモジュールでサポートされているかについては、[DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped) プロジェクトを参照すると確認できます。このプロジェクトは、TypeScript のすべての型定義のソースとなるもので、コミュニティが中心となって作成しています。

## <a name="running-the-local-nodejs-app"></a>ローカル Node.js アプリの実行

コードをひととおり確認したら、実際にアプリを実行してみましょう。 Visual Studio Code でアプリを実行するには、**F5** キーを押します。 **F5** (デバッグ モード) でコードを実行すると、Visual Studio Code によってアプリが起動され、 **[デバッグ コンソール]** ウィンドウが表示されて、アプリの StdOut が表示されます。

![デバッグ コンソールでアプリの StdOut を監視](../../media/node-howto-e2e/visual-studio-code-debug-console.png)

また、**デバッグ コンソール** は最近実行したアプリにアタッチされるので、JavaScript の式を入力すれば、そのアプリ内で評価されます。オートコンプリートも機能します。 この動作を確認するには、コンソールに「`process.env`」と入力します。

![デバッグ コンソールにコードを入力](../../media/node-howto-e2e/visual-studio-code-debug-console-autocomplete.png)

先ほど **F5** キーを押してアプリを実行できたのは、現在開いているファイルが JavaScript ファイル (*server.js*) であるためです。 その結果、プロジェクトが Node.js アプリであると Visual Studio Code によって認識されました。 Visual Studio Code で JavaScript ファイルをすべて閉じて、**F5** キーを押した場合は、環境を選択するように求められます。

![ランタイム環境の指定](../../media/node-howto-e2e/visual-studio-code-select-environment.png)

ブラウザーを開いて `http://localhost:8080` に移動し、実行中のアプリを表示します。 テキスト ボックスにメッセージを入力し、いくつかの To Do の追加と削除を行って、アプリの動作を確かめます。

![アプリを使用して To Do を追加または削除する](../../media/node-howto-e2e/add-remove-todos-app.png)

## <a name="debugging-the-local-nodejs-app"></a>ローカル Node.js アプリのデバッグ

統合コンソールを使用してアプリを実行して操作できることに加え、コード内に直接ブレークポイントを設定できます。 たとえば、**Ctrl**+**P** キーを押して、ファイル ピッカーを表示します。 ファイル ピッカーが表示されたら、「`route`」と入力して *route.js* ファイルを選択します。

28 行目にブレークポイントを設定します。これは、アプリが To Do エントリを追加しようとしたときに呼び出される Express ルートを表します。 ブレークポイントは、次の図のようにエディター内で行番号の左側の領域をクリックするだけで設定できます。

![Visual Studio Code 内でブレークポイントを設定](../../media/node-howto-e2e/visual-studio-code-set-breakpoint.png)

> [!NOTE]
> Visual Studio Code では標準的なブレークポイントに加え、条件付きブレークポイントもサポートされており、アプリの実行を一時停止するタイミングをカスタマイズすることができます。 条件付きブレークポイントを設定するには、実行を一時停止する行の左側の領域を右クリックし、 **[条件付きブレークポイントの追加]** を選択します。次に、JavaScript の式 (例: `foo = "bar"`) または実行回数を指定して、実行を一時停止する条件を定義します。

ブレークポイントを設定したら、実行中のアプリに戻って、To Do エントリを追加します。 To Do エントリを追加するとすぐに、ブレークポイントを設定した 28 行目でアプリの実行が一時停止されます。

![Visual Studio Code によりブレークポイントで実行が一時停止される](../../media/node-howto-e2e/visual-studio-code-pause-breakpoint-execution.png)

アプリケーションが一時停止された後、コードの式にマウス カーソルを合わせると、現在の値を表示することができます。さらに、ローカル変数/ウォッチ式や呼び出し履歴を調べたり、デバッグ ツール バーを使ってコードをステップ実行することもできます。 **F5** キーを押すと、アプリの実行が再開されます。

## <a name="local-full-stack-debugging-in-visual-studio-code"></a>Visual Studio Code でのローカルのフルスタック デバッグ

このトピックの最初に述べたように、この To Do アプリは MEAN アプリです。つまり、そのフロントエンドとバックエンドはどちらも JavaScript で記述されています。 そのため現在はバックエンド (Node/Express) コードをデバッグしていますが、いずれフロントエンド (Angular) コードのデバッグが必要になることが考えられます。 そのため、Visual Studio Code には広大な拡張機能のエコシステムが存在します。その例として統合 Chrome デバッグが挙げられます。

**[拡張機能]** タブに切り替えて、検索ボックスに「`chrome`」と入力します。

![Visual Studio Code 用の Chrome デバッグ拡張機能](../../media/node-howto-e2e/visual-studio-code-chrome-extension.png)

**Debugger for Chrome** という名前の拡張機能を選択し、 **[インストール]** を選択します。 Chrome デバッグ拡張機能をインストールしたら、 **[再読み込み]** を選択してください。拡張機能をアクティブ化するために、Visual Studio Code を一度閉じて開き直します。

![Chrome デバッグ拡張機能をインストールした後で Visual Studio Code を再度読み込む](../../media/node-howto-e2e/visual-studio-code-reload-extension.png)

Node.js コードの実行とデバッグを行ううえで Visual Studio Code に特別な構成は必要ありませんでしたが、フロントエンド Web アプリをデバッグするためには、アプリの実行方法を Visual Studio Code に伝える *launch.json* ファイルを生成する必要があります。

## <a name="create-a-full-stack-launchjson-file-for-visual-studio-code"></a>Visual Studio Code 用のフルスタック launch.json ファイルを作成する

*launch.json* ファイルを生成するには、 **[デバッグ]** タブに切り替えて、歯車アイコン (上部に小さな赤色の点が付いています) を選択し、**node.js** 環境を選択します。

![launch.json ファイルを構成するための Visual Studio Code オプション](../../media/node-howto-e2e/visual-studio-code-debug-gear.png)

作成された *launch.json* ファイルは、次のようになります。アプリをデバッグするために、それをどのように起動し、どのようにアタッチするかが、このファイルによって Visual Studio Code に伝えられます。

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "node",
            "request": "launch",
            "name": "Launch Program",
            "program": "${workspaceRoot}/server.js"
        },
        {
            "type": "node",
            "request": "attach",
            "name": "Attach to Port",
            "address": "localhost",
            "port": 5858
        }
    ]
}
```

このアプリのスタートアップ スクリプトが *server.js* であることが Visual Studio Code で検出できました。

*launch.json* ファイルを開いた状態で、 **[構成の追加]** (右下) を選択し、**Chrome:Launch with userDataDir** を選択します。

![Chrome の構成を Visual Studio Code に追加](../../media/node-howto-e2e/visual-studio-code-add-chrome-config.png)

Chrome 用に新しい実行構成を追加することで、フロントエンド JavaScript コードをデバッグできるようになります。 

指定されているいずれかの設定にマウス カーソルを合わせると、その設定の内容についてのドキュメントが表示されます。 さらに、アプリの URL が Visual Studio Code によって自動的に検出されることがわかります。 アプリのフロントエンド アセットの検索対象となる場所を Chrome デバッガーが認識できるように、**webRoot** プロパティを `${workspaceRoot}/public` に更新します。

```json
{
   "type": "chrome",
   "request": "launch",
   "name": "Launch Chrome",
   "url": "http://localhost:8080",
   "webRoot": "${workspaceRoot}/public",
   "userDataDir": "${workspaceRoot}/.vscode/chrome"
}
```

フロントエンドとバックエンドの両方を同時に起動してデバッグするには、"*複合*" 実行構成を作成する必要があります。これにより、並列実行する構成のセットが Visual Studio Code に伝えられます。

次のスニペットを *launch.json* ファイル内の最上位のプロパティとして (既存の **configurations** プロパティの兄弟として) 追加します。

```json
"compounds": [
   {
      "name": "Full-Stack",
      "configurations": ["Launch Program", "Launch Chrome"]
   }
]
```

**compounds.configurations** 配列に指定した文字列値は、**configurations** のリストに含まれる各エントリの **name** を表します。 これらの名前を変更した場合は、配列に適宜変更を加える必要があります。 たとえば、[デバッグ] タブに切り替えて、選択構成を **[Full-Stack]** (複合構成の名前) に変更し、**F5** キーを押して実行します。

![Visual Studio Code での構成の実行](../../media/node-howto-e2e/visual-studio-code-full-stack-configuration.png)

この構成を実行すると、デバッグ コンソール出力に見られるような Node.js アプリと、`http://localhost:8080` の Node.js アプリに移動するように構成された Chrome が起動します。

**Ctrl**+**P** キーを押して、*todos.js* と入力 (または選択) します。これが、アプリのフロントエンドのメイン Angular コントローラーになります。

11 行目 (作成中の新しい To Do エントリのエントリ ポイント) にブレークポイントを設定します。

実行中のアプリに戻って新しい To Do エントリを追加すると、Visual Studio Code が今度は Angular コード内で実行を一時停止させていることがわかります。

![Visual Studio Code でのフロントエンド コードのデバッグ](../../media/node-howto-e2e/visual-studio-code-chrome-pause.png)

Node.js のデバッグと同様、式にマウス カーソルを置くことでローカル変数/ウォッチ式を確認したり、コンソール内で式を評価したりすることができます。 

ここで便利な機能を 2 つ紹介します。

1. **[呼び出し履歴]** ウィンドウには、2 つの異なるスタックが表示されます: **Node** と **Chrome**。これにより、現在どちらが一時停止されているのかがわかります。

1. フロントエンドとバックエンドのコードの間でステップ実行することができます。**F5** キーを押すと、Express ルートに先ほど設定したブレークポイントに到達します。

この設定によって、フロントエンド、バックエンド、またはフルスタックの JavaScript コードを Visual Studio Code 内から直接効率的にデバッグすることができます。

また、複合デバッガーの概念は、2 つのターゲット プロセスに限定されたものではなく、また JavaScript に限定されたものでもありません。 したがってマイクロサービス アプリ (多言語も含む) の開発を行う場合も、(言語/フレームワークの適切な拡張機能をインストールしたうえで) まったく同じワークフローを利用することができます。

## <a name="next-steps"></a>次のステップ

* [コンテナーをデプロイする](../deploy-containers.md)
