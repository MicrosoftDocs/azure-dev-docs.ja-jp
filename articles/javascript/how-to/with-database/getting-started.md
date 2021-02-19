---
title: Azure データベースの概要
description: Azure でホストされている任意のデータベースを使用するための一般的なタスクについて説明します。
ms.topic: how-to
ms.date: 02/09/2021
ms.custom: devx-track-js
ms.openlocfilehash: 5d104dcbfe6aa89db6ee434fafcb2f4796de1b18
ms.sourcegitcommit: b380f6e637b47e6e3822b364136853e1d342d5cd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/14/2021
ms.locfileid: "100424201"
---
# <a name="getting-started-with-databases-on-azure"></a>Azure のデータベースの概要

Azure クラウド プラットフォームでは、任意の Azure データベースを (サービスとして) 使用することも、独自のデータベースを使用することもできます。 サーバーとデータベースをセットアップしたら、既存のコードで接続設定を変更するだけで済みます。 

Azure でデータベースを使用する場合、JavaScript アプリからデータベースを使用するために実行する必要がある一般的なタスクがいくつかあります。 Azure でのデータベースの取得と使用について詳しく説明します。 

## <a name="select-a-database-to-use-on-azure"></a>Azure で使用するデータベースを選択する

Microsoft では、次のデータベース用のマネージド サービスを提供しています。

|データベース|Azure サービス|詳細情報|
|--|--|--|
|Cassandra|[Azure Cosmos DB](/azure/cosmos-db/)|[クイック スタート:Node.js SDK と Azure Cosmos DB を使用して Cassandra アプリを構築する](/azure/cosmos-db/create-cassandra-nodejs)|
|Gremlin|[Azure Cosmos DB](/azure/cosmos-db/)|[クイック スタート:Azure Cosmos DB Gremlin API アカウントを使用して Node.js アプリケーションをビルドする](/azure/cosmos-db/create-graph-nodejs)|
|MongoDB|[Azure Cosmos DB](/azure/cosmos-db/)|[Azure で MongoDB を使用して JavaScript アプリケーションを開発する方法](use-mongodb-as-cosmosdb.md)<br>[クイック スタート:既存の MongoDB Node.js Web アプリを Azure Cosmos DB に移行する](/azure/cosmos-db/create-mongodb-nodejs)|
|MariaDB|[Azure Database for MariaDB](/azure/mariadb/)|[Azure で MariaDB を使用して JavaScript アプリケーションを開発する方法](use-mariadb.md)|
|MySQL|[Azure Database for MySQL](/azure/mysql/)|[クイック スタート:Node.js を使用して Azure Database for MySQL に接続してデータを照会する](/azure/mysql/connect-nodejs)<br>[Azure で MySQL を使用して JavaScript アプリケーションを開発する方法](use-mysql-db.md)|
|PostgreSQL|[Azure Database for PostgreSQL](/azure/postgresql/)|[クイック スタート:Node.js を使用して Azure Database for PostgreSQL - Single Server に接続し、データにクエリを実行する](/azure/postgresql/connect-nodejs)<br>[Azure で PostgreSQL を使用して JavaScript アプリケーションを開発する方法](use-postgresql-db.md)|
|Redis|[Azure Cache for Redis](/azure/azure-cache-for-redis/)|[クイックスタート: Node.js で Azure Cache for Redis を使用する](/azure/azure-cache-for-redis/cache-nodejs-get-started)|
|SQL|[Azure Cosmos DB](/azure/cosmos-db/)|[クイック スタート:Node.js を使用して Azure Cosmos DB の SQL API アカウントに接続してデータを照会する](/azure/cosmos-db/create-sql-api-nodejs)|
|テーブル|[Azure Cosmos DB](/azure/cosmos-db/)|[Node.js から Azure Table Storage または Azure Cosmos DB Table API を使用する方法](/azure/cosmos-db/table-storage-how-to-use-nodejs)|

**選択する上でヘルプが必要な場合** 
* [必要な操作](https://azure.microsoft.com/product-categories/databases/)に基づいてデータベースを選択します
* [Azure Database Migration Service](/azure/dms/) を使用して Azure に移行します。 

**データベースが見つからなかった場合:**
データベースをコンテナーまたは仮想マシンのいずれかとして配置します。 これらのサービを使用する任意の種類のデータベースを配置でき、他の Azure リソースに対する高可用性とセキュリティを実現できます。 トレードオフは、インフラストラクチャ (コンテナーまたは VM) を自分で管理する必要があることです。 このドキュメントの残りの部分は、コンテナーまたは VM の使用に役立てることができますが、Azure データベース サービスを選択する上でさらに役立ちます。 

## <a name="create-the-server"></a>サーバーを作成する

サーバーの作成は、データベースがホストされているサブスクリプションの特定の Azure サービスのリソースを作成すると完了します。 

リソースを作成するには次のようにします。

|ツール|目的|
|--|--|
|Azure portal|めったに作成されないデータベースまたは最初の場合には、Azure portal を使用します。|
|Azure CLI|反復可能またはスクリプトの作成が可能なシナリオに使用します。|
|Visual Studio Code 拡張機能 (対象のサービス用)|開発 IDE 内に留まる場合に使用します。|
|npm ARM ライブラリ (対象のサービス用)|JavaScript 言語内に留まる場合に使用します。| 

サーバーを作成した後、サービスによっては、次のことが必要になる場合があります。

* ファイアウォールや SSL 強制などのセキュリティ設定を構成する
* 接続情報を取得する
* データベースの作成

## <a name="configure-security-settings-for-your-database"></a>データベースのセキュリティ設定を構成する

サービスに対して構成する一般的なセキュリティ設定は次のとおりです。

* クライアント IP アドレスに対してファイアウォールを開く
* SSL 強制を構成する
* パブリック要求を受け入れるか、またはすべての要求が別の Azure サービスから来ることを要求する

## <a name="create-a-database-on-the-azure-server"></a>Azure サーバーでデータベースを作成する

サーバーを作成したときと同じツールを使用して、接続情報を取得できます。 接続情報を使用してサーバーにアクセスします。 やはり、アプリケーションに固有のデータベースを作成する必要があります。 

サーバーにアクセスする方法: 

* pgAdmin、SQL Server Management Studio、MySQL Workbench など、そのデータベースの種類に固有のツールを使用します。 
* 引き続き Microsoft ツールを使用します
    * [Azure Cloud Shell](https://shell.azure.com) には、psql や mysql などの多くのデータベース CLI が含まれています。
    * Visual Studio Code 拡張機能
    * JavaScript 用 npm パッケージ
    * Azure portal

## <a name="programmatically-access-the-server-and-database-with-javascript"></a>JavaScript を使用してサーバーとデータベースにプログラムでアクセスする

接続情報を取得したら、業界標準の npm パッケージと JavaScript を使用してサーバーにアクセスできます。 

データベースを作成または移行した後は、新しいサーバーおよびデータベースへの接続情報のみを変更する必要があります。 

## <a name="configure-an-azure-web-apps-connection-to-database"></a>データベースに対する Azure Web アプリの接続を構成する

Azure Web アプリがデータベースに接続する場合は、接続情報のアプリ設定を変更する必要があります。 

## <a name="next-steps"></a>次のステップ

* [Azure で MongoDB を使用して JavaScript アプリケーションを開発する](use-mongodb-as-cosmosdb.md)