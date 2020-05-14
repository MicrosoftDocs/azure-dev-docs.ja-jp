---
author: edburns
ms.author: edburns
ms.date: 1/21/2020
ms.openlocfilehash: 73f3893f22764280ed5d7c030e7c8eab7d2bf6b1
ms.sourcegitcommit: 226ebca0d0e3b918928f58a3a7127be49e4aca87
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/08/2020
ms.locfileid: "82988788"
---
アプリケーションにホスト OS に依存するコードが含まれている場合は、それをリファクタリングしてそれらの依存関係を削除する必要があります。 たとえば、ファイル システムのパスで `/` または `\` を使用している場合、[`File.Separator`](https://docs.oracle.com/javase/8/docs/api/java/io/File.html#separator) または [`Paths.get`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Paths.html#get-java.lang.String-java.lang.String...-) に置き換えることが必要になる可能性があります。
