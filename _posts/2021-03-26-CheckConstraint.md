---
title: "Check constraint is changed on creation"
date: 2021-03-22T13:17:30-04:00
categories:
  - Blog
tags:
  - T-SQL
  - Microsoft
  - SQL
  - DACPAC
  - Development
---

Today I performed a schema compare between my SQL Database Project and the deployed version of that project. I expected some differences because of various reasons, but one really struck my eye. The definition of a check constraint was different, see the following two code blocks. The first block it the definition in the SQL database project, the second is the result from the diff. While logically the same, the syntax is different.

Definition:

```sql
CONSTRAINT CK_Value CHECK ([Value] IN ('Enabled', 'Disabled'))
```

Result:

```sql
CONSTRAINT [CK_Value] CHECK  (([Value]='Disabled' OR [Value]='Enabled'))
```

I was wondering why this is happening, because I like my code to be equal over the project and the actual result.

- Is the constraint not deployed?
- Did someone change the database, outside of the project?

## TL/DR

I don't have an explanation for this. I am hoping that through this blog we can get some more spread and find the reasoning for this?

Let's get into some details of what I have found sofar.

## Deployment switches

At first I thought it was something funny in the way the database is deployed. The compared database is deployed using Azure DevOps pipelines. So maybe there is a switch in the [sqlpackage.exe](https://docs.microsoft.com/en-us/sql/tools/sqlpackage/sqlpackage-publish?view=sql-server-ver15) that I'm not aware of, which is not that far fetched by looking at the sheer number of switches. Quickly browsing through the switches online, and in Visual Studio, didn't bring me any closer to a reason why.

I deployed the project to my local machine again, but the same thing happening is there.

When I took a look in the actual script that is deployed, you can see the constraint being dropped and re-added in the same form as in the database project.

## New database

Next I choose to deploy the database as a new database on my local machine. Maybe the constraint is not updated when it is already there, and logically equal.
Unfortunately the same thing happens. The code in the project is not reflected in the result.

## Remove publish from the equation

Next thing I tried is just adding a constraint to a table, but removing Visual Studio and SqlPackage.exe from the equation. If you want. you can even do this yourself with Azure Data Studio, or SQL Server Management Studio.

```sql
DROP TABLE IF EXISTS dbo.Table1;
GO

CREATE TABLE Table1
(
    [Key] INT IDENTITY(1,1) NOT NULL
    , [Value] VARCHAR(32) NOT NULL
);
GO

ALTER TABLE dbo.Table1 WITH CHECK ADD CONSTRAINT CK_Value CHECK ([Value] IN ('Enabled', 'Disabled'));
GO
```

After executing the script, go to the table in the object explorer and script the table as create:

```sql
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[Table1](
  [Key] [int] IDENTITY(1,1) NOT NULL,
  [Value] [varchar](32) NOT NULL
) ON [PRIMARY]
GO
ALTER TABLE [dbo].[Table1]  WITH CHECK ADD  CONSTRAINT [CK_Value] CHECK  (([Value]='Disabled' OR [Value]='Enabled'))
GO
ALTER TABLE [dbo].[Table1] CHECK CONSTRAINT [CK_Value]
GO
```

Even without using Visual Studio and/or SqlPackage, the resulting defintion is different from the source defintion. The IN clause is replaced by an OR clause, without warning or anything. The only small reference found is from Pinal Dave (see third reference below), where he mentions that the query optimizer automatically maps the IN statement to an OR statement. So maybe the optimizer does this optimization at creation time, so it doesn't have to do it at run time.

## Questions

My questions that still remain after this day, and hoping we can resolve together:

- Why does this happen?
- Is it better/faster for the optimizer?
- Should I write all my IN clauses as OR clauses?

## References

- [IN (Transact SQL](https://docs.microsoft.com/en-us/sql/t-sql/language-elements/in-transact-sql?view=sql-server-ver15)
- [Create check constraints](https://docs.microsoft.com/en-us/sql/relational-databases/tables/create-check-constraints?view=sql-server-ver15)
- [IN vs OR](https://blog.sqlauthority.com/2018/06/13/sql-server-performance-comparison-in-vs-or/)
