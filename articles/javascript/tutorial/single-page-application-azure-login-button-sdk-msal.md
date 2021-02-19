---
title: チュートリアル:Microsoft ログイン ボタンを React SPA に追加する
description: このチュートリアルで示されている Azure Active Directory 認証は、ログインとログアウトのボタンであり、ユーザーのユーザー名　(メールアドレス) へのアクセスを提供します。 シングルページ アプリケーション (SPA) でのユーザーの操作を管理するために、Azure クライアント側 SDK (`@azure/msal-browser`) を使用して TypeScript アプリケーションを開発します。
ms.topic: tutorial
ms.date: 02/16/2020
ms.custom: devx-track-js, "azure-sdk-javascript-@azure/msal-browser-2.7.0"
ms.history:
- 20210216:fix public issue 443
ms.openlocfilehash: d5d64f1de3637c79a081e64dbf3cd632e15a3ef3
ms.sourcegitcommit: 450cfb580f0fef7aabd38a7db17ecf7502126ce4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/16/2021
ms.locfileid: "100548843"
---
# <a name="add-microsoft-login-button-to-a-single-page-application-for-authentication"></a>認証用のシングルページ アプリケーションに Microsoft ログイン ボタンを追加する

この TypeScript チュートリアルで示されている Azure 認証はログインとログアウトのボタンであり、それによってユーザーのアカウントへのアクセスが提供されます。 シングルページ アプリケーション (SPA) でのユーザーの操作を管理するために、Azure クライアント側 SDK (`@azure/msal-browser`) を使用してアプリケーションを開発します。

* [サンプル コード](https://github.com/Azure-Samples/js-e2e-client-azure-login-button)

## <a name="application-architecture-and-functionality"></a>アプリケーションのアーキテクチャと機能

このチュートリアルで構築する SPA は、次のタスクを行う React アプリ (create-react-app) です。

- Office 365 や Outlook.com など、Microsoft がサポートするログインを使用してログインする
- アプリケーションからログオフする

このサンプルでは、高速で単純なシングルページ アプリケーションを提供するために、TypeScript の **create-react-app** を使用します。 このフロントエンド フレームワークでは、Azure SDK を使用した一般的なクライアント開発でのいくつかのショートカットが用意されています。

- バンドル (クライアント アプリケーションで使用される Azure SDK に必要)
- `.env` ファイル内の環境変数

## <a name="1-set-up-development-environment"></a>1.開発環境をセットアップする

次のソフトウェアがローカル コンピューターにインストールされていることを確認します。

- アクティブなサブスクリプションが含まれる Azure アカウント。 [無料で作成できます](https://azure.microsoft.com/free/)。
- [Node.js および npm](https://nodejs.org/en/download) - ローカル コンピューターにインストール済み。
- [Visual Studio Code](https://code.visualstudio.com/)、または Bash シェルかターミナルを備えた同等の IDE - ローカル コンピューターにインストール済み。 

## <a name="2-keep-value-for-environment-variable"></a>2.環境変数の値を保持する

アプリ登録のクライアント ID 値をコピーする場所 (テキスト ファイルなど) を確保します。 このクライアント ID は、次のセクションの手順 5 で取得します。 この値は、Web アプリの環境変数として使用されます。  

## <a name="3-create-app-registration-for-authentication"></a>3.認証用のアプリ登録を作成する

1. 既定のディレクトリのアプリを登録するために [Azure portal](https://portal.azure.com/?quickstart=True#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps) に **サインイン** します。
1. **[+ 新規登録]** を選択します。
1. 次の表を使用して、**アプリ登録データを入力します**。

   | フィールド                   | 値                                                                                                                                                                      |説明|
   | ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |--|
   | 名前                    | `Simple Auth Tutorial`|これはユーザーがアプリにサインインするときにアクセス許可フォームに表示されるアプリ名です。                                                 |
   | サポートされているアカウントの種類 | **任意の組織のディレクトリ (Azure AD ディレクトリ - マルチテナント) 内のアカウントと、個人用の Microsoft アカウント**|これはほとんどのアカウントの種類に対応しています。 |
   | リダイレクト URI の種類           | **シングル ページ アプリケーション (SPA)**                                                                                        | |
   | リダイレクト URI の値           | `http://localhost:3000` | 認証が成功または失敗した後に返される URL。                                                                                        |

    :::image type="content" source="../media/tutorial-login-button/azure-active-directory-create-new-app-registration.png" alt-text="Azure の新しいアプリの登録。":::  

1. **[登録]** を選択します。 アプリの登録プロセスが完了するまで待ちます。
1. アプリ登録の概要セクションから **アプリケーション (クライアント) ID をコピーします**。 この値は後でクライアント アプリの環境変数に追加します。

## <a name="4-create-react-single-page-application-for-typescript"></a>4.TypeScript 用の React シングルページ アプリケーションを作成する

1. Bash シェルで、言語として TypeScript を使用して、**create-react-app を作成します**。

   ```bash
   npx create-react-app tutorial-demo-login-button --template typescript
   ```

1. 新しいディレクトリに移動し、`@azure/msal-browser` 認証パッケージをインストールします。

   ```bash
   cd tutorial-demo-login-button && npm install @azure/msal-browser
   ```

1. ルート レベルに `.env` ファイルを作成し、次の行を追加します。

    :::code language="env" source="~/../js-e2e-client-azure-login-button/.env"  :::

    `.env` ファイルは、create-react-app フレームワークの一部として読み取られます。 このファイルには、ローカル開発用のクライアント ID を格納できます。 

1. アプリケーション (クライアント) ID を 2 番目の値にコピーします。

## <a name="5-add-login-and-logoff-buttons"></a>5.ログインとログオフのボタンを追加する

1. `./src` フォルダー内に、Azure 固有のファイル用の `azure` という名前のサブフォルダーを作成します。 

1. `azure` フォルダーに、`azure-authentication-config.ts` という名前の認証構成用の新しいファイルを作成し、次のコードをコピーします。

    :::code language="typescript" source="~/../js-e2e-client-azure-login-button/src/azure/azure-authentication-config.ts"  highlight="3-4,, 11-12":::

    このファイルでは、`.env` ファイルからアプリケーション ID を読み取り、Cookie ではなくセッションをブラウザー ストレージとして設定し、個人情報に配慮したログ記録を提供します。

1. `azure` フォルダーに、`azure-authentication-context.ts` という名前の Azure 認証ミドルウェア用の新しいファイルを作成し、次のコードをコピーします。

    :::code language="typescript" source="~/../js-e2e-client-azure-login-button/src/azure/azure-authentication-context.ts"  highlight="43, 58, 65":::

    このファイルは、`login` と `logout` の機能を持つユーザーに Azure への認証を提供します。

1. `azure` フォルダーに、`azure-authentication-component.tsx` という名前のユーザー インターフェイス ボタン コンポーネント ファイル用の新しいファイルを作成し、次のコードをコピーします。

   :::code language="typescript" source="~/../js-e2e-client-azure-login-button/src/azure/azure-authentication-component.tsx"  highlight="3, 11, 23, 29, 36":::

   このボタン コンポーネントによってユーザーがログインし、ユーザー アカウントが呼び出し元/親コンポーネントに渡されます。

   ボタンのテキストと機能は、ユーザーが現在ログインしているかどうかに基づいて切り替わり、コンポーネントに渡されるプロパティとして `onAuthenticated` 関数を使用してキャプチャされます。

   ユーザーがログインすると、ボタンはコールバック関数として `returnedAccountInfo` を使用して、Azure 認証ライブラリ メソッド `authenticationModule.login` を呼び出します。 返されたユーザー アカウントは、`onAuthenticated` 関数を使用して親コンポーネントに戻されます。


1. `./src/App.tsx` ファイルを開き、すべてのコードを次のコードに置き換えて、ログイン/ログアウト ボタン コンポーネントを組み込みます。

   :::code language="typescript" source="~/../js-e2e-client-azure-login-button/src/App.tsx"  highlight="10, 37-42":::

    ユーザーがログオンし、認証がこのアプリにリダイレクトされると、currentUser オブジェクトが表示されます。 

## <a name="6-run-react-spa-app-with-login-button"></a>6.ログイン ボタンを使用して React SPA アプリを実行する

1. Visual Studio Code ターミナルで、アプリを起動します。

    ```bash
    npm run start
    ```

1. Web ブラウザーで **[Login]\(ログイン\)** ボタンを選択します。 

    :::image type="content" source="../media/tutorial-login-button/create-react-app-before-authentication-login-button-display.png" alt-text="**Login**\(ログイン\) ボタンを選択する。":::  

1. ユーザー アカウントを選択します。 Azure portal にアクセスするために使用したものと同じアカウントである必要はありませんが、Microsoft 認証を提供するアカウントである必要があります。

    :::image type="content" source="../media/tutorial-login-button/authentication-popup-select-user-account.png" alt-text="ユーザー アカウントを選択する。Azure portal にアクセスするために使用したものと同じアカウントである必要はないが、Microsoft 認証を提供するアカウントである必要がある。":::

1. 1\) ユーザー名、2) アプリ名、3) 同意するアクセス許可が表示されていることを確認し、 **[Yes]\(はい\)** を選択します。

    :::image type="content" source="../media/tutorial-login-button/authentication-popup-let-this-app-access-your-info.png" alt-text="1) ユーザー名、2) アプリ名、3) 同意するアクセス許可が表示されていることを確認し、[Yes]\(はい\) を選択する。":::

1. ユーザー アカウント情報を確認します。 

    :::image type="content" source="../media/tutorial-login-button/create-react-app-after-authentication-login-button-succeeds.png" alt-text="ユーザー アカウント情報を確認する。":::  

1. アプリから **[Logout]\(ログアウト\)** ボタンを選択します。 アプリには、アクセス許可を取り消すための Microsoft ユーザー アプリへの便利なリンクも用意されています。 

## <a name="7-clean-up-resources"></a>7.リソースをクリーンアップする

このチュートリアルを完了したら、Azure portal の[アプリの登録リスト](https://portal.azure.com/?quickstart=True#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps)からアプリケーションを削除します。

アプリを保持しながら、特定のユーザー アカウントによってアプリに与えられたアクセス許可を取り消す場合は、次のいずれかのリンクを使用します。

* [AAD のアクセス許可を取り消す](https://myapps.microsoft.com/)
* [コンシューマーのアクセス許可を取り消す](https://account.live.com/consent/manage)

## <a name="next-steps"></a>次のステップ

このアプリはアプリのユーザー認証を提供し、ユーザー情報を返します。 単純なバージョンのアプリの場合、認証機能はここで止めることができます。あるいは、アプリのユーザー管理およびユーザー承認を管理するための機能をアプリ機能に追加することもできます。 

ユーザー管理は、選択した機能とツールに応じて、Azure Active Directory または独自のデータベースに格納できます。 

ユーザー承認は Azure によって提供するか、Azure を使用せずに承認を開発するか、または承認、ロール、アプリ機能のカスタム エクスペリエンスのために、この 2 つを組み合わせることもできます。 

* [MSAL ライブラリ](/azure/active-directory/develop/msal-overview)を引き続き使用してユーザー プロファイルを取得し、サイレント サインオンを提供する
* [Microsoft Graph](/graph/overview) を追加して、電子メールと予定表の予定を含む Microsoft 365 のユーザー アカウントにアクセスする