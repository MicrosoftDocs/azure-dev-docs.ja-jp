---
title: Azure Event Hubs を使用する Spring Cloud Stream Binder アプリケーションを作成する方法
description: Spring Boot Initializr を使用して作成された Java ベースの Spring Cloud Stream Binder アプリケーションを、Azure Event Hubs を使用するように構成する方法について説明します。
services: event-hubs
documentationcenter: java
ms.date: 02/08/2021
ms.service: event-hubs
ms.tgt_pltfrm: na
ms.topic: article
ms.custom: devx-track-java, devx-track-azurecli
ms.openlocfilehash: d0c87ce32caddc0100b91abd800a18179ba4101e
ms.sourcegitcommit: bccbab4883e6b6b4926fc194c35ad948b11ccc3f
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2021
ms.locfileid: "99822721"
---
# <a name="how-to-create-a-spring-cloud-stream-binder-application-with-azure-event-hubs"></a>Azure Event Hubs を使用する Spring Cloud Stream Binder アプリケーションを作成する方法

この記事では、Spring Boot Initializr を使用して作成された Java ベースの Spring Cloud Stream Binder アプリケーションを、Azure Event Hubs を使用するように構成する方法について説明します。

## <a name="prerequisites"></a>前提条件

* Azure サブスクリプション。Azure サブスクリプションをまだお持ちでない場合は、[MSDN サブスクライバーの特典]を有効にするか、または[無料の Azure アカウント]にサインアップできます。
* サポートされている Java Development Kit (JDK)。 Azure での開発時に使用可能な JDK の詳細については、<https://aka.ms/azure-jdks> を参照してください。
* [Apache Maven](http://maven.apache.org/) バージョン 3.0 以降。

> [!IMPORTANT]
> この記事の手順を完了するには、Spring Boot バージョン 2.2 または 2.3 が必要です。

## <a name="create-an-azure-event-hub-using-the-azure-portal"></a>Azure portal を使用して Azure イベント ハブを作成する

次の手順では、Azure イベント ハブを作成します。

### <a name="create-an-azure-event-hub-namespace"></a>Azure イベント ハブの名前空間を作成する

1. Azure portal (<https://portal.azure.com/>) を参照し、サインインします。

1. **[+ リソースの作成]** を選択し、 *[Event Hubs]* を検索します。

1. **[作成]** を選択します。

   >[!div class="mx-imgBorder"]
   >![Azure イベント ハブの名前空間を作成する][IMG01]

1. **[名前空間の作成]** ページで、次の情報を入力します。

   * 名前空間に使用する **サブスクリプション** を選択します。
   * 名前空間の新しい **リソース グループ** を作成するか、既存のリソース グループを選択するかを指定します。
   * 一意の **名前空間名** を入力します。この名前は、イベント ハブの名前空間の URI の一部になります。 たとえば、 **[名前空間名]** に「*wingtiptoys-space*」と入力した場合、URI は `wingtiptoys-space.servicebus.windows.net` になります。
   * イベント ハブの名前空間の **場所** を指定します。
   * 価格レベル。
   * 名前空間の **[スループット ユニット]** を指定することもできます。
   
   >[!div class="mx-imgBorder"]
   >![Azure イベント ハブの名前空間のオプションを指定する][IMG02]

1. 上記のオプションを指定したら、 **[確認および作成]** を選択し、指定した内容を確認し、 **[作成]** を選択して名前空間を作成します。

## <a name="create-an-azure-event-hub-in-your-namespace"></a>名前空間に Azure イベント ハブを作成する

名前空間をデプロイしたら、 **[リソースに移動]** を選択して **[Event Hubs 名前空間]** のページを開きます。ここで、名前空間にイベント ハブを作成できます。

1. 前のセクションで作成した名前空間に移動します。

1. 上部のメニュー バーの **[+ Event Hubs]** を選択します。

1. イベント ハブに名前を指定します。

1. **[作成]** を選択します。

   >[!div class="mx-imgBorder"]
   >![イベント ハブの作成][IMG05]

### <a name="create-an-azure-storage-account-for-your-event-hub-checkpoints"></a>イベント ハブのチェックポイント用の Azure ストレージ アカウントを作成する

次の手順では、イベント ハブのチェックポイント用のストレージ アカウントを作成します。

1. Azure portal (<https://portal.azure.com/>) を参照します。

1. **[+ リソースの作成]** を選択し、 **[ストレージ]** を選択して、 **[ストレージ アカウント]** を選択します。

1. **[ストレージ アカウントの作成]** ページで、次の情報を入力します。

   * ストレージ アカウントに使用する **サブスクリプション** を選択します。
   * ストレージ アカウントの新しい **リソース グループ** を作成するか、既存のリソース グループを選択するかを指定します。
   * ストレージ アカウント用に一意の **名前** を入力します。
   * ストレージ アカウントの **場所** を指定します。

   >[!div class="mx-imgBorder"]
   >![Azure ストレージ アカウントのオプションを指定する][IMG08]

1. 上記のオプションを指定したら、 **[確認および作成]** を選択してストレージ アカウントを作成します。

1. 指定した内容を確認し、 **[作成]** を選択します。  デプロイには数分かかります。

## <a name="create-a-simple-spring-boot-application-with-the-spring-initializr"></a>Spring Initializr でシンプルな Spring Boot アプリケーションを作成する

次の手順では、Spring Boot アプリケーションを作成します。

1. [https://www.microsoft.com](<https://start.spring.io/>) を参照します。

1. 次のオプションを指定します。

   * **Java** で **Maven** プロジェクトを生成します。
   * **Spring Boot** のバージョンとして、**2.3** を指定します。
   * アプリケーションの **グループ (Group)** と **成果物 (Artifact)** の名前を指定します。
   * Java バージョンとして **8** を選択します。
   * *Web* 依存関係を追加します。

   >[!div class="mx-imgBorder"]
   >![基本的な Spring Initializr オプション][SI01]

   > [!NOTE]
   > Spring Initializr では、**グループ (Group)** と **成果物 (Artifact)** の名前を使用してパッケージ名を作成します (例: *com.contoso.eventhubs.sample*)。

1. 上記のオプションを指定したら、 **[生成]** を選択します。

1. メッセージが表示されたら、ローカル コンピューター上のパスにプロジェクトをダウンロードします。

1. ファイルをローカル システム上に展開したら、シンプルな Spring Boot アプリケーションの編集を開始できます。

## <a name="configure-your-spring-boot-app-to-use-the-azure-event-hub-starter"></a>Azure Event Hub スターターを使用するように Spring Boot アプリを構成する

1. アプリのルート ディレクトリで *pom.xml* ファイルを探します。次に例を示します。

   *C:\SpringBoot\eventhubs-sample\pom.xml*

   - または -

   */users/example/home/eventhubs-sample/pom.xml*

1. テキスト エディターで *pom.xml* ファイルを開き、Spring Cloud Azure Event Hub Stream Binder スターターを `<dependencies>` のリストに追加します。

   ```xml
   <dependency>
     <groupId>com.azure.spring</groupId>
     <artifactId>azure-spring-cloud-stream-binder-eventhubs</artifactId>
     <version>2.1.0</version>
   </dependency>
   ```

1. JDK バージョン 9 以上を使用している場合は、次の依存関係を追加します。

   ```xml
   <dependency>
       <groupId>javax.xml.bind</groupId>
       <artifactId>jaxb-api</artifactId>
       <version>2.3.1</version>
   </dependency>
   <dependency>
       <groupId>org.glassfish.jaxb</groupId>
       <artifactId>jaxb-runtime</artifactId>
       <version>2.3.1</version>
       <scope>runtime</scope>
   </dependency>
   ```

1. *pom.xml* ファイルを保存して閉じます。

## <a name="configure-your-spring-boot-app-to-use-your-azure-event-hub"></a>Azure イベント ハブを使用するように Spring Boot アプリを構成する

1. アプリの *resources* ディレクトリで *application.yaml* を探します。次に例を示します。

   *C:\SpringBoot\eventhubs-sample\src\main\resources\application.yaml*

   - または -

   */users/example/home/eventhubs-sample/src/main/resources/application.yaml*

2. テキスト エディターで *application.yaml* ファイルを開きます。次の行を追加し、サンプルの値をイベント ハブの適切なプロパティに置き換えます。

   ```yaml
    spring:
      cloud:
        azure:
          eventhub:
            connection-string: [eventhub-namespace-connection-string]
            checkpoint-storage-account: wingtiptoysstorage
            checkpoint-access-key: [checkpoint-access-key]
            checkpoint-container: wingtiptoyscontainer
            
        stream:
          bindings:
            consume-in-0:
              destination: wingtiptoyshub
              group: $Default
            supply-out-0:
              destination: wingtiptoyshub
   
          eventhub:
            bindings:
              consume-in-0:
                consumer:
                  checkpoint-mode: MANUAL
          function:
            definition: consume;supply;
          poller:
            initial-delay: 0
            fixed-delay: 1000
   ```

   各値の説明:

   |                          フィールド                           |                                                                                   説明                                                                                    |
   |----------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
   |               `spring.cloud.azure.eventhub.connection-string`                |                                        Azure portal の自分のイベント ハブ名前空間で取得した接続文字列を指定します。                                   |
   |               `spring.cloud.azure.function.definition`                |                                        バインドによって公開されている外部送信先にバインドする、機能 Bean を指定します。                                   |
   |               `spring.cloud.azure.poller.fixed-delay`                |                                        既定のポーラーの固定の遅延 (ミリ秒単位) を指定します (既定値は 1000L)。                                   |
   |               `spring.cloud.azure.poller.initial-delay`                |                                       定期的なトリガーの初期遅延を指定します (既定値は 0)。                                   |
   |               `spring.cloud.stream.bindings.consume-in-0.destination`                 |                            このチュートリアルで使用したイベント ハブを指定します。                         |
   |               `spring.cloud.stream.bindings.consume-in-0.group`                    |                               Event Hubs インスタンスのコンシューマー グループを指定します。                                |
   |               `spring.cloud.stream.bindings.supply-out-0.destination`                |                             このチュートリアルで使用したのと同じイベント ハブを指定します。                        |
   | `spring.cloud.stream.eventhub.bindings.consume-in-0.consumer.checkpoint-mode` |                                                       `MANUAL`を指定します。                                                   |
   |               `spring.cloud.stream.eventhub.checkpoint-access-key` |                                                      ストレージ アカウントのアクセス キーを指定します。                                                   |
   |               `spring.cloud.stream.eventhub.checkpoint-container` |                                                       ストレージ アカウントのコンテナーを指定します。                                                   |
   |               `spring.cloud.stream.eventhub.checkpoint-storage-account` |                                                 このチュートリアルで作成したストレージ アカウントを指定します。                                               |

3. *application.yaml* ファイルを保存して閉じます。

## <a name="add-sample-code-to-implement-basic-event-hub-functionality"></a>イベント ハブの基本的な機能を実装するサンプル コードを追加する

このセクションでは、イベント ハブにイベントを送信するために必要な Java クラスを作成します。

### <a name="modify-the-main-application-class"></a>アプリケーションのメイン クラスを変更する

1. アプリのパッケージ ディレクトリでメイン アプリケーションの Java ファイルを探します。次に例を示します。

   *C:\SpringBoot\eventhubs-sample\src\main\java\com\contoso\eventhubs\sample\EventhubSampleApplication.java*

   - または -

   */users/example/home/eventhubs-sample/src/main/java/com/contoso/eventhubs/sample/EventhubSampleApplication.java*

1. テキスト エディターでメイン アプリケーションの Java ファイルを開き、ファイルに次の行を追加します。

   ```java
    package com.contoso.eventhubs.sample;
    
    import com.azure.spring.integration.core.api.reactor.Checkpointer;
    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.context.annotation.Bean;
    import org.springframework.messaging.Message;
    
    import java.util.function.Consumer;
    
    import static com.azure.spring.integration.core.AzureHeaders.CHECKPOINTER;
    
    @SpringBootApplication
    public class EventhubSampleApplication {
    
        public static final Logger LOGGER = LoggerFactory.getLogger(EventhubSampleApplication.class);
    
        public static void main(String[] args) {
            SpringApplication.run(EventhubSampleApplication.class, args);
        }
    
        @Bean
        public Consumer<Message<String>> consume() {
            return message -> {
                Checkpointer checkpointer = (Checkpointer) message.getHeaders().get(CHECKPOINTER);
                LOGGER.info("New message received: '{}'", message);
                checkpointer.success()
                            .doOnSuccess(success -> LOGGER.info("Message '{}' successfully checkpointed", message))
                            .doOnError(error -> LOGGER.error("Exception: {}", error.getMessage()))
                            .subscribe();
            };
        }
    
    }
   ```

1. メイン アプリケーションの Java ファイルを保存して閉じます。

### <a name="create-a-new-configuration-class"></a>新しい構成クラスを作成する

1. アプリのパッケージ ディレクトリに *EventProducerConfiguration.java* という名前の新しい Java ファイルを作成し、テキスト エディターでファイルを開いて、次の行を追加します。

    ```java
    package com.contoso.eventhubs.sample;
    
    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.messaging.Message;
    import reactor.core.publisher.EmitterProcessor;
    import reactor.core.publisher.Flux;
    
    import java.util.function.Supplier;
    
    @Configuration
    public class EventProducerConfiguration {
    
        private static final Logger LOGGER = LoggerFactory.getLogger(EventProducerConfiguration.class);
    
        @Bean
        public EmitterProcessor<Message<String>> emitter() {
            return EmitterProcessor.create();
        }
    
        @Bean
        public Supplier<Flux<Message<String>>> supply(EmitterProcessor<Message<String>> emitter) {
            return () -> Flux.from(emitter)
                             .doOnNext(m -> LOGGER.info("Manually sending message {}", m))
                             .doOnError(t -> LOGGER.error("Error encountered", t));
        }
    }
    ```
1. *EventProducerConfiguration.java* ファイルを保存して閉じます。

### <a name="create-a-new-controller-class"></a>新しいコントローラー クラスを作成する

1. アプリのパッケージ ディレクトリに *EventProducerController.java* という名前の新しい Java ファイルを作成し、テキスト エディターでファイルを開いて、次の行を追加します。

   ```java
   package com.contoso.eventhubs.sample;
   
   import org.slf4j.Logger;
   import org.slf4j.LoggerFactory;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.http.ResponseEntity;
   import org.springframework.messaging.Message;
   import org.springframework.messaging.support.MessageBuilder;
   import org.springframework.web.bind.annotation.PostMapping;
   import org.springframework.web.bind.annotation.RequestBody;
   import org.springframework.web.bind.annotation.RestController;
   import reactor.core.publisher.EmitterProcessor;
   
   @RestController
   public class EventProducerController {
   
       public static final Logger LOGGER = LoggerFactory.getLogger(EventProducerController.class);
   
       @Autowired
       private EmitterProcessor<Message<String>> emitterProcessor;
   
       @PostMapping("/messages")
       public ResponseEntity<String> sendMessage(@RequestBody String message) {
           LOGGER.info("Going to add message {} to emitter", message);
           emitterProcessor.onNext(MessageBuilder.withPayload(message).build());
           return ResponseEntity.ok("Sent!");
       }
   }
   ```

1. *EventProducerController.java* ファイルを保存して閉じます。

## <a name="build-and-test-your-application"></a>アプリケーションをビルドしてテストする

次の手順に従って、アプリケーションをビルドしてテストします。

1. コマンド プロンプトを開き、ディレクトリを *pom.xml* ファイルが置かれているフォルダーに変更します。次に例を示します。

   ```cmd
    cd C:\SpringBoot\eventhubs-sample
   ```
   または

   ```bash
   cd /users/example/home/eventhubs-sample
   ```

1. Spring Boot アプリケーションを Maven でビルドし、実行します。次に例を示します。

   ```bash
   mvn clean package -Dmaven.test.skip=true
   mvn spring-boot:run
   ```

1. アプリケーションが実行されたら、`curl` を使用してアプリケーションをテストできます。次に例を示します。

   ```bash
   curl -X POST -H "Content-Type: text/plain" -d "hello" http://localhost:8080/messages
   ```
   アプリケーションのログに送信された "hello" が表示されます。 次に例を示します。

   ```output
   2020-09-11 15:11:12.138  INFO 7616 --- [      elastic-4] c.contoso.eventhubs.sample.EventhubSampleApplication  : New message received: 'hello'
   2020-09-11 15:11:12.406  INFO 7616 --- [ctor-http-nio-1] c.contoso.eventhubs.sample.EventhubSampleApplication  : Message 'hello' successfully checkpointed
   ```

## <a name="next-steps"></a>次のステップ

Spring および Azure の詳細については、Azure ドキュメント センターで引き続き Spring に関するドキュメントをご確認ください。

> [!div class="nextstepaction"]
> [Azure の Spring](./index.yml)

### <a name="additional-resources"></a>その他のリソース

Azure による Event Hub Stream Binder のサポートの詳細については、次の記事をご覧ください。

* [Azure Event Hubs とは](/azure/event-hubs/event-hubs-about)

* [Azure portal を使用して Event Hubs 名前空間とイベント ハブを作成する](/azure/event-hubs/event-hubs-create)

* [Azure Event Hubs で Apache Kafka 用 Spring Boot Starter を使用する方法](configure-spring-cloud-stream-binder-java-app-kafka-azure-event-hub.md)

Java での Azure の使用の詳細については、「[Java 開発者向けの Azure]」および「[Azure DevOps と Java の操作]」を参照してください。

**[Spring Framework]** は Java 開発者のエンタープライズ レベルのアプリケーション作成を支援するオープンソース ソリューションです。 このプラットフォームで構築される特に知られたプロジェクトの 1 つが [Spring Boot] です。これによって、スタンドアロンの Java アプリケーションの作成方法が簡略化されます。 Spring Boot を使い始めた開発者を支援するために、<https://github.com/spring-guides/> では、サンプルの Spring Boot パッケージがいくつか用意されています。 基本的な Spring Boot プロジェクトの一覧から選択するだけでなく、 **[Spring Initializr]** は、開発者がカスタム Spring Boot アプリケーションの作成を開始できるように支援します。

<!-- URL List -->

[無料の Azure アカウント]: https://azure.microsoft.com/pricing/free-trial/
[Java 開発者向けの Azure]: ../index.yml
[Azure DevOps と Java の操作]: /azure/devops/
[MSDN サブスクライバーの特典]: https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/
[Spring Boot]: http://projects.spring.io/spring-boot/
[Spring Initializr]: https://start.spring.io/
[Spring Framework]: https://spring.io/

<!-- IMG List -->

[IMG01]: media/configure-spring-cloud-stream-binder-java-app-azure-event-hub/create-event-hub-01.png
[IMG02]: media/configure-spring-cloud-stream-binder-java-app-azure-event-hub/create-event-hub-02.png
[IMG05]: media/configure-spring-cloud-stream-binder-java-app-azure-event-hub/create-event-hub-05.png
[IMG08]: media/configure-spring-cloud-stream-binder-java-app-azure-event-hub/create-event-hub-08.png
[SI01]: media/configure-spring-cloud-stream-binder-java-app-azure-event-hub/create-project-01.png
