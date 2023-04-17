---
title: "Three is a charm"
date: 2023-04-16T13:15:30-04:00
categories:
  - Blog
tags:
  - Database
  - Development
  - tSQLt
---

In this post I want to go over a small improvement which I learned recently. This includes creating a reference to a dacpac instead of including all the code to build that dacpac.

TL/DR; You can reference a dacpac in you database development, excluding a whole lot of repetative code in you source control system(s).

## History

I've made it a best practice to include testing for all the objects I created, and for database projects my preference went to [tSQLt][tsqlt]. So for the small number of projects that I was a part of, it always included a copy of all the objects of the tSQLt framework.

### The old way

In the previous projects I deployed tSQLt on my (local development) instances as a database named tSQLt. This is straightforward, you hit the download link on the [tSQLt][tsqlt] site and run the provided script on a newly created database.

In Visual Studio I'd create a new database project in my solution, and import the newly created database. In yet another project I'd create the actual tests and have a reference to the tSQLt project.

During Azure DevOps pipeline runs, the tSQLt project was build generating an artifact (dacpac) and deployed (on the test environment).

This would be done every deployment, and all the defintions of the tSQLt framework were in source control. In with my team we had multiple solutions, all having this project in it. So a lot of duplication in source control, for items that we really don't need to keep in source control and modify.

### The new way

In this new way, I just copy the (correct version) of the dacpac (also provided in the download) to my source control folder (i.e. .\dep\tsqlt\tsqlt.dacpac) and reference that in the [database reference][composite] dialogue window.

This way, you still have access to all the objects (that are contained in the dacpac file), without the need to have the actual objects (tables/stored procedures/functions, etc.) in source control.

In the pipeline, while deploying the actual test files/stored procedures, you would specify the option to include the composite objects ([IncludeCompositeObjects][sqlpackage]='true'). This will ensure the tSQLt objects will also be deployed in the same publish operation. You do need to publish the dacpac from the source control, but it will also be in the build output.

> NOTE\
> Although you specify to deploy the compisite objects, you will have to reference it somewhere in your code, otherwise it will *not* be deployed.

By doing it this way, you safe on a number of things:

- Less actual files in source control
- Less repetion (if you have more than one solution)
- Less time required building the tSQLt framework

Hope you can use this to your benefit.

[tsqlt]: https://tsqlt.org
[composite]: https://learn.microsoft.com/en-us/sql/ssdt/add-database-reference-dialog-box?view=sql-server-ver16#to-create-a-composite-project
[sqlpackage]: https://learn.microsoft.com/en-us/sql/tools/sqlpackage/sqlpackage-publish?view=sql-server-ver16
