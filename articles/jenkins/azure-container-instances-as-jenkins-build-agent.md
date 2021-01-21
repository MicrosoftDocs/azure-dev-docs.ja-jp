---
title: チュートリアル - Azure Container Instances を Jenkins ビルド エージェントとして使用する
description: Azure Container Instances でビルド ジョブを実行するように Jenkins サーバーを構成する方法について説明します
keywords: jenkins, azure, devops, container instances, ビルド エージェント
ms.topic: article
ms.date: 01/08/2021
ms.custom: devx-track-jenkins,devx-track-azurecli
ms.openlocfilehash: 7633d88897d76f4ed75fa1d7d6c5b0c620db4919
ms.sourcegitcommit: 593d177cfb5f56f236ea59389e43a984da30f104
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/18/2021
ms.locfileid: "98561598"
---
# <a name="tutorial-use-azure-container-instances-as-a-jenkins-build-agent"></a>チュートリアル:Azure Container Instances を Jenkins ビルド エージェントとして使用する

[!INCLUDE [jenkins-integration-with-azure.md](includes/jenkins-integration-with-azure.md)]

Azure Container Instances (ACI) は、コンテナー化ワークロードを実行するためのバースト対応のオンデマンド分離環境を提供します。 これらの特性により、ACI は大規模な環境で Jenkins ビルド ジョブを実行するための優れたプラットフォームを作成します。 この記事では、ACI をデプロイし、Jenkins コントローラー用の永続的なビルド エージェントとして追加する方法について説明します。

Azure Container Instances の詳細については、「[Azure Container Instances について](/azure/container-instances/container-instances-overview)」を参照してください。

## <a name="prerequisites"></a>前提条件

- **Azure サブスクリプション**:Azure サブスクリプションをお持ちでない場合は、開始する前に [無料の Azure アカウント](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)を作成してください。
- **Jenkins サーバー**: Jenkins サーバーがインストールされていない場合は、[Azure に Jenkins サーバーを作成します](./configure-on-linux-vm.md)。

## <a name="prepare-the-jenkins-controller"></a>Jenkins コントローラーを準備する

1. Jenkins ポータルに移動します。

1. メニューから、 **[Manage Jenkins]\(Jenkins の管理\)** を選択します。

1. **[System Configuration]\(システム構成\)** の **[Configure System]\(システムの構成\)** を選択します。

1. **Jenkins URL** が、Jenkins インストール環境の HTTP アドレス (`http://<your_host>.<your_domain>:8080/`) に設定されていることを確認します。

1. メニューから、 **[Manage Jenkins]\(Jenkins の管理\)** を選択します。

1. **[セキュリティ]** で、 **[Configure Global Security]\(グローバル セキュリティの構成\)** を選択します。

1. **[エージェント]** で、**固定** ポートを指定し、環境に適したポート番号を入力します。

    構成の例:![TCP ポートの構成](./media/azure-container-instances-as-jenkins-build-agent/agent-port.png)

1. **[保存]** を選択します。

## <a name="create-jenkins-work-agent"></a>Jenkins 作業エージェントを作成する

1. Jenkins ポータルに移動します。

1. メニューから、 **[Manage Jenkins]\(Jenkins の管理\)** を選択します。

1. **[System Configuration]\(システム構成\)** で、 **[Manage Nodes and Clouds]\(ノードとクラウドの管理\)** を選択します。

1. メニューから **[New Node]\(新しいノード\)** を選択します。

1. **[Node Name]\(ノード名\)** の値を入力します。

1. **[Permanent Agent]\(永続的なエージェント\)** を選択します。

1. **[OK]** を選択します。

1. **リモート ルート ディレクトリ** の値を入力します。 たとえば、`/home/jenkins/work` のように指定します。

1. 必要に応じて、ラベルを入力します。 ラベルは、複数のエージェントを 1 つの論理グループとしてグループ化するために使用します。 ラベルの例として、Linux エージェントをグループ化する `linux` があります。

1. **[Launch method]\(起動方法\)** を **[Launch agent by connecting to the master]\(マスターに接続してエージェントを起動する\)** に設定します。

1. すべての必須フィールドが指定または入力されていることを確認します。

    ![Jenkins エージェントの構成例](./media/azure-container-instances-as-jenkins-build-agent/agent-config.png)

1. **[保存]** を選択します。

1. エージェントの状態ページに、`JENKINS_SECRET` と `AGENT_NAME` が表示されます。 次のスクリーン ショットは、値を識別する方法を示しています。 Azure Container Instance を作成するときは両方の値が必要です。

    ![ビルド エージェントのシークレットは、それが正常に作成されると表示されます。](./media/azure-container-instances-as-jenkins-build-agent/jenkins-secret.png)

## <a name="create-azure-container-instance-with-cli"></a>CLI を使用して Azure Container Instance を作成する

1. [az group create](/cli/azure/group?#az_group_create) を使用して Azure リソース グループを作成します。

      ```azurecli
      az group create --name my-resourcegroup --location westus
      ```

1. [az container create](/cli/azure/container#az_container_create) を使用して、Azure Container Instance を作成します。 プレースホルダーは、作業エージェントの作成時に取得した値に置き換えます。

    ```azurecli
    az container create \
      --name my-dock \
      --resource-group my-resourcegroup \
      --ip-address Public --image jenkins/inbound-agent:latest \
      --os-type linux \
      --ports 80 \
      --command-line "jenkins-agent -url http://jenkinsserver:port <JENKINS_SECRET> <AGENT_NAME>"
    ```

    コンテナーは、起動すると自動的に Jenkins コントローラー サーバーに接続されます。

    ![エージェントが正常に起動しました](./media/azure-container-instances-as-jenkins-build-agent/agent-start.png)

## <a name="create-a-build-job"></a>ビルド ジョブを作成する

Jenkins のビルド ジョブが作成され、Azure コンテナー インスタンス上で Jenkins のビルドをデモできるようになりました。

1. **[新しい項目]** を選択し、ビルド プロジェクトに **aci-demo** などの名前を付け、**[Freestyle project]\(Freestyle プロジェクト\)** を選択し、**[OK]** を選択します。

   ![ビルド ジョブの名前のボックスと、プロジェクトの種類の一覧](./media/azure-container-instances-as-jenkins-build-agent/jenkins-new-job.png)

2. **[全般]** で、**[Restrict where this project can be run]\(このプロジェクトを実行できる場所を制限する\)** が選択されていることを確認します。 ラベル式に「**linux**」と入力します。 この構成により、このビルド ジョブが ACI クラウド上で実行されます。

   ![構成の詳細画表示された [全般] タブ](./media/azure-container-instances-as-jenkins-build-agent/jenkins-job-01.png)

3. **[ビルド]** で **[ビルド ステップの追加]** を選択し、**[シェルの実行]** を選択します。 コマンドとして `echo "aci-demo"` を入力します。

   ![ビルド ステップが選ばれた [ビルド] タブ](./media/azure-container-instances-as-jenkins-build-agent/jenkins-job-02.png)

5. **[保存]** を選択します。

## <a name="run-the-build-job"></a>ビルド ジョブの実行

ビルド ジョブをテストし、ビルド プラットフォームとしての Azure コンテナー インスタンスを確認するには、ビルドを手動で開始します。

1. **[Build Now]\(今すぐビルド\)** を選択してビルド ジョブを開始します。 ジョブが開始するまで､数分かかります｡ 次の図ような状態が表示されます。

   ![ジョブの状態が表示された "ビルド履歴" 情報](./media/azure-container-instances-as-jenkins-build-agent/jenkins-job-status.png)

2. ジョブの実行中に、Azure Portal を開き、Jenkins リソース グループを参照します。 コンテナー インスタンスが作成されています。 Jenkins ジョブはこのインスタンス内で実行されています。

   ![リソース グループ内のコンテナー インスタンス](./media/azure-container-instances-as-jenkins-build-agent/jenkins-aci.png)

3. Jenkins が Jenkins 実行子の数 (既定値は 2) より多くのジョブを実行すると、複数のコンテナー インスタンスが作成されます。

   ![新しく作成されたコンテナー インスタンス](./media/azure-container-instances-as-jenkins-build-agent/jenkins-aci-multi.png)

4. すべてのビルド ジョブが完了すると、コンテナー インスタンスは削除されます。

   ![コンテナー インスタンスが削除されたリソース グループ](./media/azure-container-instances-as-jenkins-build-agent/jenkins-aci-none.png)

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [Azure App Service への CI/CD](/azure/jenkins/tutorial-jenkins-deploy-web-app-azure-app-service)