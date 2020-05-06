---
title: チュートリアル- Terraform を使用して Packer カスタム イメージから Azure 仮想マシン スケール セットを作成する
description: Terraform を使用して、Packer で生成されたカスタム イメージ (仮想ネットワークおよび接続された管理ディスクを含む) から Azure 仮想マシン スケール セットを構成し、バージョンを管理する方法について説明します。
ms.topic: tutorial
ms.date: 11/07/2019
ms.openlocfilehash: 1710614c783ffb60f54da1291f3a0be039c46589
ms.sourcegitcommit: be67ceba91727da014879d16bbbbc19756ee22e2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/05/2020
ms.locfileid: "82171878"
---
# <a name="tutorial-create-an-azure-virtual-machine-scale-set-from-a-packer-custom-image-by-using-terraform"></a>チュートリアル:Terraform を使用して Packer カスタム イメージから Azure 仮想マシン スケール セットを作成する

[Azure 仮想マシン スケール セット](/azure/virtual-machine-scale-sets)を使用すると、同一の VM を構成できます。 VM インスタンスの数は、需要またはスケジュールに基づいて調整できます。 詳細については、「[Azure Portal で仮想マシン スケール セットを自動的にスケーリングする](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-portal)」を参照してください。

このチュートリアルでは、以下の内容を学習します。

> [!div class="checklist"]
> * Terraform デプロイを設定します。
> * Terraform デプロイ用の変数と出力を使用します。
> * ネットワーク インフラストラクチャを作成してデプロイします。
> * Packer を使用してカスタム仮想マシン イメージを作成します。
> * カスタム イメージを使用して、仮想マシン スケール セットを作成してデプロイします。
> * ジャンプボックスを作成してデプロイします。

[!INCLUDE [hashicorp-support.md](includes/hashicorp-support.md)]

## <a name="prerequisites"></a>前提条件

[!INCLUDE [open-source-devops-prereqs-azure-subscription.md](../includes/open-source-devops-prereqs-azure-subscription.md)]
- **Terraform**:[Terraform をインストールし、Azure へのアクセスを構成します](install-configure.md)。
- **SSH キーの組**:[SSH キーの組を作成します](/azure/virtual-machines/linux/mac-create-ssh-keys)。
- **Packer**:[Packer をインストールします](https://www.packer.io/docs/install/index.html)。

## <a name="create-the-file-structure"></a>ファイル構造を作成する

空のディレクトリに次の名前で新しいファイルを 3 つ作成します。

- `variables.tf`:このファイルには、テンプレートに使用される変数の値が格納されます。
- `output.tf`:このファイルには、デプロイ後に表示される設定が記述されます。
- `vmss.tf`:このファイルには、デプロイしているインフラストラクチャのコードが格納されます。

##  <a name="create-the-variables"></a>変数を作成する 

この手順では、Terraform で作成されたリソースをカスタマイズする変数を定義します。

`variables.tf` ファイルを編集し、次のコードをコピーしてから変更を保存します。

```hcl
variable "location" {
  description = "The location where resources are created"
  default     = "East US"
}

variable "resource_group_name" {
  description = "The name of the resource group in which the resources are created"
  default     = ""
}

```

> [!NOTE]
> 変数 resource_group_name の既定値は設定されていません。 独自の値を定義してください。

ファイルを保存します。

Terraform テンプレートをデプロイするときに、アプリケーションへのアクセスに使用される完全修飾ドメイン名を取得できます。 Terraform の `output` リソースの種類を使用して、リソースの `fqdn` プロパティを取得します。 

`output.tf` ファイルを編集し、次のコードをコピーして、仮想マシンの完全修飾ドメイン名が表示されるようにします。 

```hcl 
output "vmss_public_ip" {
    value = azurerm_public_ip.vmss.fqdn
}
```

## <a name="define-the-network-infrastructure-in-a-template"></a>テンプレートでネットワーク インフラストラクチャを定義する 

この手順では、新しい Azure リソース グループに次のネットワーク インフラストラクチャを作成します。 
  - 10.0.0.0/16 のアドレス空間を使用する 1 つの 仮想ネットワーク。
  - 10.0.2.0/24 のアドレス空間を使用する 1 つのサブネット。
  - 2 つのパブリック IP アドレス。 1 つは、仮想マシン スケール セット ロード バランサーによって使用されます。 もう 1 つは、SSH ジャンプボックスへの接続に使用されます。

また、すべてのリソースの作成場所となるリソース グループも必要です。 

次のコードを編集し、`vmss.tf` ファイルにコピーします。 

```hcl

resource "azurerm_resource_group" "vmss" {
  name     = var.resource_group_name
  location = var.location

  tags {
    environment = "codelab"
  }
}

resource "azurerm_virtual_network" "vmss" {
  name                = "vmss-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = var.location
  resource_group_name = azurerm_resource_group.vmss.name

  tags {
    environment = "codelab"
  }
}

resource "azurerm_subnet" "vmss" {
  name                 = "vmss-subnet"
  resource_group_name  = azurerm_resource_group.vmss.name
  virtual_network_name = azurerm_virtual_network.vmss.name
  address_prefix       = "10.0.2.0/24"
}

resource "azurerm_public_ip" "vmss" {
  name                         = "vmss-public-ip"
  location                     = var.location
  resource_group_name          = azurerm_resource_group.vmss.name
  allocation_method            = "static"
  domain_name_label            = azurerm_resource_group.vmss.name

  tags {
    environment = "codelab"
  }
}

``` 

> [!NOTE]
> 今後識別しやすいように、Azure にデプロイするリソースにタグを付けてください。

## <a name="create-the-network-infrastructure"></a>ネットワーク インフラストラクチャを作成する

`.tf` ファイルを作成したディレクトリで次のコマンドを実行して、Terraform 環境を初期化します。

```bash
terraform init 
```
 
コマンドを実行したディレクトリの `.terraform` フォルダー内に、Terraform レジストリからプロバイダー プラグインがダウンロードされます。

次のコマンドを実行して、Azure にインフラストラクチャをデプロイします。

```bash
terraform apply
```

パブリック IP アドレスの完全修飾ドメイン名が、お使いの構成と対応していることを確認します。

![仮想マシン スケール セットのパブリック IP アドレスの Terraform 完全修飾ドメイン名](./media/create-vm-scaleset-network-disks-using-packer-hcl/tf-create-vmss-step4-fqdn.png)

リソース グループには次のリソースが含まれています。

![仮想マシン スケール セットの Terraform ネットワーク リソース](./media/create-vm-scaleset-network-disks-using-packer-hcl/tf-create-vmss-step4-rg.png)


## <a name="create-an-azure-image-by-using-packer"></a>Packer を使用して Azure イメージを作成する
チュートリアル「[Packer を使用して Azure に Linux 仮想マシンのイメージを作成する方法](/azure/virtual-machines/linux/build-image-with-packer)」の手順に従って、カスタム Linux イメージを作成します。
 
チュートリアルに従って、Nginx がインストールされているプロビジョニング解除済みの Ubuntu イメージを作成します。

![Packer イメージを作成すると、イメージが準備できます](./media/create-vm-scaleset-network-disks-using-packer-hcl/packerimagecreated.png)

> [!NOTE]
> このチュートリアルのために、Packer イメージ内でコマンドを実行して Nginx をインストールします。 作成時に独自のスクリプトを実行することもできます。

## <a name="edit-the-infrastructure-to-add-the-virtual-machine-scale-set"></a>インフラストラクチャを編集して仮想マシン スケール セットを追加する

この手順では、以前にデプロイしたネットワーク上に次のリソースを作成します。
- アプリケーションにサービスを提供する Azure ロードバランサー。 以前にデプロイしたパブリック IP アドレスにこれを接続します。
- アプリケーションにサービスを提供する 1 つの Azure ロードバランサーと複数の規則。 以前に構成したパブリック IP アドレスにこれを接続します。
- Azure バックエンド アドレス プール。 ロードバランサーにこれを割り当てます。
- アプリケーションで使用する正常性プローブ ポート (ロード バランサー上に構成します)。
- ロードバランサーの背後に配置され、以前にデプロイされた仮想ネットワーク上で実行される仮想マシン スケール セット。
- カスタム イメージからインストールした仮想マシン スケールのノード上の [Nginx](https://nginx.org/)。


`vmss.tf` ファイルの末尾に次のコードを追加します。

```hcl

resource "azurerm_lb" "vmss" {
  name                = "vmss-lb"
  location            = var.location
  resource_group_name = azurerm_resource_group.vmss.name

  frontend_ip_configuration {
    name                 = "PublicIPAddress"
    public_ip_address_id = azurerm_public_ip.vmss.id
  }

  tags {
    environment = "codelab"
  }
}

resource "azurerm_lb_backend_address_pool" "bpepool" {
  resource_group_name = azurerm_resource_group.vmss.name
  loadbalancer_id     = azurerm_lb.vmss.id
  name                = "BackEndAddressPool"
}

resource "azurerm_lb_probe" "vmss" {
  resource_group_name = azurerm_resource_group.vmss.name
  loadbalancer_id     = azurerm_lb.vmss.id
  name                = "ssh-running-probe"
  port                = var.application_port
}

resource "azurerm_lb_rule" "lbnatrule" {
  resource_group_name            = azurerm_resource_group.vmss.name
  loadbalancer_id                = azurerm_lb.vmss.id
  name                           = "http"
  protocol                       = "Tcp"
  frontend_port                  = var.application_port
  backend_port                   = var.application_port
  backend_address_pool_id        = azurerm_lb_backend_address_pool.bpepool.id
  frontend_ip_configuration_name = "PublicIPAddress"
  probe_id                       = azurerm_lb_probe.vmss.id
}

data "azurerm_resource_group" "image" {
  name = "myResourceGroup"
}

data "azurerm_image" "image" {
  name                = "myPackerImage"
  resource_group_name = data.azurerm_resource_group.image.name
}

resource "azurerm_virtual_machine_scale_set" "vmss" {
  name                = "vmscaleset"
  location            = var.location
  resource_group_name = azurerm_resource_group.vmss.name
  upgrade_policy_mode = "Manual"

  sku {
    name     = "Standard_DS1_v2"
    tier     = "Standard"
    capacity = 2
  }

  storage_profile_image_reference {
    id=data.azurerm_image.image.id
  }

  storage_profile_os_disk {
    name              = ""
    caching           = "ReadWrite"
    create_option     = "FromImage"
    managed_disk_type = "Standard_LRS"
  }

  storage_profile_data_disk {
    lun          = 0
    caching        = "ReadWrite"
    create_option  = "Empty"
    disk_size_gb   = 10
  }

  os_profile {
    computer_name_prefix = "vmlab"
    admin_username       = "azureuser"
    admin_password       = "Passwword1234"
  }

  os_profile_linux_config {
    disable_password_authentication = true

    ssh_keys {
      path     = "/home/azureuser/.ssh/authorized_keys"
      key_data = file("~/.ssh/id_rsa.pub")
    }
  }

  network_profile {
    name    = "terraformnetworkprofile"
    primary = true

    ip_configuration {
      name                                   = "IPConfiguration"
      subnet_id                              = azurerm_subnet.vmss.id
      load_balancer_backend_address_pool_ids = [azurerm_lb_backend_address_pool.bpepool.id]
      primary = true
    }
  }
  
  tags {
    environment = "codelab"
  }
}

```

次のコードを `variables.tf` に追加してデプロイをカスタマイズします。

```hcl
variable "application_port" {
    description = "The port that you want to expose to the external load balancer"
    default     = 80
}

variable "admin_password" {
    description = "Default password for admin"
    default = "Passwwoord11223344"
}
``` 


## <a name="deploy-the-virtual-machine-scale-set-in-azure"></a>Azure に仮想マシン スケール セットをデプロイする

次のコマンドを実行して、仮想マシン スケール セットのデプロイを視覚化します。

```bash
terraform plan
```

コマンドの出力は次の図のようになります。

![Terraform の仮想マシン スケール セット プランの追加](./media/create-vm-scaleset-network-disks-using-packer-hcl/tf-create-vmss-step6-plan.png)

Azure で他のリソースをデプロイします。 

```bash
terraform apply 
```

リソース グループの内容は次の図のようになります。

![Terraform の仮想マシン スケール セットのリソース グループ](./media/create-vm-scaleset-network-disks-using-packer-hcl/tf-create-vmss-step6-apply.png)

ブラウザーを開き、コマンドから返された完全修飾ドメイン名に接続します。 


## <a name="add-a-jumpbox-to-the-existing-network"></a>ジャンプボックスを既存のネットワークに追加する 

この手順ではジャンプボックスを使用して、仮想マシン スケール セットのインスタンスへの SSH アクセスを有効にします (省略可能)。

既存のデプロイに次のリソースを追加します。
- 仮想マシン スケール セットと同じサブネットに接続されているネットワーク インターフェイス
- このネットワーク インターフェイスに接続されている仮想マシン

`vmss.tf` ファイルの末尾に次のコードを追加します。

```hcl 
resource "azurerm_public_ip" "jumpbox" {
  name                         = "jumpbox-public-ip"
  location                     = var.location
  resource_group_name          = azurerm_resource_group.vmss.name
  allocation_method            = "static"
  domain_name_label            = "${azurerm_resource_group.vmss.name}-ssh"

  tags {
    environment = "codelab"
  }
}

resource "azurerm_network_interface" "jumpbox" {
  name                = "jumpbox-nic"
  location            = var.location
  resource_group_name = azurerm_resource_group.vmss.name

  ip_configuration {
    name                          = "IPConfiguration"
    subnet_id                     = azurerm_subnet.vmss.id
    private_ip_address_allocation = "dynamic"
    public_ip_address_id          = azurerm_public_ip.jumpbox.id
  }

  tags {
    environment = "codelab"
  }
}

resource "azurerm_virtual_machine" "jumpbox" {
  name                  = "jumpbox"
  location              = var.location
  resource_group_name   = azurerm_resource_group.vmss.name
  network_interface_ids = [azurerm_network_interface.jumpbox.id]
  vm_size               = "Standard_DS1_v2"

  storage_image_reference {
    publisher = "Canonical"
    offer     = "UbuntuServer"
    sku       = "16.04-LTS"
    version   = "latest"
  }

  storage_os_disk {
    name              = "jumpbox-osdisk"
    caching           = "ReadWrite"
    create_option     = "FromImage"
    managed_disk_type = "Standard_LRS"
  }

  os_profile {
    computer_name  = "jumpbox"
    admin_username = "azureuser"
    admin_password = "Password1234!"
  }

  os_profile_linux_config {
    disable_password_authentication = true

    ssh_keys {
      path     = "/home/azureuser/.ssh/authorized_keys"
      key_data = file("~/.ssh/id_rsa.pub")
    }
  }

  tags {
    environment = "codelab"
  }
}
```

`outputs.tf` を編集して次のコードを追加し、デプロイの完了時にジャンプボックスのホスト名が表示されるようにします。

```
output "jumpbox_public_ip" {
    value = azurerm_public_ip.jumpbox.fqdn
}
```

## <a name="deploy-the-jumpbox"></a>ジャンプボックスをデプロイする

ジャンプボックスをデプロイします。

```bash
terraform apply 
```

デプロイが完了すると、リソース グループの内容は次の図のようになります。

![Terraform の仮想マシン スケール セットのリソース グループ](./media/create-vm-scaleset-network-disks-using-packer-hcl/tf-create-create-vmss-step8.png)

> [!NOTE]
> デプロイしたジャンプボックスと仮想マシン スケール セットでは、パスワードによるサインインは無効にされています。 VM にアクセスするには、SSH を使用してサインインします。

## <a name="clean-up-the-environment"></a>環境のクリーンアップ

次のコマンドは、このチュートリアルで作成したリソースを削除します。

```bash
terraform destroy
```

リソースの削除の確認を求めるメッセージが表示されたら、「*yes*」と入力します。 この削除プロセスが完了するまでに数分かかることがあります。

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"] 
> [Azure での Terraform の使用について詳細を参照](/azure/terraform)
