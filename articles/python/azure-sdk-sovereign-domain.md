---
title: Python 用 Azure ライブラリを使用してすべてのリージョンに接続する - マルチクラウド
description: msrestazure の azure_cloud モジュールを使用して、さまざまなソブリン リージョンで Azure に接続する方法
ms.date: 11/18/2020
ms.topic: conceptual
ms.custom: devx-track-python
ms.openlocfilehash: ba751604351bd7038a7696b6c8e93d663aa9263f
ms.sourcegitcommit: 29930f1593563c5e968b86117945c3452bdefac1
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/23/2020
ms.locfileid: "95485671"
---
# <a name="multi-cloud-connect-to-all-regions-with-the-azure-libraries-for-python"></a>マルチクラウド:Python 用 Azure ライブラリを使用してすべてのリージョンに接続する

Python 用 Azure ライブラリを使用して、Azure が[利用可能な](https://azure.microsoft.com/regions/services)すべてのリージョンに接続できます。

既定では、Azure ライブラリはグローバル Azure クラウドに接続するように構成されています。

## <a name="using-pre-defined-sovereign-cloud-constants"></a>ソブリン クラウドの定義済み定数の使用

ソブリン クラウドの定義済み定数は、`msrestazure` ライブラリ (0.4.11+) の `azure_cloud` モジュールによって提供されます。

- `AZURE_PUBLIC_CLOUD`
- `AZURE_CHINA_CLOUD`
- `AZURE_US_GOV_CLOUD`
- `AZURE_GERMAN_CLOUD`

定義を使用するには、`msrestazure.azure_cloud` から適切な定数をインポートし、クライアント オブジェクトを作成するときにこれを適用します。 

`DefaultAzureCredential` を使用する場合は、次の例に示すように、`azure.identity.AzureAuthorityHosts` から適切な値を使用する必要もあります。

```python
import os
from msrestazure.azure_cloud import AZURE_CHINA_CLOUD as cloud
from azure.mgmt.resource import ResourceManagementClient, SubscriptionClient
from azure.identity import DefaultAzureCredential

# Assumes the subscription ID to use is in the AZURE_SUBSCRIPTION_ID environment variable
subscription_id = os.environ["AZURE_SUBSCRIPTION_ID"]

# When using sovereign domains (that is, any cloud other than AZURE_PUBLIC_CLOUD),
# you must use an authority with DefaultAzureCredential.
credential = DefaultAzureCredential(authority=cloud.endpoints.active_directory)

resource_client = ResourceManagementClient(credential,
    subscription_id, base_url=cloud.endpoints.resource_manager,
    credential_scopes=[cloud.endpoints.resource_manager + ".default'"])

subscription_client = SubscriptionClient(credential,
    base_url=stack_cloud.endpoints.resource_manager,
    credential_scopes=[cloud.endpoints.resource_manager + ".default'"])
```
  
## <a name="using-your-own-cloud-definition"></a>独自のクラウド定義の使用

次のコードでは、プライベート クラウド (たとえば、Azure Stack 上に構築されたもの) の Azure Resource Manager エンドポイントで `get_cloud_from_metadata_endpoint` を使用します。

```python
import os
from msrestazure.azure_cloud import get_cloud_from_metadata_endpoint
from azure.mgmt.resource import ResourceManagementClient, SubscriptionClient
from azure.identity import DefaultAzureCredential

# Assumes the subscription ID to use is in the AZURE_SUBSCRIPTION_ID environment variable
subscription_id = os.environ["AZURE_SUBSCRIPTION_ID"]

stack_cloud = get_cloud_from_metadata_endpoint("https://contoso-azurestack-arm-endpoint.com")

# When using a private, you must use an authority with DefaultAzureCredential.
# The active_directory endpoint should be a URL like https://login.microsoftonline.com.
# https:// is optional in the URL but required on the endpoint.
credential = DefaultAzureCredential(authority=stack_cloud.endpoints.active_directory)

resource_client = ResourceManagementClient(credential, subscription_id,
    base_url=stack_cloud.endpoints.resource_manager,
    credential_scopes=[cloud.endpoints.resource_manager + ".default'"])

subscription_client = SubscriptionClient(credential,
    base_url=stack_cloud.endpoints.resource_manager,
    credential_scopes=[stack_cloud.endpoints.resource_manager + ".default'"])
```
