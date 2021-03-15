---
title: Azure サービスを使用して Python アプリケーションを認証する方法
description: Azure ライブラリを使用して、Azure サービスで Python アプリを認証するために必要な資格情報オブジェクトを取得する方法
ms.date: 01/19/2021
ms.topic: conceptual
ms.custom: devx-track-python
ms.openlocfilehash: 9a33dc8c98655f78e7e433081655d96137d9d8d8
ms.sourcegitcommit: 576c878c338d286060010646b96f3ad0fdbcb814
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102117846"
---
# <a name="how-to-authenticate-and-authorize-python-apps-on-azure"></a>Azure で Python アプリを認証および認可する方法

Azure にデプロイされたほとんどのクラウド アプリケーションは、ストレージ、データベース、格納されているシークレットなど、他の Azure リソースにアクセスする必要があります。 これらのリソースにアクセスするには、アプリケーションは認証され、認可される必要があります。

- **認証** では、Azure Active Directory を使用してアプリの ID が確認されます。

- **認可** では、認証されたアプリが特定のリソースに対して実行できる操作が決定されます。 認可された操作は、そのリソースのアプリ ID に割り当てられた **ロール** によって定義されています。 Azure Key Vault などのいくつかのケースでは、認可は、アプリ ID に割り当てられた追加の **アクセス ポリシー** によっても決定されます。

この記事では、認証と認可の詳細について説明します。

- アプリ ID を割り当てる方法
- ID にアクセス許可を付与する方法
- 認証と認可が行われる方法とタイミング
- アプリが Azure ライブラリを使用して Azure で認証を行うさまざまな方法。 `DefaultAzureCredential` を使用することをお勧めしますが、必須ではありません。

## <a name="how-to-assign-an-app-identity"></a>アプリ ID を割り当てる方法

Azure では、アプリ ID は **サービス プリンシパル** によって定義されます。 (サービス プリンシパルとは、人間のユーザーやユーザー グループではなく、いわば 1 つのコードであるアプリやサービスを識別するために使用される特定の種類の "セキュリティ プリンシパル" です。)

使用されるサービス プリンシパルは、次のセクションで説明するように、アプリが実行されている場所によって異なります。

### <a name="identity-when-running-the-app-on-azure"></a>Azure でアプリを実行しているときの ID

クラウド (たとえば、運用環境) で実行している場合、アプリでは、**システムによって割り当てられたマネージド ID** (以前は "MSI" と呼ばれていました) が使用されるのが最も一般的です。 [マネージド ID](/azure/active-directory/managed-identities-azure-resources/overview) では、リソースのロールとアクセス許可を割り当てるときにアプリの名前を使用します。 Azure は、基になるサービス プリンシパルを自動的に管理し、他の Azure リソースで自動的にアプリを認証します。 そのため、サービス プリンシパルを直接処理する必要はありません。 さらに、アプリ コードでは、Azure リソースのアクセス トークン、シークレット、または接続文字列を処理する必要はありません。そのため、このような情報が漏洩したり侵害されたりするリスクが軽減されます。

マネージド ID の構成は、アプリをホストするために使用するサービスによって異なります。 各サービスの手順へのリンクについては、[マネージド ID をサポートするサービス](/azure/active-directory/managed-identities-azure-resources/services-support-managed-identities)に関する記事を参照してください。 たとえば、Azure App Service にデプロイされた Web アプリの場合、マネージド ID を有効にするには、Azure portal の **[ID]**  >  **[システム割り当て済み]** オプションを使用するか、Azure CLI で `az webapp identity assign` コマンドを使用します。

マネージド ID を使用できない場合は、代わりに Azure Active Directory にアプリケーションを手動で登録します。 登録すると、サービス プリンシパルがアプリに割り当てられます。これは、ロールとアクセス許可を割り当てるときに使用します。 詳細については、[アプリケーションの登録](/azure/active-directory/develop/quickstart-register-app)に関するトピックを参照してください。

### <a name="identity-when-running-the-app-locally"></a>ローカルでアプリを実行しているときの ID

開発中に、クラウド内の Azure リソースにアプリのコードがアクセスできるようにした状態のまま、開発者ワークステーションでそのコードを実行したりデバッグしたりしたいことがよくあります。 この場合は、Azure Active Directory を使用して、ローカル開発専用の別個のサービス プリンシパルを作成します。 このサービス プリンシパルに、対象のリソースのロールとアクセス許可を再び割り当てます。 通常、この開発 ID には、非運用リソースにのみアクセスすることを認可します。

ローカル サービス プリンシパルを作成し、Azure ライブラリで使用できるようにする方法の詳細については、[ローカル開発環境の構成](configure-local-development-environment.md)に関する記事を参照してください。 この 1 回限りの構成を完了すると、環境に固有の変更を加えなくても、ローカルとクラウドで同じアプリ コードを実行できます。

それぞれの開発者は、ワークステーションのユーザー アカウント内でセキュリティ保護され、ソース管理リポジトリに保存されることのない独自のサービス プリンシパルを持つ必要があります。 サービス プリンシパルが盗まれたり侵害されたりした場合は、簡単にそれを削除してそのすべてのアクセス許可を取り消した後、その開発者のサービス プリンシパルを再作成できます。 詳細については、「[サービス プリンシパルを管理する方法](how-to-manage-service-principals.md)」を参照してください。

> [!NOTE]
> 独自の Azure ユーザー資格情報を使用してアプリを実行することはできますが、それによって、クラウドにデプロイするときにアプリに必要な特定のリソース アクセス許可を確立することはできません。 開発用のサービス プリンシパルを設定して、必要なロールとアクセス許可を割り当てることをお勧めします。これは、デプロイされたアプリのマネージド ID またはサービス プリンシパルを使用してレプリケートできます。

## <a name="assign-roles-and-permissions-to-an-identity"></a>ロールとアクセス許可を ID に割り当てる

Azure 上とローカルで実行する際の両方のアプリ ID を把握したら、ロールベースのアクセス制御 (RBAC) を使用して、Azure portal または Azure CLI を使用してアクセス許可を付与します。 詳細については、「[アプリ ID またはサービス プリンシパルにロールのアクセス許可を割り当てる方法](/azure/role-based-access-control/role-assignments-steps)」を参照してください。

## <a name="when-does-authentication-and-authorization-occur"></a>認証と認可が行われるタイミング

Python 用 Azure ライブラリ (SDK) を使用してアプリ コードを記述するときは、次のパターンを使用して Azure リソースにアクセスします。

1. この記事の後半で説明するいずれかの方法を使用して、アプリ ID を記述する資格情報を取得します。

1. その資格情報を使用して、目的のリソースのクライアント オブジェクトを取得します。 (リソースの種類ごとに、Azure ライブラリに独自のクライアント オブジェクトがあり、それに対してリソースの URL を指定します。)

1. そのクライアント オブジェクトを介してリソースにアクセスしたり変更を加えたりしようとすると、リソースの REST API に対する HTTP 要求が生成されます。 API 呼び出しが行われたときに、Azure ではアプリ ID の認証と認可の確認の両方を実行します。

次のコードでは、Azure Key Vault へのアクセスを試行することによって、これらの手順を説明して示します。

```python
import os
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient

# Acquire the resource URL. In this code we assume the resource URL is in an
# environment variable, KEY_VAULT_URL in this case.

vault_url = os.environ["KEY_VAULT_URL"]


# Acquire a credential object for the app identity. When running in the cloud,
# DefaultAzureCredential uses the app's managed identity (MSI) or user-assigned service principal.
# When run locally, DefaultAzureCredential relies on environment variables named
# AZURE_CLIENT_ID, AZURE_CLIENT_SECRET, and AZURE_TENANT_ID.

credential = DefaultAzureCredential()


# Acquire an appropriate client object for the resource identified by the URL. The
# client object only stores the given credential at this point but does not attempt
# to authenticate it.
#
# **NOTE**: SecretClient here is only an example; the same process applies to all
# other Azure client libraries.

secret_client = SecretClient(vault_url=vault_url, credential=credential)

# Attempt to perform an operation on the resource using the client object (in
# this case, retrieve a secret from Key Vault). The operation fails for any of
# the following reasons:
#
# 1. The information in the credential object is invalid (for example, the AZURE_CLIENT_ID
#    environment variable cannot be found).
# 2. The app identity cannot be authenticated using the information in the credential object.
# 3. The app identity is not authorized to perform the requested operation on the
#    resource (identified in this case by the vault_url.

retrieved_secret = secret_client.get_secret("secret-name-01")
```

ここでも、クライアント オブジェクトを介して Azure REST API への特定の要求がコードによって行われるまでは、認証または認可は行われません。 `DefaultAzureCredential` を作成するステートメント (次のセクションを参照) は、クライアント側のオブジェクトをメモリ内に作成するだけで、他のチェックは実行しません。

SDK [`SecretClient`](/python/api/azure-keyvault-secrets/azure.keyvault.secrets.secretclient) オブジェクトを作成する場合も、対象のリソースとの通信は必要ありません。 `SecretClient` オブジェクトは、基になる Azure REST API の単なるラッパーであり、アプリのランタイム メモリにのみ存在します。 

コードによって [`get_secret`](/python/api/azure-keyvault-secrets/azure.keyvault.secrets.secretclient#get-secret-name--version-none----kwargs-) メソッドが呼び出されたときに初めて、クライアント オブジェクトは Azure への適切な REST API 呼び出しを生成します。 その後、`get_secret` に対応する Azure のエンドポイントで、呼び出し元の ID が認証され、承認がチェックされます。

## <a name="authenticate-with-defaultazurecredential"></a>DefaultAzureCredential を使用して認証する

ほとんどのアプリケーションでは、[`azure-identity`](/python/api/azure-identity/azure.identity) ライブラリの [`DefaultAzureCredential`](/python/api/azure-identity/azure.identity.defaultazurecredential) クラスには、最も単純で推奨される認証方法が用意されています。

`DefaultAzureCredential` は、クラウドでは自動的にアプリのマネージド ID (MSI) を使用し、ローカルで実行しているときには自動的に環境変数からローカル サービス プリンシパルを読み込みます (「[Azure 用のローカル Python 開発環境を構成する - 認証を構成する](configure-local-development-environment.md#configure-authentication)」を参照してください)。

```python
import os
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient

# Acquire the resource URL
vault_url = os.environ["KEY_VAULT_URL"]

# Acquire a credential object
credential = DefaultAzureCredential()

# Acquire a client object
secret_client = SecretClient(vault_url=vault_url, credential=credential)

# Attempt to perform an operation
retrieved_secret = secret_client.get_secret("secret-name-01")
```

前述のコードでは、Azure Key Vault にアクセスするときに `DefaultAzureCredential` が使用されています。この場合、Key Vault の URL は `KEY_VAULT_URL` という名前の環境変数で提供されています。 このコードでは明らかに、一般的なライブラリの使用パターン (資格情報オブジェクトを取得し、Azure リソースの適切なクライアント オブジェクトを作成した後、そのクライアント オブジェクトを使用してそのリソースに対して操作を実行する) が実装されています。 ここでも、認証と認可はこの最後のステップで初めて実行されます。

コードを Azure にデプロイして実行すると、`DefaultAzureCredential` はシステムによって割り当てられたマネージド ID (MSI) を自動的に使用します。これは、そのホストとなるあらゆるサービス内のアプリに対して有効にできます。 Azure Storage や Azure Key Vault などの特定のリソースに対するアクセス許可は、Azure portal または Azure CLI を使用してその ID に割り当てられます。 これらのケースでは、Azure によって管理されるこのマネージド ID によって最大限のセキュリティが確保されます。明示的なサービス プリンシパルをコード内で一切扱う必要がないためです。

コードをローカルで実行した場合は、`AZURE_TENANT_ID`、`AZURE_CLIENT_ID`、`AZURE_CLIENT_SECRET` という名前の各環境変数によって表されるサービス プリンシパルが `DefaultAzureCredential` によって自動的に使用されます。 その後、クライアント オブジェクトは、API エンドポイントを呼び出すときに、これらの値を (安全に) HTTP 要求ヘッダーに含めます。 ローカルまたはクラウドで実行するときに、コードの変更は必要ありません。 サービス プリンシパルの作成と環境変数の設定について詳しくは、「[Azure 用のローカル Python 開発環境を構成する - 認証を構成する](configure-local-development-environment.md#configure-authentication)」を参照してください。

どちらの場合も、関連する ID には、適切なリソースのアクセス許可が割り当てられている必要があります。 一般的なプロセスについては、[ロールのアクセス許可を割り当てる方法](/azure/role-based-access-control/role-assignments-steps)に関する記事を参照してください。詳細については、個々のサービスのドキュメントをご覧ください。 たとえば、前のコードで必要となる Key Vault のアクセス許可の詳細については、「[アクセス制御ポリシーを使用して Key Vault の認証を提供する](/azure/key-vault/general/group-permissions-for-apps)」を参照してください。

### <a name="using-defaultazurecredential-with-sdk-management-libraries"></a>SDK 管理ライブラリで DefaultAzureCredential を使用する

`DefaultAzureCredential` は、[azure.core を使用するライブラリ](azure-sdk-library-package-index.md#libraries-using-azurecore)の一覧にも記載されている Azure SDK 管理ライブラリ (つまり、名前に "mgmt" が付いている "リソース管理" ライブラリ) の新しいバージョンと連携して動作します。 (また、更新されたライブラリの pypi ページには、通常、変更を示す "Credential system has been completely revamped (資格情報システムが完全に改良された)" という行が含まれています。)

たとえば、バージョン 15.0.0 以上の azure-mgmt-resource で `DefaultAzureCredential` を使用できます。

```python
from azure.identity import DefaultAzureCredential
from azure.mgmt.resource import SubscriptionClient

credential = DefaultAzureCredential()
subscription_client = SubscriptionClient(credential)

sub_list = subscription_client.subscriptions.list()
print(list(sub_list))
```

ライブラリが更新されていない場合、次のセクションで説明するように、`DefaultAzureCredential` を使用したコードを実行すると、「object has no attribute 'signed-session' (オブジェクトに属性 'signed-session' がありません)」というメッセージが表示されます。

### <a name="credential-object-has-no-attribute-signed_session"></a>資格情報オブジェクトに属性 'signed-session' がない

azure.core を使用するように更新されていないライブラリで `DefaultAzureCredential` (または `AzureCliCredential` と azure.identity からのその他の資格情報オブジェクト) を使用しようとした場合、クライアント オブジェクトを通じた呼び出しは、「'DefaultAzureCredential' object has no attribute 'signed-session' ('DefaultAzureCredential' オブジェクトに属性 'signed-session' がありません)」というあいまいなエラーで失敗します。 このようなエラーが発生するのは、たとえば、前のセクションでバージョン 15 より下の azure-mgmt-resource ライブラリを利用するコードを使用した場合です。

このエラーが発生するのは、SDK 管理ライブラリの azure.core 以外のバージョンでは、資格情報オブジェクトに `signed_session` プロパティが含まれていると想定されているためです。これは、`DefaultAzureCredential` および azure.identity のその他の資格情報オブジェクトには存在しません。

使用する管理ライブラリがまだ更新されていない場合は、次の代替方法を使用できます。

1. この記事の後続のセクションで取り上げる他のいずれかの認証方法を使用します。SDK 管理ライブラリ "*のみ*" を使用し、なおかつクラウドにデプロイされないコードであれば、これらの認証方法がうまく機能します。この場合、利用できるのはローカルのサービス プリンシパルのみです。

1. Azure SDK エンジニアリング チームのメンバーから提供されている [CredentialWrapper クラス (cred_wrapper.py)](https://gist.github.com/lmazuel/cc683d82ea1d7b40208de7c9fc8de59d) を `DefaultAzureCredential` の代わりに使用します。 目的の管理ライブラリが使用可能になったら、`DefaultAzureCredential` に切り替えます。 この方法の利点は、SDK クライアントと管理ライブラリの両方で同じ資格情報を使用できる点、またローカルでもクラウドでも使用できる点です。

    プロジェクト フォルダーに *cred_wrapper.py* のコピーがダウンロード済みである場合、先ほどのコードは次のようになります。

    ```python
    from cred_wrapper import CredentialWrapper
    from azure.mgmt.resource import SubscriptionClient

    credential = CredentialWrapper()
    subscription_client = SubscriptionClient(credential)
    subscription = next(subscription_client.subscriptions.list())
    print(subscription.subscription_id)
    ```

    ここでも、更新された管理ライブラリが使用可能になったら、元のコード例に示すように `DefaultAzureCredential` を直接使用できます。

## <a name="other-authentication-methods"></a>その他の認証方法

`DefaultAzureCredential` は、ほとんどのシナリオで推奨される認証方法ですが、次の注意事項の下で他の方法も利用できます。

- それらの方法の大半は、明示的なサービス プリンシパルで動作し、クラウドにデプロイされたコードでマネージド ID の利点が活かされません。 運用環境のコードで使用する場合は、対象のクラウド アプリケーション用に別個のサービス プリンシパルを自分で管理、保守する必要があります。

- CLI ベースの認証など一部の方法はローカル スクリプトでしか機能せず、運用環境のコードでは使用できません。 CLI ベースの認証では、Azure ログインのアクセス許可が使用され、明示的なロールの割り当てを必要としないため、開発作業の際に便利です。

クラウドにデプロイされたアプリケーションのサービス プリンシパルは、ご利用のサブスクリプションの Active Directory で管理されます。 詳細については、「[サービス プリンシパルを管理する方法](how-to-manage-service-principals.md)」を参照してください。

いずれの場合も、適切なサービス プリンシパルまたはユーザーが、該当するリソースと操作に対する適切なアクセス許可を持っている必要があります。

### <a name="authenticate-with-a-json-file"></a>JSON ファイルを使用して認証する

この方法では、サービス プリンシパルに必要な資格情報を含む JSON ファイルを作成します。 そのファイルを使用して SDK クライアント オブジェクトを作成します。 この方法は、ローカルでもクラウドでも使用できます。

1. 次の形式の JSON ファイルを作成します。

    ```json
    {
        "subscriptionId": "<azure_aubscription_id>",
        "tenantId": "<tenant_id>",
        "clientId": "<application_id>",
        "clientSecret": "<application_secret>",
        "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
        "resourceManagerEndpointUrl": "https://management.azure.com/",
        "activeDirectoryGraphResourceId": "https://graph.windows.net/",
        "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
        "galleryEndpointUrl": "https://gallery.azure.com/",
        "managementEndpointUrl": "https://management.core.windows.net/"
    }
    ```

    4 つのプレースホルダーは、Azure サブスクリプション ID、テナント ID、クライアント ID、クライアント シークレットに置き換えてください。

    > [!TIP]
    > [ローカル開発環境の構成](configure-local-development-environment.md#create-a-service-principal-and-environment-variables-for-development)に関するセクションで説明されているように、[az ad sp create-for-rbac](/cli/azure/ad/sp#az-ad-sp-create-for-rbac) コマンドに `--sdk-auth` パラメーターを指定することで、この JSON 形式をすぐに生成できます。

1. ファイルに名前 (*credentials.json* など) を付けて、コードからアクセスできる安全な場所に保存します。 資格情報の安全性を確保するため、このファイルはソース管理から除外し、他の開発者と共有しないようにしてください。 つまり、サービス プリンシパルのテナント ID、クライアント ID、クライアント シークレットは、常に自分の開発ワークステーションで隔離されている必要があります。

1. JSON ファイルのパスを値とする `AZURE_AUTH_LOCATION` という名前の環境変数を作成します。

    # <a name="cmd"></a>[cmd](#tab/cmd)

    ```cmd
    set AZURE_AUTH_LOCATION=../credentials.json
    ```

    # <a name="bash"></a>[bash](#tab/bash)

    ```bash
    AZURE_AUTH_LOCATION="../credentials.json"
    ```

    ---

    この例では、JSON ファイルが *credentials.json* という名前でプロジェクトの親フォルダーに格納されていることを想定しています。

1. [get_client_from_auth_file](/python/api/azure-common/azure.common.client_factory#get-client-from-auth-file-client-class--auth-path-none----kwargs-) メソッドを使用してクライアント オブジェクトを作成します。

    ```python
    from azure.common.client_factory import get_client_from_auth_file
    from azure.mgmt.resource import SubscriptionClient

    # This form of get_client_from_auth_file relies on the AZURE_AUTH_LOCATION
    # environment variable.
    subscription_client = get_client_from_auth_file(SubscriptionClient)

    subscription = next(subscription_client.subscriptions.list())
    print(subscription.subscription_id)
    ```

または、`auth_path` 引数を使用してコード内で直接パスを指定してもかまいません。その場合、環境変数は不要です。

```python
subscription_client = get_client_from_auth_file(SubscriptionClient, auth_path="../credentials.json")
```

### <a name="authenticate-with-a-json-dictionary"></a>JSON ディクショナリを使用した認証

```python
import os
from azure.common.client_factory import get_client_from_json_dict
from azure.mgmt.resource import SubscriptionClient

# Retrieve the IDs and secret to use in the JSON dictionary
subscription_id = os.environ["AZURE_SUBSCRIPTION_ID"]
tenant_id = os.environ["AZURE_TENANT_ID"]
client_id = os.environ["AZURE_CLIENT_ID"]
client_secret = os.environ["AZURE_CLIENT_SECRET"]

config_dict = {
   "subscriptionId": subscription_id,
    "tenantId": tenant_id,
   "clientId": client_id,
   "clientSecret": client_secret,
   "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
   "resourceManagerEndpointUrl": "https://management.azure.com/",
   "activeDirectoryGraphResourceId": "https://graph.windows.net/",
   "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
   "galleryEndpointUrl": "https://gallery.azure.com/",
   "managementEndpointUrl": "https://management.core.windows.net/"
}

subscription_client = get_client_from_json_dict(SubscriptionClient, config_dict)

subscription = next(subscription_client.subscriptions.list())
print(subscription.subscription_id)
```

前のセクションで説明したように、ファイルを使用する代わりに、必要な JSON データを変数に作成し、[get_client_from_json_dict](/python/api/azure-common/azure.common.client_factory#get-client-from-json-dict-client-class--config-dict----kwargs-) を呼び出すことができます。 このコードは、[ローカル開発環境の構成](configure-local-development-environment.md#create-a-service-principal-and-environment-variables-for-development)に関するセクションで説明されている環境変数が作成済みであることを想定しています。 クラウドにデプロイされるコードの場合、これらの環境変数をサーバー VM 上に作成できるほか、Azure App Service や Azure Functions などのプラットフォーム サービスを使用しているときは、アプリケーション設定として作成することができます。

または、環境変数を使わずに、値を Azure Key Vault に格納して実行時に取得してもかまいません。

### <a name="authenticate-with-token-credentials"></a>トークン資格情報で認証を行う

明示的なサブスクリプション、テナント、およびクライアントの識別子とクライアント シークレットを使用して、Azure ライブラリで認証を行うことができます。

azure.core に基づく新しい SDK ライブラリを使用する場合は、[azure.identity ライブラリの `ClientSecretCredential` オブジェクト](#using-clientsecretcredential-azureidentity)を使用します。 以前の SDK ライブラリを使用する場合は、[azure.common ライブラリの `ServicePrincipalCredentials`](#using-serviceprincipalcredentials-azurecommon) を使用します。

`ServicePrincipalCredentials` を使用する既存のコードを新しいライブラリ バージョンに移行するには、次のセクションに示すように、このクラスを使用している部分を `ClientSecretCredential` に置き換えます。 パラメーター名が 2 つのコンストラクターでわずかに異なることに注意してください。`tenant` が `tenant_id` に、`secret` が `client_secret` になっています。

#### <a name="using-clientsecretcredential-azureidentity"></a>ClientSecretCredential (azure.identity) を使用する

```python
import os
from azure.mgmt.resource import SubscriptionClient
from azure.identity import ClientSecretCredential

# Retrieve the IDs and secret to use with ClientSecretCredential
subscription_id = os.environ["AZURE_SUBSCRIPTION_ID"]
tenant_id = os.environ["AZURE_TENANT_ID"]
client_id = os.environ["AZURE_CLIENT_ID"]
client_secret = os.environ["AZURE_CLIENT_SECRET"]

credential = ClientSecretCredential(tenant_id=tenant_id, client_id=client_id, client_secret=client_secret)

subscription_client = SubscriptionClient(credential)

subscription = next(subscription_client.subscriptions.list())
print(subscription.subscription_id)
```

この方法 (前述のとおり、azure.core に基づく新しいライブラリで使用されます) では、Azure Key Vault や環境変数など、安全な記憶域から取得した資格情報を使用して [`ClientSecretCredential`](/python/api/azure-identity/azure.identity.clientsecretcredential) オブジェクトを作成します。 前のコードは、[ローカル開発環境の構成](configure-local-development-environment.md#create-a-service-principal-and-environment-variables-for-development)に関するセクションで説明されている環境変数が作成済みであることを想定しています。

#### <a name="using-serviceprincipalcredentials-azurecommon"></a>ServicePrincipalCredentials (azure.common) を使用する

```python
import os
from azure.mgmt.resource import SubscriptionClient
from azure.common.credentials import ServicePrincipalCredentials

# Retrieve the IDs and secret to use with ServicePrincipalCredentials
subscription_id = os.environ["AZURE_SUBSCRIPTION_ID"]
tenant_id = os.environ["AZURE_TENANT_ID"]
client_id = os.environ["AZURE_CLIENT_ID"]
client_secret = os.environ["AZURE_CLIENT_SECRET"]

credential = ServicePrincipalCredentials(tenant=tenant_id, client_id=client_id, secret=client_secret)

subscription_client = SubscriptionClient(credential)

subscription = next(subscription_client.subscriptions.list())
print(subscription.subscription_id)
```

この方法 (前述のとおり、azure.core に基づいていない以前のライブラリで使用されます) では、Azure Key Vault や環境変数など、安全な記憶域から取得した資格情報を使用して [`ServicePrincipalCredentials`](/python/api/msrestazure/msrestazure.azure_active_directory.serviceprincipalcredentials) オブジェクトを作成します。 前のコードは、[ローカル開発環境の構成](configure-local-development-environment.md#create-a-service-principal-and-environment-variables-for-development)に関するセクションで説明されている環境変数が作成済みであることを想定しています。

#### <a name="use-an-azure-sovereign-national-cloud"></a>Azure ソブリン (国内) クラウド を使用する

これらのトークン資格情報の方法のいずれかを使用する場合は、クライアント オブジェクトに `base_url` 引数を指定することで、Azure パブリック クラウドではなく [Azure ソブリン クラウド (国内クラウド)](/azure/active-directory/develop/authentication-national-cloud) を使用できます。

```python
from msrestazure.azure_cloud import AZURE_CHINA_CLOUD

#...

subscription_client = SubscriptionClient(credentials, base_url=AZURE_CHINA_CLOUD.endpoints.resource_manager)
```

ソブリン クラウドの定数は、[msrestazure.azure_cloud ライブラリ](https://github.com/Azure/msrestazure-for-python/blob/master/msrestazure/azure_cloud.py)にあります。

### <a name="authenticate-with-token-credentials-and-an-adal-context"></a>トークン資格情報と ADAL コンテキストを使用して認証する

トークン資格情報を使用して、より細かく制御する必要がある場合は、[Azure Active Directory Authentication Library (ADAL) for Python](https://github.com/AzureAD/azure-activedirectory-library-for-python) と SDK ADAL ラッパーを使用します。

```python
import os, adal
from azure.mgmt.resource import SubscriptionClient
from msrestazure.azure_active_directory import AdalAuthentication
from msrestazure.azure_cloud import AZURE_PUBLIC_CLOUD

# Retrieve the IDs and secret to use with ServicePrincipalCredentials
subscription_id = os.environ["AZURE_SUBSCRIPTION_ID"]
tenant_id = os.environ["AZURE_TENANT_ID"]
client_id = os.environ["AZURE_CLIENT_ID"]
client_secret = os.environ["AZURE_CLIENT_SECRET"]

LOGIN_ENDPOINT = AZURE_PUBLIC_CLOUD.endpoints.active_directory
RESOURCE = AZURE_PUBLIC_CLOUD.endpoints.active_directory_resource_id

context = adal.AuthenticationContext(LOGIN_ENDPOINT + '/' + tenant_id)

credential = AdalAuthentication(context.acquire_token_with_client_credentials,
    RESOURCE, client_id, client_secret)

subscription_client = SubscriptionClient(credential)

subscription = next(subscription_client.subscriptions.list())
print(subscription.subscription_id)
```

ADAL ライブラリが必要な場合は、`pip install adal` を実行します。

この方法を使用した場合は、Azure パブリック クラウドではなく [Azure ソブリン クラウド (国内クラウド)](/azure/active-directory/develop/authentication-national-cloud) を使用できます。

```python
from msrestazure.azure_cloud import AZURE_CHINA_CLOUD

# ...

LOGIN_ENDPOINT = AZURE_CHINA_CLOUD.endpoints.active_directory
RESOURCE = AZURE_CHINA_CLOUD.endpoints.active_directory_resource_id
```

`AZURE_PUBLIC_CLOUD` を、[msrestazure.azure_cloud ライブラリ](https://github.com/Azure/msrestazure-for-python/blob/master/msrestazure/azure_cloud.py)にある適切なソブリン クラウド定数に置き換えるだけです。

### <a name="cli-based-authentication-development-purposes-only"></a>CLI ベースの認証 (開発目的のみ)

この方法では、Azure CLI コマンド `az login` でサインインしているユーザーの資格情報を使用してクライアント オブジェクトを作成します。 CLI ベースの認証は、運用環境では使用できないため、開発目的でのみ機能します。

Azure ライブラリでは既定のサブスクリプション ID が使用されます。または、[`az account`](/cli/azure/manage-azure-subscriptions-azure-cli) を使用して、コードを実行する前にサブスクリプションを設定することもできます。

CLI ベースの認証を使用する場合、アプリケーションは、CLI のログイン資格情報で許可されているすべての操作に対して承認されます。 結果的に、自分がサブスクリプションの所有者または管理者であれば、特定のアクセス許可を割り当てなくても、そのサブスクリプションのほとんどのリソースに対するアクセス権がコードに最初から割り当てられます。 この動作は、実験目的であれば利便性が高いといえます。 ただし、これから皆さんは、個々の ID に対して厳密にアクセス許可を割り当て、それらを運用環境にデプロイする前に、アクセス許可をテスト環境で正しく検証する方法を身に付けることになるでしょう。そのため、運用環境のコードを作成する段階になったら、個別のサービス プリンシパルを使用し、個別のアクセス許可を割り当てることを強くお勧めします。

#### <a name="cli-based-authentication-with-azurecore-libraries"></a>azure.core ライブラリを使用した CLI ベースの認証

[azure.core 用に更新された Azure ライブラリ](./azure-sdk-library-package-index.md#libraries-using-azurecore)を使用する場合は、azure-identity ライブラリ (バージョン 1.4.0 以上) の [`AzureCliCredential`](/python/api/azure-identity/azure.identity.azureclicredential) オブジェクトを使用します。 たとえば、次のコードは、azure-mgmt-resource バージョン 15.0.0 以上で使用できます。

```python
from azure.identity import AzureCliCredential
from azure.mgmt.resource import SubscriptionClient

credential = AzureCliCredential()
subscription_client = SubscriptionClient(credential)

subscription = next(subscription_client.subscriptions.list())
print(subscription.subscription_id)
```

#### <a name="cli-based-authentication-with-older-non-azurecore-libraries"></a>古い (azure.core でない) ライブラリを使用した CLI ベースの認証

azure.core 用に更新されていない古い Azure ライブラリを使用する場合は、azure-cli-core ライブラリの [`get_client_from_cli_profile`](/python/api/azure-common/azure.common.client_factory#get-client-from-cli-profile-client-class----kwargs-) メソッドを使用できます。 たとえば、次のコードは、15.0.0 より下のバージョンの azure-mgmt-resource で使用できます。

```python
from azure.common.client_factory import get_client_from_cli_profile
from azure.mgmt.resource import SubscriptionClient

subscription_client = get_client_from_cli_profile(SubscriptionClient)

subscription = next(subscription_client.subscriptions.list())
print(subscription.subscription_id)
```

同じスクリプトでさまざまなサブスクリプションを参照する必要がある場合は、この記事で前述した [' get_client_from_auth_file'](#authenticate-with-a-json-file) または [`get_client_from_json_dict`](#authenticate-with-a-json-dictionary) メソッドを使用してください。

### <a name="deprecated-authenticate-with-userpasscredentials"></a>非推奨:UserPassCredentials を使用して認証する

[Azure Active Directory Authentication Library (ADAL) for Python](https://github.com/AzureAD/azure-activedirectory-library-for-python) が利用できるようになるまでは、現在は非推奨となっている [`UserPassCredentials`](/python/api/msrestazure/msrestazure.azure_active_directory.userpasscredentials) クラスを使用する必要がありました。 このクラスは 2 要素認証をサポートしていないため、今後は使用しないでください。

## <a name="see-also"></a>関連項目

- [Azure 用のローカル Python 開発環境を構成する](configure-local-development-environment.md)
- [ロールのアクセス許可を割り当てる方法](/azure/role-based-access-control/role-assignments-steps)
- [例:リソース グループをプロビジョニングする](azure-sdk-example-resource-group.md)
- [例:Azure Storage をプロビジョニングして使用する](azure-sdk-example-storage.md)
- [例:Web アプリをプロビジョニングしてコードをデプロイする](azure-sdk-example-web-app.md)
- [例:MySQL データベースをプロビジョニングして使用する](azure-sdk-example-database.md)
- [例:仮想マシンをプロビジョニングする](azure-sdk-example-virtual-machines.md)
- [仮想マシンで Azure Managed Disks を使用する](azure-sdk-samples-managed-disks.md)
- [Azure SDK for Python に関する簡単なアンケートに回答する](https://microsoft.qualtrics.com/jfe/form/SV_bNFX0HECjzPWMiG?Q_CHL=docs)
