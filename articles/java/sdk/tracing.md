---
title: Azure SDK for Java でトレースを構成する
description: トレースに関連した Azure SDK for Java の概念の概要
author: samvaity
ms.date: 02/02/2021
ms.topic: conceptual
ms.custom: devx-track-java
ms.author: savaity
ms.openlocfilehash: 2dc2085ac71167cefd8fed5475dc9744cf520fea
ms.sourcegitcommit: 71847ee0a1fee3f3320503629d9a8c82319a1f6a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/03/2021
ms.locfileid: "99528458"
---
# <a name="configure-tracing-in-the-azure-sdk-for-java"></a>Azure SDK for Java でトレースを構成する

この記事では、トレース機能を統合するために Azure SDK for Java を構成する方法の概要について説明します。

Azure SDK for Java では、[OpenTelemetery](https://opentelemetry.io/) ベースの [azure-core-tracing-opentelemetry プラグイン](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/core/azure-core-tracing-opentelemetry#azure-tracing-opentelemetry-client-library-for-java)に依存関係を組み込むことによって、すべてのクライアント ライブラリでトレースを有効にします。 OpenTelemetry は、クラウドネイティブ ソフトウェアのテレメトリ データを生成、キャプチャ、収集するためによく使用されるオープンソースの監視フレームワークです。

トレースに関連する 2 つの重要な概念があります。**スパン** と **トレース** です。 スパンは、トレース内の 1 つの操作を表します。 スパンは、HTTP 要求、リモート プロシージャ コール (RPC)、データベース クエリ、またはコードのパスを表すことができます。 トレースは、システムでの作業のパスを示すスパンのツリーです。 TraceID と呼ばれる一意の 16 バイト シーケンスを使用して、トレースをそれ自体で区別できます。 これらの概念、および OpenTelemetry との関係の詳細については、[OpenTelemetry のドキュメント](https://opentelemetry.io/docs/)を参照してください。

Java 用 Azure クライアント ライブラリでトレースを有効にするには、2 つの方法があります。

1. Azure SDK for Java に組み込まれている機能を有効にする。
2. インプロセス エージェントでトレース データを収集し、コードを変更せずにそれを送信できるようにする。

## <a name="enable-tracing-in-the-azure-sdk-for-java"></a>Azure SDK for Java でトレースを有効にする

すべての Azure Java クライアント ライブラリでトレースを有効にするには、`azure-core-tracing-opentelemetry` と `opentelemetry-sdk` の依存関係をアプリケーションに追加します。 たとえば、Maven で、次の依存関係を追加します。

```xml
<dependency>
  <groupId>com.azure</groupId>
  <artifactId>azure-core-tracing-opentelemetry</artifactId>
  <version>1.0.0-beta.6</version>
</dependency>

<dependency>
  <groupId>io.opentelemetry</groupId>
  <artifactId>opentelemetry-sdk</artifactId>
  <version>0.8.0</version>
</dependency>
```

この依存関係を追加すると、トレースが有効になり、すべての HTTP 要求にトレースが含まれます。 現在、次の 2 つの問題があります。

1. 受信親スパンとの統合がない。
2. 生成されたトレースは、後で分析するためにどこにもエクスポートされない。

次のセクションでは、これらの問題に取り組みます。

### <a name="integrate-parent-spans"></a>親スパンを統合する

前述のように、依存関係を含めることにより、Azure クライアント ライブラリ内のトレースが有効になります。 ただし、受信要求によって Azure クライアント ライブラリが呼び出される Web 環境などで、受信トレース データと統合されません。 トレースを有効にするには、アプリケーションにルート スパンを作成し、それを Azure クライアント ライブラリ呼び出しに渡すことができます。これにより、このスパンは Azure サービスへの適切な送信要求にカプセル化されます。 このタスクを実行するには、次の例に示すように、すべてのクライアント メソッドで `Context` パラメーターを使用します。

```java
// The 'span' given in this context is the parent span key received from the incoming request.
Context traceContext = new Context(PARENT_SPAN_KEY, span);

// This example code passes a new configuration setting to a service, but also includes
// the traceContext from above, so that it may be picked up by the http transport and included as appropriate.
appConfigClient.setConfigurationSettingWithResponse(new ConfigurationSetting().setKey("hello").setValue("world"), true, traceContext);
```

親スパンが指定されていない場合は、すべてのクライアント ライブラリの送信要求をカプセル化するために、新しい親スパンが作成されます。 Azure クライアント ライブラリのメソッドを呼び出すたびに 2 つのスパンが作成されます。1 つはクライアント ライブラリの進行状況をトレースし、もう 1 つは発信 HTTP 要求スパンをトレースします。

#### <a name="tracer-span-attributes"></a>トレーサーのスパン属性

OpenTelemetry の[セマンティック規則](https://github.com/open-telemetry/opentelemetry-specification/blob/e9340d74f1ba0b651b3581d6bd5df6a92b772e18/semantic-conventions.md)に記載されている必須の標準属性に加えて、Azure クライアント ライブラリでは、次の属性を使ってスパンに注釈が付けられます。

* `az.namespace`:Azure サービスにマップされた Microsoft リソース プロバイダーの[名前空間](/azure/azure-resource-manager/management/azure-services-resource-providers)。
* `x-ms-request-id`:要求の一意の識別子。
* `span.kind`:トレースにおけるスパン、その親、およびその子の関係を示します。
* `span.status.message`:終了したスパンの状態を表します。
* `span.status.code`:終了したスパンの状態コードを表します。

実行されている操作に関するさらなるメタデータは、スパン名に取り込まれます。 HTTP スパン名は URI パスの値に設定され、ライブラリ メソッド呼び出しスパンは `<namespace qualified type>.<method name>` の形式になります。

たとえば、構成設定を指定するための App Configuration クライアント要求 (つまり `appConfigClient.setConfigurationSettingWithResponse(new ConfigurationSetting().setKey("hello").setValue("world")`) では、次の 2 つのスパンが作成されます。

* `AppConfig.setKey` という名前のライブラリ メソッド スパン。
* `/kv/hello` という名前の HTTP 送信要求スパン。

### <a name="configure-tracing-exports"></a>トレースのエクスポートを構成する

トレース情報を利用する必要があるアプリケーションでは、トレースを分散トレース ストア ([Zipkin](https://zipkin.io/)、[Jaeger](https://www.jaegertracing.io/)、[Azure Monitor](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/monitor/microsoft-opentelemetry-exporter-azuremonitor#azure-monitor-opentelemetry-exporter-client-library-for-java) など) にエクスポートする必要があります。 次の例では、Jaeger 固有の API を使用して、localhost ポート 14250 で実行されている Jaeger 分散トレース ストアにトレース情報をエクスポートするように構成します。

```java
ManagedChannel channel = ManagedChannelBuilder.forAddress("localhost", 14250).usePlaintext().build();
JaegerGrpcSpanExporter exporter = JaegerGrpcSpanExporter.newBuilder()
    .setChannel(channel)
    .setServiceName("Sample")
    .setDeadline(0)
    .build();
TracerSdkFactory tracerSdkFactory = (TracerSdkFactory) OpenTelemetry.getTracerFactory();
tracerSdkFactory.addSpanProcessor(SimpleSpansProcessor.newBuilder(exporter).build());
```

## <a name="enable-tracing-with-the-in-process-agent"></a>インプロセス エージェントでトレースを有効にする

[Azure Monitor](/azure/azure-monitor/overview) の機能である Application Insights を使用すると、大規模な分散システムでアプリケーションを後で分析するためにデータを自動的に収集して送信することができます。 このインストルメンテーションによってアプリケーションが監視され、"インストルメンテーション キー" と呼ばれる一意の GUID を使用して、テレメトリ データが [Azure Application Insights リソース](/azure/azure-monitor/app/app-insights-overview)に転送されます。

[Java インプロセス エージェント](/azure/azure-monitor/app/java-in-process-agent)を使用することで、コードを変更することなく、アプリケーションの監視を有効にできます。 また、[azure-core-tracing-opentelemetry](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/core/azure-core-tracing-opentelemetry#azure-tracing-opentelemetry-client-library-for-java) 依存関係をプロジェクトに追加する必要もあります。 このタスクを完了したら、Application Insights ダッシュボードを使用して要求をインストルメント化し、パフォーマンス カウンターを収集し、パフォーマンスに関する問題と例外を診断し、アプリケーション内でのユーザーの操作内容を追跡するコードを作成できます。

## <a name="next-steps"></a>次のステップ

Azure SDK for Java の主要な分野横断的機能の概要を理解できたので、[Java および Azure ID を使用した Azure 認証](identity.md)に関する記事を参照して、セキュリティで保護されたアプリケーションの作成方法を確認してください。
