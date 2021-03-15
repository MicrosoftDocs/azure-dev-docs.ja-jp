---
title: Azure での Spring Cloud Function の概要
description: Azure で Spring Cloud Function を使用する方法について説明します。
documentationcenter: java
author: jdubois
manager: brborges
ms.author: judubois
ms.date: 07/17/2019
ms.service: azure-functions
ms.tgt_pltfrm: multiple
ms.topic: article
ms.custom: devx-track-java
ms.openlocfilehash: 5c13a69769bef56c2b67607118b0a20c4f59cd5c
ms.sourcegitcommit: 576c878c338d286060010646b96f3ad0fdbcb814
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102118435"
---
# <a name="getting-started-with-spring-cloud-function-in-azure"></a>Azure での Spring Cloud Function の概要

この記事では、[Spring Cloud Function](https://spring.io/projects/spring-cloud-function) を使用して Java 関数を開発し、それを Azure Functions に発行する手順について説明します。 完了すると、関数コードは Azure の[従量課金プラン](/azure/azure-functions/functions-scale#consumption-plan)で実行され、HTTP 要求を使用してトリガーできるようになります。

[!INCLUDE [quickstarts-free-trial-note](includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>前提条件

Java を使用して関数を開発するには、以下のものがインストールされている必要があります。

- [Java Developer Kit](../fundamentals/java-jdk-long-term-support.md)、バージョン 8
- [Apache Maven](https://maven.apache.org)、バージョン 3.0 以降
- [Azure CLI](/cli/azure)
- [Azure Functions Core Tools](/azure/azure-functions/functions-run-local#v2) バージョン 2.7.1158 以上

> [!IMPORTANT]
> このクイックスタートを行うには、JAVA_HOME 環境変数を JDK のインストール場所に設定する必要があります。

## <a name="what-we-are-going-to-build"></a>ビルドするもの

Azure Functions 上で実行される古典的な "Hello, World" 関数をビルドします。これは、Spring Cloud Function を使用して構成されます。

これは、ユーザー名が含まれているシンプルな `User` JSON オブジェクトを受け取り、そのユーザーへのウェルカム メッセージが含まれている `Greeting` オブジェクトを返します。

ここでビルドするプロジェクトは [https://github.com/Azure-Samples/hello-spring-function-azure](https://github.com/Azure-Samples/hello-spring-function-azure) で入手できるため、このクイックスタートで詳しく説明されている完成形を確認したい場合は、そのサンプル リポジトリを直接使用することができます。

## <a name="create-a-new-maven-project"></a>新しい Maven プロジェクトを作成する

空の Maven プロジェクトを作成し、Spring Cloud Function と Azure Functions を使用してそれを構成します。

空のフォルダーに新しい *pom.xml* を作成し、[https://github.com/Azure-Samples/hello-spring-function-azure/blob/master/pom.xml](https://github.com/Azure-Samples/hello-spring-function-azure/blob/master/pom.xml) にあるサンプル プロジェクトの内容をコピーして貼り付けます。

> [!NOTE]
> このファイルでは、Spring Boot と Spring Cloud Function 両方の Maven 依存関係を使用し、Spring Boot と Azure Functions の Maven プラグインを構成します。

いくつかのプロパティは、自分のアプリケーションに合わせてカスタマイズする必要があります。

- `<functionAppName>` は、自分の Azure 関数の名前です
- `<functionAppRegion>` は、自分の関数がデプロイされる Azure リージョンの名前です
- `<functionResourceGroup>` は、自分が使用している Azure リソース グループの名前です

これらのプロパティは、*pom.xml* ファイルの先頭付近で直接変更する必要があります。

```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>

    <azure.functions.java.library.version>1.4.0</azure.functions.java.library.version>
    <azure.functions.maven.plugin.version>1.9.1</azure.functions.maven.plugin.version>

    <!-- customize those two properties. The functionAppName should be unique across Azure -->
    <functionResourceGroup>my-spring-function-resource-group</functionResourceGroup>
    <functionAppName>my-spring-function</functionAppName>

    <functionAppRegion>eastus</functionAppRegion>
    <stagingDirectory>${project.build.directory}/azure-functions/${functionAppName}</stagingDirectory>
    <start-class>com.example.DemoApplication</start-class>
    <spring.boot.wrapper.version>1.0.26.RELEASE</spring.boot.wrapper.version>
</properties>
```

## <a name="create-azure-configuration-files"></a>Azure 構成ファイルを作成する

*src/main/azure* フォルダーを作成し、それに次の Azure Functions 構成ファイルを追加します。

*host.json*:

```json
{
  "version": "2.0",
  "functionTimeout": "00:10:00"
}
```

*local.settings.json*:

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "",
    "FUNCTIONS_WORKER_RUNTIME": "java",
    "MAIN_CLASS":"com.example.DemoApplication",
    "AzureWebJobsDashboard": ""
  }
}
```

## <a name="create-domain-objects"></a>ドメイン オブジェクトを作成する

Azure Functions では、JSON 形式のオブジェクトを送受信できます。
ここでは、ドメイン モデルを表す `User` オブジェクトと `Greeting` オブジェクトを作成します。
このクイックスタートをカスタマイズして、もっと自分の関心に沿ったものにしたい場合は、さらに多くのプロパティを備えた、より複雑なオブジェクトを作成できます。

*src/main/java/com/example/model* フォルダーを作成して、次の 2 つのファイルを追加します。

*User.java*:

```java
package com.example.model;

public class User {

    public User() {
    }

    public User(String name) {
        this.name = name;
    }

    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

*Greeting.java*:

```java
package com.example.model;

public class Greeting {

    public Greeting() {
    }

    public Greeting(String message) {
        this.message = message;
    }

    private String message;

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}
```

## <a name="create-the-spring-boot-application"></a>Spring Boot アプリケーションを作成する

このアプリケーションは、すべてのビジネス ロジックを管理し、Spring Boot エコシステム全体へのアクセス権を備えたものとなります。
このため、標準の Azure 関数に比べて、2 つの主なメリットがあります。

- これは Azure Functions API に依存していないため、他のシステムに簡単に移植できます。 たとえば、通常の Spring Boot アプリケーションで再利用することもできます。
- Spring Boot のすべての `@Enable` 注釈を使用して、新しい強力な機能を簡単に追加できます。

*src/main/java/com/example* フォルダーで次のファイルを作成します。これは通常の Spring Boot アプリケーションです。

*DemoApplication.java*:

```java
package com.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) throws Exception {
        SpringApplication.run(HelloFunction.class, args);
    }
}
```

次に、以下のファイルを作成します。これには、実行する関数を表す Spring Boot コンポーネントが含まれています。

*HelloFunction.java*:

```java
package com.example;

import com.example.model.Greeting;
import com.example.model.User;
import org.springframework.stereotype.Component;

import java.util.function.Function;

@Component
public class HelloFunction implements Function<User, Greeting> {

    @Override
    public Greeting apply(User user) {
        return new Greeting("Hello, " + user.getName() + "!\n");
    }
}
```

> [!NOTE]
> `HelloFunction` は、非常に具体的な関数です。
>
> - これは、このクイックスタートで使用される関数である `java.util.function.Function` です。 これにはビジネス ロジックが含まれており、標準の Java API を使用して、あるオブジェクトを別のオブジェクトに変換します。
> - これは `@Component` 注釈を備えているため、Spring Bean です。既定では、小文字で始まるクラスのいずれか (`helloFunction`) がその名前になっています。 これは自分のアプリケーションに他の関数を作成したい場合に重要です。この名前が、次のセクションで作成する Azure 関数の名前と一致する必要があるためです。

## <a name="create-the-azure-function"></a>Azure 関数を作成する

完全な Azure Functions API のメリットを得るために、次は特定のクラスのコードを書きます。これは、その実行を前の手順で作成した Spring Coud 関数に委任する Azure 関数です。

*src/main/java/com/example* フォルダーで、次の Azure 関数を作成します。

*HelloHandler.java*:

```java
package com.example;

import com.example.model.Greeting;
import com.example.model.User;
import com.microsoft.azure.functions.*;
import com.microsoft.azure.functions.annotation.AuthorizationLevel;
import com.microsoft.azure.functions.annotation.FunctionName;
import com.microsoft.azure.functions.annotation.HttpTrigger;
import org.springframework.cloud.function.adapter.azure.AzureSpringBootRequestHandler;

import java.util.Optional;

public class HelloHandler extends AzureSpringBootRequestHandler<User, Greeting> {

    @FunctionName("hello")
    public HttpResponseMessage execute(
            @HttpTrigger(name = "request", methods = {HttpMethod.GET, HttpMethod.POST}, authLevel = AuthorizationLevel.ANONYMOUS) HttpRequestMessage<Optional<User>> request,
            ExecutionContext context) {
        User user = request.getBody()
                .filter((u -> u.getName() != null))
                .orElseGet(() -> new User(
                        request.getQueryParameters()
                                .getOrDefault("name", "world")));
        context.getLogger().info("Greeting user name: " + user.getName());
        return request
                .createResponseBuilder(HttpStatus.OK)
                .body(handleRequest(user, context))
                .header("Content-Type", "application/json")
                .build();
    }
}
```

この Java クラスは Azure 関数であり、次のような興味深い機能を備えています。

- これによって、`AzureSpringBootRequestHandler` が拡張され、Azure Functions と Spring Cloud Function のリンクが行われます。 これは、`body()` メソッドで使用される `handleRequest()` メソッドを提供します。
- `@FunctionName("hello")` で定義されているように、関数の名前は `hello` です。
- これは本物の Azure 関数であるため、ここでは完全な Azure Functions API を使用できます。

## <a name="add-unit-tests"></a>単体テストを追加する

当然、この手順は省略可能です。しかし、優れた開発者として、アプリケーションが正常に機能することを検証するための単体テストを追加することが推奨されます。

*src/test/java/com/example* フォルダーを作成し、次の JUnit テストを追加します。

*HelloFunctionTest.java*:

```java
package com.example;

import com.example.model.Greeting;
import com.example.model.User;
import org.junit.jupiter.api.Test;
import org.springframework.cloud.function.adapter.azure.AzureSpringBootRequestHandler;

import static org.assertj.core.api.Assertions.assertThat;

public class HelloFunctionTest {

    @Test
    public void test() {
        Greeting result = new HelloFunction().apply(new User("foo"));
        assertThat(result.getMessage()).isEqualTo("Hello, foo!\n");
    }

    @Test
    public void start() throws Exception {
        AzureSpringBootRequestHandler<User, Greeting> handler = new AzureSpringBootRequestHandler<>(
                HelloFunction.class);
        Greeting result = handler.handleRequest(new User("foo"), null);
        handler.close();
        assertThat(result.getMessage()).isEqualTo("Hello, foo!\n");
    }
}
```

ここで、Maven を使用して自分の Azure 関数をテストできます。

```bash
mvn clean test
```

## <a name="run-the-function-locally"></a>関数をローカルで実行する

自分のアプリケーションを Azure 関数にデプロイする前に、まずはそれをローカルでテストしてみましょう。

最初に、自分のアプリケーションを Jar ファイルにパッケージ化する必要があります。

```bash
mvn package
```

アプリケーションがパッケージ化されたら、`azure-functions` Maven プラグインを使用してそれを実行できます。

```bash
mvn azure-functions:run
```

Azure 関数は、ポート 7071 を使って自分の localhost で使用できるようになっているはずです。 JSON 形式の `User` オブジェクトが含まれた POST 要求を送信して、関数をテストできます。 たとえば、cURL を使用した場合:

```bash
curl http://localhost:7071/api/hello -d "{\"name\":\"Azure\"}"
```

関数は JSON 形式のままで `Greeting` オブジェクトを返すはずです。

```Output
{
  "message": "Welcome, Azure"
}
```

次に示すのは、画面の上部に cURL 要求があり、下部にローカルの Azure 関数があるスクリーンショットです。

 ![ローカルで実行中の Azure 関数][RFL01]

## <a name="deploy-the-function-to-azure-functions"></a>関数を Azure Functions にデプロイする

次に、Azure 関数を運用環境に発行します。 *pom.xml* で自分が定義した `<functionAppName>`、`<functionAppRegion>`、`<functionResourceGroup>` の各プロパティは、自分の関数を構成するために使用されることを覚えておいてください。

> [!NOTE]
> Maven プラグインでは Azure を使って認証を行う必要があります。Azure CLI がインストールされている場合は、続行する前に `az login` を使用してください。
> 認証オプションの詳細については、[こちら](https://github.com/microsoft/azure-maven-plugins/wiki/Authentication)確認してください。

Maven を実行して自分の関数を自動でデプロイします。

```bash
mvn azure-functions:deploy
```

次に [Azure portal](https://portal.azure.com) に移動して、作成された `Function App` を見つけます。

関数をクリックします。

- 関数の概要で、関数の URL をメモします。
- **[プラットフォーム機能]** タブを選択して **ログ ストリーミング** サービスを見つけます。次に、そのサービスを選択して、自分が実行している関数を確認します。

ここで、前のセクションで行ったように cURL を使用して、実行中の関数にアクセスします。 `your-function-name` は実際の関数名に置き換えてください。

```bash
curl https://your-function-name.azurewebsites.net/api/hello -d "{\"name\":\"Azure\"}"
```

前のセクションと同様に、関数は JSON 形式のままで `Greeting` オブジェクトを返すはずです。

```Output
{
  "message": "Welcome, Azure"
}
```

おめでとうございます。Azure Functions 上で実行される Spring Cloud 関数ができました。

<!-- IMG List -->

[RFL01]: media/getting-started-with-spring-cloud-function-in-azure/RFL01.png
