---
title: JavaScript 開発者向けの Azure の主要タスク
description: 現在のタスクの例を参照してください。
ms.topic: how-to
ms.date: 03/03/2021
ms.custom: devx-track-js
ms.openlocfilehash: 22196c4e31c184748408dbbb340c59533ca8811f
ms.sourcegitcommit: 576c878c338d286060010646b96f3ad0fdbcb814
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102118180"
---
# <a name="top-tasks-for-javascript-developers"></a>JavaScript 開発者向けの主要タスク

現在のタスクの例を参照してください。 タスクが見つからない場合は、タスクを要求するフィードバックを残しておきます。 

## <a name="app-registration"></a>アプリの登録

[アプリの登録のドキュメント](/azure/active-directory/develop/quickstart-register-app)

|タスク|using|
|--|--|
|アプリの登録を作成する|[ポータル](../tutorial/single-page-application-azure-login-button-sdk-msal.md#3-create-app-registration-for-authentication)|

## <a name="application-settings"></a>アプリケーションの設定

|タスク|using|
|--|--|
|ローカル環境変数を設定する|[Bash](../tutorial/static-web-app/create-computer-vision-resource-use-in-code.md#add-environment-variables-to-your-local-environment)|
|Storage コンテナーの Shared Access Signature (SAS) トークンを作成する|[ポータル](../tutorial/browser-file-upload-azure-storage-blob.md#5-generate-your-shared-access-signature-sas-token)|
|コードに SAS トークンを設定する|[TypeScript](../tutorial/browser-file-upload-azure-storage-blob.md#set-sas-token-in-code-file)|
|Storage 用に CORS を構成する|[ポータル](../tutorial/browser-file-upload-azure-storage-blob.md#6-configure-cors-for-azure-storage-resource)|

## <a name="authentication"></a>認証

|タスク|using|
|--|--|
|`@azure/msal-browser` を使用した Microsoft ログイン/ログオフ ボタン|[React/TypeScript](../tutorial/single-page-application-azure-login-button-sdk-msal.md#5-add-login-and-logoff-buttons)|
|AAD のアクセス許可を取り消す|[https://myapplications.microsoft.com/](https://myapplications.microsoft.com/)|
|コンシューマーのアクセス許可を取り消す|[https://account.live.com/consent/manage](https://account.live.com/consent/manage)|

## <a name="azure-login"></a>Azure ログイン

|タスク|using|
|--|--|
|ログイン|[Azure CLI](../tutorial/deploy-deno-app-azure-app-service-azure-cli.md#2-sign-in-to-azure-cli)<br>[Visual Studio Code 拡張機能](../tutorial/tutorial-vscode-docker-node/tutorial-vscode-docker-node-01.md#sign-in-to-azure)|


## <a name="azure-resource-groups"></a>Azure リソース グループ

|タスク|using|
|--|--|
|リソース グループの作成|[Azure CLI](../tutorial/static-web-app/create-computer-vision-resource-use-in-code.md#create-azure-resources)|
|リソース グループの削除|[Azure CLI](../tutorial/static-web-app/clean-up-resources.md#remove-all-the-resources-by-removing-resource-group)|

## <a name="apps"></a>アプリ

### <a name="static-web-apps"></a>静的 Web アプリ

[サービスのドキュメント](/azure/static-web-apps/)

|タスク|using|
|--|--|
|Angular アプリの作成|[Bash](../tutorial/tutorial-vscode-static-website-node/tutorial-vscode-static-website-node-02.md?tabs=angular)|
|Deno アプリの作成|[Bash](../tutorial/deploy-deno-app-azure-app-service-azure-cli.md#3-create-local-deno-api-app)|
|JavaScript 言語を対象とする React アプリの作成|[Bash](../tutorial/tutorial-vscode-static-website-node/tutorial-vscode-static-website-node-02.md?tabs=react)|
|TypeScript 言語を対象とする React アプリの作成|[Bash](../tutorial/single-page-application-azure-login-button-sdk-msal.md#4-create-react-single-page-application-for-typescript)|
|Vue アプリの作成|[Bash](../tutorial/tutorial-vscode-static-website-node/tutorial-vscode-static-website-node-02.md?tabs=vue)|
|Svelte アプリの作成|[Bash](../tutorial/tutorial-vscode-static-website-node/tutorial-vscode-static-website-node-02.md?tabs=svelte)|
|静的 Web アプリを作成する|[Visual Studio Code 拡張機能](../tutorial/static-web-app/create-static-web-app-visual-studio-code-extension.md#create-a-static-web-app-resource)|
|ストレージでホストされる静的アプリの作成|[Visual Studio Code 拡張機能](../tutorial/tutorial-vscode-static-website-node/tutorial-vscode-static-website-node-03.md)|
|ストレージでホストされる静的アプリのデプロイ|[Visual Studio Code 拡張機能](../tutorial/tutorial-vscode-static-website-node/tutorial-vscode-static-website-node-04.md)|
|サイトの参照|[Visual Studio Code 拡張機能](../tutorial/static-web-app/create-static-web-app-visual-studio-code-extension.md#view-azure-static-web-site-in-browser)|

### <a name="function-serverless-apps"></a>関数 (サーバーレス) アプリ

[サービスのドキュメント](/azure/azure-functions/)

|タスク|using|
|--|--|
|Functions アプリをローカルで作成する|[Visual Studio Code 拡張機能](../tutorial/tutorial-vscode-serverless-node-create-local.md)|
|HTTP トリガー コード|[JavaScript](../tutorial/tutorial-vscode-serverless-node-create-local.md#http-function-javascript-template-code)|
|関数をローカルでデバッグ/テストする|[Visual Studio Code](../tutorial/tutorial-vscode-serverless-node-test-local.md)|
|関数を Azure クラウドにデプロイする|[Visual Studio Code 拡張機能](../tutorial/tutorial-vscode-serverless-node-deploy-hosting.md)|
|関数がパブリック URL で使用可能であることを確認する|[Visual Studio Code 拡張機能/ブラウザー](../tutorial/tutorial-vscode-serverless-node-deploy-hosting.md#verify-functions-app-is-publicly-available-with-browser)|
|関数アプリ リソースを削除する|[Visual Studio Code 拡張機能](../tutorial/tutorial-vscode-serverless-node-remove-resource.md)|

### <a name="app-service---full-stack-server-only-or-client-only-apps"></a>アプリ サービス - フル スタック、サーバーのみ、またはクライアントのみのアプリ

[サービスのドキュメント](/azure/app-service/)

|タスク|using|
|--|--|
|ローカル環境で Express.js アプリを作成する|[Bash](../tutorial/deploy-nodejs-azure-app-service-with-visual-studio-code.md?tabs=bash#3-create-a-local-expressjs-app)|
|アプリ リソースの作成 - Express.js アプリのデプロイ、ログのストリーミングを含む|[Visual Studio Code 拡張機能](../tutorial/deploy-nodejs-mongodb-app-service-from-visual-studio-code.md#create-web-app-resource-and-deploy-expressjs-app)|
|アプリ リソースの作成 - Express.js アプリのデプロイ、アプリ設定の構成、npm インストールの実行、デプロイされた Web サイトの参照を含む|[Visual Studio Code 拡張機能](../tutorial/deploy-nodejs-azure-app-service-with-visual-studio-code.md?tabs=bash#6-create-app-service-resource-in-visual-studio-code)|
|アプリ リソースを作成する|[Azure CLI](../tutorial/tutorial-vscode-azure-cli-node/tutorial-vscode-azure-cli-node-03.md)|
|アプリの作成、デプロイ、ブラウザー アプリ、ログの表示|[Azure CLI](../tutorial/tutorial-vscode-azure-cli-node/tutorial-vscode-azure-cli-node-03.md)|
|データベース接続文字列を使用するように Web アプリを構成する|[Azure CLI](./with-azure-cli/create-mongodb-cosmosdb.md#configure-your-azure-web-app-with-the-connection-string)|
|コンテナーを使用するように Web アプリを構成する|[Azure CLI](./with-azure-cli/create-container-registry-resource.md#configure-web-app-to-use-container)|
|Web アプリのカスタム ドメイン名を構成する|[Azure CLI](./with-azure-cli/configure-app-service-custom-domain-name.md#register-a-domain-name-with-your-azure-app)|
|アプリ リソースを削除する|[Visual Studio Code 拡張機能](../tutorial/deploy-nodejs-mongodb-app-service-from-visual-studio-code.md#clean-up-resources)<br>[Azure CLI](../tutorial/tutorial-vscode-azure-cli-node/tutorial-vscode-azure-cli-node-07.md)|
|アプリをデプロイまたは再デプロイする|[Visual Studio Code 拡張機能](deploy-web-app.md#deploy-or-redeploy-to-app-service-with-visual-studio-code)|
|Web アプリの外部 IP を取得する|[Azure CLI](./with-azure-cli/configure-app-service-custom-domain-name.md#register-a-domain-name-with-your-azure-app)|
|ドメイン名を購入して DNS レコードを構成する|[Azure CLI](./with-azure-cli/configure-app-service-custom-domain-name.md#purchase-a-domain-name-and-configure-dns-record)|
|リモート ログをストリーミングする|[Visual Studio Code 拡張機能](../tutorial/deploy-nodejs-azure-app-service-with-visual-studio-code.md?tabs=bash#7-stream-remote-service-logs-in-visual-studio-code)<br>[Azure CLI](../tutorial/tutorial-vscode-azure-cli-node/tutorial-vscode-azure-cli-node-05.md)|

## <a name="cognitive-services"></a>Cognitive Services

[サービス グループのドキュメント](/azure/cognitive-services/)

|タスク|using|
|--|--|
|Cognitive Services _ComputerVision_ リソースを作成する|[Azure CLI](../tutorial/static-web-app/create-computer-vision-resource-use-in-code.md#create-azure-resources)|
|Cognitive Services _ComputerVision_ リソースを取得する|[Azure CLI](../tutorial/static-web-app/create-computer-vision-resource-use-in-code.md#create-azure-resources)|
|Azure SDK をインストールする|[Bash](../tutorial/static-web-app/add-computer-vision-react-app.md#add-computer-vision-to-local-react-app)|
|[`@azure/cognitiveservices-computervision`](https://www.npmjs.com/package/@azure/cognitiveservices-computervision) を使用して画像を分析する|[Visual Studio Code](../tutorial/static-web-app/add-computer-vision-react-app.md#add-computer-vision-code-as-separate-module)|

## <a name="containers-including-docker-tasks"></a>Docker タスクを含むコンテナー

|タスク|using|
|--|--|
|Docker ファイルをローカル プロジェクトに追加する|[Visual Studio Code 拡張機能](../tutorial/tutorial-vscode-docker-node/tutorial-vscode-docker-node-04.md#add-docker-files)|
|ローカル プロジェクトから Docker イメージを構築する|[Visual Studio Code 拡張機能](../tutorial/tutorial-vscode-docker-node/tutorial-vscode-docker-node-04.md#build-a-docker-image)|
|ローカル JavaScript プロジェクトからコンテナー イメージを作成する|[Visual Studio Code](./with-visual-studio-code/containerize-local-project.md#create-a-container)|
|コンテナー レジストリ リソースを作成する|[Visual Studio Code 拡張機能](../tutorial/tutorial-vscode-docker-node/tutorial-vscode-docker-node-02.md#create-an-azure-container-registry)<br>[Azure CLI](./with-azure-cli/create-container-registry-resource.md#create-a-container-registry)|
|Dockerfile を作成します。|[Visual Studio Code 拡張機能](./with-visual-studio-code/containerize-local-project.md#create-a-dockerfile-in-your-project)|
|イメージをアプリ サービスにデプロイする|[Visual Studio Code 拡張機能](../tutorial/tutorial-vscode-docker-node/tutorial-vscode-docker-node-05.md#deploy-image)|
|レジストリへの管理者アクセスを有効にする|[Visual Studio Code 拡張機能](../tutorial/tutorial-vscode-docker-node/tutorial-vscode-docker-node-05.md#enable-admin-access-on-the-registry)|
|Azure コンテナー レジストリの資格情報を取得する|[Azure CLI](./with-azure-cli/create-container-registry-resource.md#get-container-registry-credentials)|
|コンテナー レジストリへのログイン|[BASH - Docker CLI](./with-azure-cli/create-container-registry-resource.md#login-to-container-registry-with-docker-cli)|
|Docker レジストリ リソースにイメージをプッシュする|[Visual Studio Code 拡張機能](./with-visual-studio-code/containerize-local-project.md#push-local-container-image-to-dockerhub)|
|Azure Container Registry リソースにイメージをプッシュする|[Visual Studio Code 拡張機能](../tutorial/tutorial-vscode-docker-node/tutorial-vscode-docker-node-04.md#push-the-image-to-a-registry)<BR>[BASH - Docker CLI](./with-azure-cli/create-container-registry-resource.md#push-your-local-image-to-your-container-registry)|
|ローカル コンテナーを実行する|[Visual Studio Code 拡張機能](with-visual-studio-code/containerize-local-project.md#build-image-from-your-project)|
|ローカル イメージにタグを付ける|[BASH - Docker CLI](./with-azure-cli/create-container-registry-resource.md#tag-your-local-image)|
|Docker のバージョンを確認する|[Bash](../tutorial/tutorial-vscode-docker-node/tutorial-vscode-docker-node-01.md#verify-docker-install)|

## <a name="databases"></a>データベース

### <a name="cassandra-api-on-cosmos-db"></a>Cosmos DB での Cassandra API

[サービスのドキュメント](/azure/cosmos-db/)

|タスク|使用|
|--|--|
|リソースを作成する|[Azure Portal](https://ms.portal.azure.com/#create/Microsoft.DocumentDB)<br>[Azure CLI](./with-azure-cli/create-cassandra-db.md#create-a-cosmos-db-resource-for-cassandra-db)|
|リソースにキーストアを作成する|[Azure CLI](./with-azure-cli/create-cassandra-db.md#create-a-keyspace-on-the-server-with-azure-cli)|
|キーストアにテーブルを作成する|[Azure CLI](./with-azure-cli/create-cassandra-db.md#create-a-table-on-the-keyspace-with-azure-cli)|
|接続情報の取得|[Azure CLI](./with-azure-cli/create-cassandra-db.md#get-the-cassandra-connection-string-with-azure-cli)|
|Cosmos DB で cassandra-driver API を使用する|[JavaScript](/azure/developer/javascript/how-to/with-database/use-cassandra-as-cosmos-db.md#use-cassandra-driver-sdk-to-connect-to-cassandra-db-on-azure)|

### <a name="mariadb"></a>MariaDB

[サービスのドキュメント](/azure/mariadb/)

|タスク|使用|
|--|--|
|MariaDB リソースを作成する|[Azure Portal](https://ms.portal.azure.com/#create/Microsoft.MariaDBServer)<br>[Azure CLI](./with-azure-cli/create-mariadb.md#create-a-mariadb-resource-with-azure-cli)<br>[@azure/arm-mariadb](https://www.npmjs.com/package/@azure/arm-mariadb)|
|リソース上に MariaDB データベースを作成する|[Azure CLI](./with-azure-cli/create-mariadb.md#create-a-mariadb-resource-with-azure-cli)|
|接続文字列を取得する|[Azure CLI](./with-azure-cli/create-mariadb.md#get-the-mariadb-connection-string-with-azure-cli)|
|データベースを使用および表示する|[Azure Cloud Shell](https://shell.azure.com/) の _mysql_ CLI<br>[MySQL Workbench](https://www.mysql.com/products/workbench/)<br>[Visual Studio Code 拡張機能](https://marketplace.visualstudio.com/items?itemName=mtxr.sqltools-driver-mysql)<br>[npm mariadb](https://www.npmjs.com/package/mariadb)<br>[JavaScript](./with-database/use-mariadb.md#use-mariadb-sdk-to-connect-to-mariadb-on-azure)|

### <a name="mongodb-api-on-cosmos-db"></a>Cosmos DB での MongoDB API

[サービスのドキュメント](/azure/cosmos-db/)

|タスク|using|
|--|--|
|Cosmos DB の作成 - MongoDB リソース|[Visual Studio Code 拡張機能](../tutorial/deploy-nodejs-mongodb-app-service-from-visual-studio-code.md)<br>[Azure CLI](./with-azure-cli/create-mongodb-cosmosdb.md#create-a-cosmos-db-resource-for-mongodb)|
|Cosmos DB の接続文字列を取得する|[Visual Studio Code 拡張機能](../tutorial/deploy-nodejs-mongodb-app-service-from-visual-studio-code.md#get-cosmos-db-connection-string)<br>[Azure CLI](./with-azure-cli/create-mongodb-cosmosdb.md#get-the-mongodb-connection-string-for-your-resource)|
|Cosmos DB の表示|[Cosmos DB Explorer](https://cosmos.azure.com/)|
|Cosmos DB で MongoDB 用 Mongoose API を使用する|[JavaScript](./with-database/use-mongodb-as-cosmosdb.md#use-mongoose-sdk-to-connect-to-mongodb-on-azure)

### <a name="mysql"></a>MySQL

[サービスのドキュメント](/azure/mysql/)

|タスク|使用|
|--|--|
|リソースを作成する|[Azure Portal](https://ms.portal.azure.com/#create/Microsoft.MySQLServer)<br>[Azure CLI](./with-azure-cli/create-mysql-db.md#create-an-azure-database-for-mysql-resource-with-azure-cli)<br>[@azure/arm-mysql](https://www.npmjs.com/package/@azure/arm-mysql)|
|リソースにデータベースを作成する|[Azure CLI](./with-database/use-mysql-db.md#create-a-database-on-the-server-with-azure-cli)|
|接続文字列を取得する|[Azure CLI](./with-database/use-mysql-db.md#get-the-mysql-connection-string-with-azure-cli)|
|データベースを使用および表示する|[MySQL Workbench](https://www.mysql.com/products/workbench/)<br>[Visual Studio Code 拡張機能](https://marketplace.visualstudio.com/items?itemName=mtxr.sqltools-driver-mysql)<br>[npm mysql](https://www.npmjs.com/package/MySQL)<br>[npm promise-mysql](https://www.npmjs.com/package/promise-mysql)|
|promise-mysql API を使用する|[JavaScript](./with-database/use-mysql-db.md#use-promise-mysql-sdk-to-connect-to-mysql-on-azure)|

### <a name="postgresql"></a>PostgreSQL

[サービスのドキュメント](/azure/postgresql/)

|タスク|using|
|--|--|
|リソースを作成する|[Visual Studio Code 拡張機能](./with-visual-studio-code/create-azure-database.md#create-a-postgresql-database)<br>[Azure CLI](./with-azure-cli/create-postgresql-server-resource.md#create-an-azure-database-for-postgresql-server-resource-with-azure-cli)<br>[Azure Portal](https://ms.portal.azure.com/#create/Microsoft.PostgreSQLServer)<br>[@azure/arm-postgresql](https://www.npmjs.com/package/@azure/arm-postgresql)|
|接続文字列を取得する|[Azure CLI](./with-azure-cli/create-postgresql-server-resource.md#get-the-postgresql-connection-string-with-azure-cli)|
|DB を表示する|[Visual Studio Code 拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-cosmosdb)<br>[Azure Cloud Shell の psql](https://shell.azure.com/)|
|DB に pg API を使用する|[JavaScript](./with-database/use-postgresql-db.md#use-pg-sdk-to-connect-to-postgresql-on-azure)

### <a name="sql-api-on-cosmos-db"></a>Cosmos DB での SQL API

* [サービスのドキュメント](/azure/cosmos-db/)
* [@azure/cosmosdb](https://www.npmjs.com/package/@azure/cosmos) npm パッケージ

|タスク|using|
|--|--|
|クライアント IP アドレスのファイアウォール規則を追加する|[Azure CLI](./with-database/use-sql-api-as-cosmos-db.md#add-firewall-rule-for-your-client-ip-address)
|Cosmos DB を作成する - SQL API リソース|[Visual Studio Code 拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-cosmosdb)<br>[Azure CLI](./with-database/use-sql-api-as-cosmos-db.md#create-a-cosmos-db-resource-for-sql-api)|
|Cosmos DB キーを取得する|[Azure CLI](./with-database/use-sql-api-as-cosmos-db.md#get-the-cosmos-db-keys-for-your-resource)|
|Cosmos DB の接続文字列を取得する|[Visual Studio Code 拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-cosmosdb)|
|Cosmos DB の表示|[Cosmos DB Explorer](https://cosmos.azure.com/)|
|Cosmos DB に SQL API を使用する|[JavaScript](./with-database/use-sql-api-as-cosmos-db.md#use--sdk-to-connect-to-database)



## <a name="git"></a>Git

|タスク|using|
|--|--|
|ローカル Git リポジトリを初期化する|[Visual Studio Code 拡張機能](../tutorial/deploy-nodejs-azure-app-service-with-visual-studio-code.md?tabs=bash#5-initialize-git-in-visual-studio-code-for-current-app)|
|ローカル ブランチを作成する|[Visual Studio Code のコマンド パレット](./with-visual-studio-code/clone-github-repository.md#create-a-branch-for-changes-with-git-cl)<br>[Visual Studio Code のステータス バー](./with-visual-studio-code/clone-github-repository.md#create-a-branch-from-status-bar)|
|GitHub からローカル コンピューターにプロジェクトを複製する|[Visual Studio Code](with-visual-studio-code/install-run-debug-nodejs.md#clone-sample-project-to-local-computer)|
|ローカル ブランチをリモート リポジトリにプッシュする|[Visual Studio Code のステータス バー](./with-visual-studio-code/clone-github-repository.md#push-a-local-branch-to-remote-from-status-bar)<br>[Visual Studio Code のソース管理拡張機能](./with-visual-studio-code/clone-github-repository.md#push-a-local-branch-to-remote-from-the-source-control-extension)|

## <a name="github"></a>GitHub 

### <a name="actions"></a>アクション 

|タスク|using|
|--|--|
|シークレットを追加する|[Visual Studio Code](../tutorial/static-web-app/create-static-web-app-visual-studio-code-extension.md#create-a-static-web-app-resource)|
|ビルド プロセスを表示する|[GitHub Web サイト](../tutorial/static-web-app/create-static-web-app-visual-studio-code-extension.md#view-the-github-action-build-process)|


## <a name="monitoring"></a>監視

|タスク|using|
|--|--|
|リソースを作成する|[Azure CLI](../tutorial/nodejs-virtual-machine-vm/create-azure-monitoring-application-insights-web-resource.md#create-azure-monitor-resource-with-azure-cli)|



## <a name="storage"></a>ストレージ

[サービスのドキュメント](/azure/storage/)

|タスク|using|
|--|--|
|リソースを作成する|[Visual Studio Code 拡張機能](../tutorial/browser-file-upload-azure-storage-blob.md#3-create-storage-resource-with-visual-studio-extension)|
|Delete resource|[Visual Studio Code 拡張機能](../tutorial/browser-file-upload-azure-storage-blob.md#clean-up-resources)|
|ストレージでホストされる静的アプリの作成|[Visual Studio Code 拡張機能](../tutorial/tutorial-vscode-static-website-node/tutorial-vscode-static-website-node-03.md)|
|ストレージでホストされる静的アプリのデプロイ|[Visual Studio Code 拡張機能](../tutorial/tutorial-vscode-static-website-node/tutorial-vscode-static-website-node-04.md)|

### <a name="blobs"></a>BLOB

|タスク|using|
|--|--|
|[`@azure/storage-blob`](https://www.npmjs.com/package/@azure/storage-blob) を使用してストレージにコンテナーを作成する|[React/TypeScript](../tutorial/browser-file-upload-azure-storage-blob.md#create-storage-client-and-manage-steps)|
|[`@azure/storage-blob`](https://www.npmjs.com/package/@azure/storage-blob) を使用してファイルをストレージにアップロードする|[React/TypeScript](../tutorial/browser-file-upload-azure-storage-blob.md#upload-button-functionality)|
|[`@azure/storage-blob`](https://www.npmjs.com/package/@azure/storage-blob) を使用して Storage コンテナー内のファイルを一覧表示する|[React/TypeScript](../tutorial/browser-file-upload-azure-storage-blob.md#get-list-of-blobs)|

## <a name="terminal-usage"></a>ターミナルの使用法

|タスク|using|
|--|--|
|統合ターミナル|[Visual Studio Code](./with-visual-studio-code/install-run-debug-nodejs.md#use-the-integrated-bash-terminal-to-install-dependencies)|

## <a name="virtual-machines"></a>仮想マシン

[サービスのドキュメント](/azure/virtual-machines/)

|タスク|using|
|--|--|
|SSH を使用して VM に接続する|[Bash](../tutorial/nodejs-virtual-machine-vm/connect-linux-virtual-machine-ssh.md#connect-with-ssh-and-change-web-app)|
|Monitoring SDK のインストール|[Bash](../tutorial/nodejs-virtual-machine-vm/connect-linux-virtual-machine-ssh.md#install-monitoring-sdk)|
|Express.js アプリに監視コードを追加する|[JavaScript](../tutorial/nodejs-virtual-machine-vm/azure-monitor-application-insights-nodejs-expressjs-code.md#edit-indexjs-for-logging-with-azure-monitor-application-insights)|
|cloud-init ファイルを作成する|[YAML](../tutorial/nodejs-virtual-machine-vm/create-linux-virtual-machine-azure-cli.md#create-a-cloud-init-file-to-expedite-linux-virtual-machine-creation)|
|Linux VM リソースを作成する|[Azure CLI](../tutorial/nodejs-virtual-machine-vm/create-linux-virtual-machine-azure-cli.md#create-a-virtual-machine-resource)|
|Linux VM のポートを開く|[Azure CLI](../tutorial/nodejs-virtual-machine-vm/create-linux-virtual-machine-azure-cli.md#open-port-for-virtual-machine)|
|ログを表示する|[Azure CLI](../tutorial/nodejs-virtual-machine-vm/azure-monitor-application-insights-nodejs-expressjs-code.md#viewing-the-vm-logs-for-nginx-and-pm2)<br>[ポータル](../tutorial/nodejs-virtual-machine-vm/azure-monitor-application-insights-logs.md#view-application-traces-in-azure-portal)|


## <a name="visual-studio-code-develop-and-debug-javascript-apps"></a>Visual Studio Code:JavaScript アプリの開発とデバッグ 

[ツールのドキュメント](https://code.visualstudio.com/docs)

|タスク|using|
|--|--|
|コード補完|[Visual Studio Code](./with-visual-studio-code/install-run-debug-nodejs.md#use-visual-studio-code-autocompletion-with-mongodb)|
|ローカル Node.js アプリをデバッグする|[Visual Studio Code](./with-visual-studio-code/install-run-debug-nodejs.md#debugging-the-local-nodejs-app)|
|ローカルのフルスタック デバッグ|[Visual Studio Code](with-visual-studio-code/install-run-debug-nodejs.md#local-full-stack-debugging-in-visual-studio-code)|
|プロジェクト ファイルとコード内を移動する|[Visual Studio Code](./with-visual-studio-code/install-run-debug-nodejs.md#navigate-the-project-files-and-code)|
|ローカル Node.js アプリを実行する|[Visual Studio Code](./with-visual-studio-code/install-run-debug-nodejs.md#running-the-local-nodejs-app)|

## <a name="samples-supporting-these-tasks"></a>これらのタスクをサポートするサンプル

|Name | 説明|
|--|--|
|Cognitive Services を使用する React アプリ|GitHub アクションを使用して、ローカル環境で React/TypeScript クライアント アプリケーションを構築し、Azure Static Web アプリにデプロイします。<br>[チュートリアル](../tutorial/static-web-app/introduction.md) - [サンプル コード](https://github.com/Azure-Samples/js-e2e-client-cognitive-services)|
|ファイルを Azure Storage Blob にアップロードする React アプリ|このサンプル プロジェクトは、Azure Storage BLOB にアップロードするファイルを選択するための HTML フォームを使用した、TypeScript React (create-react-app) フレームワーク クライアント アプリです。<br>[チュートリアル](../tutorial/browser-file-upload-azure-storage-blob.md) - [サンプル コード](https://github.com/Azure-Samples/js-e2e-browser-file-upload-storage-blob)|
|ログイン ボタンを使用する React アプリ|このチュートリアルで構築する SPA は、次のタスクを行う React アプリ (create-react-app) です。<br>* Office 365 や Outlook.com など、Microsoft がサポートするログインを使用してログインする<br>* アプリケーションからログオフする<br>[チュートリアル](../tutorial/single-page-application-azure-login-button-sdk-msal.md) - [サンプル コード](https://github.com/Azure-Samples/js-e2e-client-azure-login-button)|
|MongoDB データベースを使用する Express js アプリ|このチュートリアルでは、拡張機能を使用して VSCode でプロジェクトをローカルに読み込んで実行する方法と、App Service でコードをリモートで実行する方法について説明します。 このチュートリアルでは、Mongo API 用に CosmosDB リソースを作成し、接続情報を取得して、クラウド データベースに接続するように App Service 構成で設定します。<br>[チュートリアル](../tutorial/deploy-nodejs-mongodb-app-service-from-visual-studio-code.md) - [サンプル コード](https://github.com/Azure-Samples/js-e2e-express-mongo)|
|cloud-init ファイルを使用して VM にデプロイされる Express js アプリ|Express.js アプリ用の Linux 仮想マシン (VM) を作成します。 この VM は、cloud-init 構成ファイルで構成され、NGINX と Express.js アプリの GitHub リポジトリが含まれています。 VM が実行されたら、SSH を使用して VM に接続し、トレースのログ記録を含めるように Web アプリを変更して、公開された Express.js サーバー アプリを Web ブラウザーで表示できます。<br>[チュートリアル](../tutorial/nodejs-virtual-machine-vm/introduction.md) - [サンプル コード](https://github.com/Azure-Samples/js-e2e-express-mongo)|

ご自身の特定のユース ケースをサポートするその他のサンプルを見つけるには、[Azure サンプル ブラウザー](/samples/browse/?languages=javascript%2cnodejs%2ctypescript)を使用してください。 

## <a name="next-steps"></a>次のステップ

* [Web アプリをデプロイする](deploy-web-app.md)