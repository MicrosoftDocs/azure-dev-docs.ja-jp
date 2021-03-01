---
ms.custom: devx-track-js
ms.topic: include
ms.date: 02/08/2021
ms.openlocfilehash: 693e78bef335871dbee846666f889ad1cf2982b1
ms.sourcegitcommit: 98a7e855206ff463c1d95f93c23dd665b26a0aa1
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/09/2021
ms.locfileid: "100004997"
---
## <a name="create-azure-database-with-visual-studio-code-extension"></a>Visual Studio Code 拡張機能を使用して Azure データベースを作成する

次の種類のリソースに対して、この手順を使用します。

* Azure Cosmos DB for MongoDB API
* SQL
* Azure Table
* Gremlin
* [PostgreSQL](#create-a-postgresql-database) 

### <a name="create-a-postgresql-database"></a>PostgreSQL データベースを作成する

1. Visual Studio Code 用の [Azure データベース](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-cosmosdb)拡張機能をインストールします。
1. Visual Studio Code で、[アクティビティ バー](https://code.visualstudio.com/docs/getstarted/userinterface)から **[Azure]** を選択し、[サイドバー](https://code.visualstudio.com/docs/getstarted/userinterface)から **[データベース]** を選択します。
1. サブスクリプション名を右クリックし、 **[サーバーの作成]** を選択します。
1. 一覧から **[PostgreSQL]** を選択します。 

    :::image type="content" source="../media/howto-visual-studio-code/create-azure-database-server.png" alt-text="一覧から [PostgreSQL] を選択します。":::

1. PostgreSQL サーバーの名前を入力します。 この名前は、接続文字列の一部として使用されます。 
1. 管理者のユーザー名を入力します。 
1. 管理者パスワードを入力し、次の画面でもう一度入力して確認します。 
1. ファイアウォール規則として追加する現在の IP アドレスを選択します。 
1. リソース グループ名を選択するか、新しく作成します。 
1. お使いのサーバーの Azure リージョンを選択します。 
1. 通知ウィンドウに状態が表示されます。 

    :::image type="content" source="../media/howto-visual-studio-code/create-azure-database-server-status.png" alt-text="通知ウィンドウに状態が表示されます。":::