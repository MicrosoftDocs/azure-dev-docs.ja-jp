---
title: Azure SDK for Java と java.util.logging によるログ
description: Azure SDK for Java と java.util.logging の統合の概要
author: srnagar
ms.date: 02/02/2021
ms.topic: conceptual
ms.custom: devx-track-java
ms.author: srnagar
ms.openlocfilehash: ca3a431debec21bad2099371e711c1df73d9b344
ms.sourcegitcommit: 576c878c338d286060010646b96f3ad0fdbcb814
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102118584"
---
# <a name="log-with-the-azure-sdk-for-java-and-javautillogging"></a>Azure SDK for Java と java.util.logging によるログ

この記事では、java.util.logging を使用したログ記録を、Azure SDK for Java を使用するアプリケーションに追加する方法の概要について説明します。 java.util.logging フレームワークは JDK の一部です。 「[Azure SDK for Java でログ記録を構成する](logging-overview.md)」で説明したように、すべての Azure クライアント ライブラリが [SLF4J](http://www.slf4j.org/) を通じてログ記録するため、[java.util.logging](https://docs.oracle.com/javase/8/docs/api/java/util/logging/Logger.html) などのログ記録フレームワークを使用できます。

java.util.logging を有効にするには、次の 2 つのことを行う必要があります。

1. java.util.logging 用 SLF4J アダプターを依存関係として含める
2. *logging.properties* という名前のファイルを */src/main/resources* プロジェクト ディレクトリの下に作成する

ロガーの構成に関する詳細については、Oracle ドキュメントの「[ログ記録出力の構成](https://docs.oracle.com/cd/E23549_01/doc.1111/e14568/handler.htm)」を参照してください。

## <a name="add-the-maven-dependency"></a>Maven の依存関係を追加する

Maven の依存関係を追加するには、プロジェクトの *pom.xml* ファイルに次の XML を含めます。 バージョン番号 *1.7.30* は、[SLF4J JDK14 Binding ページ](https://mvnrepository.com/artifact/org.slf4j/slf4j-jdk14)に示されている最新リリースのバージョン番号に置き換えてください。

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-jdk14</artifactId>
    <version>1.7.30</version> <!-- replace this version with the latest available version on Maven central -->
</dependency>
```

## <a name="add-loggingproperties-to-your-project"></a>プロジェクトへの logging.properties の追加

`java.util.logging` を使用してログ記録するには、*logging.properties* という名前のファイルをプロジェクトの *./src/main/resources* ディレクトリの下に作成します。 このファイルには、ログ記録のニーズをカスタマイズするためのログ記録構成が含まれます。 詳細については、「[Java のログ記録:構成](http://tutorials.jenkov.com/java-logging/configuration.html) を選択します。

*logging.properties* とは別のファイル名を使用したい場合は、`java.util.logging.config.file` システム プロパティを設定してください。 このプロパティは、ロガー インスタンスの作成前に設定する必要があります。

### <a name="console-logging"></a>[コンソールのログ記録]

コンソールにログ記録する構成を作成する例を、次に示します。 この例は、INFO レベル以上であればすべてのレベルのログ イベントがログ記録されるよう構成されています。

```properties
handlers = java.util.logging.ConsoleHandler
.level = INFO

java.util.logging.ConsoleHandler.level = INFO
java.util.logging.ConsoleHandler.formatter = java.util.logging.SimpleFormatter
java.util.logging.SimpleFormatter.format=[%1$tF %1$tT] [%4$s] %5$s %n
```

### <a name="log-to-a-file"></a>ファイルにログ記録する

前の例でログを記録しているコンソールは、通常はログ記録する場所として好ましくありません。 代わりにファイルへのログ記録を構成するには、次の構成を使用します。

```properties
handlers = java.util.logging.FileHandler
.level = INFO

java.util.logging.FileHandler.pattern = %h/myapplication.log
java.util.logging.FileHandler.formatter = java.util.logging.SimpleFormatter
java.util.logging.FileHandler.level = INFO
```

このコードにより、*myapplication.log* という名前のファイルがホーム ディレクトリ (`%h`) に作成されます。 このロガーでは、一定期間後のファイルの自動ローテーションがサポートされていません。 この機能が必要な場合は、ログ ファイル ローテーションを管理するためのスケジューラを作成する必要があります。

## <a name="next-steps"></a>次のステップ

この記事では、`java.util.logging` の構成と、Azure SDK for Java でこれをログに使用する方法について説明しました。 Azure SDK for Java はすべての SLF4J ログ記録フレームワークで機能するため、「[SLF4J ユーザー マニュアル](http://www.slf4j.org/manual.html)」で詳細を参照してください。

ログ記録について習得した後は、Azure が提供する、[Spring](../spring-framework/spring-boot-starters-for-azure.md) や [MicroProfile](../eclipse-microprofile/index.yml) などのフレームワークへの統合を検討してください。