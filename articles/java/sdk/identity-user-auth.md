---
title: ユーザー資格情報を使用した Azure 認証
description: ユーザー資格情報を使用したアプリケーション認証に関する Azure SDK for Java の概念の概要
author: g2vinay
ms.date: 02/02/2021
ms.topic: conceptual
ms.custom: devx-track-java
ms.author: vigera
ms.openlocfilehash: d2b5b24b0f39d56ca2235ec56a4ea56c4be6b185
ms.sourcegitcommit: 71847ee0a1fee3f3320503629d9a8c82319a1f6a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/03/2021
ms.locfileid: "99528524"
---
# <a name="azure-authentication-with-user-credentials"></a>ユーザー資格情報を使用した Azure 認証

この記事では、ユーザーの入力した資格情報による Azure Active Directory トークン認証が、Azure Identity ライブラリでどのようにサポートされているかについて説明します。 このサポートは、下で説明する TokenCredential 実装のセットによって可能になります。

この記事では、次のトピックについて説明します。

* [デバイス コードの資格情報](#device-code-credential)
* [対話型ブラウザーの資格情報](#interactive-browser-credential)
* [ユーザー名とパスワードの資格情報](#username-password-credential)

## <a name="device-code-credential"></a>デバイス コードの資格情報

デバイス コードの資格情報は、UI が制限されているデバイスでユーザーを対話形式で認証するものです。 アプリケーションは、認証を試みるときに、ブラウザー対応のコンピューターでログイン URL にアクセスするようにユーザーに求めます。 そこでユーザーは、指示に記載されているデバイス コードを、ログイン資格情報とともに入力します。 認証が成功すると、認証を要求したアプリケーションは、それが実行されているデバイスで正常に認証されます。

詳細については、「[Microsoft ID プラットフォームと OAuth 2.0 デバイス許可付与フロー](/azure/active-directory/develop/v2-oauth2-device-code)」をご覧ください。

### <a name="enable-applications-for-device-code-flow"></a>アプリケーションのデバイス コード フローを有効にする

ユーザーをデバイス コード フローで認証するには、次の手順に従います。

1. Azure portal で Azure Active Directory に移動して、アプリ登録を見つけます。
2. **[認証]** セクションに移動します。
3. **[推奨されるリダイレクト URI]** で、`/common/oauth2/nativeclient` で終わる URI をオンにします。
4. **[既定のクライアントの種類]|** で、[`Treat application as a public client`] に [`yes`] を選択します。

これらの手順によってアプリケーションは認証されますが、Active Directory にログインしたり、リソースにアクセスしたりするためのアクセス許可はまだありません。 この問題に対処するには、 **[API のアクセス許可]** に移動して、Microsoft Graph と、アクセスするリソース (Azure Service Management、Key Vault など) を有効にします。

また、初めてログインするときは、アプリケーションに同意を付与するためにテナントの管理者である必要もあります。

Active Directory でデバイス コード フロー オプションを構成できない場合は、アプリをマルチテナントにすることが必要になる可能性があります。 アプリをマルチテナントにするには、 **[認証]** パネルに移動してから、 **[任意の組織のディレクトリ内のアカウント]** を選択します。 次に、 *[アプリケーションは、パブリック クライアントとして扱います]* に **[はい]** を選択します。

### <a name="authenticate-a-user-account-with-device-code-flow"></a>ユーザー アカウントをデバイス コード フローで認証する

次の例では、IoT デバイスで `DeviceCodeCredential` を使用して、[Java 用 Azure Key Vault シークレット クライアント ライブラリ][secrets_client_library]の `SecretClient` を認証する場合を示しています。

```java
/**
 * Authenticate with device code credential.
 */
DeviceCodeCredential deviceCodeCredential = new DeviceCodeCredentialBuilder()
    .challengeConsumer(challenge -> {
    // Lets the user know about the challenge.
    System.out.println(challenge.getMessage());
    }).build();

// Azure SDK client builders accept the credential as a parameter.
SecretClient client = new SecretClientBuilder()
    .vaultUrl("https://<your Key Vault name>.vault.azure.net")
    .credential(deviceCodeCredential)
    .buildClient();
```

## <a name="interactive-browser-credential"></a>対話型ブラウザーの資格情報

この資格情報では、既定のシステム ブラウザーを使用してユーザーを対話形式で認証します。自分の資格情報を使用してアプリケーションを認証することで円滑に認証できます。

### <a name="enable-applications-for-interactive-browser-oauth-2-flow"></a>アプリケーションの対話型ブラウザー OAuth 2 フローを有効にする

`InteractiveBrowserCredential` を使用するには、Azure Active Directory のアプリケーションを、ユーザーの代わりにログインするアクセス許可で登録する必要があります。 上のデバイス コード フローの手順に従って、アプリケーションを登録します。 前述のように、ユーザー アカウントがログインする前に、テナントの管理者がアプリケーションに同意を与える必要があります。

お気づきのように、`InteractiveBrowserCredentialBuilder` にはリダイレクト URL が必要です。 登録した AAD アプリケーションの **[認証]** セクションの **[リダイレクト URI]** サブセクションにリダイレクト URL を追加します。

### <a name="authenticate-a-user-account-interactively-in-the-browser"></a>ユーザー アカウントをブラウザーで対話形式で認証する

次の例では、`InteractiveBrowserCredential` を使用して [azure-security-keyvault-secrets][secrets_client_library] クライアント ライブラリの `SecretClient` を認証する場合を示しています。

```java
/**
 * Authenticate interactively in the browser.
 */
InteractiveBrowserCredential interactiveBrowserCredential = new InteractiveBrowserCredentialBuilder()
    .clientId("<your app client ID>")
    .redirectUrl("YOUR_APP_REGISTERED_REDIRECT_URL")
    .build();

// Azure SDK client builders accept the credential as a parameter.
SecretClient client = new SecretClientBuilder()
    .vaultUrl("https://<your Key Vault name>.vault.azure.net")
    .credential(interactiveBrowserCredential)
    .buildClient();
```

## <a name="username-password-credential"></a>ユーザー名とパスワードの資格情報

`UsernamePasswordCredential` は、多要素認証が不要なユーザー資格情報を使用してパブリック クライアント アプリケーションを認証するのに役立ちます。 次の例では、[azure-security-keyvault-secrets][secrets_client_library] クライアント ライブラリの `SecretClient` を、`UsernamePasswordCredential` を使用して認証する場合を示しています。 ユーザーは多要素認証をオンにしてはなりません。

```java
/**
 * Authenticate with username, password.
 */
UsernamePasswordCredential usernamePasswordCredential = new UsernamePasswordCredentialBuilder()
    .clientId("<your app client ID>")
    .username("<your username>")
    .password("<your password>")
    .build();

// Azure SDK client builders accept the credential as a parameter.
SecretClient client = new SecretClientBuilder()
    .vaultUrl("https://<your Key Vault name>.vault.azure.net")
    .credential(usernamePasswordCredential)
    .buildClient();
```

詳細については、「[Microsoft ID プラットフォームと OAuth 2.0 リソース所有者のパスワード資格情報](/azure/active-directory/develop/v2-oauth-ropc)」を参照してください。

## <a name="next-steps"></a>次のステップ

この記事では、ユーザー資格情報を使用した認証について説明しました。 この形式の認証は、Azure SDK for Java でできる複数の認証方法のうちの 1 つです。 次の記事では、他の方法について説明します。

* [開発環境での Azure 認証](identity-dev-env-auth.md)
* [Azure でホストされているアプリケーションの認証](identity-azure-hosted-auth.md)
* [サービス プリンシパルを使用した認証](identity-service-principal-auth.md)

認証について習得した後、SDK によって提供されるログ機能の詳細について「[Azure SDK for Java でログを構成する](logging-overview.md)」を参照してください。

<!-- LINKS -->
[secrets_client_library]: https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/keyvault/azure-security-keyvault-secrets
