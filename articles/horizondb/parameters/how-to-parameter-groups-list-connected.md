---
title: List Clusters Connected to Parameter Groups in Azure HorizonDB
description: This article describes how to list clusters connected to parameter groups in Azure HorizonDB.
#customer intent: As a user, I want to list the clusters connected to a parameter group, so that I can see which clusters are affected by that group's settings.
author: nachoalonsoportillo
ms.author: ialonso
ms.reviewer: maghan
ms.date: 07/14/2026
ms.service: azure-horizondb
ms.subservice: parameters-group
ms.topic: how-to
---

# List clusters connected to parameter groups in Azure HorizonDB (Preview)

Given a parameter group, you can list the clusters that are connected to it and see the current synchronization status for each cluster.

## Steps to list clusters connected to parameter groups

### [Portal](#tab/portal-clusters-connected-parameter-group)

Use the [Azure portal](https://portal.azure.com):

1. Browse the [**Azure HorizonDB (Preview) parameter groups**](https://ms.portal.azure.com/#browse/Microsoft.HorizonDB%2F2FparameterGroups).

1. Use the filtering buttons and the search box to find the parameter group that you want to check. Select the parameter group.

    :::image type="content" source="./media/how-to-list-connected-clusters-parameter-groups/filter-search-parameter-groups.png" alt-text="Screenshot that shows the browse for Azure HorizonDB (Preview) parameter groups page. Filter by the name of the parameter group that you want to check what clusters are connected to it." lightbox="./media/how-to-list-connected-clusters-parameter-groups/filter-search-parameter-groups.png":::

1. In the **Connected clusters** section, find the list of clusters that are connected to the parameter group.

    :::image type="content" source="./media/how-to-list-connected-clusters-parameter-groups/connected-clusters.png" alt-text="Screenshot that shows the Overview page of the selected parameter group with the clusters that are connected to it." lightbox="./media/how-to-list-connected-clusters-parameter-groups/connected-clusters.png":::

### [CLI](#tab/cli-clusters-connected-parameter-group)

Use the [az horizondb parameter-group list-connections](/cli/azure/horizondb/parameter-group?view=azure-cli-latest#az-horizondb-parameter-group-list-connections) command to list the clusters connected to a parameter group:

```azurecli-interactive
az horizondb parameter-group list-connections \
  --resource-group <resource_group>
  --name <parameter_group>
```

---

## Related content

- [Parameter groups in Azure HorizonDB (Preview)](concepts-parameter-groups.md)
- [Create parameter groups](how-to-parameter-groups-create.md)
- [Update parameter groups](how-to-parameter-groups-update.md)
- [Delete parameter groups](how-to-parameter-groups-delete.md)
- [List parameter groups](how-to-parameter-groups-list.md)
- [Connect clusters to parameter groups](how-to-parameter-groups-connect.md)
- [Identity parameter group connected to a cluster](how-to-parameter-groups-identify-connected-cluster.md)
