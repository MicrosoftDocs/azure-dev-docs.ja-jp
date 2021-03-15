---
title: ツールの選択 - JavaScript - Azure
description: Azure での Node.js および JavaScript 開発用の各ツールをインストールします
ms.topic: conceptual
ms.date: 03/02/2021
ms.custom: seo-javascript-september2019, seo-javascript-october2019, devx-track-js
ms.openlocfilehash: 8d34bae3e1ac789d24db59a41ab22673e15cc819
ms.sourcegitcommit: 576c878c338d286060010646b96f3ad0fdbcb814
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102117836"
---
# <a name="tools-for-javascript-developers-on-azure"></a>Azure での JavaScript 開発者向けツール 

JavaScript は多くのツールからなるエコシステムです。 この記事は、Microsoft が JavaScript 開発者向けに構築して管理するツールを選択したものです。 新しい Azure サービスやデプロイ/ホスティングのシナリオに対応するツールが向上しています。 

これらのツールを使用して Azure を使用する必要はありません。これらは単に機能とサポートの両方でエクスペリエンスを大幅に向上させるものです。 

## <a name="azure-portal"></a>Azure portal

[Azure portal](https://portal.azure.com/) を使用すると、アカウントのすべてのサブスクリプションとリソースにアクセスできます。 

## <a name="visual-studio-code"></a>Visual Studio Code

[Visual Studio Code](https://code.visualstudio.com) は、Azure の JavaScript 開発に適した IDE です。 インターフェイス、機能、および拡張機能が連携することで、開発時間が短縮され、開発のフラストレーションが軽減されます。 

ローカル開発プロジェクトのルートにプロジェクト ワークスペースを作成し、関連するすべての構成、設定、および拡張機能を追加します。 ワークスペース ファイルをプロジェクトにチェックインし、すべてのチーム メンバーがプロジェクトに必要な設定とツールにアクセスできるようにします。

Visual Studio Code の使用には、いくつかの利点があります。

* Visual Studio Code には、Azure のリファレンス ドキュメントがインラインで表示されます
* Visual Studio Code では、ステートメント入力候補が表示されます
* あいまいな型やオブジェクトがほとんどありません

Visual Studio Code には、[JavaScript プロジェクトで使用する](https://code.visualstudio.com/docs/nodejs/working-with-javascript)ための豊富なドキュメントが用意されています。 

## <a name="visual-studio-code-extensions"></a>Visual Studio Code の拡張機能

Visual Studio Code で直接 Azure サービスを使用するには、次の無料の拡張機能を使用します。

| ツール | 説明  |
|:---------:|---------|
|[Azure ツール](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack)|この Microsoft の 1 つの拡張機能で、Web サイトのホスティング、SQL と MongoDB のデータ、Docker コンテナー、サーバーレス関数などをすべて Azure 上で VS Code から取得できます。|

個々の拡張機能をインストールする場合、この一覧には最も人気のある Azure サービスが含まれています。

| ツール | 説明  |
|:---------:|---------|
| [Azure アカウント](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azure-account)| サインイン情報とサブスクリプションの管理|
| [Azure Functions](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions "Azure Functions 拡張機能へのリンク") <br> [![Azure Functions ツール](media/node-azure-tools/icon-azure-functions.png)](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) | 関数の作成、管理、表示、デバッグ、およびデプロイを行います|
| [Azure App Service](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureappservice "Azure App Service 拡張機能へのリンク") <br> [![App Service ツール](media/node-azure-tools/icon-azure-app-service.png)](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureappservice) | サイトと Azure portal の参照、新しいサイトの作成、スロットへのデプロイを行います |
| [Cosmos DB](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-cosmosdb "Cosmos DB 拡張機能へのリンク" )  <br> [![Cosmos DB ツール](media/node-azure-tools/icon-cosmos-db.png)](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-cosmosdb)| Azure でグローバル分散型のマルチモデル データベースを作成、参照、および更新します |
| [Docker](https://marketplace.visualstudio.com/items?itemName=formulahendry.docker-explorer)   <br> [![Docker](media/node-azure-tools/icon-docker.png)](https://marketplace.visualstudio.com/items?itemName=formulahendry.docker-explorer)| Docker コンテナーとイメージ、Docker Hub、および Azure コンテナー レジストリを管理します |
|[Storage](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurestorage)|BLOB コンテナー、ファイル共有、テーブル、およびキューを含む Azure Storage|

> [!div class="nextstepaction"]
> [Visual Studio Code Marketplace のその他の Azure 拡張機能 を取得する](https://marketplace.visualstudio.com/search?term=azure&target=VSCode&category=All%20categories&sortBy=Relevance)


## <a name="typescript"></a>TypeScript

[TypeScript](https://www.typescriptlang.org/download) は、JavaScript のすべての機能と、その上にこれらの追加のレイヤーを提供しています。TypeScript の型システム。 既存の動作している JavaScript コードも TypeScript コードです。 TypeScript の主な利点は、コード内の予期しない動作を強調表示し、バグの可能性を減らせることです。

## <a name="typescript-and-the-azure-sdk-client-libraries"></a>TypeScript と Azure SDK クライアント ライブラリ

クライアント ライブラリは TypeScript で記述されているため、Azure SDK クライアント ライブラリのリファレンス ドキュメントは TypeScript で記述されています。 Azure SDK クライアント ライブラリを使用するために TypeScript を使用する必要はありません。 

Azure SDK の TypeScript ガイドラインの詳細については、[こちら](https://azure.github.io/azure-sdk/typescript_introduction.html)を参照してください。


## <a name="azure-cli"></a>Azure CLI
Azure CLI は、コマンド ラインから Azure リソースを管理する目的に最適化されています。 

Azure CLI には、次の使用シナリオがあります。

* [Azure CLI のローカル インストール](/cli/azure/install-az-cli2)
* [Azure Cloud Shell](https://shell.azure.com/)
* [Docker コンテナー](/cli/azure/run-azure-cli-docker)

Azure portal を使用する場合、ポータルの上部のナビゲーション バーから Azure CLI を使用できます。

:::image type="content" source="media/azure-tools/azure-portal-select-azure-cloud-shell.png" alt-text="Azure portal を使用する場合、ポータルの上部のナビゲーション バーから Azure CLI を使用できる。":::

