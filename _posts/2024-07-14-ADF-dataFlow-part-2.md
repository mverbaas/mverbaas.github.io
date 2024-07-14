---
title: "Azure Data Factory column mapping"
date: 2024-07-14T10:15:30+01:00
categories:
  - Blog
tags:
  - Development
  - Azure Data Factory
  - Data Flows
  - Copy Activity
  - Mapping
---

This is a follow up to my earlier [post][part1]. It goes into the mapping issue that I encountered when using a newly created linked service.

As the performance increase using the new linked service (that uses the managed private endpoint) is really required, I decided to focus on resolving the mapping issue. The method I want to use is based on meta-driven pipelines, explained [here] by [André Kamman][andre].

To use this method, I need to have the mapping in a (in my case) SQL server table and retrieve that during pipeline execution with a stored procedure

## Setup

I've created a mapping table with the following structure:

```SQL
CREATE TABLE dbo.mapping (
    mappingId INT IDENTITY(1,1)
    , tableName NVARCHAR(128) NOT NULL
    , sourceColumnName NVARCHAR(128) NOT NULL
    , sinkColumnName NVARCHAR(128) NOT NULL
    , CONSTRAINT pk_mapping PRIMARY KEY CLUSTERED (mappingId)
)
```
And a stored procedure as following:

```SQL
CREATE PROCEDURE dbo.uspGetMapping @tableName NVARCHAR(128)
AS
BEGIN
    SET NOCOUNT ON;

    SELECT sourceColumnName AS 'source.name'
        , sinkColumnName AS 'sink.name'
    FROM dbo.mapping
    WHERE tableName = @tableName
    FOR JSON PATH;
END
```
A few remarks about the code above:

- The stored procedure outputs a JSON object, and the column aliases use a dotted notation to format the object. See [here][json] for more on that notation.
- The schema for the JSON file I got from [this example][mappingSchema].

## Data Factory

Within Data Factory I had to (re)introduce a pipeline, because of the following advice from [Microsoft documentation][foreach].:

> It's possible to iterate over multiple activities (for example: copy and web activities) in a ForEach activity. In this scenario, we recommend that you abstract out multiple activities into a separate pipeline. Then, you can use the ExecutePipeline activity in the pipeline with ForEach activity to invoke the separate pipeline with multiple activities.

Thus in the foreach loop, I call a pipeline with several parameters, most imporantantly:

- sourceFolder
- sourceFile
- sinkSchema
- sinkTable

The pipeline uses a lookup activity with the tableName parameter to get the JSON object with the column mapping. This is passed into a copy activity that with the parameters can do the copy activity.

## Concluding

With these changes I was able to have similar performance as experienced when there were still data flows in place, with the added cost savings in place. The pipeline that was reintroduced is (probably) reusable to also load the data to the on-premises SQL Server and other pipelines in use.

## Future

There are still some things that potentially can speed up the process. 

- Data Factory gives feedback that there are indexes are enabled.  
  I either want to remove the indexes completely, or create something that stores the index statement in some way or form, removes them before loading and recreated them after loading.
- Increase the storage size of the SQL Database.  
  The size of the storage on the SQL server is close to what we need, but I heard that maybe increasing the storage volume might increase the IOPS.

So there might be two follow ups coming after this. Stay tuned, and thanks for reading.

[data-exposed]: https://learn.microsoft.com/en-us/shows/data-exposed/start-your-metadata-driven-adventure-in-azure-data-factory-data-exposed-mvp-edition
[andre]: https://www.linkedin.com/in/andrekamman/
[json]: https://learn.microsoft.com/en-us/sql/relational-databases/json/format-query-results-as-json-with-for-json-sql-server?view=sql-server-ver16&tabs=json-path#control-output-with-for-json-path
[mappingSchema]: https://learn.microsoft.com/en-us/azure/data-factory/copy-activity-schema-and-type-mapping#tabular-source-to-tabular-sink
[part1]: https://mverbaas.github.io/blog/ADF-dataFlow-copyActivity/
[foreach]: https://learn.microsoft.com/en-us/azure/data-factory/control-flow-for-each-activity#iterate-over-multiple-activities
