---
title: "Unit Testing PowerShell with Pester"
date: 2021-03-22T13:17:30-04:00
categories:
  - Blog
tags:
  - PowerShell
  - Pester
  - UnitTest
  - Pipeline
  - Azure
---

Today I spend a number of hours on writing PowerShell Pester tests for a PowerShell function. To not forget the learnings of Today, I thought I write a post about it. This is also a thank you to my co-worker [Roger Baten]()

## Pester

For those unaware of what Pester is, "*Pester is the ubiquitous test and mock framework for PowerShell*". According to its own [website](https://pester.dev/). You can contribute to Pester, as it is [open source](https://github.com/pester/pester).

## Unit Tests

For the how and why about unit testing, I'll refer to this  description from [Microsoft](https://docs.microsoft.com/en-us/azure/architecture/framework/devops/release-engineering-testing#unit-testing)

> Unit tests are tests typically run by each new version of code committed into version control. Unit Tests should be extensive (should cover ideally 100% of the code) and quick (typically under 30 seconds, although this number is not a rule set in stone). Unit testing could verify things like the syntax correctness of application code, Resource Manager templates or Terraform configurations, that the code is following best practices, or that they produce the expected results when provided certain inputs.
>
>Unit tests should be applied both to application code and infrastructure code.
