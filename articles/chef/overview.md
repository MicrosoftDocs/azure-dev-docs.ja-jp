---
title: Azure で Chef を使用する
description: Chef を使用した Azure インフラストラクチャの構成およびテストの概要
keywords: azure、chef、devops、仮想マシン、概要、自動化
ms.date: 02/22/2020
ms.topic: article
ms.openlocfilehash: 94cfe5d57673db151d53133fb8b2ef66c5629b93
ms.sourcegitcommit: a32ca0946275165ce24216c6fa243ec21d6c9193
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/08/2020
ms.locfileid: "80892991"
---
# <a name="using-chef-with-azure"></a>Azure で Chef を使用する

[Chef](https://www.chef.io) は、Azure 上の仮想マシン インフラストラクチャをコードに変換する強力な自動化プラットフォームです。 Chef は、インフラストラクチャがそのサイズに関係なく、ネットワーク経由で構成、デプロイ、および管理される方法を自動化します。

この記事では、Chef を使用して Azure インフラストラクチャを管理する利点について説明します。

## <a name="chef-extension-on-azure"></a>Azure での Chef 拡張機能

[Azure portal](https://go.microsoft.com/fwlink/p/?LinkID=525040) 上の [Chef 拡張機能](https://docs.microsoft.com/azure/chef/chef-extension-portal)でバックグラウンド サービスとして実行されている Chef Client を使用して仮想マシンをプロビジョニングします。 プロビジョニングが済むと、これらの仮想マシンは Chef サーバーで管理できるようになります。

## <a name="chef-cloud-shell"></a>Chef Cloud Shell

Azure Cloud Shell で Chef ワークステーションを直接使用します。 すべての Chef ユーティリティと InSpec を Cloud Shell から実行します。 以下から Chef コマンドを使用できます。

* [chef](https://docs.chef.io/ctl_chef.html)
* [kitchen](https://docs.chef.io/ctl_kitchen.html)
* [inspec](https://www.inspec.io/docs/reference/cli/)
* [knife](https://docs.chef.io/knife.html)
* [cookstyle](https://docs.chef.io/cookstyle.html)
* [chef-run](https://www.chef.sh/docs/chef-workstation/getting-started/)

弊社のコマンド ユーティリティと、Cloud Shell で使用できる `git`、`az-cli`、`terraform` などの他のツールを組み合わせて、ブラウザーからインフラストラクチャとコンプライアンスの自動化を記述します。

## <a name="automate-everything-with-one-platform"></a>1 つのプラットフォームですべてを自動化

企業がデジタル マーケットプレースで競争するには、スピード、ベロシティ、および安全性が必要です。 Chef と Microsoft は連携して、個人、チーム、および企業がこれらすべてを達成できるように支援します。 Chef Automate という 1 つのプラットフォームを使用することで、インフラストラクチャ、アプリケーション、およびお持ちの Microsoft 資産全体のコンプライアンスを自動化し、継続的に提供することができるようになりました。

## <a name="test-drive-chef-automate-on-azure"></a>Azure の Chef Automate の体験版

Chef でサポートされている [Chef Automate Azure Marketplace ソリューション](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/chef-software.chef-automate)を使用すると、共同作業でインフラストラクチャおよびアプリケーションのビルド、デプロイ、管理ができます。 クリック 1 回で、Chef Automate 付属のすべての商用機能へ迅速にアクセスしたり、資産全体をエンドツーエンドで可視化したり、継続的なコンプライアンスを実現したり、統合ワークフローによりすべての変更を管理することが可能です。

## <a name="next-steps"></a>次のステップ

* [Chef を使用して Windows 仮想マシンを Azure 上に作成する](windows-vm-configure.md)
