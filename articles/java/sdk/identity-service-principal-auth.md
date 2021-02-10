---
title: サービス プリンシパルを使用した Azure 認証
description: サービス プリンシパルを使用したアプリケーションの認証に関する Azure SDK for Java の概念の概要
author: g2vinay
ms.date: 02/02/2021
ms.topic: conceptual
ms.custom: devx-track-java
ms.author: vigera
ms.openlocfilehash: fd8c107d4fb5d37e3f2c2e99ba45ad46a6de34ae
ms.sourcegitcommit: 71847ee0a1fee3f3320503629d9a8c82319a1f6a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/03/2021
ms.locfileid: "99528551"
---
# <a name="azure-authentication-with-service-principal"></a>サービス プリンシパルを使用した Azure 認証

この記事では、サービス プリンシパルを使用した Azure Active Directory トークン認証が、Azure Identity ライブラリによってどのようにサポートされているかについて説明します。 この記事では、次のトピックについて説明します。

* [Azure CLI でサービス プリンシパルを作成する](#create-a-service-principal-with-the-azure-cli)
* [クライアント シークレットの資格情報](#client-secret-credential)
* [クライアント証明書の資格情報](#client-certificate-credential)

詳しくは、「[Azure Active Directory のアプリケーションとサービス プリンシパルのオブジェクト](/azure/active-directory/develop/app-objects-and-service-principals)」をご覧ください。

## <a name="create-a-service-principal-with-the-azure-cli"></a>Azure CLI でサービス プリンシパルを作成する

クライアント シークレットの資格情報を作成または取得するには、下の [Azure CLI][azure_cli] の例を使用します。

次のコマンドを使用して、サービス プリンシパルを作成し、Azure リソースへのアクセスを構成します。

```azurecli
az ad sp create-for-rbac -n <your application name> --skip-assignment
```

このコマンドにより、次の出力のような値が返されます。

```output
{
"appId": "generated-app-ID",
"displayName": "dummy-app-name",
"name": "http://dummy-app-name",
"password": "random-password",
"tenant": "tenant-ID"
}
```

次のコマンドを使用して、証明書と共にサービス プリンシパルを作成します。 この証明書のパスと場所をメモしておきます。

```azurecli
az ad sp create-for-rbac -n <your application name> --skip-assignment --cert <certificate name> --create-cert
```

返された資格情報を確認し、次の情報をメモしておきます。

* `AZURE\_CLIENT\_ID` (appId)。
* `AZURE\_CLIENT\_SECRET` (パスワード)。
* `AZURE\_TENANT\_ID` (テナント)。

## <a name="client-secret-credential"></a>クライアント シークレットの資格情報

この資格情報は、作成されたサービス プリンシパルを、そのクライアント シークレット (パスワード) を使用して認証します。 この例では、`ClientSecretCredential` を使用して [azure-security-keyvault-secrets][secrets_client_library] クライアント ライブラリから `SecretClient` を認証することを示しています。

```java
/**
 *  Authenticate with client secret.
 */
ClientSecretCredential clientSecretCredential = new ClientSecretCredentialBuilder()
  .clientId("<your client ID>")
  .clientSecret("<your client secret>")
  .tenantId("<your tenant ID>")
  .build();

// Azure SDK client builders accept the credential as a parameter.
SecretClient client = new SecretClientBuilder()
  .vaultUrl("https://<your Key Vault name>.vault.azure.net")
  .credential(clientSecretCredential)
  .buildClient();
```

## <a name="client-certificate-credential"></a>クライアント証明書の資格情報

この資格情報は、作成されたサービス プリンシパルを、そのクライアント証明書を使用して認証します。 この例では、`ClientCertificateCredential` を使用して [azure-security-keyvault-secrets][secrets_client_library] クライアント ライブラリから `SecretClient` を認証することを示しています。

```java
/**
 *  Authenticate with a client certificate.
 */
ClientCertificateCredential clientCertificateCredential = new ClientCertificateCredentialBuilder()
  .clientId("<your client ID>")
  .pemCertificate("<path to PEM certificate>")
  // Choose between either a PEM certificate or a PFX certificate.
  //.pfxCertificate("<path to PFX certificate>", "PFX CERTIFICATE PASSWORD")
  .tenantId("<your tenant ID>")
  .build();

// Azure SDK client builders accept the credential as a parameter.
SecretClient client = new SecretClientBuilder()
  .vaultUrl("https://<your Key Vault name>.vault.azure.net")
  .credential(clientCertificateCredential)
  .buildClient();
```

## <a name="next-steps"></a>次のステップ

この記事では、サービス プリンシパルを使用した認証について説明しました。 この形式の認証は、Azure SDK for Java で可能な認証方法のうちの 1 つです。 次の記事では、他の方法について説明します。

* [開発環境での Azure 認証](identity-dev-env-auth.md)
* [Azure でホストされるアプリケーションの認証](identity-azure-hosted-auth.md)
* [ユーザー資格情報を使用した認証](identity-user-auth.md)

認証について習得した後、SDK によって提供されるログ機能の詳細について「[Azure SDK for Java でログを構成する](logging-overview.md)」を参照してください。

<!-- LINKS -->
[azure_cli]: /cli/azure
[secrets_client_library]: https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/keyvault/azure-security-keyvault-secrets
