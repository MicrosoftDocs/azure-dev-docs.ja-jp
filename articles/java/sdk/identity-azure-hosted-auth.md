---
title: Azure でホストされる Java アプリケーションを認証する
description: Azure 内でホストされるアプリケーションの認証に関する Azure SDK for Java の概念の概要
author: g2vinay
ms.date: 02/02/2021
ms.topic: conceptual
ms.custom: devx-track-java
ms.author: vigera
ms.openlocfilehash: e9097a432abf5957c9983ec2846bfb18ba581db4
ms.sourcegitcommit: 71847ee0a1fee3f3320503629d9a8c82319a1f6a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/03/2021
ms.locfileid: "99528533"
---
# <a name="authenticate-azure-hosted-java-applications"></a>Azure でホストされる Java アプリケーションを認証する

この記事では、Azure でホストされているアプリケーションの Azure Active Directory トークン認証が、Azure Identity ライブラリによってどのようにサポートされているかについて説明します。 このサポートは、下で説明する一連の `TokenCredential` 実装によって可能になります。

この記事では、次のトピックについて説明します。

* [既定の Azure 資格情報](#default-azure-credential)
* [マネージド ID 資格情報](#managed-identity-credential)

## <a name="default-azure-credential"></a>既定の Azure 資格情報

`DefaultAzureCredential` は、アプリケーションが最終的に Azure クラウドで実行されるほとんどのシナリオに適しています。 `DefaultAzureCredential` は、デプロイ時の認証に一般的に使用される資格情報と、開発環境での認証に使用される資格情報を組み合わせたものです。 `DefaultAzureCredential` は、次のメカニズムを順に経由して認証を試行します。

![DefaultAzureCredential の認証フロー](./media/defaultazurecredential.svg)

* 環境 - `DefaultAzureCredential` は、[環境変数](#environment-variables)で指定されたアカウント情報を読み取り、それを使用して認証を行います。
* マネージド ID - マネージド ID が有効になっている Azure ホストにアプリケーションがデプロイされている場合、`DefaultAzureCredential` はそのアカウントを使用して認証を行います。
* IntelliJ - Azure Toolkit for IntelliJ 経由で認証した場合、`DefaultAzureCredential` はそのアカウントを使用して認証を行います。
* Visual Studio Code - Visual Studio Code Azure Account プラグインを使用して認証した場合、`DefaultAzureCredential` はそのアカウントを使用して認証を行います。
* Azure CLI - Azure CLI の `az login` コマンドを使用してアカウントを認証した場合、`DefaultAzureCredential` はそのアカウントを使用して認証を行います。

### <a name="configure-defaultazurecredential"></a>DefaultAzureCredential を構成する

`DefaultAzureCredential` は、`DefaultAzureCredentialBuilder` のセッターまたは環境変数による一連の構成をサポートしています。

* [環境変数](#environment-variables)で定義されているように環境変数 `AZURE_CLIENT_ID`、`AZURE_CLIENT_SECRET`、および `AZURE_TENANT_ID` を設定することで、それらの値によって指定されたサービス プリンシパルとして認証するように `DefaultAzureCredential` が構成されます。
* ビルダーで `.managedIdentityClientId(String)`、または環境変数 `AZURE_CLIENT_ID` を設定すると、`DefaultAzureCredential` はユーザー定義マネージド ID として認証するように構成され、これらを空のままにしておくと、システム割り当てマネージド ID として認証するように構成されます。
* ビルダーで `.tenantId(String)`、たは環境変数 `AZURE_TENANT_ID` を設定すると、`DefaultAzureCredential` は共有トークン キャッシュ、Visual Studio Code、IntelliJ IDEA 用の特定のテナントに対して認証するように構成されます。
* 環境変数 `AZURE_USERNAME` を設定すると、`DefaultAzureCredential` は共有トークン キャッシュから対応するキャッシュされたトークンを選択するように構成されます。
* ビルダーで `.intelliJKeePassDatabasePath(String)` を設定すると、`DefaultAzureCredential` は IntelliJ 資格情報で認証するときに特定の KeePass ファイルを読み取るように構成されます。

### <a name="authenticate-with-defaultazurecredential"></a>DefaultAzureCredential を使用して認証する

次の例では、`DefaultAzureCredential` を使用して [azure-security-keyvault-secrets][secrets_client_library] クライアント ライブラリから `SecretClient` を認証することを示しています。

```java
// Azure SDK client builders accept the credential as a parameter.
SecretClient client = new SecretClientBuilder()
  .vaultUrl("https://<your Key Vault name>.vault.azure.net")
  .credential(new DefaultAzureCredentialBuilder().build())
  .buildClient();
```

### <a name="authenticate-a-user-assigned-managed-identity-with-defaultazurecredential"></a>DefaultAzureCredential を使用してユーザー割り当てマネージド ID を認証する

次の例では、ユーザー割り当てマネージド ID が構成された Azure リソースにデプロイされた `DefaultAzureCredential` を使用して、[azure-security-keyvault-secrets][secrets_client_library] クライアント ライブラリから `SecretClient` を認証することを示しています。

```java
/**
 * The default credential will use the user-assigned managed identity with the specified client ID.
 */
DefaultAzureCredential defaultCredential = new DefaultAzureCredentialBuilder()
  .managedIdentityClientId("<managed identity client ID>")
  .build();

// Azure SDK client builders accept the credential as a parameter.
SecretClient client = new SecretClientBuilder()
  .vaultUrl("https://<your Key Vault name>.vault.azure.net")
  .credential(defaultCredential)
  .buildClient();
```

### <a name="authenticate-a-user-in-azure-toolkit-for-intellij-with-defaultazurecredential"></a>DefaultAzureCredential を使用して Azure Toolkit for IntelliJ でユーザーを認証する

次の例では、IntelliJ IDEA がインストールされ、ユーザーが Azure アカウントを使用して Azure Toolkit for IntelliJ にサインインしたワークステーション上で、`DefaultAzureCredential` を使用して [azure-security-keyvault-secrets][secrets_client_library] クライアント ライブラリから `SecretClient` を認証することを示しています。

IntelliJ IDEA の構成の詳細については、「[IntelliJCredential を目的とした Azure Toolkit for IntelliJ へのサインイン](identity-dev-env-auth.md#sign-in-azure-toolkit-for-intellij-for-intellijcredential)」を参照してください。

```java
/**
 * The default credential will use the KeePass database path to find the user account in IntelliJ on Windows.
 */
// KeePass configuration is required only for Windows. No configuration needed for Linux / Mac.
DefaultAzureCredential defaultCredential = new DefaultAzureCredentialBuilder()
  .intelliJKeePassDatabasePath("C:\\Users\\user\\AppData\\Roaming\\JetBrains\\IdeaIC2020.1\\c.kdbx")
  .build();

// Azure SDK client builders accept the credential as a parameter.
SecretClient client = new SecretClientBuilder()
  .vaultUrl("https://<your Key Vault name>.vault.azure.net")
  .credential(defaultCredential)
  .buildClient();
```

## <a name="managed-identity-credential"></a>マネージド ID 資格情報

マネージド ID は、Azure リソースのマネージド ID (システムまたはユーザー割り当て) を認証します。 そのため、アプリケーションが、`IDENTITY/MSI`、`IMDS` エンドポイント、またはその両方によってマネージド ID をサポートする Azure リソース内で実行されている場合、この資格情報によってアプリケーションが認証され、シークレットを使用しない優れた認証エクスペリエンスが提供されます。

詳細については、「[Azure リソースのマネージド ID とは](/azure/active-directory/managed-identities-azure-resources/overview)」を参照してください。

### <a name="authenticate-in-azure-with-managed-identity"></a>マネージド ID を使用して Azure で認証する

次の例では、システム割り当てまたはユーザー割り当てのマネージド ID を有効にして、Azure 上の仮想マシン、アプリ サービス、関数アプリ、Cloud Shell、サービス ファブリック、Arc、または AKS 環境の `ManagedIdentityCredential` を使用して、[azure-security-keyvault-secrets][secrets_client_library] クライアント ライブラリから `SecretClient` を認証することを示しています。

```java
/**
 * Authenticate with a managed identity.
 */
ManagedIdentityCredential managedIdentityCredential = new ManagedIdentityCredentialBuilder()
  .clientId("<user-assigned managed identity client ID>") // required only for user-assigned
  .build();

// Azure SDK client builders accept the credential as a parameter.
SecretClient client = new SecretClientBuilder()
  .vaultUrl("https://<your Key Vault name>.vault.azure.net")
  .credential(managedIdentityCredential)
  .buildClient();
```

## <a name="environment-variables"></a>環境変数

環境変数を使用して `DefaultAzureCredential` と `EnvironmentCredential` を構成できます。 認証の種類ごとに、特定の変数の値が必要です。

### <a name="service-principal-with-secret"></a>シークレットを持つサービス プリンシパル

| 変数名         | 値                                                  |
| --------------------- | ------------------------------------------------------ |
| `AZURE_CLIENT_ID`     | Azure Active Directory アプリケーションの ID。           |
| `AZURE_TENANT_ID`     | アプリケーションの Azure Active Directory テナントの ID。 |
| `AZURE_CLIENT_SECRET` | アプリケーションのクライアント シークレットの 1 つ。               |

### <a name="service-principal-with-certificate"></a>証明書を使用したサービス プリンシパル

| 変数名         | 値                                                                                                |
| --------------------- | ---------------------------------------------------------------------------------------------------- |
| `AZURE_CLIENT_ID`     | Azure Active Directory アプリケーションの ID。                                                         |
| `AZURE_TENANT_ID`     | アプリケーションの Azure Active Directory テナントの ID。                                               |
| `AZURE_CLIENT_CERTIFICATE_PATH` | (パスワード保護のない) 秘密キーを含む、PEM でエンコードされた証明書ファイルへのパス。|

### <a name="username-and-password"></a>ユーザー名とパスワード

| 変数名         | 値                                            |
| --------------------- | ------------------------------------------------ |
| `AZURE_CLIENT_ID`     | Azure Active Directory アプリケーションの ID。     |
| `AZURE_USERNAME`      | ユーザー名 (通常は電子メール アドレス)。           |
| `AZURE_PASSWORD`      | 指定されたユーザー名に関連するパスワード。  |

構成は上記の順序で試行されます。 たとえば、クライアント シークレットと証明書の値が両方存在する場合、クライアント シークレットが使用されます。

## <a name="next-steps"></a>次のステップ

この記事では、Azure でホストされるアプリケーションの認証について説明しました。 この形式の認証は、Azure SDK for Java で可能な認証方法のうちの 1 つです。 次の記事では、他の方法について説明します。

* [開発環境での Azure 認証](identity-dev-env-auth.md)
* [サービス プリンシパルを使用した認証](identity-service-principal-auth.md)
* [ユーザー資格情報を使用した認証](identity-user-auth.md)

認証について習得した後、SDK によって提供されるログ機能の詳細について「[Azure SDK for Java でログを構成する](logging-overview.md)」を参照してください。

<!-- LINKS -->
[secrets_client_library]: https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/keyvault/azure-security-keyvault-secrets
