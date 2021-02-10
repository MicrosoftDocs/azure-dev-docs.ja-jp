---
title: Azure SDK for Java を使用する
description: Azure リソースのプロビジョニング、使用、管理の際の生産性を向上させる、Azure SDK for Java の機能の概要。
author: jonathangiles
ms.date: 02/02/2021
ms.topic: conceptual
ms.custom: devx-track-java
ms.author: jogiles
ms.openlocfilehash: 64a13c99f58b109a066acd322aa90f39a74ea0e9
ms.sourcegitcommit: 71847ee0a1fee3f3320503629d9a8c82319a1f6a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/03/2021
ms.locfileid: "99528483"
---
# <a name="use-the-azure-sdk-for-java"></a>Azure SDK for Java を使用する

オープンソースである Azure SDK for Java により、Java アプリケーション コードからの Azure リソースのプロビジョニング、管理、使用が容易になります。

## <a name="important-details"></a>重要な詳細

* Azure ライブラリは、ローカルまたはクラウドで実行する Java コード からの Azure サービスとの通信手段です。
* ライブラリは、Java 8 以降をサポートしており、Java 8 ベースラインと最新の Java "長期サポート" リリースの両方に対してテストされています。
* ライブラリには Java モジュールの完全なサポートが含まれています。つまり、これらは Java モジュールの要件に完全に準拠し、使用するすべての関連パッケージをエクスポートします。
* Azure SDK for Java は、特定の Azure サービスに関連した多数の個別の Java ライブラリのみで構成されています。 "SDK" には他のツールはありません。
* "管理" と "クライアント" という別個のライブラリが存在します ("管理プレーン" ライブラリおよび "データ プレーン" ライブラリと呼ばれることもあります)。 各セットはさまざまな目的を果たし、さまざまな種類のコードで使用されます。 詳細については、この記事の後の方にある次のセクションを参照してください。
  * [クライアント ライブラリを使用して Azure リソースに接続し、そのリソースを使用する。](#connect-to-and-use-azure-resources-with-client-libraries)
  * [管理ライブラリを使用して Azure リソースをプロビジョニングし、管理する。](#provision-and-manage-azure-resources-with-management-libraries)
* ライブラリのドキュメントは、[Azure for Java のリファレンス](/java/api/overview/azure/) (Azure サービスごとに編成) または 「[Java API ブラウザー](/java/api/)」(パッケージ名ごとに編成) を参照してください。

## <a name="other-details"></a>その他の詳細

* Azure SDK for Java ライブラリは、基になる Azure REST API の上に構築されているため、使い慣れた Java のパラダイムからそれらの API を使用できます。 ただし、REST API は、必要に応じていつでも Java コードから直接使用できます。
* Azure ライブラリのソース コードは、[GitHub リポジトリ](https://github.com/Azure/azure-sdk-for-java)にあります。 オープンソース プロジェクトとして、皆さんの参加を歓迎します。
* 現在、認証プロトコル、ログ、トレース、トランスポート プロトコル、バッファーされた応答、再試行などの一般的なクラウド パターンを共有するために、Azure SDK for Java のライブラリを更新しています。
  * この共有機能は、[azure-core](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/core/azure-core) ライブラリに含まれています。
* ライブラリに適用されるガイドラインの詳細については、「[Java Azure SDK 設計ガイドライン](https://azure.github.io/azure-sdk/java_introduction.html)」を参照してください。

## <a name="connect-to-and-use-azure-resources-with-client-libraries"></a>クライアント ライブラリを使用して Azure リソースに接続し、そのリソースを使用する

クライアント (つまり、"データ プレーン") ライブラリを使用すると、既にプロビジョニングされたサービスを操作する Java アプリケーション コードの記述に役立ちます。 クライアント ライブラリは、クライアント API をサポートするサービスに対してのみ存在します。 これらは、その Maven グループ ID が `com.azure` であるため識別できます。

すべての Azure Java クライアント ライブラリは、クライアントのインスタンスの作成を担当する Java builder クラスを提供する場合と同じ API 設計パターンに従います。 このパターンでは、クライアントの定義およびインスタンス化をその操作から分離することで、クライアントが不変であることを可能にし、その結果、使用しやすくなります。 さらに、すべてのクライアント ライブラリは、次のいくつかの重要なパターンに従います。

* 同期と非同期の両方の API をサポートするクライアント ライブラリでは、これらの API を別個のクラスで提供する必要があります。 つまり、たとえば、同期 API の `KeyVaultClient` と非同期 API の `KeyVaultAsyncClient` が存在する場合などです。

* 同期と非同期の両方の API を構築する役割を担う 1 つのビルダー クラスがあります。 ビルダーには、同期クライアント クラスに似た、`Builder` を含む名前が付けられます。 たとえば、「 `KeyVaultClientBuilder` 」のように入力します。 このビルダーには、必要に応じてクライアント インスタンスを作成するための `buildClient()` および `buildAsyncClient()` メソッドが用意されています。

これらの規則により、`Client` で終わるすべてのクラスは不変であり、Azure サービスとやりとりするための操作を提供します。 `ClientBuilder` で終わるすべてのクラスは、特定のクライアントの種類のインスタンスを構成および作成するための操作を提供します。

### <a name="client-libraries-example"></a>クライアント ライブラリの例

次のコード例は、同期キー コンテナー `KeyClient` を作成する方法を示しています。

```java
KeyClient client = new KeyClientBuilder()
        .endpoint(<your Key Vault URL>)
        .credential(new DefaultAzureCredentialBuilder().build())
        .buildClient();
```

次のコード例は、非同期キー コンテナー `KeyAsyncClient` を作成する方法を示しています。

```java
KeyAsyncClient client = new KeyClientBuilder()
        .endpoint(<your Key Vault URL>)
        .credential(new DefaultAzureCredentialBuilder().build())
        .buildAsyncClient();
```

各クライアント ライブラリの操作の詳細については、[SDK の GitHub リポジトリ](https://github.com/Azure/azure-sdk-for-java)内のライブラリのプロジェクト ディレクトリにある *README.md* ファイルを参照してください。 また、その他のコード スニペットについては、[リファレンス ドキュメント](/java/api)および [Azure のサンプル](/samples/browse/?products=azure&languages=java)で確認できます。

## <a name="provision-and-manage-azure-resources-with-management-libraries"></a>管理ライブラリを使用して Azure リソースをプロビジョニングし、管理する

管理 (つまり、"管理プレーン") ライブラリは、Java アプリケーション コードから Azure リソースの作成、プロビジョニング、その他の管理を行う場合に役立ちます。 これらのライブラリは `com.azure.resourcemanager` Maven グループ ID 内で見つけることができます。 すべての Azure サービスには、対応する管理ライブラリがあります。

管理ライブラリを使用すると、[Azure portal](https://portal.azure.com/) または [Azure CLI](/cli/azure/install-azure-cli) で行うのと同じタスクを実行する構成またはデプロイのスクリプトを作成することができます。

すべての Azure Java 管理ライブラリには、サービス API として `*Manager` クラスが用意されています。たとえば、Azure コンピューティング サービス向けの `ComputeManager` や、よく使用されるサービスの集合向けの `AzureResourceManager` があります。

### <a name="management-libraries-example"></a>管理ライブラリの例

次のコード例は、`ComputeManager` を作成する方法を示しています。

```java
ComputeManager computeManager = ComputeManager
    .authenticate(
        new DefaultAzureCredentialBuilder().build(),
        new AzureProfile(AzureEnvironment.AZURE));
```

次のコード例は、新しい仮想マシンをプロビジョニングする方法を示しています。

```java
VirtualMachine virtualMachine = computeManager.virtualMachines()
    .define(<your virtual machine>)
    .withRegion(Region.US_WEST)
    .withExistingResourceGroup(<your resource group>)
    .withNewPrimaryNetwork("10.0.0.0/28")
    .withPrimaryPrivateIPAddressDynamic()
    .withoutPrimaryPublicIPAddress()
    .withPopularLinuxImage(KnownLinuxVirtualMachineImage.UBUNTU_SERVER_18_04_LTS)
    .withRootUsername(<virtual-machine username>)
    .withSsh(<virtual-machine SSH key>)
    .create();
```

次のコード例は、既存の仮想マシンを取得する方法を示しています。

```java
VirtualMachine virtualMachine = computeManager.virtualMachines()
    .getByResourceGroup(<your resource group>, <your virtual machine>);
```

次のコード例は、仮想マシンを更新し、新しいデータ ディスクを追加する方法を示しています。

```java
virtualMachine.update()
    .withNewDataDisk(10)
    .apply();
```

各管理ライブラリの操作の詳細については、[SDK の GitHub リポジトリ](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/resourcemanager#readme)内のライブラリのプロジェクト ディレクトリにある *README.md* ファイルを参照してください。 また、その他のコード スニペットについては、[リファレンス ドキュメント](/java/api)および [Azure のサンプル](/samples/browse/?products=azure&languages=java)で確認できます。

## <a name="get-help-and-connect-with-the-sdk-team"></a>SDK チームによるサポートと連絡先

* [Azure SDK for Java のドキュメント](https://azure.github.io/azure-sdk-for-java/)を参照します。
* ご質問は、[Stack Overflow](https://stackoverflow.com/questions/tagged/azure-sdk-for-java) のコミュニティに投稿してください。
* [GitHub リポジトリ](https://github.com/Azure/azure-sdk-for-java/issues)で SDK に対して issue をオープンします。
* Twitter で [@AzureSDK](https://twitter.com/AzureSdk/) について言及します。

## <a name="next-steps"></a>次のステップ

Azure SDK for Java の概要を理解したので、ライブラリを使用するときに生産性を向上させる多くの横断的な概念について詳しく調べることができます。 次の記事は、適切な出発点になります。

* [HTTP クライアントとパイプライン](http-client-pipeline.md)
* [非同期プログラミング](async-programming.md)
* [改ページと反復](pagination.md)
* [長時間にわたって実行される操作](lro.md)
* [プロキシを構成する](proxying.md)
* [トレースを構成する](tracing.md)
