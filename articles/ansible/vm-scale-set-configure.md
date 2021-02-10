---
title: チュートリアル - Ansible を使用して Azure 内に仮想マシン スケール セットを構成する
description: Ansible を使用して、Azure で仮想マシン スケール セットを作成して構成する方法について説明します
keywords: ansible, azure, devops, bash, プレイブック, 仮想マシン, 仮想マシン スケール セット, vmss
ms.topic: tutorial
ms.date: 04/30/2019
ms.custom: devx-track-ansible
ms.openlocfilehash: 88b8c32f4eb1a7422cc6aa55e8a2520a01476656
ms.sourcegitcommit: 3f8aa923e4626b31cc533584fe3b66940d384351
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/01/2021
ms.locfileid: "99224716"
---
# <a name="tutorial-configure-virtual-machine-scale-sets-in-azure-using-ansible"></a>チュートリアル:Ansible を使用して Azure 内に仮想マシン スケール セットを構成する

[!INCLUDE [ansible-29-note.md](includes/ansible-29-note.md)]

[!INCLUDE [open-source-devops-intro-vm-scale-set.md](../includes/open-source-devops-intro-vm-scale-set.md)]

[!INCLUDE [ansible-tutorial-goals.md](includes/ansible-tutorial-goals.md)]

> [!div class="checklist"]
>
> * VM 用のリソースを構成する
> * スケール セットを構成する
> * その VM インスタンスを増加してスケール セットをスケーリングする 

## <a name="prerequisites"></a>前提条件

[!INCLUDE [open-source-devops-prereqs-azure-subscription.md](../includes/open-source-devops-prereqs-azure-subscription.md)]
[!INCLUDE [ansible-prereqs-cloudshell-use-or-vm-creation2.md](includes/ansible-prereqs-cloudshell-use-or-vm-creation2.md)]

## <a name="configure-a-scale-set"></a>スケール セットを構成する

このセクションのプレイブック コードは、次のリソースを定義します。

* **リソース グループ**。すべてのリソースがデプロイされるリソース グループです。
* **仮想ネットワーク**。10.0.0.0/16 アドレス空間内の仮想ネットワークです。
* **サブネット**。仮想ネットワーク内のサブネットです。
* **パブリック IP アドレス**。このアドレスを使用して、インターネットのリソースにアクセスできます。
* **ネットワーク セキュリティ グループ**。スケール セットへのネットワーク トラフィックおよびスケール セットからのネットワーク トラフィックのフローを制御します。
* **ロード バランサー**。ロード バランサー ルールを使用して、定義された一連の VM にトラフィックを分散します。
* **仮想マシン スケール セット**。作成されたすべてのリソースを使用します。

サンプル プレイブックを取得するには、次の 2 つの方法があります。

* [プレイブックをダウンロード](https://github.com/Azure-Samples/ansible-playbooks/blob/master/vmss/vmss-create.yml)して、`vmss-create.yml` に保存する。
* `vmss-create.yml` という名前の新規ファイルを作成して、次のコンテンツをコピーする。

```yml
- hosts: localhost
  vars:
    resource_group: myResourceGroup
    vmss_lb_name: myScaleSetLb
    location: eastus
    admin_username: azureuser
    admin_password: "{{ admin_password }}"

  tasks:
    - name: Create a resource group
      azure_rm_resourcegroup:
        name: "{{ resource_group }}"
        location: "{{ location }}"
    - name: Create virtual network
      azure_rm_virtualnetwork:
        resource_group: "{{ resource_group }}"
        name: "{{ vmss_name }}"
        address_prefixes: "10.0.0.0/16"
    - name: Add subnet
      azure_rm_subnet:
        resource_group: "{{ resource_group }}"
        name: "{{ vmss_name }}"
        address_prefix: "10.0.1.0/24"
        virtual_network: "{{ vmss_name }}"
    - name: Create public IP address
      azure_rm_publicipaddress:
        resource_group: "{{ resource_group }}"
        allocation_method: Static
        name: "{{ vmss_name }}"
    - name: Create Network Security Group that allows SSH
      azure_rm_securitygroup:
        resource_group: "{{ resource_group }}"
        name: "{{ vmss_name }}"
        rules:
          - name: SSH
            protocol: Tcp
            destination_port_range: 22
            access: Allow
            priority: 1001
            direction: Inbound

    - name: Create a load balancer
      azure_rm_loadbalancer:
        resource_group: "{{ resource_group }}"
        name: "{{ vmss_name }}lb"
        location: "{{ location }}"
        frontend_ip_configurations:
          - name: "{{ vmss_name }}front-config"
            public_ip_address: "{{ vmss_name }}"
        backend_address_pools:
          - name: "{{ vmss_name }}backend-pool"
        probes:
          - name: "{{ vmss_name }}prob0"
            port: 8080
            interval: 10
            fail_count: 3
        inbound_nat_pools:
          - name: "{{ vmss_name }}nat-pool"
            frontend_ip_configuration_name: "{{ vmss_name }}front-config"
            protocol: Tcp
            frontend_port_range_start: 50000
            frontend_port_range_end: 50040
            backend_port: 22
        load_balancing_rules:
          - name: "{{ vmss_name }}lb-rules"
            frontend_ip_configuration: "{{ vmss_name }}front-config"
            backend_address_pool: "{{ vmss_name }}backend-pool"
            frontend_port: 80
            backend_port: 8080
            load_distribution: Default
            probe: "{{ vmss_name }}prob0"

    - name: Create VMSS
      no_log: true
      azure_rm_virtualmachinescaleset:
        resource_group: "{{ resource_group }}"
        name: "{{ vmss_name }}"
        vm_size: Standard_DS1_v2
        admin_username: "{{ admin_username }}"
        admin_password: "{{ admin_password }}"
        ssh_password_enabled: true
        capacity: 2
        virtual_network_name: "{{ vmss_name }}"
        subnet_name: "{{ vmss_name }}"
        upgrade_policy: Manual
        tier: Standard
        managed_disk_type: Standard_LRS
        os_disk_caching: ReadWrite
        image:
          offer: UbuntuServer
          publisher: Canonical
          sku: 16.04-LTS
          version: latest
        load_balancer: "{{ vmss_name }}lb"
        data_disks:
          - lun: 0
            disk_size_gb: 20
            managed_disk_type: Standard_LRS
            caching: ReadOnly
          - lun: 1
            disk_size_gb: 30
            managed_disk_type: Standard_LRS
            caching: ReadOnly

```

プレイブックを実行する前に、次の注意事項を参照してください。

* `vars` セクションで、`{{ admin_password }}` プレースホルダーを自分のパスワードに置き換えます。

[ansible-playbook](https://docs.ansible.com/ansible/latest/cli/ansible-playbook.html) を使用してプレイブックを実行します

```bash
ansible-playbook vmss-create.yml
```

プレイブックを実行すると、次の結果のような出力が表示されます。

```Output
PLAY [localhost] 

TASK [Gathering Facts] 
ok: [localhost]

TASK [Create a resource group] 
changed: [localhost]

TASK [Create virtual network] 
changed: [localhost]

TASK [Add subnet] 
changed: [localhost]

TASK [Create public IP address] 
changed: [localhost]

TASK [Create Network Security Group that allows SSH] 
changed: [localhost]

TASK [Create a load balancer] 
changed: [localhost]

TASK [Create Scale Set] 
changed: [localhost]

PLAY RECAP 
localhost                  : ok=8    changed=7    unreachable=0    failed=0

```

## <a name="view-the-number-of-vm-instances"></a>VM インスタンスの数を表示する

[構成されているスケール セット](#configure-a-scale-set)には、現在 2 つのインスタンスがあります。 次の手順を使用してその値を確認します。

1. [Azure portal](https://go.microsoft.com/fwlink/p/?LinkID=525040) にサインインします。

1. 構成したスケール セットに移動します。

1. 次のように、インスタンスの数がかっこ内に入ったスケール セット名を確認できます。`Standard_DS1_v2 (2 instances)`

1. 次のコマンドを実行して、[Azure Cloud Shell](https://shell.azure.com/) でインスタンスの数を確認することもできます。

    ```azurecli-interactive
    az vmss show -n myScaleSet -g myResourceGroup --query '{"capacity":sku.capacity}' 
    ```

    Cloud Shell で Azure CLI コマンドを実行した結果には、2 つのインスタンスが存在することが示されています。

    ```bash
    {
      "capacity": 2,
    }
    ```

## <a name="scale-out-a-scale-set"></a>スケール セットをスケールアウトする

次のセクションのプレイブック コードは、スケール セットに関する情報を取得して、その容量を 2 から 3 に変更します。

サンプル プレイブックを取得するには、次の 2 つの方法があります。

* [プレイブックをダウンロード](https://github.com/Azure-Samples/ansible-playbooks/blob/master/vmss/vmss-scale-out.yml)して、`vmss-scale-out.yml` に保存する。
* `vmss-scale-out.yml` という名前の新規ファイルを作成して、次のコンテンツをコピーする。

```yml
---
- hosts: localhost
  gather_facts: false
  
  vars:
    resource_group: myTestRG
    vmss_name: myTestVMSS
  
  tasks:

    - name: Get scaleset info
      azure_rm_virtualmachine_scaleset_facts:
        resource_group: "{{ resource_group }}"
        name: "{{ vmss_name }}"
        format: curated
      register: output_scaleset

    - name: set image fact
      set_fact:
        vmss_image: "{{ output_scaleset.vmss[0].image }}"

    - name: Create VMSS
      no_log: true
      azure_rm_virtualmachinescaleset:
        resource_group: "{{ resource_group }}"
        name: "{{ vmss_name }}"
        capacity: 3
        image: "{{ vmss_image }}"
```

[ansible-playbook](https://docs.ansible.com/ansible/latest/cli/ansible-playbook.html) を使用してプレイブックを実行します

```bash
ansible-playbook vmss-scale-out.yml
```

プレイブックを実行すると、次の結果のような出力が表示されます。

```Output
PLAY [localhost] 

TASK [Gathering Facts] 
ok: [localhost]

TASK [Get scaleset info] 
ok: [localhost]

TASK [Set image fact] 
ok: [localhost]

TASK [Change VMSS capacity] 
changed: [localhost]

PLAY RECAP 
localhost                  : ok=3    changed=1    unreachable=0    failed=0
```

## <a name="verify-the-results"></a>結果を確認する

次のように、Azure portal を使用して作業の結果を確認します。

1. [Azure portal](https://go.microsoft.com/fwlink/p/?LinkID=525040) にサインインします。

1. 構成したスケール セットに移動します。

1. 次のように、インスタンスの数がかっこ内に入ったスケール セット名を確認できます。`Standard_DS1_v2 (3 instances)`

1. 次のコマンドを実行して、[Azure Cloud Shell](https://shell.azure.com/) で変更を検証することもできます。

    ```azurecli-interactive
    az vmss show -n myScaleSet -g myResourceGroup --query '{"capacity":sku.capacity}' 
    ```

    Cloud Shell で Azure CLI コマンドを実行した結果には、現在 3 つのインスタンスが存在することが示されています。 

    ```bash
    {
      "capacity": 3,
    }
    ```

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [チュートリアル:Ansible を使用して Azure の仮想マシン スケール セットにアプリをデプロイする](./vm-scale-set-deploy-app.md)
