---
title: Visual Studio Code からコンテナー レジストリを使用する
description: Docker チュートリアル パート2、コンテナー レジストリを使用する
ms.topic: tutorial
ms.date: 09/20/2019
ms.custom: devx-track-js
ms.openlocfilehash: d790e36163a7e654e32c8d5a675d3f7abfad8a99
ms.sourcegitcommit: f723980ade4cbc13548a5d8ac3f3fa681b8a2dbd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/16/2020
ms.locfileid: "97609298"
---
# <a name="use-a-container-registry"></a>コンテナー レジストリを使用する

[前の手順:概要と前提条件](tutorial-vscode-docker-node-01.md)

この手順では、お使いのアプリ イメージに適したコンテナー レジストリを設定します。 その後、コンテナー対応のホスティング サービス (Azure App Service など) で、レジストリからイメージをプルできます。

このチュートリアルでは、イメージに対して [Azure Container Registry](https://azure.microsoft.com/services/container-registry/) を使用します。これは、セキュリティで保護されたプライベートなホスト型レジストリです。 ただし、ここで示されているツールとプロセスは、[Docker Hub](https://hub.docker.com/) などの他のレジストリでも機能します。

## <a name="create-an-azure-container-registry"></a>Azure コンテナー レジストリの作成

[Free 価格レベルのリソース](../../core/what-is-azure-for-javascript-development.md#free-tier-resources)を作成します。

1. Visual Studio Code で、**F1** または **Ctrl + Shift + P** キーを押してコマンド パレットを開きます。

1. [検索] ボックスに「**レジストリ**」と入力します。 結果から、 **[Azure Container Registry:レジストリの作成]** を選択します。

   ![Visual Studio Code での Docker エクスプローラー](../../media/deploy-containers/docker-create-registry.jpg)

1. 次の値を入力または選択します。

    - **[レジストリ名]** に、Azure 内で一意の、5 から 50 文字の英数字を含む名前を入力します。
    - **[SKU]** で **[Basic]** を選択します。
    - **[リソース グループ]** に、サブスクリプション内で一意の値を入力します。
    - **[場所]** で、近くのリージョンを選択します。

    Visual Studio Code によって、Azure でレジストリ作成プロセスが開始されます。 終了すると、次のような通知が表示されます。 この通知により、レジストリが正常に作成されたことが確認されます。

   ![Visual Studio Code でのレジストリが作成されたことの確認](../../media/deploy-containers/registry-created.jpg)

1. **Docker** エクスプローラーを開きます。 設定したレジストリ エンドポイントが **[Registries]\(レジストリ\)** の下に表示されていることを確認します。

   ![レジストリが Docker エクスプローラーに表示されていることの確認](../../media/deploy-containers/docker-explorer-registry.jpg)

## <a name="sign-in-to-azure-container-registry"></a>Azure Container Registry にサインインする

Azure レジストリは Docker 拡張機能に表示されますが、Container Registry にサインインするまでは、それらにイメージをプッシュすることはできません。

1. Visual Studio で、**Ctrl**+ **`** を選択して統合ターミナルを開きます。

1. 次の Azure CLI コマンドを実行して、Container Registry にサインインします。 `<your-registry-name>` を、作成したレジストリの名前に置き換えます。

    ```bash
    az acr login --name <your-registry-name>
    ```

> [!div class="nextstepaction"]
> [レジストリを作成しました](tutorial-vscode-docker-node-03.md) [問題が発生しました](https://www.research.net/r/PWZWZ52?tutorial=docker-extension&step=create-registry)
 
