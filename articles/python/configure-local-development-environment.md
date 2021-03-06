---
title: Azure で開発するためのローカル Python 環境を構成する
description: Visual Studio Code、Azure SDK ライブラリ、ライブラリ認証に必要な資格情報など、Azure で作業するためのローカル Python 開発環境を設定する方法。
ms.date: 01/04/2021
ms.topic: conceptual
ms.custom: devx-track-python, devx-track-azurecli
ms.openlocfilehash: 184996eca52c096602863beb1c73ae4337695829
ms.sourcegitcommit: 3843092e47691fbd32452c93d51f894a0cab31db
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/29/2021
ms.locfileid: "99069079"
---
# <a name="configure-your-local-python-dev-environment-for-azure"></a>Azure 用のローカル Python 開発環境を構成する

クラウド アプリケーションを作成する場合、開発者は通常、Azure などのクラウド環境にコードをデプロイする前に、ローカル ワークステーションでそのコードをテストしたいと考えます。 ローカル開発には、幅広いデバッグ ツールを使用して迅速にテストできるという利点があります。

この記事では、Azure 上の Python に適したローカル開発環境を作成および検証するための 1 回限りのセットアップ手順について説明します。

- [必須コンポーネントをインストールします](#required-components) (つまり Azure アカウント、Python、Azure CLI)。
- Azure ライブラリを使用して Azure リソースをプロビジョニング、管理、およびアクセスするときに使用する[認証を構成します](#configure-authentication)。
- プロジェクトごとに、[Python 仮想環境を使用する](#use-python-virtual-environments)プロセスを確認します。

ワークステーションを構成しておけば、このデベロッパー センターや Azure のドキュメントで紹介されているさまざまなクイックスタートやチュートリアルを完了するために必要な追加構成を最小限に抑えることができます。

ローカル開発のこのセットアップは、Azure でアプリケーションの "*クラウド環境*" を構成する [プロビジョニング リソース](cloud-development-flow.md)とは別のものです。 開発プロセスでは、それらのクラウド リソースにアクセスできるローカル開発環境でコードを実行しますが、コードはクラウドで[適切なホスティング サービス](quickstarts-app-hosting.md)にまだデプロイされていません。 [Azure 開発フロー](cloud-development-flow.md)記事で説明されているように、その開発手順は後で行われます。

## <a name="install-components"></a>コンポーネントをインストールする

### <a name="required-components"></a>必須コンポーネント

| 名前またはインストーラー | 説明 |
| --- | --- |
| [アクティブなサブスクリプションが含まれる Azure アカウント](https://azure.microsoft.com/free/?utm_source=campaign&utm_campaign=python-dev-center&mktingSource=environment-setup) | アカウントおよびサブスクリプションは無料です。また、無料で利用できるサービスも多数用意されています。 |
| [Python 2.7 以降または 3.5.3 以降](https://www.python.org/downloads) | Python 言語ランタイム。 特定のバージョン要件がない限り、最新バージョンの Python 3.x を使用することをお勧めします。 |
| [Azure コマンド ライン インターフェイス (CLI)](/cli/azure/install-azure-cli) | Azure リソースをプロビジョニングおよび管理するための CLI コマンドの完全なスイートが用意されています。 Python 開発者は、通常、Azure 管理ライブラリを使用するカスタムの Python スクリプトと共に Azure CLI を使用します。 |

注:

- 必要に応じて、プロジェクトごとに個々の Azure ライブラリ パッケージをインストールします。 プロジェクトごとに [Python 仮想環境を使用する](#use-python-virtual-environments)ことをお勧めします。 Python 用のスタンドアロン "SDK" インストーラーはありません。
- Azure PowerShell は全体として Azure CLI と同等ですが、Python を使用する場合は Azure CLI をお勧めします。

### <a name="recommended-components"></a>推奨コンポーネント

| 名前またはインストーラー | 説明 |
| --- | --- |
| [Visual Studio Code](https://code.visualstudio.com) | 適切なエディターや IDE を使用することもできますが、Python 開発者の間では、Microsoft が無料で提供するこの軽量な IDE が広く普及しています。 概要については、[VS Code での Python](https://code.visualstudio.com/docs/python/python-tutorial)に関するページをご覧ください。 |
| [VS Code 用の Python 拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-python.python) | VS Code に Python のサポートを追加します。 |
| [VS Code 用の Azure 拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack) | さまざまな Azure サービスのサポートを VS Code に追加します。 特定のサービスのサポートを個別にインストールすることもできます。 |
| [git](https://git-scm.com/downloads) | ソース管理用のコマンドライン ツール。 好みに応じて、さまざまなソース管理ツールを使用できます。 |

### <a name="optional-components"></a>オプション コンポーネント

| 名前またはインストーラー | 説明 |
| --- | --- |
| [VS Code 用の Docker 拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker) | VS Code に Docker のサポートを追加します。これは、普段からコンテナーを使用しているユーザーには便利です。 |

### <a name="verify-components"></a>コンポーネントを確認する

1. ターミナルまたはコマンド プロンプトを開きます。
1. `python --version` コマンドを実行して、Python のバージョンを確認します。
1. `az --version` を実行して、Azure CLI のバージョンを確認します。
1. VS Code のインストールを確認します。
    1. `code .` を実行して、現在のフォルダーに VS Code を開きます。
    1. VS Code で、 **[ビュー]**  >  **[拡張機能]** を選択して拡張機能ビューを開き、一覧に "Python" と "Azure アカウント" が表示されることを確認します (その他 "Azure" の拡張機能や "Docker" もインストールしている場合は、その拡張機能が表示されることも確認します)。

## <a name="sign-in-to-azure-from-the-cli"></a>CLI から Azure にサインインする

ターミナルまたはコマンド プロンプトで、Azure サブスクリプションにサインインします。

```azurecli
az login
```

`az` コマンドは、Azure CLI のルート コマンドです。 `az` の後には、1 つまたは複数の特定のコマンド (`login` など) が続きます。 [az login](/cli/azure/authenticate-azure-cli) のコマンド リファレンスを参照してください。

Azure CLI では通常、セッション間のサインインが維持されますが、新しいターミナルまたはコマンド プロンプトを開くたびに `az login` を実行することをお勧めします。

## <a name="configure-authentication"></a>認証を構成する。

[アプリの認証方法](azure-sdk-authenticate.md#identity-when-running-the-app-locally)に関する記事で説明されているように、アプリ コードをローカルでテストするときに、各開発者にはアプリケーション ID として使用するサービス プリンシパルが必要になります。

以下のセクションでは、サービス プリンシパルと、必要に応じて Azure ライブラリにサービス プリンシパルのプロパティを提供する環境変数を作成する方法について説明します。

組織内の開発者それぞれが、これらの手順を実行する必要があります。

### <a name="create-a-service-principal-and-environment-variables-for-development"></a>開発用のサービス プリンシパルと環境変数を作成する

1. Azure CLI (`az login`) にサインインしたターミナルまたはコマンド プロンプトを開きます。

1. 次のとおり、サービス プリンシパルを作成します。

    ```azurecli
    az ad sp create-for-rbac --name localtest-sp-rbac --skip-assignment --sdk-auth > local-sp.json
    ```

    このコマンドは *local-sp.json* に出力を保存します。 コマンドとその引数の詳細については、「[create-for-rbac コマンドの機能](#what-the-create-for-rbac-command-does)」を参照してください。

    組織内のユーザーの場合、このコマンドを実行するためのアクセス許可がサブスクリプションにない可能性があります。 その場合は、サブスクリプションの所有者に連絡して、代わりにサービス プリンシパルを作成するように依頼してください。

1. 次のコマンドを使用して、Azure ライブラリに必要な環境変数を作成します。 (azure-identity ライブラリの `DefaultAzureCredential` オブジェクトでこれらの変数が検索されます。)

    # <a name="cmd"></a>[cmd](#tab/cmd)

    ```cmd
    set AZURE_SUBSCRIPTION_ID="aa11bb33-cc77-dd88-ee99-0918273645aa"
    set AZURE_TENANT_ID=00112233-7777-8888-9999-aabbccddeeff
    set AZURE_CLIENT_ID=12345678-1111-2222-3333-1234567890ab
    set AZURE_CLIENT_SECRET=abcdef00-4444-5555-6666-1234567890ab
    ```

    # <a name="bash"></a>[bash](#tab/bash)

    ```bash
    AZURE_SUBSCRIPTION_ID="aa11bb33-cc77-dd88-ee99-0918273645aa"
    AZURE_TENANT_ID="00112233-7777-8888-9999-aabbccddeeff"
    AZURE_CLIENT_ID="12345678-1111-2222-3333-1234567890ab"
    AZURE_CLIENT_SECRET="abcdef00-4444-5555-6666-1234567890ab"
    ```

    ---

    これらのコマンドに示されている値を、お客様固有のサービス プリンシパルの値で置き換えます。

    サブスクリプション ID を取得するには、[`az account show`](/cli/azure/account#az-account-show) コマンドを実行し、その出力で `id` プロパティを探します。

    利便性のため、これらの同じコマンドを含むコマンド ライン スクリプト ファイル (macOS/Linux の場合は *setenv.sh*、Windows の場合は *setenv.cmd*) を作成します。 その後、ローカル テストのためにターミナルまたはコマンド プロンプトを開いたときにいつでも、このスクリプトを実行して変数を設定できます。 この場合も、スクリプト ファイルはソース管理には追加せず、ユーザー アカウント内にのみ残します。

1. クライアント ID とクライアント シークレット (およびこれを格納するファイル) を保護して、これが常にワークステーション上の特定のユーザー アカウント内にとどまるようにします。 これらのプロパティをソース管理に保存したり、他の開発者と共有したりしないでください。 必要に応じて、サービス プリンシパルを削除し、新しいサービス プリンシパルを作成することができます。

    セキュリティをさらに強化するために、定期的なスケジュールでサービス プリンシパルを削除して再作成するポリシーを作成し、以前の ID とシークレットを無効にできます。

    さらに、開発サービス プリンシパルは、非運用環境のリソースに対してのみ承認するか、開発目的のみに使用される Azure サブスクリプション内に作成することをお勧めします。 したがって、運用環境のアプリケーションでは、デプロイ済みクラウド アプリケーションに対してのみ承認されている別のサブスクリプションと運用環境リソースを使用します。

1. 後でサービス プリンシパルを変更または削除する方法については、「[サービス プリンシパルを管理する方法](how-to-manage-service-principals.md)」を参照してください。

#### <a name="what-the-create-for-rbac-command-does"></a>create-for-rbac コマンドの機能

`az ad sp create-for-rbac` コマンドは、"ロールベースの認証" (RBAC) のサービス プリンシパルを作成します。 (サービス プリンシパルの詳細については、「[Azure で Python アプリを認証および認可する方法](azure-sdk-authenticate.md)」を参照してください。)

- `ad` は Azure Active Directory を意味します。`sp` は "サービス プリンシパル" を意味し、`create-for-rbac` は Azure の主要な承認形式である "ロールベースのアクセス制御用に作成する" ことを意味します。 [az ad sp create-rbac](/cli/azure/ad/sp#az-ad-sp-create-for-rbac) のコマンド リファレンスを参照してください。

- `--name` 引数は組織内で一意である必要があり、通常はそのサービス プリンシパルを使用する開発者の名前を指定します。 この引数を省略した場合、Azure CLI では `azure-cli-<timestamp>` という形式の汎用名が使用されます。 サービス プリンシパルの名前は、必要に応じて Azure portal で変更できます。

- `--skip-assignment` 引数によって、既定のアクセス許可を持たないサービス プリンシパルが作成されます。 したがって、サービス プリンシパルに特定のアクセス許可を割り当てて、ローカルで実行されるコードが任意のリソースにアクセスできるようにする必要があります。 詳細については、「[Azure のロールベースのアクセス制御 (RBAC) とは](/azure/role-based-access-control/overview)」と「[ロールの割り当てを追加する手順](/azure/role-based-access-control/role-assignments-steps)」を参照してください。 関連する特定のリソースでサービス プリンシパルを認可する方法の詳細については、さまざまなクイックスタートやチュートリアルも参照してください。

- このコマンドは JSON 出力を提供します。この例では、*local-sp.json* という名前のファイルに保存されています。

- `--sdk-auth` 引数により、次の値のような JSON 出力が生成されます。 ご使用の ID 値およびシークレットはすべて異なったものになります。

    <pre>
    {
      "clientId": "12345678-1111-2222-3333-1234567890ab",
      "clientSecret": "abcdef00-4444-5555-6666-1234567890ab",
      "subscriptionId": "00000000-0000-0000-0000-000000000000",
      "tenantId": "00112233-7777-8888-9999-aabbccddeeff",
      "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
      "resourceManagerEndpointUrl": "https://management.azure.com/",
      "activeDirectoryGraphResourceId": "https://graph.windows.net/",
      "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
      "galleryEndpointUrl": "https://gallery.azure.com/",
      "managementEndpointUrl": "https://management.core.windows.net/"
    }
    </pre>

    `--sdk-auth` 引数を指定しない場合、このコマンドによりもっと単純な出力が生成されます。

    <pre>
    {
      "appId": "12345678-1111-2222-3333-1234567890ab",
      "displayName": "localtest-sp-rbac",
      "name": "http://localtest-sp-rbac",
      "password": "abcdef00-4444-5555-6666-1234567890ab",
      "tenant": "00112233-7777-8888-9999-aabbccddeeff"
    }
    </pre>

    この場合、`tenant` はテナント ID、`appId` はクライアント ID、`password` はクライアント シークレットです。

    > [!WARNING]
    >  `az ad sp create-for-rbac` を使用してサービス プリンシパルを作成する場合は、パスワード、クライアント シークレット、証明書など、保護する必要がある資格情報が出力に含まれます。 これらの資格情報は、ソース管理にコミットされているコードまたはファイルには保存しないでください。
    > 既定では、`az ad sp create-for-rbac` によって、[共同作成者ロール](/azure/role-based-access-control/built-in-roles#contributor)がサブスクリプション スコープでサービス プリンシパルに割り当てられます。 サービス プリンシパルが侵害された場合のリスクを軽減するには、より具体的なロールを割り当て、そのスコープをリソースまたはリソース グループに絞り込みます。
    > (ローカル開発ではなく) 実稼働コードの場合は、特定のサービス プリンシパルではなく、可能な場合は[マネージド ID](/azure/active-directory/managed-identities-azure-resources/overview) を使用します。

    > [!IMPORTANT]
    > クライアント シークレットおよびパスワードを確認できる場所は、このコマンドからの出力のみです。 後でシークレットとパスワードを取得することはできません。 ただし必要に応じて、サービス プリンシパルや既存のシークレットを無効にせずに、新しいシークレットを追加することができます。

## <a name="use-python-virtual-environments"></a>Python 仮想環境を使用する

常にプロジェクトごとに、次の手順に従って "*仮想環境*" を作成し、アクティブにすることをお勧めします。

1. ターミナルまたはコマンド プロンプトを開きます。

1. プロジェクトのフォルダーを作成します。

1. 次のコマンドを実行して、仮想環境を作成します。

    # <a name="cmd"></a>[cmd](#tab/cmd)

    ```bash
    # py -3 uses the global python interpreter. You can also use python -m venv .venv.
    py -3 -m venv .venv
    ```

    # <a name="bash"></a>[bash](#tab/bash)

    ```bash
    # On Windows, use py -3 -m venv .venv
    python3 -m venv .venv
    ```

    ---

    このコマンドにより、Python `venv` モジュールが実行され、`.venv` という名前のフォルダーに仮想環境が作成されます。

1. 次のコマンドを実行して、仮想環境をアクティブにします。

    # <a name="cmd"></a>[cmd](#tab/cmd)

    ```cmd
    .venv\scripts\activate
    ```

    # <a name="bash"></a>[bash](#tab/bash)

    ```bash
    source .venv/bin/activate
    ```

    ---

仮想環境は、特定の Python インタープリターのコピーを分離する、プロジェクト内のフォルダーです。 この環境がアクティブになったら (アクティブ化は Visual Studio Code によって自動的に実行されます)、`pip install` を実行することで、ライブラリがその環境にのみインストールされます。 次に、Python コードを実行すると、すべてのライブラリの特定バージョンを使用して、環境の正確なコンテキストでコードが実行されます。 また `pip freeze` を実行すると、これらのライブラリの正確な一覧が表示されます (このドキュメントの例の多くでは、必要なライブラリを示す *requirements.txt* ファイルを作成し、`pip install -r requirements.txt` を使用します。 コードを Azure にデプロイする場合は、通常、要件ファイルが必要になります)。

仮想環境を使用しない場合、Python は "*グローバル環境*" で実行されます。 グローバル環境を使用するのは簡単で便利ですが、プロジェクトや実験用にインストールされるライブラリで時間と共に環境が肥大化する傾向があります。 さらに、1 つのプロジェクト用のライブラリを更新すると、そのライブラリのさまざまなバージョンに依存する他のプロジェクトが破損する可能性があります。 また、グローバル環境は任意の数のプロジェクトによって共有されるので、1 つのプロジェクトの依存関係の一覧を取得するために `pip freeze` を使用することはできません。

グローバル環境は、複数のプロジェクトで使用するツール パッケージをインストールするための場所です。 たとえば、グローバル環境で `pip install gunicorn` を実行すると、Gunicorn Web サーバーをどこでも利用できるようになります。

## <a name="use-source-control"></a>ソース管理を使用する

プロジェクトを開始するときは常にソース管理リポジトリを作成するという習慣を身に付けることをお勧めします。 Git がインストールされている場合は、次のコマンドを実行するだけで済みます。

```cmd
git init
```

そこから、`git add` や `git commit` などのコマンドを実行して変更をコミットできます。 変更を定期的にコミットすることにより、コミット履歴が作成されるので、以前の状態に戻すことができるようになります。

プロジェクトのオンライン バックアップを作成するには、リポジトリを [GitHub](https://github.com) または [Azure DevOps](/azure/devops/user-guide/code-with-git?view=azure-devops&preserve-view=true) にアップロードすることをお勧めします。 最初にローカル リポジトリを初期化した場合は、`git remote add` を使用して、ローカル リポジトリを GitHub または Azure DevOps にアタッチします。

Git のドキュメントは、[git-scm.com/docs](https://git-scm.com/docs) と、インターネット上のさまざまな場所で参照できます。

Visual Studio Code には、数多くの組み込み Git 機能が含まれています。 詳細については、「[VS Code でのバージョン管理の使用](https://code.visualstudio.com/docs/editor/versioncontrol)」を参照してください。

他の任意のソース管理ツールを使用することもできます。Git は最も広く使用され、サポートされているソース管理ツールの 1 つにすぎません。

## <a name="next-step"></a>次のステップ

ローカルの開発環境が整ったら、Azure ライブラリの一般的な使用パターンを簡単に確認します。

> [!div class="nextstepaction"]
> [一般的な使用パターンを確認する >>>](azure-sdk-library-usage-patterns.md)
