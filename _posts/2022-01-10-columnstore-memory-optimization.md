---
layout: post
title: Synapse SQL Optimization II
date:   2022-01-10 10:01:27 +1300
tags: [synapse, cloud, azure]
category: Cloud Azure
---


Push the limits and try to loading a fact table from the finest granular level of data you could ever get, the size of the table is morn than 5 billions a year.

To do that and to build a 'self-service' report on top of it to interact with, Here the check list:

1. Use CCI (cluster columnstore index) for fact table, this means better query performance
2. Hash date column to create distributions (in %99 cases)
3. Use fewer columns, means only keep those necessary columns for fact tables
4. Use fewer string column in fact table, that means use numeric type rather than string types if you could, because string type require more memory than the numeric or date data type, that also means use smallest possible column which support your data so will improve performance. You could put all descriptive columns into dimension tables.
5. Avoid over-partitioning, Columnstore indexes create one or more rowgroups per partition, if the table has too many partitions, there might not be enough rows to fill the rowgroups. A simple way to estimate you rows in one partition is: `[number of rows]/([number of partitions] * 60) > 1 million ? `
6. A group of fact tables if it's too big (billions per tables), then use a unioned view for reporting, which could reduce the index scanning time.
7. Allocate more memory via increase DWUs, means you can simply scale up the underlying dedicated pool.
8. Maximize `rowgroup` quality for CCI performance

9. The typical fact table DDL could look like this:
        ```
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

### Maximize rowgroup quality for columnstore index performance

Rowgroup quality is determined by the number of rows in a rowgroup. Increasing the available memory can maximize the number of rows a columnstore index compresses into each rowgroup. Use these methods to improve compression rates and query performance for columnstore indexes.

> Since a columnstore index scans a table by scanning column segments of individual rowgroups, maximizing the number of rows in each rowgroup enhances query performance. When rowgroups have a high number of rows, data compression improves which means there is less data to read from disk.


### How to estimate memory requirements
The maximum required memory to compress one rowgroup is, approximately, as follows:

- 72 MB +
- rows * #columns * 8 bytes +
- rows * #short-string-columns * 32 bytes +
- long-string-columns * 16 MB for compression dictionary

> Where short-string-columns use string data types of <= 32 bytes and long-string-columns use string data types of > 32 bytes.



