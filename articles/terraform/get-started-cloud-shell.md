---
title: クイック スタート - Azure Cloud Shell を使用して Terraform を構成する
description: このクイックスタートでは、Azure Cloud Shell に Terraform をインストールして構成する方法について説明します。
keywords: Azure DevOps Terraform インストール 構成 Cloud Shell 初期化 プラン 適用 実行 portal ログイン RBAC サービス プリンシパル 自動スクリプト
ms.topic: quickstart
ms.date: 09/27/2020
ms.custom: devx-track-terraform, devx-track-azurecli
adobe-target: true
ms.openlocfilehash: 6befee6004b23410693f82c83f62e52318fd1c08
ms.sourcegitcommit: b380f6e637b47e6e3822b364136853e1d342d5cd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/14/2021
ms.locfileid: "100395407"
---
# <a name="quickstart-configure-terraform-using-azure-cloud-shell"></a>クイック スタート:Azure Cloud Shell を使用して Terraform を構成する
 
[!INCLUDE [terraform-intro.md](includes/terraform-intro.md)]

この記事では、[Azure で Terraform](https://www.terraform.io/docs/providers/azurerm/index.html) の使用を開始する方法について説明します。

この記事では、次のことについて説明します。
> [!div class="checklist"]
> * Azure に対して認証します
> * Azure CLI を使用して Azure サービス プリンシパルを作成する
> * サービス プリンシパルを使用して Azure に対して認証する
> * 現在の Azure サブスクリプションを設定する - 複数のサブスクリプションがある場合に使用
> * 基本の Terraform 構成ファイルを作成する
> * Terraform 実行プランを作成して適用する
> * 実行プランを破棄する

## <a name="prerequisites"></a>前提条件

[!INCLUDE [open-source-devops-prereqs-azure-subscription.md](../includes/open-source-devops-prereqs-azure-subscription.md)]

## <a name="configure-your-environment"></a>環境を構成する

1. [Azure ポータル](https://portal.azure.com)にアクセスします。

1. まだログインしていない場合は、利用可能な Microsoft アカウントの一覧が Azure portal に表示されます。 1 つ以上のアクティブな Azure サブスクリプションに関連付けられている Microsoft アカウントを選択し、資格情報を入力して続行します。

1. Cloud Shell を開きます。

    ![Cloud Shell へのアクセス](media/install-configure/portal-cloud-shell.png)

1. Cloud Shell を以前に使用したことがない場合は、環境とストレージの設定を構成します。 この記事では、Bash 環境を使用します。

**注**:
- Cloud Shell には、Terraform の最新バージョンが自動的にインストールされます。 また、Terraform は、現在の Azure サブスクリプションの情報を自動的に使用します。 そのため、インストールや構成は必要ありません。

## <a name="authenticate-to-azure"></a>Azure に対して認証します

Cloud Shell は、Azure portal へのログインに使用した Microsoft アカウントで自動的に認証されます。 お使いのアカウントに複数の Azure サブスクリプションがある場合は、[他のいずれかのサブスクリプションに切り替える](#set-the-current-azure-subscription)ことができます。

Terraform では、Azure に対して認証を行うためのオプションがいくつかサポートされています。 この記事では、次の手法について説明します。

- Terraform を対話的に使用する場合は、[Microsoft アカウントを使用した認証](#authenticate-via-microsoft-account)をお勧めします。
- Terraform をコードから使用する場合は、[Azure サービス プリンシパルを使用した認証](#authenticate-via-azure-service-principal)が推奨される方法の 1 つです。

### <a name="authenticate-via-microsoft-account"></a>Microsoft アカウントを使用した認証

パラメーターを指定せずに `az login` を呼び出すと、URL とコードが表示されます。 URL を参照してコードを入力し、指示に従って、ご自身の Microsoft アカウントを使用して Azure にログインします。 ログインしたら、ポータルに戻ります。

```azurecli
az login
```

**注**:

- ログインが成功すると、`az login` により、ログインした Microsoft アカウントに関連付けられている Azure サブスクリプションの一覧が表示されます。
- 使用可能な Azure サブスクリプションごとにプロパティの一覧が表示されます。 `isDefault` プロパティは、使用している Azure サブスクリプションを識別します。 別の Azure サブスクリプションに切り替える方法については、「[現在の Azure サブスクリプションを設定する](#set-the-current-azure-subscription)」セクションを参照してください。

### <a name="authenticate-via-azure-service-principal"></a>Azure サービス プリンシパルを使用した認証

**Azure サービス プリンシパルを作成する**:サービス プリンシパルを使用して Azure サブスクリプションにログインするには、まずサービス プリンシパルへのアクセスが必要です。 サービス プリンシパルを既にお持ちの場合は、セクションのこの部分を省略できます。

Azure サービスをデプロイまたは使用する自動化ツール (Terraform など) のアクセス許可は、常に制限されている必要があります。 完全な特権を持つユーザーとしてアプリケーションをログインさせる代わりに、Azure にはサービス プリンシパルが用意されています。 しかし、ログインに使用するサービス プリンシパルがない場合はどうすればよいでしょうか。 そのようなシナリオの場合、ユーザー資格情報を使用してログインしてから、サービス プリンシパルを作成できます。 サービス プリンシパルが作成されると、その情報を今後のログイン試行に使用できるようになります。

[Azure CLI でサービス プリンシパルを作成](/cli/azure/create-an-azure-service-principal-azure-cli?)するときは、多くのオプションがあります。 この記事では、[az ad sp create-for-rbac](/cli/azure/ad/sp?#az-ad-sp-create-for-rbac) を作成して、**共同作成者** ロールを持つサービス プリンシパルを作成します。 **共同作成者** ロール (既定) には、Azure アカウントに対して読み取りと書き込みを行うための完全なアクセス許可があります。 ロールベースのアクセス制御 (RBAC) とロールの詳細については、[RBAC の組み込みのロール](/azure/active-directory/role-based-access-built-in-roles)に関するページをご覧ください。

次のコマンドを入力します。`<subscription_id>` を、使用するサブスクリプションアカウントの ID に置き換えます。

```azurecli
az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/<subscription_id>"
```

**注**:

- 正常に完了すると、`az ad sp create-for-rbac` によっていくつかの値が表示されます。 `name`、`password`、および `tenant` の値は、次のステップで使用します。
- このパスワードは、紛失した場合、取得できません。 したがって、パスワードは安全な場所に保管してください。 パスワードを忘れた場合は、[サービス プリンシパルの資格情報をリセット](/cli/azure/create-an-azure-service-principal-azure-cli#reset-credentials)する必要があります。

**Azure サービス プリンシパルを使用してログインする**: 次の `az login` への呼び出しで、プレースホルダーをご自身のサービス プリンシパルの情報に置き換えます。

```azurecli
az login --service-principal -u <service_principal_name> -p "<service_principal_password>" --tenant "<service_principal_tenant>"
```

## <a name="set-the-current-azure-subscription"></a>現在の Azure サブスクリプションを設定する

Microsoft アカウントは、複数の Azure サブスクリプションに関連付けることができます。 次の手順では、サブスクリプションを切り替える方法の概要を説明します。

1. 現在の Azure サブスクリプションを表示するには、[az account show](/cli/azure/account#az-account-show) を使用します。

    ```azurecli
    az account show
    ```

1. 複数の利用可能な Azure サブスクリプションにアクセスできる場合は、[az account list](/cli/azure/account#az-account-list) を使用して、サブスクリプション名 ID の値の一覧を表示します。

    ```azurecli
    az account list --query "[].{name:name, subscriptionId:id}"
    ```

1. 現在の Cloud Shell セッションで特定の Azure サブスクリプションを使用するには、[az account set](/cli/azure/account#az-account-set) を使用します。 `<subscription_id>` プレースホルダーを、使用するサブスクリプションの ID (または名前) に置き換えます。

    ```azurecli
    az account set --subscription="<subscription_id>"
    ```

    **注**:

    - `az account set` を呼び出すと、指定した Azure サブスクリプションに切り替えた結果が表示されません。 ただし、`az account show` を使用して、現在の Azure サブスクリプションが変更されたことを確認できます。

[!INCLUDE [terraform-create-base-config-file.md](includes/terraform-create-base-config-file.md)]

[!INCLUDE [terraform-create-and-apply-execution-plan.md](includes/terraform-create-and-apply-execution-plan.md)]

[!INCLUDE [terraform-reverse-execution-plan.md](includes/terraform-reverse-execution-plan.md)]

[!INCLUDE [terraform-troubleshooting.md](includes/terraform-troubleshooting.md)]

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [Terraform を使用して Linux VM を作成する](create-linux-virtual-machine-with-infrastructure.md)
