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
During examination of the machines I noticed that the configuration was in place, so the conclusion shifted to why my collection was reporting it didn't come to the same conclusion? Quickly I came to the realization that the query collection the data was using @@SERVERNAME property and did it did not return the right data. In one case it was just empty, and in another case it was not what I assumed.

## When can this occur?

In my quick thought of this, I came to the following two situation where this is more likely happen:

- Cold standby
- Accidental removal

### Cold standby

In a cold standby situation there is a system ready to take over the load if something happens to the primary system. Mostly this is used with an storage system that can be mounted to the secondary system. Data is replicated on SAN level and failover happens by mounting it be either system. As both system can't have the same name on the network, this is happening.

### Accidental removal

The @@SERVERNAME property returns data from the sys.servers table. There is nothing stopping anyone (with the correct permissions) to execute sp_dropserver to remove the item from the table.

## Resolution & Conclusion

The resolution involved changing the queries to use the SERVERPROPERTY function to return the ServerInstance on which the query was executing.

The conclusion is that you should check your assumptions and know what functions you're using, either in T-SQL or any other language that you use. This isn't the first time I ran into this specific situation, and I'm afraid it won't be the last time. In case of Microsoft the functions mostly/always have notes or other warnings that can prevent you from bumping your head, but who starts with reading the documetation if you have some experience.

[servername]: https://learn.microsoft.com/en-us/sql/t-sql/functions/servername-transact-sql?view=sql-server-ver16
[serverproperty]: https://learn.microsoft.com/en-us/sql/t-sql/functions/serverproperty-transact-sql?view=sql-server-ver16#servername-property
