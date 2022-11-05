---
title: "T-SQL Tuesday 138: Managing Technology Changes"
date: 2021-05-11T13:17:30-04:00
categories:
  - Blog
tags:
  - T-SQL
  - T-SQL Tuesday
---

This is my first contribution to T-SQL Tuesday. It is about how I manage technology changes. As this is the question from this months host, [Andy Leonard](https://andyleonard.blog/2021/05/t-sql-tuesday-138-managing-technology-changes/). Thank you for hosting this edition of the T-SQL Tuesday, Andy ([T](https://twitter.com/AndyLeonard))!

[![T-SQL Tuesday Logo](/assets/images/T-SQL-Tuesday-Logo.jpg)](https://andyleonard.blog/2021/05/t-sql-tuesday-138-managing-technology-changes/)

I think my way of responding to change is greatly influenced by one of my former employers. It was not a question if we should go to newer versions of the, mostly Microsoft, products we then used. Being it SQL Server, Exchange or the Windows operating system on either the server park, or the client. It was more of a question how fast can we go there? After an employer change, I was kind of shocked getting a laptop with Windows 7 and the OCS client on it, being "spoiled" with mostly the latest versions available.

I still embrace this change! After putting in some real work to get an application deployed using Azure DevOps pipelines, Microsoft introduced YAML pipelines. And directly I could see the benefits over the drawbacks and started migrating the pipeline to YAML.

## General

Most of the times, the technology change, or new method of achieving the same results will be standard method of the supplier of the method. So it is a matter of time that the migration or upgrade has to performed anyway.

In one adventure in my career there was an application that "couldn't" be migrated of a lower version of SQL server. Every 6 months the manager came and asked us whether the old technology was giving us problems, which it didn't. Nobody touched it, and it kept doing its job swiftly. This was a great contradiction to the new SQL Server that was using newer technology, like Always-On availability groups on Windows Core and dacpac deployments of an application was being developed and deployed on the daily basis. The amount of hours that were spent on these instance was much higher, but it was every day a little bit and from every hour spend we gained some knowledge.

## disaster strikes

This was all fine until the day came the old machine, and the other servers in the same datacenter, was hit with an disaster. The department running on the old servers were down for several days untill the application was moved to the newer servers while applications running on the Always-On instances were online again in hours. It took a lot of ad-hoc, dedicated time and nightly hours to get this application upgraded. A lot of assumptions and guesses were made. Fortunately it worked out fine in the end.

## Conclusion

Change is inevitable in technology and in life in general. Accepting this fact, makes it easier to be prepared and respond to it.
