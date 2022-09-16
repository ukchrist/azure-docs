---
title: Copy and transform data from an SAP ODP source with the SAP CDC connector in Azure Data Factory or Azure Synapse Analytics
titleSuffix: Azure Data Factory & Azure Synapse
description: Learn how to copy and transform data from an SAP ODP source to supported sink data stores by using a copy activity and mapping data flows in an Azure Data Factory or Azure Synapse Analytics pipeline.
author: ukchrist
ms.author: ulrichchrist
ms.service: data-factory
ms.subservice: data-movement
ms.topic: conceptual
ms.custom: synapse
ms.date: 09/05/2022
---

# Copy and transform data from an SAP ODP source using Azure Data Factory or Azure Synapse Analytics

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

This article outlines how to use the copy activity in Azure Data Factory and Azure Synapse Analytics pipelines to copy data from an SAP table. For more information, see [Copy activity overview](copy-activity-overview.md).

This article outlines how to use Copy Activity to copy data, and use Data Flow to transform data from an SAP ODP source using the SAP CDC connector. To learn more, read the introductory article for [Azure Data Factory](introduction.md) or [Azure Synapse Analytics](../synapse-analytics/overview-what-is.md). For an introduction to copying and transforming data with Azure Data Factory and Azure Synapse analytics, read [Copy activity overview](copy-activity-overview.md) and [mapping data flow](concepts-data-flow-overview.md).

>[!TIP]
>To learn the overall support on SAP data integration scenario, see [SAP data integration using Azure Data Factory whitepaper](https://github.com/Azure/Azure-DataFactory/blob/master/whitepaper/SAP%20Data%20Integration%20using%20Azure%20Data%20Factory.pdf) with detailed introduction on each SAP connector, comparsion and guidance.

## Supported capabilities

This SAP CDC connector is supported for the following capabilities:

| Supported capabilities|IR |
|---------| --------|
|[Copy activity](copy-activity-overview.md) (source/-)|&#9313;|
|[Mapping data flow](concepts-data-flow-overview.md) (source/-)|&#9313;|
|[Lookup activity](control-flow-lookup-activity.md)|&#9313;|

<small>*&#9312; Azure integration runtime &#9313; Self-hosted integration runtime*</small>

For a list of the data stores that are supported as sources or sinks by the copy activity, see the [Supported data stores](copy-activity-overview.md#supported-data-stores-and-formats) table.

This SAP CDC connector leverages the SAP ODP framework to extract data from SAP source systems. For an introduction to the architecture of the solution, read [Introduction and architecture to SAP change data capture (CDC)](sap-change-data-capture-introduction-architecture.md) in our [SAP knowledge center](industry-sap-overview.md).

The SAP ODP framework is contained in most SAP NetWeaver based systems, including SAP ECC, SAP S/4HANA, SAP BW, SAP BW/4HANA, SAP LT Replication Server (SLT), except very old ones. For prerequisites and minimum required releases, see [Prerequisites and configuration](sap-change-data-capture-prerequisites-configuration.md#sap-system-requirements).  

The SAP CDC connector supports basic authentication or Secure Network Communications (SNC), if SNC is configured.

## Prerequisites

To use this SAP CDC connector, you need to:

- Set up a self-hosted integration runtime (version 3.17 or later). For more information, see [Create and configure a self-hosted integration runtime](create-self-hosted-integration-runtime.md).

- Download the 64-bit [SAP Connector for Microsoft .NET 3.0](https://support.sap.com/en/product/connectors/msnet.html) from SAP's website, and install it on the self-hosted integration runtime machine. During installation, make sure you select the **Install Assemblies to GAC** option in the **Optional setup steps** window.

  :::image type="content" source="./media/connector-sap-business-warehouse-open-hub/install-sap-dotnet-connector.png" alt-text="Install SAP Connector for .NET":::

- The SAP user who's being used in the SAP table connector must have the permissions described in [](sap-change-data-capture-prerequisites-configuration.md#sap-user-configurations):


## Get started

[!INCLUDE [data-factory-v2-connector-get-started](includes/data-factory-v2-connector-get-started.md)]

## Create a linked service for the SAP CDC connector using UI

Follow the steps described in [Prepare the SAP ODP linked service](sap-change-data-capture-prepare-linked-service-source-dataset.md#prepare-the-sap-odp-linked-service) to create a linked service for the SAP CDC connector in the Azure portal UI.

## Dataset properties

To prepare an SAP CDC dataset, follow [Prepare the SAP ODP source dataset](sap-change-data-capture-prepare-linked-service-source-dataset.md#prepare-the-sap-odp-source-dataset)

## Copy or transform data with the SAP CDC connector

SAP CDC datasets can be used as source in Copy activity or Mapping data flow. While Copy activity will only allow you to land the raw change data feed coming from the ODP source, integration into Mapping data flow takes care of interpreting the change data feed correctly by evaluating technical attributes provided by the ODP framework (e.g., ODQ_CHANGEMODE) automatically. This allows users to concentrate on the required transformation logic without having to bother with the internals of the SAP ODP change feed.  

### Copy activity properties

To configure a copy activity for an SAP CDC source dataset, follow [Create a copy activity with an SAP CDC data source](sap-change-data-capture-copy-activity.md#create-a-copy-activity-with-an-sap-cdc-data-source).

### Mapping data flow properties

To create a mapping data flow using the SAP CDC connector as a source, complete the following steps:

1.	In ADF Studio, go to the **Data flows** section of the **Author** hub, select the **…** button to drop down the **Data flow actions** menu, and select the **New data flow** item. Turn on debug mode by using the **Data flow debug** button in the top bar of data flow canvas.

    :::image type="content" source="media/sap-change-data-capture-solution/sap-cdc-mdf-data-flow-debug.png" alt-text="Screenshot of the data flow debug button in mapping data flow.":::

1. In the mapping data flow editor, select **Add Source**.

    :::image type="content" source="media/sap-change-data-capture-solution/sap-cdc-mdf-add-source.png" alt-text="Screenshot of add source in mapping data flow.":::

1. On the tab **Source settings** select a prepared SAP CDC dataset or select the **New** button to create a new one. Alternatively, you can also select **Inline** in the **Source type** property and continue without defining an explicit dataset.

    :::image type="content" source="media/sap-change-data-capture-solution/sap-cdc-mdf-select-dataset.png" alt-text="Screenshot of the select dataset option in source settings of mapping data flow source.":::

1. On the tab **Source options** select the option **Full on every run** if you want to load full snapshots on every execution of your mapping data flow, or **Full on the first run, then incremental** if you want to subscribe to a change feed from the SAP source system. In this case, the first run of your pipeline will do a delta initialization, which means it will return a current full data snapshot and create an ODP delta subscription in the source system so that with subsequent runs, the SAP source system will return incremental changes since the previous run only. In case of incremental loads it is required to specify the keys of the ODP source object in the **Key columns** property.

    :::image type="content" source="media/sap-change-data-capture-solution/sap-cdc-mdf-run-mode.png" alt-text="Screenshot of the run mode property in source options of mapping data flow source.":::

    :::image type="content" source="media/sap-change-data-capture-solution/sap-cdc-mdf-key-columns.png" alt-text="Screenshot of the key columns selection in source options of mapping data flow source.":::

1. For details on the tabs **Projection**, **Optimize** and **Inspect**, please follow [mapping data flow](concepts-data-flow-overview.md).
