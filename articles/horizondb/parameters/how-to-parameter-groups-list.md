---
title: List Parameter Groups in Azure HorizonDB
description: This article describes how to list parameter groups in Azure HorizonDB.
#customer intent: As an user, I want to list all parameter groups in my subscription, so that I can review my current configuration inventory.
author: nachoalonsoportillo
ms.author: ialonso
ms.reviewer: maghan
ms.date: 07/14/2026
ms.service: azure-horizondb
ms.subservice: parameters-group
ms.topic: how-to
---

# List parameter groups in Azure HorizonDB (Preview)

When you list parameter groups, you can scope the operation to a single parameter group, to all parameter groups in a resource group, or to all parameter groups in any resource group of a given subscription.

## Steps to list parameter groups

### [Portal](#tab/portal-list-parameter-groups)

Use the [Azure portal](https://portal.azure.com):

1. Browse the [**Azure HorizonDB (Preview) parameter groups**](https://ms.portal.azure.com/#browse/Microsoft.HorizonDB%2F2FparameterGroups).

1. Use the filtering buttons and the search box to find the parameter groups that you want.

    :::image type="content" source="./media/how-to-list-parameter-groups/filter-search-parameter-groups.png" alt-text="Screenshot that shows the browse for Azure HorizonDB (Preview) parameter groups page filtered by the name of the parameter group which you want to delete." lightbox="./media/how-to-list-parameter-groups/filter-search-parameter-groups.png":::


### [CLI](#tab/cli-list-parameter-groups)

Use the [az horizondb parameter-group list](/cli/azure/horizondb/parameter-group?view=azure-cli-latest#az-horizondb-parameter-group-list) command to list the parameter groups in current subscription:

```azurecli-interactive
az horizondb parameter-group list
```

Use the [az horizondb parameter-group list](/cli/azure/horizondb/parameter-group?view=azure-cli-latest#az-horizondb-parameter-group-list) command with the `--resource-group` parameter to list the parameter groups in a resource group of currrent subscription:

```azurecli-interactive
az horizondb parameter-group list \
  --resource-group <resource_group>
```

Use the [az horizondb parameter-group show](/cli/azure/horizondb/parameter-group?view=azure-cli-latest#az-horizondb-parameter-group-show) command to show a specific parameter group:

```azurecli-interactive
az horizondb parameter-group list \
  --resource-group <resource_group>
  --name <parameter_group>
```

---

## Related content

- [Parameter groups in Azure HorizonDB (Preview)](concepts-parameter-groups.md)
- [Create parameter groups](how-to-parameter-groups-create.md)
- [Update parameter groups](how-to-parameter-groups-update.md)
- [Delete parameter groups](how-to-parameter-groups-delete.md)
- [Connect clusters to parameter groups](how-to-parameter-groups-connect.md)
- [List clusters connected to parameter groups](how-to-parameter-groups-list-connected.md)
- [Identity parameter group connected to a cluster](how-to-parameter-groups-identify-connected-cluster.md)
