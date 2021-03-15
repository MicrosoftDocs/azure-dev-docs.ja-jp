---
title: Python 用 Azure ライブラリを使用してリソース グループとリソースを一覧表示する
description: Azure SDK for Python のリソース管理ライブラリを使用して、リソース グループと特定のグループ内のリソースを一覧表示します。
ms.date: 01/28/2021
ms.topic: conceptual
ms.custom: devx-track-python
ms.openlocfilehash: ed5ada8332635b6d84a25dcfa70064ae3c9128b1
ms.sourcegitcommit: 576c878c338d286060010646b96f3ad0fdbcb814
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102117816"
---
# <a name="example-use-the-azure-libraries-to-list-resource-groups-and-resources"></a>例:Azure ライブラリを使用してリソース グループとリソースを一覧表示する

この例では、Python スクリプトで Azure SDK の管理ライブラリを使用して、次の 2 つのタスクを実行する方法を示します。

- Azure サブスクリプションに含まれているすべてのリソース グループを一覧表示する。
- 特定のリソース グループ内のリソースを一覧表示する。

特に記載のない限り、この記事で使用されているコマンドはいずれも、Linux と macOS の bash および Windows のコマンド シェルで同じように動作します。

[同等の Azure CLI コマンド](#for-reference-equivalent-azure-cli-commands)については、この記事の後半で説明します。

## <a name="1-set-up-your-local-development-environment"></a>1:ローカルの開発環境を設定する

「[Azure 用のローカル Python 開発環境を構成する](configure-local-development-environment.md)」の手順をすべて実行します (まだ済んでいない場合)。

必ず、このプロジェクトの仮想環境を作成してアクティブにしてください。

## <a name="2-install-the-azure-library-packages"></a>2:Azure ライブラリ パッケージをインストールする

*requirements.txt* という名前のファイルを作成し、内容を次のようにします。

```text
azure-mgmt-resource>=1.15.0
azure-identity>=1.5.0
```

これらのバージョンのライブラリを使用するようにしてください。 以前のバージョンを使用すると、"'AzureCliCredential' オブジェクトに属性 'signed_session' が含まれていません" などのエラーが発生します。

仮想環境をアクティブ化し、ターミナルまたはコマンド プロンプトで要件をインストールします。

```cmd
pip install -r requirements.txt
```



## <a name="3-write-code-to-work-with-resource-groups"></a>3:リソース グループを操作するコードを記述する

### <a name="3a-list-resource-groups-in-a-subscription"></a>3a. サブスクリプション内のリソース グループを一覧表示する

次のコードを使用して、*list_groups.py* という名前の Python ファイルを作成します。 詳細はコメントで説明しています。

```python
# Import the needed credential and management objects from the libraries.
from azure.identity import AzureCliCredential
from azure.mgmt.resource import ResourceManagementClient
import os

# Acquire a credential object using CLI-based authentication.
credential = AzureCliCredential()

# Retrieve subscription ID from environment variable.
subscription_id = os.environ["AZURE_SUBSCRIPTION_ID"]

# Obtain the management object for resources.
resource_client = ResourceManagementClient(credential, subscription_id)

# Retrieve the list of resource groups
group_list = resource_client.resource_groups.list()

# Show the groups in formatted output
column_width = 40

print("Resource Group".ljust(column_width) + "Location")
print("-" * (column_width * 2))

for group in list(group_list):
    print(f"{group.name:<{column_width}}{group.location}")
```

### <a name="3b-list-resources-within-a-specific-resource-group"></a>3b. 特定のリソース グループ内のリソースを一覧表示する

次のコードを使用して、*list_resources.py* という名前の Python ファイルを作成します。 詳細はコメントで説明しています。

```python
# Import the needed credential and management objects from the libraries.
from azure.identity import AzureCliCredential
from azure.mgmt.resource import ResourceManagementClient
import os

# Acquire a credential object using CLI-based authentication.
credential = AzureCliCredential()

# Retrieve subscription ID from environment variable.
subscription_id = os.environ["AZURE_SUBSCRIPTION_ID"]

# Obtain the management object for resources.
resource_client = ResourceManagementClient(credential, subscription_id)

# Retrieve the list of resources in "myResourceGroup" (change to any name desired).
# The expand argument includes additional properties in the output.
resource_list = resource_client.resources.list_by_resource_group(
    "myResourceGroup",
    expand = "createdTime,changedTime"
)

# Show the resources in formatted output
column_width = 36

print("Resource".ljust(column_width) + "Type".ljust(column_width)
    + "Create date".ljust(column_width) + "Change date".ljust(column_width))
print("-" * (column_width * 4))

for resource in list(resource_list):
    print(f"{resource.name:<{column_width}}{resource.type:<{column_width}}"
       f"{str(resource.created_time):<{column_width}}{str(resource.changed_time):<{column_width}}")
```

### <a name="authentication-in-the-code"></a>コード内の認証

[!INCLUDE [cli-auth-note](includes/cli-auth-note.md)]

### <a name="reference-links-for-classes-used-in-the-code"></a>コードで使用されているクラスの参照リンク

- [AzureCliCredential (azure.identity)](/python/api/azure-identity/azure.identity.azureclicredential)
- [ResourceManagementClient (azure.mgmt.resource)](/python/api/azure-mgmt-resource/azure.mgmt.resource.resourcemanagementclient)

## <a name="4-run-the-scripts"></a>4:スクリプトの実行

サブスクリプションに含まれているすべてのリソース グループを一覧表示します。

```cmd
python list_groups.py
```

特定のリソース グループ内のすべてのリソースを一覧表示します。

```cmd
python list_resources.py
```

### <a name="for-reference-equivalent-azure-cli-commands"></a>参考: 同等の Azure CLI コマンド

次の Azure CLI コマンドは、JSON 出力を使用してサブスクリプション内のリソース グループを一覧表示します。

```azurecli
az group list
```

次のコマンドは、centralus リージョンの "myResourceGroup" 内のリソースを一覧表示します (location 引数は、特定のデータ センターを識別するために必要です)。

```azurecli
az resource list --resource group myResourceGroup --location centralus
```

## <a name="see-also"></a>関連項目

- [例:リソース グループをプロビジョニングする](azure-sdk-example-resource-group.md)
- [例:Azure Storage をプロビジョニングする](azure-sdk-example-storage.md)
- [例:Azure Storage を使用する](azure-sdk-example-storage-use.md)
- [例:Web アプリをプロビジョニングしてコードをデプロイする](azure-sdk-example-web-app.md)
- [例:データベースをプロビジョニングしてクエリを実行する](azure-sdk-example-database.md)
- [例:仮想マシンをプロビジョニングする](azure-sdk-example-virtual-machines.md)
- [仮想マシンで Azure Managed Disks を使用する](azure-sdk-samples-managed-disks.md)
- [Azure SDK for Python に関する簡単なアンケートに回答する](https://microsoft.qualtrics.com/jfe/form/SV_bNFX0HECjzPWMiG?Q_CHL=docs)
