---
title: Azure SDK for Java を使用する作業の開始
description: Azure クラウド リソースを作成し、それらのリソースに接続して Java アプリケーションで使用する方法について説明します。
keywords: Azure, Java, SDK, API, 認証, 概要
author: bmitchell287
ms.author: brendm
ms.date: 11/20/2020
ms.topic: article
ms.service: multiple
ms.assetid: b1e10b79-f75e-4605-aecd-eed64873e2d3
ms.custom: seo-java-august2019, devx-track-java, devx-track-azurecli
ms.openlocfilehash: 5de0885c0fc3ac8ca2664b5b4da36fd97be3948a
ms.sourcegitcommit: 71847ee0a1fee3f3320503629d9a8c82319a1f6a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/03/2021
ms.locfileid: "99528719"
---
# <a name="get-started-with-cloud-development-using-java-on-azure"></a>Azure での Java を使用したクラウド開発の開始

この記事では、Java での Azure 開発用の開発環境を設定する手順について説明します。 Azure リソースをいくつか作成し、それらに接続して基本的なタスク (ファイルのアップロードや Web アプリケーションのデプロイなど) を行います。 作業が完了すると、独自の Java アプリケーションで Azure サービスの使用を開始できるようになります。

## <a name="prerequisites"></a>前提条件

- Azure アカウント。 所有していない場合は、[無料試用版を入手](https://azure.microsoft.com/free/)してください。
- [Azure Cloud Shell](/azure/cloud-shell/quickstart) または [Azure CLI 2.0](/cli/azure/install-az-cli2)。
- [Java 8](../fundamentals/java-jdk-long-term-support.md) (Azure Cloud Shell に付属)。
- [Maven 3](https://maven.apache.org/download.cgi) (Azure Cloud Shell に付属)。

## <a name="set-up-authentication"></a>認証の設定

このチュートリアルのサンプル コードを実行する Java アプリケーションには、Azure サブスクリプションの読み取りと作成のアクセス許可が必要です。 サービス プリンシパルを作成し、その資格情報で動作するようにアプリケーションを構成してください。 サービス プリンシパルによって、自分の ID に関連付けられた非対話型のアカウントを作成し、アプリの実行に必要な権限だけを付与できます。

[Azure CLI 2.0 を使ってサービス プリンシパルを作成](/cli/azure/create-an-azure-service-principal-azure-cli)し、その出力をキャプチャしてください。

```azurecli-interactive
az ad sp create-for-rbac --name AzureJavaTest
```

次の形式で応答が返されます。

```json
{
  "appId": "a487e0c1-82af-47d9-9a0b-af184eb87646d",
  "displayName": "AzureJavaTest",
  "name": "http://AzureJavaTest",
  "password": "aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa",
  "tenant": "tttttttt-tttt-tttt-tttt-tttttttttttt"
}
```

次に、環境変数を構成します。

- `AZURE_SUBSCRIPTION_ID`:Azure CLI 2.0 の `az account show` から得られる *id* 値を使用します。
- `AZURE_CLIENT_ID`:サービス プリンシパルの出力から得られる *appId* 値を使用します。
- `AZURE_CLIENT_SECRET`:サービス プリンシパルの出力から得られる *password* 値を使用します。
- `AZURE_TENANT_ID`:サービス プリンシパルの出力から得られる *tenant* 値を使用します。

その他の認証オプションについては、「[Java 用 Azure ID クライアント ライブラリ](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/identity/azure-identity#azure-identity-client-library-for-java)」を参照してください。

## <a name="tooling"></a>ツール

### <a name="create-a-new-maven-project"></a>新しい Maven プロジェクトを作成する

> [!NOTE]
> この記事では Maven ビルド ツールを使用して、サンプル コードをビルドして実行します。 Gradle などの他のビルド ツールも、Azure SDK for Java で動作します。

ご利用のシステム上の新しいディレクトリに、コマンド ラインから Maven プロジェクトを作成します。

```shell
mkdir java-azure-test
cd java-azure-test
mvn archetype:generate -DgroupId=com.fabrikam -DartifactId=AzureApp \
-DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

この手順により、基本的な Maven プロジェクトが *testAzureApp* ディレクトリに作成されます。 プロジェクトの *pom.xml* ファイルに次のエントリを追加して、このチュートリアルのサンプル コードで使用するライブラリをインポートします。

```XML
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-identity</artifactId>
    <version>1.2.0</version>
</dependency>
<dependency>
    <groupId>com.azure.resourcemanager</groupId>
    <artifactId>azure-resourcemanager</artifactId>
    <version>2.1.0</version>
</dependency>
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-storage-blob</artifactId>
    <version>12.8.0</version>
</dependency>
<dependency>
    <groupId>com.microsoft.sqlserver</groupId>
    <artifactId>mssql-jdbc</artifactId>
    <version>6.2.1.jre8</version>
</dependency>
```

[maven-exec-plugin](https://www.mojohaus.org/exec-maven-plugin/) を使ってサンプルを実行するために、最上位の `project` 要素に `build` エントリを追加します。

```XML
<build>
    <plugins>
        <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>exec-maven-plugin</artifactId>
            <configuration>
                <mainClass>com.fabrikam.AzureApp</mainClass>
            </configuration>
        </plugin>
    </plugins>
</build>
 ```

### <a name="install-the-azure-toolkit-for-intellij"></a>Azure Toolkit for IntelliJ をインストールする

Web アプリまたは API をプログラムを使用してデプロイする場合は、[Azure ツールキット](../toolkit-for-intellij/index.yml)が必要です。 これは現時点では、他の種類の開発には使用されていません。 次の手順は、インストール プロセスをまとめたものです。 クイック スタートについては、[Azure Toolkit for IntelliJ](../toolkit-for-intellij/create-hello-world-web-app.md) に関するページをご覧ください。

1. **[File]\(ファイル\)** メニューを選択し、 **[Settings]\(設定\)** を選択します。
1. **[Browse repositories]\(リポジトリを参照\)** を選択し、"**Azure**" を検索して、**Azure Toolkit for Intellij** をインストールします。
1. Intellij を再起動します。

### <a name="install-the-azure-toolkit-for-eclipse"></a>Azure Toolkit for Eclipse をインストールする

Web アプリまたは API をプログラムを使用してデプロイする場合は、[Azure ツールキット](../toolkit-for-eclipse/index.yml)が必要です。 これは現時点では、他の種類の開発には使用されていません。 次の手順は、インストール プロセスをまとめたものです。 クイック スタートについては、[Azure Toolkit for Eclipse](../toolkit-for-eclipse/create-hello-world-web-app.md) に関するページをご覧ください。

1. **[Help]\(ヘルプ\)** メニューを選択し、 **[Install new software]\(新しいソフトウェアのインストール\)** を選択します。
1. **[Work with]\(作業対象\)** ボックスに「`http://dl.microsoft.com/eclipse/`」と入力し、**Enter** キーを押します。
1. **[Azure toolkit for Java]** の横にあるチェック ボックスをオンにします。 **[Contact all update sites during install to find required software]\(インストール中にすべての更新サイトに接続して必要なソフトウェアを調べる\)** のチェック ボックスをオフにします。 **[次へ]** を選択します。

## <a name="create-a-linux-virtual-machine"></a>Linux 仮想マシンの作成

プロジェクトの *src/main/java/com/fabrikam* ディレクトリに *AzureApp.java* という名前の新しいファイルを作成し、次のコード ブロックを貼り付けます。 `userName` 変数と `sshKey` 変数は、ご利用のマシンの実際の値に置き換えてください。 このコードによって、米国東部 Azure リージョンで実行されているリソース グループ `sampleResourceGroup` に、`testLinuxVM` という名前の新しい Linux 仮想マシン (VM) が作成されます。

```java
package com.fabrikam;

import com.azure.core.credential.TokenCredential;
import com.azure.core.http.policy.HttpLogDetailLevel;
import com.azure.core.management.AzureEnvironment;
import com.azure.core.management.Region;
import com.azure.core.management.profile.AzureProfile;
import com.azure.identity.AzureAuthorityHosts;
import com.azure.identity.EnvironmentCredentialBuilder;
import com.azure.resourcemanager.AzureResourceManager;
import com.azure.resourcemanager.compute.models.KnownLinuxVirtualMachineImage;
import com.azure.resourcemanager.compute.models.VirtualMachine;
import com.azure.resourcemanager.compute.models.VirtualMachineSizeTypes;

public class AzureApp {

    public static void main(String[] args) {

        final String userName = "YOUR_VM_USERNAME";
        final String sshKey = "YOUR_PUBLIC_SSH_KEY";

        try {
            TokenCredential credential = new EnvironmentCredentialBuilder()
                    .authorityHost(AzureAuthorityHosts.AZURE_PUBLIC_CLOUD)
                    .build();

            // If you don't set the tenant ID and subscription ID via environment variables,
            // change to create the Azure profile with tenantId, subscriptionId, and Azure environment.
            AzureProfile profile = new AzureProfile(AzureEnvironment.AZURE);

            AzureResourceManager azureResourceManager = AzureResourceManager.configure()
                    .withLogLevel(HttpLogDetailLevel.BASIC)
                    .authenticate(credential, profile)
                    .withDefaultSubscription();

            // Create an Ubuntu virtual machine in a new resource group.
            VirtualMachine linuxVM = azureResourceManager.virtualMachines().define("testLinuxVM")
                    .withRegion(Region.US_EAST)
                    .withNewResourceGroup("sampleVmResourceGroup")
                    .withNewPrimaryNetwork("10.0.0.0/24")
                    .withPrimaryPrivateIPAddressDynamic()
                    .withoutPrimaryPublicIPAddress()
                    .withPopularLinuxImage(KnownLinuxVirtualMachineImage.UBUNTU_SERVER_16_04_LTS)
                    .withRootUsername(userName)
                    .withSsh(sshKey)
                    .withUnmanagedDisks()
                    .withSize(VirtualMachineSizeTypes.STANDARD_D3_V2)
                    .create();

        } catch (Exception e) {
            System.out.println(e.getMessage());
            e.printStackTrace();
        }
    }
}
```

コマンド ラインでサンプルを実行します。

```shell
mvn compile exec:java
```

コンソールには、REST の要求と応答がいくつか表示されます。これは、SDK が内部的に Azure REST API を呼び出して、VM とそのリソースを構成しているためです。 プログラムの実行が完了したら、サブスクリプション内の VM を Azure CLI 2.0 で確認します。

```azurecli-interactive
az vm list --resource-group sampleVmResourceGroup
```

コードが正しく動作したことを確認した後、CLI を使用して VM とそのリソースを削除します。

```azurecli-interactive
az group delete --name sampleVmResourceGroup
```

## <a name="deploy-a-web-app-from-a-github-repo"></a>GitHub リポジトリからの Web アプリのデプロイ

`AzureApp.java` の main メソッドを次のものに置き換えます。 コードを実行する前に、`appName` 変数を一意の値に更新します。 このコードは、無料の価格レベルで稼働する新しい [Azure App Service Web App](/azure/app-service-web/app-service-web-overview) に、パブリック GitHub リポジトリの `master` ブランチから Web アプリケーションをデプロイするものです。

```java
    public static void main(String[] args) {
        try {

            final String appName = "YOUR_APP_NAME";

            TokenCredential credential = new EnvironmentCredentialBuilder()
                    .authorityHost(AzureAuthorityHosts.AZURE_PUBLIC_CLOUD)
                    .build();

            // If you don't set the tenant ID and subscription ID via environment variables,
            // change to create the Azure profile with tenantId, subscriptionId, and Azure environment.
            AzureProfile profile = new AzureProfile(AzureEnvironment.AZURE);
            
            AzureResourceManager azureResourceManager = AzureResourceManager.configure()
                    .withLogLevel(HttpLogDetailLevel.BASIC)
                    .authenticate(credential, profile)
                    .withDefaultSubscription();

            WebApp app = azureResourceManager.webApps().define(appName)
                    .withRegion(Region.US_WEST2)
                    .withNewResourceGroup("sampleWebResourceGroup")
                    .withNewWindowsPlan(PricingTier.FREE_F1)
                    .defineSourceControl()
                    .withPublicGitRepository(
                            "https://github.com/Azure-Samples/app-service-web-java-get-started")
                    .withBranch("master")
                    .attach()
                    .create();

        } catch (Exception e) {
            System.out.println(e.getMessage());
            e.printStackTrace();
        }
    }
```

先ほどと同様、このコードも Maven を使用して実行します。

```shell
mvn clean compile exec:java
```

次の CLI を使用して、ブラウザーを開いてアプリケーションにアクセスします。

```azurecli-interactive
az appservice web browse --resource-group sampleWebResourceGroup --name YOUR_APP_NAME
```

デプロイを検証した後、Web アプリとプランをサブスクリプションから削除します。

```azurecli-interactive
az group delete --name sampleWebResourceGroup
```

## <a name="connect-to-an-azure-sql-database"></a>Azure SQL データベースに接続する

`AzureApp.java` にある現行の main メソッドを次のコードに置き換えます。 変数を実際の値に設定します。
このコードでは、リモート アクセスを許可するファイアウォール規則を使用して、新しい SQL データベースを作成します。 次に、SQL Database JBDC ドライバーを使用してそれに接続します。

```java
    public static void main(String args[])
    {
        // Create the db using the management libraries.
        try {
            TokenCredential credential = new EnvironmentCredentialBuilder()
                    .authorityHost(AzureAuthorityHosts.AZURE_PUBLIC_CLOUD)
                    .build();

            // If you don't set the tenant ID and subscription ID via environment variables,
            // change to create the Azure profile with tenantId, subscriptionId, and Azure environment.
            AzureProfile profile = new AzureProfile(AzureEnvironment.AZURE);

            AzureResourceManager azureResourceManager = AzureResourceManager.configure()
                    .withLogLevel(HttpLogDetailLevel.BASIC)
                    .authenticate(credential, profile);

            final String adminUser = "YOUR_USERNAME_HERE";
            final String sqlServerName = "YOUR_SERVER_NAME_HERE";
            final String sqlDbName = "YOUR_DB_NAME_HERE";
            final String dbPassword = "YOUR_PASSWORD_HERE";
            final String firewallRuleName = "YOUR_RULE_NAME_HERE";

            SqlServer sampleSQLServer = azureResourceManager.sqlServers().define(sqlServerName)
                    .withRegion(Region.US_EAST)
                    .withNewResourceGroup("sampleSqlResourceGroup")
                    .withAdministratorLogin(adminUser)
                    .withAdministratorPassword(dbPassword)
                    .defineFirewallRule(firewallRuleName)
                        .withIpAddressRange("0.0.0.0","255.255.255.255")
                        .attach()
                    .create();

            SqlDatabase sampleSQLDb = sampleSQLServer.databases().define(sqlDbName).create();

            // Assemble the connection string to the database.
            final String domain = sampleSQLServer.fullyQualifiedDomainName();
            String url = "jdbc:sqlserver://"+ domain + ":1433;" +
                    "database=" + sqlDbName +";" +
                    "user=" + adminUser+ "@" + sqlServerName + ";" +
                    "password=" + dbPassword + ";" +
                    "encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.database.windows.net;loginTimeout=30;";

            // Connect to the database, create a table, and insert an entry into it.
            Connection conn = DriverManager.getConnection(url);

            String createTable = "CREATE TABLE CLOUD ( name varchar(255), code int);";
            String insertValues = "INSERT INTO CLOUD (name, code ) VALUES ('Azure', 1);";
            String selectValues = "SELECT * FROM CLOUD";
            Statement createStatement = conn.createStatement();
            createStatement.execute(createTable);
            Statement insertStatement = conn.createStatement();
            insertStatement.execute(insertValues);
            Statement selectStatement = conn.createStatement();
            ResultSet rst = selectStatement.executeQuery(selectValues);

            while (rst.next()) {
                System.out.println(rst.getString(1) + " "
                        + rst.getString(2));
            }


        } catch (Exception e) {
            System.out.println(e.getMessage());
            System.out.println(e.getStackTrace().toString());
        }
    }
```

コマンド ラインでサンプルを実行します。

```shell
mvn clean compile exec:java
```

その後、CLI を使用して、リソースをクリーンアップします。

```azurecli-interactive
az group delete --name sampleSqlResourceGroup
```

## <a name="write-a-blob-into-a-new-storage-account"></a>新しいストレージ アカウントへの BLOB の書き込み

`AzureApp.java` にある現行の main メソッドを次のコードに置き換えます。 このコードでは、[Azure ストレージ アカウント](/azure/storage/common/storage-introduction)が作成されます。 次に、Azure Storage Libraries for Java を使用して、新しいテキスト ファイルがクラウドに作成されます。

```java
    public static void main(String[] args) {

        try {
            TokenCredential tokenCredential = new EnvironmentCredentialBuilder()
                    .authorityHost(AzureAuthorityHosts.AZURE_PUBLIC_CLOUD)
                    .build();

            // If you don't set the tenant ID and subscription ID via environment variables,
            // change to create the Azure profile with tenantId, subscriptionId, and Azure environment.
            AzureProfile profile = new AzureProfile(AzureEnvironment.AZURE);

            AzureResourceManager azureResourceManager = AzureResourceManager.configure()
                    .withLogLevel(HttpLogDetailLevel.BASIC)
                    .authenticate(tokenCredential, profile)
                    .withDefaultSubscription();

            // Create a new storage account.
            String storageAccountName = "YOUR_STORAGE_ACCOUNT_NAME_HERE";
            StorageAccount storage = azureResourceManager.storageAccounts().define(storageAccountName)
                    .withRegion(Region.US_WEST2)
                    .withNewResourceGroup("sampleStorageResourceGroup")
                    .create();

            // Create a storage container to hold the file.
            List<StorageAccountKey> keys = storage.getKeys();
            PublicEndpoints endpoints = storage.endPoints();
            String accountName = storage.name();
            String accountKey = keys.get(0).value();
            String endpoint = endpoints.primary().blob();

            StorageSharedKeyCredential credential = new StorageSharedKeyCredential(accountName, accountKey);

            BlobServiceClient storageClient =new BlobServiceClientBuilder()
                    .endpoint(endpoint)
                    .credential(credential)
                    .buildClient();

            // Container name must be lowercase.
            BlobContainerClient blobContainerClient = storageClient.getBlobContainerClient("helloazure");
            blobContainerClient.create();

            // Make the container public.
            blobContainerClient.setAccessPolicy(PublicAccessType.CONTAINER, null);

            // Write a blob to the container.
            String fileName = "helloazure.txt";
            String textNew = "Hello Azure";

            BlobClient blobClient = blobContainerClient.getBlobClient(fileName);
            InputStream is = new ByteArrayInputStream(textNew.getBytes());
            blobClient.upload(is, textNew.length());

        } catch (Exception e) {
            System.out.println(e.getMessage());
            e.printStackTrace();
        }
    }
```

コマンド ラインでサンプルを実行します。

```shell
mvn clean compile exec:java
```

ストレージ アカウント内の *helloazure.txt* ファイルは、Azure portal から、または [Azure Storage Explorer](/azure/vs-azure-tools-storage-explorer-blobs) で参照することができます。

CLI を使用して、ストレージ アカウントをクリーンアップします。

```azurecli-interactive
az group delete --name sampleStorageResourceGroup
```

## <a name="explore-more-samples"></a>その他のサンプルを探す

Azure Management Libraries for Java を使ってリソースを管理したりタスクを自動化したりする方法をさらに詳しく知るには、[仮想マシン](virtual-machine-samples.md)、[Web アプリ](web-apps-samples.md)、[SQL データベース](sql-database-samples.md)に関するサンプル コードを参照してください。

## <a name="reference-and-release-notes"></a>リファレンスとリリース ノート

すべてのパッケージには、[リファレンス](/java/api)が提供されています。

## <a name="get-help-and-give-feedback"></a>質問とフィードバック

ご質問は、[Stack Overflow](https://stackoverflow.com/questions/tagged/azure+java) のコミュニティに投稿してください。 Azure SDK for Java に関する未解決の問題やバグは、[GitHub リポジトリ](https://github.com/Azure/azure-sdk-for-java)にご報告ください。
