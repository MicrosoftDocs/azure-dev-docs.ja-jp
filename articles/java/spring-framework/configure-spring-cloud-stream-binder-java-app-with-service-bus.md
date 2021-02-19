---
title: Azure Service Bus 用の Spring Cloud Azure Stream Binder を使用する方法
description: この記事では、Spring Cloud Stream Binder を使用して Azure Service Bus との間でメッセージを送受信する方法について説明します。
author: seanli1988
manager: kyliel
ms.author: seal
ms.date: 02/04/2021
ms.topic: article
ms.custom: devx-track-java
ms.openlocfilehash: 4223927cd99e652e8fb06dd8250c39b191f36a94
ms.sourcegitcommit: bccbab4883e6b6b4926fc194c35ad948b11ccc3f
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2021
ms.locfileid: "99822705"
---
# <a name="how-to-use-spring-cloud-azure-stream-binder-for-azure-service-bus"></a>Azure Service Bus 用の Spring Cloud Azure Stream Binder を使用する方法

[!INCLUDE [spring-boot-20-note.md](includes/spring-boot-20-note.md)]

この記事では、Spring Cloud Stream Binder を使用して Service Bus `queues` および `topics` との間でメッセージを送受信する方法について説明します。

Azure には、[Azure Service Bus](/azure/service-bus-messaging/service-bus-messaging-overview) ("Service Bus") という、[Advanced Message Queueing Protocol 1.0](http://www.amqp.org/) ("AMQP 1.0") 標準に基づいた非同期のメッセージング プラットフォームが用意されています。 Service Bus は、サポートされている Azure プラットフォームの範囲全体で使用することができます。

## <a name="prerequisites"></a>前提条件

この記事の前提条件は次のとおりです。

1. Azure サブスクリプション。Azure サブスクリプションをまだお持ちでない場合は、[MSDN サブスクライバーの特典](https://azure.microsoft.com/pricing/member-offers/credit-for-visual-studio-subscribers/)を有効にするか、または[無料のアカウント](https://azure.microsoft.com/free/)にサインアップできます。

1. サポートされている Java Development Kit (JDK) (バージョン 8 以降)。 Azure での開発時に使用可能な JDK の詳細については、<https://aka.ms/azure-jdks> を参照してください。

1. Apache [Maven](http://maven.apache.org/) バージョン 3.2 以降。

1. 構成済みの Service Bus キューまたはトピックが既にある場合は、Service Bus 名前空間が次の要件を満たすようにしてください。

    1. すべてのネットワークからのアクセスを許可する
    1. Standard (またはそれ以上)
    1. 自分のキューおよびトピックに対する読み取り/書き込みアクセス権限を含んだアクセス ポリシーがある

1. 構成済みの Service Bus キューまたはトピックがない場合は、Azure portal を使用して [Service Bus キュー](/azure/service-bus-messaging/service-bus-quickstart-portal)または [Service Bus トピック](/azure/service-bus-messaging/service-bus-quickstart-topics-subscriptions-portal)を作成します。 前の手順で指定された要件を名前空間が確実に満たすようにしてください。 また、名前空間に含まれる接続文字列をメモしてください。これは、このチュートリアルのテスト アプリに必要です。

1. Spring Boot アプリケーションがない場合は、[Spring Initializr](https://start.spring.io/) で **Maven** プロジェクトを作成します。 必ず、 **[Maven プロジェクト]** を選択し、 **[依存関係]** に **[Web]** 依存関係を追加し、 **[Spring Boot]** で 2.3.8 を選択し、Java のバージョン **8** を選択してください。


## <a name="use-the-spring-cloud-stream-binder-starter"></a>Spring Cloud Stream Binder スターターを使用する

1. 自分のアプリの親ディレクトリで *pom.xml* ファイルを探します。例:

    `C:\SpringBoot\servicebus\pom.xml`

    または

    `/users/example/home/servicebus/pom.xml`

1. テキスト エディターで *pom.xml* ファイルを開きます。

1. Service Bus キューとトピックのどちらを使用しているかに応じて、 **&lt;dependencies>** 要素に次のコード ブロックを追加します。

    **Service Bus キュー**

    ```xml
    <dependency>
        <groupId>com.azure.spring</groupId>
        <artifactId>azure-spring-cloud-stream-binder-servicebus-queue</artifactId>
        <version>2.1.0</version>
    </dependency>
    ```

    **Service Bus トピック**

    ```xml
    <dependency>
        <groupId>com.azure.spring</groupId>
        <artifactId>azure-spring-cloud-stream-binder-servicebus-topic</artifactId>
        <version>2.1.0</version>
    </dependency>
    ```


1. *pom.xml* ファイルを保存して閉じます。

## <a name="configure-the-app-for-your-service-bus"></a>アプリでサービス バス用の構成を行う

接続文字列または資格情報ファイルに基づいて、自分のアプリを構成できます。 このチュートリアルでは、接続文字列を使用します。 資格情報ファイルの使用の詳細については、「[Service Bus キュー用 Spring Cloud Azure Stream Binder のコード サンプル](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/spring/azure-spring-boot-samples/azure-spring-cloud-sample-servicebus-queue-binder
)」と「[Service Bus トピック用 Spring Cloud Azure Stream Binder のコード サンプル](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/spring/azure-spring-cloud-stream-binder-servicebus-topic)」を参照してください。

1. アプリの *resources* ディレクトリ内で *application.properties* ファイルを探します。次に例を示します。

   `C:\SpringBoot\servicebus\src\main\resources\application.properties`

   または

   `/users/example/home/servicebus/src/main/resources/application.properties`

1. テキスト エディターで *application.properties* ファイルを開きます。

1. Service Bus キューとトピックのどちらを使用しているかに応じて、*application.properties* ファイルの最後に適切なコードを追加します。 [「フィールドの説明」の表](#fd)を使用して、サンプルの値を、実際のサービス バスの適切なプロパティに置き換えてください。

    **Service Bus キュー**

    ```yaml
    spring:
      cloud:
        azure:
          servicebus:
            connection-string: <ServiceBusNamespaceConnectionString>
        stream:
          bindings:
            consume-in-0:
              destination: examplequeue
            supply-out-0:
              destination: examplequeue
          servicebus:
            queue:
              bindings:
                consume-in-0:
                  consumer:
                    checkpoint-mode: MANUAL
          function:
            definition: consume;supply;
          poller:
            fixed-delay: 1000
            initial-delay: 0
    ```

    **Service Bus トピック**

    ```yaml
    spring:
      cloud:
        azure:
          servicebus:
            connection-string: <ServiceBusNamespaceConnectionString>
        stream:
          bindings:
            consume-in-0:
              destination: exampletopic
              group: examplesubscription
            supply-out-0:
              destination: exampletopic
          servicebus:
            topic:
              bindings:
                consume-in-0:
                  consumer:
                    checkpoint-mode: MANUAL
          function:
            definition: consume;supply;
          poller:
            fixed-delay: 1000
            initial-delay: 0
    ```

    **<a name="fd">フィールドの説明</a>**

    |                                        フィールド                                   |                                                                                   説明                                                                                    |
    |--------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
    |               `spring.cloud.azure.function.definition`                |                                        バインドによって公開されている外部送信先にバインドする、機能 Bean を指定します。                                   |
    |               `spring.cloud.azure.poller.fixed-delay`                |                                        既定のポーラーの固定の遅延 (ミリ秒単位) を指定します (既定値は 1000L)。                                   |
    |               `spring.cloud.azure.poller.initial-delay`                |                                       定期的なトリガーの初期遅延を指定します (既定値は 0)。                                   |
    |               `spring.cloud.azure.servicebus.connection-string`                |                                        Azure portal の自分の Service Bus 名前空間で取得した接続文字列を指定します。                                   |
    |               `spring.cloud.stream.bindings.consume-in-0.destination`                 |                            このチュートリアルで自分が使用した Service Bus キューまたは Service Bus トピックを指定します。                         |
    |                  `spring.cloud.stream.bindings.consume-in-0.group`                    |                                            Service Bus トピックを使用した場合は、トピックのサブスクリプションを指定します。                                |
    |               `spring.cloud.stream.bindings.supply-out-0.destination`                |                               入力先に使用したものと同じ値を指定します。                        |
    | `spring.cloud.stream.servicebus.queue.bindings.consume-in-0.consumer.checkpoint-mode` |                                                       `MANUAL` を指定します。                                                   |
    | `spring.cloud.stream.servicebus.topic.bindings.consume-in-0.consumer.checkpoint-mode` |                                                       `MANUAL`を指定します。                                                   |

1. *application.properties* ファイルを保存して閉じます。

## <a name="implement-basic-service-bus-functionality"></a>基本的な Service Bus 機能を実装する

このセクションでは、自分のサービス バスにメッセージを送信するために必要な Java クラスを作成します。

### <a name="modify-the-main-application-class"></a>アプリケーションのメイン クラスを変更する

1. アプリのパッケージ ディレクトリでメイン アプリケーションの Java ファイルを探します。次に例を示します。

    `C:\SpringBoot\servicebus\src\main\java\com\example\ServiceBusBinderApplication.java`

   または

   `/users/example/home/servicebus/src/main/java/com/example/ServiceBusBinderApplication.java`

1. テキスト エディターでメイン アプリケーションの Java ファイルを開きます。

1. 次のコードをファイルに追加します。

    ```java
   package com.example;
   
   import com.azure.spring.integration.core.api.Checkpointer;
   import org.slf4j.Logger;
   import org.slf4j.LoggerFactory;
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.context.annotation.Bean;
   import org.springframework.messaging.Message;
   
   import java.util.function.Consumer;
   
   import static com.azure.spring.integration.core.AzureHeaders.CHECKPOINTER;
   
   @SpringBootApplication
   public class ServiceBusBinderApplication {
   
       private static final Logger LOGGER = LoggerFactory.getLogger(ServiceBusBinderApplication.class);
   
       public static void main(String[] args) {
           SpringApplication.run(ServiceBusBinderApplication.class, args);
       }
   
       @Bean
       public Consumer<Message<String>> consume() {
           return message -> {
               Checkpointer checkpointer = (Checkpointer) message.getHeaders().get(CHECKPOINTER);
               LOGGER.info("New message received: '{}'", message);
               checkpointer.success().handle((r, ex) -> {
                   if (ex == null) {
                       LOGGER.info("Message '{}' successfully checkpointed", message);
                   }
                   return null;
               });
           };
       }
   }
    ```

1. ファイルを保存して閉じます。

### <a name="create-a-new-producer-configuration-class"></a>新しいプロデューサー構成クラスを作成する

1. テキスト エディターを使用して、*ServiceProducerConfiguration.java* という名前の Java ファイルを自分のアプリのパッケージ ディレクトリに作成します。

1. この新しいファイルに次のコードを追加します。

    ```java
   package com.example;
   
   import org.slf4j.Logger;
   import org.slf4j.LoggerFactory;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   import org.springframework.messaging.Message;
   import reactor.core.publisher.EmitterProcessor;
   import reactor.core.publisher.Flux;
   
   import java.util.function.Supplier;
   
   @Configuration
   public class ServiceProducerConfiguration {
   
       private static final Logger LOGGER = LoggerFactory.getLogger(ServiceProducerConfiguration.class);
   
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

1. *ServiceProducerConfiguration.java* ファイルを保存して閉じます。

### <a name="create-a-new-controller-class"></a>新しいコントローラー クラスを作成する

1. テキスト エディターを使用して、*ServiceProducerController.java* という名前の Java ファイルを自分のアプリのパッケージ ディレクトリに作成します。

1. この新しいファイルに次のコード行を追加します。

    ```java
   package com.example;
   
   import org.slf4j.Logger;
   import org.slf4j.LoggerFactory;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.http.ResponseEntity;
   import org.springframework.messaging.Message;
   import org.springframework.messaging.support.MessageBuilder;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.PostMapping;
   import org.springframework.web.bind.annotation.RequestParam;
   import org.springframework.web.bind.annotation.RestController;
   import reactor.core.publisher.EmitterProcessor;
   
   @RestController
   public class ServiceProducerController {
   
       private static final Logger LOGGER = LoggerFactory.getLogger(ServiceProducerController.class);
   
       @Autowired
       private EmitterProcessor<Message<String>> emitterProcessor;
   
       @PostMapping("/messages")
       public ResponseEntity<String> sendMessage(@RequestParam String message) {
           LOGGER.info("Going to add message {} to emitter", message);
           emitterProcessor.onNext(MessageBuilder.withPayload(message).build());
           return ResponseEntity.ok("Sent!");
       }
   
       @GetMapping("/")
       public String welcome() {
           return "welcome";
       }
   }
    ```

1. *ServiceProducerController.java* ファイルを保存して閉じます。

## <a name="build-and-test-your-application"></a>アプリケーションをビルドしてテストする

1. コマンド プロンプトを開きます。

1. ディレクトリを自分の *pom.xml* ファイルの保存先に変更します。例:

    `cd C:\SpringBoot\servicebus`

    - または -

    `cd /users/example/home/servicebus`

2. 自分の Spring Boot アプリケーションを Maven でビルドし、実行します。

    ```shell
    mvn clean spring-boot:run
    ```

3. 自分のアプリケーションが実行されたら、*curl* を使用してアプリケーションをテストできます。

    ```shell
    curl -X POST localhost:8080/messages?message=hello
    ```

    自分のアプリケーションのログに送信された "hello" が表示されます。

    ```shell
    New message received: 'hello'
    Message 'hello' successfully checkpointed
    ```

## <a name="clean-up-resources"></a>リソースをクリーンアップする

予想外の課金を防ぐために、この記事で作成したリソースが不要になったら、[Azure portal](https://portal.azure.com/) を使用して削除してください。

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [Azure の Spring](/java/azure/spring-framework)