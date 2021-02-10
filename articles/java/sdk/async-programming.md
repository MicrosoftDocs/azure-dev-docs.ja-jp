---
title: Azure SDK for Java での非同期プログラミング
description: Azure SDK for Java の非同期プログラミング モデルの概要
author: srnagar
ms.date: 02/02/2021
ms.topic: conceptual
ms.custom: devx-track-java
ms.author: srnagar
ms.openlocfilehash: 64a692b10175a7c08ff8a32e56a638209f239c19
ms.sourcegitcommit: 71847ee0a1fee3f3320503629d9a8c82319a1f6a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/03/2021
ms.locfileid: "99528563"
---
# <a name="asynchronous-programming-in-the-azure-sdk-for-java"></a>Azure SDK for Java での非同期プログラミング

この記事では、Azure SDK for Java の非同期プログラミング モデルについて説明します。

Azure SDK には当初、Azure サービスと対話するための非ブロッキングの非同期 API のみが含まれていました。 これらの API を使用すると、Azure SDK を使って、システム リソースを効率的に用いるスケーラブルなアプリケーションを構築できます。 ただし、[Azure SDK for Java](https://github.com/Azure/azure-sdk-for-java#client-new-releases) には、より多くのユーザーに対応し、非同期プログラミングに慣れていないユーザーに対してクライアント ライブラリを使いやすいものにする同期クライアントも含まれています。 (Azure SDK 設計ガイドラインの「[使いやすさ](https://azure.github.io/azure-sdk/general_introduction.html#approachable)」を参照してください)。このように、Azure SDK for Java のすべての Java クライアント ライブラリでは、非同期と同期の両方のクライアントが提供されています。 ただし、実稼働システムでは、システム リソースを最大限に使用するために、非同期クライアントを使用することをお勧めします。

## <a name="reactive-streams"></a>Reactive Streams

「[Java Azure SDK 設計ガイドライン](https://azure.github.io/azure-sdk/java_introduction.html)」の「[非同期サービス クライアント](https://azure.github.io/azure-sdk/java_introduction.html#async-service-clients)」セクションを参照すると、Java 8 によって提供される `CompletableFuture` を使用する代わりに、非同期 API でリアクティブ型が使用されることがわかります。 JDK でネイティブに使用可能な型ではなく、なぜリアクティブ型を選択したのでしょうか。

Java 8 では、[ストリーム](https://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html)、[ラムダ](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)、[CompletableFuture](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html) などの機能が導入されました。 これらの機能は多くの機能を提供しますが、いくつかの制限があります。

`CompletableFuture` はコールバックベースの非ブロッキング機能を提供し、`CompletionStage` インターフェイスでは一連の非同期操作を簡単に構成することができます。 ラムダによって、これらのプッシュベースの API が読みやすくなります。 ストリームは、データ要素のコレクションを処理する関数型の操作を提供します。 ただし、ストリームは同期的であり、再利用することはできません。 `CompletableFuture` を使用すると、単一の要求を行うことができ、コールバックがサポートされ、"_単一の_" 応答が予期されます。 ただし、多くのクラウド サービスでは、Event Hubs などのデータをストリーミングする機能が必要です。

Reactive Streams は、ソースからサブスクライバーに要素をストリーミングすることによって、これらの制限を克服するのに役立ちます。 サブスクライバーがソースからデータを要求すると、ソースは任意の数の結果を返します。 すべてを一度に送信する必要はありません。 送信するデータがソースにある場合は常に、一定期間にわたって転送が行われます。

このモデルでは、サブスクライバーは、データを受信時に処理するためにイベント ハンドラーを登録します。 これらのプッシュベースの相互作用では、個別のシグナルを使用してサブスクライバーに通知します。

- `onSubscribe()` 呼び出しは、データ転送が開始されようとしていることを示します。
- `onError()` 呼び出しは、エラーが発生したことを示し、データ転送の終了も示します。
- `onComplete()` 呼び出しは、データ転送が正常に完了したことを示します。

Java Streams とは異なり、Reactive Streams ではエラーは最上位イベントとして扱われます。 Reactive Streams には、ソースがエラーをサブスクライバーに通知するための専用のチャネルがあります。 また、Reactive Streams を使用すると、サブスクライバーはデータ転送レートをネゴシエートして、これらのストリームをプッシュプル モデルに変換することができます。

[Reactive Streams](https://github.com/reactive-streams/reactive-streams-jvm#reactive-streams) 仕様は、データ転送の実行方法に関する標準を提供します。 大まかに言えば、この仕様は次の 4 つのインターフェイスを定義し、これらのインターフェイスの実装方法に関する規則を指定します。

- **パブリッシャー** は、データ ストリームのソースです。
- **サブスクライバー** は、データ ストリームのコンシューマーです。
- **サブスクリプション** は、パブリッシャーとサブスクライバーの間のデータ転送の状態を管理します。
- **プロセッサ** は、パブリッシャーとサブスクライバーの両方です。

[RxJava](https://github.com/ReactiveX/RxJava)、[Akka Streams](https://doc.akka.io/docs/akka/current/stream/stream-introduction.html)、[Vert.x](https://vertx.io/docs/#reactive)、[Project Reactor](https://projectreactor.io/docs/core/release/reference/) など、この仕様の実装を提供するいくつかの有名な Java ライブラリがあります。

Azure SDK for Java では、非同期 API を提供するために Project Reactor が採用されています。 この決定の主な要因は、同じく Project Reactor を使用する [Spring Webflux](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html) とのスムーズな統合を実現することでした。 RxJava よりも Project Reactor を選択したもう 1 つの要因として、Project Reactor では Java 8 が使用されていましたが、RxJava はその時点ではまだ Java 7 だったことがあります。 Project Reactor には、データ処理パイプラインを構築するための宣言型コードを記述できる、コンポーザブルな多数の演算子セットも提供されています。 Project Reactor のもう 1 つの利点は、Project Reactor 型を他の一般的な実装型に変換するためのアダプターがあることです。

## <a name="comparing-apis-of-synchronous-and-asynchronous-operations"></a>同期および非同期操作の API の比較

同期クライアントと、非同期クライアントのオプションについて説明しました。 下の表は、これらのオプションを使用して設計された API がどのようなものかをまとめたものです。

| API の種類                                           | 値なし                   | 単一値            | [複数の値]                         |
|----------------------------------------------------|----------------------------|-------------------------|-------------------------------|
| 標準の Java - 同期 API                   | `void`                     | `T`                     | `Iterable<T>`                 |
| 標準の Java - 非同期 API                  | `CompletableFuture<Void>`  | `CompletableFuture<T>`  | `CompletableFuture<List<T>>`   |
| Reactive Streams インターフェイス                        | `Publisher<Void>`          | `Publisher<T>`          | `Publisher<T>`                 |
| Reactive Streams の Project Reactor 実装 | `Mono<Void>`               | `Mono<T>`               | `Flux<T>`                      |

完全性を期すために、Java 9 では 4 つの Reactive Streams インターフェイスを含む [Flow](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/Flow.html) クラスが導入されていることを特筆します。 ただし、このクラスには実装が含まれていません。

## <a name="use-async-apis-in-the-azure-sdk-for-java"></a>Azure SDK for Java で非同期 API を使用する

Reactive Streams 仕様では、パブリッシャーの種類が区別されません。 Reactive Streams 仕様では、パブリッシャーによって単に 0 個以上のデータ要素が生成されます。 多くの場合、最大で 1 つのデータ要素を生成するパブリッシャーと、0 個以上を生成するものとの間には、有用な区別があります。 クラウドベースの API では、この区別は、要求が単一値の応答か、コレクションのどちらを返すかを示します。 Project Reactor には、この区別を行うために [Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html) と [Flux](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html) の 2 種類が用意されています。 `Mono` を返す API には、最大で 1 つの値を持つ応答が含まれ、`Flux` を返す API には 0 個以上の値を持つ応答が含まれます。

たとえば、[ConfigurationAsyncClient](/java/api/com.azure.data.appconfiguration.configurationasyncclient) を使用して、Azure App Configuration サービスを使用して格納されている構成を取得するとします。 (詳細については、「[Azure App Configuration とは](/azure/azure-app-configuration/overview)」を参照してください。)

`ConfigurationAsyncClient` を作成し、クライアントで `getConfigurationSetting()` を呼び出すと、`Mono` が返されます。これは、応答に単一値が含まれていることを示します。 ただし、このメソッドを呼び出すだけでは何も実行されません。 このクライアントは Azure App Configuration サービスに要求をまだ行っていません。 この段階では、この API によって返される `Mono<ConfigurationSetting>` は、データ処理パイプラインの "アセンブリ" にすぎません。 これは、データを使用するために必要なセットアップが完了したことを意味します。 実際にデータ転送をトリガーする (つまり、サービスに要求を行い、応答を取得する) には、返された `Mono` をサブスクライブする必要があります。 そのため、これらの Reactive Streams を処理するときは、`subscribe()` を呼び出す必要があることに留意してください。それを行うまで何も起こらないためです。

次の例は、`Mono` をサブスクライブし、構成値をコンソールに出力する方法を示しています。

```java
ConfigurationAsyncClient asyncClient = new ConfigurationClientBuilder()
    .connectionString("<your connection string>")
    .buildAsyncClient();

asyncClient.getConfigurationSetting("<your config key>", "<your config value>").subscribe(
    config -> System.out.println("Config value: " + config.getValue()),
    ex -> System.out.println("Error getting configuration: " + ex.getMessage()),
    () -> System.out.println("Successfully retrieved configuration setting"));

System.out.println("Done");
```

クライアントで `getConfigurationSetting()` を呼び出した後、このサンプル コードは結果をサブスクライブし、3 つの別個のラムダを提供していることに注目してください。 最初のラムダは、応答が成功したときにトリガーされ、サービスから受信したデータを使用します。 2 番目のラムダは、構成の取得中にエラーが発生した場合にトリガーされます。 3 番目のラムダは、データ ストリームが完了したときに呼び出されます。つまり、このストリームにはこれ以上のデータ要素が予期されないことを意味します。

> [!NOTE]
> すべての非同期プログラミングと同様に、サブスクリプションが作成された後、実行は通常どおり続きます。 プログラムをアクティブにして実行を維持するものがない場合、非同期操作が完了する前に終了することがあります。 `subscribe()` を呼び出した main スレッドは、ユーザーが Azure App Configuration へのネットワーク呼び出しを行って応答を受信するまで待機しません。 実稼働システムでは、他の処理を続行する場合がありますが、この例では、`Thread.sleep()` を呼び出すことによって短い遅延を追加したり、`CountDownLatch` を使用して非同期処理が完了できるようにしたりすることが可能です。

次の例に示すように、`Flux` を返す API も同様のパターンに従います。 違いは、`subscribe()` メソッドに提供される最初のコールバックが、応答の各データ要素に対して複数回呼び出されることです。 エラーまたは完了のコールバックは正確に 1 回だけ呼び出され、終了シグナルと見なされます。 これらのいずれかのシグナルをパブリッシャーから受信した場合、他のコールバックは呼び出されません。

```java
EventHubConsumerAsyncClient asyncClient = new EventHubClientBuilder()
    .connectionString("<your connection string>")
    .consumerGroup("<your consumer group>")
    .buildAsyncConsumerClient();

asyncClient.receive().subscribe(
    event -> System.out.println("Sequence number of received event: " + event.getData().getSequenceNumber()),
    ex -> System.out.println("Error receiving events: " + ex.getMessage()),
    () -> System.out.println("Successfully completed receiving all events"));
```

### <a name="backpressure"></a>バックプレッシャ

サブスクライバーが処理できるよりも速い速度でソースがデータを生成している場合、何が起きるでしょうか。 サブスクライバーがデータを処理しきれずに、メモリ不足エラーが発生する可能性があります。 サブスクライバーには、処理しきれない場合に速度を落とすようにパブリッシャーに伝える方法が必要です。 既定では、上の例に示すように `Flux` で `subscribe()` を呼び出すと、サブスクライバーは、バインドされていないデータのストリームを要求し、パブリッシャーに対してできるだけ迅速にデータを送信するように指示します。 この動作は常に望ましいとは限りません。サブスクライバーは、"バックプレッシャ" によって発行の速度を制御する必要がある場合があります。 バックプレッシャを使用すると、サブスクライバーがデータ要素のフローを制御できます。 サブスクライバーは、処理可能な限定された数のデータ要素を要求します。 サブスクライバーがこれらの要素の処理を完了した後、サブスクライバーはさらに要求できます。 バックプレッシャを使用すると、データ転送のプッシュ モデルをプッシュプル モデルに変換できます。

次の例は、Event Hubs コンシューマーがイベントを受信する速度を制御する方法を示しています。

```java
EventHubConsumerAsyncClient asyncClient = new EventHubClientBuilder()
    .connectionString("<your connection string>")
    .consumerGroup("<your consumer group>")
    .buildAsyncConsumerClient();

asyncClient.receive().subscribe(new Subscriber<PartitionEvent>() {
    private Subscription subscription;

    @Override
    public void onSubscribe(Subscription subscription) {
        this.subscription = subscription;
        this.subscription.request(1); // request 1 data element to begin with
    }

    @Override
    public void onNext(PartitionEvent partitionEvent) {
        System.out.println("Sequence number of received event: " + partitionEvent.getData().getSequenceNumber());
        this.subscription.request(1); // request another event when the subscriber is ready
    }

    @Override
    public void onError(Throwable throwable) {
        System.out.println("Error receiving events: " + throwable.getMessage());
    }

    @Override
    public void onComplete() {
        System.out.println("Successfully completed receiving all events")
    }
});
```

サブスクライバーがパブリッシャーに最初に "接続" すると、パブリッシャーはサブスクライバーに `Subscription` インスタンスを渡します。これにより、データ転送の状態が管理されます。 この `Subscription` は、`request()` を呼び出して、さらに処理できるデータ要素の数を指定することによって、サブスクライバーがバックプレッシャを適用できる手段です。

サブスクライバーが、`onNext()` を呼び出すたびに複数のデータ要素を要求する場合 (たとえば `request(10)`)、パブリッシャーは次の 10 個の要素を (それらが利用可能であるか利用可能になった場合に) 直ちに送信します。 これらの要素はサブスクライバー側のバッファーに蓄積されます。また、各 `onNext()` の呼び出しではさらに 10 個が要求されるため、送信するデータ要素がパブリッシャーになくなるか、サブスクライバーのバッファーがオーバーフローしてメモリ不足エラーが発生するまで、バックログが増加し続けます。

### <a name="cancel-a-subscription"></a>サブスクリプションを取り消す

サブスクリプションは、パブリッシャーとサブスクライバーの間のデータ転送の状態を管理します。 サブスクリプションは、すべてのデータがサブスクライバーに転送される処理がパブリッシャー側で完了するか、サブスクライバーがデータの受信を要求しなくなるまでアクティブです。 下に示すように、いくつかの方法でサブスクリプションを取り消すことができます。

次の例では、サブスクライバーを破棄してサブスクリプションを取り消します。

```java
EventHubConsumerAsyncClient asyncClient = new EventHubClientBuilder()
    .connectionString("<your connection string>")
    .consumerGroup("<your consumer group>")
    .buildAsyncConsumerClient();

Disposable disposable = asyncClient.receive().subscribe(
    partitionEvent -> {
        Long num = partitionEvent.getData().getSequenceNumber()
        System.out.println("Sequence number of received event: " + num);
    },
    ex -> System.out.println("Error receiving events: " + ex.getMessage()),
    () -> System.out.println("Successfully completed receiving all events"));

// much later on in your code, when you are ready to cancel the subscription,
// you can call the dispose method, as such:
disposable.dispose();
```

次の例では、`Subscription` で `cancel()` メソッドを呼び出してサブスクリプションを取り消します。

```java
EventHubConsumerAsyncClient asyncClient = new EventHubClientBuilder()
    .connectionString("<your connection string>")
    .consumerGroup("<your consumer group>")
    .buildAsyncConsumerClient();

asyncClient.receive().subscribe(new Subscriber<PartitionEvent>() {
    private Subscription subscription;

    @Override
    public void onSubscribe(Subscription subscription) {
        this.subscription = subscription;
        this.subscription.request(1); // request 1 data element to begin with
    }

    @Override
    public void onNext(PartitionEvent partitionEvent) {
        System.out.println("Sequence number of received event: " + partitionEvent.getData().getSequenceNumber());
        this.subscription.cancel(); // Cancels the subscription. No further event is received.
    }

    @Override
    public void onError(Throwable throwable) {
        System.out.println("Error receiving events: " + throwable.getMessage());
    }

    @Override
    public void onComplete() {
        System.out.println("Successfully completed receiving all events")
    }
});
```

## <a name="conclusion"></a>まとめ

スレッドは高価なリソースであるため、リモート サービス呼び出しからの応答の待機に無駄遣いすべきではありません。 マイクロサービス アーキテクチャの導入が増加するにつれて、リソースを効率的にスケーリングして使用することがきわめて重要になります。 非同期 API は、ネットワーク バインドされた操作がある場合に有利です。 Azure SDK for Java では、システム リソースを最大限に活用するために、非同期操作用の豊富な API セットが提供されています。 これらの非同期クライアントを試すことを強くお勧めします。

特定のタスクに最適な演算子の詳細については、「[Reactor 3 リファレンス ガイド](https://projectreactor.io/docs/core/release/reference/index.html)」の「[どの演算子が必要か](https://projectreactor.io/docs/core/release/reference/#which-operator)」を参照してください。

## <a name="next-steps"></a>次のステップ

さまざまな非同期プログラミングの概念を深く理解したら、結果を反復処理する方法を学習することが重要です。 最適な反復戦略と、改ページ位置の自動修正のしくみの詳細については、「[Azure SDK for Java での改ページ位置の自動修正と反復処理](pagination.md)」を参照してください。
