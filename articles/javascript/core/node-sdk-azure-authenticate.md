---
title: Node.js 用 Azure 管理モジュールを使った認証
description: Node.js 用 Azure 管理モジュールへの認証にサービス プリンシパルを使う方法について説明します。
ms.topic: how-to
ms.date: 01/04/2021
ms.custom: devx-track-js
ms.openlocfilehash: e5774f0453960b41679a01170882fad1d9f50bad
ms.sourcegitcommit: b09d3aa79113af04a245b05cec2f810e43062152
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/02/2021
ms.locfileid: "99476438"
---
# <a name="authenticate-with-the-azure-management-modules-for-javascript"></a>JavaScript 用 Azure 管理モジュールを使用した認証

すべての [SDK クライアント ライブラリ](../azure-sdk-library-package-index.md)は、`credentials` オブジェクトを介して認証を行う必要があります。 必要な資格情報を認証したり作成したりするには、複数の方法があります。

認証は、すべてのソフトウェアやサービスと同様に、長年にわたって改善されています。 自分のサービスがどの認証ライブラリを使用しているかを知っておくことが重要です。 

認証ライブラリには、次のものが含まれます。

* @azure/identity - 最新の認証パッケージ
* @azure/ms-rest-nodeauth
* @azure/ms-rest-browserauth

古い認証パッケージが使用されています。 これらのパッケージを使用している場合は、より安全で堅牢なエクスペリエンスを実現するために、古い認証方法からの移行を検討してください。 

## <a name="best-practices-with-azure-sdk-client-library-authentication"></a>Azure SDK クライアント ライブラリ認証におけるベスト プラクティス

各 npm パッケージは、その正確なクライアント ライブラリの認証を表示します。 すべてのパッケージがその npm パッケージ ページで同じ認証パッケージを使用している場合を除き、認証パッケージとコードを組み合わせないでください。 

## <a name="azure-identity-library"></a>Azure ID ライブラリ

Azure ID ライブラリは、Azure 用の最新の認証パッケージです。 ご使用のクライアント ライブラリの Readme で、この新しいライブラリの使用がサポートされているかどうかを確認してください。

[@azure/identity](https://www.npmjs.com/package/@azure/identity) ライブラリにより、Azure SDK ライブラリの Azure Active Directory に対する認証が簡略化されます。 これには、API 要求を認証するために SDK ライブラリに渡すことができる TokenCredential 実装のセットが用意されています。 これは、Azure Active Directory サービス プリンシパルまたはマネージド ID を使用するトークン認証をサポートしています。

```javascript
const { DefaultAzureCredential } = require("@azure/identity");
const { BlobServiceClient } = require("@azure/storage-blob");
 
// Enter your storage account name
const account = "<account>";
const defaultAzureCredential = new DefaultAzureCredential();
 
const blobServiceClient = new BlobServiceClient(
  `https://${account}.blob.core.windows.net`,
  defaultAzureCredential
);
```

上記の JavaScript コード例では、Azure ID ライブラリを使用して既定の Azure 資格情報を作成し、この資格情報を使用して Azure Storage リソースにアクセスする方法を示しています。

## <a name="azure-ms-rest--libraries"></a>Azure ms-rest-* ライブラリ
最新の `@azure` スコープの[クライアント ライブラリ](../azure-sdk-library-package-index.md#modern-javascripttypescript-libraries)では、サービスを使用するためにトークンが必要です。 トークンを取得するには、資格情報を返す Azure SDK クライアント認証方法を使用します。 

```javascript
const msRestNodeAuth = require("@azure/ms-rest-nodeauth");
msRestNodeAuth.interactiveLogin().then((credential) => {
    // service code goes here
}).catch((err) => {
    // error code goes here
    console.error(err);
});
```

上記の JavaScript コード例では、最新の Azure 認証ライブラリを使用して対話型ログインを行い、資格情報を取得する方法を示しています。

```javascript
// service code - this is an example only and not best practices for code flow
const { BlobServiceClient } = require('@azure/storage-blob');
const billingManagementClient = new billing.BillingManagementClient(credential, subscriptionId);
billingManagementClient.enrollmentAccounts.list().then((enrollmentList) => {
    console.log("The result is:");
    console.log(result);
})
```

上記の JavaScript コード例では、次のコード サンプルで使用されている Storage サービスなど、特定の Azure サービス クライアント ライブラリにその資格情報を渡す方法を示しています。 クライアント ライブラリは資格情報を受け取って、トークンを生成します。 サービスでは、トークンを使用して、要求に対するサービス レベルの認証が検証されます。 

クライアント ライブラリはトークンを管理し、トークンを更新するタイミングを認識します。 コード ベースを使用する開発者は、これを管理する必要がありません。

## <a name="older-azure-sdk-client-authentication"></a>古い Azure SDK クライアント認証 

古い Azure SDK クライアントは最終的に、上記で使用した最新式の認証に移行します。 その移行まで、古いクライアント ライブラリでは、異なる認証クライアントが使用されます。または、リソース キーなどの完全に別のメカニズムを備えた認証が使用される可能性があります。 

古いクライアント ライブラリを使用して最適な結果を得るためには: 
* 各 npm パッケージは、その正確なクライアント ライブラリの認証を表示します。  
* 現在のコードで、最新の `@azure/ms-*` ライブラリと以前の認証ライブラリが同じコード ベースで使用されている場合は、次のようになります。
    * サービスでは、以前の azure 以外のスコープが指定されたライブラリが最新のものであることを確認します。 この点については、サービスのドキュメントを参照してください。 
    * 最新の認証ライブラリと以前の認証ライブラリを引き続き使用する必要がある場合は、資格情報の有効期限を指定し、以前のライブラリを更新してコード ベースのアプリケーション ロジックを一致させる必要があります。 

## <a name="authentication-with-azure-services-while-developing"></a>開発中の Azure サービスでの認証

開発中に必要な資格情報を作成する一般的な方法:

| Azure 認証の種類|目的|
|--|--|
|**サービス プリンシパル**|この認証は "_推奨されている方法_" です。 [Azure サービス プリンシパルを作成する](node-sdk-azure-authenticate-principal.md)方法をご覧ください。 サービス プリンシパルを使用すると、個人用の Azure アカウントとは別の Azure に接続できます。 これは、一時的なアカウントでも、個人用アカウントの代わりとなる長期存続アカウントでもかまいません。|
| **対話**| これは、Azure サービスを試す場合の最も簡単な認証方法です。 ブラウザーで個人用アカウントを使用してログインする必要があります。 |
|**Basic**|この認証では、個人のユーザー名とパスワードを入力する必要があります。 これは安全性が最も低い方法であり、推奨されません。| 

## <a name="authentication-with-azure-services-and-production-code"></a>Azure サービスと運用コードを使用した認証

運用コードで必要な資格情報を作成るための一般的な方法:

|Azure 認証の種類|目的|
|--|--|
|**管理対象サービス ID (MSI)**|[MSI 認証](/azure/active-directory/managed-identities-azure-resources/overview)は、運用シナリオに最適です。 これは、ローカル開発環境では使用されません。 MSI がサポートされる[サービス](/azure/active-directory/managed-identities-azure-resources/services-support-managed-identities)。|
|**証明書**|[証明書](/azure/cloud-services/cloud-services-certs-create)は、[portal](/azure/cloud-services/cloud-services-configure-ssl-certificate-portal) を使用して Azure にアップロードする必要があります。|

## <a name="javascript-authentication-samples-for-azure"></a>Azure 用の JavaScript 認証サンプル

|認証パッケージ|認証スクリプトのサンプル|
|--|--|
|[@azure/ms-rest-nodeauth](https://www.npmjs.com/package/@azure/ms-rest-nodeauth) <br>(推奨)|[証明書を使用したサービス プリンシパル](https://github.com/Azure/ms-rest-nodeauth/blob/master/samples/authFileWithSpCert.ts)<br>[ファイルからのサービス プリンシパル](https://github.com/Azure/ms-rest-nodeauth/blob/master/samples/authFileWithSpSecret.ts)<br>[対話](https://github.com/Azure/ms-rest-nodeauth/blob/master/samples/interactivePersonalAccount.ts)<br>[Basic](https://github.com/Azure/ms-rest-nodeauth/blob/master/samples/usernamePassword.ts)|
|[@azure/ms-rest-browserauth](https://www.npmjs.com/package/@azure/ms-rest-browserauth)<br>(推奨)|[ポップアップを使用した認証 (create-react-app)](https://github.com/Azure/ms-rest-browserauth/tree/master/samples/authentication-with-popup)<br>[ポップアップを使用しない react](https://github.com/Azure/ms-rest-browserauth/tree/master/samples/react-app)<br>[[ログイン] ボタンを使用した HTML](https://github.com/Azure/ms-rest-browserauth/tree/master/samples/vanilla)|
|[ms-rest-azure](https://www.npmjs.com/package/ms-rest-azure)|[サービス プリンシパル](https://github.com/Azure/azure-sdk-for-node/blob/master/Documentation/Authentication.md#service-principal-authentication)<br>[対話](https://github.com/Azure/azure-sdk-for-node/blob/master/Documentation/Authentication.md#interactive-login)<br>[Basic](https://github.com/Azure/azure-sdk-for-node/blob/master/Documentation/Authentication.md#basic-authentication)|

## <a name="next-steps"></a>次のステップ   

* [Azure npm パッケージ](../azure-sdk-library-package-index.md)
* [Azure npm パッケージのドキュメント](/javascript/api/overview/azure/)
