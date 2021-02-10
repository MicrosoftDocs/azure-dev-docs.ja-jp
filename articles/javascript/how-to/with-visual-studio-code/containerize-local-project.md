---
title: ローカル プロジェクトからコンテナーを作成する
description: Visual Studio Code を使用して、ローカル開発プロジェクトの Dockerfile を作成します
ms.topic: how-to
ms.date: 01/28/2021
ms.custom: devx-track-js
ms.openlocfilehash: a299e4647e3257627b62242a4ab63b16fc5590a3
ms.sourcegitcommit: 3f8aa923e4626b31cc533584fe3b66940d384351
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/01/2021
ms.locfileid: "99230182"
---
# <a name="create-a-container-image-from-your-local-javascript-project"></a>ローカル JavaScript プロジェクトからコンテナー イメージを作成する

コンテナーでアプリを実行すると、Web アプリ ユーザーのために一貫したエクスペリエンスをデプロイできます。 Docker は習得が容易であることから、Visual Studio Code には、一般的な Docker タスクを単純化する拡張機能が用意されています。

## <a name="prepare-your-environment"></a>環境を準備する 

[Docker](https://www.docker.com/) をインストールして実行する必要があります。 次のコマンドを使用してこれを確認します。

```bash
docker system info
```

このコマンドでは、Docker がインストールされていない場合、エラーが返されます。 

## <a name="create-a-container"></a>コンテナーを作成する

1. Visual Studio で、既存の JavaScript プロジェクトを開きます。 
1. アクティビティ バーで **[Extensions]\(拡張機能\)** アイコンを選択し、次に `docker` を検索し、**Docker** の[拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker)を選択します。
1. Docker の拡張機能をインストールし、Visual Studio Code を再度読み込みます。

    ![Visual Studio Code 用の Docker 拡張機能をインストール](../../media/node-howto-e2e/visual-studio-code-docker-extension.png)

    Visual Studio Code 用 Docker 拡張機能には、既存プロジェクト用の *Dockerfile* および *docker-compose.yml* ファイルを生成するためのコマンドが含まれています。

1. アクティビティ バーで Docker アイコンを選択し、サイド バーに Docker コンテナーを表示します。

    :::image type="content" source="../../media/howto-containerize-local-project/docker-extension-activity-bar-side-bar.png" alt-text="&quot;git branch&quot; を検索し、次を選択します: &quot;Git: Create Branch&quot;。":::

## <a name="view-available-docker-commands"></a>利用可能な Docker コマンドを表示する

利用可能な Docker コマンドを確認するには、コマンド パレットを表示し (**F1** キー)、「`docker`」と入力します。

![Visual Studio Code 用 Docker 拡張機能でサポートされているコマンド ](../../media/node-howto-e2e/visual-studio-code-available-docker-codes.png)

## <a name="create-a-dockerfile-in-your-project"></a>プロジェクト内に Dockerfile を作成する

1. **git** などのソース管理を使用する場合は、他に変更がないことを確認します。 これにより、どのファイルが作成されるかを確認できます。

1. **Docker: Add docker files to workspace** (ワークスペースに docker ファイルを追加する) を、自分のプロジェクト用の設定を使用して選択します。 

    これらの設定は、Node.js プロジェクトに共通しています。

    |設定|値|
    |--|--|
    |アプリケーション プラットフォーム|Node.js|
    |Package.json|package.json|
    |公開するポート|8080 - または自動検出された値|
    |省略可能な docker ファイル (.dockerignore) を含める |yes|

    ![Visual Studio Code で生成された Dockerfile](../../media/node-howto-e2e/visual-studio-code-complete-dockerfile.png)

    この Docker コマンドによって、すぐに使うことができる完全な *Dockerfile* と Docker-compose ファイルが生成されます。

## <a name="build-image-from-your-project"></a>プロジェクトからのイメージをビルドする

1. **F1** キーを選択し、コマンド パレットに `dockerb` と入力して、次を選択します: **Docker: Build Image** コマンド。 
1. 先ほど生成した *Dockerfile* を選択します。 
1. package.json に name プロパティがある場合は、それがコンテナーのイメージ名として使用されます。 
    package.json がない場合は、`ALIAS/IMAGE-NAME` の形式でタグを指定します。ここで、ALIAS は Docker エイリアスで、IMAGE-NAME はプロジェクトのイメージの名前です。 タグの例は `diberry/express-web-app` です。 
1. **Enter** キーを選択して統合ターミナル ウィンドウを起動し、ビルド中の Docker イメージの出力を表示します。

    ![Docker イメージのビルド出力](../../media/node-howto-e2e/docker-build-image-output.png)

    このコマンドにより、`docker build` の実行プロセスが自動化されました。

1. アクティビティ バーで Docker アイコンを選択し、**イメージ** を選択すると、イメージの一覧に新しいイメージが表示されます。 
    
    また、一覧に他の新しいイメージが含まれていることに気付く場合もあります。 Docker によって、コンテナーの基になっているイメージが取得されます。  

## <a name="run-local-container-project"></a>ローカル コンテナー プロジェクトを実行する

1. アクティビティ バーから Docker アイコンを選択します。
1. **イメージ** の一覧からイメージ名を右クリックし、 **[Run]\(実行\)** を選択します。

    :::image type="content" source="../../media/howto-containerize-local-project/docker-extension-running-container-visual-studio-code.png" alt-text="イメージの一覧からイメージ名を右クリックし、[Run]\(実行\) を選択":::

## <a name="push-local-container-image-to-dockerhub"></a>ローカル コンテナーイメージを DockerHub にプッシュする

イメージから Azure Web アプリを作成するには、イメージをレジストリから利用できるようにする必要があります。 イメージは、コミュニティ レジストリ、または [Azure Container Registry](/azure/container-registry/) などの認証を使用してアクセスされるプライベート レジストリで公開できます。 

イメージをプッシュするには、CLI から `docker login` を実行し、アカウントの資格情報を入力して、あらかじめ DockerHub に対して認証を行っておく必要があります。

1. Visual Studio Code で、F1 を使用してコマンド パレットを開きます。
1. `dockerpush` を入力し、`Docker: Push` コマンドを選択します。 
1. 先ほどビルドしたイメージ タグ (例: `diberry/express-web-app`) を選択して、**Enter** キーを押します。 
1. このコマンドによって自動的に `docker push` が呼び出され、統合ターミナルに出力結果が表示されます。

## <a name="push-local-container-image-to-azure-container-registry"></a>Azure Container Registry にローカル コンテナー イメージをプッシュする

[認証して、独自の Azure Container Registry にプッシュする](../with-azure-cli/create-container-registry-resource.md)手順をお読みください。

## <a name="next-steps"></a>次のステップ

* [Azure Container Registry リソースを作成する](../with-azure-cli/create-container-registry-resource.md)