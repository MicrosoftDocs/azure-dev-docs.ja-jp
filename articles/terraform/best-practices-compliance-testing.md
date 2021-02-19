---
title: チュートリアル - Terraform と Azure を使用したコンプライアンス テスト
description: 振る舞い駆動開発 (BDD) スタイルのコンプライアンス テストを Terraform の構成に適用する方法について説明します
ms.topic: tutorial
ms.date: 07/31/2020
ms.custom: devx-track-terraform
ms.openlocfilehash: d19c1cfa2c794910d65f98ba71f108e478b1c52b
ms.sourcegitcommit: b19420d1a8d526a81e0835012cccc46717fadd69
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/15/2021
ms.locfileid: "100524887"
---
# <a name="tutorial-compliance-testing-with-terraform-and-azure"></a>チュートリアル:Terraform と Azure を使用したコンプライアンス テスト

[!INCLUDE [terraform-intro.md](includes/terraform-intro.md)]

この記事では、次のタスクを行う方法について説明します。

> [!div class="checklist"]
> * コンプライアンス テストを使用する状況を理解する
> * コンプライアンス チェックの実行方法について学習する

## <a name="prerequisites"></a>前提条件

[!INCLUDE [open-source-devops-prereqs-azure-subscription.md](../includes/open-source-devops-prereqs-azure-subscription.md)]
- **Terraform のインストール**:ご使用の環境に応じて、[Terraform をダウンロードしてインストール](https://www.terraform.io/downloads.html)します。
- **Docker**:[Docker のインストール](https://docs.docker.com/get-docker/)
- **Terraform-compliance**:[Terraform-compliance ツールをインストール](https://terraform-compliance.com/pages/installation/docker)します。
- **テスト用サンプルをフォークする**:[GitHub の Terraform サンプル プロジェクト](https://github.com/Azure/terraform)をフォークし、開発/テスト用コンピューターに複製します。

## <a name="what-is-compliance-testing"></a>コンプライアンス テストとは

コンプライアンス テストは、システムが規定の標準を満たしているかどうかを判定するための非機能テスト手法です。 コンプライアンス テストは "*準拠合致テスト*" とも呼ばれています。

ほとんどのソフトウェア チームは、標準が適切に適用および実装されていることを確認するための分析を行います。 多くの場合、並行して標準の改善に取り組むことで、結果的に品質が向上します。

コンプライアンス テストにより、各開発ライフサイクル フェーズの出力が、合意された要件に準拠していることが確認されます。

コンプライアンス チェックは、プロジェクトの開始時に開発サイクルに組み込む必要があります。 要件自体が適切に文書化されていない場合、後の段階でコンプライアンス チェックを追加しようとすると、その作業がますます難しくなります。

## <a name="understanding-compliance-checks"></a>コンプライアンス チェックについて

コンプライアンス チェックは簡単に実行できます。 開発ライフサイクルのフェーズごとに、一連の標準と手順が作成され、文書化されています。 各フェーズの出力が、文書化された要件と比較されます。 テストの結果として、事前に定義された標準に準拠していないという点での "ギャップ" が示されます。 コンプライアンス テストは検査プロセスを通じて実施され、レビュー プロセスの結果は文書化する必要があります。

具体的な例を見てみましょう。

一般的な問題として、複数の開発者が相容れない変更を適用した場合に環境が中断するという問題があります。 ある人が変更に取り組んで、リソースを適用している (たとえば、テスト環境で VM を作成している) とします。 その後に、別の人が、その VM の別のバージョンをプロビジョニングする別のバージョンのコードを適用します。 ここで必要なことは、規定された規則への準拠を確認するための監視です。

この問題に対処する 1 つの方法は、`role` や `creator` タグなどで、リソースにタグ付けするポリシーを定義することです。 ポリシーを定義したら、[Terraform-compliance](https://terraform-compliance.com) のようなツールを使用して、ポリシーが守られていることを確認します。

Terraform-compliance は "*ネガティブ テスト*" に重点を置いています。 ネガティブ テストは、予期しない入力や望ましくない動作をシステムが適切に処理できるようにするプロセスです。 "*ファジー テスト*" は、ネガティブ テストの 1 つの例です。 ファジー テストを使用すると、入力を受け取るシステムがテストされ、それが予期しない入力を安全に処理できることが確認されます。

幸い、Terraform は、クラウド インフラストラクチャ エンティティを作成、更新、または破棄する API の抽象化レイヤーです。 また Terraform を使用すると、ローカル構成とリモート API 応答が確実に同期されます。Terraform は主にクラウド API に対して使用されるため、インフラストラクチャに対してデプロイされたコードが特定のポリシーに従っていることを確認する手段がやはり必要です。 Terraform-compliance (無料のオープンソース ツール) では、Terraform の構成にこの機能が提供されます。

VM の例を使用すると、コンプライアンス ポリシーは次のようになります。"*Azure リソースを作成している場合、それにタグを含める必要があります*"。

Terraform-compliance ツールには、この例のようなポリシーを作成するテスト フレームワークが用意されています。 次に、それらのポリシーを Terraform 実行プランに対して実行します。

Terraform-compliance を使用すると、BDD、つまり "*振る舞い駆動開発*" の原則を適用できます。 BDD は、すべての関係者が連携してシステムの動作を定義するコラボレーション プロセスです。 これらの関係者には、通常、開発者、テスト担当者、および開発中のシステムに既得権を持つ (またはこのシステムによる影響を受ける) ユーザーが含まれます。 BDD の目的は、システムの動作についての共通の理解を示す具体的な例をチームが構築できるようにすることです。

## <a name="looking-at-an-example"></a>例を確認する

この記事では先ほど、テスト環境用の VM の作成でのコンプライアンス テストの例について説明しました。 このセクションでは、その例を BDD の機能とシナリオに変換する方法について説明します。 この規則は、最初に *Cucumber* (BDD をサポートするために使用するツール) を使用して表現します。

```Cucumber
when creating Azure resources, every new resource should have a tag
```

先ほどの規則は次のように変換されます。

```Cucumber
If the resource supports tags
Then it must contain a tag
And its value must not be null
```

Terraform HCL コードは、次のようにこの規則に従います。

```hcl
resource "random_uuid" "uuid" {}

resource "azurerm_resource_group" "rg" {
  name     = "rg-hello-tf-${random_uuid.uuid.result}"
  location = var.location

  tags = {
    environment = "dev"
    application = "Azure Compliance"
  } 
}
```

最初のポリシーは、次のように [BDD 機能のシナリオ](https://cucumber.io/docs/gherkin/reference/)として記述できます。

```Cucumber
Feature: Test tagging compliance  # /target/src/features/tagging.feature
    Scenario: Ensure all resources have tags
        If the resource supports tags
        Then it must contain a tag
        And its value must not be null
```

次のコードは、特定のタグのテストを示しています。

```Cucumber
Scenario Outline: Ensure that specific tags are defined
    If the resource supports tags
    Then it must contain a tag <tags>
    And its value must match the "<value>" regex

    Examples:
      | tags        | value              |
      | Creator     | .+                 |
      | Application | .+                 |
      | Role        | .+                 |
      | Environment | ^(prod\|uat\|dev)$ |
```

## <a name="running-the-sample"></a>サンプルの実行

このセクションでは、サンプルをダウンロードしてテストします。

1. [コンプライアンス テストのサンプルをダウンロード](https://github.com/Azure/terraform/tree/master/samples/compliance-testing)します。

1. ディレクトリを `src` ディレクトリに変更します。

1. [terraform init](https://www.terraform.io/docs/commands/init.html) を実行して、作業ディレクトリを初期化します。 この手順によって、Azure リソース グループを作成するために必要な Azure モジュールがダウンロードされます。

    ```bash
    terraform init
    ```
    
1. [terraform validate](https://www.terraform.io/docs/commands/validate.html) を実行して、構成ファイルの構文を検証します。

    ```bash
    terraform validate
    ```
    
1. [terraform plan](https://www.terraform.io/docs/commands/plan.html) を実行して、実行プランを作成します。

    ```bash
    terraform plan -out tf.out
    ```
    
1. [terraform apply](https://www.terraform.io/docs/commands/apply.html) を実行して、実行プランを適用します。

    ```bash
    terraform apply -target=random_uuid.uuid
    ```
    
1. [docker pull](https://docs.docker.com/engine/reference/commandline/pull/) を実行して、terraform-compliance のイメージをダウンロードします。

    ```bash
    docker pull eerkunt/terraform-compliance
    ```
    
1. [docker run](https://docs.docker.com/engine/reference/commandline/run/) を実行して、Docker コンテナーでテストを実行します。 **テストは失敗します**。 タグの存在を必要とする最初の規則では成功します。 しかし、2 番目の規則では、`Role` と `Creator` タグが欠落しているという点で失敗します。

    ```bash
    docker run --rm -v $PWD:/target -it eerkunt/terraform-compliance -f features -p tf.out
    ```
    
    ![失敗したテストの例](media/best-practices-compliance-testing/best-practices-compliance-testing-tagging-fail.png)

1. 次のように `main.tf` を変更して、エラーを修正します。

    ```terraform
      tags = {
        Environment = "dev"
        Application = "Azure Compliance"
        Creator     = "Azure Compliance"
        Role        = "Azure Compliance"
      } 
    
    ```
    
1. `terraform validate` を再度実行して構文を検証します。

    ```bash
    terraform validate
    ```
    
1. `terraform plan` を再度実行して、新しい実行プランを作成します。

    ```bash
    terraform plan -out tf.out
    ```
    
1. [docker run](https://docs.docker.com/engine/reference/commandline/run/) を再度実行して、構成をテストします。 今回は、完全な仕様が実装されているため、テストは成功します。

    ```bash
    docker run --rm -v $PWD:/target -it eerkunt/terraform-compliance -f features -p tf.out
    ```

    ![成功したテストの例](media/best-practices-compliance-testing/best-practices-compliance-testing-tagging-succeed.png)

[!INCLUDE [terraform-troubleshooting.md](includes/terraform-troubleshooting.md)]

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [Terraform プロジェクトでのエンド ツー エンド テストの作成と実行](best-practices-end-to-end-testing.md)
