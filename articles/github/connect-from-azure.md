---
title: GitHub と Azure を接続する
description: Azure およびその他のサービスから GitHub に接続するためのリソース
author: N-Usha
ms.author: ushan
ms.topic: reference
ms.service: azure
ms.date: 02/17/2021
ms.custom: github-actions-azure, devx-track-azurecli
ms.openlocfilehash: 136e11c6059ab8c25af85212f0f0f96b88652a56
ms.sourcegitcommit: 576c878c338d286060010646b96f3ad0fdbcb814
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102118240"
---
# <a name="use-github-actions-to-connect-to-azure"></a>GitHub Actions を使用して Azure に接続する

[Azure ログイン](https://github.com/Azure/login)を [Azure PowerShell](https://github.com/Azure/PowerShell) または [Azure CLI](https://github.com/Azure/CLI) とともに使用して Azure リソースを操作する方法について説明します。

GitHub Actions ワークフローで Azure PowerShell または Azure CLI を使用するには、まず、[Azure login](https://github.com/marketplace/actions/azure-login) アクションを使用してログインする必要があります。
Azure login アクションを使用すると、[Azure AD サービス プリンシパル](/azure/active-directory/develop/app-objects-and-service-principals#service-principal-object)のコンテキストでワークフロー内のコマンドを実行できます。

既定では、このアクションは Azure CLI を使用してログインし、Azure CLI に対して GitHub アクション ランナー環境を設定します。 Azure login アクションの `enable-AzPSSession` プロパティを使用すると、Azure PowerShell を使用できます。 これは、Azure PowerShell モジュールを使用して GitHub アクション ランナー環境を設定します。

Azure ログインを使用して、Azure Government や Azure Stack Hub などのパブリックまたはソブリン クラウドに接続できます。

## <a name="create-a-service-principal-and-add-it-to-github-secret"></a>サービス プリンシパルを作成して GitHub シークレットに追加する

[Azure ログイン](https://github.com/marketplace/actions/azure-login)を使用するには、まず、Azure サービス プリンシパルをシークレットとして GitHub リポジトリに追加する必要があります。

この例では、Azure での認証に使用できる `AZURE_CREDENTIALS` という名前のシークレットを作成します。  

1. 既存のアプリケーションがない場合は、サービス プリンシパルで使用する[新しい Active Directory アプリケーション](/azure/active-directory/develop/howto-create-service-principal-portal#register-an-application-with-azure-ad-and-create-a-service-principal&preserve-view=true)を登録します。

    ```azurecli-interactive
        appName="myApp"

        az ad app create \
        --display-name $appName \
        --homepage "http://localhost/$appName" \
        --identifier-uris http://localhost/$appName
    ```

1. Azure portal または [Azure CLI](/cli/azure/install-azure-cli) でローカルに [Azure Cloud Shell](/azure/cloud-shell/overview) を開きます。

    > [!NOTE]
    > Azure Stack Hub を使用している場合は、SQL 管理エンドポイントを `not supported` に設定する必要があります。
    > `az cloud update -n {environmentName} --endpoint-sql-management https://notsupported`

1. Azure portal でアプリ用の[新しいサービス プリンシパルを作成します](/cli/azure/create-an-azure-service-principal-azure-cli)。 サービス プリンシパルには、共同作成者ロールが割り当てられている必要があります。

    ```azurecli-interactive
        az ad sp create-for-rbac --name "myApp" --role contributor \
                                    --scopes /subscriptions/{subscription-id}/resourceGroups/{resource-group} \
                                    --sdk-auth
    ```
    
1. サービス プリンシパルの JSON オブジェクトをコピーします。

    ```json
    {
        "clientId": "<GUID>",
        "clientSecret": "<GUID>",
        "subscriptionId": "<GUID>",
        "tenantId": "<GUID>",
        (...)
    }
    ```

1. GitHub リポジトリを開き、 **[Settings]\(設定\)** に移動します。

    :::image type="content" source="media/github-repo-settings.png" alt-text="ナビゲーションで [Settings]\(設定\) を選択する":::

1. **[Settings]\(設定\)** 、 **[New Secret]\([新しいシークレット\)]** の順に選択します。

    :::image type="content" source="media/select-secrets.png" alt-text="シークレットの追加を選択する":::

1. `AZURE_CREDENTIALS` という名前のサービス プリンシパルの JSON オブジェクトを貼り付けます。 

    :::image type="content" source="media/azure-secret-add.png" alt-text="GitHub でシークレットを追加する":::

1. **[Add secret]\(シークレットの追加\)** を選択して保存します。

## <a name="use-the-azure-login-action"></a>Azure ログイン アクションを使用する

Azure で認証するには、サービス プリンシパル シークレットと [Azure Login アクション](https://github.com/Azure/login)を使用します。

このワークフローでは、`secrets.AZURE_CREDENTIALS` に格納されているサービス プリンシパルの詳細を使用して、Azure login アクションを使って認証を行います。 次に、Azure CLI アクションを実行します。 ワークフロー ファイルで GitHub シークレットを参照する方法の詳細については、GitHub Docs の「[ワークフローでの暗号化されたシークレットの使用](https://docs.github.com/en/actions/reference/encrypted-secrets#using-encrypted-secrets-in-a-workflow)」をご覧ください。

動作する Azure ログイン ステップを作成したら、[Azure PowerShell](https://github.com/Azure/PowerShell) または[ Azure CLI](https://github.com/Azure/CLI) のアクションを使用できます。 [Azure Webapp のデプロイ](https://github.com/Azure/webapps-deploy)や [Azure Functions](https://github.com/Azure/functions-action) など、その他の Azure Actions を使用することもできます。

```yaml
on: [push]

name: AzureLoginSample

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Log in with Azure
        uses: azure/login@v1
        with:
          creds: '${{ secrets.AZURE_CREDENTIALS }}'
```

## <a name="use-the-azure-powershell-action"></a>Azure PowerShell アクションを使用する

この例では、[Azure Login アクション](https://github.com/Azure/login)を使用してログインし、[Azure PowerShell アクション](https://github.com/azure/powershell)を使用してリソース グループを取得します。

```yaml
on: [push]

name: AzureLoginSample

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Log in with Azure
        uses: azure/login@v1
        with:
          creds: '${{ secrets.AZURE_CREDENTIALS }}'
          enable-AzPSSession: true
      - name: Azure PowerShell Action
        uses: Azure/powershell@v1
        with:
          inlineScript: Get-AzVM -ResourceGroupName "< YOUR RESOURCE GROUP >"
          azPSVersion: 3.1.0
```

## <a name="use-the-azure-cli-action"></a>Azure CLI アクションを使用する

この例では、[Azure ログイン アクション](https://github.com/Azure/login)を使用してログインし、[Azure CLI アクション](https://github.com/Azure/CLI)を使用してリソース グループを取得します。


```yaml
on: [push]

name: AzureLoginSample

jobs:
build-and-deploy:
    runs-on: ubuntu-latest
    steps:

    - name: Log in with Azure
        uses: azure/login@v1
        with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}

    - name: Azure CLI script
        uses: azure/CLI@v1
        with:
        azcliversion: 2.0.72
        inlineScript: |
            az account show
            az storage -h
```

## <a name="connect-to-azure-government-and-azure-stack-hub-clouds"></a>Azure Government および Azure Stack Hub クラウドに接続する

いずれかの Azure Government クラウドにログインするには、サポートされているクラウド名 `AzureUSGovernment` または `AzureChinaCloud` を使用して、オプションのパラメーター環境を設定する必要があります。 このパラメーターを指定しないと、既定値の `AzureCloud` が使用され、Azure パブリック クラウドに接続されます。

```yaml
   - name: Login to Azure US Gov Cloud with CLI
     uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_US_GOV_CREDENTIALS }}
          environment: 'AzureUSGovernment'
          enable-AzPSSession: false
   - name: Login to Azure US Gov Cloud with Az Powershell
      uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_US_GOV_CREDENTIALS }}
          environment: 'AzureUSGovernment'
          enable-AzPSSession: true
```

## <a name="connect-with-other-azure-services"></a>他の Azure サービスと接続する

次の記事では、Azure およびその他のサービスから GitHub に接続する方法について詳しく説明しています。  

### <a name="azure-active-directory"></a>Azure Active Directory 

- [Azure AD で GitHub Enterprise にサインインする (シングル サインオン)](/azure/active-directory/saas-apps/github-tutorial)   

### <a name="power-bi"></a>Power BI

- [Power BI を GitHub に接続する](/power-bi/service-connect-to-github)   

### <a name="connectors"></a>Connectors

- [Azure Logic Apps、Power Automate、および Power Apps 用の GitHub コネクタ](/connectors/github/)   

### <a name="azure-databricks"></a>Azure Databricks

- [GitHub をノートブックのバージョン管理として使用する](/azure/databricks/notebooks/github-version-control) 

> [!div class="nextstepaction"]
> [GitHub から Azure にアプリをデプロイする](deploy-to-azure.md)
