---
title: インクルード ファイル 3
description: インクルード ファイル 3
ms.topic: tutorial
ms.date: 06/01/2020
ms.custom: devx-track-js
ms.openlocfilehash: d186468b865ddf221c5232d5312018dd3d3858af
ms.sourcegitcommit: 5541f993c01ce356e1b0eaa8f95aea9051c3c21e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/03/2020
ms.locfileid: "93278672"
---
この手順では、Azure CLI を使用して Azure に Deno アプリをデプロイします。

1. 次のコマンドを使用して `deno-quickstart` という名前のリソース グループを作成します。

    ```bash
    az groups create -n deno-quickstart -l eastus
    ```

    リソース グループの名前を変更することにした場合、次の手順で必ずすべての `-g` フラグを更新してください。

1. このコマンドを使用し、Web サイトを保持する `deno-plan` という名前の AppService Plan を作成します。

    ```bash
    az appservice plan create -g deno-quickstart -n deno-plan --is-linux
    ```

1. 次に、Web アプリ自体を作成します。 このコマンドによって新しい AppService が作成され、以前に作成した Plan にバインドされます。 Web アプリに付ける名前の `<your-app-name>` タグを変更します。これは一意にする必要があります。

    ```bash
    az webapp create -n <your-app-name> -g deno-quickstart -p deno-plan -i anthonychu/azure-webapps-deno:1.0.2
    ```

    この AppService では、Deno コードを実行するための基本機能を提供する Docker イメージを実行します。 この処理は、完了するまでに数秒かかる場合があります。

1. 作成後、変数をいくつか構成する必要があります。 このコマンドを発行することでそれを行うことができます。

    ```bash
    az webapp config container set -n <your-app-name> -g deno-quickstart -i anthonychu/azure-webapps-deno:1.0.2 -r 'https://index.docker.io' -u '' -p  '' -t true && \
    az webapp config set -n <your-app-name> -g deno-quickstart --startup-file '' && \
    az webapp config appsettings set -n <your-app-name> -g deno-quickstart --settings WEBSITE_RUN_FROM_PACKAGE=1 WEBSITES_ENABLE_APP_SERVICE_STORAGE=true
    ```

これで AppService が構成されました。前の手順のアプリを受信する待機状態に入っています。 ただし、それを実行するには、`.zip` パッケージでアプリをパッケージ化する必要があります。 次の手順で行うことができます。

1. `deno-demo` フォルダーに移動します。

    ```bash
    cd deno-demo
    ```

1. `zip` コマンドを実行します

    ```bash
    zip demo demo.ts
    ```

    このコマンドの結果、`demo.ts` ファイルと同じフォルダーに `demo.zip` という名前のファイルが生成されます。

1. パッケージ化後、実行する AppService にファイルをアップロードできます。

    ```bash
    az webapp deployment source config-zip -n <your-app-name> -g deno-quickstart --src ./demo.zip && \
    az webapp config set -n <your-app-name> -g deno-quickstart --startup-file 'deno run --allow-net demo.ts'
    ```

1. `https://<your-app-name>.azurewebsites.net` に進むことでアプリケーションをテストします。