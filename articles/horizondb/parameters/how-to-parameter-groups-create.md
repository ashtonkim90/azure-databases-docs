---
title: Create Parameter Groups in Azure HorizonDB
description: This article describes how to create parameter groups in Azure HorizonDB.
#customer intent: As a user, I want to create a parameter group in Azure HorizonDB, so that I can customize PostgreSQL configuration settings for my clusters.
author: nachoalonsoportillo
ms.author: ialonso
ms.reviewer: maghan
ms.date: 07/14/2026
ms.service: azure-horizondb
ms.subservice: parameters-group
ms.topic: how-to
---

# Create parameter groups in Azure HorizonDB (Preview)

When you create a parameter group, you must provide at least one parameter. The underlying operation in the backend merges your input with the system defaults for the specified `pgVersion`.

## Steps to create parameter groups

### [Portal](#tab/portal-create-parameter-groups)

Use the [Azure portal](https://portal.azure.com):

1. Browse the [**Azure HorizonDB (Preview) parameter groups**](https://ms.portal.azure.com/#browse/Microsoft.HorizonDB%2F2FparameterGroups).

1. In the command bar, select **Create**.

    :::image type="content" source="./media/how-to-create-parameter-groups/browse-parameter-groups.png" alt-text="Screenshot that shows the browse for Azure HorizonDB (Preview) parameter groups page." lightbox="./media/how-to-create-parameter-groups/browse-parameter-groups.png":::

1. In **Create a parameter group**, select the subscription and resource group where you want to create the parameter group.

    :::image type="content" source="./media/how-to-create-parameter-groups/select-subscription-resource-group.png" alt-text="Screenshot that shows the Create a parameter group page and a subscription and resource group selected." lightbox="./media/how-to-create-parameter-groups/select-subscription-resource-group.png":::

1. Enter a name that's unique among all parameter groups in the resource group and subscription. Embed some form of encoded description in the name so that you can later identify the potential target clusters of that configuration.

    :::image type="content" source="./media/how-to-create-parameter-groups/parameter-group-name.png" alt-text="Screenshot that shows the Create a parameter group page and a parameter group name provided." lightbox="./media/how-to-create-parameter-groups/parameter-group-name.png":::

1. Select a location for the parameter group. You can only connect parameter groups created in a location to clusters that also exist in that same location.

    :::image type="content" source="./media/how-to-create-parameter-groups/location.png" alt-text="Screenshot that shows the Create a parameter group page and a location selected." lightbox="./media/how-to-create-parameter-groups/location.png":::

1. Although not required, provide a description explaining in more detail the purpose of the parameter group configuration. It can also describe the ideal target clusters for which it was conceived.

    :::image type="content" source="./media/how-to-create-parameter-groups/description.png" alt-text="Screenshot that shows the Create a parameter group page and a description provided." lightbox="./media/how-to-create-parameter-groups/description.png":::

1. Select the version of PostgreSQL for which the parameter group is supported. Parameter groups created for a given version of PostgreSQL can't be applied to clusters of a different version of PostgreSQL.

    :::image type="content" source="./media/how-to-create-parameter-groups/postgresql-version.png" alt-text="Screenshot that shows the Create a parameter group page and a PostgreSQL version selected." lightbox="./media/how-to-create-parameter-groups/postgresql-version.png":::

1. Select **Next** to configure the values of the modifiable parameters whose defaults you want to change in this parameter group.

    :::image type="content" source="./media/how-to-create-parameter-groups/configure-parameters.png" alt-text="Screenshot that shows the Create a parameter group page and the first page of parameters available in the default parameter group selected." lightbox="./media/how-to-create-parameter-groups/configure-parameters.png":::

1. Search for the names of the parameters whose default values you want to override, and change their defaults. When finished, select **Create**.

    :::image type="content" source="./media/how-to-create-parameter-groups/change-max-connections-parameter.png" alt-text="Screenshot that shows the Create a parameter group page and the max_connections parameter default value changed." lightbox="./media/how-to-create-parameter-groups/change-max-connections-parameter.png":::

1. A new deployment is initialized to create the parameter group.

    :::image type="content" source="./media/how-to-create-parameter-groups/initializing-deployment.png" alt-text="Screenshot that shows the Create a parameter group page and the initializing deployment notification." lightbox="./media/how-to-create-parameter-groups/initializing-deployment.png":::

1. Wait until the deployment completes.

    :::image type="content" source="./media/how-to-create-parameter-groups/deployment-progress.png" alt-text="Screenshot that shows the Deployment is in progress page." lightbox="./media/how-to-create-parameter-groups/deployment-progress.png":::

1. When the deployment completes, select **Go to resource** to visit the newly created parameter group.

    :::image type="content" source="./media/how-to-create-parameter-groups/deployment-completed.png" alt-text="Screenshot that shows the Deployment is completed page." lightbox="./media/how-to-create-parameter-groups/deployment-completed.png":::

1. You can now inspect all details associated with the newly created parameter group.

    :::image type="content" source="./media/how-to-create-parameter-groups/parameter-group-created.png" alt-text="Screenshot that shows the Overview page of the newly created parameter group." lightbox="./media/how-to-create-parameter-groups/parameter-group-created.png":::

### [CLI](#tab/cli-create-parameter-groups)

To create a parameter group and apply changes immediately, use the [az horizondb parameter-group create](/cli/azure/horizondb/parameter-group?view=azure-cli-latest#az-horizondb-parameter-group-create) command.

```azurecli-interactive
az horizondb parameter-group create \
  --location <location>
  --resource-group <resource_group>
  --name <parameter_group>
  --version <version>
  --parameters <parameter_name_1=parameter_value_1 parameter_name_2=parameter_value_2... parameter_name_n=parameter_value_n>
  --apply-immediately true
  --description <description>
```

To create a parameter group and don't apply changes immediately, use the [az horizondb parameter-group create](/cli/azure/horizondb/parameter-group?view=azure-cli-latest#az-horizondb-parameter-group-create) command.

```azurecli-interactive
az horizondb parameter-group create \
  --location <location>
  --resource-group <resource_group>
  --name <parameter_group>
  --version <version>
  --parameters <parameter_name_1=parameter_value_1 parameter_name_2=parameter_value_2... parameter_name_n=parameter_value_n>
  --description <description>
```

#### Possible errors

| Error code | Description |
| --- | --- |
| `ParameterGroupNameConflictsWithDefault` | When the name of the parameter group matches any of the names reserved for default parameter groups. |
| `ParameterGroupAlreadyExists` | When a parameter group with the same resource identifier already exists. |
| `ParameterGroupPgVersionRequired` | When `pgVersion` isn't included as one of the properties in the input. |
| `ParameterNotRecognized` | When one or more parameter names in the input aren't recognized among the ones supported for the version of PostgreSQL for which the parameter group is defined. |
| `ParameterIsReadOnly` | When one or more parameters in the input are read-only parameters. |
| `ParameterValueInvalid` | When the value assigned to one or more parameters in the input isn't valid according to the data type and allowed values of that parameter. |
| `ParameterGroupParametersRequired` | When the input doesn't include any parameter. |

---

## Related content

- [Parameter groups in Azure HorizonDB (Preview)](concepts-parameter-groups.md)
- [Update parameter groups](how-to-parameter-groups-update.md)
- [Delete parameter groups](how-to-parameter-groups-delete.md)
- [List parameter groups](how-to-parameter-groups-list.md)
- [Connect clusters to parameter groups](how-to-parameter-groups-connect.md)
- [List clusters connected to parameter groups](how-to-parameter-groups-list-connected.md)
- [Identity parameter group connected to a cluster](how-to-parameter-groups-identify-connected-cluster.md)
