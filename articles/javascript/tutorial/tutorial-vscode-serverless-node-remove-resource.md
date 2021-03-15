---
title: コストのかかるリモートの Azure リソースを Azure Functions アプリケーションのデプロイ後に削除する
description: リモートの Azure リソースを削除 (クリーンアップ) して、コストがかからないようにします。 リソースをクリーンアップするには、Azure Functions エクスプローラーで関数アプリを右クリックし、**関数アプリの削除** を選択します。
ms.topic: tutorial
ms.date: 08/31/2020
ms.custom: devx-track-js, contperf-fy21q2
ms.openlocfilehash: cfb5941e6152a2919e3f2b7570d94df66e0c74f9
ms.sourcegitcommit: 576c878c338d286060010646b96f3ad0fdbcb814
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102117786"
---
# <a name="5-clean-up-azure-resources-for-azure-functions-tutorial"></a>5.Azure Functions の Azure リソースをクリーンアップするチュートリアル

[前の手順:Functions アプリをデプロイする](tutorial-vscode-serverless-node-deploy-hosting.md)

## <a name="remove-remote-azure-resources"></a>リモートの Azure リソースを削除する

作成した Functions アプリには、最小限のコストが発生する可能性があるリソースが含まれます ([Functions の価格](https://azure.microsoft.com/pricing/details/functions/)に関するページを参照してください)。 Visual Studio Code の拡張機能である Azure リソース グループを使用して、リソース グループとグループ内のすべてのリソースを削除します。 

1. 一覧でリソース グループ名を探します。
1. リソース グループの名前を右クリックして、 **[削除]** を選択します。

    :::image type="content" source="../media/visual-studio-code-azure-resources-extension-remove-resource-group.png" alt-text="Visual Studio Code の拡張機能である Azure リソース グループを使用して、リソース グループとグループ内のすべてのリソースを削除します。":::

[!INCLUDE [Next steps for using VSCode extensions](../includes/tutorial-next-steps-vscode-extensions.md)]

[!INCLUDE [Next steps for using JavaScript on Azure](../includes/tutorial-next-steps-js-azure.md)]

## <a name="learn-more-about-azure-functions"></a>Azure Functions の詳細について

* [Azure Functions 開発者ガイド](/azure/azure-functions/functions-reference)
* [Azure Functions の JavaScript 開発者向けガイド](/azure/azure-functions/functions-reference-node)
* [Azure Functions のセキュリティ保護](/azure/azure-functions/security-concepts)
* [ストレージ](/azure/azure-functions/storage-considerations)と[パフォーマンス](/azure/azure-functions/functions-best-practices)の考慮事項

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [終わりました](../how-to/develop-serverless-apps.md) [問題が発生しました](https://www.research.net/r/PWZWZ52?tutorial=node-deployment-azurefunctions&step=clean-up-resources)
