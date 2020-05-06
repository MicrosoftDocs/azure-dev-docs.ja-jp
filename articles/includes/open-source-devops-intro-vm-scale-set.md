---
author: tomarchermsft
ms.service: ansible
ms.topic: include
ms.date: 04/22/2019
ms.author: tarcher
ms.openlocfilehash: 2bf0a888cd2d4e49899b384f643430f5e30c95e8
ms.sourcegitcommit: be67ceba91727da014879d16bbbbc19756ee22e2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/05/2020
ms.locfileid: "82109888"
---
[Azure Virtual Machine Scale Sets](https://docs.microsoft.com/azure/virtual-machine-scale-sets/overview) は、負荷分散が行われる同一の VM のグループを構成するための Azure 機能です。 スケール セットに追加コストはなく、仮想マシンから構築されます。 ユーザーは、VM インスタンス、ロード バランサー、マネージド ディスク ストレージなど、基本的なコンピューティング リソースに対してのみ支払います。 スケール セットには、アプリケーションの実行とスケーリングを行うための管理レイヤーと自動化レイヤーがあります。 代わりに手動で個々 の VM を作成し管理できます。 ただし、スケール セットの使用には、2 つの主な利点があります。 それらは Azure に組み込まれ、アプリケーションのニーズを満たすように自動的に仮想マシンを拡大縮小します。