---
title: Azure SDK for Java でログを構成する
description: ログに関する Azure SDK for Java の概念の概要
author: srnagar
ms.date: 02/02/2021
ms.topic: conceptual
ms.custom: devx-track-java
ms.author: srnagar
ms.openlocfilehash: bbb9733a21eabf5d13a698a0785908dcd997af45
ms.sourcegitcommit: 71847ee0a1fee3f3320503629d9a8c82319a1f6a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/03/2021
ms.locfileid: "99528486"
---
# <a name="configure-logging-in-the-azure-sdk-for-java"></a>Azure SDK for Java でログを構成する

この記事では、Azure SDK for Java を利用するアプリケーションでログを有効にする方法の概要について説明します。 Java 用 Azure クライアント ライブラリには、2 つのログ オプションがあります。

* 一時的なデバッグ用の組み込みのログ記録フレームワーク。
* [SLF4J](https://www.slf4j.org/) インターフェイスを使用したログのサポート。

SLF4J は Java エコシステムでよく知られており、詳しく文書化されているので、これを使用することをお勧めします。 詳細については、[SLF4J のユーザー マニュアル](https://www.slf4j.org/manual.html)を参照してください。

この記事は、広く使用されている Java ログ記録フレームワークの多くを対象とする他の記事にリンクしています。 それらの他の記事では、構成の例を紹介し、Azure クライアント ライブラリでのログ記録フレームワークの使用方法について説明しています。

どのようなログ構成を使用しても、Java 用 Azure クライアント ライブラリのすべてのログ出力は azure-core `ClientLogger` 抽象化を使用してルーティングされるため、いずれにしても同じログ出力を利用できます。

この記事の残りの部分で、使用可能なすべてのログ オプションの構成について詳しく説明します。

## <a name="default-logger-for-temporary-debugging"></a>既定のロガー (一時的なデバッグ用)

前述のように、すべての Azure クライアント ライブラリでは SLF4J がログに使用されます。ただし、Java 用 Azure クライアント ライブラリに組み込まれた既定のロガーというフォールバックがあります。 この既定のロガーは、アプリケーションがデプロイ済みで、ログが必要だが、SLF4J ロガーを組み込んだアプリケーションを再デプロイできない場合のために提供されています。 このロガーを有効にするには、まず SLF4J ロガーが存在しないことを確認してから (これが優先されるため)、`AZURE_LOG_LEVEL` 環境変数を設定します。 次の表は、この環境変数に使用できる値を示しています。

| ログ レベル              | 使用可能な環境変数の値     |
|------------------------|-----------------------------------------|
| 詳細                | "verbose"、"debug"                      |
| 情報          | "info"、"information"、"informational"  |
| WARNING                | "warn"、"warning"                       |
| ERROR                  | "err"、"error"                          |

環境変数を設定した後、アプリケーションを再起動してこの環境変数を有効にします。 このロガーはコンソールにログします。また、ロールオーバーやファイルへのログなど、SLF4J 実装の高度なカスタマイズ機能を提供しません。 ログを再度無効にするには、この環境変数を削除し、アプリケーションを再起動するだけです。

## <a name="slf4j-logging"></a>SLF4J のログ

既定では、SLF4J でサポートされているログ記録フレームワークを使用してログを構成する必要があります。 まず、プロジェクトから、関連する SLF4J ログ実装を依存関係として含めます。 詳細については、SLF4J ユーザー マニュアルの「[ログに対するプロジェクトの依存関係の宣言](http://www.slf4j.org/manual.html#projectDep)」を参照してください。 次に、ログ レベルの設定、ログするクラスとログしないクラスの構成など、環境内で必要に応じて動作するようにロガーを構成します。 下のリンクでいくつかの例が示されていますが、詳細については、選択したログ記録フレームワークのドキュメントを参照してください。

## <a name="next-steps"></a>次のステップ

Azure SDK for Java でのログのしくみを確認したので、SLF4J と Java クライアント ライブラリを操作するために、より広く使用されている Java ログ記録フレームワークのいくつかを構成する方法のガイダンスについて、下のリンクを確認することを検討してください。

* [java.util.logging](logging-jul.md)
* [Logback](logging-logback.md)
* [Log4J](logging-log4j.md)
