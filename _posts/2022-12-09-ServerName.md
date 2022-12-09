---
title: "ServerName"
date: 2022-12-08T21:17:30-04:00
categories:
  - Blog
tags:
  - SQL Server
  - T-SQL
  - Coding
---
I often use the [@@SERVERNAME][servername] function to get the ServerInstance that I'm working on in t-SQL queries. Although I have read the warnings from Microsoft many times, I still bumbed into the situation recently where I didn't apply the knowledge. The following is copied from the [@@SERVERNAME][servername] document by Microsoft.

> Although the @@SERVERNAME function and the SERVERNAME property of SERVERPROPERTY function may return strings with similar formats, the information can be different. The SERVERNAME property automatically reports changes in the network name of the computer.

There is an equivalent property in the [SERVERPROPERTY][serverproperty] function.

## What happened?

During the execution of a query on multiple machine, the @@SERVERNAME function was used to differentiate the results sets by machine. The resulting report on that data suggested that multiple machines were not having the expected configuration.
During examination of the machines I noticed that the configuration was in place, so the conclusion shifted to why my collection was reporting it didn't come to the same conclusion.

## When can this occur?

- Backup/cold standby
- Accidental removal


[servername]: https://learn.microsoft.com/en-us/sql/t-sql/functions/servername-transact-sql?view=sql-server-ver16
[serverproperty]: https://learn.microsoft.com/en-us/sql/t-sql/functions/serverproperty-transact-sql?view=sql-server-ver16#servername-property
