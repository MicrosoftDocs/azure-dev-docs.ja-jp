---
title: Visual Studio Code を使用して変更を加えた後に Azure App Service にコンテナーを再デプロイする
description: 'チュートリアル ステップ 6: コンテナー イメージをリビルドして再デプロイするための簡単な手順。'
ms.topic: conceptual
ms.date: 09/20/2019
ms.openlocfilehash: 7920bc9ddb2b9b7cc06f936fb97400a5c1d9dd7d
ms.sourcegitcommit: f89c59f772364ec717e751fb59105039e6fab60c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/06/2020
ms.locfileid: "80740650"
---
# <a name="make-changes-and-redeploy"></a>変更を加えて再デプロイする

[前の手順:アプリ イメージをデプロイする](tutorial-vscode-docker-node-05.md)

アプリには必然的に変更を加えるため、最終的にコンテナーのリビルドと再デプロイは何回も実施することになります。 さいわい、このプロセスは単純です。

1. アプリケーションに変更を加えて、ローカルでテストします

1. Visual Studio Code で**コマンド パレット**を開き (**F1**)、**Docker Images:Build Image** を実行してイメージをリビルドします。 アプリのコードのみを変更する場合は、ビルドにかかる時間はわずか数秒です。

1. イメージをレジストリにプッシュするには、再度**コマンド パレット**を開き (**F1**)、先ほどビルドしたイメージを選択して **Docker Images:Push** を実行します。 前と同様に、アプリ コードへの変更が小さく、そのレイヤーだけをプッシュすればよいので、通常このプロセスは数秒で完了します。

1. **[Azure: App Service]** エクスプローラーで、適切な App Service を右クリックし、 **[再起動]** を選択します。 App Service を再起動すると、最新のコンテナー イメージがレジストリから自動的にプルされます。

1. 約 15 から 20 秒後に、App Service の URL にもう一度アクセスして、更新プログラムを確認します。

> [!div class="nextstepaction"]
> [変更を確認しました](tutorial-vscode-docker-node-07.md) [問題が発生しました](https://www.research.net/r/PWZWZ52?tutorial=node-deployment-docker-extension&step=deploy-changes)
