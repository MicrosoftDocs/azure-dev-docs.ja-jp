---
title: Visual Studio Code から静的 Node.js Web サイトを Azure にデプロイする
description: 静的 Web アプリのチュートリアル パート 1、概要と前提条件。
ms.topic: tutorial
ms.date: 09/23/2019
ms.custom: devx-track-js
ms.openlocfilehash: a4280e3c28ee99e38eb4430c991450667277b29f
ms.sourcegitcommit: f723980ade4cbc13548a5d8ac3f3fa681b8a2dbd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/16/2020
ms.locfileid: "97609314"
---
# <a name="deploy-a-static-website-to-azure-from-visual-studio-code"></a>Visual Studio Code から静的 Web サイトを Azure にデプロイする

このチュートリアルでは、[Azure Storage](/azure/storage) を使用して、静的 Web サイトを作成し、Azure にデプロイします。 静的 Web サイトは、HTML、CSS、JavaScript、および画像やフォントなどのその他のファイルで構成されています。 静的サイトは通常、Angular、React、または Vue で記述されたシングルページ アプリケーション ([SPA](https://en.wikipedia.org/wiki/Single-page_application)) です。 アプリの設計方法に関係なく、これらのファイルは、Web サーバーを使用するのではなく、_ストレージ_ から直接ホストして提供します。 ストレージでのホスティングは、Web サーバーを維持するよりも簡単でコストが低くなります。

## <a name="walkthrough-video"></a>チュートリアル ビデオ

この記事の内容の完全なチュートリアルについては、こちらのビデオをご覧ください。

> [!VIDEO https://channel9.msdn.com/Shows/Docs-Azure/Deploy-static-website-to-Azure-from-Visual-Studio-Code/player]

## <a name="prerequisites"></a>前提条件

- [Azure サブスクリプション](#azure-subscription)。
- [Visual Studio Code](https://code.visualstudio.com/)。
- [Azure Storage 拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurestorage)。
- [Node.js と npm](https://nodejs.org/en/download)、Node.js パッケージ マネージャー。 (この要件は、サンプル プロジェクトを生成するためにのみ使用されます。 既にアプリ コードを持っている場合は、Node.js をインストールする必要はありません。)

### <a name="azure-subscription"></a>Azure サブスクリプション

Azure サブスクリプションをお持ちでない場合は、無料アカウントに[今すぐご登録](https://azure.microsoft.com/free/?utm_source=campaign&utm_campaign=vscode-tutorial-static-website&mktingSource=vscode-tutorial-static-website)いただけます。Azure クレジットの 200 ドルを使用してさまざまな組み合わせのサービスをお試しください。

## <a name="sign-in-to-azure"></a>Azure へのサインイン

[!INCLUDE [azure-sign-in](../../includes/azure-sign-in.md)]

> [!div class="nextstepaction"] 
> [前提条件をインストールしました](tutorial-vscode-static-website-node-02.md) [問題が発生しました](https://www.research.net/r/PWZWZ52?tutorial=node-deployment-staticwebsite&step=getting-started)
