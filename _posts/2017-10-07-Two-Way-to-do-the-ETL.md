---
layout: post
title: Two Ways to do the 'ETL'
date:   2017-10-07 10:01:27 +1300
tags: SSIS ETL Staging
picture_frame: shadow
---

*The first approach is what I call the “obvious approach”. Many tutorials about SSIS are written from the standpoint that you obviously extract and transform data using data flows in SSIS. I’ve run into many roadblocks using the obvious approach, which led me to an alternative approach which relies on using staging tables. This staging table approach not only solves many of the issues I had before, it also provides a simple and robust way to design and build ETL processes in SSIS. Once your familiar with this approach, you’ll be able to create a basic BI solutions hours instead of days.*
<!--more-->

The ‘T’ is ETL stands for transformation. The goal of transformation is to convert raw input data to an OLAP-friendly data model. This is also known as dimensional modeling. If you’re not familiar with this I highly recommend reading Ralph Kimball’s book “The Data Warehouse Toolkit: The Complete Guide to Dimensional Modeling”. Understanding dimensional modeling is essential for building BI systems.

### What’s ETL again?
When designing the ETL process it’s good to think about the three fundamental things it needs to do:

- **Extract** data from the external data sources such as line-of-business systems, CRM systems, relational databases, web services, and SharePoint lists.
- **Transform** the data. This includes cleansing the data and converting it to a OLAP-friendly data model. The OLAP-friendly data model traditionally consists of dimension and fact tables in a star or snowflake schema and closely maps SSAS’s dimensional model (SSAS stands for SQL Server Analyses Services ).
- **Load** the data so that it can be quickly accessed by querying tools such as reports. In practice this implies processing SSAS cubes.

An ETL process is a program that periodically runs on a server and orchestrates the refresh of the data in the BI system. SQL Server Integration Services (SSIS) is a development tool and runtime that is optimized for building ETL processes. Learning SSIS involves a steep learning curve and if you have a software development background like I do, you might first be inclined to build your ETL program from scratch using a general purpose programming language such as C#. However, once you master SSIS you’ll be able to write very efficient ETL processes much more quickly. This is because SSIS lets you design ETL processes in a graphical way (but if needed you can write parts using VB or C#). The SSIS components are highly optimized for ETL type tasks and the SSIS run-time executes independent tasks in parallel where possible. If you’re a programmer you’ll find it amazingly difficult to write your own ETL process using a general purpose language and make it run more efficient than one developed in SSIS.

### The obvious way: using SSIS data flows to transform the data

In SSIS you can design your ETL process using control flows and data flows. Data flows in SSIS are a type of control flow that allow you to extract data from an external data sources, flow that data through a number of transformations such as sorting, filtering, merging it with other data and converting data types, and finally store the result at a destination, usually a table in the data warehouse. This is very powerful and data flows seem to lend themselves very well for integrating the extract and transformation tasks within them. This is why I call this the “obvious” approach and many tutorials about SSIS follow this approach. The obvious approach seems especially attractive because it is very efficient and there’s no need to store intermediate results. The figure below illustrates this process:

![obvious approach](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/01/57/11/metablogapi/0841.clip_image001_thumb_32A68E1A.jpg)

The top level control flow in the Integration Services project may look like this:

![obvious approach](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/01/57/11/metablogapi/3566.clip_image003_thumb_094F461C.jpg)

The “Extract and Transform” box is a sequence container that holds a data flow for each of the tables that will be refreshed in the data warehouse. In this example there is one fact table and there are three dimension tables. SSIS will execute the data flows in parallel, and when all of them have successfully completed the cube will be processed.

The transformation of data takes place in the data flows. The transformations needed in each of the data flows would typically look something like this:

![transformations data flows](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/01/57/11/metablogapi/6278.clip_image005_thumb_6D5E1123.gif)

### What’s wrong with the obvious approach?

There’s nothing necessarily wrong with obvious approach, it works fine and when the transformations are designed well it’s very efficient. But there are several reasons why I dislike using this approach:

· I have found that developing and debugging transformations within data flows is very time-consuming. You’re working on a very low abstract level, as if you’re building a complex machine from scratch using basic components such as nuts and bolts.

· Every time you test run a data flow it will need to fetch data from the external data source. This slows down testing and debugging even more because the external data source might have millions of rows of data. Moreover, you are putting a high burden on the external data source, which is a production database. 
Of course you can work around this by having a test external database on your local development machine, but then you need to spend time creating it and filling it with a subset of the data.

· When you update an existing data flow that already is in production, you must be very careful, especially in case the data flow maintains state such as with a slowly moving dimension with historic attributes. Your changed data flow may no longer be compatible with the stored state, causing lots of headaches.

### Using staging tables
A much better approach is to keep extraction and transformation as two strictly separated steps. First you extract data from the external data source and store a “raw” copy of the data in staging tables in the data warehouse. With “raw” I mean that you keep the column names the same as in the source database and you don’t convert data, calculate new data fields, etc. You may however filter unneeded rows and columns as you extract data so that you don’t waste resources on unneeded data. That being said, if size and performance are not an issue it’s more convenient to just load the entire source tables.

As with the obvious approach, you use data flow components to pull the data from the data sources. However, since they are now only used to extract data, the flows will be much simpler. Instead of using data flows in SSIS to transform the data to dimension and fact tables, you can now use database views to convert the raw data in the staging tables to dimension and fact views. At some point before deploying to production you will likely replace views by tables and stored procedures. I’ll come back to that later.

Tip: There’s a quick and easy way to create staging tables from within SSIS. When you add a destination data source to a data flow, connect it to incoming data and then edit the destination’s properties, you will be asked to select a destination table. At that point you also have the option of creating a new table. Doing so will create a database table with all the right columns.

![Use staging table](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/01/57/11/metablogapi/6278.clip_image006_thumb_0A83B92E.jpg)

The top level control flow now looks like this:

![control flow](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/01/57/11/metablogapi/2072.clip_image008_thumb_60C03E3A.gif)

Instead of combining Extract and Transform, they have now become separate steps in the top level control flow.

When you look at the Extract sequence container, you’ll there are now more data flows than in the obvious approach because we now have a data flow per staging table instead of one per dimension/fact table. If the source data is highly normalized as in this example, the number of staging tables will typically be bigger than the number of dimension and fact tables.

For each data source we now have two operations: truncate the table and then retrieve data through a data flow task. This is because this example uses a very simple approach to refresh the data in our data warehouse: first empty the table (which contains data from the previous ETL run) using an SQL TRUNCATE TABLE statement, and then reload the entire table from the data source using a data flow.

Let’s take a look at one of the data flows, SalesOrderHeader:

![data flows](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/01/57/11/metablogapi/2061.clip_image010_thumb_7732DCC1.gif)

As you see the data flow for retrieving the data is very straightforward compared to the data flow in the obvious approach: it simply gets the data from the external database and then stores it in the staging table in the data warehouse. The other data flows are also this simple.

Once the data is extracted, the ETL process will continue with the transformation step. Since we implement transformations as database views instead of using data flow transformations in SSIS, we don’t need to do anything special during the transformation phase in the SSIS package (for now at least).

For example, the transformation for the sales transaction fact table could look something like:

```SQL
CREATE VIEW [Fact sale transaction] AS
SELECT d.[SalesOrderID] AS [Sales order key]
,d.[ProductID] AS [Product key]
,[TerritoryID] AS [Territory key]
,dbo.DateToDateKey(h.[OrderDate]) AS [Date key]
,d.[OrderQty] AS [Order quantity]
,d.[UnitPrice] AS [Unit price]
,d.[UnitPriceDiscount] AS [Unit price discount]
,d.[LineTotal] AS [Line total]
FROM [Staging].[SalesOrderDetail] d
JOIN [Staging].[SalesOrderHeader] h ON h.[SalesOrderID] = d.[SalesOrderID]
```

A couple of things to point out in this view:

· Each of the columns are made “business user friendly” by giving meaningful column names and including spaces. The idea is that this table is targeted towards business users, not software developers.

· ID fields are renamed to “key” fields and these will later be used to join fact tables to dimension tables. Many BI specialis recommend introducing your own key fields instead of relying on ID fields in the source database. I often ignore that advice and can’t remember every running into trouble because of it. If we would need to introduce our own row identifiers, it’s not difficult to do so later.

· You’ll often need to join multiple staging tables in order to produce the right results for the dimension and fact tables. In this example most fields are from the order detail table, but the order date needs to come from the order header table. This is an example of how OLAP data models are less normalized than the highly normalized data models which are typical for transactional databases.

### How is using staging tables better than the obvious approach?

Remember the disadvantages of the obvious approach:

In the obvious approach transformations are very time-consuming to develop and debug. With staging tables, transformations are implemented as database views, which are much simpler and less time consuming to develop because you can do it interactively in SSMS. If your SQL skills are a bit rusty, it’s definitely worth investing some time on improving those skills that instead of learning how to use all the different data transformations in SSIS.

In the obvious approach, in order to test the transformations you need to run the data flow which each time pulls the data from the external data sources. With staging tables, you only need to run the Extract data flows once in order to fill the staging tables with test data. From that point onwards you can simply write and test SQL queries within your data warehouse database.

Perhaps you may still want to use fake databases to mimic the external databases. However, I find I can often get away using the production databases during development because I hardly ever need to re-run the extract part of the ETL process. (I do not recommend using the production data warehouse database during development! You should have your own copy of the data warehouse database on your development machine.)

In the obvious approach you needed to be very careful when updating transformations. With staging tables the transformation is just a database view and you won’t corrupt any data by changing the view.

### Further improving the approach with staging tables

There are several ways we can further improve to the approach with staging tables.

First of all, although the use of database views for implementing transformations simplifies development, it’s not very efficient from a runtime performance perspective. This may not be a problem if the amount of data is small and when the cube is the only “client” of the database views, but otherwise the views can quickly become a bottleneck.

The way to solve this is to replace the database views by tables which are filled by stored procedures. For example, instead of having the Fact sale transaction view, you would create a Fact sale transaction table and a stored procedure which runs a similar query as the original view and stores the results in the table.

The nice thing about this approach is that you can start developing the system by creating views and then, before deploying to production, replace the views by tables and stored procedures. The stored procedures are called within the Transform step in the ETL process.

A downside compared to the obvious approach is that the data warehouse database will require more disk space because data is stored twice (first at the staging level and then at the dim/fact level). Depending how efficient the transformations are made, the obvious approach may also perform better. Therefore, if you work with big datasets (hundreds of millions of rows or more) you may still want to consider using the obvious approach instead of staging tables.

A second area for improvement is the way staging tables are being refreshed using the flush-and-reload method. When the data flow fails, for example because the source database is offline, we end up with a couple of empty staging, dimension and fact tables in the data warehouse and the result is that we cannot process the cube. It’s better to be more resilient by processing the cube even though some of the data sources have failed. For the failed data sources, the cube should use the data from the previous ETL run. There are several solutions to solving this problem, but that’s outside the scope of this post.