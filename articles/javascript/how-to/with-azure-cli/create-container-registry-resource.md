---
title: カスタム コンテナー レジストリを作成する
description: コンテナー レジストリは、Azure にデプロイするコンテナー イメージに適しています。 このレジストリを使用すると、コンテナーのリポジトリおよびバージョンを管理できます。
ms.topic: how-to
ms.date: 01/28/2021
ms.custom: devx-track-js, devx-track-azurecli
ms.openlocfilehash: c4f02f04203b467cf166e07be16d5884c9a4e8d4
ms.sourcegitcommit: 576c878c338d286060010646b96f3ad0fdbcb814
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102118534"
---
# <a name="create-and-use-container-registry"></a>コンテナー レジストリを作成および使用する

コンテナー レジストリは、Azure にデプロイするコンテナー イメージに適しています。

このレジストリを使用すると、コンテナーのリポジトリおよびバージョンを管理できます。  

## <a name="create-a-container-registry"></a>コンテナー レジストリの作成

リソース名を使用してレジストリを作成します。 リソース名は、リソースのログイン サーバー名の一部になります。 

[az acr create](/cli/azure/acr#az_acr_create) コマンドを使用して レジストリを作成します。 

```azurecli
az acr create \
    --resource-group YOUR-RESOURCE-GROUP
    --name YOUR-REGISTRY-NAME 
    --location westus 
    --admin-enabled
    --sku Basic
    --public-network-enabled false
```

## <a name="get-container-registry-credentials"></a>コンテナー レジストリの資格情報を取得する

資格情報を取得するには、[az acr credential show](/cli/azure/acr/credential#az_acr_credential_show) コマンドを実行し、表示されたユーザー名とパスワードをメモします。

```azurecli
az acr credential show \
    --resource-group YOUR-RESOURCE-GROUP \
    --name YOUR-REGISTRY-NAME
```

## <a name="login-to-container-registry-with-docker-cli"></a>Docker CLI を使用してコンテナー レジストリにログインする

レジストリには、前の手順で取得した資格情報と、個々に割り当てられたログイン サーバーを使い、標準的な Docker CLI ワークフローでログインすることができます。

```bash
docker login YOUR-LOGIN_SERVER \
    --username USERNAME
    --password PASSWORD
```

## <a name="tag-your-local-image"></a>ローカル イメージにタグを付ける

Docker コンテナーにタグ付けし、プライベート レジストリに関連付けられていることを示すには、次のコマンドを使用します。`YOURALIAS/IMAGENAME` には、実際のコンテナー イメージに付けた名前を指定してください。

```bash
docker tag YOURALIAS/IMAGENAME \
    YOUR-LOGIN_SERVER/YOURALIAS/IMAGENAME:v1
```

## <a name="push-your-local-image-to-your-container-registry"></a>ローカル イメージをコンテナー レジストリにプッシュする

```bash
docker push YOUR-LOGIN_SERVER/YOURALIAS/IMAGENAME:v1
```

## <a name="configure-web-app-to-use-container"></a>コンテナーを使用するように Web アプリを構成する 

レジストリからイメージをプルするように App Service Web アプリを構成するには、次の [az appservice web config container set](/cli/azure/webapp/config/container#az_webapp_config_container_set) コマンドを実行します。

```azurecli
az appservice web config container set \
    --resource-group YOUR-RESOURCE-GROUP \
    --name YOUR-WEBAPP-NAME
    --docker-registry-server-url YOUR-LOGIN_SERVER \
    --docker-custom-image-name YOUR-LOGIN_SERVER/YOURALIAS/IMAGENAME:v1 \
    -u USERNAME \
    -p PASSWORD
```

`--docker-registry-server-url` オプションの先頭には、必ず `https://` プレフィックスを追加してください。 一方、コンテナー イメージの名前にはプレフィックスを追加しないでください。

## <a name="next-steps"></a>次のステップ

* [MongoDB 用 Cosmos DB リソースを作成する](create-mongodb-cosmosdb.md)