---
title: Azure SDK for Go を使用する開発者向けのツール
description: Azure SDK for Go および Azure サービスを操作するためのツール
ms.date: 09/05/2018
ms.topic: conceptual
ms.openlocfilehash: e52845218de7d874e045a11c004360b7a35baa03
ms.sourcegitcommit: 39f3f69e3be39e30df28421a30747f6711c37a7b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/21/2020
ms.locfileid: "90831308"
---
# <a name="tools-for-developers-using-the-azure-sdk-for-go"></a>Azure SDK for Go を使用する開発者向けのツール

Go コードを効果的に記述し、Azure サービスでシームレスに動作させるための推奨ツールを紹介します。

## <a name="azure-cli"></a>Azure CLI

Azure CLI は、サブスクリプションで Azure リソースを作成して構成するためのコマンド ライン インターフェイスを提供します。 CLI を使用すると、一般的な共有 Azure リソースの作成を速やかに開始できるため、サービスのより複雑な用途に集中できます。 CLI はクエリとフィルターの機能を備えているので、好みのコマンド ライン ツールに出力を直接パイプ処理できます。 CLI はローカル システムにインストールすることも、Docker イメージとしてインストールすることもできます。また、[Azure Cloud Shell](/azure/cloud-shell/overview) を介してインストールすることも可能です。

> [!div class="nextstepaction"]
> [Azure CLI のインストール](/cli/azure/install-azure-cli)

## <a name="visual-studio-code"></a>Visual Studio Code

Visual Studio Code は、Go をサポートする軽量なエディターです。 この拡張機能により、オートコンプリート、`impl` テンプレート、リファクタリング、デバッグなどの機能が提供されます。 Visual Studio Code では、エディター内からソース管理にアクセスすることが可能で、Azure サービスを操作するための拡張機能も提供されています。

* [Visual Studio Code をインストールする](https://code.visualstudio.com/Download)
* [Visual Studio Code Go 拡張機能を入手する](https://code.visualstudio.com/docs/languages/go)
* [Visual Studio Code の Azure ツール拡張機能を入手する](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-azureextensionpack)

## <a name="cicd-with-azure-devops-project"></a>Azure DevOps プロジェクトによる CI/CD

Azure DevOps プロジェクトのパイプラインを使用すると、Go アプリケーションの継続的インテグレーション システムを設定できます。 Git リポジトリだけで、Azure 上で直接デプロイとテストを実行できます。

> [!div class="nextstepaction"]
> [Azure DevOps Projects で CI/CD パイプラインを作成する方法を確認する](/azure/devops-project/azure-devops-project-go)

## <a name="dependency-management-with-dep"></a>dep を使用した依存関係の管理

Azure SDK for Go では、依存関係の管理に dep を使用します。 dep コマンドを使用すると、Go アプリケーションのベンダー要件をプルし、バージョン間の競合を回避し、プロジェクトが適切に動作することを確認できます。

> [!div class="nextstepaction"]
> [dep 依存関係マネージャーを入手する](https://github.com/golang/dep)