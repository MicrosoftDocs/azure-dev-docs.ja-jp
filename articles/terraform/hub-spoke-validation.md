---
title: チュートリアル - Terraform を使用して Azure でハブとスポーク ネットワークを検証する
description: すべての仮想ネットワークが相互に接続されているハブとスポークのネットワーク トポロジを検証する方法について説明します。
ms.topic: tutorial
ms.date: 03/08/2021
ms.custom: devx-track-terraform
ms.openlocfilehash: 71d764da80888326be9625a9e90c1aba164b6f55
ms.sourcegitcommit: 7991f748720673d2dc50baaa8658348ff6cc1044
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102604143"
---
# <a name="tutorial-validate-a-hub-and-spoke-network-in-azure-using-terraform"></a>チュートリアル: Terraform を使用して Azure でハブとスポーク ネットワークを検証する

この記事では、このシリーズの前の記事で作成した terraform ファイルを実行します。 その結果は、デモ仮想ネットワーク間の接続の検証です。

このチュートリアルに含まれるタスクは次のとおりです。

> [!div class="checklist"]
> * HCL (HashiCorp 言語) を使用して、ハブ スポーク トポロジでハブ VNet を実装する
> * Terraform plan を使用して、デプロイするリソースを確認する
> * Terraform apply を使用して、Azure にリソースを作成する
> * 異なるネットワークの間の接続を確認する
> * Terraform を使用して、すべてのリソースを破棄する

## <a name="prerequisites"></a>前提条件

1. [Azure で Terraform を使用して、ハブ スポーク ハイブリッド ネットワーク トポロジを作成します](./hub-spoke-introduction.md)。
1. [Azure で Terraform を使用して、オンプレミス仮想ネットワークを作成します](./hub-spoke-on-prem.md)。
1. [Azure で Terraform を使用して、ハブ仮想ネットワークを作成します](./hub-spoke-hub-network.md)。
1. [Azure で Terraform を使用して、ハブ仮想ネットワーク アプライアンスを作成します](./hub-spoke-hub-nva.md)。
1. [Azure で Terraform を使用して、スポーク仮想ネットワークを作成します](./hub-spoke-spoke-network.md)。

## <a name="verify-your-configuration"></a>構成の確認

[前提条件](#prerequisites)の作業が完了したら、適切な構成ファイルが存在することを確認します。

1. [Azure ポータル](https://portal.azure.com)にアクセスします。

1. [Azure Cloud Shell](/azure/cloud-shell/overview) を開きます。 前に環境を選択しなかった場合、環境として **Bash** を選択します。

    ![Cloud Shell のプロンプト](./media/common/azure-portal-cloud-shell-button-min.png)

1. ディレクトリを `clouddrive` ディレクトリに変更します。

    ```bash
    cd clouddrive
    ```

1. 新しいディレクトリに移動します。

    ```bash
    cd hub-spoke
    ```

1. 作業ディレクトリ内のファイルを一覧表示して、前のチュートリアルで作成した `.tf` 構成ファイルが一覧に表示されていることを確認します。 (スクリーンショットには、次の手順で Terraform を初期化するまで使用できない他のファイルが表示されています)。

    ![Terraform デモ構成ファイル](./media/hub-and-spoke-tutorial-series/hub-spoke-config-files.png)

## <a name="deploy-the-resources"></a>リソースのデプロイ

1. Terraform プロバイダーを初期化します。
    
    ```bash
    terraform init
    ```
    
    !["terraform init" コマンドの結果例](./media/hub-and-spoke-tutorial-series/hub-spoke-terraform-init.png)
    
1. 実行する前に、`terraform plan` コマンドを実行して、デプロイの影響を確認します。

    ```bash
    terraform plan
    ```
    
    !["terraform plan" コマンドの結果例](./media/hub-and-spoke-tutorial-series/hub-spoke-terraform-plan.png)

1. ソリューションをデプロイします。

    ```bash
    terraform apply
    ```
    
    デプロイの確認を求められたら、「`yes`」と入力します。

    !["terraform apply" コマンドの結果例](./media/hub-and-spoke-tutorial-series/hub-spoke-terraform-apply.png)
    
## <a name="test-the-hub-vnet-and-spoke-vnets"></a>ハブ VNet とスポーク VNet のテスト

このセクションでは、シミュレートされたオンプレミスの環境からハブ VNet への接続のテスト方法を示します。

1. Azure portal で、**onprem-vnet-rg** リソース グループを参照します。

1. **[onprem-vnet-rg]** タブで、「**onprem-vm**」という名前の VM を選択します。

1. **パブリック IP アドレス** の値をメモします。

1. コマンドラインに戻り、`ssh` を実行して、シミュレートされたオンプレミス環境に接続します。 `variables.tf` ファイルに指定されたパスワードを使用します。

   ```bash
   ssh testadmin@<onprem_vm_ip_address>
   ```

1. **onprem-vm** 仮想マシンに接続したら、`ping` コマンドを実行してハブ VNet 内のジャンプボックス VM への接続をテストします。

   ```bash
   ping 10.0.0.68
   ```

1. `ping` コマンドを実行して、各スポーク内のジャンプボックス VM への接続をテストします。

   ```bash
   ping 10.1.0.68
   ping 10.2.0.68
   ```

1. **onprem-vm** 仮想マシンで ssh セッションを終了するには、 「`exit`」と入力して、&lt;Enter> キーを押します。

## <a name="troubleshoot-vpn-issues"></a>VPN の問題のトラブルシューティング

VPN エラーの解決方法の詳細については、[ハイブリッド VPN 接続のトラブルシューティング](/azure/architecture/reference-architectures/hybrid-networking/troubleshoot-vpn)の記事をご覧ください。

## <a name="clean-up-resources"></a>リソースをクリーンアップする

このチュートリアル シリーズで作成したリソースが不要になったら、削除してください。

1. プランで宣言されているリソースを削除します。

    ```bash
    terraform destroy
    ```

    リソースの削除の確認を求められたら、「`yes`」と入力します。

1. ディレクトリを親ディレクトリに変更します。

    ```bash
    cd ..
    ```

1. `hub-scope` ディレクトリ (その中のすべてのファイルを含む) を削除します。

    ```bash
    rm -r hub-spoke
    ```

[!INCLUDE [terraform-troubleshooting.md](includes/terraform-troubleshooting.md)]

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"] 
> [Azure での Terraform の使用について詳細を参照](/azure/terraform)