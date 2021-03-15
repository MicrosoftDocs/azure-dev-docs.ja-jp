---
title: チュートリアル - Terraform を使用して Azure でハブ スポーク ハイブリッド ネットワーク トポロジを作成する
description: Azure で Terraform を使用して、ハイブリッド ネットワーク参照アーキテクチャ全体を作成する方法について説明します。
ms.topic: tutorial
ms.date: 03/08/2021
ms.custom: devx-track-terraform
ms.openlocfilehash: 13c5d1a12ad0e2cf03c84c34c97ffac0a5d575bd
ms.sourcegitcommit: 7991f748720673d2dc50baaa8658348ff6cc1044
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102604153"
---
# <a name="tutorial-create-a-hub-and-spoke-hybrid-network-topology-in-azure-using-terraform"></a>チュートリアル: Terraform を使用して Azure でハブ スポーク ハイブリッド ネットワーク トポロジを作成する

このチュートリアル シリーズでは、Azure で Terraform を使用して、[ハブ スポーク ネットワーク トポロジ](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke)を実装する方法を示します。 

ハブ スポーク トポロジは、一般的なサービスを共有しながらワークロードを分離する方法です。 これらのサービスには、ID とセキュリティが含まれます。 ハブは、オンプレミス ネットワークへの中心となる接続ポイントとして機能する仮想ネットワーク (VNet) です。 スポークはハブとピア接続する Vnet です。 共有サービスはハブにデプロイされ、個々のワークロードはスポーク ネットワークにデプロイされます。

このチュートリアルに含まれるタスクは次のとおりです。

> [!div class="checklist"]
> * HCL (HashiCorp 言語) を使用して、ハブ スポーク ハイブリッド ネットワーク参照アーキテクチャ リソースをレイアウトする
> * Terraform を使用して、ハブ ネットワーク アプライアンス リソースを作成する
> * Terraform を使用して、Azure 上にすべてのリソースの共通ポイントとして機能するハブ ネットワークを作成する
> * Terraform を使用して、Azure 上に個々のワークロードをスポーク VNet として作成する
> * Terraform を使用して、オンプレミス ネットワークと Azure ネットワーク間のゲートウェイと接続を確立する
> * Terraform を使用して、スポーク ネットワークへの VNet ピアリングを作成する

## <a name="prerequisites"></a>前提条件

[!INCLUDE [open-source-devops-prereqs-azure-subscription.md](../includes/open-source-devops-prereqs-azure-subscription.md)]

- **Terraform のインストールと構成**: VM などのインフラストラクチャを Azure にプロビジョニングするための [Terraform のインストールと構成](get-started-cloud-shell.md)

## <a name="hub-and-spoke-topology-architecture"></a>ハブ スポーク トポロジ アーキテクチャ

ハブ スポーク トポロジでは、ハブは VNet になります。 VNet は、オンプレミス ネットワークの中心となる接続ポイントとして機能します。 スポークは、ハブに対して配置される VNet であり、ワークロードを分離するために使用されます。 トラフィックは、ExpressRoute または VPN ゲートウェイ接続を経由してオンプレミスのデータセンターとハブの間を流れます。 次の図は、ハブ スポーク トポロジのコンポーネントを示しています。

![Azure 上のハブ スポーク トポロジ アーキテクチャ](./media/hub-and-spoke-tutorial-series/hub-spoke-architecture.png)

## <a name="benefits-of-the-hub-and-spoke-topology"></a>ハブ スポーク トポロジの利点

ハブ スポーク ネットワーク トポロジは、一般的なサービスを共有しながらワークロードを分離する方法です。 これらのサービスには、ID とセキュリティが含まれます。 ハブは、オンプレミス ネットワークへの中心となる接続ポイントとして機能する VNet です。 スポークはハブとピア接続する Vnet です。 共有サービスはハブにデプロイされ、個々のワークロードはスポーク ネットワークにデプロイされます。 ハブ スポーク ネットワーク トポロジのいくつかの利点を次に示します。

- **コストの削減**: 複数のワークロードによって共有できる単一の場所にサービスを一元化します。 これらのワークロードには、ネットワーク仮想アプライアンスと DNS サーバーが含まれます。
- **サブスクリプション数の上限の解消**: 中央のハブにささまざまなサブスクリプションの VNet をピアリングします。
- **問題の分離**: 中央の IT (SecOps、InfraOps) とワークロード (DevOps) の間で実施します。

## <a name="typical-uses-for-the-hub-and-spoke-architecture"></a>ハブ スポーク アーキテクチャの一般的な用途

ハブ スポーク アーキテクチャの一般的な用途の一部を次に示します。

- 多くのお客様には、さまざまな環境にデプロイされているワークロードがあります。 これらの環境には、開発、テスト、および運用環境が含まれます。 多くの場合、これらのワークロードでは、DNS、IDS、NTP、AD DS などのサービスを共有する必要があります。 これらの共有サービスをハブ VNet に配置できます。 この方法で、各環境をスポークにデプロイして、分離を維持します。
- 相互接続は必要ないが、共有サービスへのアクセスは必要なワークロード。
- セキュリティ面の一元管理を必要とする企業。
- 各スポークでワークロードの分別管理を必要とする企業。

## <a name="preview-the-demo-components"></a>デモ コンポーネントのプレビュー

このシリーズのチュートリアルを進めていくときに、さまざまなコンポーネントを個別の Terraform スクリプトで定義します。 作成してデプロイするデモ アーキテクチャは、次のコンポーネントで構成されます。

- **オンプレミス ネットワーク**。 組織内で運用されているプライベート ローカル エリア ネットワーク。 ハブ スポーク参照アーキテクチャでは、Azure 上の VNet を使用してオンプレミス ネットワークがシミュレートされます。

- **VPN デバイス**: VPN デバイスまたはサービスによって、オンプレミス ネットワークへの外部接続が提供されます。 VPN デバイスは、ハードウェア アプライアンスまたはソフトウェア ソリューションが可能です。 

- **ハブ VNet**: ハブは、オンプレミス ネットワークの中心となる接続ポイントであり、サービスをホストする場所です。 スポーク VNet でホストされているさまざまなワークロードで、これらのサービスを使用できます。

- **ゲートウェイ サブネット**。 VNet ゲートウェイは、同じサブネット内に保持されます。

- **スポーク VNet**: スポークを使用すると、独自の VNet にワークロードを分離して、他のスポークとは別に管理できます。 各ワークロードには複数の階層が含まれる場合があります。これらの階層には、Azure ロード バランサーを使用して接続されている複数のサブネットがあります。 

- **VNet ピアリング**: ピアリング接続を使用して、2 つの VNet を接続できます。 ピアリング接続は、VNet 間の非推移的な低遅延の接続です。 ピアリングが完了すると、VNet では、ルーターがなくても Azure バックボーンを使用してトラフィックを交換します。 ハブ スポーク ネットワーク トポロジでは、VNet ピアリングを使用して、ハブを各スポークに接続します。 同じリージョンまたは異なるリージョンの VNet をピアリングできます。

## <a name="create-the-directory-structure"></a>ディレクトリ構造を作成する

デモ用の Terraform 構成ファイルを保持するディレクトリを作成します。

1. [Azure ポータル](https://portal.azure.com)にアクセスします。

1. [Azure Cloud Shell](/azure/cloud-shell/overview) を開きます。 前に環境を選択しなかった場合、環境として **Bash** を選択します。

    ![Cloud Shell のプロンプト](./media/common/azure-portal-cloud-shell-button-min.png)

1. ディレクトリを `clouddrive` ディレクトリに変更します。

    ```bash
    cd clouddrive
    ```

1. `hub-spoke` という名前のディレクトリを作成します。

    ```bash
    mkdir hub-spoke
    ```

1. 新しいディレクトリに移動します。

    ```bash
    cd hub-spoke
    ```

## <a name="declare-the-azure-provider"></a>Azure プロバイダーを宣言する

Azure プロバイダーを宣言する Terraform 構成ファイルを作成します。

1. Cloud Shell で、`main.tf` という名前の新しいファイルを開きます。

    ```bash
    code main.tf
    ```

1. 以下のコードをエディターに貼り付けます。

    ```hcl
    terraform {
      required_providers {
          azurerm = {
            source  = "hashicorp/azurerm"
            version = "~>2.0"
          }
      }
    }
    provider "azurerm" {
      features {}
    }
    ```

1. ファイルを保存し、エディターを終了します。

## <a name="create-the-variables-file"></a>変数ファイルを作成する

さまざまなスクリプトで使用される共通変数用の Terraform 構成ファイルを作成します。

1. Cloud Shell で、`variables.tf` という名前の新しいファイルを開きます。

    ```bash
    code variables.tf
    ```

1. 以下のコードをエディターに貼り付けます。

    ```hcl
    variable "location" {
      description = "Location of the network"
      default     = "eastus"
    }
    
    variable "username" {
      description = "Username for Virtual Machines"
      default     = "testadmin"
    }
    
    variable "password" {
      description = "Password for Virtual Machines"
      default     = "Password1234!"
    }
    
    variable "vmsize" {
      description = "Size of the VMs"
      default     = "Standard_DS1_v2"
    }
    ```

    **メモ**: このチュートリアルでは、わかりやすくするために、変数ファイルにハードコーディングされたパスワードを使用します。 実際のアプリでは、SSH 公開および秘密キーのペアを使用することを検討してください。 SSH キーと Azure の詳細については、「[Azure 上の Windows で SSH キーを使用する方法](/azure/virtual-machines/linux/ssh-from-windows)」を参照してください。

1. ファイルを保存し、エディターを終了します。

[!INCLUDE [terraform-troubleshooting.md](includes/terraform-troubleshooting.md)]

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"] 
> [Azure で Terraform を使用してオンプレミス仮想ネットワークを作成する](./hub-spoke-on-prem.md)