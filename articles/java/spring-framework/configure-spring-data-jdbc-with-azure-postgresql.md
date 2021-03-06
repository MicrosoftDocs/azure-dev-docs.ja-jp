---
title: Azure Database for PostgreSQL で Spring Data JDBC を使用する
description: Azure Database for PostgreSQL データベースで Spring Data JDBC を使用する方法を説明します。
documentationcenter: java
ms.date: 10/13/2020
ms.service: postgresql
ms.tgt_pltfrm: multiple
ms.author: judubois
ms.topic: article
ms.custom: devx-track-java
ms.openlocfilehash: a27b8122b3758e997cf5d7595cfd246084acf071
ms.sourcegitcommit: 709fa38a137b30184a7397e0bfa348822f3ea0a7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/01/2020
ms.locfileid: "96441927"
---
# <a name="use-spring-data-jdbc-with-azure-database-for-postgresql"></a>Azure Database for PostgreSQL で Spring Data JDBC を使用する

このトピックでは、[Spring Data JDBC](https://spring.io/projects/spring-data-jdbc) を使って [Azure Database for PostgreSQL](/azure/postgresql/) データベースで情報を格納および取得するサンプル アプリケーションを作成する方法を説明します。

[JDBC](https://en.wikipedia.org/wiki/Java_Database_Connectivity) は、従来のリレーショナル データベースに接続するための標準の Java API です。

[!INCLUDE [spring-data-prerequisites.md](includes/spring-data-prerequisites.md)]

## <a name="sample-application"></a>サンプル アプリケーション

この記事では、サンプル アプリケーションをコーディングします。 より早く進めたい場合は、このアプリケーションは既にコーディングされており、[https://github.com/Azure-Samples/quickstart-spring-data-jdbc-postgresql](https://github.com/Azure-Samples/quickstart-spring-data-jdbc-postgresql) で入手できます。

[!INCLUDE [spring-data-postgresql-setup.md](includes/spring-data-postgresql-setup.md)]

### <a name="generate-the-application-by-using-spring-initializr"></a>Spring Initializr を使用してアプリケーションを生成する

以下のコマンドを使用して、コマンド ラインでアプリケーションを生成します。

```bash
curl https://start.spring.io/starter.tgz -d dependencies=web,data-jdbc,postgresql -d baseDir=azure-database-workshop -d bootVersion=2.3.4.RELEASE -d javaVersion=8 | tar -xzvf -
``` 
 
### <a name="configure-spring-boot-to-use-azure-database-for-postgresql"></a>Azure Database for PostgreSQL を使用するように Spring Boot を構成する

*src/main/resources/application.properties* ファイルを開き、以下のテキストを追加します。

```properties
logging.level.org.springframework.jdbc.core=DEBUG

spring.datasource.url=jdbc:postgresql://$AZ_DATABASE_NAME.postgres.database.azure.com:5432/demo
spring.datasource.username=spring@$AZ_DATABASE_NAME
spring.datasource.password=$AZ_POSTGRESQL_PASSWORD

spring.datasource.initialization-mode=always
```

2 つの `$AZ_DATABASE_NAME` 変数と、`$AZ_POSTGRESQL_PASSWORD` 変数を、この記事の冒頭で構成した値に置き換えます。

> [!WARNING]
> 構成プロパティ `spring.datasource.initialization-mode=always` は、サーバーが起動されるたびに、*schema.sql* ファイル (後ほど作成します) を使用して、Spring Boot がデータベース スキーマを自動的に生成することを意味します。 これはテストには適していますが、再起動するたびにデータが削除されるため、運用環境では使用しないでください。

これで、次のように、提供されている Maven Wrapper を使用してアプリケーションを起動できるはずです。

```bash
./mvnw spring-boot:run
```

アプリケーションを初めて実行したときのスクリーンショットを次に示します。

[![実行中のアプリケーション](media/configure-spring-data-jdbc-with-azure-postgresql/create-postgresql-01.png)](media/configure-spring-data-jdbc-with-azure-postgresql/create-postgresql-01.png#lightbox)

### <a name="create-the-database-schema"></a>データベース スキーマを作成する

Spring Boot は、データベース スキーマを作成するために *src/main/resources/schema.sql* ファイルを自動的に実行します。 このファイルを作成し、次の内容を追加します。

```sql
DROP TABLE IF EXISTS todo;
CREATE TABLE todo (id SERIAL PRIMARY KEY, description VARCHAR(255), details VARCHAR(4096), done BOOLEAN);
```

次のコマンドを使用して、実行中のアプリケーションを停止して再起動します。 これで、アプリケーションは、先ほど作成した `demo` データベースを使用し、その中に `todo` テーブルを作成します。

```bash
./mvnw spring-boot:run
```

## <a name="code-the-application"></a>アプリケーションをコーディングする

次に、JDBC を使用して PostgreSQL サーバーにデータを格納および取得する Java コードを追加します。

[!INCLUDE [spring-data-jdbc-create-application.md](includes/spring-data-jdbc-create-application.md)]

これらの cURL 要求のスクリーンショットを次に示します。

[![cURL を使用してテストする](media/configure-spring-data-jdbc-with-azure-postgresql/create-postgresql-02.png)](media/configure-spring-data-jdbc-with-azure-postgresql/create-postgresql-02.png#lightbox)

お疲れさまでした。 JDBC を使用して Azure Database for PostgreSQL にデータを格納および取得する Spring Boot アプリケーションを作成しました。

[!INCLUDE [spring-data-conclusion.md](includes/spring-data-conclusion.md)]

### <a name="additional-resources"></a>その他のリソース

Spring Data JDBC の詳細については、Spring の[リファレンス ドキュメント](https://docs.spring.io/spring-data/jdbc/docs/current/reference/html/#reference)を参照してください。

Java での Azure の使用の詳細については、「[Java 開発者向けの Azure](../index.yml)」および [Azure DevOps と Java の操作](/azure/devops/)に関するページを参照してください。