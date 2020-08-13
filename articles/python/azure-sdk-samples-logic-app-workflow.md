---
title: ロジック アプリ ワークフローを作成する
description: ロジック アプリ ワークフローを作成する
ms.topic: conceptual
ms.date: 6/15/2017
ms.custom: devx-track-python
ms.openlocfilehash: c14a575cb1a1dbc8caa88c8fe439fedbe0886c57
ms.sourcegitcommit: 980efe813d1f86e7e00929a0a3e1de83514ad7eb
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/07/2020
ms.locfileid: "87982644"
---
# <a name="create-a-logic-app-workflow"></a>ロジック アプリ ワークフローを作成する

次のコードでは、ロジック アプリ ワークフローを作成します。

```python
from azure.mgmt.logic.models import Workflow

group_name = 'myresourcegroup'
workflow_name = '12HourHeartBeat'
logic_client.workflows.create_or_update(
    group_name,
    workflow_name,
    Workflow(
        location = 'East US',
        definition={
            "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
            "contentVersion": "1.0.0.0",
            "parameters": {},
            "triggers": {},
            "actions": {},
            "outputs": {}
        }
    )
)
```

