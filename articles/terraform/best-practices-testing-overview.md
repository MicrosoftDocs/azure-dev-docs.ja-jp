---
title: チュートリアル - Terraform のテストの概要
description: Terraform プロジェクトを検証するために構成できる、さまざまなテスト オプションについて説明します。
ms.topic: overview
ms.date: 07/31/2020
ms.custom: devx-track-terraform
adobe-target: true
ms.openlocfilehash: ede7fdfc1dafedacfdc74f657fee0a4ae11feb85
ms.sourcegitcommit: b380f6e637b47e6e3822b364136853e1d342d5cd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/14/2021
ms.locfileid: "100395327"
---
# <a name="tutorial-terraform-testing-overview"></a>チュートリアル:Terraform のテストの概要

[!INCLUDE [terraform-intro.md](includes/terraform-intro.md)]

Terraform は、Infrastructure as Code (IaC) ツールです。 このカテゴリのツールは、Terraform ファイルをプロジェクトのソース コードと同じように扱うことを意味します。 そのプロセスの一部として、バージョン管理とソース コード管理が含まれます。 また、テストもプロセスに組み込む必要があります。 この記事では、Terraform プロジェクトに対して実行できる、さまざまな種類のテストの概要を示します。

## <a name="integration-testing"></a>統合テスト

統合テストでは、新しく導入されたコード変更によって既存のコードが破損しないことが検証されます。 DevOps の場合、継続的インテグレーション (CI) とは、コード ベースが変更される (たとえば、PR を Git リポジトリにマージしようとする) たびにシステム全体を構築するプロセスを指します。 次の一覧には、統合テストの一般的な例が含まれています。

- lint や format などの静的コード分析ツール。
- [terraform validate](https://www.terraform.io/docs/commands/validate.html) を実行して、構成ファイルの構文を検証します。
- [terraform plan](https://www.terraform.io/docs/commands/validate.html) を実行して、構成が想定どおりに機能することを確認します。

> [!div class="nextstepaction"]
> [統合テストの詳細情報](best-practices-integration-testing.md)

## <a name="unit-testing"></a>単体テスト

単体テストでは、プログラムの特定の部分または機能が正しく動作することを確認します。 単体テストは、機能の開発者によって記述されます。 この種類のテストは、テスト駆動開発 (TDD) と呼ばれることもあり、連続する短い開発サイクルを伴います。 Terraform プロジェクトのコンテキストでは、単体テストは `terraform plan` を使用する形式をとって、生成されたプランで得られる実際の値が想定していた値と等しいことを確認できます。 

単体テストは、Terraform モジュールが複雑になり始めたときに特に役立つ可能性があります。

- 動的ブロックの生成
- ループの使用
- ローカル変数の計算

統合テストと同様に、多くの場合、単体テストは継続的インテグレーション プロセスに含まれています。

## <a name="compliance-testing"></a>コンプライアンス テスト

コンプライアンス テストは、プロジェクトに対して定義したポリシーに構成が従っていることを確認するために使用します。 たとえば、Azure リソースに地理的な名前付け規則を定義する場合があります。 または、定義したイメージのサブセットから仮想マシンを作成する場合もあります。 コンプライアンス テストは、これらの規則を適用するために使用します。

また、コンプライアンス テストは通常、継続的インテグレーション プロセスの一環として定義します。

> [!div class="nextstepaction"]
> [コンプライアンス テストの詳細情報](best-practices-compliance-testing.md)

## <a name="end-to-end-e2e-testing"></a>エンド ツー エンド (E2E) テスト

E2E テストでは、運用環境にデプロイする前に、プログラムが動作することを検証します。 シナリオの例として、2 つの仮想マシンを 1 つの仮想ネットワークにデプロイする Terraform モジュールがあるとします。 2 つのマシンが互いに ping しないようにしたい場合があります。 この例では、デプロイ前に意図した結果を確認するためのテストを定義できます。

E2E テストは通常、3 段階のプロセスです。 最初に、構成をテスト環境に適用します。 次に、コードを実行して結果を確認します。 最後に、テスト環境を再初期化または停止 (仮想マシンの割り当て解除など) します。

> [!div class="nextstepaction"]
> [エンド ツー エンド テストの詳細情報](best-practices-end-to-end-testing.md)

[!INCLUDE [terraform-troubleshooting.md](includes/terraform-troubleshooting.md)]
