---
title: カスタム ドメイン名を Web アプリに追加する
description: Azure CLI を使用して、カスタム ドメイン名を Azure Web アプリに追加します。
ms.topic: how-to
ms.date: 01/29/2021
ms.custom: devx-track-js, devx-track-azurecli
ms.openlocfilehash: 977376fda6e7c93390c45196ae5baae751f5ad64
ms.sourcegitcommit: 3f8aa923e4626b31cc533584fe3b66940d384351
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/01/2021
ms.locfileid: "99230190"
---
# <a name="configuring-a-custom-domain-name"></a>カスタム ドメイン名の構成

Azure CLI を使用して、カスタム ドメイン名を Azure Web アプリに追加します。 

アプリ サービスには、`YOUR-RESOURCE-NAME.azurewebsites.net` の形式で、テストに適した便利な DNS 名が用意されています。 いずれかの時点でカスタム ドメイン名を Web アプリに追加することもできます。 

## <a name="purchase-a-domain-name-and-configure-dns-record"></a>ドメイン名を購入して DNS レコードを構成する

1. ドメイン名レジストラーからドメイン名を購入します。 
1. DNS レコードの場合は、Web アプリの外部 IP (実際にはロード バランサー) を指す `A` レコードを DNS レコードに追加します。 外部 IP を取得するには、次のセクションの手順を使用します。

    `A` レコードを追加することに加え、これまで使用してきた `*.azurewebsites.net` ドメインを指す `TXT` レコードをドメインに追加する必要があります。 Azure では、ユーザーがドメインを所有していることが `A` レコードと `TXT` レコードの組み合わせによって確認されます。

## <a name="get-web-app-external-ip"></a>Web アプリの外部 IP を取得する

この IP は、次のコマンドを実行することで取得できます。

```azurecli
az webapp config hostname get-external-ip --name
```

<a name="register-a-domain-name-with-your-azure-app"></a>

## <a name="configure-web-app-domain-name"></a>Web アプリのドメイン名を構成する 

これらのレコードが作成されて DNS の変更が反映されたら、そのカスタム ドメインを Azure に登録して、受信トラフィックを正しく予測できるようにしてください。

[az webapp config hostname add](/cli/azure/webapp/config/hostname) コマンドを使用します。

```azurecli
az webapp config hostname add \
    --hostname YOUR-DOMAIN-NAME
    --webapp-name YOUR-WEBAPP-NAME
    --resource-group YOUR-RESOURCE-GROUP-NAME
```

> [!NOTE]
> このコマンドは、DNS の変更が反映されるまで正しく機能しません。

ブラウザーを開いてカスタム ドメインにアクセスし、Azure 上にデプロイされているアプリへと解決されることを確かめましょう。

## <a name="next-steps"></a>次のステップ

* [コンテナー レジストリ リソースを作成する](create-container-registry-resource.md)