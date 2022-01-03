---
layout: post
title: WHY USE TABULAR?
date:   2017-10-02 10:01:27 +1300
tags: BI SSAS Tabular
picture_frame: shadow
---

*SSAS can be either deployed in multi-dimensional mode or tabular mode or power pivot for SharePoint as well. Tabular mode is a new enhancement in SQL Server 2014 analysis service database structure.*<!--more-->*It is a columnar database capable of incredible performance and compression ratio. At this point, there is a lot of confusion in users on why to use the tabular model when we already have multidimensional model. So, let’s discuss these points first before creating a tabular model project. Let us learn about the SSAS Tabular Data Model.*


## Why to Use:

1. Tabular model is quite easy to understand and implement, and is particularly made for empowering information workers.
2. The tabular model uses DAX for scripting which is similar to using excel formulas and is faster to learn.
3. It uses Vertipaq (x-velocity) engine for in memory column storage, which gives great performance and direct query mode for retrieving data and aggregates directly from the database which is beneficial for querying data in real time.
4. Powerpivot models can be easily upgraded to tabular models, thus providing a path for business users and IT professionals to author models in familiar tools like MS Excel.
5. All the client applications that support multi-dimensional mode will also support tabular and work natively with it. This is because tabular uses the same data provider that understands both MDX and DAX queries.

## When Not to Use:

1. When the source is based on dimensional modeling and has complex relationships with **very large volume** of data.
2. No support for writing back or **parent-child hierarchy**
3. When you want to do **complex calculations**, scoping and named sets


