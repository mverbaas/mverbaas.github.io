---
title: "Azure DevOps Pull Request Suggestions"
date: 2022-03-26T16:28
categories:
  - Blog
tags:
  - Azure DevOps
  - Pull Requests
---
This week I finally found out how to do suggestions in Azure DevOps Repos Pull Requests. After using Azure DevOps for the last year and only now founding out this feature made me think this is not wide spread known. That is why I wrote this blog post, maybe someone elsee can benefit from it.

## What are sugggestions in PRs

Suggestions in PR enable the reviewer of the Pull Request to suggest, or even change the committed code of the other person. This can be helpful in situations where you notice a small update to the code that can have better results in the end. As reviewer you can even directly add a commit to the existing PR.

After discovering it is easier to find the official documentation, so not to repeat that here is the [link](https://docs.microsoft.com/en-us/azure/devops/repos/git/review-pull-requests?view=azure-devops&tabs=browser#suggest-changes) to it.

## GitHub

Same functionality is available in the GitHub Pull Requests, and as a reference the [link](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/incorporating-feedback-in-your-pull-request) to that documentation as well.

## Conclusion

Using suggestions during Pull Requests can help speed up the delivery, but also maintain the quality of the code. It als helps collaboration between contributors.

That's it for this small post, but I hope it helps someone out there.
