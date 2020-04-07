---
title: Visual Studio Code から Node.js アプリのコンテナー イメージをデプロイする
description: チュートリアル パート 4、Azure App Service にイメージをデプロイする
ms.topic: conceptual
ms.date: 09/20/2019
ms.openlocfilehash: 8fe8024adca9edda2142dc6582b6456b77ea4b8f
ms.sourcegitcommit: a65fa8dbb168bd39e225a293d9ee73d18ece1864
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/27/2020
ms.locfileid: "80362781"
---
# <a name="deploy-the-image-to-azure-app-service"></a>Azure App Service にイメージをデプロイする

[前の手順:アプリ イメージを作成する](tutorial-vscode-docker-node-03.md)

この手順では、レジストリにプッシュしたイメージを Visual Studio Code から直接 [Azure App Service](https://azure.microsoft.com/services/app-service/) にデプロイします。

1. **DOCKER** エクスプローラーで、 **[Registries]\(レジストリ\)** でイメージのノードを展開し、`:latest` を右クリックし、 **[Deploy Image to Azure App Service]\(Azure App Service にイメージをデプロイする\)** を選択します。

    ![エクスプローラーからデプロイする](media/deploy-containers/deploy-image-command.png)

1. 入力を求められたら、App Service の値を指定します。

    - この名前は Azure 全体で一意である必要があります。
    - 既存のリソース グループを選択するか、新しいリソース グループを作成します。 (**リソース グループ**は、基本的に、Azure のアプリケーション リソースの名前付きコレクションです。)
    - 既存の App Service プランを選択するか、新しいプランを作成します。 (Web サイトをホストする物理リソースは、**App Service プラン**によって定義されます。 このチュートリアルでは、Basic または無料プラン サービス レベルを使用できます)。
    - 新しい App Service プランの価格レベルを選択します。
    - 新しいリソースの場所を選択します (近くの場所)。

1. デプロイが完了すると、Visual Studio Code は、Web サイトの URL を示す通知を表示します。

    ![デプロイ成功のメッセージ](media/deploy-containers/deploy-successful.png)

1. 結果は、Visual Studio Code の **[出力]** パネルの **[Docker]** セクションでも確認することができます。

    ![デプロイ成功の出力](media/deploy-containers/deploy-output.png)

1. デプロイされた Web サイトを参照するには、 **[出力]** パネルの URL を **Ctrl**+**クリック**することもできます。 新しい App Service は、Visual Studio Code で **AZURE** エクスプローラーの **[APP SERVICE]** セクションにも表示されます。そこで、Web サイトを右クリックして、 **[Web サイトの参照]** を選択できます。

> [!div class="nextstepaction"]
> [サイトは Azure にあります](tutorial-vscode-docker-node-05.md) [問題が発生しました](https://www.research.net/r/PWZWZ52?tutorial=docker-extension&step=deploy-app)
