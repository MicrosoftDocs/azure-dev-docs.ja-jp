---
title: チュートリアル - Jenkins を使用して GitHub から Azure Kubernetes Service にデプロイする
description: GitHub からの継続的インテグレーション (CI) と Azure Kubernetes Service (AKS) への継続的配置 (CD) のために Jenkins 構成する方法について説明します。
keywords: jenkins, azure, devops, aks, azure kubernetes service, github
ms.topic: article
ms.date: 02/05/2021
ms.custom: devx-track-jenkins, devx-track-azurecli
ms.openlocfilehash: d6a45e227436970fa34505ccfbf28dc6e8c2a5d8
ms.sourcegitcommit: 5f4a041aede2a0b7e2cf479185f62885209eaf9f
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/09/2021
ms.locfileid: "102517176"
---
# <a name="tutorial-deploy-from-github-to-azure-kubernetes-service-using-jenkins"></a>チュートリアル:Jenkins を使用して GitHub から Azure Kubernetes Service にデプロイする

[!INCLUDE [jenkins-integration-with-azure.md](includes/jenkins-integration-with-azure.md)]

このチュートリアルでは、Jenkins で継続的インテグレーション (CI) と継続的デプロイ (CD) を設定して、GitHub から [Azure Kubernetes Service (AKS)](/azure/aks/intro-kubernetes) クラスターにサンプル アプリをデプロイします。

このチュートリアルでは、以下のタスクを完了します。

> [!div class="checklist"]
> * サンプルの Azure 投票アプリを AKS クラスターにデプロイする
> * 基本的な Jenkins プロジェクトを作成する
> * ACR とやりとりする Jenkins 用の資格情報を設定する
> * Jenkins ビルド ジョブと自動ビルド用の GitHub Webhook を作成する
> * GitHub コード コミットに基づき、CI/CD パイプラインをテストして AKS でアプリケーションをアップデートする

## <a name="prerequisites"></a>前提条件

このチュートリアルを完了するには、以下のものが必要です。

- Kubernetes、Git、CI/CD、およびコンテナーのイメージについての基本的な理解

- [AKS クラスター](/azure/aks/kubernetes-walkthrough)、および [AKS クラスターの資格情報](/cli/azure/aks#az-aks-get-credentials)で構成されています `kubectl`。

- [ACR レジストリで認証を行う](/azure/aks/cluster-container-registry-integration) ために構成されている [Azure Container Registry (ACR) レジストリ](/azure/container-registry/container-registry-get-started-azure-cli)、ACR ログイン サーバー名、および AKS クラスター。

- Azure 仮想マシンにデプロイされた [Jenkins コントローラー](https://docs.microsoft.com/azure/developer/jenkins/configure-on-linux-vm)。

- インストールおよび構成済みの Azure CLI バージョン 2.0.46 以降。 バージョンを確認するには、`az --version` を実行します。 インストールまたはアップグレードする必要がある場合は、[Azure CLI のインストール](/cli/azure/install-azure-cli)に関するページを参照してください。

- 開発システムに[インストールされた Docker](https://docs.docker.com/install/)

- GitHub アカウント、[GitHub 個人用アクセス トークン](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/)、および開発システムにインストールされた Git クライアント

- このサンプル スクリプトによる方法ではなく、独自の Jenkins インスタンスを指定して、Jenkins をデプロイする場合、Jenkins インスタンスには[インストールされて構成された Docker](https://docs.docker.com/install/) と [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) が必要です。

## <a name="prepare-your-app"></a>アプリケーションの準備

この記事では、一時的なデータ ストレージ用の Web インターフェイスと Redis を含むサンプルの Azure 投票アプリケーションを使用します。

Jenkins と AKS を自動デプロイ用に統合する前に、まず手動で Azure 投票アプリケーションを準備し、AKS クラスターにデプロイします。 この手動のデプロイにより、アプリケーションが動作していることを確認できます。

> [!NOTE]
> サンプルの Azure 投票アプリケーションでは、Linux ノード上で実行するようにスケジュールされた Linux ポッドを使用します。 この記事で概要を説明するフローは、Windows Server ノード上でスケジュールされた Windows Server ポッドでも機能します。

サンプル アプリケーション ([https://github.com/Azure-Samples/azure-voting-app-redis](https://github.com/Azure-Samples/azure-voting-app-redis)) の GitHub リポジトリをフォークします。 自身の GitHub アカウントにリポジトリをフォークするには、右上隅の **[Fork]** ボタンを選択します。

開発システム用にフォークの複製を作成します。 このリポジトリを複製するときに、フォークの URL を使用するようにします。

```console
git clone https://github.com/<your-github-account>/azure-voting-app-redis.git
```

複製したフォークのディレクトリに変更します。

```console
cd azure-voting-app-redis
```

サンプル アプリケーションに必要なコンテナー イメージを作成するには、`docker-compose` で *docker-compose.yaml* ファイルを使用します。

```console
docker-compose up -d
```

必要な基本イメージがプルされ、アプリケーション コンテナーがビルドされます。 その後、[docker images](https://docs.docker.com/engine/reference/commandline/images/) コマンドを使用して、作成されたイメージを確認できます。 3 つのイメージがダウンロードまたは作成されていることを確認してください。 `azure-vote-front` イメージはアプリケーションを含み、`nginx-flask` イメージをベースとして使用します。 `redis` イメージは、Redis インスタンスを起動するために使用されます。

```
$ docker images

REPOSITORY                   TAG        IMAGE ID            CREATED             SIZE
azure-vote-front             latest     9cc914e25834        40 seconds ago      694MB
redis                        latest     a1b99da73d05        7 days ago          106MB
tiangolo/uwsgi-nginx-flask   flask      788ca94b2313        9 months ago        694MB
```

Azure コンテナー インスタンスにサインインします。

```azurecli
az login -n <acrLoginServer>
```

`<acrLoginServer>` は、実際の ACR ログイン サーバーに置き換えてください。

[docker tag](https://docs.docker.com/engine/reference/commandline/tag/) コマンドを使用して、ACR ログイン サーバー名とバージョン番号 `v1` でイメージにタグを付けます。 前のステップで取得した独自の `<acrLoginServer>` 名を使用します。

```console
docker tag azure-vote-front <acrLoginServer>/azure-vote-front:v1
```

最後に、*azure-vote-front* イメージを ACR レジストリにプッシュします。 ここでも、`<acrLoginServer>` を `myacrregistry.azurecr.io` などの独自の ACR レジストリのログイン サーバー名と置き換えます。

```console
docker push <acrLoginServer>/azure-vote-front:v1
```

## <a name="deploy-the-sample-application-to-aks"></a>サンプル アプリケーションを AKS にデプロイする

サンプル アプリケーションを AKS クラスターにデプロイするには、Azure 投票リポジトリのルートにある Kubernetes マニフェスト ファイルを使用します。 `vi` などのエディターを使用して `azure-vote-all-in-one-redis.yaml` マニフェスト ファイルを開きます。 `microsoft` は、実際の ACR ログイン サーバー名に置き換えてください。 この値は、マニフェスト ファイルの **60** 行にあります。

```yaml
containers:
- name: azure-vote-front
  image: azuredocs/azure-vote-front
```

次に、[kubectl apply](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply) コマンドを使用して、アプリケーションを AKS クラスターにデプロイします。

```console
kubectl apply -f azure-vote-all-in-one-redis.yaml
```

アプリケーションをインターネットに公開するための Kubernetes ロード バランサー サービスが作成されます。 このプロセスには数分かかることがあります。 ロード バランサーのデプロイの進行状況を監視するには、[kubectl get service](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get) コマンドを使用し、`--watch` 引数を指定します。 *EXTERNAL-IP* アドレスが "*保留中*" から "*IP アドレス*" に変わったら、`Control + C` を使用して kubectl ウォッチ プロセスを停止します。

```console
$ kubectl get service azure-vote-front --watch

NAME               TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
azure-vote-front   LoadBalancer   10.0.215.27   <pending>     80:30747/TCP   22s
azure-vote-front   LoadBalancer   10.0.215.27   40.117.57.239   80:30747/TCP   2m
```

アプリケーションが動作していることを確認するには、Web ブラウザーを開いてサービスの外部 IP アドレスにアクセスします。 次の例に示すように、Azure 投票アプリケーションが表示されます。

![AKS で実行されているサンプルの Azure 投票アプリケーション](media/deploy-from-github-to-aks/azure-vote.png)

## <a name="configure-jenkins-controller"></a>Jenkins コントローラーの構成

次の変更を適用して、Jenkins コントローラーからの AKS デプロイを有効にします。

ポート `80` を受信用に開きます。

```azurecli
az vm open-port \
--resource-group <Resource_Group_name> \
--name <Jenkins_Controller_VM>  \
--port 80 --priority 1020
```

`<Resource_Group_name>` と `<Jenkins_Controller_VM>` を、適切な値に置き換えます。

Jenkins コントローラーに SSH で接続します。

```azurecli
ssh azureuser@<PublicIPAddres>
```

`<PublicIPAddress>`を Jenkins コントローラーの IP アドレスに置き換えます。

### <a name="install--log-into-azcli"></a>AzCLI のインストールとログイン

```azurecli
curl -L https://aka.ms/InstallAzureCli | bash
``````

```azurecli
az login
```

   > [!NOTE]
   > AzCLI を手動でインストールするには、これらの[手順](https://docs.microsoft.com/cli/azure/install-azure-cli)に従います。

### <a name="install-docker"></a>Docker のインストール

```bash
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common -y;
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -;
sudo apt-key fingerprint 0EBFCD88;
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable";
sudo apt-get update;
sudo apt-get install docker-ce -y;
```

### <a name="install-kubectl-and-connect-to-aks"></a>Kubectl をインストールして AKS に接続する

```azurecli
sudo az aks install-cli
sudo az aks get-credentials --resource-group <Resource_Group> --name <AKS_Name>
```

`<Resource_Group>` と `<AKS_Name>` を、適切な値に置き換えます。

### <a name="configure-access"></a>アクセスを構成する

```bash
sudo usermod -aG docker jenkins;
sudo usermod -aG docker azureuser;
sudo touch /var/lib/jenkins/jenkins.install.InstallUtil.lastExecVersion;
sudo service jenkins restart;
sudo cp ~/.kube/config /var/lib/jenkins/.kube/
sudo chmod 777 /var/lib/jenkins/
sudo chmod 777 /var/lib/jenkins/config
```

## <a name="create-a-jenkins-environment-variable"></a>Jenkins の環境変数を作成する

ACR ログイン サーバー名を保持するために Jenkins の環境変数が使用されます。 この変数は、Jenkins のビルド ジョブ中に参照されます。 この環境変数を作成するには、次の手順を完了します。

- Jenkins ポータルの左側にある **[Jenkins の管理]**  >  **[システムの構成]** を選択します。
- **[グローバル プロパティ]** の **[環境変数]** を選択します。 名前 `ACR_LOGINSERVER` と ACR ログイン サーバーの値で変数を追加します。

    ![Jenkins の環境変数](media/deploy-from-github-to-aks/env-variables.png)

- 完了したら、ページの下部にある **[保存]** を選択します。

## <a name="create-a-jenkins-credential-for-acr"></a>ACR の Jenkins 資格情報を作成する

CI/CD プロセス中に、Jenkins がアプリケーションの更新プログラムに基づいて新しいコンテナーのイメージをビルドし、その後これらのイメージを ACR レジストリに *プッシュ* する必要があります。

Jenkins が更新されたコンテナー イメージを ACR にプッシュできるようにするには、ACR の資格情報を指定する必要があります。 

ロールとアクセス許可を分離するために、ACR レジストリへの *Contributor* アクセス許可で Jenkins 用のサービス プリンシパルを構成します。

### <a name="create-a-service-principal-for-jenkins-to-use-acr"></a>ACR を使用する Jenkins 用のサービス プリンシパルを作成する

まず、[az ad sp create-for-rbac](/cli/azure/ad/sp#az-ad-sp-create-for-rbac) コマンドを使用して、サービス プリンシパルを作成します。

```azurecli
az ad sp create-for-rbac --skip-assignment
```

```bash
{
  "appId": "626dd8ea-042d-4043-a8df-4ef56273670f",
  "displayName": "azure-cli-2018-09-28-22-19-34",
  "name": "http://azure-cli-2018-09-28-22-19-34",
  "password": "1ceb4df3-c567-4fb6-955e-f95ac9460297",
  "tenant": "72f988bf-86f1-41af-91ab-2d7cd011db48"
}
```

*appId* と *password* を書き留めておいてください。 これらの値は、Jenkins で資格情報リソースを構成する次の手順で使用します。

[az acr show](/cli/azure/acr#az-acr-show) コマンドを使用して ACR レジストリのリソース ID を取得し、それを変数として保存します。

```azurecli
ACR_ID=$(az acr show --resource-group <Resource_Group> --name <acrLoginServer> --query "id" --output tsv)
```

`<Resource_Group>` と `<acrLoginServer>` を、適切な値に置き換えます。

サービス プリンシパルの *Contributor* 権限を ACR レジストリに割り当てるためのロールの割り当てを作成します。

```azurecli
az role assignment create --assignee <appID> --role Contributor --scope $ACR_ID
```

`<appId>`を、サービス プリンシパルの作成に使用した、前のコマンドの出力に指定された値に置き換えます。

### <a name="create-a-credential-resource-in-jenkins-for-the-acr-service-principal"></a>ACR サービス プリンシパル用の資格情報リソースを Jenkins に作成する

Azure で作成したロールの割り当てを使用して、ACR の資格情報を Jenkins 資格情報オブジェクトに保存します。 これらの資格情報は、Jenkins ビルド ジョブの間に参照されます。

Jenkins ポータルの左側に戻り、 **[Manage Jenkins]\(Jenkins の管理\)**  >  **[資格情報の管理]**  >  **[Jenkins Store]\(Jenkins ストア\)**  >  **[Global credentials (unrestricted)]\(グローバル資格情報 (無制限)\)**  >  **[Add Credentials]\(資格情報の追加\)** を選択します。

資格情報の種類が **[Username with password]\(ユーザー名とパスワード\)** であることを確認し、次の項目を入力します。

- **[Username]\(ユーザー名\)** - ACR レジストリでの認証に作成されるサービス プリンシパルの *appId* です。
- **パスワード** - ACR レジストリでの認証に作成されるサービス プリンシパルの *パスワード* です。
- **ID** - *acr-credentials* などの資格情報の識別子

完了すると、資格情報の形式は、次の例のようになります。

![サービス プリンシパル情報を使用して、Jenkins 資格情報オブジェクトを作成する](media/deploy-from-github-to-aks/acr-credentials.png)

**[OK]** を選択して、Jenkins ポータルに戻ります。

## <a name="create-a-jenkins-project"></a>Jenkins プロジェクトを作成する

Jenkins ポータルのホーム ページから左側の **[新しい項目]** を選択します。

1. ジョブ名として *azure-vote* を入力します。 **Freestyle プロジェクト** を選択し､ **[OK]** をクリックします
1. **[General]\(一般\)** セクションで **[GitHub project]\(GitHub プロジェクト\)** を選択し、フォークしたリポジトリの URL (例: *https:\//github.com/\<your-github-account\>/azure-voting-app-redis*) を入力します
1. **[Source code management]\(ソース コードの管理\)** セクションで **[Git]** を選択し、フォークしたリポジトリの `.git` の URL (例: *https:\//github.com/\<your-github-account\>/azure-voting-app-redis.git*) を入力します

1. **[Build Triggers]** セクションから **GitHub hook trigger for GITscm polling** を選択します
1. **[Build Environment]\(ビルド環境\)** で、 **[Use secret texts or files]\(シークレット テキストまたはファイルを使用する\)** を選びます
1. **[Bindings]\(バインド\)** で、 **[Add]\(追加\)**  >  **[Username and password (separated)]\(ユーザー名とパスワード (別)\)** を選びます
   - **[Username Variable]\(ユーザー名変数\)** に「`ACR_ID`」と入力し、 **[Password Variable]\(パスワード変数\)** に「`ACR_PASSWORD`」と入力します

     ![Jenkins のバインド](media/deploy-from-github-to-aks/bindings.png)

1. **実行シェル** タイプの **ビルド ステップ** を追加し、次のテキストを使います。 このスクリプトは、新しいコンテナー イメージをビルドし、ACR レジストリにプッシュします。

    ```bash
    # Build new image and push to ACR.
    WEB_IMAGE_NAME="${ACR_LOGINSERVER}/azure-vote-front:kube${BUILD_NUMBER}"
    docker build -t $WEB_IMAGE_NAME ./azure-vote
    docker login ${ACR_LOGINSERVER} -u ${ACR_ID} -p ${ACR_PASSWORD}
    docker push $WEB_IMAGE_NAME
    ```

1. **実行シェル** タイプの **ビルド ステップ** をもう 1 つ追加し、次のテキストを使います。 このスクリプトは、AKS のアプリケーションの展開を ACR からの新しいコンテナー イメージで更新します。

    ```bash
    # Update kubernetes deployment with new image.
    WEB_IMAGE_NAME="${ACR_LOGINSERVER}/azure-vote-front:kube${BUILD_NUMBER}"
    kubectl set image deployment/azure-vote-front azure-vote-front=$WEB_IMAGE_NAME
    ```

1. 終了したら、 **[Save]\(保存\)** をクリックします。

## <a name="test-the-jenkins-build"></a>Jenkins のビルドをテストする

GitHub のコミットに基づいてジョブを自動化する前に、Jenkins ビルドを手動でテストします。

このビルドにより、ジョブが正しく構成されていることを検証します。 適切な Kubernetes 認証ファイルが配置されていること、および ACR への認証が動作することが確認されます。

プロジェクトの左側にあるメニューの **[Build Now]\(今すぐビルド\)** をクリックします。

![Jenkins のテスト ビルド](media/deploy-from-github-to-aks/test-build.png)

Docker イメージ レイヤーが Jenkins サーバーにプルダウンされるため、最初のビルドは時間がかかります。

ビルドにより、次のタスクが実行されます。

1. GitHub リポジトリを複製する
1. 新しいコンテナー イメージをビルドする
1. ACR レジストリにコンテナー イメージをプッシュする
1. AKS デプロイで使用されるイメージを更新する

アプリケーション コードは変更されていないため、Web UI は変更されません。

ビルド ジョブが完了したら、ビルド履歴の下の **ビルド #1** を選択します。 **[コンソール出力]** を選択し、ビルド プロセスからの出力を表示します。 最後の行でビルドの成功が示されている必要があります。

## <a name="create-a-github-webhook"></a>GitHub Webhook の作成

手動ビルドが完了したら、ここで GitHub を Jenkins ビルドに組み込みます。 Webhook を使用して、コードが GitHub にコミットされるたびに Jenkins のビルド ジョブを実行します。

GitHub Webhook を作成するには、次の手順を完了します。

1. Web ブラウザーで、フォークされた GitHub リポジトリに移動します。
1. **[Settings]\(設定\)** を選択して、左側の **[Integrations & services]\(統合とサービス\)** を選択します。
1. **[Add webhook]\(Webhook の追加\)** を選択します。 *[Payload URL]\(ペイロード URL\)* に、「`http://<publicIp:8080>/github-webhook/`」と入力します。`<publicIp>` は、Jenkins サーバーの IP アドレスです。 必ず、末尾の `/` を含めるようにしてください。 他の規定値 ([Content type]\(コンテンツの種類\) と *push* イベントでトリガー) は既定値のままにします。
1. **[Add webhook]\(Webhook の追加\)** を選択します。

    ![Jenkins 用の GitHub Webhook を作成する](media/deploy-from-github-to-aks/webhook.png)

## <a name="test-the-complete-cicd-pipeline"></a>完全な CI/CD パイプラインをテストする

ここで CI/CD パイプライン全体をテストできます。 コード コミットを GitHub にプッシュすると、次のステップが行われます。

1. GitHub Webhook により、Jenkins に通知されます。
1. Jenkins がビルド ジョブを開始し、GitHub から最新のコード コミットをプルします。
1. Docker ビルドが更新されたコードを使用して開始され、新しいコンテナー イメージが最新のビルド番号でタグ付けされます。
1. この新しいコンテナー イメージが Azure Container Registry にプッシュされます。
1. Azure Kubernetes Service で実行されているアプリケーションが、Azure Container Registry からの最新イメージで更新されます。

開発マシンで、複製されたアプリケーションをコード エディターで開きます。 */azure-vote/azure-vote* ディレクトリで、**config_file.cfg** という名前のファイルを開きます。 次の例に示すように、このファイルの投票値を cats と dogs 以外のものに更新します。

```
# UI Configurations
TITLE = 'Azure Voting App'
VOTE1VALUE = 'Blue'
VOTE2VALUE = 'Purple'
SHOWHOST = 'false'
```

更新したら、ファイルを保存し、変更をコミットして、GitHub リポジトリのフォークにそれらをプッシュします。 GitHub Webhook は、Jenkins の新しいビルド ジョブをトリガーします。 Jenkins Web ダッシュ ボードでビルド プロセスを監視します。 最新のコードをプルし、更新されたイメージを作成してプッシュして、AKS で更新されたアプリケーションをデプロイするには数秒かかります。

ビルドが完了したら、サンプルの Azure 投票アプリケーションの Web ブラウザーを更新します。 次の例に示すように、変更内容が表示されます。

![Jenkins ビルド ジョブで更新された AKS のサンプル Azure 投票](media/deploy-from-github-to-aks/azure-vote-updated.png)

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [Azure 上の Jenkins](/azure/jenkins)
