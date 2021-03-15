---
ms.custom: devx-track-js
ms.topic: include
ms.date: 02/08/2021
ms.openlocfilehash: 8353ffea6f141b2bf25c726bfadd7f8926e552ce
ms.sourcegitcommit: b0a119a624e9cb6b76d968951543a414bd08eaa0
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/09/2021
ms.locfileid: "102118751"
---
## <a name="add-firewall-rule-for-your-client-ip-address"></a>クライアント IP アドレスのファイアウォール規則を追加する

リソースを作成した後に、それにクライアント IP アドレスを追加して、JavaScript によるサーバーへのクライアント接続が正常に行われるようにするには、この手順を使用します。 ファイアウォール規則を更新するには、[az cosmosdb update](/cli/azure/cosmosdb#az_cosmosdb_update) コマンドを使用します。


```azurecli
az cosmosdb update \
    --subscription YOUR-SUBSCRIPTION-ID-OR-NAME \
    --resource-group YOUR-RESOURCE-GROUP \
    --name YOUR-RESOURCE_NAME \
    --ip-range-filter 123.123.123.123
```

複数の IP アドレスを構成するには、コンマ区切りのリストを使用します。

```azurecli
az cosmosdb update \
    --subscription YOUR-SUBSCRIPTION-ID-OR-NAME \
    --resource-group YOUR-RESOURCE-GROUP \
    --name YOUR-RESOURCE_NAME \
    --ip-range-filter 123.123.123.123,456.456.456.456
```