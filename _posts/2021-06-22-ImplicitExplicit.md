---
title: "Implicit vs Explicit transactions"
date: 2021-06-22T13:17:30-04:00
categories:
  - Blog
tags:
  - Microsoft
  - SQL
  - Azure
  - T-SQL
---
Recently I was reminded about a query tuning technique that I have learned some time ago. I would to share it with everyone so it can also benefit you.

## TL/DR

By explicity starting transactions for INSERT statements, it might improve the time it take to complete them.

## See by yourself

It is rather easy to show the difference it makes. Let's create an empty database and table with a single column, as always do this on a test/personal system and don't play around on production servers!

### Implicit transaction

I explicitely set a nice size for the data and the log files, so they don't have to autogrow during the testing. Be aware that I'm running my tests on a Docker container running Ubuntu, some settings might differ!

```SQL
CREATE DATABASE [ExpImp]
ON
( NAME = Sales_dat,
    FILENAME = '/var/opt/mssql/data/ExpImp.mdf',
    SIZE = 1000MB,
    MAXSIZE = 5000MB,
    FILEGROWTH = 250MB )
LOG ON
( NAME = Sales_log,
    FILENAME = '/var/opt/mssql/data/ExpImp_log.ldf',
    SIZE = 1000MB,
    MAXSIZE = 2000MB,
    FILEGROWTH = 250MB );
GOGO

USE [ImpExp];
GO

CREATE TABLE [dbo].[t] (
    C1 INT NOT NULL
);
```

Next step is to insert sample data into the table. I will use a while loop and insert the counter value into the C1 column. In this case I will insert 100K rows.

```SQL
DECLARE @Counter INT = 1

WHILE @Counter <= 100000
BEGIN
    INSERT T (C1)
    VALUES (@Counter)

    SET @Counter = @Counter + 1
END
GO
```

This statement runs for an average of 13 minutes for multiple runs on my system.

### Explicit transaction

First clean the table

```SQL
TRUNCATE TABLE [dbo].[t];
GO
```

Now redo the insert, but adding the explicit start and commit to the transacation.

```SQL
BEGIN TRAN
    DECLARE @Counter INT = 1

    WHILE @Counter < 100000
    BEGIN
            INSERT T (C1)
            VALUES (@Counter)
        
            SET @Counter = @Counter + 1
    END
 
 COMMIT TRAN
```

This statement runs for an averare of 30 seconds for mutliple runs on my system. This is 25X decrease in required time!

## Why

If you look at the output of the undocumented function sys.fn_dblog(NULL, NULL) you can see what is written to the transaction log during the execution of the query. You'll find a pattern:

```SQL
LOP_BEGIN_XACT
LOP_INSERT_ROWS
LOP_COMMIT_XACT
LOP_BEGIN_XACT
LOP_INSERT_ROWS
LOP_COMMIT_XACT
```

For every row we insert:

1. An (implicit) transaction is started
2. A row is inserted
3. The (implicit) transaction is committed

This will add up to 300000 log records (and thus size).

In comparison with the explicit transaction, the pattern changes to:

```SQL
LOP_BEGIN_XACT
LOP_INSERT_ROWS
LOP_INSERT_ROWS
...
LOP_INSERT_ROWS
LOP_INSERT_ROWS
LOP_COMMIT_XACT
```

This is what happens now:

1. The (explicit) transaction is started
2. All the rows are inserted
3. The (explicit) transactoin is committed

This will add up to 100002 log records in the transaction log, 1 for starting the transaction, 1 for committing the transaction and 100000 inserts.

## Conclusion

Just by explicitely starting and committing transactions in INSERT statements will improve the speed of the statement and will reduce your footprint on the transaction log (both in required space and writing to it).

As always, don't just copy the code and apply it to a production system. Test the scenario on test systems and only apply them after quality control!
