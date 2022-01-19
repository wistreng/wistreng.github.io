---
layout: post
title: Round-Robin vs HASH distribution
date:  2022-01-05 10:01:27 +1300
tags: [SQLDW, azure]
category: SQL Azure DW
---

>This topic explains the various Azure SQL Data Warehouse distributed table types, and offers guidance for choosing the type of distributed table to use and when. 


### Hash Distributed Table Basics

A hash distributed table is a table whose rows are dispersed across multiple distributions based on a hash function applied to a column. Each SQL instance contains a group of one or more rows. The following diagram depicts how table within SQL DW gets stored as a hash distributed table.

![HASH](https://msdntnarchive.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/50/01/metablogapi/6052.clip_image002_03844B8D.png)

When processing queries involving distributed tables, SQL DW instances execute multiple internal queries, in parallel, within each SQL instance, one per distribution.  These separate processes (independent internal SQL queries) are executed to handle different distributions during query and load processing.

A distribution column is a single column (specified at table creation time) that SQL DW uses to assign each row to a distribution. A deterministic hash function uses the value in the distribution column to assign each row to belong to one and only one distribution. Two identical column values with the same data type will be hashed the same and thus will end up in the same distribution.

In the diagram, each row in the original file is stored on one distribution. The number of rows in each distribution can vary and is usually not identical from distribution to distribution.

There are performance considerations for the selection of a distribution column, such as minimizing data skew, minimizing data movement, and the types of queries executed on the system. For example, query performance improves when two distributed tables are joined on a column that is of the same data type and size. This is called a distribution compatible join or a co-located join.

### Round-Robin Distributed Table Basics

A round-robin distributed table is a table where the data is evenly (or as evenly as possible) distributed among all the distributions without the use of a hash function. A row in a round-robin distributed table is non-deterministic and can end up in different distributions each time they are inserted.

Each JOIN to a round-robin distributed table is a data movement operation. The data movement needed to perform join operations is a separate topic and will be published as a separate blog soon.

Usually common dimension tables or tables that doesn’t distribute evenly are good candidates for round-robin distributed table.

The following diagram shows a round-robin distributed table that is stored on different distribution.

![round robin](https://msdntnarchive.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/50/01/metablogapi/0574.clip_image0027_41F7F018.png)

### Best Practices

In SQL DW, a user query is a logical query that gets divided into many physical queries one for each distribution. The Engine Service on the control node acts as a coordinator and waits for each of these individual queries to finish before returning results or the next part of the multi-step query is executed.

When creating a table in SQL DW, you need to decide if the table will be hash distributed or round-robin distributed. This decision has implications for query performance. Each of these distributed tables may require data movement during query processing when joined together. Data movement in MPP RDBMS system is an expensive but sometimes unavoidable step. In my 8 plus years of working with MPP data warehouse I haven’t seen a real customer workload that can completely eliminate data movement. The objective of a good data warehouse design in SQL DW is to minimize data movement so let’s keep that in mind while choosing table design.

Here are considerations for choosing whether to use a round-robin distributed table or a hash distributed table:

1. To choose a good distribution design with SQL DW, one should know their data, DDL and queries. This is not unique to SQL DW but for most MPP RDBMS system. You need to minimize data movement queries but also watch out for data that can heavily skew a certain distribution. If one of the distribution has more data than others, it will be the slowest performing distribution. Since SQL DW queries are as fast as its slowest distribution, we need to take notes of any data-heavy (skewed or hot) distribution for the same table.

2. A nullable column is a bad candidate for any hash distributed table. All null columns are hashed the same and thus the rows will end up on the same distribution creating a skewed (hot) distribution. If most of the columns are null able and no good hash distribution can be achieved, that table is a good candidate for round-robin distribution. Choose ‘not null’ columns when creating table that will be hash distributed.

3. Any fact tables that has a default value in a column is also not a good candidate to create a hash distributed table. DW Developers will sometime assign -1 value to an otherwise unknown value or early arriving values for a fact table. These values will create data skew on a particular distribution. Avoid these kind of default value column unless you know for sure that the -1 values are negligible in your data.

4. Large fact tables or historical transaction tables are usually stored as hash distributed tables. These tables usually have a surrogate key that is monotonically increasing and are used in JOIN conditions with other fact and dimension tables. These surrogate keys are a good candidate for distributing the data as there are many unique values in that column. This allows the query operations to be performed across all distributions. Each distribution can work independently on separate subsets of data. This takes advantage of the processing resources across the MPP system. Queries on distributed tables may require data movement between distributions during query execution and that is okay.

5. Dimension tables or other lookup tables in a schema can usually be stored as round-robin tables. Usually these tables connects to more than one fact tables and optimizing for one join may not be the best idea. Also usually dimension tables are smaller which can leave some distributions empty when hash distributed. Round-robin by definition guarantees a uniform data distribution.

6. If you are unsure of query patterns and data, you can start with all tables in round-robin distribution. And as you learn the patterns the data can be easily redistributed on a hash key.

7. When using ‘group by’ SQL DW will shuffle the data on the group by key. When multiple keys are present and statistics is up-to-date SQL DW’s cost based optimizer will pick the right key to shuffle the data. If this group by key is heavily non-unique then the query will be slower. A worst case example would be grouping by gender of a large customer table. If your query is running slower, look into explain plan (add the word ‘explain’ before your query and execute) to find out what key is being used as the shuffle key. There may or may not be anything you can do to change this based on the query.