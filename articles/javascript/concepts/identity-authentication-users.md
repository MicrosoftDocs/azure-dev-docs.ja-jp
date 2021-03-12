---
title: 認証と承認 - JavaScript - Azure
description: Azure で ID、認証、ユーザーを使用して JavaScript アプリを開発する方法について説明します。
ms.topic: conceptual
ms.date: 12/09/2020
ms.custom: devx-track-js
ms.openlocfilehash: 68f53743d001b44aa1495ece36d0c2764ec749d4
ms.sourcegitcommit: 576c878c338d286060010646b96f3ad0fdbcb814
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102118464"
---
# <a name="identity-authentication-and-users"></a>ID、認証、ユーザー

認証と承認は、特定のプログラム タスクやユーザーとアプリケーションの対話に縮小できる、Web アプリケーションに関する広範なテーマです。 この記事では、JavaScript 開発者が一般に理解しておく必要がある重要な概念を中心に説明します。 

## <a name="authentication-with-azure"></a>Azure での認証

認証は、ID でオブジェクトにアクセスできるようにするための機能です。 

Azure での JavaScript 開発者の場合は、通常、次のことを行う機能を意味します。

* プログラムによる Azure リソースへのアクセスを許可する
* ユーザーが、通常はログイン後、アプリにアクセスするのを許可する

|必須|パースペクティブ|説明|
|--|--|--|
|はい|開発者|アプリケーション コードで Azure リソースにアクセスするためには、必要な資格情報を Azure に渡す必要があります。|
|いいえ|ユーザー|アプリケーションのユーザーの場合、認証は匿名にすることも、ユーザー アカウントを要求することもできます。 この制限付きアクセスには、Microsoft などの一般的な認証プロバイダーを使用することも、ユーザー用に独自の認証レイヤーを作成することもできます。|

## <a name="authentication-for-developers-to-azure-services"></a>開発者のための Azure サービスに関する認証

Azure への認証をプログラムで行うには、コードで使用されるサービスのための有効な資格情報が必要です。 サービスに関するクイックスタート ドキュメントを読み、サービスで要求されている資格情報の種類を理解する必要があります。 

### <a name="local-developer-environment-for-authenticating-to-azure"></a>Azure への認証を行うためのローカル開発者環境

サービスに接続する方法を理解したら、サービス プリンシパルを作成し、開発用コンピューターの環境変数にサービス プリンシパルを設定する必要があります。 そのステップにより、Azure との直接的なやり取りに個人用アカウントが使用されなくなり、ソース コードで資格情報をチェックインすることによって個人アカウントが侵害されるリスクが排除されます。 

### <a name="cloud-apps-authenticating-to-azure"></a>クラウド アプリでの Azure に対する認証

クラウドベースのアプリの場合、Azure ホスティング サービスにより、環境変数やシークレットなどの[アプリケーション設定](../how-to/configure-web-app-settings.md)へのアクセスが提供されます。 Web アプリに別のセキュリティ レイヤーを追加するには、シークレットを Azure [Key Vault](/azure/key-vault) に格納し、ホストされているアプリからプログラムでそれらのシークレットにアクセスします。 

## <a name="modern-programmatic-authentication-with-azureidentity"></a>最新の @azure/identity を使用したプログラムによる認証

現在の Azure SDK ライブラリでは、[@azure/identity](https://www.npmjs.com/package/@azure/identity) npm パッケージを使用した Azure サービスに対するプログラムによる認証のために、サービス プリンシパルが使用されます。 この認証によりプロセスが簡略化され、[最新の Azure SDK パッケージ](https://www.npmjs.com/package/@azure/identity#client-libraries-supporting-authentication-with-azure-identity)でそれを使用できます。 

```javascript
// The default credential first checks environment variables for configuration.
// If environment configuration is incomplete, it will try managed identity.

// Azure service to use
const { KeyClient } = require("@azure/keyvault-keys");

// Azure authentication library to access Azure service
const { DefaultAzureCredential } = require("@azure/identity");

// Azure SDK clients accept the credential as a parameter
const credential = new DefaultAzureCredential();

// Create authenticated client
const client = new KeyClient(vaultUrl, credential);

// Use service from authenticated client
const getResult = await client.getKey("MyKeyName");
```

## <a name="classic-programmatic-authentication"></a>従来のプログラムによる認証

維持されている他のほとんどの Azure SDK ライブラリの場合は、次のパッケージのいずれかを使用します。 

* [@azure/ms-rest-js](https://www.npmjs.com/package/@azure/ms-rest-js) - ブラウザーおよび Node.js 環境で動作します
* [@azure/ms-rest-nodeauth](https://www.npmjs.com/package/@azure/ms-rest-nodeauth) - 対話形式、サービス プリンシパル、ユーザーとパスワードなど、複数の異なる認証メカニズムが提供されます
* [@azure/ms-rest-browserauth](https://www.npmjs.com/package/@azure/ms-rest-browserauth) - Azure AD アプリが必要です

次の例では、サービスによって提供されるキーとエンドポイントを使用して認証を行う方法を示します。

```javascript
// Azure service to use
const { QnAMakerRuntimeClient } = require("@azure/cognitiveservices-qnamaker-runtime");

// Azure authentication library to access Azure service
const { CognitiveServicesCredentials } = require("@azure/ms-rest-azure-js");  
 
// QnA Maker runtime credentials
const QNAMAKER_KEY = process.env["QNAMAKER_KEY"];
const QNAMAKER_ENDPOINT = process.env["QNAMAKER_ENDPOINT"];
const KNOWLEDGEBASE_ID = process.env["QNAMAKER_KNOWLEDGE_BASE_ID"];

const cognitiveServicesCredentials = new CognitiveServicesCredentials(QNAMAKER_KEY);
const client = new QnAMakerRuntimeClient(cognitiveServicesCredentials, QNAMAKER_ENDPOINT);
const customHeaders = { Authorization: `EndpointKey ${QNAMAKER_KEY}` };

// A question you'd like to get a response for, from the knowledge base. For example
const question = "How are you?";

// Maximum number of answer to retreive
const top = 1;

// Find only answers that contain these metadata
const strictFilters = [{ name: "editorial", value: "chitchat" }];

client.runtime.generateAnswer( 
        KNOWLEDGEBASE_ID,
        { question, top, strictFilters },
        { customHeaders }
).then(result =>{
    console.log(JSON.stringify(result));

    // Sample Result
    // {
    //   answers: [
    //     {
    //       questions: [
    //         "How are you?",
    //         "How is your tuesday?"
    //       ],
    //       answer:
    //         ""I'm doing great, thanks for asking!",
    //       score: 100,
    //       id: 90,
    //       source:
    //         "qna_chitchat_Friendly.tsv",
    //       metadata: [{ name: "editorial", value: "chitchat" }],
    //       context: { isContextOnly: false, prompts: [] }
    //     }
    //   ],
    //   debugInfo: null,
    //   activeLearningEnabled: false
    // }

});

```

## <a name="user-authentication-with-an-app-registration"></a>アプリの登録を使用したユーザー認証

Microsoft Authentication Library (MSAL) は、Web 開発でユーザー認証を行うために推奨されるライブラリです。 このライブラリは、複数の[言語とフレームワーク](/azure/active-directory/develop/msal-overview#languages-and-frameworks)で使用できます。

MSAL を使用するには、Web アプリで Microsoft に[アプリを登録する](/azure/active-directory/develop/quickstart-register-app)必要があります。 アプリの登録には、ユーザー スコープのアクセス許可やリダイレクト URL などの一般的な認証情報が含まれます。 

サンプル プロジェクトの詳細については、こちらの [MSAL クイックスタート](/azure/active-directory/develop/quickstart-v2-javascript)を参照してください。

ユーザーは、アプリにログインするときに、アプリへのアクセス許可を付与します。 このアクセス許可はユーザーと共に保存され、管理および取り消しができます。

* コンシューマー アプリのアクセス許可管理 - [https://account.live.com/consent/manage](https://account.live.com/consent/manage)
* Active Directory アプリのアクセス許可管理 - [https://myapplications.microsoft.com/](https://myapplications.microsoft.com/)

## <a name="next-steps"></a>次の手順

* [Azure アプリ サービスを構成する](../how-to/configure-web-app-settings.md)