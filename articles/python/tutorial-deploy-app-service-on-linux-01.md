---
title: チュートリアル:Visual Studio Code から Azure App Service on Linux に Python アプリをデプロイする
description: チュートリアル ステップ 1、App Service の環境を構成する
ms.topic: conceptual
ms.date: 11/20/2020
ms.custom: devx-track-python, seo-python-october2019
adobe-target: true
ms.openlocfilehash: f73f968849cdf6e9324074427f6156a706d65dc0
ms.sourcegitcommit: b380f6e637b47e6e3822b364136853e1d342d5cd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/14/2021
ms.locfileid: "100395347"
---
# <a name="tutorial-deploy-python-apps-to-azure-app-service-on-linux-from-visual-studio-code"></a>チュートリアル:Visual Studio Code から Azure App Service on Linux に Python アプリをデプロイする

この記事では、Visual Studio Code と [Azure App Service](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureappservice) 拡張機能を使用して、Azure App Service on Linux に Python アプリケーションをデプロイする手順について説明します。

このチュートリアルのいずれかの手順で問題が発生した場合は、詳細をお知らせください。 フィードバックを送信するには、各記事の最後にある "**問題がある場合はお知らせください。** " のリンクを使用します。

デモ ビデオについては、仮想 PyCon 2020 の <a href="https://www.youtube.com/watch?v=dNVvFttc-sA&feature=youtu.be&ocid=AID3006292" target="_blank">VS Code と Azure App Service での WebApps のビルド</a>に関する動画 (youtube.com 2020) をご覧ください。

> [!NOTE]
> CLI を使用してアプリをデプロイする場合は、「 **[クイックスタート: Azure App Service on Linux で Python アプリを作成する](/azure/app-service/quickstart-python)** 」を参照してください。

> [!TIP]
> [Azure App Service on Linux](/azure/app-service/overview#app-service-on-linux) は、定義済みの Docker コンテナーでソース コードを実行します。 そのコンテナーでは、Python 3.6+ で [Gunicorn](https://gunicorn.org) Web サーバーを使用してアプリが実行されます。 このコンテナーの特性は、[App Service on Linux 向けの Python アプリの構成](/azure/app-service/configure-language-python)に関するページで説明されています。 コンテナーの定義は [github.com/Azure-App-Service/python](https://github.com/Azure-App-Service/python/tree/master/) にあります。

## <a name="configure-your-environment"></a>環境を構成する

- Azure アカウントとアクティブなサブスクリプションをお持ちでない場合は、[無料で作成できます](https://azure.microsoft.com/free/?utm_source=campaign&utm_campaign=vscode-tutorial-appservice-extension&mktingSource=vscode-tutorial-appservice-extension)。

- [Python 3.7 または 3.8 がローカルにインストールされている](https://python.org/downloads)ことを確認します。 バージョンを確認するには、次のコマンドを実行します。

    ```bash
    python --version
    ```

- 次のソフトウェアをインストールします。
  - [Visual Studio Code](https://code.visualstudio.com/)。
  - Python および [Python](https://marketplace.visualstudio.com/items?itemName=ms-python.python) 拡張機能。[VS Code Python チュートリアルの前提条件](https://code.visualstudio.com/docs/python/python-tutorial)に説明があります。
  - [Azure App Service](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureappservice) 拡張機能。VS Code 内から Azure App Service と対話できるようにする機能です。 全般的な情報については、[App Service 拡張機能のチュートリアル](https://code.visualstudio.com/tutorials/app-service-extension/getting-started)をご覧のうえ、[vscode-azureappservice GitHub リポジトリ](https://github.com/Microsoft/vscode-azureappservice)にアクセスしてください。

## <a name="sign-in-to-azure"></a>Azure へのサインイン

[!INCLUDE [azure-sign-in](includes/azure-sign-in.md)]

> [!div class="nextstepaction"]
> [Azure にサインインしました - 手順 2 に進む >>>](tutorial-deploy-app-service-on-linux-02.md)

[問題がある場合は、お知らせください。](https://aka.ms/FlaskVSCQuickstartHelp)
