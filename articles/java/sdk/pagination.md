---
title: Azure SDK for Java での改ページと反復処理
description: 改ページと反復処理に関連した Azure SDK for Java の概念の概要
author: anuchandy
ms.date: 02/02/2021
ms.topic: conceptual
ms.custom: devx-track-java
ms.author: anuchan
ms.openlocfilehash: 0cb9e519931d24dcd97aa4a52742974427df5969
ms.sourcegitcommit: 71847ee0a1fee3f3320503629d9a8c82319a1f6a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/03/2021
ms.locfileid: "99528482"
---
# <a name="pagination-and-iteration-in-the-azure-sdk-for-java"></a>Azure SDK for Java での改ページと反復処理

この記事では、Azure SDK for Java の改ページと反復処理の機能を使用して、大規模なデータ セットを効率的かつ生産的に操作する方法の概要について説明します。

Azure Java SDK 内のクライアント ライブラリによって提供される多くの操作では、複数の結果が返されます。 Azure Java SDK では、一貫性を通じて開発者エクスペリエンスが最大限に向上するように、そのような場合の一連の許容可能な戻り値の型が定義されています。 使用される戻り値の型は、同期 API では `PagedIterable`、非同期 API では `PagedFlux` になります。 これらの API は、ユース ケースの違いによって若干異なりますが、概念的には同じ要件を持っています。

- コレクション内の各要素を簡単に個別に反復処理できるようにすると、手動による改ページや後続トークン追跡の必要性を無視できます。 `PagedIterable` と `PagedFlux` はどちらも、特定の型 `T` に逆シリアル化された、改ページされた応答を反復処理することで、このタスクを簡単にします。 `PagedIterable` では `Iterable` インターフェイスが実装され、`Stream` を受信する API が提供されます。一方、`PagedFlux` では `Flux` が提供されます。 どの場合も、改ページは透過的に行われ、反復する結果がまだある間は反復処理は続行されます。

- ページごとに明示的に反復処理できるようにします。 こうすることで、要求が行われたときをより明確に把握でき、ページごとの応答情報にアクセスできます。 `PagedIterable` と `PagedFlux` のどちらにも、個別の要素ごとではなく、ページごとに反復処理する適切な型を返すメソッドがあります。

この記事は、Java Azure SDK の同期と非同期 API の間で分割されています。 同期クライアントを操作するときには同期反復処理 API が表示され、非同期クライアントを操作するときには非同期反復 API が表示されます。

## <a name="synchronous-pagination-and-iteration"></a>同期的な改ページと反復処理

このセクションでは、同期 API について説明します。

### <a name="iterate-over-individual-elements"></a>個々の要素を反復処理する

前述のように、最も一般的なユース ケースは、ページごとではなく、各要素を個別に反復処理することです。 次のコード例では、`PagedIterable` API を使用して、任意の反復スタイルでこの機能を実装する方法を示します。

#### <a name="use-a-for-each-loop"></a>for-each ループを使用する

`PagedIterable` では `Iterable` が実装されているため、次の例に示すように要素を反復処理できます。

```java
PagedIterable<Secret> secrets = client.listSecrets();
for (Secret secret : secrets) {
   System.out.println("Secret is: " + secret);
}
```

#### <a name="use-stream"></a>Stream を使用する

`PagedIterable` には `stream()` メソッドが定義されているため、次の例に示すように、それを呼び出して標準の Java Stream API を使用できます。

```java
client.listSecrets()
      .stream()
      .forEach(secret -> System.out.println("Secret is: " + secret));
```

#### <a name="use-iterator"></a>Iterator を使用する

`PagedIterable` では `Iterable` が実装されているため、次の例に示すように、Java の Iterator プログラミング スタイルを可能にする `iterator()` メソッドもあります。

```java
Iterator<Secret> secrets = client.listSecrets().iterator();
while (it.hasNext()) {
   System.out.println("Secret is: " + it.next());
}
```

### <a name="iterate-over-pages"></a>ページを反復処理する

個々のページを操作する場合は、たとえば、HTTP 応答情報が必要な場合や、反復処理の履歴を保持するために後続トークンが重要な場合などに、ページごとに反復処理を行うことができます。 反復処理をページごとまたは各項目ごとのどちらで行っているかにかかわらず、パフォーマンスや、サービスの呼び出し回数に違いはありません。 基になる実装によって、オンデマンドで次のページが読み込まれます。また、いつでも `PagedFlux` の登録を解除すると、サービスの呼び出しは行われなくなります。

#### <a name="use-a-for-each-loop"></a>for-each ループを使用する

`listSecrets()` を呼び出すと `PagedIterable` を取得します。これには、`iterableByPage()` API があります。 この API では、`Iterable<Secret>` ではなく `Iterable<PagedResponse<Secret>>` が生成されます。 `PagedResponse` では、次の例に示すように、応答メタデータと、後続トークンへのアクセスが提供されます。

```java
Iterable<PagedResponse<Secret>> secretPages = client.listSecrets().iterableByPage();
for (PagedResponse<Secret> page : secretPages) {
   System.out.println("Response code: " + page.getStatusCode());
   System.out.println("Continuation Token: " + page.getContinuationToken());
   page.getElements().forEach(secret -> System.out.println("Secret value: " + secret))
}
```

また、後続トークンを受け取る `iterableByPage` オーバーロードもあります。 このオーバーロードは、後で同じ反復地点に戻るときに呼び出すことができます。

#### <a name="use-stream"></a>Stream を使用する

次の例では、上に示したのと同じ操作を `streamByPage()` メソッドで実行する方法を示します。 この API にも、後で同じ反復地点に戻るための後続トークンのオーバーロードがあります。

```java
client.listSecrets()
      .streamByPage()
      .forEach(page -> {
          System.out.println("Response code: " + page.getStatusCode());
          System.out.println("Continuation Token: " + page.getContinuationToken());
          page.getElements().forEach(secret -> System.out.println("Secret value: " + secret))
      });
```

## <a name="asynchronously-observe-pages-and-individual-elements"></a>ページと個々の要素を非同期に監視する

このセクションでは、非同期 API について説明します。 非同期 API では、ネットワーク呼び出しは、`subscribe()` を呼び出すメイン スレッドとは異なるスレッドで行われます。 これは、結果が使用可能になる前にメイン スレッドが終了する可能性があることを意味します。 非同期操作が完了するまでアプリケーションが終了しないようにすることもできます。

### <a name="observe-individual-elements"></a>個々の要素を監視する

次の例では、`PagedFlux` API を使用して個々の要素を非同期に観察する方法を示します。 Flux の型をサブスクライブするには、さまざまな方法があります。 詳細については、[Reactor 3 リファレンス ガイド](https://projectreactor.io/docs/core/release/reference)で [Flux または Mono を作成してサブスクライブする簡単な方法](https://projectreactor.io/docs/core/release/reference/#_simple_ways_to_create_a_flux_or_mono_and_subscribe_to_it)を参照してください。 この例は 1 つの種類で、3 つのラムダ式 (コンシュ―マー、エラー コンシュ―マー、および完了コンシュ―マー用にそれぞれ 1 つ) があります。 3 つすべてを指定することをお勧めしますが、指定する必要があるのコンシュ―マーだけ (また状況によっては、エラー コンシュ―マーも) の場合もあります。

 ```java
asyncClient.listSecrets()
    .subscribe(secret -> System.out.println("Secret value: " + secret),
        ex -> System.out.println("Error listing secrets: " + ex.getMessage()),
        () -> System.out.println("Successfully listed all secrets"));
 ```

### <a name="observe-pages"></a>ページを監視する

 次の例では、`PagedFlux` API を使用して各ページを非同期に監視する方法を示します。この場合も、`byPage()` API を使用し、コンシュ―マー、エラー コンシュ―マー、完了コンシュ―マーを指定して行います。

  ```java
asyncClient.listSecrets().byPage()
    .subscribe(page -> {
            System.out.println("Response code: " + page.getStatusCode());
            System.out.println("Continuation Token: " + page.getContinuationToken());
            page.getElements().forEach(secret -> System.out.println("Secret value: " + secret))
        },
        ex -> System.out.println("Error listing pages with secret: " + ex.getMessage()),
        () -> System.out.println("Successfully listed all pages with secret"));
 ```

## <a name="next-steps"></a>次のステップ

Azure SDK for Java での改ページと反復処理について理解したので、「[Azure SDK for Java での実行時間の長い操作](lro.md)」を確認することを検討してください。 実行時間の長い操作とは、通常、サーバー側で何らかの作業が必要なために、多くの標準的な HTTP 要求よりも長い時間実行される操作のことです。
