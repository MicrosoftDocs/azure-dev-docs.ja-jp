---
title: Azure SDK for Java と Log4j によるログ
description: Azure SDK for Java と log4j の統合の概要
author: srnagar
ms.date: 02/02/2021
ms.topic: conceptual
ms.custom: devx-track-java
ms.author: srnagar
ms.openlocfilehash: 0de76274b7f33f724c339eb0137a89d74f3e8678
ms.sourcegitcommit: 576c878c338d286060010646b96f3ad0fdbcb814
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102118474"
---
# <a name="log-with-the-azure-sdk-for-java-and-log4j"></a>Azure SDK for Java と Log4j によるログ

この記事では、Log4j を使用したログを、Azure SDK for Java を使用するアプリケーションに追加する方法の概要について説明します。 「[Azure SDK for Java でログを構成する](logging-overview.md)」で説明したように、すべての Azure クライアント ライブラリが [SLF4J](http://www.slf4j.org/) を通じてログするため、[log4j](https://logging.apache.org/log4j/2.x/) などのログ記録フレームワークを使用できます。

この記事では Log4J 2.x リリースを使用する場合について説明しますが、Azure SDK for Java では Log4J 1.x が同じようにサポートされています。 Log4j のログを有効にするには、次の 2 つのことを行う必要があります。

1. log4j ライブラリを依存関係として含める。
2. 構成ファイル (*log4j2.properties* または *log4j2.xml*) を */src/main/resources* プロジェクト ディレクトリの下に作成する。

log4j の構成に関する詳細については、「[Log4j 2 へようこそ](https://logging.apache.org/log4j/2.x/manual/index.html)」を参照してください。

## <a name="add-the-maven-dependency"></a>Maven の依存関係を追加する

Maven の依存関係を追加するには、プロジェクトの *pom.xml* ファイルに次の XML を含めます。 バージョン番号 *2.14.0* は、[Apache Log4j SLF4J Binding ページ](https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-slf4j-impl)に示されている最新リリースのバージョン番号に置き換えてください。

```xml
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-slf4j-impl</artifactId>
    <version>2.14.0</version>
</dependency>
```

## <a name="configuring-log4j"></a>Log4j を構成する

Log4j を構成する一般的な方法として、外部プロパティ ファイルを使用するものと、外部 XML ファイルを使用するものの 2 つがあります。 これらの方法を以下で説明します。

### <a name="using-a-property-file"></a>プロパティ ファイルを使用する

*log4j2.properties* という名前のフラット プロパティ ファイルを、プロジェクトの */src/main/resource* ディレクトリに置きます。 このファイルの構造は、次のようになります。

```properties
appender.console.type = Console
appender.console.name = STDOUT
appender.console.layout.type = PatternLayout
appender.console.layout.pattern = %msg%n

logger.app.name = com.azure.core
logger.app.level = ERROR

rootLogger.level = info
rootLogger.appenderRefs = stdout
rootLogger.appenderRef.stdout.ref = STDOUT
```

### <a name="using-an-xml-file"></a>XML ファイルを使用する

*log4j2.xml* という名前の XML ファイルを、プロジェクトの */src/main/resource* ディレクトリに置きます。 このファイルの構造は、次のようになります。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="INFO">
    <Appenders>
        <Console name="console" target="SYSTEM_OUT">
            <PatternLayout pattern="%msg%n" />
        </Console>
    </Appenders>
    <Loggers>
        <Logger name="com.azure.core" level="error" additivity="true">
            <appender-ref ref="console" />
        </Logger>
        <Root level="info" additivity="false">
            <appender-ref ref="console" />
        </Root>
     </Loggers>
</Configuration>
```

## <a name="next-steps"></a>次のステップ

この記事では、Log4j の構成と、Azure SDK for Java でこれをログに使用する方法について説明しました。 Azure SDK for Java はすべての SLF4J ログ記録フレームワークで動作するため、「[SLF4J ユーザー マニュアル](http://www.slf4j.org/manual.html)」で詳細を参照してください。 Log4j を使用する場合は、その Web サイトに数多くの構成ガイダンスがあります。 詳細については、「[Log4j 2 へようこそ](https://logging.apache.org/log4j/2.x/manual/index.html)」をご覧ください。

ログ記録について習得した後は、Azure が提供する、[Spring](../spring-framework/spring-boot-starters-for-azure.md) や [MicroProfile](../eclipse-microprofile/index.yml) などのフレームワークへの統合を検討してください。