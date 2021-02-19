---
title: チュートリアル - Terraform と Azure を使用した統合テスト
description: 統合テストと、Azure DevOps を使用した Terraform プロジェクトの継続的インテグレーションの構成方法について説明します。
ms.topic: tutorial
ms.date: 10/08/2020
ms.custom: devx-track-terraform
ms.openlocfilehash: 7b10693907feedf94db60315d8819d45fdd47412
ms.sourcegitcommit: b19420d1a8d526a81e0835012cccc46717fadd69
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/15/2021
ms.locfileid: "100524897"
---
# <a name="tutorial-configure-integration-tests-for-terraform-projects-in-azure"></a>チュートリアル:Azure で Terraform プロジェクトの統合テストを構成する

[!INCLUDE [terraform-intro.md](includes/terraform-intro.md)]

この記事では、次のタスクを行う方法について説明します。

> [!div class="checklist"]
> * Terraform プロジェクトの統合テストの基本について学びます。
> * Azure DevOps を使用して継続的インテグレーション パイプラインを構成します。
> * Terraform コードに対する静的コード分析を実行します。
> * `terraform validate` を実行して、ローカル コンピューター上の Terraform 構成ファイルを検証します。
> * `terraform plan` を実行して、リモート サービスの観点からその Terraform 構成ファイルを検証します。
> * Azure パイプラインを使用して継続的インテグレーションを自動化します。

## <a name="prerequisites"></a>前提条件

[!INCLUDE [open-source-devops-prereqs-azure-subscription.md](../includes/open-source-devops-prereqs-azure-subscription.md)]
- **Azure DevOps 組織とプロジェクト**:まだお持ちでない場合は、[Azure DevOps 組織を作成します](/azure/devops/organizations/projects/create-project)。
- **Terraform の Build & Release Tasks 拡張機能**:Azure DevOps 組織に [Terraform のビルド/リリース タスク拡張機能をインストール](https://marketplace.visualstudio.com/items?itemName=charleszipp.azure-pipelines-tasks-terraform)します。
- **Azure サブスクリプションへの Azure DevOps のアクセスを許可する**:`terraform-basic-testing-azure-connection` という名前の [Azure サービス接続](/azure/devops/pipelines/library/connect-to-azure)を作成して、Azure Pipelines を Azure サブスクリプションに接続できるようにします。
- **Terraform のインストール**:ご使用の環境に応じて、[Terraform をダウンロードしてインストール](https://www.terraform.io/downloads.html)します。
- **テスト用サンプルをフォークする**:[GitHub の Terraform サンプル プロジェクト](https://github.com/Azure/terraform)をフォークし、開発/テスト用コンピューターに複製します。

## <a name="validate-a-local-terraform-configuration"></a>ローカルの Terraform の構成を検証する

[terraform validate](https://www.terraform.io/docs/commands/validate.html) コマンドは、Terraform ファイルが格納されているディレクトリでコマンド ラインから実行します。 このコマンドの主な目的は、構文の検証です。

1. 任意のコマンド ライン環境を開きます。 多くのコード エディター (Visual Studio Code など) は、コマンド ライン インターフェイスを備えています。

1. ディレクトリをローカル リポジトリの `samples/integration-testing/src` ディレクトリに変更します。 そこには、単純な Terraform プロジェクトが含まれています。

1. [terraform init](https://www.terraform.io/docs/commands/init.html) を使用して Terraform のデプロイを初期化します。 この手順によって、Azure リソース グループを作成するために必要な Azure モジュールがダウンロードされます。

    ```bash
    terraform init
    ```

1. [terraform validate](https://www.terraform.io/docs/commands/validate.html) を使用してテスト用の Terraform ファイルを検証します。

    ```bash
    terraform validate
    ```

    構成が有効であることを示すメッセージが表示されます。

1. コード エディターで、`main.tf` ファイルを開きます。

1. 5 行目に入力ミスを挿入して、構文を無効にします。 たとえば、`var.location` を `var.loaction` に置き換えます。

1. ファイルを保存します。

1. 検証をもう一度実行します。

    ```bash
    terraform validate
    ```

    今回は、誤りのある行とエラーの説明を示すエラー メッセージが表示されます。

ご覧のように、Terraform によって構成コードの構文の問題が検出されました。 この問題が原因で、構成をデプロイできません。

Terraform ファイルをバージョン管理システムにプッシュする前に、それに対して `terraform validate` を常に実行することをお勧めします。 また、このレベルの検証は、継続的インテグレーション パイプラインの一部として行う必要があります。 このチュートリアルの後半では、[自動的に検証するように Azure パイプラインを構成](#automate-integration-tests-using-azure-pipeline)する方法について説明します。

## <a name="validate-terraform-configuration-can-be-deployed-on-azure"></a>Terraform の構成を Azure にデプロイできるかどうかを検証する

前のセクションでは、Terraform の構成を検証する方法を確認しました。 このレベルのテストは、構文のみに対するものです。 そのテストでは、既に Azure にデプロイされているものは考慮されませんでした。

Terraform は "*宣言型言語*" です。つまり、最終的な結果として何が必要かを宣言します。 たとえば、1 つのリソース グループに 10 台の仮想マシンがあるとします。 この場合に、3 つの仮想マシンを定義する Terraform ファイルを作成します。 このプランを適用しても、合計数は 13 に増えません。 代わりに、Terraform によって仮想マシンのうちの 7 台が削除され、最終的に 3 台になります。 `terraform plan` を実行すると、実行プランを適用したときの想定される結果を確認し、想定外の事態を避けることができます。

Terraform の実行プランを生成するには、[terraform plan](https://www.terraform.io/docs/commands/plan.html) を実行します。 このコマンドは、ターゲットの Azure サブスクリプションに接続して、構成のどの部分が既にデプロイされているかを確認します。 その後、Terraform によって、Terraform ファイルに記載されている要件を満たすために必要な変更が 特定されます。 この段階では、Terraform によって何もデプロイされていません。 プランを適用した場合に何が生じるかが示されます。

チュートリアルに従って前のセクションの手順を完了している場合は、`terraform plan` コマンドを実行します。

```bash
terraform plan
```

`terraform plan` を実行すると、実行プランを適用した場合に想定される結果が Terraform によって表示されます。 出力には、追加、変更、破棄される Azure リソースが示されます。

既定では、Terraform により、Terraform ファイルと同じローカル ディレクトリに状態が格納されます。 このパターンは、シングル ユーザーのシナリオに適しています。 しかし、複数のユーザーが同じ Azure リソースで作業すると、ローカルの状態ファイルが同期されない場合があります。この問題を解決するために、Terraform ではリモート データ ストア (Azure Storage など) への状態ファイルの書き込みがサポートされています。 このシナリオでは、`terraform plan` をローカル コンピューターで実行し、リモート マシンをターゲットにすると問題が発生する可能性があります。 そのため、[この検証ステップを継続的インテグレーション パイプラインの一環として自動化](#automate-integration-tests-using-azure-pipeline)することが適切であると考えられます。

## <a name="run-static-code-analysis"></a>静的コード分析を実行する

静的コード分析は Terraform 構成コードで直接実施でき、それを実行する必要はありません。 この分析は、セキュリティの問題やコンプライアンスの不整合などの問題の検出に役立ちます。

次のツールによって、Terraform ファイルの静的分析を行うことができます。

- [Checkov](https://github.com/bridgecrewio/checkov/)
- [Terrascan](https://github.com/accurics/terrascan)
- [tfsec](https://github.com/tfsec/tfsec)
- [Deepsource](https://deepsource.io/blog/release-terraform-static-analysis/) 

静的分析は、多くの場合、継続的インテグレーション パイプラインの一環として実行されます。 これらのテストでは、実行プランまたはデプロイを作成する必要ありません。 そのため、これらは他のテストより短時間で実行され、通常は継続的インテグレーション プロセスで最初に実行されます。

## <a name="automate-integration-tests-using-azure-pipeline"></a>Azure パイプラインを使用して統合テストを自動化する

継続的インテグレーションには、変更の導入時におけるシステム全体のテストが含まれています。 このセクションでは、継続的インテグレーションの実装に使用される Azure パイプラインの構成について説明します。

1. 任意のエディターを使用して、[GitHub の Terraform サンプル プロジェクト](https://github.com/Azure/terraform)のローカルの複製を参照します。

1. `samples/integration-testing/src/azure-pipeline.yaml` ファイルを開きます。

1. **steps** セクションまで下方向にスクロールすると、さまざまなインストールと検証ルーチンの実行に使用される標準的なステップのセットが表示されます。

1. **Step 1: run the Checkov Static Code Analysis** という行を確認します。 このステップでは、前に説明した `Checkov` プロジェクトで、サンプルの Terraform の構成に対して静的コード分析を実行します。 

    ```yaml
    - bash: $(terraformWorkingDirectory)/checkov.sh $(terraformWorkingDirectory)
      displayName: Checkov Static Code Analysis
    ```
    
    注:
    
    - このスクリプトによって、Docker コンテナー内にマウントされた Terraform ワークスペースで Checkov が実行されます。 Microsoft が管理するエージェントでは Docker が有効になっています。 Docker コンテナー内でツールを実行する方が簡単であり、Azure パイプライン エージェントに Checkov をインストールする必要がなくなります。
    - `$(terraformWorkingDirectory)` 変数は、`azure-pipeline.yaml` ファイルで定義されています。

1. **Step 2: install Terraform on the Azure Pipelines agent** という行を確認します。 前にインストールした [Terraform の Build & Release Task 拡張機能](https://marketplace.visualstudio.com/items?itemName=charleszipp.azure-pipelines-tasks-terraform)には、Azure パイプラインを実行しているエージェントに Terraform をインストールするためのコマンドがあります。 このタスクは、このステップで実行されているものです。

    ```yaml
    - task: charleszipp.azure-pipelines-tasks-terraform.azure-pipelines-tasks-terraform-installer.TerraformInstaller@0
      displayName: 'Install Terraform'
      inputs:
        terraformVersion: $(terraformVersion)
    ```
    
    注:

    - インストールする Terraform のバージョンは、`terraformVersion` という名前の Azure パイプライン変数を使用して指定し、`azure-pipeline.yaml` ファイルで定義します。

1. **Step 3: run Terraform init to initialize the workspace** という行を確認します。 Terraform がエージェントにインストールされたので、Terraform ディレクトリを初期化できます。

    ```yaml
    - task: charleszipp.azure-pipelines-tasks-terraform.azure-pipelines-tasks-terraform-cli.TerraformCLI@0
      displayName: 'Run terraform init'
      inputs:
        command: init
        workingDirectory: $(terraformWorkingDirectory)
    ```
    
    メモ:

    - `command` の入力では、実行する Terraform コマンドを指定します。
    - `workingDirectory` の入力は、Terraform ディレクトリのパスを示します。
    - `$(terraformWorkingDirectory)` 変数は、`azure-pipeline.yaml` ファイルで定義されています。

1. **Step 4: run Terraform validate to validate HCL syntax** という行を確認します。 プロジェクト ディレクトリが初期化されると、サーバーの構成を検証するために `terraform validate` が実行されます。

    ```yaml
    - task: charleszipp.azure-pipelines-tasks-terraform.azure-pipelines-tasks-terraform-cli.TerraformCLI@0
      displayName: 'Run terraform validate'
      inputs:
        command: validate
        workingDirectory: $(terraformWorkingDirectory)
    ```
    
1. **Step 5: run Terraform plan to validate HCL syntax** という行を確認します。 先ほど説明したように、実行プランの生成は、デプロイ前に Terraform の構成が有効かどうかを確認するために行います。

    ```yaml
    - task: charleszipp.azure-pipelines-tasks-terraform.azure-pipelines-tasks-terraform-cli.TerraformCLI@0
      displayName: 'Run terraform plan'
      inputs:
        command: plan
        workingDirectory: $(terraformWorkingDirectory)
        environmentServiceName: $(serviceConnection)
        commandOptions: -var location=$(azureLocation)
    ```
    
    メモ:

    - `environmentServiceName` の入力は、「[前提条件](#prerequisites)」で作成された Azure サービスの接続の名前を示します。 この接続を使用して、Terraform がユーザーの Azure サブスクリプションにアクセスできます。
    - `commandOptions` の入力は、Terraform コマンドに引数を渡すために使用されます。 この場合、場所が指定されています。 `$(azureLocation)` 変数は、前に YAML ファイルで定義されています。

### <a name="import-the-pipeline-into-azure-devops"></a>Azure DevOps にパイプラインをインポートする

1. Azure DevOps プロジェクトを開き、Azure Pipelines のセクションに進みます。
 
1. **[パイプラインを作成]** ボタンを選択します。

1. **[コードはどこにありますか?]** オプションで、 **[GitHub (YAML)]** を選択します。

    ![コードはどこにありますか?](media/best-practices-integration-testing/new-pipeline-where-github-yaml.png)

1. この時点で、Azure DevOps に、ご自身の組織へのアクセスを承認することが必要になる可能性があります。 このトピックの詳細については、記事「[GitHub リポジトリの構築](/azure/devops/pipelines/repos/github)」を参照してください。

1. リポジトリの一覧で、GitHub 組織で作成したリポジトリのフォークを選択します。

1. **[パイプラインを構成する]** ステップで、既存の YAML パイプラインから開始することを選択します。

    ![既存の YAML パイプライン](media/best-practices-integration-testing/new-pipeline-existing-yaml.png)

1. **[Select existing YAML pipeline]\(既存の YAML パイプラインを選択する\)** ページが表示されたら、ブランチ `master` を指定し、YAML パイプラインへのパス (`samples/integration-testing/src/azure-pipeline.yaml`) を入力します。

    ![既存の YAML パイプラインを選択する](media/best-practices-integration-testing/select-existing-yaml-pipeline.png)

1. **[続ける]** を選択して、GitHub から Azure YAML パイプラインを読み込みます。

1. **[パイプライン YAML をレビューする]** ページが表示されたら、 **[実行]** を選択して、初めてパイプラインを作成し、手動でトリガーします。

    ![Azure パイプラインを実行する](media/best-practices-integration-testing/run-pipeline.png)

### <a name="run-the-pipeline"></a>パイプラインを実行する

パイプラインは Azure DevOps UI から手動で実行できます。 ただし、この記事の要点は、自動化された継続的インテグレーションを示すことです。 フォークされたリポジトリの `samples/integration-testing/src` フォルダーへの変更をコミットして、プロセスをテストします。 この変更により、コードをプッシュしているブランチで新しいパイプラインが自動的にトリガーされるようになります。

![GitHub から実行中のパイプライン](media/best-practices-integration-testing/pipeline-running-from-github.png)

このステップが完了したら、Azure DevOps の詳細にアクセスして、すべてが正しく実行されていることを確認します。

![Azure DevOps の緑色のパイプライン](media/best-practices-integration-testing/azure-devops-green-pipeline.png)

[!INCLUDE [terraform-troubleshooting.md](includes/terraform-troubleshooting.md)]

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [Terraform プロジェクトでのコンプライアンス テストの作成と実行](best-practices-compliance-testing.md)
