---
title: Azure SDK for Java でプロキシを構成する
description: プロキシに関連した Azure SDK for Java の概念の概要
author: alzimmermsft
ms.date: 02/02/2021
ms.topic: conceptual
ms.custom: devx-track-java
ms.author: alzimmer
ms.openlocfilehash: ce4305e1f76a15ab799523f67002315b0883c3e1
ms.sourcegitcommit: 71847ee0a1fee3f3320503629d9a8c82319a1f6a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/03/2021
ms.locfileid: "99528478"
---
# <a name="configure-proxies-in-the-azure-sdk-for-java"></a>Azure SDK for Java でプロキシを構成する

この記事では、プロキシを適切に使用するように Azure SDK for Java を構成する方法の概要について説明します。

## <a name="http-proxy-configuration"></a>HTTP プロキシの構成

Java 用 Azure クライアント ライブラリには、`HttpClient` 用にプロキシを構成するための複数の方法が用意されています。

各プロキシ提供方法には、それぞれの長所と短所があり、さまざまなレベルのカプセル化が用意されています。 `HttpClient` 用にプロキシを構成した場合は、残りの有効期間中ずっとそのプロキシが使用されます。 プロキシが個々の `HttpClient` に関連付けられている場合、アプリケーションは複数の `HttpClient` インスタンスを使用でき、そのそれぞれが異なるプロキシを使用してアプリケーションのプロキシ要件を満たすことができます。

プロキシ構成オプションは次のとおりです。

* [環境プロキシを使用する](#use-an-environment-proxy)
* [構成プロキシを使用する](#use-a-configuration-proxy)
* [明示的なプロキシを使用する](#use-an-explicit-proxy)

### <a name="use-an-environment-proxy"></a>環境プロキシを使用する

既定で、HTTP クライアント ビルダーによって環境のプロキシ構成が検査されます。 このプロセスでは、Azure SDK for Java の `Configuration` API が使用されます。 ビルダーによってクライアントが作成されると、`Configuration.getGlobalConfiguration()` を呼び出すことによって取得された "グローバル構成" のコピーを使用して構成されます。 この呼び出しで、システム環境から HTTP プロキシ構成が読み取られます。

ビルダーによって環境が検査されるときに、指定された順序で次の環境構成が検索されます。

1. `HTTPS_PROXY`
2. `HTTP_PROXY`
3. `https.proxy*`
4. `http.proxy*`

`*` は、既知の Java プロキシのプロパティを表します。 詳細については、Oracle のドキュメント内の「[Java ネットワークとプロキシ](https://docs.oracle.com/javase/8/docs/technotes/guides/net/proxies.html)」を参照してください。

いずれかの環境構成が検出されると、`ProxyOptions.fromConfiguration(Configuration.getGlobalConfiguration())` が呼び出され、`ProxyOptions` インスタンスが作成されます。 この記事の下の部分で `ProxyOptions` の種類についてより詳しく説明します。

> [!Important]
> どのプロキシを使用する場合も、システム環境プロパティ `java.net.useSystemProxies` を `true` に設定する必要があります。

また、システム環境変数内にあるプロキシ構成を使用しない HTTP クライアント インスタンスを作成することもできます。 既定の動作をオーバーライドするには、HTTP クライアント ビルダーで、異なる方法で構成された `Configuration` を明示的に設定します。 ビルダーで `Configuration` を設定すると、`Configuration.getGlobalConfiguration()` は呼び出されなくなります。 たとえば、`Configuration.NONE` を使用して `configuration(Configuration)` を呼び出すと、ビルダーによって環境の構成が検査されるのを明示的に禁止できます。

次の例では、`HTTP_PROXY` 環境変数を値 `localhost:8888` と共に使用して、Fiddler をプロキシとして使用します。 このコードは、Netty と OkHttp HTTP クライアントの作成方法を示しています。 (HTTP クライアント構成の詳細については、[HTTP クライアントとパイプライン](http-client-pipeline.md)に関する記事を参照してください)。

```bash
export HTTP_PROXY=localhost:8888
```

```java
HttpClient nettyHttpClient = new NettyAsyncHttpClientBuilder().build();
HttpClient okhttpHttpClient = new OkHttpAsyncHttpClientBuilder().build();
```

環境プロキシが使用されないようにするには、次の例に示すように、`Configuration.NONE` を使用して HTTP クライアント ビルダーを構成します。

```java
HttpClient nettyHttpClient = new NettyAsyncHttpClientBuilder()
    .configuration(Configuration.NONE)
    .build();

HttpClient okhttpHttpClient = new OkHttpAsyncHttpClientBuilder()
    .configuration(Configuration.NONE)
    .build();
```

### <a name="use-a-configuration-proxy"></a>構成プロキシを使用する

環境から読み取るのではなく、環境から既に受け取っている同じプロキシ設定でカスタム `Configuration` を使用するように HTTP クライアント ビルダーを構成できます。 この場合、制限されたユース ケースを対象とする再利用可能な構成を持つことができます。 HTTP クライアント ビルダーは、`HttpClient` の構築時に、`ProxyOptions.fromConfiguration(<Configuration passed into the builder>)` から返された `ProxyOptions` を使用します。

次の例では、`Configuration` オブジェクトに設定された `http.proxy*` 構成を使用して、Fiddler をプロキシとして認証するプロキシを使用します。

```java
Configuration configuration = new Configuration()
    .put("java.net.useSystemProxies", "true")
    .put("http.proxyHost", "localhost")
    .put("http.proxyPort", "8888")
    .put("http.proxyUser", "1")
    .put("http.proxyPassword", "1");

HttpClient nettyHttpClient = new NettyAsyncHttpClientBuilder()
    .configuration(configuration)
    .build();

HttpClient okhttpHttpClient = new OkHttpAsyncHttpClientBuilder()
    .configuration(configuration)
    .build();
```

### <a name="use-an-explicit-proxy"></a>明示的なプロキシを使用する

Java クライアント ライブラリには、プロキシを構成するための Azure クライアント ライブラリの型として機能する `ProxyOptions` クラスが付属しています。 プロキシ要求を送信するために使用されるネットワーク プロトコル、プロキシ アドレス、プロキシ認証資格情報、および非プロキシ ホストを指定して `ProxyOptions` を構成できます。 プロキシ ネットワーク プロトコルとプロキシ アドレスのみが必須です。 認証資格情報を使用する場合は、ユーザー名とパスワードの両方を設定する必要があります。

次の例では、要求を既定の Fiddler アドレス (`localhost:8888`) にプロキシする単純な `ProxyOptions` インスタンスを作成します。

```java
ProxyOptions proxyOptions = new ProxyOptions(ProxyOptions.HTTP, new InetSocketAddress("localhost", 8888));
```

次の例では、プロキシ認証を必要とする Fiddler インスタンスに要求をプロキシする認証済み `ProxyOptions` を作成します。

```java
// Fiddler uses username "1" and password "1" with basic authentication as its proxy authentication requirement.
ProxyOptions proxyOptions = new ProxyOptions(ProxyOptions.HTTP, new InetSocketAddess("localhost", 8888))
    .setCredentials("1", "1");
```

使用する明示的なプロキシを示すために、`ProxyOptions` を使用して HTTP クライアント ビルダーを直接構成できます。 この構成は、プロキシを指定するための最もきめ細かな方法であり、一般に、プロキシ要件を更新するために変更できる `Configuration` を渡すのと比べて柔軟ではありません。

次の例では、Fiddler をプロキシとして使用するために `ProxyOptions` を使用しています。

```java
ProxyOptions proxyOptions = new ProxyOptions(ProxyOptions.HTTP, new InetSocketAddress("localhost", 8888));

HttpClient nettyHttpClient = new NettyAsyncHttpClientBuilder()
    .proxy(proxyOptions)
    .build();

HttpClient okhttpHttpClient = new OkHttpAsyncHttpClientBuilder()
    .proxy(proxyOptions)
    .build();
```

## <a name="next-steps"></a>次のステップ

Azure SDK for Java でのプロキシ構成の概要を理解したので、アプリケーション内のフローをよりよく理解して、問題の診断に役立てるために、「[Azure SDK for Java でトレースを構成する](tracing.md)」を参照してください。
