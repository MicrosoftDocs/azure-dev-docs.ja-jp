---
title: Visual Studio Code で Azure Functions アプリケーションをローカルで実行する
description: Azure にデプロイする前に、Azure Functions プロジェクトをローカルで実行してテストします。 サーバーレス関数が応答を返す直前に、ブレークポイントを設定します。
ms.topic: tutorial
ms.date: 03/02/2021
ms.custom: devx-track-js, contperf-fy21q2
ms.openlocfilehash: e16daa0f9c3db2edf2335c3f35277b1b95fcfef8
ms.sourcegitcommit: 576c878c338d286060010646b96f3ad0fdbcb814
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102117826"
---
# <a name="3-test-the-function-locally"></a>3.関数をローカルでテストする

[前の手順:Functions アプリを作成する](tutorial-vscode-serverless-node-create-local.md)

この手順では、Azure にデプロイする前に、Azure Functions プロジェクトをローカルで実行してテストします。 サーバーレス関数が応答を返す直前に、ブレークポイントを設定します。

Functions アプリを作成したときに、Azure Functions 拡張機能によって、VS Code 起動構成がプロジェクトに自動的に追加されました。これは、 *.vscode/launch.json* ファイルにあります。 この構成では、Azure で実行されるのと同じランタイムが使用されるため、クラウドにデプロイする前にソース コードが動作することを確認できます。

## <a name="run-the-local-serverless-function"></a>ローカルのサーバーレス関数を実行する

1. Visual Studio Code で、**F5** を押して (または **[デバッグ]**  >  **[Start Debugging]\(デバッグの開始\)** メニュー コマンドを使用して) デバッガーを起動し、Azure Functions ホストに接続します。 このコマンドでは、Azure Functions が作成したデバッグ構成が自動的に使用されます。

1. Functions Core ツールの出力は、VS Code の **[Terminal]\(ターミナル\)** パネルに表示されます。 ホストが起動したら、出力に表示されるローカル URL を **Alt** を押しながらクリックしてブラウザーを開き、関数を実行します。

    ![ローカルでデバッグをしているときに VS Code の [Terminal]\(ターミナル\) パネルに表示される出力](../media/functions-extension/local-test-output.png)

1. 既定の HTTP トリガーによって作成されたコードは、`name` クエリ パラメーターを解析して、応答をカスタマイズします。 ブラウザーで、`?name=<yourname>` をブラウザーの URL に追加して、応答が正しく出力されることを確認します。

    ![URL パラメーターを解析する HTTP トリガー関数](../media/functions-extension/local-test-browser.png)

## <a name="set-and-stop-at-break-point-in-serverless-app"></a>サーバーレス アプリのブレークポイントの設定と停止

1. 関数をローカルで実行すると、コードのさまざまな部分にブレークポイントを設定できます。 *index.js* を開き、エディター ウィンドウで 11 行目の左側の余白をクリックします。 ブレークポイントを示す小さな赤い点が表示されます。 次に、ブラウザーの URL から `?name=` 引数を削除します。 ブラウザーがその要求を行うと、VS Code はそのブレークポイントの関数コードを停止します。

    ![VS Code がブレークポイントで停止](../media/functions-extension/debugging-breakpoint.png)

    VS Code でのブレークポイントとデバッグの詳細については、「[デバッグ](https://code.visualstudio.com/docs/editor/debugging)」をご覧ください。

> [!div class="nextstepaction"]
> [Function アプリをローカルで実行しました](tutorial-vscode-serverless-node-deploy-hosting.md) [問題が発生しました](https://www.research.net/r/PWZWZ52?tutorial=node-deployment-azurefunctions&step=run-app)
