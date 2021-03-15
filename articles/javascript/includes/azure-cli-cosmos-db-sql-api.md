---
ms.custom: devx-track-js
ms.topic: include
ms.date: 02/08/2021
ms.openlocfilehash: 4cca83242756b75161331116a4e0a563e0e0f073
ms.sourcegitcommit: b0a119a624e9cb6b76d968951543a414bd08eaa0
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/09/2021
ms.locfileid: "102118760"
---
## <a name="create-a-cosmos-db-resource-for-sql-api"></a>SQL API の Cosmos DB リソースを作成する

[Azure Cloud Shell](https://shell.azure.com) で次の Azure CLI [az cosmosdb create](/cli/azure/cosmosdb#az_cosmosdb_create) コマンドを使用して、新しい Cosmos DB リソースを作成します。 このコマンドは、完了するまでに数分かかる場合があります。 

```azurecli
az cosmosdb create \
--subscription YOUR-SUBSCRIPTION-ID-OR-NAME \
--resource-group YOUR-RESOURCE-GROUP \
--name YOUR-RESOURCE-NAME \
--kind YOUR-DB-KIND \
--ip-range-filter YOUR-CLIENT-IP
```

ローカル コンピューターから自分のリソースへのファイアウォール アクセスを有効にするには、`123.123.123.123` を自分のクライアント IP に置き換えます。 複数の IP アドレスを構成するには、コンマ区切りのリストを使用します。

[!INCLUDE [Azure CLI command - Cosmos DB Update - firewall IP range](azure-cli-cosmos-db-update-with-firewall.md)]

[!INCLUDE [Azure CLI command - Cosmos DB - get keys](azure-cli-cosmos-db-get-keys.md)]
