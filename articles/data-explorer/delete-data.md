---
title: Delete data from Azure Data Explorer
description: This article describes bulk delete scenarios in Azure Data Explore, including purge and retention based deletes.
author: orspod
ms.author: v-orspod
ms.reviewer: mblythe
ms.service: data-explorer
services: data-explorer
ms.topic: conceptual
ms.date: 09/24/2018
---

# Delete data from Azure Data Explorer

Azure Data Explorer supports several bulk delete approaches, which we cover in this article. It doesn't support per-record deletion in real time, because it's optimized for fast read access.

* If the database is no longer needed, delete it using the drop database command.

    ```Kusto
    .drop database <DatabaseName>
    ```

* If one or more tables is no longer needed, delete them using the drop table or drop tables command.

    ```Kusto
    .drop table <TableName>

    .drop tables (<TableName1>, <TableName2>,...)
    ```

* If old data is no longer needed, delete it by changing the retention period at the database or table level.

    Consider a database or table that is set for 90 days of retention. Business needs change, so now only 60 days of data is needed. In this case, delete the older data in one of the following ways.

    ```Kusto
    .alter-merge database <DatabaseName> policy retention softdelete = 60d

    .alter-merge table <TableName> policy retention softdelete = 60d
    ```

    For more information, see [Retention policy](https://docs.microsoft.com/azure/kusto/concepts/retentionpolicy).

* You can delete individual records using the *purge* operation, based on a predicate like `where CustomerName == 'contoso'`. That said, a purge is a bulk delete that isn't designed for real-time deletion. The following example shows a purge.

    ```Kusto
    .purge table Customer records
    | where CustomerName =='contoso'
    ```

If you need assistance with data deletion issues, please open a support request in the [Azure portal](https://portal.azure.com).