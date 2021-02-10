---
title: Python 用 Azure ライブラリにログを構成する
description: Azure ライブラリでは、ライブラリごとまたは操作ごとに構成される標準の Python ログ モジュールが使用されます。
ms.date: 02/01/2021
ms.topic: conceptual
ms.custom: devx-track-python
ms.openlocfilehash: f42941ec54876fec5854a0a82cee1cbcf30506ce
ms.sourcegitcommit: b09d3aa79113af04a245b05cec2f810e43062152
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/02/2021
ms.locfileid: "99476448"
---
# <a name="configure-logging-in-the-azure-libraries-for-python"></a>Python 用 Azure ライブラリにログを構成する

[azure.core ページに基づく](azure-sdk-library-package-index.md#libraries-using-azurecore) Python 用 Azure ライブラリでは、標準の Python [ログ](https://docs.python.org/3/library/logging.html) ライブラリを使用してログ出力を提供します。

ログを操作する一般的なプロセスは次のとおりです。

1. 目的のライブラリのログ オブジェクトを取得し、ログ レベルを設定します。
1. ログ ストリームのハンドラーを登録します。
1. HTTP 情報を含めるには、`logging_enable=True` パラメーターをクライアント オブジェクト コンストラクター、資格情報オブジェクト コンストラクター、または特定のメソッドに渡します。

詳細については、この記事の残りのセクションで説明します。

原則として、ライブラリ内のログの使用状況を理解するための最適なリソースは、[github.com/Azure/azure-sdk-for-python](https://github.com/Azure/azure-sdk-for-python) で SDK ソース コードを参照することです。 次のセクションで提案しているように、必要に応じて詳細を簡単に検索できるように、このリポジトリをローカルに複製することをお勧めします。

## <a name="set-logging-levels"></a>ログ レベルの設定

```python
import logging

# ...

# Acquire the logger for a library (azure.storage.blob in this example)
logger = logging.getLogger('azure.storage.blob')

# Set the desired logging level
logger.setLevel(logging.DEBUG)
```

- この例では、`azure.storage.blob` ライブラリのロガーを取得してから、ログ レベルを `logging.DEBUG` に設定します。

- いつでも `logger.setLevel` を呼び出して、コードのさまざまなセグメントのログ レベルを変更できます。

別のライブラリのレベルを設定するには、`logging.getLogger` の呼び出しでそのライブラリの名前を使用します。 たとえば、azure-eventhubs ライブラリでは `azure.eventhubs` という名前のロガーが提供されており、azure-storage-queue ライブラリでは `azure.storage.queue` という名前のロガーが提供されているといった具合です。 (SDK ソース コードでは、含まれているモジュールの名前を使用してロガーを取得する、ステートメント `logging.getLogger(__name__)` が頻繁に使用されます。)

より一般的な名前空間を使用することもできます。 たとえば、次のように入力します。

```python
import logging

# Set the logging level for all azure-storage-* libraries
logger = logging.getLogger('azure.storage')
logger.setLevel(logging.INFO)

# Set the logging level for all azure-* libraries
logger = logging.getLogger('azure')
logger.setLevel(logging.ERROR)
```

`logger.isEnabledFor` メソッドを使用して、特定のログ レベルが有効になっているかどうかを確認できます。

```python
print(f"Logger enabled for ERROR={logger.isEnabledFor(logging.ERROR)}, " \
    f"WARNING={logger.isEnabledFor(logging.WARNING)}, " \
    f"INFO={logger.isEnabledFor(logging.INFO)}, " \
    f"DEBUG={logger.isEnabledFor(logging.DEBUG)}")
```

ログ レベルは、[標準ログ ライブラリ レベル](https://docs.python.org/3/library/logging.html#levels)と同じです。 次の表で、Python 用 Azure ライブラリでのこれらのログ レベルの一般的な使用方法について説明します。

| ログ記録レベル             | 一般的な使用方法 |
| ---                       | ---         |
| logging.ERROR             | アプリケーションが回復する可能性が低いエラー (メモリ不足など)。 |
| logging.WARNING (既定) | 関数が目的のタスクの実行に失敗する (ただし、REST API 呼び出しの再試行など、関数が回復できる場合は失敗しない)。 関数は、通常、例外発生時に警告をログに記録します。 警告レベルでは、エラー レベルが自動的に有効になります。 |
| logging.INFO              | 関数が正常に動作しているか、サービス呼び出しが取り消されました。 一般に、情報イベントには、要求、応答、ヘッダーが含まれます。 情報レベルでは、エラー レベルと警告レベルが自動的に有効になります。 |
| logging.DEBUG             | 通常、トラブルシューティングに使用され、例外のスタック トレースが含まれている詳細情報。 デバッグ レベルでは、情報、警告、エラー レベルが自動的に有効になります。 注意:`logging_enable=True` も設定した場合、デバッグ レベルにはヘッダー内のアカウント キーやその他の資格情報などの機密情報が含まれます。 セキュリティ侵害を避けるため、これらのログは必ず保護するようにしてください。 |
| logging.NOTSET            | すべてのログを無効にします。 |

### <a name="library-specific-logging-level-behavior"></a>ライブラリ固有のログ レベルの動作

各レベルでの正確なログ動作は、該当するライブラリによって異なります。 azure.eventhub などの一部のライブラリでは、広範なログが実行されますが、他のライブラリではほとんど実行されません。

ライブラリの正確なログを確認する最善の方法は、Azure SDK for Python のソース コードでログ レベルを検索することです。

1. リポジトリ フォルダーで、*sdk* フォルダーに移動し、目的の特定のサービスのフォルダーに移動します。

1. そのフォルダーで、次の文字列のいずれかを検索します。

    - `_LOGGER.error`
    - `_LOGGER.warning`
    - `_LOGGER.info`
    - `_LOGGER.debug`

## <a name="register-a-log-stream-handler"></a>ログ ストリーム ハンドラーを登録する

ログ出力をキャプチャするには、コードに少なくとも 1 つのログ ストリーム ハンドラーを登録する必要があります。

```python
import logging

# Direct logging output to stdout. Without adding a handler,
# no logging output is captured.
handler = logging.StreamHandler(stream=sys.stdout)
logger.addHandler(handler)
```

この例では、ログ出力を stdout に向けるハンドラーを登録します。 Python ドキュメントの「[logging.handlers](https://docs.python.org/3/library/logging.handlers.html)」で説明されているように他の種類のハンドラーを使用することも、標準の [logging.basicConfig](https://docs.python.org/3/library/logging.html#logging.basicConfig) メソッドを使用することもできます。

## <a name="enable-http-logging-for-a-client-object-or-operation"></a>クライアント オブジェクトまたは操作の HTTP ログを有効にする

既定では、Azure ライブラリ内のログには HTTP 情報は含まれません。 HTTP 情報を (DEBUG レベルとして) ログ出力に含めるには、`logging_enable=True` をクライアントまたは資格情報オブジェクト コンストラクター、あるいは特定のメソッドに明確に渡す必要があります。

**注意**:HTTP ログによって、ヘッダー内のアカウント キーやその他の資格情報などの機密情報が漏洩する場合があります。 セキュリティ侵害を避けるため、これらのログは必ず保護するようにしてください。

### <a name="enable-http-logging-for-a-client-object-debug-level"></a>クライアント オブジェクトの HTTP ログを有効にする (DEBUG レベル)

```python
from azure.storage.blob import BlobClient
from azure.identity import DefaultAzureCredential

# Enable HTTP logging on the client object when using DEBUG level
# endpoint is the Blob storage URL.
client = BlobClient(endpoint, DefaultAzureCredential(), logging_enable=True)
```

クライアント オブジェクトの HTTP ログを有効にすると、そのオブジェクトを通じて呼び出されたすべての操作のログが有効になります。

### <a name="enable-http-logging-for-a-credential-object-debug-level"></a>資格情報オブジェクトの HTTP ログを有効にする (DEBUG レベル)

```python
from azure.storage.blob import BlobClient
from azure.identity import DefaultAzureCredential

# Enable HTTP logging on the credential object when using DEBUG level
credential = DefaultAzureCredential(logging_enable=True)

# endpoint is the Blob storage URL.
client = BlobClient(endpoint, credential)
```

資格情報オブジェクトの HTTP ログを有効にすると、特にそのオブジェクトを通じて呼び出されたすべての操作のログが有効になります。ただし、認証を必要としないクライアント オブジェクトでの操作は除外されます。

### <a name="enable-logging-for-an-individual-method-debug-level"></a>個々のメソッドのログを有効にする (DEBUG レベル)

```python
from azure.storage.blob import BlobClient
from azure.identity import DefaultAzureCredential

# endpoint is the Blob storage URL.
client = BlobClient(endpoint, DefaultAzureCredential())

# Enable HTTP logging for only this operation when using DEBUG level
client.create_container("container01", logging_enable=True)
```

## <a name="example-logging-output"></a>ログ出力の例

次のコードは、[例: ストレージ アカウントの使用](azure-sdk-example-storage-use.md)で示されているものに、DEBUG と HTTP ログの有効化を追加したものです (簡潔にするためにコメントは省略しています)。

```python
import os, sys, logging
from azure.identity import DefaultAzureCredential
from azure.storage.blob import BlobClient

logger = logging.getLogger('azure.storage.blob')
logger.setLevel(logging.DEBUG)

handler = logging.StreamHandler(stream=sys.stdout)
logger.addHandler(handler)

credential = DefaultAzureCredential()
storage_url = os.environ["AZURE_STORAGE_BLOB_URL"]

blob_client = BlobClient(storage_url, container_name="blob-container-01",
    blob_name="sample-blob.txt", credential=credential)

with open("./sample-source.txt", "rb") as data:
    blob_client.upload_blob(data, logging_enable=True)
```

ログ出力は次のようになります。

<pre>
Request URL: 'https://pythonsdkstorage12345.blob.core.windows.net/blob-container-01/sample-blob.txt'
Request method: 'PUT'
Request headers:
    'Content-Type': 'application/octet-stream'
    'Content-Length': '79'
    'x-ms-version': '2019-07-07'
    'x-ms-blob-type': 'BlockBlob'
    'If-None-Match': '*'
    'x-ms-date': 'Mon, 01 Jun 2020 22:54:14 GMT'
    'x-ms-client-request-id': 'd081f88e-a45a-11ea-b9eb-0c5415dfd03a'
    'User-Agent': 'azsdk-python-storage-blob/12.3.1 Python/3.8.3 (Windows-10-10.0.18362-SP0)'
    'Authorization': '*****'
Request body:
b"Hello there, Azure Storage. I'm a friendly file ready to be stored in a blob.\r\n"
Response status: 201
Response headers:
    'Content-Length': '0'
    'Content-MD5': 'kvMIzjEi6O8EqTVnZJNakQ=='
    'Last-Modified': 'Mon, 01 Jun 2020 22:54:14 GMT'
    'ETag': '"0x8D8067EB52FF7BC"'
    'Server': 'Windows-Azure-Blob/1.0 Microsoft-HTTPAPI/2.0'
    'x-ms-request-id': '5df479b1-f01e-00d0-5b67-382916000000'
    'x-ms-client-request-id': 'd081f88e-a45a-11ea-b9eb-0c5415dfd03a'
    'x-ms-version': '2019-07-07'
    'x-ms-content-crc64': 'QmecNePSHnY='
    'x-ms-request-server-encrypted': 'true'
    'Date': 'Mon, 01 Jun 2020 22:54:14 GMT'
Response content:
</pre>
