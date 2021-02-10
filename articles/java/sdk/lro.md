---
title: Azure SDK for Java での実行時間の長い操作
description: 実行時間の長い操作に関する Azure SDK for Java の概念の概要
author: anuchandy
ms.date: 02/02/2021
ms.topic: conceptual
ms.custom: devx-track-java
ms.author: anuchan
ms.openlocfilehash: a20fb7a638fefd0182bea89e3d5c4555442a4547
ms.sourcegitcommit: 71847ee0a1fee3f3320503629d9a8c82319a1f6a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/03/2021
ms.locfileid: "99528487"
---
# <a name="long-running-operations-in-the-azure-sdk-for-java"></a>Azure SDK for Java での実行時間の長い操作

この記事では、Azure SDK for Java での実行時間の長い操作の使用に関する概要について説明します。

Azure での特定の操作は、完了するまでに長時間かかる場合があります。 こうした操作は、迅速な要求と応答のフローである標準的な HTTP スタイルの範囲外です。 たとえば、ソース URL から Storage BLOB へのデータのコピーや、フォームを認識するモデルのトレーニングは、数秒から数分かかる操作です。 このような操作は実行時間の長い操作と呼ばれ、多くの場合、「LRO」と略されます。 LRO が完了するまでには、要求された操作とサーバー側で実行する必要があるプロセスに応じて、数秒、数分、数時間、数日、またはそれ以上長く時間がかかる場合があります。

Azure 用 Java クライアント ライブラリには、実行時間の長い操作はすべて `begin` プレフィックスで始まるという規則があります。 このプレフィックスは、この操作は実行時間が長いこと、また、この操作とのやりとりの方法は通常の要求と応答のフローと若干異なることを示しています。 `begin` プレフィックスに加えて、実行時間の長い操作の全機能を有効にするために、操作の戻り値の型も通常とは異なります。 Azure SDK for Java の大半のものと同様に、実行時間の長い操作向けに同期と非同期の両方の API があります。

* 同期クライアントでは、実行時間の長い操作によって `SyncPoller` インスタンスが返されます。
* 非同期クライアントでは、実行時間の長い操作によって `PollerFlux` インスタンスが返されます。

`SyncPoller` と `PollerFlux` はどちらも、実行時間の長いサーバー側操作とのやりとりを簡略化することを目的とした、クライアント側の抽象化です。 この記事の残りの部分では、これらの種類を操作する際のベスト プラクティスについて概説します。

## <a name="synchronous-long-running-operations"></a>実行時間の長い同期操作

`SyncPoller` を返す API を呼び出すと、実行時間の長い操作が直ちに開始されます。 API によって `SyncPoller` が即時に返されるので、実行時間の長い操作の進行状況を監視し、最終的な結果を取得できます。 次の例は、`SyncPoller` を使用して、実行時間の長い操作の進行状況を監視する方法を示しています。

```java
SyncPoller<UploadBlobProgress, UploadedBlobProperties> poller = syncClient.beginUploadFromUri(<URI to upload from>)
PollResponse<UploadBlobProgress> response;

do {
    response = poller.poll();
    System.out.println("Status of long running upload operation: " + response.getStatus());
    Duration pollInterval = response.getRetryAfter();
    TimeUnit.MILLISECONDS.sleep(pollInterval.toMillis());
} while (!response.getStatus().isComplete());
```

この例は、`SyncPoller` で `poll()` メソッドを使用して、実行時間の長い操作の進行状況に関する情報を取得します。 このコードではコンソールに状態が出力されますが、よりうまく実装すれば、この状態に基づいて適切な決定を行えます。

`getRetryAfter()` メソッドは、次のポーリングまでの待機時間に関する情報を返します。 Azure の実行時間の長い操作のほとんどは、HTTP 応答の一部としてポーリング遅延 (つまり、一般的に使用される `retry-after` ヘッダー) を返します。 応答にポーリング遅延が含まれていない場合、`getRetryAfter()` メソッドは、実行時間の長い操作の呼び出し時に指定された期間を返します。

上記の例では、`do..while` ループを使用して、実行時間の長い操作が完了するまで繰り返しポーリングします。 これらの途中結果に関心がない場合は、代わりに `waitForCompletion()` を呼び出すことができます。 この呼び出しは、実行時間の長い操作が完了し、最後のポーリング応答を返すまで、現在のスレッドをブロックします。

```java
PollResponse<UploadBlobProgress> response = poller.waitForCompletion();
```

最後のポーリング応答で、実行時間の長い操作が正常に完了したことが示された場合は、`getFinalResult()` を使用して最終的な結果を取得できます。

```java
if (LongRunningOperationStatus.SUCCESSFULLY_COMPLETED == response.getStatus()) {
    UploadedBlobProperties result = poller.getFinalResult();
}
```

`SyncPoller` のその他の便利な API には、次のものがあります。

1. `waitForCompletion(Duration)`: 指定されたタイムアウト期間中、実行時間の長い操作の完了を待機します。
1. `waitUntil(LongRunningOperationStatus)`: 実行時間の長い操作の、指定された状態が受信されるまで待機します。
1. `waitUntil(LongRunningOperationStatus, Duration)`: 実行時間の長い操作の指定された状態が受信されるまで、または指定されたタイムアウト期間が過ぎるまで待機します。

## <a name="asynchronous-long-running-operations"></a>実行時間の長い非同期操作

下の例では、`PollerFlux` を使用して実行時間の長い操作を監視する方法を示します。 非同期 API では、ネットワーク呼び出しは、`subscribe()` を呼び出すメイン スレッドとは異なるスレッドで行われます。 これは、結果が使用可能になる前にメイン スレッドが終了する可能性があることを意味します。 非同期操作が完了するまでアプリケーションが終了しないようにするのは、ご自身で行ってください。

非同期 API から `PollerFlux` が直ちに返されますが、実行時間の長い操作自体は、ユーザーが `PollerFlux` をサブスクライブするまで開始されません。 このプロセスは、`Flux` ベースの API すべての動作方法です。 次の例は、実行時間の長い非同期操作を示しています。

```java
asyncClient.beginUploadFromUri(...)
    .subscribe(response -> System.out.println("Status of long running upload operation: " + response.getStatus()));
```

次の例では、実行時間の長い操作に対する断続的な状態更新情報を取得します。 これらの更新情報を使用して、実行時間の長い操作が予期した方法で引き続き動作しているかどうかを判断できます。 この例ではコンソールに状態が出力されますが、よりうまく実装すれば、この状態に基づいて適切なエラー処理を行えます。

途中の状態更新情報に関心がなく、最終的な結果が到着したときに通知を受け取りたいだけであれば、次の例のようなコードを使用できます。

```java
asyncClient.beginUploadFromUri(...)
    .last()
    .flatMap(response -> {
        if (LongRunningOperationStatus.SUCCESSFULLY_COMPLETED == response.getStatus()) {
            return response.getFinalResult();
        }
        return Mono.error(new IllegalStateException("Polling completed unsuccessfully with status: "+ response.getStatus()));
    })
    .subscribe(
        finalResult -> processFormPages(finalResult),
        ex -> countDownLatch.countDown(),
        () -> countDownLatch.countDown());
```

このコードでは、`last()` を呼び出して、実行時間の長い操作の最終的な結果を取得します。 この呼び出しでは、すべてのポーリングの完了を待機することを `PollerFlux` に指示します。その時点で、実行時間の長い操作が最終状態に達しているので、その状態を検査して結果を判断できます。 実行時間の長い操作が正常に完了したことがポーラーにより示された場合は、最終的な結果を取得し、それをサブスクライブ呼び出しのコンシューマーに渡すことができます。

## <a name="next-steps"></a>次のステップ

Azure SDK for Java での実行時間の長い API を理解したので、「[Azure SDK for Java でプロキシを構成する](proxying.md)」を参照して HTTP クライアントをさらにカスタマイズする方法について確認します。
