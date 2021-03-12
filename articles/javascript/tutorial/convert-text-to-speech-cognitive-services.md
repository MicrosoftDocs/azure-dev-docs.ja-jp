---
title: Express.js アプリで Cognitive Services Speech を使用してテキストから音声への変換を行う
description: Cognitive Services Speech を使用してテキストを音声に変換します。これを、クライアント側とサーバー側で実行する方法を説明します。
ms.topic: tutorial
ms.date: 01/20/2021
ms.custom: languages:JavaScript, devx-track-javascript
ms.openlocfilehash: 028871f55a7be2e38191efef37a65ad41ab9c3dc
ms.sourcegitcommit: 576c878c338d286060010646b96f3ad0fdbcb814
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102117886"
---
# <a name="expressjs-app-converts-text-to-speech-with-cognitive-services-speech"></a>Express.js アプリで Cognitive Services Speech を使用してテキストから音声への変換を行う

このチュートリアルでは、既存の Express.js アプリに Cognitive Services Speech を追加して、Cognitive Services Speech サービスを使用したテキストから音声への変換を追加します。 テキストを音声に変換することにより、オーディオを手動で生成するコストをかけずに、オーディオを提供することができます。 

このチュートリアルでは、Azure Cognitive Services Speech からテキストを音声に変換する 3 種類の方法について説明します。

* クライアントの JavaScript でオーディオを直接取得する 
* サーバーの JavaScript でファイルからオーディオを取得する (*.MP3)
* サーバーの JavaScript でインメモリ arrayBuffer からオーディオを取得する

## <a name="application-architecture"></a>アプリケーションのアーキテクチャ

このチュートリアルでは、次の組み合わせを使用することにより、最小限の Express.js アプリを使用して機能を追加します。

* サーバー API によってテキストから音声への返還を行い、MP3 ストリームを返すための新しいルート
* HTML フォームによって情報を入力できるようにするための新しいルート
* JavaScript を使用した新しい HTML フォーム。クライアント側で Speech サービスを呼び出すことができます

このアプリケーションには、音声テキスト変換を行う、次の 3 つの異なる呼び出しが用意されています。

* 最初のサーバー呼び出しは、サーバー上にファイルを作成し、そのファイルをクライアントに返すためのものです。 この呼び出しは、通常、長いテキストや複数回使用するとわかっているテキストに使用します。 
* 2 番目のサーバー呼び出しは、短期のテキストのためのもので、クライアントに返される前のメモリ内の支援となります。 
* クライアント呼び出しは、SDK を使用した Speech サービスへの直接呼び出しを示すためのものです。 サーバーを使用しないクライアントのみのアプリケーションがある場合は、この呼び出しを選択できます。 

## <a name="prerequisites"></a>前提条件


- [Node.js 10.1+ および npm](https://nodejs.org/en/download) - ローカル コンピューターにインストール済み。
- [Visual Studio Code](https://code.visualstudio.com/) - ローカル コンピューターにインストール済み。 
- VS Code 用の [Azure App Service 拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureappservice) (VS Code 内からインストール)。
- [Git](https://git-scm.com/downloads) - GitHub にプッシュするために使用されます。これにより GitHub アクションがアクティブになります。
- Bash [![埋め込み起動](../../includes/media/cloud-shell-try-it/hdi-launch-cloud-shell.png "Azure Cloud Shell を起動する")](https://shell.azure.com)を使用して [Azure Cloud Shell](/azure/cloud-shell/quickstart) を使用します。   
- 必要に応じて、Azure CLI を[インストール](/cli/azure/install-azure-cli)して、CLI リファレンス コマンドを実行します。
   - ローカル インストールを使用する場合は、[az login](/cli/azure/reference-index#az-login) コマンドを使用して Azure CLI でサインインします。  認証プロセスを完了するには、ターミナルに表示される手順に従います。  サインイン オプションの詳細については、「[Azure CLI を使用してサインインする](/cli/azure/authenticate-azure-cli)」を参照してください。
  - 初回使用時にインストールを求められたら、Azure CLI 拡張機能をインストールします。  拡張機能の詳細については、「[Azure CLI で拡張機能を使用する](/cli/azure/azure-cli-extensions-overview)」を参照してください。
  - [az version](/cli/azure/reference-index?#az_version) を実行し、インストールされているバージョンおよび依存ライブラリを検索します。 最新バージョンにアップグレードするには、[az upgrade](/cli/azure/reference-index?#az_upgrade) を実行します。

## <a name="download-sample-expressjs-repo"></a>Express.js のサンプル リポジトリをダウンロードする 

1. git を使用して、Express.js のサンプル リポジトリをローカル コンピューターにクローンします。 

    ```bash
    git clone https://github.com/Azure-Samples/js-e2e-express-server
    ```

1. サンプル用の新しいディレクトリに移動します。

    ```bash
    cd js-e2e-express-server
    ```

1. Visual Studio Code でプロジェクトを開きます。

    ```bash
    code .
    ```

1. Visual Studio Code で新しいターミナルを開き、プロジェクトの依存関係をインストールします。

    ```bash
    npm install
    ```

## <a name="install-cognitive-services-speech-sdk-for-javascript"></a>JavaScript 用 Cognitive Services Speech SDK をインストールする

Visual Studio Code のターミナルから、Azure Cognitive Services Speech SDK をインストールします。

```bash
npm install microsoft-cognitiveservices-speech-sdk
```

## <a name="create-a-speech-module-for-the-expressjs-app"></a>Express.js アプリ用の Speech モジュールを作成する

1. Speech SDK を Express.js アプリケーションに統合するため、`src` フォルダーに `azure-cognitiveservices-speech.js` という名前のファイルを作成します。
1. 次のコードを追加して、依存関係を取得し、テキストを音声に変換するための関数を作成します。

    :::code language="javascript" source="~/../js-e2e-express-server-cognitive-services/text-to-speech/src/azure-cognitiveservices-speech.js" highlight="3,21,32" :::

    * パラメーター - このファイルは、SDK、ストリーム、バッファー、ファイル システム (fs) を使用するための依存関係を取得します。 `textToSpeech` 関数は、4 つの引数を受け取ります。 ローカル パスを含むファイル名が送信された場合、テキストはオーディオ ファイルに変換されます。 ファイル名が送信されない場合は、インメモリ オーディオ ストリームが作成されます。 
    * Speech SDK メソッド - Speech SDK メソッド [synthesizer.speakTextAsync](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechsynthesizer#speakTextAsync_string___e__SpeechSynthesisResult_____void___e__string_____void__AudioOutputStream___PushAudioOutputStreamCallback___PathLike_) は、受信した構成に基づいて異なる型を返します。 
        このメソッドは結果を返します。返される結果は、メソッドで実行するように要求された内容によって異なります。
        * [ファイルの作成] 
        * インメモリ ストリームをバッファーの配列として作成する
    * オーディオ形式 - 選択されているオーディオ形式は MP3 ですが、[別の形式](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechsynthesisoutputformat?preserve-view=true&view=azure-node-latest)や、別の[オーディオ構成メソッド](/javascript/api/microsoft-cognitiveservices-speech-sdk/audioconfig?preserve-view=true&view=azure-node-latest#methods)も存在します。 

    ローカルメソッド `textToSpeech` は、SDK のコールバック関数をラップし、Promise に変換します。 

## <a name="create-a-new-route-for-the-expressjs-app"></a>Express.js アプリ用の新しいルートを作成する

1. `src/server.js` ファイルを開きます。 
1. ファイルの先頭に、依存関係として `azure-cognitiveservices-speech.js` モジュールを追加します。

    :::code language="javascript" source="~/../js-e2e-express-server-cognitive-services/text-to-speech/src/server.js" range="3"  :::
    

1. このチュートリアルの前のセクションで作成した **textToSpeech** メソッドを呼び出す新しい API ルートを追加します。 

    :::code language="javascript" source="~/../js-e2e-express-server-cognitive-services/text-to-speech/src/server.js" range="30-51" highlight="45-50" :::

    このメソッドは、querystring から `textToSpeech` メソッドに必須および省略可能なパラメーターを受け取ります。 ファイルを作成する必要がある場合は、一意のファイル名が作成されます。 `textToSpeech` メソッドは非同期に呼び出され、その結果を応答 (`res`) オブジェクトにパイプ処理します。 

## <a name="update-the-client-web-page-with-a-form"></a>クライアントの Web ページをフォームで更新する 

必要なパラメーターを収集するフォームを使用して、クライアントの HTML Web ページを更新します。 省略可能なパラメーターは、ユーザーがどのオーディオ コントロールを選択するかに基づいて渡されます。 このチュートリアルでは、クライアントから Azure Speech サービスを呼び出すためのメカニズムについて説明しているため、その JavaScript も示します。 

`/public/client.html` ファイルを開いて、その内容を次のコードに置き換えます。

:::code language="html" source="~/../js-e2e-express-server-cognitive-services/text-to-speech/public/client.html" highlight="75, 102 137" :::

ファイル内の強調表示された行: 

* 行 74:Azure Speech SDK が、`cdn.jsdelivr.net` サイトを使用して NPM パッケージを配信するクライアント ライブラリに取り込まれます。 
* 行 102:`updateSrc` メソッドで、キー、リージョン、テキストを含む querystring を使用して、オーディオ コントロールの `src` URL が更新されます。 
* 行 137:ユーザーが `Get directly from Azure` ボタンを選択すると、Web ページはクライアント ページから Azure に直接呼び出しを実行して、結果を処理します。 

## <a name="create-cognitive-services-speech-resource"></a>Cognitive Services Speech リソースを作成する

Azure Cloud Shell で、Azure CLI コマンドを使用して Speech リソースを作成します。


1. [Azure Cloud Shell](https://shell.azure.com) にログインします。 そのためには、有効な Azure サブスクリプションに対するアクセス許可を持つアカウントを使用して、ブラウザーで認証を行う必要があります。 
1. Speech リソース用のリソース グループを作成します。 

    ```azurecli
    az group create \
        --location eastus \
        --name tutorial-resource-group-eastus
    ```

1. そのリソース グループ内に Speech リソースを作成します。

    ```azurecli
    az cognitiveservices account create \
        --kind SpeechServices \
        --location eastus \
        --name tutorial-speech \
        --resource-group tutorial-resource-group-eastus \
        --sku F0
    ```

    無料の Speech リソースのみが既に作成されている場合、このコマンドは失敗します。 

1. コマンドを使用して、新しい Speech リソースのキー値を取得します。 

    ```azurecli
    az cognitiveservices account keys list \
        --name tutorial-speech \
        --resource-group tutorial-resource-group-eastus \
        --output table
    ```

1. キーのいずれかをコピーします。 

    そのキーを Web フォームで使用して、Azure Speech サービスに対して認証を行います。

## <a name="run-the-expressjs-app-to-convert-text-to-speech"></a>Express.js アプリを実行してテキストを音声に変換する

1. 次の Bash コマンドを使用してアプリを起動します。

    ```bash
    npm start
    ```

1. ブラウザーで Web アプリを開きます。

    ```
    http://localhost:3000    
    ```

1. 強調表示されているテキスト ボックスに Speech キーを貼り付けます。 

    :::image type="content" source="../media/speech-tutorial/expressjs-webapp-form-with-speech-key-field-highlighted.png" alt-text="Speech キーの入力フィールドが強調表示された Web フォームのブラウザー スクリーンショット。":::

1. 必要に応じて、テキストを新しいものに変更します。 

1. 次の 3 つのボタンのいずれかを選択して、オーディオ形式への変換を開始します。
    * [Get directly from Azure]\(Azure から直接取得する\) - Azure へのクライアント側呼び出し
    * [Audio control for audio from file]\(ファイルからのオーディオ用オーディオ コントロール\)
    * [Audio control for audio from buffer]\(バッファーからのオーディオ用オーディオ コントロール\)

    コントロールを選択してからオーディオが再生されるまで、少し時間がかかる場合があります。 

## <a name="create-new-azure-app-service-in-visual-studio-code"></a>Visual Studio Code で新しい Azure アプリ サービスを作成する

1. コマンド パレット (**Ctrl**+**Shift**+**P**) で、「create web (Web の作成)」と入力して、 **[Azure App Service: Create New Web App...Advanced]\(Azure App Service: 新しい Web アプリの作成\)、[詳細設定]** の順に選択します。 詳細設定コマンドを使って、Linux の既定値を使用する代わりに、リソース グループ、App Service プラン、オペレーティング システムなどのデプロイを完全に制御できます。

1. プロンプトに次のように応答します。

    - **サブスクリプション** アカウントを選択します。
    - **[Enter a globally unique name]\(グローバルに一意の名前を入力する\)** には、`my-text-to-speech-app` など。 
        - Azure 全体で一意の名前を入力します。 英数字 ('A-Z'、'a-z'、および '0-9') とハイフン ('-') のみを使用します。
    - リソース グループの `tutorial-resource-group-eastus` を選択します。
    - `Node 10.1` 以上の **ランタイム スタックを選択** します。 
    - Linux オペレーティング システムを選択します。
    - **[新しい App Service プランの作成]** を選択し、名前を指定して (例: `my-text-to-speech-app-plan`)、 **[F1 Free]** [価格レベル](../core/what-is-azure-for-javascript-development.md#free-tier-resources)を選択します。
    - **F1** Free 価格レベルを選択します。
    - Application Insights リソースに対して **[Skip for now]\(今はしない\)** を選択します。
    - `eastus` の場所を選択します。 

1. しばらくすると、作成が完了したことが Visual Studio Code により通知されます。 **[X]** ボタンを使って、通知を閉じます。

## <a name="deploy-local-expressjs-app-to-remote-app-service-in-visual-studio-code"></a>Visual Studio Code でローカルの Express.js アプリをリモートのアプリ サービスにデプロイする

1. Web アプリを配置したら、ローカル コンピューターからコードをデプロイします。 Azure アイコンを選択して **Azure App Service** エクスプローラーを開き、サブスクリプション ノードを展開します。作成した Web アプリの名前を右クリックし、 **[Web アプリにデプロイ]** を選択します。

1. プロンプトで、Express.js アプリのルート フォルダーを選択し、**サブスクリプション** のアカウントをもう一度選択して、先ほど作成した Web アプリの名前 (`my-text-to-speech-app`) を選択します。

1. Linux へのデプロイ時に、ターゲット サーバーで `npm install` を実行するように構成を更新するよう求められる場合は、 **[はい]** を選択します。

    ![ターゲットの Linux サーバー上で構成を更新するように求めるメッセージ](../media/deploy-azure/server-build.png)

1. デプロイが完了したら、プロンプトで **[Web サイトの参照]** を選択して、新しくデプロイした Web アプリを表示します。 

1. (省略可能):コード ファイルに変更を加えてから、Azure App Service 拡張機能で **[Web アプリにデプロイ]** を使用して、Web アプリを更新することもできます。

## <a name="stream-remote-service-logs-in-visual-studio-code"></a>Visual Studio Code でリモート サービス ログをストリーミングする

実行中のアプリで `console.log` を呼び出すことによって生成される出力を表示 (テール) します。 この出力は、Visual Studio Code の **[出力]** ウィンドウに表示されます。

1. **Azure App Service** エクスプローラーで、新しいアプリ ノードを右クリックし、 **[Start Streaming Logs]\(ログのストリーム配信を開始する\)** を選択します。

    <pre>
    Starting Live Log Stream ---
    </pre>

1. ブラウザーで数回、Web ページを最新の情報に更新して追加のログ出力を確認します。


## <a name="clean-up-resources-by-removing-resource-group"></a>リソース グループを削除してリソースをクリーンアップする

このチュートリアルを完了したら、リソース グループを削除する必要があります。これにはリソースが含まれているため、そうすることにより、さらに使用して課金されることがないようにすることができます。 

Azure Cloud Shell で、[Azure CLI コマンド](/cli/azure/group#az_group_delete)を使用してリソース グループを削除します。

```azurecli
az group delete --name tutorial-resource-group-eastus  -y
```

このコマンドには数分かかる場合があります。 

## <a name="next-steps"></a>次のステップ

* [Visual Studio Code から App Service に Express.js MongoDB アプリをデプロイする](deploy-nodejs-mongodb-app-service-from-visual-studio-code.md)