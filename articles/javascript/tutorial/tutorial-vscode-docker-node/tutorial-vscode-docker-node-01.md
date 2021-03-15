---
title: Visual Studio Code から Docker コンテナーを Azure App Service にデプロイする
description: Docker チュートリアル パート 1、概要と前提条件。
ms.topic: tutorial
ms.date: 03/04/2021
ms.custom: devx-track-js
ms.openlocfilehash: a6058d682a8dec3b06d0854fed09534454270171
ms.sourcegitcommit: 576c878c338d286060010646b96f3ad0fdbcb814
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102117986"
---
# <a name="deploy-containers-to-azure-app-service"></a>Azure App Service にコンテナーをデプロイする

このチュートリアルでは、Visual Studio Code を使用して、Docker を使ってコンテナー化された Node.js アプリケーションを作成し、コンテナー イメージをレジストリにプッシュした後、イメージを Azure App Service にデプロイします。

## <a name="walkthrough-video"></a>チュートリアル ビデオ

この記事の内容の完全なチュートリアルについては、こちらのビデオをご覧ください。

> [!VIDEO https://channel9.msdn.com/Shows/Docs-Azure/Deploy-containers-Azure-App-Service/player]

## <a name="prerequisites"></a>前提条件

- [Azure サブスクリプション](#azure-subscription)。
- [Visual Studio Code](https://code.visualstudio.com/)。
- Visual Studio Code 拡張機能
    - [Azure Account 拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azure-account)
    - [Azure App Service 拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureappservice)。
    - [Azure Resources 拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureresourcegroups)
    - [Docker](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker)。
- [Node.js と npm](https://nodejs.org/en/download)、Node.js パッケージ マネージャー。
- [Docker](https://www.docker.com/community-edition)。

### <a name="azure-subscription"></a>Azure サブスクリプション

Azure サブスクリプションをお持ちでない場合は、無料アカウントに[今すぐご登録](https://azure.microsoft.com/free/?utm_source=campaign&utm_campaign=vscode-tutorial-docker-extension&mktingSource=vscode-tutorial-docker-extension)いただけます。Azure クレジットの 200 ドルを使用してさまざまな組み合わせのサービスをお試しください。

## <a name="sign-in-to-azure"></a>Azure へのサインイン

[!INCLUDE [azure-sign-in](../../includes/azure-sign-in.md)]

## <a name="verify-docker-install"></a>Docker のインストールを確認する

ターミナルまたはコマンド プロンプトで次のコマンドを実行して、Docker が正しくインストールされていることを確認します。

```bash
docker --version
```

出力は次のようになります。

<pre>
Docker Version 17.12.0-ce, build c97c6d6
</pre>

> [!div class="nextstepaction"]
> [Docker 拡張機能をインストールしました](tutorial-vscode-docker-node-02.md) [問題が発生しました](https://www.research.net/r/PWZWZ52?tutorial=docker-extension&step=getting-started)
