---
title: Java 開発環境での Azure 認証
description: 開発環境内での認証に関する Azure SDK for Java の概念の概要
author: g2vinay
ms.date: 02/02/2021
ms.topic: conceptual
ms.custom: devx-track-java
ms.author: vigera
ms.openlocfilehash: 7913430a81a0fe223d6eb23a48541e0f752323b2
ms.sourcegitcommit: 71847ee0a1fee3f3320503629d9a8c82319a1f6a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/03/2021
ms.locfileid: "99528530"
---
# <a name="azure-authentication-in-java-development-environments"></a>Java 開発環境での Azure 認証

この記事では、Azure Active Directory トークン認証のための Azure Identity ライブラリ サポートの概要について説明します。 このサポートにより、一連の `TokenCredential` 実装によって開発者のコンピューターでローカルに実行されているアプリケーションの認証が可能になります。

この記事では、次のトピックについて説明します。

* [デバイス コードの資格情報](#device-code-credential)
* [対話型ブラウザーの資格情報](#interactive-browser-credential)
* [Azure CLI の資格情報](#azure-cli-credential)
* [IntelliJ の資格情報](#intellij-credential)
* [Visual Studio Code の資格情報](#visual-studio-code-credential)

## <a name="device-code-credential"></a>デバイス コードの資格情報

デバイス コードの資格情報は、UI が制限されているデバイスでユーザーを対話形式で認証するものです。 これは、アプリケーションが認証を試みるときに、ブラウザー対応のコンピューターでログイン URL にアクセスするようにユーザーに求めることで機能します。 そこでユーザーは、指示に記載されているデバイス コードを、ログイン資格情報とともに入力します。 認証が成功すると、認証を要求したアプリケーションは、それが実行されているデバイスで正常に認証されます。

詳細については、「[Microsoft ID プラットフォームと OAuth 2.0 デバイス許可付与フロー](/azure/active-directory/develop/v2-oauth2-device-code)」をご覧ください。

### <a name="enable-applications-for-device-code-flow"></a>アプリケーションのデバイス コード フローを有効にする

ユーザーをデバイス コード フローで認証するには、次の手順に従います。

1. Azure portal で Azure Active Directory に移動して、自分のアプリ登録を見つけます。
2. **[認証]** セクションに移動します。
3. **[Suggested Redirected URIs]\(推奨されるリダイレクト URI\)** で、`/common/oauth2/nativeclient` で終わる URI をオンにします。
4. **[既定のクライアントの種類]** で、 **[アプリケーションは、パブリック クライアントとして扱います]** に対して *[はい]* を選択します。

これらの手順によってアプリケーションは認証されますが、まだ Active Directory にログインするためのアクセス許可や自分のアクセス リソースがありません。 この問題に対処するには、 **[API のアクセス許可]** に移動して、Microsoft Graph と、アクセスするリソースを有効にします。

また、初めてログインするときは、アプリケーションに同意を付与するためにテナントの管理者である必要もあります。

Active Directory でデバイス コード フロー オプションを構成できない場合は、アプリがマルチテナントであることが必要な場合があります。 アプリをマルチテナントにするには、 **[認証]** パネルに移動してから、 **[任意の組織のディレクトリ内のアカウント]** を選択します。 その後で、 **[アプリケーションは、パブリック クライアントとして扱います]** に *[はい]* を選択します。

### <a name="authenticate-a-user-account-with-device-code-flow"></a>ユーザー アカウントをデバイス コード フローで認証する

次の例では、IoT デバイス上で `DeviceCodeCredential` を使用して [azure-security-keyvault-secrets][secrets_client_library] クライアント ライブラリから `SecretClient` を認証することを示しています。

```java
DeviceCodeCredential deviceCodeCredential = new DeviceCodeCredentialBuilder()
  .challengeConsumer(challenge -> {
    // lets user know of the challenge
    System.out.println(challenge.getMessage());
  }).build();

// Azure SDK client builders accept the credential as a parameter
SecretClient client = new SecretClientBuilder()
  .vaultUrl("https://<your Key Vault name>.vault.azure.net")
  .credential(deviceCodeCredential)
  .buildClient();
```

## <a name="interactive-browser-credential"></a>対話型ブラウザーの資格情報

この資格情報では、既定のシステム ブラウザーを使用してユーザーを対話形式で認証し、自分の資格情報を使用してアプリケーションを認証できるようにすることで、スムーズな認証エクスペリエンスを提供します。

### <a name="enable-applications-for-interactive-browser-oauth-2-flow"></a>アプリケーションの対話型ブラウザー OAuth 2 フローを有効にする

`InteractiveBrowserCredential` を使用するには、Azure Active Directory のアプリケーションを、ユーザーの代わりにログインするアクセス許可で登録する必要があります。 デバイス コード フローでアプリケーションを登録するには、上記の手順に従います。 前述のように、ユーザー アカウントがログインする前に、テナントの管理者がアプリケーションに同意を与える必要があります。

お気づきのように、`InteractiveBrowserCredentialBuilder` にはリダイレクト URL が必要です。 登録した AAD アプリケーションの **[認証]** セクションの **[リダイレクト URI]** サブセクションにリダイレクト URL を追加します。

### <a name="authenticate-a-user-account-interactively-in-the-browser"></a>ユーザー アカウントをブラウザーで対話形式で認証する

次の例では、`InteractiveBrowserCredential` を使用して [azure-security-keyvault-secrets][secrets_client_library] クライアント ライブラリから `SecretClient` を認証することを示しています。

```java
InteractiveBrowserCredential interactiveBrowserCredential = new InteractiveBrowserCredentialBuilder()
  .clientId("<your client ID>")
  .redirectUrl("http://localhost:8765")
  .build();

// Azure SDK client builders accept the credential as a parameter
SecretClient client = new SecretClientBuilder()
  .vaultUrl("https://<your Key Vault name>.vault.azure.net")
  .credential(interactiveBrowserCredential)
  .buildClient();
```

## <a name="azure-cli-credential"></a>Azure CLI の資格情報

Visual Studio Code の資格情報は、Azure CLI で有効になっているユーザーまたはサービス プリンシパルを使用して、開発環境で認証を行います。 既にログインしているユーザーの Azure CLI を使用し、その CLI を使用して、Azure Active Directory に対してアプリケーションを認証します。

### <a name="sign-in-azure-cli-for-azureclicredential"></a>AzureCliCredential を目的とした Azure CLI へのサインイン

次の [Azure CLI][azure_cli] コマンドを使用して、ユーザーとしてサインインします。

```bash
az login
```

次のコマンドを使用して、サービス プリンシパルとしてサインインします。

```bash
az login --service-principal --username <client ID> --password <client secret> --tenant <tenant ID>
```

アカウントまたはサービス プリンシパルに複数のテナントへのアクセス権がある場合は、次のコマンドの出力で、目的のテナントまたはサブスクリプションの状態が "有効" になっていることを確認してください。

```bash
az account list
```

コードで `AzureCliCredential` を使用する前に、次のコマンドを実行して、アカウントが正常に構成されていることを確認します。

```bash
az account get-access-token
```

組織内の更新トークンの有効性によっては、一定期間後にこのプロセスを繰り返す必要がある場合があります。 一般に、更新トークンの有効期間は数週間から数か月です。 `AzureCliCredential` によって、もう一度サインインするように求められます。

### <a name="authenticate-a-user-account-with-azure-cli"></a>ユーザー アカウントを Azure CLI で認証する

次の例では、Azure CLI がインストール済みでサインイン済みのワークステーション上で `AzureCliCredential` を使用して [azure-security-keyvault-secrets][secrets_client_library] クライアント ライブラリから `SecretClient` を認証することを示しています。

```java
AzureCliCredential cliCredential = new AzureCliCredentialBuilder().build();

// Azure SDK client builders accept the credential as a parameter.
SecretClient client = new SecretClientBuilder()
  .vaultUrl("https://<your Key Vault name>.vault.azure.net")
  .credential(cliCredential)
  .buildClient();
```

## <a name="intellij-credential"></a>IntelliJ の資格情報

Visual Studio Code の資格情報は、Azure Toolkit for IntelliJ のアカウントを使用して、開発環境で認証を行います。 IntelliJ IDE でログインしているユーザー情報を使用し、それを使用して Azure Active Directory に対してアプリケーションを認証します。

## <a name="sign-in-azure-toolkit-for-intellij-for-intellijcredential"></a>IntelliJCredential を目的とした Azure Toolkit for IntelliJ へのサインイン

下に示す手順に従います。

1. IntelliJ ウィンドウで、 **[File]\(ファイル\) > [Settings]\(設定\) > [Plugins]\(プラグイン\)** を開きます。
1. マーケットプレースで「Azure Toolkit for IntelliJ」を検索します。 IDE をインストールして再起動します。
1. 新しいメニュー項目 **[Tools]\(ツール\) > [Azure] > [Azure Sign In…]\(Azureサインイン...\)** を見つけます。
1. **[Device Login]\(デバイス ログイン\)** は、ユーザー アカウントとしてログインするのに役立ちます。 デバイス コードを使用して、指示に従って `login.microsoftonline.com` Web サイトにログインします。 IntelliJ によって、サブスクリプションを選択するように求められます。 アクセスするリソースを含むサブスクリプションを選択します。

Windows では、IntelliJ の資格情報を読み取るための KeePass データベースのパスも必要になります。 このパスは、 **[File]\(ファイル\) > [Settings]\(設定\) > [Appearance & Behavior]\(外観と動作\) > [System Settings]\(システム設定\) > [Passwords]\(パスワード\)** の下の IntelliJ 設定から見つけることができます。 KeePassDatabase パスの場所をメモしておきます。

## <a name="authenticate-a-user-account-with-intellij-idea"></a>ユーザー アカウントを IntelliJ IDEA で認証する

次の例では、IntelliJ IDEA がインストールされ、ユーザーが Azure アカウントを使用してサインインしたワークステーション上で、`IntelliJCredential` を使用して [azure-security-keyvault-secrets][secrets_client_library] クライアント ライブラリから `SecretClient` を認証します。

```java
IntelliJCredential intelliJCredential = new IntelliJCredentialBuilder()
  // KeePass configuration isrequired only for Windows. No configuration needed for Linux / Mac.
  .keePassDatabasePath("C:\\Users\\user\\AppData\\Roaming\\JetBrains\\IdeaIC2020.1\\c.kdbx")
  .build();

// Azure SDK client builders accept the credential as a parameter
SecretClient client = new SecretClientBuilder()
  .vaultUrl("https://<your Key Vault name>.vault.azure.net")
  .credential(intelliJCredential)
  .buildClient();
```

## <a name="visual-studio-code-credential"></a>Visual Studio Code の資格情報

Visual Studio Code の資格情報を使用すると、[VS Code Azure Account 拡張機能](https://github.com/Microsoft/vscode-azure-account)を使用して VS Code がインストールされた開発環境での認証が可能になります。 VS Code IDE でログインしているユーザー情報を使用し、それを使用して Azure Active Directory に対してアプリケーションを認証します。

### <a name="sign-in-visual-studio-code-azure-account-extension-for-visualstudiocodecredential"></a>VisualStudioCodeCredential を目的とした Visual Studio Code Azure Account 拡張機能へのサインイン

Visual Studio Code 認証は、[Azure Account 拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azure-account)との統合によって処理されます。 この形式の認証を使用するには、Azure Account 拡張機能をインストールしてから、 **[View]\(表示\) > [Command Palette]\(コマンドパレット\)**  を使用して **Azure:Sign In** コマンドを検索します。 このコマンドを実行すると、ブラウザー ウィンドウが開き、Azure にサインインするためのページが表示されます。 ログイン プロセスが完了したら、指示に従ってブラウザーを閉じることができます。 (デバッガーで、または開発用コンピューター上の任意の場所で) アプリケーションを実行すると、サインイン時の資格情報が使用されます。

### <a name="authenticate-a-user-account-with-visual-studio-code"></a>ユーザー アカウントを Visual Studio Code で認証する

次の例では、Visual Studio Code がインストールされ、ユーザーが Azure アカウントを使用してサインインしたワークステーション上で、`VisualStudioCodeCredential` を使用して [azure-security-keyvault-secrets][secrets_client_library] クライアント ライブラリから `SecretClient` を認証します。

```java
VisualStudioCodeCredential visualStudioCodeCredential = new VisualStudioCodeCredentialBuilder().build();

// Azure SDK client builders accept the credential as a parameter.
SecretClient client = new SecretClientBuilder()
  .vaultUrl("https://<your Key Vault name>.vault.azure.net")
  .credential(visualStudioCodeCredential)
  .buildClient();
```

## <a name="next-steps"></a>次のステップ

この記事では、お使いのコンピューターで使用可能な資格情報を使用した開発時の認証について説明しました。 この形式の認証は、Azure SDK for Java で可能な認証方法のうちの 1 つです。 次の記事では、他の方法について説明します。

* [Azure でホストされるアプリケーションの認証](identity-azure-hosted-auth.md)
* [サービス プリンシパルを使用した認証](identity-service-principal-auth.md)
* [ユーザー資格情報を使用した認証](identity-user-auth.md)

認証について習得した後、SDK によって提供されるログ機能の詳細について「[Azure SDK for Java でログを構成する](logging-overview.md)」を参照してください。

<!-- LINKS -->
[azure_cli]: /cli/azure
[secrets_client_library]: https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/keyvault/azure-security-keyvault-secrets
