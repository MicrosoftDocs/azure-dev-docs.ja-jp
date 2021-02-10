---
title: VSCode を使用して GitHub リポジトリをクローンする
description: Visual Studio Code を使用して、GitHub からローカル コンピューターにパブリック リポジトリをクローンします。
ms.topic: how-to
ms.date: 01/28/2021
ms.custom: devx-track-js
ms.openlocfilehash: f3bf194f7520779b92d4cfb5966eedb0f599edea
ms.sourcegitcommit: 3f8aa923e4626b31cc533584fe3b66940d384351
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/01/2021
ms.locfileid: "99224806"
---
# <a name="clone-and-use-a-github-repository-in-visual-studio-code"></a>Visual Studio Code で GitHub リポジトリをクローンして使用する

Visual Studio Code を使用して、GitHub からローカル コンピューターにパブリック リポジトリをクローンする手順について説明します。

リポジトリを使用した Visual Studio Code での作業には、個別のツールを使用します。

|アイコン|Information|[アクセス元](https://code.visualstudio.com/docs/getstarted/userinterface)|
|--|--|--|
|| [Git CLI](https://code.visualstudio.com/docs/getstarted/userinterface#_command-palette)|コマンド パレット - F1|
|:::image type="content" source="../../media/how-to-clone-github-repo/git-commit-icon-activity-bar.png" alt-text="ソース管理のアイコン。":::|ソース管理の拡張機能|アクティビティ バー|
|:::image type="content" source="../../media/how-to-clone-github-repo/github-icon-activity-bar.png" alt-text="GitHub の PR と問題のアイコン":::|GitHub 拡張機能|アクティビティ バー|

次の手順では、[Visual Studio Code ユーザー インターフェイス](https://code.visualstudio.com/docs/getstarted/userinterface)の指定された部分を使用します。 

## <a name="use-command-palette-to-clone-repository"></a>コマンド パレットを使用してリポジトリをクローンする

最初に、次の手順でサンプル プロジェクトをダウンロードします。

1. **F1** キーを押してコマンド パレットを表示します。

1. コマンド パレットのプロンプトに「`gitcl`」と入力し、**Git: Clone** コマンドを選択して、**Enter** キーを押します。

    ![Visual Studio Code コマンド パレットのプロンプトに gitcl コマンドを入力](../../media/how-to-clone-github-repo/visual-studio-code-git-clone.png)

1. **[リポジトリの URL]** の入力を求められたら、GitHub リポジトリの URL を入力し、**Enter** キーを押します。

1. プロジェクトの複製先となるローカル ディレクトリを選択 (または作成) します。

    ![Visual Studio Code エクスプローラー](../../media/how-to-clone-github-repo/visual-studio-code-explorer.png)

## <a name="create-a-branch-for-changes-with-git-cl"></a>Git CL を使用して変更するためのブランチを作成する

コマンド パレットで Git を使用して、新しいブランチを作成します。

1. **F1** キーを押してコマンド パレットを表示します。
1. `git branch` を検索して、`Git: Create Branch` を選択します。

    :::image type="content" source="../../media/how-to-clone-github-repo/git-cli-branch-list.png" alt-text="「git branch」を検索し、次を選択します: [Git:Create Branch]。":::

1. 新しいブランチ名を入力します。 ブランチ名はステータス バーに表示されます。 

    :::image type="content" source="../../media/how-to-clone-github-repo/git-branch-status-bar-visual-studio-code.png" alt-text="ブランチ名はステータス バーに表示されます。":::

## <a name="create-a-branch-from-status-bar"></a>ステータス バーからブランチを作成する

1. ステータス バーでブランチ名を選択します。 

    通常、ステータス バーは Visual Studio Code の下部にあります。 

1. コマンド パレットで、 **[+ + 新しいブランチの作成]** を選択します。
1. 新しいブランチ名を入力します。 

1. 新しいブランチ名を入力します。 ブランチ名はステータス バーに表示されます。 

    :::image type="content" source="../../media/how-to-clone-github-repo/git-branch-status-bar-visual-studio-code.png" alt-text="ブランチ名はステータス バーに表示されます。":::

## <a name="commit-changes-with-git"></a>Git で変更をコミットする 

ブランチに変更を加えたら、変更をコミットします

1. アクティビティ バーに切り替えて、Git アイコンを選択します。

1. **[メッセージ]** ボックスにコミット メッセージを入力し **Ctrl**+**Enter** を押すか、ソース管理バーのチェック マークをオンにします。

    ![yarn.lock ファイルを Git に追加](../../media/how-to-clone-github-repo/visual-studio-code-add-yarn-lock.png)

## <a name="push-a-local-branch-to-remote-from-status-bar"></a>ステータス バーからローカル ブランチをリモートにプッシュする

1. ブランチ名の右側にあるプッシュ アイコンを選択します。 
1. ポップアッ プ ボックスからリモート名を選択します。 リモートが 1 つしかない場合、リモート名の選択は求められません。 

## <a name="push-a-local-branch-to-remote-from-the-source-control-extension"></a>ソース管理の拡張機能からローカル ブランチをリモートにプッシュする
1. アクティビティ バーからソース管理アイコンを選択します。 
1. 省略記号 (...) を選択し、 **[プル]、[プッシュ]** を選択して、 **[プッシュ先...]** を選択します。 
1. ポップアッ プ ボックスからリモート名を選択します。 リモートが 1 つしかない場合、リモート名の選択は求められません。 

## <a name="next-steps"></a>次のステップ

* [Web アプリをデプロイする](../deploy-web-app.md)方法
