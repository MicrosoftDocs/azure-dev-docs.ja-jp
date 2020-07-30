---
author: mikeparker104
ms.author: miparker
ms.date: 07/27/2020
ms.service: mobile-services
ms.topic: include
ms.openlocfilehash: fb2311fe9256daa53b2687c43b508c4af4f97bd2
ms.sourcegitcommit: cf23d382eee2431a3958b1c87c897b270587bde0
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/29/2020
ms.locfileid: "87401693"
---
### <a name="create-a-firebase-project-and-enable-firebase-cloud-messaging-for-android"></a>Firebase プロジェクトを作成し、Android 用に Firebase Cloud Messaging を有効にする

1. [Firebase コンソール](https://firebase.google.com/console/)にサインインします。 新しい Firebase プロジェクトを作成し、 **[プロジェクト名]** に「**PushDemo**」と入力します。

    > [!NOTE]
    > 一意の名前が生成されます。 既定では、指定した名前を小文字にしたものと生成された数値がダッシュで区切られます。 グローバルに一意になるように指定する場合は、この設定を変更できます。

1. プロジェクトを作成した後、 **[Add Firebase to your Android app]\(Android アプリに Firebase を追加する\)** を選択します。

    ![Android アプリへの Firebase の追加](../media/notification-hubs-add-firebase-to-android-app.png)

1. **[Android アプリへの Firebase の追加]** ページで、次の手順を実行します。
    1. **[Android パッケージ名]** に、お使いのパッケージの名前を入力します。 (例: `com.<organization_identifier>.<package_name>`)。

        ![パッケージ名を指定する](../media/specify-package-name-firebase-cloud-messaging-settings.png)
    1. **[アプリの登録]** を選択します。  
    1. **[google-services.json のダウンロード]** を選択します。 次に、後で使用するためにファイルをローカル フォルダーに保存し、 **[次へ]** を選択します。  

        ![google-services.json をダウンロードする](../media/download-google-service-button.png)
    1. **[次へ]** を選択します。
    1. **[Continue to console] (コンソールに続行する)** を選択します。

        > [!NOTE]
        > *インストールの確認*のために **[Continue to console] (コンソールに続行する)** ボタンが有効になっていない場合は、 **[この手順をスキップする]** を選択します。

1. Firebase コンソールで、プロジェクトの歯車アイコンを選択します。 次に、 **[Project Settings]\(プロジェクト設定\)** を選択します。

    ![[Project Settings]\(プロジェクト設定\) の選択](../media/notification-hubs-firebase-console-project-settings.png)

    > [!NOTE]
    > **google-services.json** ファイルをダウンロードしていない場合は、このページでダウンロードできます。

1. 上部にある **[クラウド メッセージング]** タブに切り替えます。 後で使用するために、**サーバー キー**をコピーし、保存します。 この値を使用して通知ハブを構成します。

    ![サーバー キーのコピー](../media/server-key.png)
