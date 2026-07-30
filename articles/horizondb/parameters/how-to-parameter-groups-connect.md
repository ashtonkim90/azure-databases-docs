---
title: Connect Clusters to Parameter Groups in Azure HorizonDB
description: This article describes how to connect parameter groups to clusters in Azure HorizonDB.
#customer intent: As a user, I want to connect a parameter group to one or more clusters, so that I can apply consistent configuration settings across my environment.
author: nachoalonsoportillo
ms.author: ialonso
ms.reviewer: maghan
ms.date: 07/14/2026
ms.service: azure-horizondb
ms.subservice: parameters-group
ms.topic: how-to
---

# Connect clusters to parameter groups in Azure HorizonDB (Preview)

You can connect one parameter group to one or more clusters, as long as the region of the parameter group and the region of the cluster match.

## Steps to connect parameter groups to clusters

### [Portal](#tab/portal-connect-parameter-groups)

Use the [Azure portal](https://portal.azure.com):

1. Browse the [**Azure HorizonDB (Preview) parameter groups**](https://ms.portal.azure.com/#browse/Microsoft.HorizonDB%2F2FparameterGroups).

1. Use the filtering buttons and the search box to find the parameter group for which you want to check what clusters are connected to it. Select the parameter group.

    :::image type="content" source="./media/how-to-connect-clusters-parameter-groups/filter-search-parameter-groups.png" alt-text="Screenshot that shows the browse for Azure HorizonDB (Preview) parameter groups page filtered by the name of the parameter group for which you want to connect to one or more clusters." lightbox="./media/how-to-connect-clusters-parameter-groups/filter-search-parameter-groups.png":::

1. In the **Connected clusters** section, select the **Connect clusters** command bar button.

    :::image type="content" source="./media/how-to-connect-clusters-parameter-groups/connect-clusters-first.png" alt-text="Screenshot that shows the Overview page of the selected parameter group from where you can connect it to one or more clusters." lightbox="./media/how-to-connect-clusters-parameter-groups/connect-clusters-first.png":::

1. In the **Connect clusters** page that opens on the side, use the filtering button and the search box to find the clusters that you want to connect to this parameter group. Select the checkbox of each cluster that you want to connect. Then, select **Connect clusters**.

    :::image type="content" source="./media/how-to-connect-clusters-parameter-groups/connect-clusters-second.png" alt-text="Screenshot that shows the Connect clusters page of the selected parameter group from where you can connect it to one or more clusters." lightbox="./media/how-to-connect-clusters-parameter-groups/connect-clusters-second.png":::

1. A notification indicates that the operation to connect the parameter group to the clusters you selected is initiated.

    :::image type="content" source="./media/how-to-connect-clusters-parameter-groups/notification-connecting.png" alt-text="Screenshot that shows the notification that indicates the connection of the parameter group to the selected clusters is initiated." lightbox="./media/how-to-connect-clusters-parameter-groups/notification-connecting.png":::

1. A few seconds later, a notification indicates that the operation completed successfully.

    :::image type="content" source="./media/how-to-connect-clusters-parameter-groups/notification-connected.png" alt-text="Screenshot that shows the notification that indicates the connection of the parameter group to the selected clusters completed successfully." lightbox="./media/how-to-connect-clusters-parameter-groups/notification-connected.png":::


### [CLI](#tab/cli-connect-parameter-groups)

Use the [az horizondb update](/cli/azure/horizondb?view=azure-cli-latest#az-horizondb-update) command to connect one specific parameter group to a cluster:

```azurecli-interactive
az horizondb update \
  --resource-group <resource_group>
  --name <cluster>
  --parameter-group <parameter_group>
```

#### Possible errors

| Error code | Description |
| --- | --- |
| `ParameterGroupApplyFailed` | Raised when the attempt to apply the parameter group to the cluster fails. |

---

## Related content

- [Parameter groups in Azure HorizonDB (Preview)](concepts-parameter-groups.md)
- [Create parameter groups](how-to-parameter-groups-create.md)
- [Update parameter groups](how-to-parameter-groups-update.md)
- [Delete parameter groups](how-to-parameter-groups-delete.md)
- [List parameter groups](how-to-parameter-groups-list.md)
- [List clusters connected to parameter groups](how-to-parameter-groups-list-connected.md)
- [Identity parameter group connected to a cluster](how-to-parameter-groups-identify-connected-cluster.md)
