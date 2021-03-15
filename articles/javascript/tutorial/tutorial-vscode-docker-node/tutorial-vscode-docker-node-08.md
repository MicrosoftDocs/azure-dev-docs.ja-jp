---
title: コンテナー化された Node.js アプリを Visual Studio Code からデプロイした後にリソースをクリーンアップする
description: Docker チュートリアル パート 8、リソースをクリーンアップする
ms.topic: tutorial
ms.date: 09/20/2019
ms.custom: devx-track-js
ms.openlocfilehash: 4a7ff0e9a84e10223bbe8b55f5309506593f6e07
ms.sourcegitcommit: 576c878c338d286060010646b96f3ad0fdbcb814
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102117896"
---
# <a name="part-8-clean-up-resources"></a>パート 8 リソースをクリーンアップする

[前の手順:ログのストリーミング](tutorial-vscode-docker-node-07.md)

## <a name="delete-resource-group"></a>リソース グループの削除

コンテナー用に作成した App Service には、バッキング App Service プランが含まれていて料金が発生する可能性があります。 Visual Studio Code の拡張機能である Azure リソース グループを使用して、リソース グループとグループ内のすべてのリソースを削除します。

1. 一覧でリソース グループ名を探します。
1. リソース グループの名前を右クリックして、 **[削除]** を選択します。

    :::image type="content" source="../../media/visual-studio-code-azure-resources-extension-remove-resource-group.png" alt-text="Visual Studio Code の拡張機能である Azure リソース グループを使用して、リソース グループとグループ内のすべてのリソースを削除します。":::

## <a name="next-steps"></a>次のステップ

[!INCLUDE [tutorial-next-steps](../../includes/tutorial-next-steps.md)]

> [!div class="nextstepaction"]
> [終わりました](../../how-to/deploy-containers.md) [問題が発生しました](https://www.research.net/r/PWZWZ52?tutorial=node-deployment-docker-extension&step=clean-up-resources)