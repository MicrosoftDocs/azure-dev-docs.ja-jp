---
title: VSCode を使用してイメージを Blob Storage にアップロードする - App Service/Cosmos DB
description: React/TypeScript アプリを使用して、ファイルを Azure Storage Blob にアップロードします。 このチュートリアルでは、Visual Studio Code の拡張機能でローカルとリモートの環境を使用する方法について説明します。
ms.topic: tutorial
ms.date: 12/17/2020
ms.custom: scenarios:getting-started, languages:JavaScript, devx-track-javascript, azure-sdk-storage-blob-typescript-version-12.2.1
ms.openlocfilehash: 116ca6d03703e96631441a5e130d948065c5c5f1
ms.sourcegitcommit: 576c878c338d286060010646b96f3ad0fdbcb814
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102118544"
---
# <a name="upload-an-image-to-an-azure-storage-blob"></a>イメージを Azure Storage Blob にアップロードする

クライアント側の React アプリを使用して、Azure Storage npm パッケージを使用してイメージ ファイルを Azure Storage Blob にアップロードします。 

TypeScript のプログラミングは完了しており、このチュートリアルでは、Azure 拡張機能を使用して Visual Studio Code 内からローカルとリモートの Azure 環境を正常に使用することに焦点を当てます。

* [**サンプル コード**](https://github.com/Azure-Samples/js-e2e-browser-file-upload-storage-blob)

## <a name="application-architecture-and-functionality"></a>アプリケーションのアーキテクチャと機能

このチュートリアルには、JavaScript 開発者向けの上位の Azure タスクがいくつか含まれています。

* Visual Studio Code を使用してローカル環境で React/TypeScript アプリを実行する
* **Azure Storage Blob** リソースを作成し、ファイル アップロード用に構成する
    * CORS を構成する
    * Shared Access Signature (SAS) トークンを作成する
* サービスに対する認証に SAS トークンを使用するために、Azure SDK クライアント ライブラリ用にコードを構成する

[GitHub で入手できる](https://github.com/Azure-Samples/js-e2e-browser-file-upload-storage-blob)サンプル React アプリは、次の要素で構成されています。

* ポート 3000 でホストされている **React アプリ**
* Storage BLOB にアップロードするための **Azure SDK クライアント ライブラリ スクリプト**

:::image type="content" source="../media/tutorial-browser-file-upload/browser-react-app-azure-storage-resource-image-uploaded-displayed.png" alt-text="Azure Storage BLOB に接続された単純な React アプリ。":::

## <a name="1-set-up-development-environment"></a>1.開発環境をセットアップする

- アクティブなサブスクリプションが含まれる Azure アカウント。 [無料で作成できます](https://azure.microsoft.com/free/)。
- [Node.js 10 と npm](https://nodejs.org/en/download) (ローカル コンピューターにインストールされている Node.js パッケージ マネージャー)。
- ローカル コンピューターにインストールされている [Visual Studio Code](https://code.visualstudio.com/)。 
- Visual Studio Code の拡張機能:
    - [Azure Storage](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurestorage) - Storage リソースを表示するために使用されます

## <a name="2-clone-and-run-the-initial-react-app"></a>2.最初の React アプリを複製して実行する

1. Visual Studio Code を開きます。
1. Git アイコンを選択し、次に **[リポジトリの複製]** を選択して、GitHub リポジトリを複製します。 このプロセスでは、GitHub にログインする必要があります。 次のようにリポジトリの URL を使用します。 

    `https://github.com/Azure-Samples/js-e2e-browser-file-upload-storage-blob`

1. サンプルの複製先となるローカル コンピューター上のフォルダーを選択します。 メッセージが表示されたら、複製されたリポジトリを開きます。 

    :::image type="content" source="../media/tutorial-browser-file-upload/vscode-git-clone-repository.png" alt-text="Git アイコンを選択し、次に [リポジトリの複製] を選択して、GitHub リポジトリを複製します。":::

1. Visual Studio Code で、ターミナル ウィンドウを開き、次のコマンドを実行して、サンプルの依存関係をインストールします。

    ```javascript
    npm install
    ```

1. 同じターミナル ウィンドウで、Web アプリを実行するコマンドを実行します。

    ```javascript
    npm start
    ```

1. Web ブラウザーを開き、次の URL を使用して、ローカル コンピューターで Web アプリを表示します。

    ```url
    http://localhost:3000/
    ```

    Storage が構成されていないというテキストを含む単純な Web アプリがブラウザーに表示されたら、チュートリアルのこのセクションは成功しています。

    :::image type="content" source="../media/tutorial-browser-file-upload/browser-react-app-no-azure-storage-resource-configured.png" alt-text="MongoDB データベースに接続された単純な Node.js アプリ。":::

1. Visual Studio Code ターミナルで、Ctrl + C を使用してコードを停止します。

## <a name="3-create-storage-resource-with-visual-studio-extension"></a>3.Visual Studio 拡張機能を使用して Storage リソースを作成する

1. Azure Storage 拡張機能に移動します。 サブスクリプションを右クリックし、[`Create Storage Account...`] を選択します。

    :::image type="content" source="../media/tutorial-browser-file-upload/visualstudiocode-storage-extension-create-resource.png" alt-text="Azure Storage 拡張機能に移動します。サブスクリプションを右クリックし、[ストレージ アカウントの作成] を選択します。":::

1. Storage リソースの作成方法を理解するために次の表を使用しながら、プロンプトに従います。

    |プロパティ|値|
    |--|--|
    |新しい Web アプリのグローバルに一意の名前を入力します。| Storage リソース名として、`fileuploaddemo` などの値を入力します。<br><br> この一意の名前は、次のセクションで使用する **ご自身のリソース名** です。文字と数字のみを使用します。長さは 24 文字までです。 この **アカウント名** は、後で使用するために必要です。|

1. アプリの作成プロセスが完了すると、新しいリソースに関する情報を含む通知が表示されます。 

    :::image type="content" source="../media/tutorial-browser-file-upload/visualstudiocode-storage-extension-create-resource-complete.png" alt-text="アプリの作成プロセスが完了すると、新しいリソースに関する情報を含む通知が表示されます。":::

## <a name="4-set-storage-account-name-in-code-file"></a>4.コード ファイルにストレージ アカウント名を設定する

`src/azure-storage-blob.ts` に前の手順からのリソース名を設定します。 

```typescript
const storageAccountName = process.env.storageresourcename || "fileuploaddemo"; 
```

## <a name="5-generate-your-shared-access-signature-sas-token"></a>5.Shared Access Signature (SAS) トークンを生成する 

CORS を構成する前に、SAS トークンを生成してください。 

1. Storage 用の Visual Studio Code 拡張機能で、リソースを右クリックし、 **[ポータルで開く]** を選択します。 これにより、正しい Storage リソースが Azure portal で開きます。
1. **[設定]** セクションで、 **[Shared Access Signature]** を選択します。 
1. 次の設定を使用して、SAS トークンを構成します。 

    | プロパティ|値|
    |--|--|
    |使用できるサービス|BLOB|
    |許可されるリソースの種類|サービス、コンテナー、オブジェクト|
    |許可されるアクセス許可|読み取り、書き込み、削除、一覧表示、追加、作成|
    |バージョンの削除を有効にする|オン|
    |開始日時と終了日時|開始日時を受け入れ、**終了日時を 24 時間後に設定します**。 SAS トークンは 24 時間のみ有効です。|
    |HTTPS のみ|オン|
    |優先ルーティング レベル|Basic|
    |署名キー|key1 が選択済み|

    :::image type="content" source="../media/tutorial-browser-file-upload/azure-portal-storage-blob-generate-sas-token.png" alt-text="図に示されているように SAS トークンを構成します。図の下に設定に関する説明があります。":::

1. **[SAS と接続文字列を生成する]** を選択します。 SAS トークンを直ちにコピーします。 このトークンを一覧表示することはできません。したがって、コピーしていない場合は、新しい SAS トークンの生成が必要になります。 

## <a name="set-sas-token-in-code-file"></a>コード ファイルに SAS トークンを設定する

SAS トークンは、クラウドベースのリソースに対してクエリを実行するときに使用されます。

1. トークンが疑問符で始まる場合は、`?` を削除します。 `?` は、コード ファイルによって自動的に提供されるため、トークン内には必要ありません。

1. `src/azure-storage-blob.ts` で、SAS トークンを空の文字列に追加して、sasToken 値に対して SAS トークンを設定します。 コードの残りの部分はそのままにします。 

```typescript
// remove `?` if it is first character of token
const sasToken = process.env.storagesastoken || "";
```

## <a name="6-configure-cors-for-azure-storage-resource"></a>6.Azure Storage リソースの CORS を構成する

クライアント側の React コードがストレージ アカウントにアクセスできるように、リソースに対して CORS を構成します。 

1. 引き続き Azure portal から、[設定] セクションで **[CORS]** を選択します。 
1. 図に示されているように CORS を構成します。 この設定の説明は、図の下にあります。 

    | プロパティ|値|
    |--|--|
    |許可されるオリジン|`*`|
    |許可されたメソッド|パッチを除くすべて。|
    |許可されるヘッダー|`*`|
    |公開されるヘッダー|`*`|
    |最長有効期間|86400|

    :::image type="content" source="../media/tutorial-browser-file-upload/azure-portal-storage-blob-cors.png" alt-text="図に示されているように CORS を構成します。図の下に設定に関する説明があります。":::

1. 設定の上の **[保存]** を選択すると、リソースに保存されます。 これらの CORS 設定に合わせるために、コードを変更する必要はありません。 

## <a name="7-run-project-locally-to-verify-connection-to-storage-account"></a>7.ストレージ アカウントへの接続を確認するためにプロジェクトをローカルで実行する

SAS トークンとストレージ アカウント名を `src/azure-storage-blob.ts` ファイルに設定したので、アプリケーションを実行する準備ができました。

1. アプリが実行されていない場合は、再度起動します。

    ```javascript
    npm start
    ```

1. ブラウザーで次の URL を開きます。

    `http://localhost:3000` 

    :::image type="content" source="../media/tutorial-browser-file-upload/browser-react-app-azure-storage-resource-configured-upload-button-displayed.png" alt-text="Azure Storage BLOB に接続されている React Web サイトには、[ファイルの選択] ボタンと [ファイルのアップロード] ボタンが表示されます。":::

1. `images` フォルダーから、アップロードするイメージを選択し、 **[アップロード]** を選択します。 を追加します。 

    React フロントエンド クライアント コードによって、`src/azure-storage-blob.ts` が呼び出されて Azure に対する認証が行われ、ストレージ コンテナーが作成され (まだ存在しない場合)、そのコンテナーにファイルがアップロードされます。 

    これで、チュートリアルの手順は完了です。 残りのセクションでは、アプリについて説明します。問題の解決にお役立てください。

## <a name="troubleshoot-local-connection-to-storage-account"></a>ストレージ アカウントへのローカル接続をトラブルシューティングする

エラーが発生したか、ファイルがコンテナーにアップロードされない場合は、次のことを確認してください。

* SAS トークンを再作成して、トークンがコンテナー レベルではなくストレージ リソース レベルで作成されていることを確認します。 コードの正しい場所に新しいトークンをコピーします。
* コードにコピーしたトークン文字列の先頭に `?` (疑問符) が含まれていないことを確認します。
* ストレージ リソースの CORS 設定を確認します。

## <a name="upload-button-functionality"></a>[アップロード] ボタンの機能

create-react-app でのそのアプリ作成の一部として、`src/App.tsx` TypeScript ファイルが提供されています。 このファイルは、[ファイルの選択] ボタンと [アップロード] ボタン、およびその機能を提供するためのサポート コードを提供するように変更されています。 

Azure Blob Storage コードに接続するコードは強調表示されています。 `uploadFileToBlob` を呼び出すと、コンテナー内のすべての BLOB (ファイル) がフラット リストとして返されます。 そのリストは、`DisplayImagesFromContainer` 関数を使用して表示されます。

:::code language="typescript" source="~/../js-e2e-browser-file-upload-storage-blob/src/App.tsx" highlight="3,28":::

## <a name="upload-file-to-azure-storage-blob-with-azure-sdk-client-library"></a>Azure SDK クライアント ライブラリを使用して Azure Storage BLOB にファイルをアップロードする

ファイルを Azure Storage にアップロードするコードは、フレームワークに依存しません。 コードはチュートリアル用に構築されているため、単純でわかりやすくなるように選択が行われました。 これらの選択について説明しています。意図的な使用、セキュリティ、および効率性については、独自のプロジェクトを確認してください。 

このサンプルでは、パブリックにアクセスできるコンテナーとファイルを作成して使用します。 独自のプロジェクト内のファイルをセキュリティで保護する場合は、リソースへの全体的な認証を要求することから、各 BLOB オブジェクトに対する非常に限定的なアクセス許可までを制御できる多くのレイヤーがあります。 

### <a name="dependencies-and-variables"></a>依存関係と変数

`azure-storage-blob.ts` TypeScript ファイルにより、依存関係が読み込まれ、環境変数またはハードコーディングされた文字列から必要な変数がプルされます。

| 変数 | 説明 |
|--|--|
|`sasToken`|Azure portal で作成された SAS トークンには、`?` が先頭に付加されます。 `sasToken` 変数に設定する前に、それを削除してください。| 
|`container`|BLOB ストレージ内のコンテナーの名前。 これは、ファイル システムのフォルダーやディレクトリに相当するものと考えることができます。|
|`storageAccountName`|対象のリソース名。|

:::code language="typescript" source="~/../js-e2e-browser-file-upload-storage-blob/src/azure-storage-blob.ts" highlight="2,5,16" id="snippet_package":::

### <a name="security-for-azure-credentials"></a>Azure 資格情報のセキュリティ

独自のプロジェクト内で、SAS トークンなどのシークレットを格納する場所を検討してください。 Azure 情報をセキュリティで保護することがアプリケーションによって要求される場合は、このストレージ コードを、クライアント上ではなく [Azure 関数](/azure/azure-functions/)でホストし、React アプリからその Azure 関数を呼び出すことを検討してください。  

### <a name="create-storage-client-and-manage-steps"></a>ストレージ クライアントを作成し、手順を管理する

`uploadFileToBlob` 関数は、ファイルの main 関数です。 これにより、Storage サービスのクライアント オブジェクトが作成され、コンテナー オブジェクトに対してクライアントが作成され、ファイルがアップロードされ、コンテナー内のすべての BLOB の一覧が取得されます。 

:::code language="typescript" source="~/../js-e2e-browser-file-upload-storage-blob/src/azure-storage-blob.ts" highlight="5,6,7" id="snippet_uploadFileToBlob":::

### <a name="upload-file-to-blob"></a>ファイルを BLOB にアップロードする

`createBlobInContainer` 関数は、`uploadBrowserData` クライアント ライブラリ メソッドを使用して、ファイルをコンテナーにアップロードします。 画像の表示など、ファイルの種類に依存するブラウザーの機能を使用する場合は、要求と共にコンテンツの種類を送信する必要があります。 

:::code language="typescript" source="~/../js-e2e-browser-file-upload-storage-blob/src/azure-storage-blob.ts" highlight="10" id="snippet_createBlobInContainer":::

### <a name="get-list-of-blobs"></a>BLOB の一覧を取得する

`getBlobsInContainer` 関数は、コンテナー内の BLOB の URL の一覧を取得します。 これらの URL は、HTML のイメージ表示の `src` として使用されるように構成されています: `<img src={item} alt={item} height="200" />`。 

:::code language="typescript" source="~/../js-e2e-browser-file-upload-storage-blob/src/azure-storage-blob.ts" highlight="10" id="snippet_getBlobsInContainer":::

## <a name="clean-up-resources"></a>リソースをクリーンアップする

Visual Studio Code で、Storage 用の Azure エクスプローラーを使用し、リソースを右クリックしてから **[ストレージ アカウントの削除]** を選択します。

## <a name="next-steps"></a>次のステップ

このアプリを引き続き使用する場合は、次のいずれかの選択肢を使用して、アプリを Azure にデプロイしてホスティングする方法について学習してください。

* [静的 Web アプリとしてアップロードする](/azure/static-web-apps/getting-started?tabs=vanilla-javascript)
* [App Service の Visual Studio Code 拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureappservice)を使用して Web アプリ リソースにアップロードする
* [アプリを Azure VM にアップロードする](nodejs-virtual-machine-vm/introduction.md)
