---
title: 静的 Web サイトに Azure Storage を使用する
description: ファイルをストレージに読み込み、それらのファイルに対して Web 上で直接サービスを提供する方法について説明している Azure Storage のドキュメントへのリンクです。
ms.date: 02/17/2021
ms.topic: conceptual
ms.custom: devx-track-python
ms.openlocfilehash: e657a20d2cb5c3d4dde50d5b12fc31a3d9980ece
ms.sourcegitcommit: 576c878c338d286060010646b96f3ad0fdbcb814
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102118748"
---
# <a name="how-to-create-static-websites-on-azure-storage"></a>Azure Storage で静的 Web サイトを作成する方法

静的 Web サイトは、HTML、CSS、JavaScript、および画像やフォントなどのその他のファイルで構成されています。 静的サイトは通常、Angular、React、Vue など、さまざまな JavaScript フレームワークで記述されたシングルページ アプリケーション (SPA) です。

アプリの設計方法に関係なく、これらのファイルには、Web サーバーを使用するのではなく、Azure Storage から直接サービスを提供できます。 ストレージでのホスティングは、Web サーバーを維持するよりも簡単で、コストも大幅に削減できます。通常、静的ホスティングの 1 か月あたりのコストはわずかです。 サーバー側の処理がどの程度必要になるかによりますが、多くの場合、Azure Functions でサポートされているサーバーレス関数を使用してこれらのニーズに対応できます。

静的 Web サイトの作成については、下のリソースで詳しく説明しています。

- [Azure Storage での静的 Web サイト ホスティング](/azure/storage/blobs/storage-blob-static-website): 静的ホスティングのために Azure Storage を構成する方法の概要です。

- [Azure Storage で静的 Web サイトをホストする](/azure/storage/blobs/storage-blob-static-website-how-to?tabs=azure-cli): Azure portal、Azure CLI、または Azure PowerShell を使用して、静的ホスティングを有効にし、ファイルをアップロードし、その他のタスクを実行する方法についてのチュートリアルです。

- [GitHub Actions を使用して静的 Web サイトを Azure Storage にデプロイする方法](/azure/storage/blobs/storage-blobs-static-site-github-actions): 更新されたファイルをソース リポジトリから Azure Storage に自動的にデプロイするように GitHub Actions を構成する方法についてのチュートリアルです。

- [Visual Studio Code から静的 Web サイトを Azure にデプロイする](/azure/developer/javascript/tutorial/tutorial-vscode-static-website-node/tutorial-vscode-static-website-node-01): Angular、React、Vue、Svelte で単純な SPA を作成し、そのアプリを Visual Studio Code 内から Azure Storage にデプロイする方法にいて説明しているチュートリアルです。
