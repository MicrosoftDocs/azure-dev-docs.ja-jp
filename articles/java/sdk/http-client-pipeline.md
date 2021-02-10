---
title: Azure SDK for Java の HTTP クライアントとパイプライン
description: Azure SDK for Java の HTTP クライアントとパイプライン機能の概要
author: srnagar
ms.date: 02/02/2021
ms.topic: conceptual
ms.custom: devx-track-java
ms.author: srnagar
ms.openlocfilehash: 93595096bc6d9fde1226f6875b866ea28b0a85bb
ms.sourcegitcommit: 71847ee0a1fee3f3320503629d9a8c82319a1f6a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/03/2021
ms.locfileid: "99528536"
---
# <a name="http-clients-and-pipelines-in-the-azure-sdk-for-java"></a>Azure SDK for Java の HTTP クライアントとパイプライン

この記事では、Azure SDK for Java 内での HTTP クライアントとパイプライン機能の使用の概要について説明します。 この機能は、すべての Azure SDK for Java ライブラリを使用する開発者に、一貫性のある強力で柔軟なエクスペリエンスを提供します。

## <a name="http-clients"></a>HTTP クライアント

Azure SDK for Java は、`HttpClient` の抽象化を使用して実装されます。 この抽象化により、複数の HTTP クライアント ライブラリまたはカスタム実装を受け入れるプラグ可能なアーキテクチャが可能になります。 ただし、ほとんどのユーザーの依存関係の管理を簡素化するために、すべての Azure クライアント ライブラリは `azure-core-http-netty` に依存しています。 そのため、すべての Azure SDK for Java ライブラリで使用される既定のクライアントは [Netty](https://netty.io) HTTP クライアントです。

Netty が既定の HTTP クライアントですが、SDK には、お使いのプロジェクトに既に存在する依存関係に応じて 3 つのクライアント実装が用意されています。 これらの実装は、次を対象とします。

* [Netty](https://netty.io)
* [OkHttp](https://square.github.io/okhttp/)
* JDK 11 で導入された新しい [HttpClient](https://openjdk.java.net/groups/net/httpclient/intro.html)

### <a name="replace-the-default-http-client"></a>既定の HTTP クライアントを置き換える

別の実装を使用する場合は、Netty の依存関係をビルド構成ファイルで除外することでこれを削除できます。 Maven の *pom.xml* ファイルで、Netty 依存関係を除外し、別の依存関係を含めます。

次の例では、`azure-security-keyvault-secrets` ライブラリの実際の依存関係から Netty 依存関係を除外する方法を示します。 ここに示すように、適切なすべての `com.azure` ライブラリから Netty を除外するようにしてください。

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-security-keyvault-secrets</artifactId>
    <version>4.2.2.</version>
    <exclusions>
      <exclusion>
        <groupId>com.azure</groupId>
        <artifactId>azure-core-http-netty</artifactId>
      </exclusion>
    </exclusions>
</dependency>

<dependency>
  <groupId>com.azure</groupId>
  <artifactId>azure-core-http-okhttp</artifactId>
  <version>1.3.3</version>
</dependency>
```

> [!NOTE]
> Netty 依存関係を削除しても、その代わりの実装を指定しない場合、アプリケーションは起動に失敗します。 `HttpClient` の実装がクラスパスに存在している必要があります。

### <a name="configure-http-clients"></a>HTTP クライアントを構成する

サービス クライアントを構築すると、既定で `HttpClient.createDefault()` が使用されます。 このメソッドは、指定された HTTP クライアントの実装に基づいて、基本的な `HttpClient` インスタンスを返します。 プロキシなどのより複雑な HTTP クライアントが必要な場合に備えて、各実装には、構成済みの `HttpClient` インスタンスを構築できるビルダーが用意されています。 ビルダーは `NettyAsyncHttpClientBuilder`、`OkHttpAsyncHttpClientBuilder`、`JdkAsyncHttpClientBuilder` です。

次の例は、Netty、OkHttp、JDK 11 HTTP クライアントを使用して `HttpClient` インスタンスを構築する方法を示しています。 これらのインスタンスは、`http://localhost:3128` を使用してプロキシを行い、ユーザー *example* とパスワード *weakPassword* を使用して認証を行います。

```java
// Netty
HttpClient httpClient = new NettyAsyncHttpClientBuilder()
    .proxy(new ProxyOptions(ProxyOptions.Type.HTTP, new InetSocketAddress("localhost", 3128))
        .setCredentials("example", "weakPassword"))
    .build();

// OkHttp
HttpClient httpClient = new OkHttpAsyncHttpClientBuilder()
    .proxy(new ProxyOptions(ProxyOptions.Type.HTTP, new InetSocketAddress("localhost", 3128))
        .setCredentials("example", "weakPassword"))
    .build();

// JDK 11 HttpClient
HttpClient client = new JdkAsyncHttpClientBuilder()
    .proxy(new ProxyOptions(ProxyOptions.Type.HTTP, new InetSocketAddress("localhost", 3128))
        .setCredentials("example", "weakPassword"))
    .build();
```

これで、構築された `HttpClient` インスタンスをサービス クライアント ビルダーに渡して、サービスと通信するためのクライアントとして使用できるようになりました。 次の例では、新しい `HttpClient` インスタンスを使用して、Azure Storage Blob クライアントを構築します。

```java
BlobClient blobClient = new BlobClientBuilder()
    .connectionString(<connection string>)
    .containerName("container")
    .blobName("blob")
    .httpClient(httpClient)
    .build();
```

管理ライブラリについては、マネージャーの構成時に `HttpClient` を設定できます。

```java
AzureResourceManager azureResourceManager = AzureResourceManager.configure()
    .withHttpClient(httpClient)
    .authenticate(credential, profile)
    .withDefaultSubscription();
```

## <a name="http-pipeline"></a>HTTP パイプライン

HTTP パイプラインは、Azure の Java クライアント ライブラリでの一貫性と診断可能性を確保する上で重要なコンポーネントの 1 つです。 HTTP パイプラインは次のもので構成されます。

* HTTP トランスポート
* HTTP パイプライン ポリシー

クライアントを作成する際に、独自のカスタム HTTP パイプラインを指定できます。 パイプラインを指定しない場合、クライアント ライブラリによって、その特定のクライアント ライブラリで動作するように構成されたものが作成されます。

### <a name="http-transport"></a>HTTP トランスポート

HTTP トランスポートは、サーバーへの接続の確立と、HTTP メッセージの送受信を行います。 HTTP トランスポートは、Azure サービスと対話するための Azure SDK クライアント ライブラリのゲートウェイを形成します。 この記事で既に説明したように、Azure SDK for Java は、HTTP トランスポートに対して既定で [Netty](https://netty.io/) を使用します。 ただし、この SDK にはプラグ可能な HTTP トランスポートも用意されているので、必要に応じて他の実装を使用できます。 この SDK には、OkHttp 用と、JDK 11 以降に付属する HTTP クライアント用に、さらに 2 つの HTTP トランスポート実装が用意されています。

### <a name="http-pipeline-policies"></a>HTTP パイプライン ポリシー

パイプラインは、HTTP 要求と応答のラウンドトリップごとに実行される一連の手順で構成されます。 各ポリシーには特定の目的があり、要求または応答、あるいはその両方に対して動作します。 すべてのクライアント ライブラリには標準の "Azure Core" レイヤーがあり、このレイヤーによって、各ポリシーがパイプライン内で順番に実行されるようになります。 要求を送信すると、ポリシーはパイプラインに追加された順序で実行されます。 サービスから応答を受信すると、ポリシーは逆の順序で実行されます。 パイプラインに追加されたすべてのポリシーは、要求を送信する前と応答を受信した後に実行されます。 ポリシーは、要求と応答のどちらか、またはその両方に対して動作するかどうかを決定する必要があります。 たとえば、ログ ポリシーは要求と応答をログに記録しますが、認証ポリシーは要求の変更のみを対象としています。

Azure Core フレームワークは、ポリシーを実行するために必要なコンテキストと共に、必要な要求および応答データをポリシーに提供します。 その後ポリシーは、提供されたデータを使用して操作を実行し、パイプラインの次のポリシーに制御を渡すことができます。

![HTTP パイプラインの図](./media/http-pipeline.svg)

### <a name="http-pipeline-policy-position"></a>HTTP パイプライン ポリシーの位置

クラウド サービスに HTTP 要求を行う場合は、一時的なエラーを処理し、失敗した試行を再試行することが重要です。 この機能は一般的な要件であるため、Azure Core は、一時的なエラーを監視し、自動的に要求を再試行できる再試行ポリシーを提供しています。

したがって、この再試行ポリシーによって、パイプライン全体が、再試行ポリシーの前に実行されるポリシーと、再試行ポリシーの後に実行されるポリシーの 2 つの部分に分割されます。 再試行ポリシーの前に追加されたポリシーは、API 操作ごとに 1 回だけ実行され、再試行ポリシーの後に追加されたポリシーは、再試行の回数だけ実行されます。

そのため、HTTP パイプラインを構築するときは、ポリシーを要求の再試行ごとに実行するのか、API 操作ごとに 1 回実行するのかを理解しておく必要があります。

### <a name="common-http-pipeline-policies"></a>一般的な HTTP パイプライン ポリシー

REST ベース サービスの HTTP パイプラインには、認証、再試行、ログ、テレメトリ、およびヘッダー内の要求 ID の指定に関するポリシーが構成されています。 Azure Core には、パイプラインに追加できる一般的に必要なこれらの HTTP ポリシーが事前に読み込まれています。

| ポリシー                | GitHub のリンク        |
|-----------------------|--------------------|
| 再試行ポリシー          | [RetryPolicy.java](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/core/azure-core/src/main/java/com/azure/core/http/policy/RetryPolicy.java) |
| Authentication Policy | [BearerTokenAuthenticationPolicy.java](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/core/azure-core/src/main/java/com/azure/core/http/policy/BearerTokenAuthenticationPolicy.java) |
| ログ ポリシー        | [HttpLoggingPolicy.java](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/core/azure-core/src/main/java/com/azure/core/http/policy/HttpLoggingPolicy.java) |
| 要求 ID ポリシー     | [RequestIdPolicy.java](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/core/azure-core/src/main/java/com/azure/core/http/policy/RequestIdPolicy.java) |
| テレメトリ ポリシー      | [UserAgentPolicy.java](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/core/azure-core/src/main/java/com/azure/core/http/policy/UserAgentPolicy.java) |

### <a name="custom-http-pipeline-policy"></a>カスタムの HTTP パイプライン ポリシー

HTTP パイプライン ポリシーは、要求と応答を変更または修飾するための便利なメカニズムを提供します。 ユーザーまたはクライアント ライブラリ開発者によって作成されたカスタム ポリシーをパイプラインに追加することができます。 ポリシーをパイプラインに追加する際に、このポリシーを呼び出しごとに実行するのか、または再試行ごとに実行するのかを指定できます。

カスタムの HTTP パイプライン ポリシーを作成するには、基本ポリシー型を拡張して抽象メソッドを実装するだけで済みます。 その後、ポリシーをパイプラインにプラグインできます。

## <a name="next-steps"></a>次のステップ

Azure SDK for Java の HTTP クライアントの機能について理解したので、「[Azure SDK for Java でプロキシを構成する](proxying.md)」を参照して、使用する HTTP クライアントをさらにカスタマイズする方法について確認します。
