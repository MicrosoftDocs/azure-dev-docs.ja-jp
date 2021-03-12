---
title: Azure CLI を使用して Azure に Node.js アプリをデプロイした後にリソースをクリーンアップする
description: チュートリアル パート 7、 Azure CLI で ソースをクリーンアップする
ms.topic: tutorial
ms.date: 09/24/2019
ms.custom: devx-track-js, devx-track-azurecli
ms.openlocfilehash: d2b45ae60a7ad1270547289d1ea8480d14fedd95
ms.sourcegitcommit: 576c878c338d286060010646b96f3ad0fdbcb814
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102117966"
---
# <a name="part-7-clean-up-resources"></a>パート 7: リソースをクリーンアップする

[前の手順:変更を加えて再デプロイする](tutorial-vscode-azure-cli-node-05.md)

作成した App Service には、バッキング App Service プランが含まれていて料金が発生する可能性があります。 リソースをクリーンアップするには、ターミナルまたはコマンド プロンプトで次のコマンドを実行します。

```azurecli
az group delete --name myResourceGroup
```

> [!div class="nextstepaction"]
> [終わりました](../../how-to/deploy-web-app.md) [問題が発生しました](https://www.research.net/r/PWZWZ52?tutorial=node-deployment&step=clean-up-resources)