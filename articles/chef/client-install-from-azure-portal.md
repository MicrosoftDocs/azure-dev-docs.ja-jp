---
title: Azure portal から Chef クライアントをインストールする
description: Azure portal から Chef クライアントをデプロイして構成する方法について説明します
keywords: azure、chef、devops、クライアント、インストール、ポータル
ms.date: 02/22/2020
ms.topic: article
ms.openlocfilehash: b5dd158bd06511bf440228d4ae0948596bca0612
ms.sourcegitcommit: a32ca0946275165ce24216c6fa243ec21d6c9193
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/08/2020
ms.locfileid: "80892901"
---
# <a name="install-the-chef-client-from-the-azure-portal"></a>Azure portal から Chef クライアントをインストールする
Azure portal から Linux または Windows マシンに直接、Chef クライアント拡張機能を追加できます。 この記事では、新しい Linux 仮想マシンを使用してそのプロセスを説明します。

## <a name="prerequisites"></a>前提条件

- **Azure サブスクリプション**:Azure サブスクリプションをお持ちでない場合は、開始する前に [無料アカウント](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) を作成してください。

- **Chef**:有効な Chef アカウントを持っていない場合は、[Hosted Chef の無料試用版](https://manage.chef.io/signup)にサインアップします。 この記事の指示に従って進めるには、Chef アカウントから次の値が必要になります。
  - organization_validation キー
  - rb
  - run_list

## <a name="configure-a-new-virtual-machine-with-chef"></a>Chef で新しい仮想マシンを構成する

このセクションでは、Azure portal を使用して、Linux コンピューターを作成します。 処理中に、新しい仮想マシンに Chef 拡張機能をインストールする方法も表示されます。

1. [Azure ポータル](https://portal.azure.com)にアクセスします。

1. 左側のメニューで **[仮想マシン]** オプションを選択します。 **[仮想マシン]** オプションがない場合は、 **[すべてのサービス]** 、 **[仮想マシン]** の順に選択します。

1. **[仮想マシン]** タブで **[追加]** を選択します。

    ![Azure portal で、新しい仮想マシンを追加する](./media/client-install-from-azure-portal/add-vm.png)

1. **[コンピューティング]** タブで、目的のオペレーティング システムを選択します。 このデモでは、 **[Ubuntu Server]** が選択されています。

1. **[Ubuntu Server]** タブで **[Ubuntu Server 16.04 LTS]** を選択します。

    ![Ubuntu 仮想マシンを作成するときに、必要なバージョンを指定する](./media/client-install-from-azure-portal/ubuntu-server-version.png)

1. **[Ubuntu Server 16.04 LTS]** タブで **[作成]** を選択します。

    ![Ubuntu 製品に関する追加情報が Ubuntu から提供される](./media/client-install-from-azure-portal/create-vm.png)

1. **[仮想マシンの作成]** タブで **[基本]** を選択します。

1. **[基本]** タブで次の値を指定し、 **[OK]** を選択します。

   - **名前**: 新しい仮想マシンの名前を入力します。
   - **VM ディスクの種類**: **[SSD]** または **[HDD]** の記憶域ディスクの種類を選択します。 Azure での仮想マシン ディスクの種類について詳しくは、[ディスクの種類の選択](https://docs.microsoft.com/azure/virtual-machines/windows/disks-types)に関する記事をご覧ください。
   - **ユーザー名**: 仮想マシンの管理者権限が付与されているユーザー名を入力します。
   - **認証の種類**: **[パスワード]** を選択します。 **[SSH 公開キー]** を選択して、SSH 公開キー値を指定することもできます。 このデモでは (およびスクリーン ショットでは)、 **[パスワード]** が選択されています。
   - **[パスワード]** と **[パスワードの確認入力]** : ユーザーのパスワードを入力します。
   - **Azure Active Directory でのログイン**: **[無効]** を選択します。
   - **サブスクリプション**: 複数のサブスクリプションがある場合は、目的の Azure サブスクリプションを選択します。
   - **リソース グループ**: リソース グループの名前を入力します。
   - **場所**: **[米国東部]** を選択します。

     ![仮想マシンを作成するための [基本] タブ](./media/client-install-from-azure-portal/add-vm-basics.png)

1. **[サイズの選択]** タブで、仮想マシンのサイズを選択して、 **[選択]** を選択します。

1. **[設定]** タブでは、前のタブで選択した値に基づいてほとんどの値が設定されます。 **[拡張機能]** を選択します。

     ![[設定] タブを通じて拡張機能が仮想マシンに追加される](./media/client-install-from-azure-portal/add-vm-select-extensions.png)

1. **[拡張機能]** タブで **[拡張機能の追加]** を選択します。

     ![[拡張機能の追加] を選択して仮想マシンに拡張機能を追加する](./media/client-install-from-azure-portal/add-vm-add-extension.png)

1. **[新しいリソース]** タブで **[Linux Chef Extension (1.2.3)]** を選択します。

     ![Linux および Windows の仮想マシン向けの Chef の拡張機能](./media/client-install-from-azure-portal/select-linux-chef-extension.png)

1. **[Linux Chef Extension]** タブで **[作成]** を選択します。

1. **[拡張機能のインストール]** タブで次の値を指定し、 **[OK]** を選択します。

    - **Chef サーバーの URL**: たとえば *https://api.chef.io/organization/mycompany* のように、組織名を含む Chef サーバーの URL を入力します。
    - **Chef ノード名**: Chef ノード名を入力します。
    - **実行の一覧**: マシンに追加する Chef の実行の一覧を入力します。 この値は空白のままにします。
    - **Validation client name (検証クライアント名)** : Chef の検証クライアント名を入力します。 たとえば、「`tarcher-validator`」のように入力します。
    - **検証キー**: コンピューターをブートストラップするときに使用する検証キーが含まれているファイルを選択します。
    - **Client configuration file (クライアント構成ファイル)** : chef クライアントの構成ファイルを選択します。 この値は空白のままにします。
    - **Chef Client version (Chef クライアントのバージョン)** : インストールする chef クライアントのバージョンを入力します。 この値は空白のままでもかまいません。空白の場合は、最新バージョンがインストールされます。
    - **SSL Verification Mode (SSL 確認モード)** : **[なし]** または **[ピア]** のいずれかを選択します。 デモ用に *None* を選択しました。
    - **Chef Environment (Chef 環境)** : このノードが属する必要がある Chef 環境を入力します。 この値は空白のままにします。
    - **Encrypted data bag secret (暗号化データ バッグのシークレット)** : このマシンがアクセスする必要がある暗号化データ バッグのシークレットが含まれているファイルを選択します。 この値は空白のままにします。
    - **Chef Server SSL certificate (Chef サーバー SSL 証明書)** : お使いの Chef サーバーに割り当てられている SSL 証明書を選択します。 この値は空白のままにします。

      ![Linux 仮想マシンに Chef サーバーをインストールする](./media/client-install-from-azure-portal/install-extension.png)

1. **[拡張機能]** タブが表示されたら、 **[OK]** を選択します。

1. **[設定]** タブが表示されたら、 **[OK]** を選択します。

1. **[作成]** タブが表示されると、選択および入力したオプションの概要が表示されます。 それらの情報と**利用規約**を確認し、 **[作成]** を選択します。

Chef 拡張機能を使用した仮想マシンの作成とデプロイのプロセスが完了すると、通知に操作の成否が示されます。 さらに、新しい仮想マシンのリソース ページが作成されると、Azure portal でそれが自動的に開かれます。

![Linux 仮想マシンに Chef サーバーをインストールする](./media/client-install-from-azure-portal/resource-created.png)

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"] 
> [Chef を使用して Windows 仮想マシンを Azure 上に作成する](windows-vm-configure.md)