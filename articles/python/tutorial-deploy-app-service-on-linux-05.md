---
title: 手順 5:VS Code を使用して Azure App Service on Linux に Python Web アプリをデプロイする
description: チュートリアルの手順 5、Web アプリのコードをデプロイする
ms.topic: conceptual
ms.date: 11/20/2020
ms.custom: devx-track-python, seo-python-october2019
ms.openlocfilehash: 7b3743d417ed3455c59f5b9887ee54728fb7318a
ms.sourcegitcommit: 29930f1593563c5e968b86117945c3452bdefac1
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/23/2020
ms.locfileid: "95485712"
---
# <a name="5-deploy-your-python-web-app-to-azure-app-service-on-linux"></a>5:Azure App Service on Linux に Python Web アプリをデプロイする

[前の手順: カスタム スタートアップ ファイルを構成する](tutorial-deploy-app-service-on-linux-04.md)

この手順を使用して、Azure App Service に Python アプリをデプロイします。

1. Visual Studio Code で **[Azure:App Service]** エクスプローラーを開き、青色の上矢印を選択します。

   ![App Service エクスプローラーで Web アプリを App Service にデプロイする](media/deploy-azure/deploy-web-app-to-app-service-in-app-service-explorer.png)

    または、App Service の名前を右クリックして **[Web アプリにデプロイ]** コマンドを選択する方法もあります。

1. 表示されたプロンプトで、次の情報を入力します。

    - "Select the folder to deploy (デプロイ先のフォルダーを選択してください)" では、現在のアプリ フォルダーを選択します。
    - "Select Web App (Web アプリを選択してください)" では、前の手順で作成した App Service を選択します。
    - ビルド コマンドを実行するためにビルド構成を更新することの確認が表示されたら、 **[Yes]\(はい\)** と答えます。
    - 既存のデプロイを上書きすることの確認が表示されたら、 **[Deploy]\(デプロイ\)** と答えます。
    - "always deploy the workspace (ワークスペースを常にデプロイする)" というメッセージが表示された場合は、 **[Yes]\(はい\)** と答えます。

1. デプロイ プロセスの進行中は、VS Code の **出力** ウィンドウで進行状況を確認できます。

    ![VS Code の出力ウィンドウに表示されるデプロイの進行状況](media/deploy-azure/view-deployment-progress-in-visual-studio-code-output.png)

1. 数分後 (インストールする必要のある依存関係の数によって異なります) にデプロイが完了すると、次のメッセージが表示されます。 **[Web サイトの参照]** ボタンを選択すると、実行中のサイトが表示されます。

    ![[Web サイトの参照] ボタンを使用したデプロイの完了](media/deploy-azure/web-app-deployment-complete-with-browse-website-button.png)

    ![App Service でアプリが正常に実行されている](media/deploy-azure/web-app-running-successfully-on-app-service.png)

1. まだ既定のアプリが表示されている場合は、デプロイ後にコンテナーが再起動されるまで 1、2 分待ってから、もう一度やり直してください。 カスタム スタートアップ コマンドを使用していて、それが正しいことを検証した場合、次に手順 6 に進み、ログを確認します。

1. ファイルがデプロイされていることを確認するには、 **[Azure: App Service]** エクスプローラーで App Service を選択し、 **[ファイル]** を展開します。

    ![App Service エクスプローラーでデプロイ ファイルを確認する](media/deploy-azure/expand-files-node-to-check-deployment-of-web-app-files.png)

    ご参考までに、ファイル *.deployment*、*antenv.tar.gz*、および *oryx-manifest.toml* は App Service ビルド システムで使用されます。 *hostingstart.html* は既定のアプリ ページです。

> [!div class="nextstepaction"]
> [アプリをデプロイしました - 手順 6 に進む >>>](tutorial-deploy-app-service-on-linux-06.md)

[問題がある場合は、お知らせください。](https://aka.ms/FlaskVSCQuickstartHelp)
