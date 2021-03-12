---
title: Azure SDK for Java と Logback によるログ
description: Azure SDK for Java と Logback の統合の概要
author: srnagar
ms.date: 02/02/2021
ms.topic: conceptual
ms.custom: devx-track-java
ms.author: srnagar
ms.openlocfilehash: 785bb65a1a6f55314246d4c1410891717f8bbd6f
ms.sourcegitcommit: 576c878c338d286060010646b96f3ad0fdbcb814
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102118160"
---
# <a name="log-with-the-azure-sdk-for-java-and-logback"></a>Azure SDK for Java と Logback によるログ

この記事では、Logback を使用したログを、Azure SDK for Java を使用するアプリケーションに追加する方法の概要について説明します。 「[Azure SDK for Java でログを構成する](logging-overview.md)」で説明されているように、すべての Azure クライアント ライブラリが [SLF4J](http://www.slf4j.org/) を通じてログするため、[Logback](http://logback.qos.ch/) などのログ記録フレームワークを使用できます。

Logback のログを有効にするには、2 つのことを行う必要があります。

1. Logback ライブラリを依存関係として含めます。
2. */src/main/resources* プロジェクト ディレクトリに *logback.xml* というファイルを作成します。

Logback の構成に関する詳細については、Logback のドキュメントの「[logbackの設定](http://logback.qos.ch/manual/configuration.html)」を参照してください。

## <a name="add-the-maven-dependency"></a>Maven の依存関係を追加する

Maven の依存関係を追加するには、プロジェクトの *pom.xml* ファイルに次の XML を含めます。 *1.2.3* バージョン番号は [Logback Classic Module ページ](https://mvnrepository.com/artifact/ch.qos.logback/logback-classic)に示されている最新のリリース バージョン番号に置き換えてください。

```xml
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.3</version>
</dependency>
```

## <a name="add-logbackxml-to-your-project"></a>プロジェクトに logback.xml を追加する

[Logback](https://logback.qos.ch/manual/introduction.html) は、よく使用されるログ記録フレームワークの 1 つです。 Logback のログを有効にするには、プロジェクトの *./src/main/resources* ディレクトリに *logback.xml* というファイルを作成します。 このファイルには、必要とするログをカスタマイズするためのログ構成が含まれます。 *logback.xml* の構成に関する詳細については、Logback のドキュメントの「[logbackの設定](https://logback.qos.ch/manual/configuration.html)」を参照してください。

### <a name="console-logging"></a>[コンソールのログ記録]

次の例に示すように、コンソールにログする Logback 構成を作成できます。 この例は、INFO レベル以上であるログ イベントをすべてその発生元に関係なくログするように構成されています。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <layout class="ch.qos.logback.classic.PatternLayout">
      <Pattern>
        %black(%d{ISO8601}) %highlight(%-5level) [%blue(%t)] %blue(%logger{100}): %msg%n%throwable
      </Pattern>
    </layout>
  </appender>

  <root level="INFO">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
```

### <a name="log-azure-core-errors"></a>Azure コアのエラーをログする

下の構成例は、前述の構成に似ていますが、すべての `com.azure.core` パッケージ化クラス (サブパッケージを含む) からログが発生するレベルを下げています。 この方法では、INFO レベル以上のものがすべてログされます。ただし、`com.azure.core` については ERROR レベル以上のものだけがログされます。 たとえば、`com.azure.core` 内のコードのノイズが多すぎると思う場合に、この方法を使用できます。 また、このような構成は、両方向で設定できます。 たとえば、`com.azure.core` 内のクラスからより多くのデバッグ情報を取得する場合は、この設定を DEBUG に変更できます。

特定のクラスまたは特定のパッケージのログをきめ細かく制御することができます。 下に示すように、`com.azure.core` はすべてのコア クラスの出力を制御しますが、実行中のアプリケーションという観点から最も有益である状況に合わせて、`com.azure.security.keyvault` または同等のものを使用して同じように出力を制御できます。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>%message%n</pattern>
    </encoder>
  </appender>

  <logger name="com.azure.core" level="ERROR" />

  <root level="INFO">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
```

### <a name="log-to-a-file-with-log-rotation-enabled"></a>ログのローテーションが有効になっているファイルにログする

上記の例では、コンソールにログしています。これは通常、推奨されるログの場所ではありません。 代わりに、次の構成を使用して、1 時間ごとのロールオーバーと gzip 形式のアーカイブを行いながらファイルにログします。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <property name="LOGS" value="./logs" />
  <appender name="RollingFile" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>${LOGS}/spring-boot-logger.log</file>
    <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
      <Pattern>%d %p %C{1.} [%t] %m%n</Pattern>
    </encoder>

    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
      <!-- rollover hourly and gzip logs -->
      <fileNamePattern>${LOGS}/archived/spring-boot-logger-%d{yyyy-MM-dd-HH}.log.gz</fileNamePattern>
    </rollingPolicy>
  </appender>

  <!-- LOG everything at INFO level -->
  <root level="INFO">
    <appender-ref ref="RollingFile" />
  </root>
</configuration>
```

### <a name="spring-applications"></a>Spring アプリケーション

Spring フレームワークは、ログ構成など、さまざまな構成について Spring *application.properties* ファイルを読み取ることで機能します。 ただし、任意のファイルから Logback 構成を読み取るように Spring アプリケーションを構成できます。 これを行うには、Spring の */src/main/resources/application.properties* ファイルに次の行を追加して、*logback.xml* 構成ファイルを指すように `logging.config` プロパティを構成します。

```properties
logging.config=classpath:logback.xml
```

## <a name="next-steps"></a>次のステップ

この記事では、Logback の構成と、Azure SDK for Java でこれをログに使用する方法について説明しました。 Azure SDK for Java はすべての SLF4J ログ記録フレームワークで動作するため、詳細については、「[SLF4J ユーザー マニュアル](http://www.slf4j.org/manual.html)」を確認することを検討してください。 Logback を使用する場合は、その Web サイトにも数多くの構成ガイダンスがあります。 詳細については、Logback のドキュメントの「[logbackの設定](http://logback.qos.ch/manual/configuration.html)」を参照してください。

ログについて理解できたら、Azure が提供する、[Spring](../spring-framework/spring-boot-starters-for-azure.md) や [MicroProfile](../eclipse-microprofile/index.yml) などのフレームワークへの統合を調査することを検討してください。