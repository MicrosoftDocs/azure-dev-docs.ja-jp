---
title: Azure で Terraform を使用する
description: Terraform を使用して、Azure にインフラストラクチャをデプロイし、バージョンを管理する方法について説明します。
ms.topic: overview
ms.date: 10/26/2019
ms.custom: devx-track-terraform
adobe-target: true
ms.openlocfilehash: 0857586db7fdac0f4decd515c3a6a71026d2d375
ms.sourcegitcommit: b380f6e637b47e6e3822b364136853e1d342d5cd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/14/2021
ms.locfileid: "100395317"
---
# <a name="terraform-with-azure"></a>Azure での Terraform

[Hashicorp Terraform](https://www.terraform.io/) は、クラウド インフラストラクチャのプロビジョニングおよび管理を行うためのオープン ソース ツールです。 クラウド リソースのトポロジを説明する構成ファイルでインフラストラクチャを体系化します。 これらのリソースには、仮想マシン、ストレージ アカウント、およびネットワーク インターフェイスが含まれます。 Terraform CLI は、Azure に構成ファイルをデプロイしそのバージョンを管理するための簡単なメカニズムを提供します。

この記事では、Terraform を使用して Azure インフラストラクチャを管理する利点について説明します。

## <a name="automate-infrastructure-management"></a>インフラストラクチャの管理を自動化する

Terraform のテンプレート ベースの構成ファイルを使用すると、Azure リソースの定義、プロビジョニング、構成を反復的および計画的に行うことができます。 インフラストラクチャの自動化には、次に示すようないくつかの利点があります。

- インフラストラクチャのデプロイおよび管理中に人為的なミスが発生する可能性が低くなります。
- 同じテンプレートを複数回デプロイして、同一の開発環境、テスト環境、実稼働環境を作成することができます。
- 開発環境およびテスト環境を要求に応じて作成することで、コストを削減できます。

## <a name="understand-infrastructure-changes-before-being-applied"></a>インフラストラクチャの変更を理解してから適用する

リソースのトポロジが複雑になるに従い、インフラストラクチャの変更の意味と、変更が及ぼす影響を理解することが難しくなる場合があります。

Terraform CLI を使用すると、ユーザーは適用の前にインフラストラクチャの変更を検証してプレビューすることができます。 インフラストラクチャの変更を安全にプレビューすることには、次に示すいくつかの利点があります。
- チーム メンバーは、提示されている変更とその影響をすばやく理解することによってより効率的に共同作業ができます。
- 開発プロセスの早い段階で、意図しない変更を捕捉できます。

## <a name="deploy-infrastructure-to-multiple-clouds"></a>複数のクラウドにインフラストラクチャをデプロイする

Terraform は、複数のクラウド プロバイダーへのインフラストラクチャのデプロイに向いています。 開発者による一貫したツールを使用して各インフラストラクチャ定義の管理を可能にします。

## <a name="next-steps"></a>次のステップ

これで Terraform の概要と利点に関する説明は終了です。推奨する次の手順は、以下のとおりです。

環境に応じて、Terraform をインストールして構成します。

- [Azure Cloud Shell および Azure CLI を使用して Terraform を構成する](get-started-cloud-shell.md)
- [Azure PowerShell を使用して Terraform を構成する](get-started-powershell.md)
