---
title: Python 用 Azure SDK ライブラリ パッケージをインストールする方法
description: pip を使用して Azure SDK または Python ライブラリをインストール、アンインストール、および検証する方法。 特定のバージョンおよびプレビュー パッケージをインストールする方法の詳細も含まれています。
ms.date: 01/22/2021
ms.topic: conceptual
ms.custom: devx-track-python
adobe-target: true
ms.openlocfilehash: 7c29e9793fbf995276ea2b860d8c8743bd40d3b3
ms.sourcegitcommit: b380f6e637b47e6e3822b364136853e1d342d5cd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/14/2021
ms.locfileid: "100395207"
---
# <a name="how-to-install-azure-library-packages-for-python"></a>Python 用 Azure ライブラリ パッケージをインストールする方法

Azure SDK for Python は、[パッケージ インデックス](azure-sdk-library-package-index.md)に記載されている多数の個別ライブラリから構成されています。 `pip install` を使用して、プロジェクトに必要な特定のライブラリ パッケージをインストールします。

これらのライブラリを使用すると、(名前に `-mgmt` が付いた管理ライブラリを使用して) Azure サービス上のリソースをプロビジョニングして管理し、(クライアント ライブラリを使用して) アプリ コードからそれらのリソースに接続できます。

## <a name="install-the-latest-version-of-a-library"></a>ライブラリの最新バージョンをインストールする

```cmd
pip install azure-storage-blob
```

```cmd
pip install azure-mgmt-storage
```

`pip install` は、現在の Python 環境にライブラリの最新バージョンを取得します。

Linux システムでは、ユーザーごとに別々にライブラリをインストールする必要があります。 `sudo pip install` を使用してすべてのユーザーのライブラリをインストールすることはサポートされていません。

## <a name="install-specific-library-versions"></a>特定のライブラリ バージョンをインストールする

```cmd
pip install azure-storage-blob==12.0.0
```

```cmd
pip install azure-mgmt-storage==10.0.0
```

`pip install` を使用して、コマンド ラインで目的のバージョンを指定します。

## <a name="install-preview-packages"></a>プレビュー パッケージをインストールする

```cmd
pip install --pre azure-storage-blob
```

```cmd
pip install --pre azure-mgmt-storage
```

ライブラリの最新のプレビューをインストールするには、コマンド ラインで `--pre` フラグを含めます。

Microsoft では、今後予定されている機能をサポートするプレビュー ライブラリ パッケージを定期的にリリースしています。ライブラリは変更される可能性があるため、実稼働プロジェクトでは使用できないことに注意してください。

## <a name="verify-a-library-installation"></a>ライブラリのインストールの確認

```cmd
pip show azure-storage-blob
```

```cmd
pip show azure-mgmt-storage
```

`pip show <library>` を使用して、ライブラリがインストールされていることを確認します。 ライブラリがインストールされている場合は、このコマンドによってバージョンとその他の概要情報が表示されます。それ以外の場合、コマンドでは何も表示されません。

`pip freeze` または `pip list` を使用して、現在の Python 環境にインストールされているすべてのライブラリを表示することもできます。

## <a name="uninstall-a-library"></a>ライブラリをアンインストールする

```cmd
pip uninstall azure-storage-blob
```

ライブラリをアンインストールするには、`pip uninstall <library>` を使用します。
