---
title: Java と Azure ID を使用した Azure 認証
description: Azure SDK の認証および ID 機能の概要
author: g2vinay
ms.date: 02/02/2021
ms.topic: conceptual
ms.custom: devx-track-java
ms.author: vigera
ms.openlocfilehash: e0349297bed9ee2e499c904483d9c3e018a6cddf
ms.sourcegitcommit: 576c878c338d286060010646b96f3ad0fdbcb814
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102118554"
---
# <a name="azure-authentication-with-java-and-azure-identity"></a>Java と Azure ID を使用した Azure 認証

この記事では、Azure SDK for Java での Azure Active Directory トークン認証サポートを可能にする Java Azure Identity ライブラリの概要について説明します。 このライブラリには、AAD トークン認証をサポートする Azure SDK クライアントを構築するために使用できる `TokenCredential` 実装のセットが用意されています。

Azure Identity ライブラリの現在のサポート内容を以下に示します。

* [Java 開発環境での Azure 認証](identity-dev-env-auth.md)によって、以下が可能になります。
  * IDEA IntelliJ 認証。[Azure Toolkit for IntelliJ](../toolkit-for-intellij/index.yml) から取得したログイン情報を使用。
  * Visual Studio Code 認証。[Visual Studio Code 用の Azure プラグイン](https://code.visualstudio.com/docs/azure/extensions)に保存されたログイン情報を使用。
  * Azure CLI 認証。[Azure CLI](/cli/azure/what-is-azure-cli) に保存されたログイン情報を使用。
* [Azure でホストされているアプリケーションの認証](identity-azure-hosted-auth.md)によって、以下が可能になります。
  * 既定の Azure 資格情報認証
  * マネージド ID 認証
* [サービス プリンシパルを使用した認証](identity-service-principal-auth.md)によって、以下が可能になります。
  * クライアント シークレット認証
  * クライアント証明書認証
* [ユーザー資格情報による認証](identity-user-auth.md)によって、以下が可能になります。
  * 対話型ブラウザーの認証
  * デバイス コード認証
  * ユーザー名/パスワード認証

上記の各リンクをクリックすると、これらの認証方式の詳細をご覧になれます。 この記事の残りの部分では、よく使われる `DefaultAzureCredential` と関連トピックを紹介します。

## <a name="add-the-maven-dependencies"></a>Maven の依存関係を追加する

Maven の依存関係を追加するには、プロジェクトの *pom.xml* ファイルに次の XML を含めます。 バージョン番号 *1.2.1* は、[Microsoft Azure Client Library For Identity ページ](https://mvnrepository.com/artifact/com.azure/azure-identity)に示されている最新リリースのバージョン番号に置き換えてください。

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-identity</artifactId>
    <version>1.2.1</version>
</dependency>
```

## <a name="key-concepts"></a>主要な概念

Azure Identity ライブラリを理解する上での主要概念として、資格情報の概念と、その資格情報の最も一般的な実装である `DefaultAzureCredential` の 2 つがあります。

資格情報はクラスの一種であり、サービス クライアントが要求を認証するために必要なデータを格納、または取得できます。 Azure SDK のサービス クライアントは、その作成時に資格情報を受け取り、これらの資格情報を使用してサービス要求の認証を行います。

Azure Identity ライブラリは Azure Active Directory での OAuth 認証に焦点を当てており、サービス要求を認証するための AAD トークンを取得できるさまざまな資格情報クラスが用意されています。 このライブラリ内のすべての資格情報クラスは [azure-core][azure_core_library] の `TokenCredential` 抽象クラスの実装です。これらはいずれも、`TokenCredential` で認証できるサービス クライアントの作成に使用できます。

`DefaultAzureCredential` は、アプリケーションを最終的に Azure Cloud で実行することを目的とするほとんどのシナリオに適しています。 `DefaultAzureCredential` は、デプロイ時に認証によく使用される資格情報と、開発環境での認証に使用される資格情報を組み合わせたものです。 `DefaultAzureCredential` を使用した例などの詳細については、「[Azure でホストされている Java アプリケーションの認証](identity-azure-hosted-auth.md)」の「[既定の Azure 資格情報](identity-azure-hosted-auth.md#default-azure-credential)」セクションを参照してください。

## <a name="examples"></a>例

「[Azure SDK for Java を使用する](overview.md#provision-and-manage-azure-resources-with-management-libraries)」で説明しているように、管理ライブラリには若干の違いがあります。 こうした違いの例として、Azure サービスを *使用する* ためのライブラリがクライアント ライブラリと呼ばれるのに対し、Azure サービスを *管理する* ためのライブラリは管理ライブラリと呼ばれます。 以下のセクションでは、クライアントと管理の両ライブラリにおける認証の概要について説明します。

### <a name="authenticate-azure-client-libraries"></a>Azure クライアント ライブラリの認証

次の例では、`DefaultAzureCredential` を使用して [azure-security-keyvault-secrets][secrets_client_library] クライアント ライブラリの `SecretClient` を認証する場合を示しています。

```java
// Azure SDK client builders accept the credential as a parameter.
SecretClient client = new SecretClientBuilder()
  .vaultUrl("https://<your Key Vault name>.vault.azure.net")
  .credential(new DefaultAzureCredentialBuilder().build())
  .buildClient();
```

### <a name="authenticate-azure-management-libraries"></a>Azure 管理ライブラリの認証

Azure 管理ライブラリでは Azure クライアント ライブラリと同じ資格情報 API を使用しますが、そのサブスクリプションの Azure リソースを管理するための [Azure サブスクリプション ID](/learn/modules/create-an-azure-account/4-multiple-subscriptions) も必要になります。

サブスクリプション ID は、[Azure portal の [サブスクリプション]](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade) ページにあります。 または、次の [Azure CLI][azure_cli] コマンドを使ってサブスクリプション ID を取得します。

```azurecli
az account list --output table
```

サブスクリプション ID は `AZURE_SUBSCRIPTION_ID` 環境変数で設定できます。 この ID は、次の例に示すように、`Manager` インスタンスの作成時に既定のサブスクリプション ID として `AzureProfile` により取得されます。

```java
AzureResourceManager azureResourceManager = AzureResourceManager.authenticate(
        new DefaultAzureCredentialBuilder().build(),
        new AzureProfile(AzureEnvironment.AZURE))
    .withDefaultSubscription();
```

この例で使用している `DefaultAzureCredential` は、`DefaultAzureCredential` を使用して `AzureResourceManager` インスタンスを認証します。 Azure Identity ライブラリに用意されている他のトークン資格情報の実装を、`DefaultAzureCredential` の代わりに使用することもできます。

## <a name="troubleshooting"></a>トラブルシューティング

資格情報による認証に失敗した場合や認証を実行できない場合、例外が発生します。 資格情報による認証に失敗すると `ClientAuthenticationException` が発生し、その `message` 属性に、認証が失敗した理由が示されます。 `ChainedTokenCredential` でこの例外が発生すると、基になる資格情報一覧のチェーン実行が停止します。

資格情報に必要な、基になるリソースの 1 つが当該コンピューターで使用できないために認証を実行できない場合、`CredentialUnavailableException` が発生し、その `message` 属性に、認証実行のために資格情報を使用できない理由が示されます。 `ChainedTokenCredential` でこの例外が発生すると、このメッセージには、チェーン内の各資格情報からのエラー メッセージが集約されます。

## <a name="next-steps"></a>次のステップ

この記事では、Azure SDK for Java で使用できる Azure Identity 機能について紹介しました。 ここで説明した `DefaultAzureCredential` は、多くの場合に適しており、よく使用されます。 次の記事では、Azure Identity ライブラリを使用した他の認証方法について説明するとともに、 `DefaultAzureCredential` についてより詳しく説明します。

* [開発環境での Azure 認証](identity-dev-env-auth.md)
* [Azure でホストされているアプリケーションの認証](identity-azure-hosted-auth.md)
* [サービス プリンシパルを使用した認証](identity-service-principal-auth.md)
* [ユーザー資格情報を使用した認証](identity-user-auth.md)

<!-- LINKS -->
[azure_cli]: /cli/azure
[azure_sub]: https://azure.microsoft.com/free/
[source]: https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/identity/azure-identity
[aad_doc]: /azure/active-directory/
[code_of_conduct]: https://opensource.microsoft.com/codeofconduct/
[keys_client_library]: https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/keyvault/azure-security-keyvault-keys
[logging]: https://github.com/Azure/azure-sdk-for-java/wiki/Logging-with-Azure-SDK
[secrets_client_library]: https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/keyvault/azure-security-keyvault-secrets
[eventhubs_client_library]: https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/eventhubs/azure-messaging-eventhubs
[azure_core_library]: https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/core
[javadoc]: https://azure.github.io/azure-sdk-for-java
[jdk_link]: /java/azure/jdk