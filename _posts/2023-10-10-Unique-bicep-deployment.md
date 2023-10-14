---
title: "Bicep unique deployments"
date: 2023-10-10T13:15:30-04:00
categories:
  - Blog
tags:
  - Bicep
  - Development
  - IaC
  - Infrastructure
  - Azure DevOps
---
This is just a short post to document some knowledge that I gained this week, and I don't want to loose. Maybe someone else on the big bad Internet will also find it useful.

I was deploying Azure resources using a main bicep file that calls modules, as shown below.

```mermaid
main --> resource1
     --> resource2
```

In my case the main file is just an entry point to the actual resources, and where I can stitch the resources together.

To deploy the main Bicep file use the [Azure Resource Manager Template Deployment task][azdotask] in Azure DevOps, this is valid for pure ARM templates or Bicep templates. One of the arguments is the DeploymentName, use the $(Build.BuildNumber) system variable to easily relate the Azure DevOps pipeline run with the deployment in Azure.

Going with the example from the [module site][bicepModule], this would be the contents of the main.bicep file. It calls a module file name storageAccount.bicep.


```bicep
module stgModule '../storageAccount.bicep' = {
  name: 'storageDeploy'
  params: {
    storagePrefix: 'examplestg1'
  }
}
```

What I notice when doing this, is that the deployment in Azure was nicely showing a separate deployment for every main file, but every separate deployment of the storage account is named storageDeploy and overwrites an earlier deployment. This is not so convenient.

To overcome this you'd just have to read the [Microsoft article][bicepModule] better. As almost always, Microsoft has documented all the posibilities, but actually taking the time to read all of it is the trick.

Prefixing the name of the deployment with the following string makes the deployments unique again and traceable:

> ${deployment().name}-

And the complete example for easier reference.

```bicep
module stgModule '../storageAccount.bicep' = {
  name: '${deployment().name}-storageDeploy'
  params: {
    storagePrefix: 'examplestg1'
  }
}
```

The result doing this is that the main deployment uses the pipeline name from Azure DevOps, and every module called from the main bicep is prefixed with this same name.

This is it, hope it is enough to save future Mark from reinventing the wheel.

All that is left is point out the complete [learning path][learn] that is available from Microsoft. This, and all the other courses on the platform are great!

[bicepModule]: https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/modules
[learn]: https://learn.microsoft.com/en-us/training/paths/fundamentals-bicep/
[azdotask]: https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/reference/azure-resource-manager-template-deployment-v3?view=azure-pipelines
