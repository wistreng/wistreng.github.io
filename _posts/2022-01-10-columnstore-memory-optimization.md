---
layout: post
title: Synapse SQL Optimization II
date:   2022-01-10 10:01:27 +1300
tags: [synapse, cloud, azure]
category: Cloud Azure
---


Push the limits and try to loading a fact table from the finest granular level of data you could ever get, the size of the table is morn than 5 billions a year.

To do that and to build a 'self-service' report on top of it to interact with, here the check list you need to take into account:

1. Use CCI (cluster columnstore index) for fact table, this means better query performance
2. Use Hash-distributed fact table and round-robin dimension table (in 99% cases)
3. Use fewer columns, means only keep those necessary columns for fact tables
4. Use fewer string column in fact table, that means use numeric type rather than string types if you could, because string type require more memory than the numeric or date data type, that also means use smallest possible column which support your data so will improve performance. You could put all descriptive columns into dimension tables.
5. Avoid over-partitioning, Columnstore indexes create one or more rowgroups per partition, if the table has too many partitions, there might not be enough rows to fill the rowgroups. A simple way to estimate you rows in one partition is: `[number of rows]/([number of partitions] * 60) > 1 million ? `
6. A group of fact tables if it's too big (billions per tables), then use a unioned view for reporting, which could reduce the index scanning time.
7. Allocate more memory via increase DWUs, means you can simply scale up the underlying dedicated pool.
8. Maximize `rowgroup` quality for CCI performance

9. The typical fact table DDL could look like this:
    ```SQL
    CREATE TABLE [schema].[you_table_name]
    (
        [DateKey] [int] NULL,
        ... 
    )
    WITH
    (
        DISTRIBUTION = HASH ( [DateKey] ),
        CLUSTERED COLUMNSTORE INDEX,
        PARTITION
        (
            [DateKey] RANGE RIGHT FOR VALUES (20220101, 20220201, 20220301, 20220401, 20220501, 20220601, 20220701, 20220801, 20220901, 20221001, 20221101, 20221201)
        )
    )
    GO
    ```



### Maximize rowgroup quality - 1 Million rule

Rowgroup quality is determined by the number of rows in a rowgroup. Increasing the available memory can maximize the number of rows a columnstore index compresses into each rowgroup. Use these methods to improve compression rates and query performance for columnstore indexes.

> Since a columnstore index scans a table by scanning column segments of individual rowgroups, maximizing the number of rows in each rowgroup enhances query performance. When rowgroups have a high number of rows, data compression improves which means there is less data to read from disk. The target size is `1,048,576` or just remember `1 million`.


#### Monitor rowgroup quality

This view exposes useful information suah as number of rows in rowgroups and the reason for trimming.

```SQL
CREATE VIEW dbo.vCS_rg_physical_stats
AS
WITH cte
AS
(
select   tb.[name]                    AS [logical_table_name]
,        rg.[row_group_id]            AS [row_group_id]
,        rg.[state]                   AS [state]
,        rg.[state_desc]              AS [state_desc]
,        rg.[total_rows]              AS [total_rows]
,        rg.[trim_reason_desc]        AS trim_reason_desc
,        mp.[physical_name]           AS physical_name
FROM    sys.[schemas] sm
JOIN    sys.[tables] tb               ON  sm.[schema_id]          = tb.[schema_id]
JOIN    sys.[pdw_table_mappings] mp   ON  tb.[object_id]          = mp.[object_id]
JOIN    sys.[pdw_nodes_tables] nt     ON  nt.[name]               = mp.[physical_name]
JOIN    sys.[dm_pdw_nodes_db_column_store_row_group_physical_stats] rg      ON  rg.[object_id]     = nt.[object_id]
                                                                            AND rg.[pdw_node_id]   = nt.[pdw_node_id]
                                        AND rg.[distribution_id]    = nt.[distribution_id]
)
SELECT *
FROM cte;
```

#### What you need to do when Rowgroups got trimmed

Trim reason could be:
- BULKLOAD: This only happen when the batch load has less than 100,000 rows.
- MEMORY_LIMITATION: you can either Adjust MAXDOP or allocated more memory, the first one is to force the load operation to run in serial mode within each distribution.
- DICTIONARY_SIZE: indicating you have a very long string column.

#### How to estimate memory requirements
The maximum required memory to compress one rowgroup is, approximately, as follows:

```
- 72 MB +
- rows * #columns * 8 bytes +
- rows * #short-string-columns * 32 bytes +
- long-string-columns * 16 MB for compression dictionary
```

> Where short-string-columns use string data types of <= 32 bytes and long-string-columns use string data types of > 32 bytes.

#### Rebuild INDEX
It would take some time but this is the ultimate solution:

```SQL
-- Reorganize the index, not recommand
ALTER INDEX [index_name] ON [table_name] REORGANIZE;

-- rebuild, recreate index
ALTER INDEX [index_name] ON [table_name] REBUILD;
```

### Resource Classes

>By default, each user is a member of the dynamic resource class smallrc.

The resource class of the service administrator is fixed at smallrc and cannot be changed. The service administrator is the user created during the provisioning process. The service administrator in this context is the login specified for the "Server admin login" when creating a new Synapse SQL pool with a new server. You do need to create a specfic user for the heavy ETL job.

To increase a user's resource class, use sp_addrolemember to add the user to a database role of a large resource class. The below code adds a user to the largerc database role. Each request gets 22% of the system memory.

```SQL
EXEC sp_addrolemember 'largerc', 'loaduser';
```




### Reference:
[Best practices for dedicated SQL pools in Azure Synapse Analytics](https://docs.microsoft.com/en-us/azure/synapse-analytics/sql/best-practices-dedicated-sql-pool)
[Maximize rowgroup quality for columnstore index performance](https://docs.microsoft.com/en-us/azure/synapse-analytics/sql/data-load-columnstore-compression)
[Columnstore indexes - Data loading guidance](https://docs.microsoft.com/en-us/sql/relational-databases/indexes/columnstore-indexes-data-loading-guidance?view=azure-sqldw-latest)
[Data Loading performance considerations with Clustered Columnstore indexes](https://techcommunity.microsoft.com/t5/datacat/data-loading-performance-considerations-with-clustered/ba-p/305223)
[Performance tuning with ordered clustered columnstore index](https://docs.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/performance-tuning-ordered-cci#data-loading-performance)
[Memory and concurrency limits for dedicated SQL pool in Azure Synapse Analytics](https://docs.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/memory-concurrency-limits)
[Cheat sheet for dedicated SQL pool (formerly SQL DW) in Azure Synapse Analytic](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/synapse-analytics/sql-data-warehouse/cheat-sheet.md)
[Hands-On with Columnstore Indexes: Part 3 Maintenance and Additional Options](https://www.red-gate.com/simple-talk/databases/sql-server/performance-sql-server/hands-on-with-columnstore-indexes-part-3-maintenance-and-additional-options/)
[Workload management with resource classes in Azure Synapse Analytics](https://docs.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/resource-classes-for-workload-management?context=/azure/synapse-analytics/context/context)