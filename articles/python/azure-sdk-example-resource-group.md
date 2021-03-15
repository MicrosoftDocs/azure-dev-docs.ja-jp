---
title: Python 用 Azure ライブラリを使用してリソース グループをプロビジョニングする
description: Azure SDK for Python のリソース管理ライブラリを使用して、Python コードからリソース グループを作成します。
ms.date: 01/28/2021
ms.topic: conceptual
ms.custom: devx-track-python
ms.openlocfilehash: 503ed658fd216edb6308f39b7a721398d6d1fc3a
ms.sourcegitcommit: 576c878c338d286060010646b96f3ad0fdbcb814
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102117806"
---
# <a name="example-use-the-azure-libraries-to-provision-a-resource-group"></a>例:Azure ライブラリを使用してリソース グループをプロビジョニングする

この例では、Python スクリプトで Azure SDK の管理ライブラリを使用してリソース グループをプロビジョニングする方法について説明します。 ([同等の Azure CLI コマンド](#for-reference-equivalent-azure-cli-commands)については、この記事の後半で説明します。 Azure portal を使用する場合は、[リソース グループの作成](/azure/azure-resource-manager/management/manage-resource-groups-portal)に関する記事を参照してください。)

特に記載のない限り、この記事で使用されているコマンドはいずれも、Linux と macOS の bash および Windows のコマンド シェルで同じように動作します。

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

## <a name="3-write-code-to-provision-a-resource-group"></a>3:リソース グループをプロビジョニングするコードを作成する

次のコードを使用して、*provision_rg.py* という名前の Python ファイルを作成します。 詳細はコメントで説明しています。

```python
# Import the needed credential and management objects from the libraries.
from azure.mgmt.resource import ResourceManagementClient
from azure.identity import AzureCliCredential
import os

# Acquire a credential object using CLI-based authentication.
credential = AzureCliCredential()

# Retrieve subscription ID from environment variable.
subscription_id = os.environ["AZURE_SUBSCRIPTION_ID"]

# Obtain the management object for resources.
resource_client = ResourceManagementClient(credential, subscription_id)

# Provision the resource group.
rg_result = resource_client.resource_groups.create_or_update(
    "PythonAzureExample-rg",
    {
        "location": "centralus"
    }
)

# Within the ResourceManagementClient is an object named resource_groups,
# which is of class ResourceGroupsOperations, which contains methods like
# create_or_update.
#
# The second parameter to create_or_update here is technically a ResourceGroup
# object. You can create the object directly using ResourceGroup(location=LOCATION)
# or you can express the object as inline JSON as shown here. For details,
# see Inline JSON pattern for object arguments at
# https://docs.microsoft.com/azure/developer/python/azure-sdk-overview#inline-json-pattern-for-object-arguments.

print(f"Provisioned resource group {rg_result.name} in the {rg_result.location} region")

# The return value is another ResourceGroup object with all the details of the
# new group. In this case the call is synchronous: the resource group has been
# provisioned by the time the call returns.

# Update the resource group with tags
rg_result = resource_client.resource_groups.create_or_update(
    "PythonAzureExample-rg",
    {
        "location": "centralus",
        "tags": { "environment":"test", "department":"tech" }
    }
)

print(f"Updated resource group {rg_result.name} with tags")

# Optional lines to delete the resource group. begin_delete is asynchronous.
# poller = resource_client.resource_groups.begin_delete(rg_result.name)
# result = poller.result()
```

[!INCLUDE [cli-auth-note](includes/cli-auth-note.md)]

### <a name="reference-links-for-classes-used-in-the-code"></a>コードで使用されているクラスの参照リンク

- [AzureCliCredential (azure.identity)](/python/api/azure-identity/azure.identity.azureclicredential)
- [ResourceManagementClient (azure.mgmt.resource)](/python/api/azure-mgmt-resource/azure.mgmt.resource.resourcemanagementclient)

## <a name="4-run-the-script"></a>4:スクリプトを実行する

```cmd
python provision_rg.py
```

## <a name="5-verify-the-resource-group"></a>5:リソース グループを確認する

グループが存在することは、Azure portal または Azure CLI から確認できます。

- Azure portal: [Azure portal](https://portal.azure.com) を開いて **[リソース グループ]** を選択し、対象のグループが一覧に表示されていることを確認します。 既にポータルが開いている場合は、 **[最新の情報に更新]** コマンドを使用してリストを最新の情報に更新します。

- Azure CLI: 次のコマンドを実行します。

    ```azurecli
    az group show -n PythonAzureExample-rg
    ```

## <a name="6-clean-up-resources"></a>6: リソースをクリーンアップする

```azurecli
az group delete -n PythonAzureExample-rg  --no-wait
```

この例でプロビジョニングしたリソース グループを残す必要がなければ、このコマンドを実行してください。 リソース グループの料金がサブスクリプションに課金され続けることはありませんが、実際に使用していないグループは普段からクリーンアップするようにしておきましょう。 `--no-wait` 引数を使用すると、操作が完了するまで待機するのではなく、直ちにコマンドから戻ることができます。

コードから [`ResourceManagementClient.resource_groups.delete`](/python/api/azure-mgmt-resource/azure.mgmt.resource.resources.v2019_10_01.operations.resourcegroupsoperations#delete-resource-group-name--custom-headers-none--raw-false--polling-true----operation-config-) メソッドを使用してリソース グループを削除することもできます。

### <a name="for-reference-equivalent-azure-cli-commands"></a>参考: 同等の Azure CLI コマンド

次の Azure CLI コマンドは、Python スクリプトと同じプロビジョニング手順を実行するものです。

```azurecli
az group create -n PythonAzureExample-rg -l centralus
```

## <a name="see-also"></a>関連項目

- [例:サブスクリプション内のリソース グループを一覧表示する](azure-sdk-example-list-resource-groups.md)
- [例:Azure Storage をプロビジョニングする](azure-sdk-example-storage.md)
- [例:Azure Storage を使用する](azure-sdk-example-storage-use.md)
- [例:Web アプリをプロビジョニングしてコードをデプロイする](azure-sdk-example-web-app.md)
- [例:データベースをプロビジョニングしてクエリを実行する](azure-sdk-example-database.md)
- [例:仮想マシンをプロビジョニングする](azure-sdk-example-virtual-machines.md)
- [仮想マシンで Azure Managed Disks を使用する](azure-sdk-samples-managed-disks.md)
- [Azure SDK for Python に関する簡単なアンケートに回答する](https://microsoft.qualtrics.com/jfe/form/SV_bNFX0HECjzPWMiG?Q_CHL=docs)
