---
title: Azure SDK for Java 使用時のネットワーク問題をトラブルシューティングする
description: Azure SDK for Java の使用に関連したネットワーク問題をトラブルシューティングする方法の概要
author: alzimmermsft
ms.date: 02/02/2021
ms.topic: conceptual
ms.custom: devx-track-java
ms.author: alzimmer
ms.openlocfilehash: 63852f253a648e473ba91ac361bc5d9d0629b8f1
ms.sourcegitcommit: 71847ee0a1fee3f3320503629d9a8c82319a1f6a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/03/2021
ms.locfileid: "99528455"
---
# <a name="troubleshoot-networking-issues"></a>ネットワークに関する問題をトラブルシューティングする

この記事では、さまざまな複雑さのネットワーク問題を診断できるいくつかのツールについて説明します。 これらの問題には、サービスからの予期しない応答値のトラブルシューティングから、Connection-closed (接続が閉じられた) 例外の根本原因を突き止めることまで、さまざまなシナリオが含まれます。

クライアント側のトラブルシューティングのために、Java 用 Azure クライアント ライブラリには、「[Azure SDK for Java でログを構成する](logging-overview.md)」で説明されているように、一貫性のある堅牢なログ記録のストーリーが用意されています。 ただし、クライアント ライブラリでは、さまざまなプロトコルでネットワーク呼び出しが行われるため、指定されたトラブルシューティングの範囲外に及ぶトラブルシューティング シナリオが発生する場合があります。 そのような問題が発生した場合、解決策は、この記事で説明されている外部ツールを使用してネットワーク問題を診断することです。

## <a name="fiddler"></a>Fiddler

[Fiddler](https://docs.telerik.com/fiddler-everywhere/introduction) は HTTP デバッグ プロキシです。これを使用すると、それを介して渡された要求および応答を現状のままログに記録できます。 キャプチャされた未加工の要求と応答は、予期しない要求がサービスで取得されたり、予期しない応答がクライアントで受信されたりするシナリオのトラブルシューティングに役立ちます。 Fiddler を使用するには、HTTP プロキシを使用してクライアント ライブラリを構成する必要があります。 HTTPS を使用する場合は、暗号化解除された要求と応答の本文を検査するために追加の構成が必要です。

### <a name="add-an-http-proxy"></a>HTTP プロキシを追加する

HTTP プロキシを追加するには、[Azure SDK for Java でのプロキシの構成](proxying.md)に関する記事のガイダンスに従ってください。 必ず、ポート 8888 で、Fiddler の既定アドレスである `localhost` を使用してください。

### <a name="enable-https-decryption"></a>HTTPS の暗号化解除を有効にする

既定では、Fiddler では HTTP トラフィックのみをキャプチャできます。 アプリケーションで HTTPS が使用される場合は、HTTPS トラフィックをキャプチャできるように、Fiddler の証明書を信頼するための追加手順を実行する必要があります。 詳細については、Fiddler のドキュメント内の [HTTPS メニュー](https://docs.telerik.com/fiddler-everywhere/user-guide/settings/https)に関する記事を参照してください。

次の手順では、Java Runtime Environment (JRE) を使用して証明書を信頼する方法について説明します。 証明書を信頼しないと、セキュリティ警告が出されて、Fiddler 経由の HTTPS 要求が失敗する場合があります。

**Linux または macOS**

1. Fiddler の証明書をエクスポートします。
1. JRE の keytool (通常、`jre/bin`) を見つけます。
1. JRE の cacert (通常、`jre/lib/security`) を見つけます。
1. Bash ウィンドウを開き、次のコマンドを実行して証明書をインポートします。

   ```bash
   sudo keytool -import -file <location of Fiddler certificate> -keystore <location of cacert> -alias Fiddler
   ```

1. パスワードを入力します。
1. 証明書を信頼します。

**Windows**

1. Fiddler の証明書をエクスポートします。
1. JRE の keytool (通常、`jre\bin`) を見つけます。
1. JRE の cacert (通常、`jre\lib\security`) を見つけます。
1. コマンド プロンプトを開き、次のコマンドを実行して証明書をインポートします。

   ```cmd
   keytool.exe -import -file <location of Fiddler certificate> -keystore <location of cacert> -alias Fiddler
   ```

1. パスワードを入力します。
1. 証明書を信頼します。

## <a name="wireshark"></a>Wireshark

[Wireshark](https://www.wireshark.org/) は、アプリケーション コードを変更することなくネットワーク トラフィックをキャプチャできるネットワーク プロトコル アナライザーです。 Wireshark は高度な構成が可能で、広範から、特定で低レベルのネットワーク トラフィックまでをキャプチャできます。 この機能は、リモート ホストによって接続が閉じられる、または操作中に接続が閉じられるといったシナリオのトラブルシューティングに役立ちます。 Wireshark GUI では、TCP 再送信や RST などの一意のキャプチャ ケースを特定する配色を使用してキャプチャが表示されます。 また、キャプチャ時または分析中にキャプチャをフィルター処理することもできます。

### <a name="configure-a-capture-filter"></a>キャプチャ フィルターを構成する

キャプチャ フィルターを使用すると、分析のためにキャプチャされるネットワーク呼び出しの数を減らすことができます。 キャプチャ フィルターを使用しない場合、Wireshark ではネットワーク インターフェイスを経由するすべてのトラフィックがキャプチャされます。 この動作によって、そのほとんどが調査にとってノイズである大量のデータが生成される可能性があります。 キャプチャ フィルターを使用すると、キャプチャするネットワーク トラフィックの範囲を事前に設定して、調査対象を絞り込むことができます。 詳細については、Wireshark のドキュメント内の[ライブ ネットワーク データのキャプチャ](https://www.wireshark.org/docs/wsug_html_chunked/ChapterCapture.html)に関する記事を参照してください。

次の例では、特定のホストとの間で送受信されるネットワーク トラフィックをキャプチャするためのキャプチャ フィルターを追加します。

Wireshark で、 **[Capture]\(キャプチャ\) > [Capture Filters...]\(キャプチャ フィルタ...\)** に移動し、`host <host IP or hostname>` の値で新しいフィルターを追加します。 このフィルターでは、そのホストとの間のトラフィックのみがキャプチャされます。 アプリケーションが複数のホストと通信する場合は、複数のキャプチャ フィルターを追加できます。または、"OR" 演算子を使用してホスト IP またはホスト名を追加し、より粗いキャプチャ フィルター処理を提供することもできます。

### <a name="capture-to-disk"></a>ディスクにキャプチャする

アプリケーションを長時間実行して、予期しないネットワーク例外を再現し、それに至るまでのトラフィックを確認する必要が生じる場合があります。 また、メモリ内にすべてのキャプチャを維持できない場合もあります。 幸いなことに、Wireshark では、後処理で使用できるように、キャプチャをディスクに記録することができます。 この方法を使用すると、問題を再現している間にメモリ不足が発生するリスクを回避できます。 詳細については、Wireshark のドキュメント内の[ファイルの入力、出力、および印刷](https://www.wireshark.org/docs/wsug_html_chunked/ChapterIO.html)に関する記事を参照してください。

次の例では、複数のファイルを含むディスクにキャプチャを永続化するように Wireshark を設定します。この場合、ファイルは 10 万キャプチャまたは 50 MB のサイズで分割されます。

Wireshark で、 **[Capture]\(キャプチャ\) > [Options]\(オプション\)** に移動し、 **[Output]\(出力\)** タブを見つけ、使用するファイル名を入力します。 この構成では、Wireshark によってキャプチャは 1 つのファイルに保持されます。 複数のファイルへのキャプチャを有効にするには、 **[Create a new file automatically]\(自動的に新しいファイルを作成する\)** を選択し、次に、 **[after 100000 packets]\(100000 パケット後\)** と **[after 50 megabytes]\(50 メガバイト後\)** を選択します。 この構成では、いずれかの述語が一致したときに、Wireshark によって新しいファイルが作成されます。 新しい各ファイルでは、入力されたファイル名と同じベース名が使用され、一意の識別子が追加されます。 Wireshark で作成できるファイルの数を制限する場合は、 **[Use a ring buffer with X files]\(X ファイルでリング バッファーを使用する\)** を選択します。 このオプションでは、指定された数のファイルのみでログ記録を行うように、Wireshark が制限されます。 そのファイル数に達すると、Wireshark では最も古いファイルから、ファイルの上書きが開始されます。

### <a name="filter-captures"></a>キャプチャをフィルター処理する

場合によっては、Wireshark によってキャプチャされるトラフィックの範囲を厳密に設定できないことがあります。たとえば、アプリケーションがさまざまなプロトコルを使用して複数のホストと通信している場合などです。 このシナリオでは、一般に上記で説明した永続的なキャプチャを使用すると、ネットワーク キャプチャ後の分析を簡単に実行できます。 Wireshark では、キャプチャを分析するためのフィルターのような構文がサポートされています。 詳細については、Wireshark のドキュメント内の[キャプチャしたパケットの操作](https://www.wireshark.org/docs/wsug_html_chunked/ChapterWork.html)に関する記事を参照してください。

次の例では、永続化されたキャプチャ ファイルを読み込み、`ip.src_host==<IP>` でフィルター処理を行います。

Wireshark で、 **[File]\(ファイル\) > [Open]\(開く\)** に移動し、前に使用されたファイルの場所から、永続化されたキャプチャを読み込みます。 ファイルがメニュー バーの下に読み込まれると、フィルター入力が表示されます。 フィルター入力に、`ip.src_host==<IP>` と入力します。 このフィルターでは、ソースが IP `<IP>` のホストからのキャプチャだけが表示されるように、キャプチャ ビューが制限されます。

## <a name="next-steps"></a>次のステップ

このトピックでは、Azure SDK for Java の使用時に発生するネットワーク問題を診断するためのさまざまなツールの使用について説明しました。 使用シナリオに関する概念を理解したので、SDK 自体の探索を開始できます。 使用可能な API の詳細については、[Azure SDK for Java ライブラリ](azure-sdk-library-package-index.md)に関するページを参照してください。
