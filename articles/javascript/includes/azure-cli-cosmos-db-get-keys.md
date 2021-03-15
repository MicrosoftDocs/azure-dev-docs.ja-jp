---
ms.custom: devx-track-js
ms.topic: include
ms.date: 02/08/2021
ms.openlocfilehash: e3a7f10e6db4a805c136e0d04289da5333e77bc7
ms.sourcegitcommit: b0a119a624e9cb6b76d968951543a414bd08eaa0
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/09/2021
ms.locfileid: "102118754"
---
## <a name="get-the-cosmos-db-keys-for-your-resource"></a>自分のリソースの Cosmos DB キーを取得する

[az cosmosdb keys list](/cli/azure/cosmosdb/keys#az_cosmosdb_keys_list) コマンドを使用して、このインスタンスのキーを取得します。

```azurecli
az cosmosdb keys list \
    --subscription YOUR-SUBSCRIPTION-ID-OR-NAME \
    --resource-group YOUR-RESOURCE-GROUP \
    --name YOUR-RESOURCE-NAME
```

これにより、4 つのキー (2 つの読み取り/書き込みと 2 つの読み取り専用) が返されます。 2 つずつあるため、2 つの異なるシステムまたは開発者に対して、個別に使用およびリサイクルするキーを提供できます。 