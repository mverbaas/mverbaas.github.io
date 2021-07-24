---
title: "View column definition"
date: 2021-07-24T13:17:30-04:00
categories:
  - Blog
tags:
  - Microsoft
  - SQL
  - T-SQL
---

This week is was discussing a pull request from my co-worker Arthur ([T](https://twitter.com/GuruArthur)), and I was nitpicking on something that probably has to with personal preference of writing T-SQL code. His quick response to me was:

> While we're nitpicking on eachother, why don't you use column definitions in your views?

I was surprised by this and my honest reactions to his remark was that I didn't know that it could be done. I've never seen any T-SQL View definition that did this. So first stop was the official [Microsoft documentation](https://docs.microsoft.com/en-us/sql/t-sql/statements/create-view-transact-sql?view=sql-server-ver15).

```SQL
-- Syntax for SQL Server and Azure SQL Database  
  
CREATE [ OR ALTER ] VIEW [ schema_name . ] view_name [ (column [ ,...n ] ) ]
[ WITH <view_attribute> [ ,...n ] ]
AS select_statement
[ WITH CHECK OPTION ]
[ ; ]  
  
<view_attribute> ::=
{  
    [ ENCRYPTION ]  
    [ SCHEMABINDING ]  
    [ VIEW_METADATA ]
}
```

And there it is, on the first line _[ (column [ ,...n ] ) ]_. Reading further in the document:
>*column*
>Is the name to be used for a column in a view. A column name is required only when a column is derived from an arithmetic expression, a function, or a constant; when two or more columns may otherwise have the same name, typically because of a join; or when a column in a view is specified a name different from that of the column from which it is derived. Column names can also be assigned in the SELECT statement.
> If column is not specified, the view columns acquire the same names as the columns in the SELECT statement.

I think I've never looked this well at the create view documentation, and always the syntax as in the first example:

```SQL
CREATE VIEW hiredate_view  
AS   
SELECT p.FirstName, p.LastName, e.BusinessEntityID, e.HireDate  
FROM HumanResources.Employee e   
JOIN Person.Person AS p ON e.BusinessEntityID = p.BusinessEntityID ;  
GO
```

## Experiment

So now it's time to experiment on a **Test** system. Let's create an empty database and some views that query sys.databases system catalog view.

```SQL
CREATE OR ALTER VIEW [dbo].[test]
AS
SELECT [database_id], [name], [recovery_model_desc]
FROM [sys].[databases]
WHERE database_id < 5;
GO

CREATE OR ALTER VIEW [dbo].[test2] (
    [id]
    , [database_name]
    , [recovery_model]
)
AS
SELECT [database_id], [name], [recovery_model_desc]
FROM [sys].[databases]
WHERE database_id < 5;
GO
```

In this particular small examples the results are equal, besides the column names, because in dbo.test2 view I purposely changed the names of the columns. This same result can be achieved with column aliases.

```SQL
CREATE OR ALTER VIEW [dbo].[test3] 
AS
SELECT [database_id] AS [id], [name] AS [database_name], [recovery_model_desc] AS [recovery_model]
FROM [sys].[databases]
WHERE database_id < 5;
GO
```

In the above query the same results are returned as compared to dbo.test2 view.

```SQL
CREATE OR ALTER VIEW [dbo].[test4] (
    [database_id]
    , [name]
    , [recovery_model_desc]
)
AS
SELECT [database_id] AS [id], [name] AS [database_name], [recovery_model_desc] AS [recovery_model]
FROM [sys].[databases]
WHERE database_id < 5;
GO
```

In this latest working example you can observe that the column definition takes precedense over the column alias.

The next examples both return errors. In dbo.test5 I've removed the column definition from the statement. The create view now fails on a mismatch on columns ('test5' has more columns than specified in the column list.). The other way around it return a similar error ('test6' has fewer columns than specified in the column list.)

```SQL
CREATE OR ALTER VIEW [dbo].[test5] (
    [id]
    , [database_name]
)
AS
SELECT [database_id], [name], [recovery_model_desc]
FROM [sys].[databases]
WHERE database_id < 5;
GO

CREATE OR ALTER VIEW [dbo].[test6] (
    [id]
    , [database_name]
    , [recovery_model]
)
AS
SELECT [name]
FROM [sys].[databases]
WHERE database_id < 5;
GO
```

## Conclusion

The question that remains is what are the benefits of using the more explicit way of defining your views including the view definitions?

While I personally like to be as explicit as possible in the code that goes into source controle, I feel like I will not be doing that for column definitions in my T-SQL views and keep using column aliases in the case that the column names should be different from the table definitions.

I would like to thank Arthur for pointing out that it can be done.
