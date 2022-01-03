---
layout: post
title: When to use T-SQL or SSIS for ETL
date:   2017-10-06 10:01:27 +1300
tags: T-SQL SSIS ETL
category: ETL
---

*When doing ETL, at first, I prefer using T-SQL as I used to be a developer. But then I find SSIS is quite useful in many scenarios. What things should I consider when deciding which one to use? I decide to take a look what the expert say.*
<!--more-->

- **Performance** With T-SQL, everything is processed within the SQL engine.  With SSIS, you are bringing all the data over to the SSIS memory space and doing the manipulation there.  So if speed is an issue, usually T-SQL is the way to go, especially when dealing with a lot of records.  Something like a JOIN statement in T-SQL will go much faster than using lookup tasks in SSIS.  Another example is a MERGE statement in T-SQL has much better performance than a SCD task in SSIS for large tasks
- **Features/capabilities** Some features can only be done in either T-SQL or SSIS.  You can shred *text* in SSIS, but can’t in T-SQL.  For example, text files with an inconsistent number of fields per row can only be done in SSIS.  So certain tasks may force you into using one or the other
Current skill set – Are the people in your IT department more familiar with SSIS or T-SQL?
- **Ease of development/maintenance** Of course, whatever one you are most familiar with will be the easiest, but if your skills at both are fairly even, then SSIS is usually *easier* to use because it is graphical, but sometimes you can develop *quicker* in T-SQL.  For example, having to join a bunch of tables will require a bunch of tasks in SSIS, where in T-SQL it is one statement.  So it might be easier to create the tasks to join the tables in SSIS, but it will take longer to build than writing a T-SQL statement
- **Complexity** SSIS can be more complex because you might need to create many tasks to accomplish your objective, where in T-SQL it might just be one statement, like in the example above for joining tables
- **Extensibility** SSIS has better extensibility because you can create a script task that uses *C#* that can do just about anything, especially for *non-database* related tasks. T-SQL is limited because it is only for database tasks. SSIS also has *logging*, which T-SQL does not
- **Likelihood of deprecation/breaking changes** Minor issue, but T-SQL is always removing features in each release that will have to be rewritten
Types/architecture of sources and destinations – SSIS is better if you have multiple types of sources. For example, it works really well with Oracle, XML, flat-files, etc. SSIS was designed from the beginning to work well with other sources, where T-SQL is designed for SQL Server and it requires more steps to access other sources, and there are additional limitations when doing so
- **Local regulations** Are there some company standards you have to adhere to that would limit which tool you can use?

If you decide T-SQL is the way to go and you just want to execute a bunch of T-SQL statements, it’s still a good idea to wrap them in SSIS Execute SQL Tasks because you can use *logging*, *auditing* and *error handling* that SSIS provides that T-SQL does not. You can also easily run SSIS Execute SQL Tasks in parallel so you are able to run stored procedures in parallel

Note that most people use a *hybrid approach*, where you use SSIS, but for certain situations use an Execute SQL Task in SSIS to execute T-SQL instead of using a data flow. And that Execute SQL Task can also call a stored procedure. Some like this because it allows you to make a change in the stored procedure, avoiding having to make a change in SSIS and re-deploying the package. The negative side of this is that instead of having everything contained within SSIS, you instead have to jump back and forth between SSIS and SSMS when you are building or debugging a project.  It’s a balance you will have to weigh as you go along.

*I think there is one more true differentiator of SSIS – its ability to combine data from different data sources and operate upon it in a single operation*.