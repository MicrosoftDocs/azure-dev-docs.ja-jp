---
title: バックエンド サービス経由で Azure Notification Hubs を使用して Xamarin.Forms アプリにプッシュ通知を送信する | Microsoft Docs
description: バックエンド サービス経由で Azure Notification Hubs を使用する Xamarin.Forms アプリにプッシュ通知を送信する方法を学習します。
author: mikeparker104
ms.service: mobile-services
ms.topic: tutorial
ms.date: 07/27/2020
ms.author: miparker
ms.openlocfilehash: d42f44226169232c18ebdd8039f0cf9ceb1018fc
ms.sourcegitcommit: 54f976887d218aaabd94371e24809716da8cf86e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/04/2021
ms.locfileid: "99554319"
---
# <a name="tutorial-send-push-notifications-to-xamarinforms-apps-using-azure-notification-hubs-via-a-backend-service"></a>チュートリアル:バックエンド サービス経由で Azure Notification Hubs を使用して Xamarin.Forms アプリにプッシュ通知を送信する  

[![サンプルのダウンロード](media/download.png) サンプルをダウンロードします](https://github.com/xamcat/mobcat-samples/tree/master/notification_hub_backend_service)  

> [!div class="op_single_selector"]
>
> * [Xamarin.Forms](notification-hubs-backend-service-xamarin-forms.md)
> * [Flutter](notification-hubs-backend-service-flutter.md)
> * [React Native](notification-hubs-backend-service-react-native.md)

このチュートリアルでは、[Azure Notification Hubs](/azure/notification-hubs/notification-hubs-push-notification-overview) を使用して、**Android** と **iOS** をターゲットとする [Xamarin.Forms](https://dotnet.microsoft.com/apps/xamarin/xamarin-forms) アプリケーションにプッシュ通知を送信します。  

[!INCLUDE [Notification Hubs Backend Service Introduction](includes/notification-hubs-backend-service-introduction.md)]

このチュートリアルでは、次の手順について説明します。

> [!div class="checklist"]
>
> * [プッシュ通知サービスと Azure Notification Hubs を設定します。](#set-up-push-notification-services-and-azure-notification-hub)
> * [ASP.NET Core Web API バックエンド アプリケーションを作成します。](#create-an-aspnet-core-web-api-backend-application)
> * [クロスプラットフォームの Xamarin.Forms アプリケーションを作成します。](#create-a-cross-platform-xamarinforms-application)
> * [プッシュ通知用のネイティブ Android プロジェクトを構成します。](#configure-the-native-android-project-for-push-notifications)
> * [プッシュ通知用のネイティブ iOS プロジェクトを構成します。](#configure-the-native-ios-project-for-push-notifications)
> * [ソリューションをテストします。](#test-the-solution)

## <a name="prerequisites"></a>前提条件

手順を進めてゆくには、以下が必要です。

* リソースを作成および管理できる [Azure サブスクリプション](https://azure.microsoft.com/free/dotnet)。
* [Visual Studio for Mac](https://visualstudio.microsoft.com/vs/mac/) がインストールされている Mac または [Visual Studio 2019](https://visualstudio.microsoft.com/vs) を実行している PC。
* [Visual Studio 2019](https://visualstudio.microsoft.com/vs) のユーザーは、 **.NET によるモバイル開発** と **ASP.NET および Web の開発** のワークロードもインストールしておく必要があります。
* **Android** (物理デバイスまたはエミュレーター デバイス) または **iOS** (物理デバイスのみ) のいずれかでアプリを実行する機能。

Android の場合は次のものが必要です。

* 開発者用のロック解除済みの物理デバイスまたはエミュレーター *(API 26 以降を実行していて Google Play 開発者サービスがインストールされているもの)* 。

iOS の場合は次のものが必要です。

* アクティブな [Apple Developer アカウント](https://developer.apple.com)。
* [開発者アカウントに登録](https://help.apple.com/developer-account/#/dev40df0d9fa)されている物理 iOS デバイス *(iOS 13.0 以降を実行しているもの)* 。
* [物理デバイスでアプリを実行](https://help.apple.com/xcode/mac/current/#/dev5a825a1ca)できるように **.p12** [開発証明書](https://help.apple.com/developer-account/#/dev04fd06d56)が **キーチェーン** にインストールされている。

> [!NOTE]
> iOS シミュレーターではリモート通知がサポートされていないため、このサンプルを iOS で試すときには物理デバイスが必要です。 ただし、このチュートリアルを完了するために、アプリを **Android** と **iOS** の両方で実行する必要はありません。

以前に経験がなくとも、この第一原理の例に記載されている手順に従うことができます。 ただし、以下の面について知識があると役立ちます。

* [Apple Developer ポータル](https://developer.apple.com)
* [ASP.NET Core](/aspnet/core/introduction-to-aspnet-core) と [Web API](https://dotnet.microsoft.com/apps/aspnet/apis)
* [Google Firebase Console](https://console.firebase.google.com/u/0/)
* [Microsoft Azure](https://portal.azure.com) と、[Azure Notification Hubs を使用して iOS アプリにプッシュ通知を送信すること](/azure/notification-hubs/ios-sdk-get-started)。
* [Xamarin](https://dotnet.microsoft.com/apps/xamarin) と [Xamarin.Forms](https://dotnet.microsoft.com/apps/xamarin/xamarin-forms)。

> [!IMPORTANT]
> 示されている手順は、[Visual Studio for Mac](https://visualstudio.microsoft.com/vs/mac/) に固有のものです。 [Visual Studio 2019](https://visualstudio.microsoft.com/vs) を使用して従うこともできますが、調整すべきいくつかの違いがある場合があります。 たとえば、ユーザー インターフェイスとワークフローの説明、テンプレート名、環境構成などです。

## <a name="set-up-push-notification-services-and-azure-notification-hub"></a>プッシュ通知サービスと Azure Notification Hubs を設定する

このセクションでは、 **[Firebase Cloud Messaging (FCM)](https://firebase.google.com/docs/cloud-messaging)** と **[Apple Push Notification Services (APNS)](https://developer.apple.com/library/archive/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html)** をセットアップします。 次に、通知ハブを作成し、それらのサービスと連携するように構成します。

[!INCLUDE [Create a Firebase project and enable Firebase Cloud Messaging](includes/notification-hubs-common-enable-firebase-cloud-messaging.md)]

[!INCLUDE [Enable Apple Push Notifications](includes/notification-hubs-common-enable-apple-push-notifications.md)]

[!INCLUDE [Create a notification hub](includes/notification-hubs-common-create-notification-hub.md)]

[!INCLUDE [Configure your notification hub with APNs information](includes/notification-hubs-common-configure-with-apns-information.md)]

[!INCLUDE [Configure your notification hub with FCM information](includes/notification-hubs-common-configure-with-fcm-information.md)]

## <a name="create-an-aspnet-core-web-api-backend-application"></a>ASP.NET Core Web API バックエンド アプリケーションを作成する

このセクションでは、[デバイスの登録](/azure/notification-hubs/notification-hubs-push-notification-registration-management#what-is-device-registration)と、Xamarin.Forms モバイル アプリへの通知の送信を処理する [ASP.NET Core Web API](https://dotnet.microsoft.com/apps/aspnet/apis) バックエンドを作成します。

[!INCLUDE [Create an ASP.NET Core Web API backend application](includes/notification-hubs-backend-service-web-api.md)]

## <a name="create-a-cross-platform-xamarinforms-application"></a>クロスプラットフォームの Xamarin.Forms アプリケーションを作成する

このセクションでは、クロスプラットフォーム方式でプッシュ通知を実装する [Xamarin.Forms](https://dotnet.microsoft.com/apps/xamarin/xamarin-forms) モバイル アプリケーションを構築します。

[!INCLUDE [Sample application generic overview](includes/notification-hubs-backend-service-sample-app-overview.md)]

[!INCLUDE [Create Xamarin.Forms application](includes/notification-hubs-backend-service-sample-app-xamarin-forms.md)]

## <a name="configure-the-native-android-project-for-push-notifications"></a>プッシュ通知用のネイティブ Android プロジェクトを構成する

[!INCLUDE [Configure the native Android project](includes/notification-hubs-backend-service-configure-xamarin-android.md)]

## <a name="configure-the-native-ios-project-for-push-notifications"></a>プッシュ通知用のネイティブ iOS プロジェクトを構成する

[!INCLUDE [Configure the native iOS project](includes/notification-hubs-backend-service-configure-xamarin-ios.md)]

## <a name="test-the-solution"></a>ソリューションをテストする

これで、バックエンド サービス経由で通知の送信をテストできるようになりました。

[!INCLUDE [Testing the solution](includes/notification-hubs-backend-service-testing.md)]

## <a name="troubleshooting"></a>トラブルシューティング

[!INCLUDE [Troubleshooting](includes/notification-hubs-backend-service-troubleshooting.md)]

## <a name="related-links"></a>関連リンク

* [Azure Notification Hubs の概要](/azure/notification-hubs/notification-hubs-push-notification-overview)
* [Visual Studio for Mac のインストール](/visualstudio/mac/installation)
* [Windows への Xamarin のインストール](/xamarin/get-started/installation/windows)
* [バックエンド操作用の Notification Hubs SDK](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/)
* [GitHub 上の Notification Hubs SDK](https://github.com/Azure/azure-notificationhubs)
* [アプリケーション バックエンドへの登録](/azure/notification-hubs/notification-hubs-ios-aspnet-register-user-from-backend-to-push-notification)
* [登録管理](/azure/notification-hubs/notification-hubs-push-notification-registration-management)
* [タグの使用](/azure/notification-hubs/notification-hubs-tags-segment-push-message)
* [カスタム テンプレートの使用](/azure/notification-hubs/notification-hubs-templates-cross-platform-push-messages)

## <a name="next-steps"></a>次のステップ

バックエンド サービスを介して通知ハブに接続された基本的な Xamarin.Forms アプリがあって、通知を送受信できるようになっているはずです。

[!INCLUDE [Next steps](includes/notification-hubs-backend-service-next-steps.md)]
