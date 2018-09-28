---
title: Copy data to/from Azure Cosmos DB using Data Factory | Microsoft Docs
description: Learn how to copy data from supported source data stores to Azure Cosmos DB (or) from Cosmos DB to supported sink stores using Data Factory.
services: data-factory, cosmosdb
documentationcenter: ''
author: linda33wj
manager: craigg
ms.reviewer: douglasl

ms.service: multiple
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/11/2018
ms.author: jingwang

---
# Copy data to or from Azure Cosmos DB using Azure Data Factory

> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [Version 1](v1/data-factory-azure-documentdb-connector.md)
> * [Current version](connector-azure-cosmos-db.md)

This article outlines how to use the Copy Activity in Azure Data Factory to copy data from and to Azure Cosmos DB (SQL API). It builds on the [copy activity overview](copy-activity-overview.md) article that presents a general overview of copy activity.

## Supported capabilities

You can copy data from Azure Cosmos DB to any supported sink data store, or copy data from any supported source data store to Azure Cosmos DB. For a list of data stores supported as sources/sinks by the copy activity, see the [Supported data stores](copy-activity-overview.md#supported-data-stores-and-formats) table.

Specifically, this Azure Cosmos DB connector supports:

- Copying data from/to Cosmos DB [SQL API](https://docs.microsoft.com/azure/cosmos-db/documentdb-introduction).
- Writing into Cosmos DB as INSERT or UPSERT.
- Importing/exporting JSON documents as-is, or copying data from/to tabular dataset e.g. SQL database, CSV files, etc. To copy documents as-is to/from JSON files or another Cosmos DB collection, see [Import/Export JSON documents](#importexport-json-documents).

Data Factory integrates with [Cosmos DB bulk executor library](https://github.com/Azure/azure-cosmosdb-bulkexecutor-dotnet-getting-started) to provide the best performance writing into Cosmos DB.

>[!TIP]
>Watch [this video](https://youtu.be/5-SRNiC_qOU) which walks through copying data from Azure Blob storage to Cosmos DB and describes performance tuning considerations for ingesting data into Cosmos DB in general.

## Getting started

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

The following sections provide details about properties that are used to define Data Factory entities specific to Azure Cosmos DB.

## Linked service properties

The following properties are supported for Azure Cosmos DB linked service:

| Property | Description | Required |
|:--- |:--- |:--- |
| type | The type property must be set to: **CosmosDb**. | Yes |
| connectionString |Specify information needed to connect to Azure Cosmos DB database. Note you need to specify database info in the connection string as below sample. Mark this field as a SecureString to store it securely in Data Factory, or [reference a secret stored in Azure Key Vault](store-credentials-in-key-vault.md). |Yes |
| connectVia | The [Integration Runtime](concepts-integration-runtime.md) to be used to connect to the data store. You can use Azure Integration Runtime or Self-hosted Integration Runtime (if your data store is located in private network). If not specified, it uses the default Azure Integration Runtime. |No |

**Example:**

```json
{
    "name": "CosmosDbLinkedService",
    "properties": {
        "type": "CosmosDb",
        "typeProperties": {
            "connectionString": {
                "type": "SecureString",
                "value": "AccountEndpoint=<EndpointUrl>;AccountKey=<AccessKey>;Database=<Database>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## Dataset properties

For a full list of sections and properties available for defining datasets, see the datasets article. This section provides a list of properties supported by Azure Cosmos DB dataset.

To copy data from/to Azure Cosmos DB, set the type property of the dataset to **DocumentDbCollection**. The following properties are supported:

| Property | Description | Required |
|:--- |:--- |:--- |
| type | The type property of the dataset must be set to: **DocumentDbCollection** |Yes |
| collectionName |Name of the Cosmos DB document collection. |Yes |

**Example:**

```json
{
    "name": "CosmosDbDataset",
    "properties": {
        "type": "DocumentDbCollection",
        "linkedServiceName":{
            "referenceName": "<Azure Cosmos DB linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "collectionName": "<collection name>"
        }
    }
}
```

### Schema by Data Factory

For schema-free data stores such as Azure Cosmos DB, copy activity infers the schema in one of the following ways. Therefore, unless you want to [import/export JSON documents as-is](#importexport-json-documents), the best practice is to specify the structure of data in the **structure** section.

*. If you specify the structure of data by using the **structure** property in the dataset definition, the Data Factory service honors this structure as the schema. In this case, if a row does not contain a value for a column, a null value will be provided for it.
*. If you do not specify the structure of data by using the **structure** property in the dataset definition, the Data Factory service infers the schema by using the first row in the data. In this case, if the first row does not contain the full schema, some columns will be missing in the result of copy operation.

## Copy activity properties

For a full list of sections and properties available for defining activities, see the [Pipelines](concepts-pipelines-activities.md) article. This section provides a list of properties supported by Azure Cosmos DB source and sink.

### Azure Cosmos DB as source

To copy data from Azure Cosmos DB, set the source type in the copy activity to **DocumentDbCollectionSource**. The following properties are supported in the copy activity **source** section:

| Property | Description | Required |
|:--- |:--- |:--- |
| type | The type property of the copy activity source must be set to: **DocumentDbCollectionSource** |Yes |
| query |Specify the Cosmos DB query to read data.<br/><br/>Example: `SELECT c.BusinessEntityID, c.Name.First AS FirstName, c.Name.Middle AS MiddleName, c.Name.Last AS LastName, c.Suffix, c.EmailPromotion FROM c WHERE c.ModifiedDate > \"2009-01-01T00:00:00\"` |No <br/><br/>If not specified, the SQL statement that is executed: `select <columns defined in structure> from mycollection` |
| nestingSeparator |Special character to indicate that the document is nested and how to flattern the result set.<br/><br/>For example, if a Cosmos DB query returns a nested result `"Name": {"First": "John"}`, copy activity will identify column name as "Name.First" with value "John" when the nestedSeparator is dot. |No (default is dot `.`) |

**Example:**

```json
"activities":[
    {
        "name": "CopyFromCosmosDB",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Document DB input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "DocumentDbCollectionSource",
                "query": "SELECT c.BusinessEntityID, c.Name.First AS FirstName, c.Name.Middle AS MiddleName, c.Name.Last AS LastName, c.Suffix, c.EmailPromotion FROM c WHERE c.ModifiedDate > \"2009-01-01T00:00:00\""
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### Azure Cosmos DB as sink

To copy data to Azure Cosmos DB, set the sink type in the copy activity to **DocumentDbCollectionSink**. The following properties are supported in the copy activity **source** section:

| Property | Description | Required |
|:--- |:--- |:--- |
| type | The type property of the copy activity sink must be set to: **DocumentDbCollectionSink** |Yes |
| writeBehavior |Describe how to write data into Cosmos DB. Allowed values are: `insert` and `upsert`.<br/>The behavior of **upsert** is to replace the document if an document of the same id already exist; otherwise insert it. Note ADF will automatically generate an id for the document if it is not specified either in the original doc or by column mapping), which means you need to make sure your document has an "id" so that upsert work as expected. |No, default is insert |
| writeBatchSize | Data Factory use [Cosmos DB bulk executor library](https://github.com/Azure/azure-cosmosdb-bulkexecutor-dotnet-getting-started) to write data into Cosmos DB. "writeBatchSize" controls the size of documents we provide to the library each time. You can try increase writeBatchSize to improve performance. |No, default is 10,000 |
| nestingSeparator |A special character in the source column name to indicate that nested document is needed. <br/><br/>For example, `Name.First` in the output dataset structure generates the following JSON structure in the Cosmos DB document:`"Name": {"First": "[value maps to this column from source]"}` when the nestedSeparator is dot. |No (default is dot `.`) |

**Example:**

```json
"activities":[
    {
        "name": "CopyToCosmosDB",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Document DB output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "DocumentDbCollectionSink",
                "writeBehavior": "upsert"
            }
        }
    }
]
```

## Import/Export JSON documents

Using this Cosmos DB connector, you can easily

* Import JSON documents from various sources into Cosmos DB, including Azure Blob, Azure Data Lake Store, and other file-based stores supported by Azure Data Factory.
* Export JSON documents from Cosmos DB collecton into various file-based stores.
* Copy documents between two Cosmos DB collections as-is.

To achieve such schema-agnostic copy:

* When using copy data tool, check the **"Export as-is to JSON files or Cosmos DB collection"** option.
* When using activity authoring, do not specify the "structure" (aka schema) section in Cosmos DB dataset(s) nor "nestingSeparator" property on Cosmos DB source/sink in copy activity. When importing from/exporting to JSON files, in the corresponding file store dataset, specify format type as "JsonFormat" and config "filePattern" properly (see [JSON format](supported-file-formats-and-compression-codecs.md#json-format) section for details), then do not specify the "structure" (aka schema) section and skip the rest format settings.

## Next steps
For a list of data stores supported as sources and sinks by the copy activity in Azure Data Factory, see [supported data stores](copy-activity-overview.md##supported-data-stores-and-formats).
