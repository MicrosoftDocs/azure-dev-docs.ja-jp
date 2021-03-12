---
title: Azure Toolkit for Eclipse のインストール
description: Azure Toolkit for Eclipse プラグインをインストールして、クラウド アプリケーションを作成し、Azure にデプロイする方法を説明します。
documentationcenter: java
ms.assetid: 9e93ff6a-f42b-4d99-b55b-624136b4a730
ms.date: 08/25/2020
ms.service: multiple
ms.tgt_pltfrm: multiple
ms.topic: article
ms.custom: devx-track-java
ms.openlocfilehash: 57dbe81eb0ce2b8b7c8766934b765dd0f057e51b
ms.sourcegitcommit: 576c878c338d286060010646b96f3ad0fdbcb814
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102118317"
---
# <a name="installing-the-azure-toolkit-for-eclipse"></a>Azure Toolkit for Eclipse のインストール

Azure Toolkit for Eclipse が提供する機能により、軽量で可用性が高くスケーラブルな Java Web アプリと HDInsight Spark ジョブを、Eclipse 開発環境を使用して簡単に作成、開発、構成、テストでき、さらに Azure にデプロイできます。

> [!NOTE] 
> 
> Azure Toolkit for Eclipse はオープン ソース プロジェクトであり、そのソース コードは、GitHub のプロジェクト サイトから MIT License で入手できます。URL は次のとおりです。 
> 
> <https://github.com/microsoft/azure-tools-for-java> 
> 

[!INCLUDE [basic-prerequisites](includes/basic-prerequisites.md)]

Azure Toolkit for Eclipse をインストールする方法は 2 つあります。具体的には、**Eclipse Marketplace** にアクセスする方法と、[ヘルプ] メニューの **[新しいソフトウェアのインストール]** オプションを使用する方法です。 以降のセクションでは、両方のインストール方法を説明します。

## <a name="eclipse-marketplace"></a>Eclipse Marketplace

Eclipse IDE の Eclipse Marketplace ウィザードでは、ユーザーは [Eclipse Marketplace](https://marketplace.eclipse.org/) を参照して、ソリューションをインストールできます。 次の 2 つのオプションを使用すると、Eclipse Marketplace に移動できます。

   * 次のボタンを実行中の Eclipse ワークスペースにドラッグします。 このボタンにより、Azure Toolkit for Eclipse が既に選択された状態で Eclipse Marketplace が開きます。

      [![実行している Eclipse* ワークスペースにドラッグします。*Eclipse Marketplace Client が必要です](https://marketplace.eclipse.org/sites/all/themes/solstice/public/images/marketplace/btn-install.png)](http://marketplace.eclipse.org/marketplace-client-intro?mpc_install=1919278 "実行している Eclipse* ワークスペースにドラッグします。*Eclipse Marketplace Client が必要です")

   * Eclipse IDE で、 **[ヘルプ]** メニューをクリックし、**Eclipse Marketplace** に移動し、"Azure Toolkit for Eclipse" を検索し、 **[インストール]** をクリックします。

      :::image type="content" source="media/installation/eclipse-marketplace-button.png" alt-text="Marketplace ウィンドウの [ヘルプ] メニュー。"::: 

1. Eclipse Marketplace ウィザードがポップアップ表示され、インストール手順と、インストールされるコンポーネントの一覧が示されます。 すべての機能が選択されていることを確認し、 **[Confirm >]\(確認 >\)** をクリックします。

   | 機能 | 説明 | 
   |---|---| 
   | **Application Insights Plugin for Java** | アプリケーションとサーバー インスタンスに対する Azure のテレメトリ ログと分析サービスを使用できるようにします。 | 
   | **Azure Common Plugin** | 他のツールキット コンポーネントで必要な一般的な機能を提供します。 | 
   | **Azure Container Tools for Eclipse** | .WAR を Docker コンテナーとして Docker マシンに構築およびデプロイできるようにします。 |
   | **Azure Explorer for Eclipse** | Azure リソースを管理するためのエクスプローラー スタイルのインターフェイスを提供します。 | 
   | **Azure HDInsight plugin for Java** | Scala で Apache Spark アプリケーションを開発できるようにします。 |
   | **Microsoft JDBC Driver 6.1 for SQL Server** | SQL Server と Microsoft Azure SQL Database 向けに、Java Platform Enterprise Edition 8 の JDBC API を提供します。 | 
   | **Package for Microsoft Azure Libraries for Java** | Microsoft Azure の各種サービス (ストレージ、Service Bus、サービス ランタイムなど) にアクセスするための API を提供します。 | 
   | **WebApp Plugin for Eclipse** | Web アプリケーションを Azure App Services としてデプロイできるようにします。 | 

1. **[Review Licenses (ライセンスの確認)]** ダイアログで、ライセンス契約の条項を確認します。 ライセンス契約の条項に同意する場合は、 **[I accept the terms of the license agreements]\(ライセンス契約の条項に同意する\)** をクリックし、 **[Finish]\(完了\)** をクリックします。 

   > [!NOTE]
   > Eclipse ワークスペースの右下隅でインストールの進行状況を確認できます。

4. インストールが完了すると、Eclipse IDE を再起動してソフトウェアの更新を適用するよう求めるプロンプトが表示されます。 **[今すぐ再起動する]** をクリックします。

## <a name="install-new-software"></a>新しいソフトウェアのインストール

Azure Toolkit for Eclipse は、新しいソフトウェアの形式で *[ヘルプ]* メニューから直接インストールできます。

1. **[ヘルプ]** メニューをクリックし、 **[新しいソフトウェアのインストール]** をクリックします。

   :::image type="content" source="media/installation/eclipse-install-software-button.png" alt-text="[新しいソフトウェアのインストール]、[ヘルプ] メニュー。"::: 

1. **[利用可能なソフトウェア]** ダイアログで、 **[操作]** テキスト ボックスに `http://azuredownloads.blob.core.windows.net/eclipse/` を入力します。

1. **[Name]\(名前\)** ウィンドウで、 **[Azure Toolkit for Java]** をオンにし、 **[Contact all update sites during install to find required software]\(インストール中にすべての更新サイトに接続して必要なソフトウェアを調べる\)** をオフにします。 画面は次のようになります。

   ![Azure Toolkit for Eclipse のインストール][02]

1. **[Azure Toolkit for Java]** を展開すると、インストールされるコンポーネントの一覧が表示されます。次に例を示します。

   | 機能 | 説明 | 
   |---|---| 
   | **Application Insights Plugin for Java** | アプリケーションとサーバー インスタンスに対する Azure のテレメトリ ログと分析サービスを使用できるようにします。 | 
   | **Azure Common Plugin** | 他のツールキット コンポーネントで必要な一般的な機能を提供します。 | 
   | **Azure Container Tools for Eclipse** | .WAR を Docker コンテナーとして Docker マシンに構築およびデプロイできるようにします。 |
   | **Azure Explorer for Eclipse** | Azure リソースを管理するためのエクスプローラー スタイルのインターフェイスを提供します。 | 
   | **Azure HDInsight plugin for Java** | Scala で Apache Spark アプリケーションを開発できるようにします。 |
   | **Microsoft JDBC Driver 6.1 for SQL Server** | SQL Server と Microsoft Azure SQL Database 向けに、Java Platform Enterprise Edition 8 の JDBC API を提供します。 | 
   | **Package for Microsoft Azure Libraries for Java** | Microsoft Azure の各種サービス (ストレージ、Service Bus、サービス ランタイムなど) にアクセスするための API を提供します。 | 
   | **WebApp Plugin for Eclipse** | Web アプリケーションを Azure App Services としてデプロイできるようにします。 | 

1. **[次へ]** をクリックします。 (ツールキットのインストール時に異常な遅延が発生する場合は、 **[Contact all update sites during install to find required software (インストール中にすべての更新サイトに接続して必要なソフトウェアを調べる)]** がオフになっていることを確認してください)。

1. **[Install Details (インストールの詳細)]** ダイアログで、 **[Next (次へ)]** をクリックします。

1. **[Review Licenses (ライセンスの確認)]** ダイアログで、ライセンス契約の条項を確認します。 ライセンス契約の条項に同意する場合は、 **[I accept the terms of the license agreements (ライセンス契約の条項に同意する)]** をクリックし、 **[Finish (完了)]** をクリックします。 (残りの手順は、ライセンス契約の条項に同意したことを前提としています。 ライセンス契約の条項に同意しない場合はインストール プロセスを終了します。)

   > [!NOTE]
   > Eclipse ワークスペースの右下隅でインストールの進行状況を確認できます。

1. インストールを完了するために Eclipse を再起動するよう求めるメッセージが表示されたら、**[Restart Now]** をクリックします。

## <a name="next-steps"></a>次のステップ

[!INCLUDE [additional-resources](includes/additional-resources.md)]

<!-- URL List -->

<!-- Legacy MSDN URL = https://msdn.microsoft.com/library/azure/hh690946.aspx -->

<!-- IMG List -->
[02]: media/installation/eclipse-installation-02.png
