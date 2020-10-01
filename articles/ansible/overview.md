---
title: Azure での Ansible の使用
description: Ansible を使用してクラウド プロビジョニング、構成管理、およびアプリケーションのデプロイを自動化する方法の概要。
keywords: Ansible, Azure, DevOps, 概要, クラウド プロビジョニング, 構成管理, アプリケーションのデプロイ, Ansible モジュール, Ansible プレイブック
ms.topic: overview
ms.date: 08/13/2020
ms.custom: devx-track-ansible
ms.openlocfilehash: 9eb90921a0d44e138c331eb716700feb85e8aa9d
ms.sourcegitcommit: 717e32b68fc5f4c986f16b2790f4211967c0524b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/30/2020
ms.locfileid: "91586103"
---
# <a name="using-ansible-with-azure"></a>Azure での Ansible の使用

[Ansible](https://www.ansible.com) は、クラウド プロビジョニング、構成管理、およびアプリケーションのデプロイを自動化するオープンソース製品です。 Ansible を使用すると、仮想マシン、コンテナー、およびネットワークをプロビジョニングし、クラウド インフラストラクチャを完成させることができます。 また、Ansible を使用すると、環境でのリソースの展開と構成を自動化することもできます。

この記事では、Azure で Ansible を使用するいくつかの利点の基本概要を示します。

## <a name="ansible-playbooks"></a>Ansible プレイブック

[Ansible プレイブック](https://docs.ansible.com/ansible/latest/playbooks.html)を使用すると、環境を構成するように Ansible に指示することができます。 プレイブックは、人間が判読できるように YAML を使用してコード化されています。 「チュートリアル」セクションでは、プレイブックを使用して Azure リソースのインストールと構成を行う多くの例を示しています。 

## <a name="ansible-modules"></a>Ansible モジュール

Ansible には一連の [Ansible モジュール](https://docs.ansible.com/ansible/2.9/modules/modules_by_category.html)が含まれています。これらのモジュールは、リモート ホストで直接実行されるか、[プレイブック](https://docs.ansible.com/ansible/latest/playbooks.html)を介して実行されます。 ユーザーは独自のモジュールを作成できます。 モジュールは、サービス、パッケージ、ファイルなどのシステム リソースの制御や、システム コマンドの実行に使用されます。

Ansible には、Azure サービスとの対話用に一連の [Ansible クラウド モジュール](https://docs.ansible.com/ansible/2.9/modules/list_of_cloud_modules.html#azure)が含まれています。 これらのモジュールを使用すると、Azure でインフラストラクチャを作成して調整できます。 

## <a name="migrate-existing-workload-to-azure"></a>Azure への既存のワークロードの移行

Ansible を使用してインフラストラクチャを定義すると、アプリケーションのプレイブックを適用して、Azure が必要に応じて環境を自動的にスケーリングすることが可能になります。 

## <a name="automate-cloud-native-application-in-azure"></a>Azure のクラウドネイティブ アプリケーションの自動化

Ansible を使用すると、[Azure Functions](https://azure.microsoft.com//services/functions/) や [Kubernetes on Azure](https://azure.microsoft.com/services/container-service/kubernetes/) などの Azure マイクロサービスを使用して Azure のクラウドネイティブ アプリケーションを自動化することができます。  

## <a name="manage-deployments-with-dynamic-inventory"></a>動的インベントリによるデプロイの管理

Ansible では、[動的インベントリ](https://docs.ansible.com/ansible/intro_dynamic_inventory.html)機能を使用して、Azure リソースからインベントリをプルすることができます。 既存の Azure デプロイにタグを付け、タグを付けたデプロイを Ansible で管理することができます。

## <a name="additional-azure-marketplace-options"></a>その他の Azure Marketplace オプション

[Ansible Tower](https://azuremarketplace.microsoft.com/marketplace/apps/redhat.ansible-tower) は、Red Hat による Azure Marketplace イメージです。 

Ansible Tower は、次の機能を備えた、Ansible 用の Web ベースの UI およびダッシュ ボードです。

* ロールベースのアクセス制御、ジョブのスケジュール設定、およびグラフィカルな在庫管理を定義できます。 
* REST API と CLI が組み込まれているため、Tower を既存のツールとプロセスに挿入できます。 
* プレイブック実行のリアルタイムの出力をサポートしています。 
* 資格情報 (Azure キーや SSH キーなど) を暗号化します。そのため、資格情報を公開せずにタスクを委任できます。

## <a name="ansible-module-and-version-matrix-for-azure"></a>Azure の Ansible モジュールとバージョンのマトリックス

Ansible には、Azure リソースのプロビジョニングと構成に使用する一連のモジュールが含まれています。 これらのリソースには、仮想マシン、スケール セット、ネットワーク サービス、コンテナー サービスがあります。 [Ansible のマトリックス](./module-version-matrix.md)に、Azure 用の Ansible モジュールと、それらが搭載される Ansible バージョンの一覧が記載されています。

## <a name="next-steps"></a>次のステップ

- [クイック スタート: Azure Cloud Shell を使用して Ansible を構成する](getting-started-cloud-shell.md)
- [クイック スタート: Azure CLI を使用して Ansible を構成する](install-on-linux-vm.md)
