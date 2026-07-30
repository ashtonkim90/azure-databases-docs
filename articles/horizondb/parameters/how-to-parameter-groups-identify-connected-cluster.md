---
title: Identify which Parameter Group is Connected to a Cluster in Azure HorizonDB
description: This article describes how to identify which parameter group is connected to a cluster in Azure HorizonDB.
#customer intent: As a user, I want to identify which parameter group is connected to a cluster, so that I can verify its configuration in Azure HorizonDB.
author: nachoalonsoportillo
ms.author: ialonso
ms.reviewer: maghan
ms.date: 07/14/2026
ms.service: azure-horizondb
ms.subservice: parameters-group
ms.topic: how-to
---

# Identify which parameter group is connected to a cluster in Azure HorizonDB (Preview)

You can identify which parameter group is connected to a cluster.

## Steps to identify which parameter group connected to a cluster

### [Portal](#tab/portal-identify-parameter-group-connected-cluster)

Use the [Azure portal](https://portal.azure.com):

1. Browse the [**Azure HorizonDB (Preview)**](https://ms.portal.azure.com/#browse/Microsoft.HorizonDB%2Fclusters).

1. By using the filtering buttons and the search box, find the cluster for which you want to check what parameter group it's connected to, and select it.

    :::image type="content" source="./media/how-to-identify-connected-cluster/filter-search-parameter-groups.png" alt-text="Screenshot that shows the browse for Azure HorizonDB (Preview) parameter groups page filtered by the name of the parameter group that you want to connect to one or more clusters." lightbox="./media/how-to-identify-connected-cluster/filter-search-parameter-groups.png":::

1. In the resource menu, under **Settings**, select **Parameters**. The name of the parameter group to which the cluster is connected appears to the side of the **Parameter group (create):** label. You can also scroll or search for parameter names to check how each parameter is configured in that parameter group.

    :::image type="content" source="./media/how-to-identify-connected-cluster/parameters.png" alt-text="Screenshot that shows the Parameters page of the selected cluster, from where you can check which parameter group the cluster is connected to." lightbox="./media/how-to-identify-connected-cluster/parameters.png":::

    > [!NOTE]
    > If you assign the default parameter group to a cluster, you don't see any parameters listed. This problem is known and will be fixed.

1. If you select the name, you're taken to the **Overview** page of the parameter group resource.

    :::image type="content" source="./media/how-to-identify-connected-cluster/parameter-group-overview.png" alt-text="Screenshot that shows the Overview page of the parameter group selected." lightbox="./media/how-to-identify-connected-cluster/parameter-group-overview.png":::

### [CLI](#tab/cli-identify-parameter-group-connected-cluster)

Use the [az horizondb show](/cli/azure/horizondb?view=azure-cli-latest#az-horizondb-show) command to identify which parameter group is connected to a cluster.

```azurecli-interactive
az horizondb show \
  --resource-group <resource_group>
  --name <parameter_group>
  --query properties.parameterGroup.id \
  --output tsv
```

Use the [az horizondb parameter-group show]() command to get the list of parameters in the parameter group that a cluster connects to.

```azurecli-interactive
az horizondb parameter-group show \
  --id $(az horizondb show --resource-group <resource_group> --name <cluster> --query properties.parameterGroup.id --output tsv)
  --query properties.parameters
```

> [!NOTE]
> If you assign the default parameter group to a cluster, you don't see any parameters listed. This problem is known and will be fixed.
> In this case, you receive the following error: `Not Found({"error":{"code":"ResourceNotFound","message":"The Resource 'Microsoft.HorizonDb/parameterGroups/default_pg17' under resource group '{resourceGroupName}' was not found. For more details please go to https://aka.ms/ARMResourceNotFoundFix"}})`

---

## Related content

- [Parameter groups in Azure HorizonDB (Preview)](concepts-parameter-groups.md)
- [Create parameter groups](how-to-parameter-groups-create.md)
- [Update parameter groups](how-to-parameter-groups-update.md)
- [Delete parameter groups](how-to-parameter-groups-delete.md)
- [List parameter groups](how-to-parameter-groups-list.md)
- [Connect clusters to parameter groups](how-to-parameter-groups-connect.md)
- [List clusters connected to parameter groups](how-to-parameter-groups-list-connected.md)
