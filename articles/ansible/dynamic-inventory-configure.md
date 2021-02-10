---
title: チュートリアル - Ansible を使用して Azure リソースの動的インベントリを構成する
description: Ansible を使用して Azure の動的インベントリを管理する方法について説明します
keywords: Ansible, Azure, DevOps, Bash, Cloud Shell, 動的インベントリ
ms.topic: tutorial
ms.date: 10/30/2020
ms.custom: devx-track-ansible, devx-track-azurecli
ms.openlocfilehash: d5d3095384fb3f192f7e8cd74b2a49b41b47f239
ms.sourcegitcommit: 3f8aa923e4626b31cc533584fe3b66940d384351
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/01/2021
ms.locfileid: "99224736"
---
# <a name="tutorial-configure-dynamic-inventories-of-your-azure-resources-using-ansible"></a>チュートリアル:Ansible を使用して Azure リソースの動的インベントリを構成する

Ansible を使用して、(Azure などのクラウド ソースを含む) さまざまなソースから "*動的インベントリ*" にインベントリ情報をプルすることができます。 

[!INCLUDE [ansible-tutorial-goals.md](includes/ansible-tutorial-goals.md)]

> [!div class="checklist"]
>
> * 2 つのテスト用仮想マシンを構成する
> * それらの仮想マシンの 1 つにタグを付ける
> * タグ付けされた仮想マシンに Nginx をインストールする
> * 構成済みの Azure リソースを含む動的インベントリを構成する

## <a name="prerequisites"></a>前提条件

[!INCLUDE [open-source-devops-prereqs-azure-subscription.md](../includes/open-source-devops-prereqs-azure-subscription.md)]
[!INCLUDE [open-source-devops-prereqs-create-service-principal.md](../includes/open-source-devops-prereqs-create-service-principal.md)]
[!INCLUDE [ansible-prereqs-cloudshell-use-or-vm-creation2.md](includes/ansible-prereqs-cloudshell-use-or-vm-creation2.md)]

## <a name="create-the-test-vms"></a>テスト VM を作成する

1. [Azure portal](https://go.microsoft.com/fwlink/p/?LinkID=525040) にサインインします。

1. [Cloud Shell](/azure/cloud-shell/overview) を開きます。

1. このチュートリアル用の仮想マシンを保持する Azure リソース グループを作成します。

    > [!IMPORTANT]
    > この手順で作成する Azure リソース グループには、名前をすべて小文字で指定する必要があります。 それ以外の場合、動的インベントリの生成は失敗します。

    ```azurecli
    az group create --resource-group ansible-inventory-test-rg --location eastus
    ```

1. 次の手法のいずれかを使用して、Azure に 2 つの Linux 仮想マシンを作成します。

    - **Ansible プレイブック** - Ansible プレイブックから仮想マシンを作成する方法については、「[Ansible を使用して Azure で基本的な仮想マシンを作成する](./vm-configure.md)」を参照してください。 プレイブックを使用して仮想マシンの一方または両方を定義する場合は、パスワードの代わりに SSH 接続を必ず使用してください。

    - **Azure CLI** - Cloud Shell で次の各コマンドを発行して、2 つの仮想マシンを作成します。

        ```azurecli
        az vm create --resource-group ansible-inventory-test-rg \
                     --name ansible-inventory-test-vm1 \
                     --image UbuntuLTS --generate-ssh-keys
        ```

        ```azurecli
        az vm create --resource-group ansible-inventory-test-rg \
                     --name ansible-inventory-test-vm2 \
                     --image UbuntuLTS --generate-ssh-keys
        ```

## <a name="tag-a-vm"></a>VM にタグを付ける

[タグを使用して、Azure リソースをユーザー定義のカテゴリ別に整理](/azure/azure-resource-manager/resource-group-using-tags#azure-cli)できます。

### <a name="using-ansible-version--28"></a>Ansible バージョン 2.8 未満を使用する
次の [az resource tag](/cli/azure/resource#az-resource-tag) コマンドを入力して、仮想マシン `ansible-inventory-test-vm1` にキー `nginx` を使用してタグを付けます。

```azurecli
az resource tag --tags nginx --id /subscriptions/<YourAzureSubscriptionID>/resourceGroups/ansible-inventory-test-rg/providers/Microsoft.Compute/virtualMachines/ansible-inventory-test-vm1
```

### <a name="using-ansible-version--28"></a>Ansible バージョン 2.8 以上を使用する
次の [az resource tag](/cli/azure/resource#az-resource-tag) コマンドを入力して、仮想マシン `ansible-inventory-test-vm1` にキー `Ansible=nginx` を使用してタグを付けます。

```azurecli
az resource tag --tags Ansible=nginx --id /subscriptions/<YourAzureSubscriptionID>/resourceGroups/ansible-inventory-test-rg/providers/Microsoft.Compute/virtualMachines/ansible-inventory-test-vm1
```

## <a name="generate-a-dynamic-inventory"></a>動的インベントリの生成

仮想マシンを定義 (およびタグ付け) した後は、動的インベントリを生成します。

### <a name="using-ansible-version--28"></a>Ansible バージョン 2.8 未満を使用する

Ansible には、Azure リソースの動的インベントリを生成する [azure_rm.py](https://github.com/ansible-collections/community.general/blob/main/scripts/inventory/azure_rm.py) という名前の Python スクリプトが用意されています。 `azure_rm.py` スクリプトを使用して 2 つのテスト用 Azure 仮想マシンに接続する手順を次に示します。

1. GNU `wget` コマンドを使用して、`azure_rm.py` スクリプトを取得します。

    ```bash
    wget https://raw.githubusercontent.com/ansible-collections/azure/dev/plugins/inventory/azure_rm.py
    ```

1. `chmod` コマンドを使用して、`azure_rm.py` スクリプトへのアクセス許可を変更します。 次のコマンドでは、`+x` パラメーターを使用して、指定されたファイル (`azure_rm.py`) の実行を許可しています。

    ```bash
    chmod +x azure_rm.py
    ```

1. [ansible コマンド](https://docs.ansible.com/ansible/2.4/ansible.html)を使用して、リソース グループに接続します。

    ```bash
    ansible -i azure_rm.py ansible-inventory-test-rg -m ping
    ```

1. 接続されると、作成される仮想マシンの結果を示す結果が表示されます。

### <a name="ansible-version--28"></a>Ansible バージョン 2.8 以上

Ansible 2.8 以降、Ansible では [Azure 動的インベントリ プラグイン](https://github.com/ansible/ansible/blob/stable-2.9/lib/ansible/plugins/inventory/azure_rm.py)が提供されています。 次の手順でこのプラグインの使用方法を示します。

1. インベントリ プラグインには構成ファイルが必要です。 構成ファイルは、末尾が `azure_rm` で、`yml` または `yaml` の拡張子を持っている必要があります。 このチュートリアルの例として、次のプレイブックを `myazure_rm.yml` として保存します。

    ```yml
        plugin: azure_rm
        include_vm_resource_groups:
        - ansible-inventory-test-rg
        auth_source: auto
    
        keyed_groups:
        - prefix: tag
          key: tags
    ```

1. 次のコマンドを実行して、リソース グループ内の VM を ping します。

    ```bash
    ansible all -m ping -i ./myazure_rm.yml
    ```

1. 既定では、ホスト キー チェックが有効になっているため、次のエラーが発生する場合があります。

    ```output
    Failed to connect to the host via ssh: Host key verification failed.
    ```

    `ANSIBLE_HOST_KEY_CHECKING` 環境変数を `False` に設定して、ホスト キーの検証を無効にします。

    ```bash
    export ANSIBLE_HOST_KEY_CHECKING=False
    ```

1. プレイブックを実行すると、次の出力のような結果が表示されます。
  
    ```output
    ansible-inventory-test-vm1_0324 : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    ansible-inventory-test-vm2_8971 : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    ```

## <a name="enable-the-vm-tag"></a>VM タグを有効にする

### <a name="if-youre-using-ansible--28"></a>Ansible 2.8 未満を使用している場合

- タグを設定した後は、そのタグを "有効" にする必要があります。 タグを有効にする 1 つの方法は、`export` コマンドを使用して `AZURE_TAGS` という名前の環境変数にタグをエクスポートするという方法です。

    ```console
    export AZURE_TAGS=nginx
    ```
    
- 次のコマンドを実行します。

    ```bash
    ansible -i azure_rm.py ansible-inventory-test-rg -m ping
    ```
    
    現在は 1 つの仮想マシン (`AZURE_TAGS` 環境変数にエクスポートされた値とタグが一致する仮想マシン) しか表示されません。

    ```output
       ansible-inventory-test-vm1 | SUCCESS => {
        "changed": false,
        "failed": false,
        "ping": "pong"
    }
    ```

### <a name="if-youre-using-ansible---28"></a>Ansible 2.8 以上を使用している場合

- コマンド `ansible-inventory -i myazure_rm.yml --graph` を実行して、次の出力を取得します。

    ```output
        @all:
          |--@tag_Ansible_nginx:
          |  |--ansible-inventory-test-vm1_9e2f
          |--@ungrouped:
          |  |--ansible-inventory-test-vm2_7ba9
    ```

- 次のコマンドを実行して、Nginx VM への接続をテストすることもできます。
  
    ```bash
    ansible -i ./myazure_rm.yml -m ping tag_Ansible_nginx
    ```


## <a name="set-up-nginx-on-the-tagged-vm"></a>タグ付けされた VM での Nginx の設定

タグの目的は、仮想マシンのサブグループをすばやく簡単に操作できるようにすることです。 たとえば、`nginx` というタグを割り当てた仮想マシンにのみ Nginx をインストールするとします。 これは、次の手順に示すように簡単に行うことができます。

1. `nginx.yml` という名前のファイルを作成します。

   ```console
   code nginx.yml
   ```

1. 以下のサンプル コードをエディターに貼り付けます。

    ```yml
        ---
        - name: Install and start Nginx on an Azure virtual machine
          hosts: all
          become: yes
          tasks:
          - name: install nginx
            apt: pkg=nginx state=present
            notify:
            - start nginx
    
          handlers:
            - name: start nginx
              service: name=nginx state=started
    ```

1. ファイルを保存し、エディターを終了します。

1. [ansible-playbook](https://docs.ansible.com/ansible/latest/cli/ansible-playbook.html) を使用してプレイブックを実行します

   - Ansible 2.8 未満:

     ```bash
     ansible-playbook -i azure_rm.py nginx.yml
     ```

   - Ansible 2.8 以上:

     ```bash
     ansible-playbook  -i ./myazure_rm.yml  nginx.yml --limit=tag_Ansible_nginx
     ```

1. プレイブックを実行すると、次の結果のような出力が表示されます。

    ```output
    PLAY [Install and start Nginx on an Azure virtual machine] 

    TASK [Gathering Facts] 
    ok: [ansible-inventory-test-vm1]

    TASK [install nginx] 
    changed: [ansible-inventory-test-vm1]

    RUNNING HANDLER [start nginx] 
    ok: [ansible-inventory-test-vm1]

    PLAY RECAP 
    ansible-inventory-test-vm1 : ok=3    changed=1    unreachable=0    failed=0
    ```

## <a name="test-nginx-installation"></a>Nginx のインストールのテスト

このセクションでは、Nginx が仮想マシンにインストールされているかどうかをテストする 1 つの方法を示します。

1. [az vm list-ip-addresses](/cli/azure/vm#az-vm-list-ip-addresses) コマンドを使用して、`ansible-inventory-test-vm1` 仮想マシンの IP アドレスを取得します。 次に、返された値 (仮想マシンの IP アドレス) を、仮想マシンに接続するための SSH コマンドのパラメーターとして使用します。

    ```azurecli
    ssh `az vm list-ip-addresses \
    -n ansible-inventory-test-vm1 \
    --query [0].virtualMachine.network.publicIpAddresses[0].ipAddress -o tsv`
    ```

1. `ansible-inventory-test-vm1` の仮想マシンに接続されている間に、[nginx -v](https://nginx.org/en/docs/switches.html) コマンドを実行して Nginx がインストールされていることを判定します。

    ```console
    nginx -v
    ```

1. `nginx -v` コマンドを実行すると、Nginx のバージョン (2 行目) が表示され、Nginx がインストールされていることがわかります。

    ```output
    tom@ansible-inventory-test-vm1:~$ nginx -v

    nginx version: nginx/1.10.3 (Ubuntu)

    tom@ansible-inventory-test-vm1:~$
    ```

1. `<Ctrl>D` のキーボードの組み合わせをクリックして SSH セッションを切断します。

1. `ansible-inventory-test-vm2` 仮想マシンに対して上記の手順を実行すると、Nginx を入手できる場所を示す情報メッセージが表示されます (これは、この時点で Nginx がインストールされていないことを意味しています)。

    ```output
    tom@ansible-inventory-test-vm2:~$ nginx -v
    The program 'nginx' can be found in the following packages:
    * nginx-core
    * nginx-extras
    * nginx-full
    * nginx-lightTry: sudo apt install <selected package>
    tom@ansible-inventory-test-vm2:~$
    ```

## <a name="clean-up-resources"></a>リソースをクリーンアップする

[!INCLUDE [ansible-delete-resource-group.md](includes/ansible-delete-resource-group.md)]

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"] 
> [クイック スタート: Ansible を使用して Azure で Linux 仮想マシンを構成する](./vm-configure.md)
