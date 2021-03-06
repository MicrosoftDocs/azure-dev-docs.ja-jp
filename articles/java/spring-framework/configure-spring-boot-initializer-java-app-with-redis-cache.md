---
title: Spring Boot Initializer アプリを作成する - Azure Redis Cache
description: Spring Initializer で作成された Spring Boot アプリケーションを、Azure Redis Cache によってクラウドで Redis を使用するように構成します。
services: redis-cache
documentationcenter: java
ms.date: 10/13/2020
ms.service: cache
ms.tgt_pltfrm: cache-redis
ms.topic: conceptual
ms.custom: devx-track-java
ms.openlocfilehash: 7d8ee875339adb741fbddeba6d4328eb22cd3e46
ms.sourcegitcommit: 5c7f5fef798413b1a304cc9ee31c8518b73f27eb
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/30/2020
ms.locfileid: "93066277"
---
# <a name="configure-a-spring-boot-initializer-app-to-use-redis-in-the-cloud-with-azure-redis-cache"></a>Azure Redis Cache によってクラウドで Redis を使用するように Spring Boot Initializer アプリを構成する

この記事では、Azure Portal を使用したクラウドでの Redis Cache の作成、 **[Spring Initializr]** を使用したカスタム アプリケーションの作成、Redis Cache を使用してデータを保存および取得する Java Web アプリケーションの作成について説明します。

## <a name="prerequisites"></a>前提条件

この記事の手順を実行するには、次の前提条件を満たす必要があります。

* Azure サブスクリプション。Azure サブスクリプションをまだお持ちでない場合は、[MSDN サブスクライバーの特典]を有効にするか、または[無料の Azure アカウント]にサインアップできます。
* サポートされている Java Development Kit (JDK)。 Azure での開発時に使用可能な JDK の詳細については、<https://aka.ms/azure-jdks> を参照してください。
* [Apache Maven](http://maven.apache.org/) バージョン 3.0 以降。

## <a name="create-a-custom-application-using-the-spring-initializr"></a>Spring Initializr を使用してカスタム アプリケーションを作成する

1. [https://www.microsoft.com](<https://start.spring.io/>) を参照します。

1. **Java** で **Maven** プロジェクトを生成することを指定して、Java バージョン **8** を選択し、アプリケーションの **[グループ]** と **[アーティファクト]** の名前を入力します。

1. **Spring Web** セクションの依存関係を追加し、 **Web** のチェックボックスをオンにします。次に、 **NoSQL** セクションまで下にスクロールし、 **[Spring Data Reactive Redis]\(Spring Data Reactive Redis\)** のチェックボックスをオンにします。 
1. ページの下部までスクロールし、 **[Generate Project]\(プロジェクトの生成\)** をクリックします。

   ![基本的な Spring Initializr オプション][SI01]

   > [!NOTE]
   >
   > Spring Initializr では、 **[Group]\(グループ\)** と **[Aritifact]\(アーティファクト\)** の名前を使用してパッケージ名を作成します (例: *com.contoso.myazuredemo* )。
   >

1. メッセージが表示されたら、ローカル コンピューター上のパスにプロジェクトをダウンロードします。

   ![カスタム Spring Boot プロジェクトのダウンロード][SI03]

1. ファイルをローカル システム上に展開したら、カスタム Spring Boot アプリケーションの編集を開始できます。

   ![カスタム Spring Boot プロジェクト ファイル][SI04]

## <a name="create-a-redis-cache-on-azure"></a>Azure で Redis Cache を作成する

1. <https://portal.azure.com/> で Azure Portal を参照し、 **[+新規]** をクリックします。

1. **[データベース]** をクリックし、 **[Redis Cache]** をクリックします。

   ![Azure portal で [Redis Cache] を選択します。][AZ02]

1. **[新規 Redis Cache]** ページで、以下の情報を指定します。

   * キャッシュの **[DNS 名]** を入力します。
   * **[サブスクリプション]** 、 **[リソース グループ]** 、 **[場所]** 、および **[価格レベル]** を指定します。
   * このチュートリアルでは、 **[ポート 6379 のブロックを解除]** を選択します。

   > [!NOTE]
   >
   > Redis キャッシュで SSL を使用できますが、Jedis のような異なる Redis クライアントを使用する必要があります。 詳細については、[Java で Azure Redis Cache を使用する方法][Redis Cache with Java]に関するページを参照してください。
   >

   これらのオプションの指定後、 **[作成]** をクリックしてキャッシュを作成します。

   ![Azure portal でキャッシュを作成します。][AZ03]

1. キャッシュが作成されると、Azure の **ダッシュボード** のほか、 **[すべてのリソース]** ブレードと **[Redis Cach]** ページにも作成したキャッシュが表示されます。 これらのいずれかの場所でキャッシュをクリックすると、そのキャッシュのプロパティ ページを開くことができます。

   ![Azure portal 内のプロビジョニングされたリソース。][AZ04]

1. キャッシュのプロパティの一覧が含まれているページが表示されたら、 **[アクセス キー]** をクリックし、キャッシュのアクセス キーをコピーします。

   ![[アクセス キー] セクションでアクセス キーをコピーします。][AZ05]

## <a name="configure-your-custom-spring-boot-to-use-your-redis-cache"></a>Redis Cache を使用するようにカスタム Spring Boot を構成する

1. アプリの *resources* ディレクトリ内で *application.properties* ファイルを探すか、まだ存在しない場合はファイルを作成します。

   ![application.properties ファイルを探す][RE01]

1. テキスト エディターで *application.properties* ファイルを開き、そのファイルに次の行を追加し、サンプルの値を実際のキャッシュの適切なプロパティに置き換えます。

   ```yaml
   # Specify the DNS URI of your Redis cache.
   spring.redis.host=myspringbootcache.redis.cache.windows.net

   # Specify the port for your Redis cache.
   spring.redis.port=6379

   # Specify the access key for your Redis cache.
   spring.redis.password=57686f6120447564652c2049495320526f636b73=
   ```

   ![application.properties ファイルの編集][RE02]

   > [!NOTE] 
   > 
   > SSL を有効にする Jedis のような異なる Redis クライアントを使用している場合は、SSL 使用する旨を *application.properties* ファイルで指定し、ポート 6380 を使用します。 次に例を示します。
   > 
   > ```yaml
   > # Specify the DNS URI of your Redis cache.
   > spring.redis.host=myspringbootcache.redis.cache.windows.net
   > # Specify the access key for your Redis cache.
   > spring.redis.password=57686f6120447564652c2049495320526f636b73=
   > # Specify that you want to use SSL.
   > spring.redis.ssl=true
   > # Specify the SSL port for your Redis cache.
   > spring.redis.port=6380
   > ```
   > 
   > 詳細については、[Java で Azure Redis Cache を使用する方法][Redis Cache with Java]に関するページを参照してください。 
   > 

1. *application.properties* ファイルを保存して閉じます。

1. パッケージのソース フォルダーの下に *controller* という名前のフォルダーを作成します。次に例を示します。

   `C:\SpringBoot\myazuredemo\src\main\java\com\contoso\myazuredemo\controller`

   または

   `/users/example/home/myazuredemo/src/main/java/com/contoso/myazuredemo/controller`

1. *HelloController.java* という名前の新しいファイルを *controller* フォルダー内に作成します。 テキスト エディターでそのファイルを開き、次の行を追加します。

   ```java
   package com.contoso.myazuredemo;

   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.data.redis.core.StringRedisTemplate;
   import org.springframework.data.redis.core.ValueOperations;

   @RestController
   public class HelloController {
   
      @Autowired
      private StringRedisTemplate template;

      @RequestMapping("/")
      // Define the Hello World controller.
      public String hello() {
      
         ValueOperations<String, String> ops = this.template.opsForValue();

         // Add a Hello World string to your cache.
         String key = "greeting";
         if (!this.template.hasKey(key)) {
             ops.set(key, "Hello World!");
         }

         // Return the string from your cache.
         return ops.get(key);
      }
   }
   ```
   
   ここでは、`com.contoso.myazuredemo` を実際のプロジェクトのパッケージ名に置き換える必要があります。

1. *HelloController.java* ファイルを保存して閉じます。

1. Spring Boot アプリケーションを Maven でビルドし、実行します。次に例を示します。

   ```shell
   mvn clean package
   mvn spring-boot:run
   ```

1. Web ブラウザーを使用して `http://localhost:8080` を参照することによって Web アプリをテストするか、または curl が使用可能な場合は次の例のような構文を使用します。

   ```shell
   curl http://localhost:8080
   ```

   "Hello World!" というメッセージがサンプル コントローラーから表示されることがわかります。これは、Redis cache から動的に取得されます。

## <a name="next-steps"></a>次のステップ

Spring および Azure の詳細については、Azure ドキュメント センターで引き続き Spring に関するドキュメントをご確認ください。

> [!div class="nextstepaction"]
> [Azure の Spring](./index.yml)

### <a name="additional-resources"></a>その他のリソース

Azure での Spring Boot アプリケーションの使用の詳細については、次の記事を参照してください。

* [Spring Boot アプリケーションを Azure App Service の Linux にデプロイする](deploy-spring-boot-java-app-on-linux.md)

* [Running a Spring Boot Application on a Kubernetes Cluster in the Azure Container Service (Azure Container Service での Kubernetes クラスター上の Spring Boot アプリケーションの実行)](deploy-spring-boot-java-app-on-kubernetes.md)

Java での Azure の使用の詳細については、「[Java 開発者向けの Azure]」および「[Azure DevOps と Java の操作]」を参照してください。

Azure 上の Java での Redis Cache の使用開始について詳しくは、[Java で Azure Redis Cache を使用する方法][Redis Cache with Java]に関するページを参照してください。

**[Spring Framework]** は Java 開発者のエンタープライズ レベルのアプリケーション作成を支援するオープンソース ソリューションです。 このプラットフォームで構築される特に知られたプロジェクトの 1 つが [Spring Boot] です。これによって、スタンドアロンの Java アプリケーションの作成方法が簡略化されます。 Spring Boot を使い始めた開発者を支援するために、<https://github.com/spring-guides/> では、サンプルの Spring Boot パッケージがいくつか用意されています。 基本的な Spring Boot プロジェクトの一覧から選択するだけでなく、 **[Spring Initializr]** は、開発者がカスタム Spring Boot アプリケーションの作成を開始できるように支援します。

<!-- URL List -->

[Java 開発者向けの Azure]: ../index.yml
[無料の Azure アカウント]: https://azure.microsoft.com/pricing/free-trial/
[Azure DevOps と Java の操作]: /azure/devops/java/
[MSDN サブスクライバーの特典]: https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/
[Spring Boot]: http://projects.spring.io/spring-boot/
[Spring Initializr]: https://start.spring.io/
[Spring Framework]: https://spring.io/
[Redis Cache with Java]: /azure/redis-cache/cache-java-get-started

<!-- IMG List -->

[AZ01]: media/configure-spring-boot-initializer-java-app-with-redis-cache/AZ01.png
[AZ02]: media/configure-spring-boot-initializer-java-app-with-redis-cache/AZ02.png
[AZ03]: media/configure-spring-boot-initializer-java-app-with-redis-cache/AZ03.png
[AZ04]: media/configure-spring-boot-initializer-java-app-with-redis-cache/AZ04.png
[AZ05]: media/configure-spring-boot-initializer-java-app-with-redis-cache/AZ05.png

[SI01]: media/configure-spring-boot-initializer-java-app-with-redis-cache/SI01.png
[SI02]: media/configure-spring-boot-initializer-java-app-with-redis-cache/SI02.png
[SI03]: media/configure-spring-boot-initializer-java-app-with-redis-cache/SI03.png
[SI04]: media/configure-spring-boot-initializer-java-app-with-redis-cache/SI04.png

[RE01]: media/configure-spring-boot-initializer-java-app-with-redis-cache/RE01.png
[RE02]: media/configure-spring-boot-initializer-java-app-with-redis-cache/RE02.png
