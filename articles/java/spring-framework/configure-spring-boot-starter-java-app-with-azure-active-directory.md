---
title: Azure Active Directory 用の Spring Boot Starter の使用方法
description: Azure Active Directory スターターを使用して、Spring Boot Initializer アプリを構成する方法について説明します。
services: active-directory
documentationcenter: java
ms.date: 10/14/2020
ms.service: active-directory
ms.tgt_pltfrm: multiple
ms.topic: article
ms.workload: identity
ms.custom: devx-track-java
adobe-target: true
ms.openlocfilehash: d98355c37767c3d39de5cfc1f8b47030ad4f53be
ms.sourcegitcommit: b380f6e637b47e6e3822b364136853e1d342d5cd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/14/2021
ms.locfileid: "100395307"
---
# <a name="tutorial-secure-a-java-web-app-using-the-spring-boot-starter-for-azure-active-directory"></a>チュートリアル:Azure Active Directory 用の Spring Boot Starter を使用して Java Web アプリをセキュリティで保護する

この記事では、Azure Active Directory (Azure AD) 用 Spring Boot Starter を使用した **[Spring Initializr]** で Java アプリを作成する方法について説明します。

このチュートリアルでは、以下の内容を学習します。

 * Spring Initializr を使用して Java アプリケーションを作成する
 * Azure Active Directory を構成する
 * Spring Boot クラスと注釈を使用してアプリケーションをセキュリティで保護する
 * Java アプリケーションをビルドしてテストする

Azure サブスクリプションをお持ちでない場合は、開始する前に [無料アカウント](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) を作成してください。

## <a name="prerequisites"></a>前提条件

この記事の手順を実行するには、次の前提条件を満たす必要があります。

* サポートされている Java Development Kit (JDK)。 Azure での開発時に使用可能な JDK の詳細については、<https://aka.ms/azure-jdks> を参照してください。
* [Apache Maven](http://maven.apache.org/) バージョン 3.0 以降。

## <a name="create-an-app-using-spring-initializr"></a>Spring Initializr を使用したアプリの作成

1. <https://start.spring.io/> を参照します。

1. **Java** で **Maven** プロジェクトを生成することを指定し、アプリケーションの **[グループ]** と **[アーティファクト]** に名前を入力します。
1. **Spring Web**、**Azure Active Directory**、**OAuth2 Client** の **依存関係** を追加します。
1. ページの下部にある **[GENERATE]\(生成\)** ボタンを選択します。
   
   >[!div class="mx-imgBorder"]
   >![グループとアーティファクトの名前を指定して依存関係を選択する][create-spring-app-01]


1. メッセージが表示されたら、ローカル コンピューター上のパスにプロジェクトをダウンロードします。

## <a name="create-azure-active-directory-instance"></a>Azure Active Directory インスタンスの作成

### <a name="create-the-active-directory-instance"></a>Active Directory インスタンスを作成する

1. <https://portal.azure.com> にログインします。

1. **[リソースの作成]** 、 **[ID]** 、 **[Azure Active Directory]** の順に選択します。
   
   >[!div class="mx-imgBorder"]
   >![新しい Azure Active Directory インスタンスを作成する (手順 1)][create-directory-00]

   >[!div class="mx-imgBorder"]
   >![新しい Azure Active Directory インスタンスを作成する (手順 2)][create-directory-01]

1. **組織名** と **初期ドメイン名** を入力します。 ディレクトリの完全な URL をコピーします。このチュートリアルでは後ほど、これを使用してユーザー アカウントを追加します
 (例: `azuresampledirectory.onmicrosoft.com`)。 

    ディレクトリの完全な URL をコピーします。このチュートリアルでは後ほど、これを使用してユーザー アカウントを追加します (例: azuresampledirectory.onmicrosoft.com.)。

    完了したら、 **[作成]** を選択します。 新しいリソースの作成には数分かかります。
   
   >[!div class="mx-imgBorder"]
   >![Azure Active Directory の名前を指定する][create-directory-02]

1. 完了したら、選択して新しいディレクトリにアクセスします。
   
   >[!div class="mx-imgBorder"]
   >![Azure アカウント名を選択する][create-directory-03]

1. **テナント ID** をコピーします。この値を使用して、このチュートリアルの後の方で *application.properties* ファイルを構成します。
   
   >[!div class="mx-imgBorder"]
   >![テナント ID のコピー][create-directory-04]

### <a name="add-an-application-registration-for-your-spring-boot-app"></a>Spring Boot アプリのアプリケーション登録を追加する

1. ポータル メニューで **[アプリの登録]** を選択してから、 **[アプリケーションの登録]** を選択します。
   
   >[!div class="mx-imgBorder"]
   >![新しいアプリ登録を追加する][create-app-registration-01]

1. アプリケーションを指定し、 **[登録]** を選択します。
   
   >[!div class="mx-imgBorder"]
   >![新しいアプリ登録を作成する][create-app-registration-02]

1. アプリ登録のページが表示されたら、**アプリケーション ID** と **テナント ID** をコピーします。このチュートリアルでは後ほど、これらの値を使用して *application.properties* ファイルを構成します。
   
   >[!div class="mx-imgBorder"]
   >![アプリの登録キーをコピーする][create-app-registration-03]

1. 左側のナビゲーション ウィンドウで、 **[証明書とシークレット]** をクリックします。  次に、 **[新しいクライアント シークレット]** を選択します。
   
   >[!div class="mx-imgBorder"]
   >![アプリの登録キーを作成する][create-app-registration-03-5]

1. **説明** を追加し、 **[有効期限]** 一覧で期間を選択します。  **[追加]** をクリックします。 キーの値は自動的に入力されます。
   
   >[!div class="mx-imgBorder"]
   >![アプリの登録キーのパラメーターを指定する][create-app-registration-04]

1. このチュートリアルの後の方で *application.properties* ファイルを構成するために、クライアント シークレットの値をコピーして保存します。 (この値は後で取得することはできません)。
   
   >[!div class="mx-imgBorder"]
   >![アプリの登録キー値をコピーする][create-app-registration-04-5]

1. 左側のナビゲーション ウィンドウで **[API のアクセス許可]** をクリックします。 

1. **[Microsoft Graph]** をクリックし、 **[サインインしたユーザーとしてディレクトリにアクセスします]** と **[サインインとユーザー プロファイルの読み取り]** をオンにします。 **[アクセス許可の付与]** をクリックし、プロンプトが表示されたら **[はい]** をクリックします。
   
   >[!div class="mx-imgBorder"]
   >![アクセス許可を追加する][create-app-registration-08]
   
1. **[Grant admin consent for Azure Sample]\(Azure サンプルに管理者の同意を与えます\)** をクリックし、 **[はい]** を選択します。
   
   >[!div class="mx-imgBorder"]
   >![アクセス許可を付与する][create-app-registration-05]

1. アプリ登録のメイン ページで、 **[認証]** を選択し、 **[プラットフォームを追加]** を選択します。  次に、 **[Web アプリケーション]** を選択します。
   
   >[!div class="mx-imgBorder"]
   >![応答 URL を編集する][create-app-registration-09]

1. 新しい **リダイレクト URI** として「 *http://localhost:8080/login/oauth2/code/azure* 」と入力し、 **[構成]** を選択します。
   
   >[!div class="mx-imgBorder"]
   >![新しい応答 URL を追加する][create-app-registration-10]

### <a name="add-a-user-account-to-your-directory-and-add-that-account-to-a-group"></a>ディレクトリにユーザー アカウントを追加し、そのアカウントをグループに追加する

1. Active Directory の **[概要]** ページで、 **[ユーザー]** を選択し、次に **[新しいユーザー]** を選択します。
   
   >[!div class="mx-imgBorder"]
   >![新しいユーザー アカウントを追加する][create-user-01]

1. **[ユーザー]** パネルが表示されたら、 **[ユーザー名]** と **[名前]** を入力します。  **[作成]** を選択します。
   
   >[!div class="mx-imgBorder"]
   >![ユーザー アカウント情報を入力する][create-user-02]

   > [!NOTE]
   > ユーザー名を入力するときに、以下のように、このチュートリアルで先に出てきたディレクトリの URL を指定する必要があります。
   >
   > `test-user@azuresampledirectory.onmicrosoft.com`

1. Active Directory の **[概要]** ページで、 **[グループ]** を選択してから、アプリケーションで認可に使用する **[新しいグループ]** を選択します。

1. **[メンバーが選択されていません]** を選択します。 (このチュートリアルでは、*group1* という名前のグループを作成します)。前の手順で作成されたユーザーを検索します。  **[選択]** を選択して、そのユーザーをグループに追加します。  次に、 **[作成]** を選択して、新しいグループを作成します。

   >[!div class="mx-imgBorder"]
   >![グループのユーザーの選択][create-user-03]

1. **[ユーザー]** パネルに戻り、テスト ユーザーを選択し、 **[パスワードのリセット]** を選択してパスワードをコピーします。これは、このチュートリアルで後ほどアプリケーションにログインするときに使用します。

   >[!div class="mx-imgBorder"]
   >![パスワードを表示する][create-user-04]

## <a name="configure-and-compile-your-app"></a>アプリの構成およびコンパイル

1. このチュートリアルで先ほど作成しダウンロードしたプロジェクト アーカイブからディレクトリにファイルを抽出します。

1. プロジェクトの *src/main/resources* フォルダーに移動し、テキスト エディターで *application.properties* ファイルを開きます。

1. 前に作成した値を使用して、アプリ登録の設定を指定します。たとえば、以下のとおりです。

   ```properties
   # Specifies your Active Directory ID:
   azure.activedirectory.tenant-id=22222222-2222-2222-2222-222222222222
   # Specifies your App Registration's Application ID:
   azure.activedirectory.client-id=11111111-1111-1111-1111-1111111111111111
   # Specifies your App Registration's secret key:
   azure.activedirectory.client-secret=AbCdEfGhIjKlMnOpQrStUvWxYz==
   # Specifies the list of Active Directory groups to use for authorization:
   azure.activedirectory.user-group.allowed-groups=group1
   ```

   各値の説明:

   | パラメーター | 説明 |
   |---|---|
   | `azure.activedirectory.tenant-id` | 前の Active Directory の **ディレクトリ ID** を指定します。 |
   | `azure.activedirectory.client-id` | 以前に完了したアプリ登録の **アプリケーション ID** を指定します。 |
   | `azure.activedirectory.client-secret` | 以前に完了したアプリ登録キーの **値** を指定します。 |
   | `azure.activedirectory.user-group.allowed-groups` | 承認に使用する Active Directory グループの一覧を指定します。 |

   > [!NOTE]
   > *application.properties* ファイルで使用できる値の完全な一覧については、GitHub にある [Azure Active Directory Spring Boot のサンプル][AAD Spring Boot Sample]を参照してください。

1. *application.properties* ファイルを保存して閉じます。

1. アプリケーションの Java ソース フォルダーに、"*controller*" という名前のフォルダーを作成します (例: *src/main/java/com/wingtiptoys/security/controller*)。

1. *controller* フォルダーに "*HelloController.java*" という名前の新しい Java ファイルを作成し、テキスト エディターで開きます。

1. 次のコードを入力し、ファイルを保存して閉じます。

   ```java
   package com.wingtiptoys.security;

   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.ResponseBody;
   import org.springframework.web.bind.annotation.RestController;
   import org.springframework.security.access.prepost.PreAuthorize;
   
   @RestController
   public class HelloController {
   
      @GetMapping("group1")
      @ResponseBody
      @PreAuthorize("hasRole('ROLE_group1')")
      public String group1() {
         return "Hello Group 1 Users!";
      }
    
      @GetMapping("group2")
      @ResponseBody
      @PreAuthorize("hasRole('ROLE_group2')")
      public String group2() {
         return "Hello Group 2 Users!";
      }
   }
   ```

   > [!NOTE]
   > `@PreAuthorize("hasRole('')")` メソッドに指定するグループ名には、*application.properties* ファイルの `azure.activedirectory.user-group.allowed-groups` フィールドに指定したグループのいずれかが含まれている必要があります。
   >
   > 異なる要求マッピングには異なる承認設定を指定することもできます。以下に例を示します。
   >
   > ``` java
   > public class HelloController {
   >
   >    @PreAuthorize("hasRole('ROLE_Users')")
   >    @RequestMapping("/")
   >    public String helloWorld() {
   >       return "Hello Users!";
   >    }
   >    @PreAuthorize("hasRole('ROLE_group1')")
   >    @RequestMapping("/Group1")
   >    public String groupOne() {
   >       return "Hello Group 1 Users!";
   >    }
   >    @PreAuthorize("hasRole('ROLE_group2')")
   >    @RequestMapping("/Group2")
   >    public String groupTwo() {
   >       return "Hello Group 2 Users!";
   >    }
   > }
   > ```

1. テキスト エディターでアプリケーション クラスを開きます。

1. 次の例に示すように、アプリケーション クラスに `@EnableWebSecurity` と `@EnableGlobalMethodSecurity(prePostEnabled = true)` を追加し、ファイルを保存して閉じます。

    ```java
    package com.wingtiptoys;

    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
    import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
    
    @EnableWebSecurity
    @EnableGlobalMethodSecurity(prePostEnabled = true)
    @SpringBootApplication
    public class SpringBootSampleActiveDirectoryApplication {   
        public static void main(String[] args) {
            SpringApplication.run(SpringBootSampleActiveDirectoryApplication.class, args);
        }   
    }
    ```

## <a name="build-and-test-your-app"></a>アプリのビルドとテスト

1. コマンド プロンプトを開き、ディレクトリをアプリの *pom.xml* ファイルがあるフォルダーに変更します。

1. Spring Boot アプリケーションを Maven でビルドし、実行します。次に例を示します。

   ```shell
   mvn clean package
   mvn spring-boot:run
   ```
   
   >[!div class="mx-imgBorder"]
   >![アプリをビルドする][build-application]

1. Maven によってアプリケーションがビルドされ、起動したら、Web ブラウザーで `http://localhost:8080/group1` を開きます。ユーザー名とパスワードを入力するように求めるメッセージが表示されます。
   
   >[!div class="mx-imgBorder"]
   ![アプリケーションへのログイン][application-login]

   > [!NOTE]
   > 新しいユーザー アカウントに初めてログインする場合、パスワードの変更を求められることがあります。

   >[!div class="mx-imgBorder"]
   >![パスワードの変更][update-password]

1. 正常にログインすると、コントローラーにサンプルの "Hello Group 1 Users!" テキストが表示されます。

   >[!div class="mx-imgBorder"]
   >![Authorized_group1][hello-group1]

   > [!NOTE]
   > 承認されていないユーザー アカウントには、**HTTP 403 Unauthorized** メッセージが表示されます。

   >[!div class="mx-imgBorder"]
   >![UnAuthorized_group2][Unauthorized-group2]
## <a name="summary"></a>まとめ

このチュートリアルでは、Azure Active Directory スターターを使用した新しい Java Web アプリケーションの作成、新しい Azure AD テナントの構成とそのテナントへの新しいアプリケーションの登録を行いました。また、Spring の注釈とクラスを使用して Web を保護するようにアプリケーションを構成しました。

## <a name="see-also"></a>関連項目
* 新しい UI オプションについて詳しくは、「[新しい Azure portal アプリの登録トレーニング ガイド](/azure/active-directory/develop/app-registrations-training-guide-for-app-registrations-legacy-users)」をご覧ください。

## <a name="next-steps"></a>次のステップ

Spring および Azure の詳細については、Azure ドキュメント センターで引き続き Spring に関するドキュメントをご確認ください。

   >[!div class="nextstepaction"]
   >[Azure の Spring](./index.yml)

<!-- URL List -->

[Azure Active Directory Documentation]: /azure/active-directory/
[AAD app manifest]: /azure/active-directory/develop/active-directory-application-manifest
[Get started with Azure AD]: /azure/active-directory/get-started-azure-ad
[Azure for Java Developers]: ../index.yml
[free Azure account]: https://azure.microsoft.com/pricing/free-trial/
[Working with Azure DevOps and Java]: /azure/devops/
[MSDN subscriber benefits]: https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/
[Spring Boot]: http://projects.spring.io/spring-boot/
[Spring Initializr]: https://start.spring.io/
[Spring Framework]: https://spring.io/
[AAD Spring Boot Sample]: https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/spring/azure-spring-boot-samples/

<!-- IMG List -->

[create-spring-app-01]: media/configure-spring-boot-starter-java-app-with-azure-active-directory/create-spring-app-01.png

[create-directory-00]: media/configure-spring-boot-starter-java-app-with-azure-active-directory/create-directory-00.png
[create-directory-01]: media/configure-spring-boot-starter-java-app-with-azure-active-directory/create-directory-01.png
[create-directory-02]: media/configure-spring-boot-starter-java-app-with-azure-active-directory/create-directory-02.png
[create-directory-03]: media/configure-spring-boot-starter-java-app-with-azure-active-directory/create-directory-03.png
[create-directory-04]: media/configure-spring-boot-starter-java-app-with-azure-active-directory/create-directory-04.png

[create-app-registration-01]: media/configure-spring-boot-starter-java-app-with-azure-active-directory/create-app-registration-01.png
[create-app-registration-02]: media/configure-spring-boot-starter-java-app-with-azure-active-directory/create-app-registration-02.png
[create-app-registration-03]: media/configure-spring-boot-starter-java-app-with-azure-active-directory/create-app-registration-03.png
[create-app-registration-03-5]: media/configure-spring-boot-starter-java-app-with-azure-active-directory/create-app-registration-03-5.png
[create-app-registration-04]: media/configure-spring-boot-starter-java-app-with-azure-active-directory/create-app-registration-04.png
[create-app-registration-04-5]: media/configure-spring-boot-starter-java-app-with-azure-active-directory/create-app-registration-04-5.png
[create-app-registration-05]: media/configure-spring-boot-starter-java-app-with-azure-active-directory/create-app-registration-05.png
[create-app-registration-08]: media/configure-spring-boot-starter-java-app-with-azure-active-directory/create-app-registration-08.png
[create-app-registration-09]: media/configure-spring-boot-starter-java-app-with-azure-active-directory/create-app-registration-09.png
[create-app-registration-10]: media/configure-spring-boot-starter-java-app-with-azure-active-directory/create-app-registration-10.png

[create-user-01]: media/configure-spring-boot-starter-java-app-with-azure-active-directory/create-user-01.png
[create-user-02]: media/configure-spring-boot-starter-java-app-with-azure-active-directory/create-user-02.png
[create-user-03]: media/configure-spring-boot-starter-java-app-with-azure-active-directory/create-user-03.png
[create-user-04]: media/configure-spring-boot-starter-java-app-with-azure-active-directory/create-user-04.png

[application-login]: media/configure-spring-boot-starter-java-app-with-azure-active-directory/application-login.png
[build-application]: media/configure-spring-boot-starter-java-app-with-azure-active-directory/build-application.png
[hello-group1]: media/configure-spring-boot-starter-java-app-with-azure-active-directory/hello-group1.png
[update-password]: media/configure-spring-boot-starter-java-app-with-azure-active-directory/update-password.png
[Unauthorized-group2]: media/configure-spring-boot-starter-java-app-with-azure-active-directory/unauthorized-group2.png
