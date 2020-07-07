---
title: Visual Studio App Center と Azure サービスを使用してモバイル アプリケーションのデプロイとリリースを自動化する
description: モバイル アプリケーションの継続的デリバリー パイプラインの設定に役立つ、App Center などのサービスについて学習します。
author: codemillmatt
ms.assetid: 34a8a070-9b3c-4faf-8588-ccff02097224
ms.service: mobile-services
ms.topic: article
ms.date: 06/08/2020
ms.author: masoucou
ms.openlocfilehash: 96ca72ffb239b7baa48c06f09afe08a2653e3e0b
ms.sourcegitcommit: f02ff55cef47581303a217cf1d310879cd722237
ms.contentlocale: ja-JP
ms.lasthandoff: 06/09/2020
ms.locfileid: "84632374"
---
# <a name="automate-the-deployment-and-release-of-your-mobile-applications-with-continuous-delivery-services"></a>継続的デリバリー サービスを使用してモバイル アプリケーションのデプロイとリリースを自動化する

開発者はコードを記述し、コード リポジトリにチェックインしますが、リポジトリにチェックインされたコミットに常に整合性があるとは限りません。 複数の開発者が同じプロジェクトで作業する場合は、統合に問題が発生する可能性があります。 チームは、物事がうまくいかず、バグが山積みになり、プロジェクトの開発が遅れるという状況に陥る可能性があります。 開発者は、ソフトウェア コード全体がビルドされ、エラーを確認するためにテストされるまで待機する必要があります。これにより、プロセスの速度が低下し、反復が少なくなります。

継続的デリバリーを使用すると、モバイル アプリケーションのデプロイとリリースを自動化できます。 アプリケーションのデリバリー先がテスト担当者または会社の従業員のグループであるか (ベータ テストの場合)、アプリ ストアであるか (運用環境の場合) は関係ありません。 継続的デリバリーにより、デプロイのリスクが減り、迅速な反復が促進されます。 また、顧客に新しい変更を継続的にリリースすることもできます。

## <a name="distribute-application-binaries-to-beta-testers"></a>ベータ テスト担当者にアプリケーション バイナリを配布する

モバイル アプリケーションのベータ テストは、アプリケーション開発プロセス時の重要な手順の 1 つです。 アプリケーションのバグや問題を早期に検出するのに役立ちます。 このフィードバックにより、運用環境で使用できるようになったときのアプリケーションの品質が向上します。

モバイル アプリで継続的デリバリー パイプラインを有効にするには、次のサービスを使用します。

### <a name="visual-studio-app-center-distribute"></a>Visual Studio の App Center Distribute

[App Center 配布](/appcenter/distribution/)は、開発者がビルドをデバイスに迅速にリリースするためのツールです。 完全なインストール ポータル エクスペリエンスにより、App Center 配布は、ベータ版アプリ テスト担当者への配布のための強力なソリューションになります。 また、パブリック アプリ ストアでの配布に代わる便利な方法でもあります。 開発者は、App Center ビルドとパブリック アプリケーション ストアの統合により、配布ワークフローをさらに自動化できます。

#### <a name="visual-studio-app-center-distribute-features"></a>Visual Studio の App Center Distribute の機能

- アプリをベータ テスト担当者とユーザーに配布し、すべてのテスト担当者が最新バージョンのアプリケーションを確実に使用するようにします。
- テスト担当者に新しいリリースを通知します。このとき、テスト担当者がダウンロード フローを再度実行することはありません。
- さまざまなバージョンのアプリケーションの配布グループを管理します。
- ストアに配布します。 
  - [Apple](/appcenter/distribution/stores/apple)
  - [Google Play](/appcenter/distribution/stores/googleplay)
  - [Intune](/appcenter/distribution/stores/intune)
- iOS、Android、macOS、tvOS、Xamarin、React Native、Unity、および Cordova のプラットフォーム サポートを利用できます。
- プロビジョニング プロファイルに iOS デバイスを自動的に登録します。

#### <a name="visual-studio-app-center-distribute-references"></a>Visual Studio の App Center Distribute の参考資料

- [Visual Studio App Center でのサインアップ](https://appcenter.ms/signup)
- [App Center Distribute を使ってみる](/appcenter/build/)

### <a name="azure-pipelines"></a>Azure Pipelines

[Azure Pipelines](https://azure.microsoft.com/services/devops/pipelines/) は、完全な機能を備えた継続的インテグレーション (CI) および継続的デリバリー (CD) サービスであり、任意の Git プロバイダーと連携します。 Azure Pipelines は、Azure サービスなど、ほとんどの主要なクラウド サービスにデプロイできます。 GitHub、GitHub Enterprise サーバー、GitLab、Bitbucket Cloud、または Azure Repos でコードを使用して作業を開始することができます。 その後、Microsoft Azure、Google Cloud Platform、アマゾン ウェブ サービス (AWS) のコードのビルド、テスト、およびデプロイを自動化できます。

#### <a name="azure-pipelines-features"></a>Azure Pipelines の機能

- **CI サーバーを設定するための簡素化されたタスクベースのエクスペリエンス:** ネイティブ (Android、iOS、Windows) およびクロスプラットフォーム (Xamarin、Cordova、React Native) の両方のモバイル アプリケーション用に CI サーバーを設定します。
- **任意の言語、プラットフォーム、クラウド:** Node.js、Python、Java、PHP、Ruby、Go、C/C++、C#、Android、iOS アプリをビルド、テスト、デプロイします。 Linux、macOS、および Windows で並列実行します。 Azure、AWS、Google Cloud Platform などのクラウド プロバイダーにデプロイします。 ベータ チャネルとアプリ ストアを使用してモバイル アプリケーションを配布します。
- **ネイティブ コンテナーのサポート:** 新しいコンテナーを簡単に作成し、任意のレジストリにプッシュします。 コンテナーを独立したホストまたは Kubernetes にデプロイします。
- **高度なワークフローと機能:** ビルド チェーンと複数のフェーズにわたるビルドを簡単に作成できます。 YAML、テスト統合、リリース ゲート、レポートなどがサポートされます。
- **拡張可能:** Slack から SonarCloud までの数百の拡張機能を含む、コミュニティによってビルドされたさまざまなビルド、テスト、デプロイ タスクを使用できます。 Jenkins などの、他の CI システムからデプロイすることもできます。 Web hook と REST API を使用すれば、統合が容易になります。
- **クラウドでホストされる無料ビルド:** これらのビルドは、パブリックおよびプライベート リポジトリで利用できます。
- **他のクラウド ベンダーへのデプロイのサポート:** ベンダーには AWS と Google Cloud Platform が含まれています。

#### <a name="azure-pipelines-references"></a>Azure Pipelines の参考資料

- [Azure Pipelines での作業の開始に関するガイド](/azure/devops/pipelines/get-started/pipelines-get-started?view=azure-devops)
- [Azure DevOps を使用して作業を開始する](https://app.vsaex.visualstudio.com/signup/)
  
## <a name="distribute-your-application-directly-to-app-stores"></a>アプリケーションをアプリ ストアに直接配布する

アプリケーションを運用環境で利用する準備ができた後、一般に使用できるようにするには、アプリ ストアに送信して、お客様がダウンロードできるようにする必要があります。 アプリケーションをアプリ ストアに直接配布するための方法は複数あります。 

### <a name="visual-studio-app-center-distribute-stores"></a>Visual Studio App Center 配布ストア

[App Center 配布](/appcenter/distribution/stores/)を使用すると、モバイル アプリケーションをアプリ ストアに直接公開できます。 アプリケーションをエンド ユーザーがダウンロードできるようになったら、Visual Studio App Center ポータルから直接アプリケーション バイナリを公開できます。 

以下に直接配布できます。

- [Apple App Store](/appcenter/distribution/stores/apple)
- [Google Play ストア](/appcenter/distribution/stores/googleplay)
- [Microsoft Intune](/appcenter/distribution/stores/intune)

### <a name="apple-app-store"></a>Apple App Store

Apple によって開発および管理されているアプリ ストアでは、ユーザーは iOS、MacOS、WatchOS、および tvOS デバイス用に開発されたアプリケーションを参照してダウンロードできます。 開発者は、iOS アプリを一般に使用できるようにするために、Apple App Store に送信する必要があります。

### <a name="google-play"></a>Google Play

Google Play は Android OS 向けの公式アプリ ストアであり、ユーザーは Android デバイス用に開発され、Google 経由で公開されたアプリケーションを参照してダウンロードすることができます。

### <a name="intune"></a>Intune

[Microsoft Intune](/intune/app-management) は、企業データを保護しながら従業員の生産性を向上させるのに役立つ、エンタープライズ モビリティ管理領域のクラウドベース サービスです。 Intune では、次のことが可能です。

- 会社のデータにアクセスするために従業員が使用するモバイル デバイスと PC を管理する。
- 従業員が使用するモバイル アプリケーションを管理する。
- 従業員が会社の情報にアクセスして共有する方法を制御することで、その情報を保護する。
- デバイスとアプリケーションが会社のセキュリティ要件に確実に準拠しているようにする。

## <a name="deploy-updates-directly-to-users-devices"></a>更新プログラムをユーザーのデバイスに直接デプロイする

### <a name="codepush"></a>CodePush

App Center の [CodePush](/appcenter/distribution/codepush/) を使用すると、Apache Cordova および React Native 開発者は、モバイル アプリケーションの更新プログラムをユーザーのデバイスに直接デプロイできます。 それは、開発者が特定の更新プログラム (JavaScript、HTML、CSS、イメージの変更など) を公開できる中央リポジトリとして機能します。 その後、アプリケーションでは、提供されたクライアント SDK を使用して、リポジトリから更新プログラムを照会できます。 そうすることで、バグに対処したり、小規模な機能を追加したりする際に、より確定的で直接的なエンゲージメント モデルをユーザーに対して使用できます。 バイナリをリビルドしたり、パブリック アプリ ストアを使用して再配布したりする必要はありません。

#### <a name="codepush-key-features"></a>CodePush の主な機能

- Cordova および React Native 開発者は、ストアでリリースすることなく、モバイル アプリケーションの更新プログラムをユーザーのデバイスに直接デプロイできます。
- バグの修正、またはバイナリをリビルドし、それぞれのストアを通じて再配布する必要のない小さな機能の追加や削除の場合に便利です。

#### <a name="codepush-references"></a>CodePush の参考資料

- [Visual Studio App Center でのサインアップ](https://appcenter.ms/signup)
- [App Center で CodePush を使ってみる](/appcenter/distribution/codepush/)
- [CodePush CLI](/appcenter/distribution/codepush/cli)
