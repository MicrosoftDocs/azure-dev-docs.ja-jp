---
title: Visual Studio App Center を使用してアプリケーションのリアルタイムの障害とエラーを監視する
description: モバイル アプリケーションの障害とエラーを監視するサービスとしての App Center について説明します。
author: codemillmatt
ms.assetid: 12a8a079-9b3c-4faf-8588-ccff02097224
ms.service: mobile-services
ms.topic: article
ms.date: 06/08/2020
ms.author: masoucou
ms.openlocfilehash: ad2a3ae817f177a54cdbf63093023c06124063a4
ms.sourcegitcommit: f02ff55cef47581303a217cf1d310879cd722237
ms.contentlocale: ja-JP
ms.lasthandoff: 06/09/2020
ms.locfileid: "84632654"
---
# <a name="monitor-failures-and-errors-in-real-time-for-your-mobile-application"></a>モバイル アプリケーションの障害とエラーをリアルタイムで監視する

アプリケーションが何千人ものユーザーによって使用されている場合、バグが発生しやすくなります。 ユーザーは、予期しないアプリケーションの障害に遭遇する可能性があります。 ユーザーにとっては、必要なのは動作する信頼性の高いアプリケーションです。 バグや障害が発生するアプリケーションでは、優れたユーザー エクスペリエンスが得られません。 失望したユーザーは、アプリをアンインストールしたり、低評価のレビューを残したり、公の場でエクスペリエンスに関して批判や不満を述べたりします。

モバイル アプリケーションでは、実際の障害の動作を理解することが不可欠です。 開発者としての仕事は、App Store にリリースすれば完了するわけではありません。 アプリの安定性を確保し、ユーザーの満足度を維持するには、パフォーマンスを継続的に監視して測定することが重要です。 必要なのは、アプリケーションを監視する方法です。 なんらかの問題が発生した場合に状況をよく理解する必要があります。これにより、ビジネス ニーズにとって重大な問題を優先して修正することができます。

## <a name="importance-of-failure-monitoring"></a>障害の監視の重要性

開発者は、次のことを行う必要があります。

- アプリケーションの正常性と、ユーザーに対してどのように動作しているかについて、詳細を知る。
- ユーザーのデバイスで障害が発生した原因と場所について、根本的な原因と理由を見つける。
- アプリケーションの安定性を確保するために、先を見越して問題のトリアージと障害の解決を行う。
- ユーザーがアプリケーションを引き続き使用できるよう優れたユーザー エクスペリエンスを提供するため、改良したアプリケーションの構築を早いサイクルで繰り返す。

次のサービスを活用して、アプリケーションの障害を監視し、問題を診断して、迅速に修正することで、安定した高品質のモバイル アプリケーション エクスペリエンスを提供します。

## <a name="visual-studio-app-center"></a>Visual Studio App Center

[App Center Diagnostics](/appcenter/diagnostics/) サービスは、アプリケーションの正常性を監視するのに役立ちます。 詳細な分析情報が提供されるため、アプリケーションに障害が発生した状況とその原因を理解することができます。 また、App Center Diagnostics は、障害レポートの優先順位付けと管理にも役立ちます。

### <a name="visual-studio-app-center-features"></a>Visual Studio App Center の機能

- リアルタイムでの障害とエラーのスマートな自動グループ化。必要とする最も重大な問題と関連情報が強調表示されます。
- スタック トレースとデバイスに関する完全なデータ。根本的な原因を特定するのに役立つコンテキストと明確さが提供されます。
- 強力な検索機能。最も気にかかる問題を見つけるのに役立ちます。
- 分析の統合。ユーザーの動作を理解し、障害につながるイベントを確認できます。
- カスタム プロパティ、ユーザー ID、および添付ファイル。問題の診断に役立つコンテキストを追加できます。
- アラート、通知、およびバグ トラッカーの統合。ユーザーに影響を与える新規および予期しない障害を確実に把握することができます。
- 単独の、1 行の SDK 統合。このサービスの使用を数分以内に開始することができます。
- iOS、Android、macOS、tvOS、Xamarin、React Native、Unity、Cordova、WPF、および WinForms でのプラットフォーム サポート。

### <a name="visual-studio-app-center-references"></a>Visual Studio App Center の参考資料

- [Visual Studio App Center でのサインアップ](https://appcenter.ms/signup?utm_source=Mobile%20Development%20Docs&utm_medium=Azure&utm_campaign=New%20azure%20docs)
- [App Center Diagnostics を使ってみる](/appcenter/diagnostics/)
