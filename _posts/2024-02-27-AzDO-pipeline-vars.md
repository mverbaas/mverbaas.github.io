---
title: "Azure DevOps pipeline variable template"
date: 2024-02-27T13:15:30-04:00
categories:
  - Blog
tags:
  - Development
  - Azure DevOps
  - variables
  - yaml
---
This post is about using pipeline variables by using a yml file, knows as a variable template.  
This is a third method of using variables besides inline or variable groups.

![image of a pipeline](/assets/images/_f9f4e05c-e2cf-489e-995d-cf8abecb17a8.jpeg)

## Benefits

I think I prefer to use this method from now on because it removes the variable values from being defined in between the pipeline code (as would be the case with inline variables), but the values are still part of the code (which is not the case with variable groups).

The only drawbacks I see now is that the support for secrets is not clear to me, I think all values are clear text. And in a regulated environment you might not want to have the developers access to the variables for the production environment, which can be achieved with variable groups (and you don't have to only use one option).

## File per environment

One way of using a variable template is to create distinct files per environment, for example a 'dev.yml' file with the values for the development environment. This resembles the way variable groups are used.

for example you could have three files, two parameter files and a pipeline file.

```yaml
# dev.yml
variables
- name: resourceGroup
  value: 'rg-dev'
```

```yaml
# prd.yml
variables
- name: resourceGroup
  value: 'rg-prd'
```

```yaml
# pipeline.yml
jobs:
- job: deployDev
  variables:
  - dev.yml

- job: deployPrd
  variables:
  - prd.yml
```

In this short example you can see two different jobs have their variables read from the two variable templates. Job 'deployDev' reads its variables from the 'dev.yml' file, and the 'deployPrd.yml' from the 'prd.yml'.  
With this you can differentiate between the two environments and have the same pipeline job (in a different stage) and another value for the resource group.

## Parameters

As it is a template, it is also possible to include parameters to the variable file. They just work just as normal parameters to a template file with jobs (as you might have used before).  
This would, for example, enable a single file (instead of a file per environment) with an environment parameter. In that case all the variables are stacked together within a single file.

The trick that you need to pull is that you create another parameter as an object and have default values:

```yaml
parameters:
  - name: environment
    type: string
    values:
      - dev
      - tst
      - prd

  - name: values
    type: object
    default:
      greetings:
        dev: 'Hello dev'
        tst: 'Hello tst'
        prd: 'Hello prd'
```

As you can see you would access the variables with the environment parameter, and with the following line it would select the correct greeting (in this case) based on the key (matching the environment).  
This line does all the magic:

```yaml
variables:
# This part creates variables from the default values per environment
- ${{ each var in parameters.values }}:
  - name: ${{ var.key }}
    value: '${{ var.value[parameters.environment] }}'
```

It is using a loop (each keyword) to access the array with the greetings and use the key as the name of a variable and the value as the value of the parameter. This is usefull when the new values aren't nicely able to use string interpolation.  

> The two snippets above are actually the whole file in this simple example.

## String interpolation

This is more usefull when you can interpolate the strings, in this simple case you could also just use a string interpolation. That would remove the need for the object parameteter. The template file would look something like:

```yaml
parameters:
  - name: environment
    type: string
    values:
      - dev
      - tst
      - prd

variables:
  - name: greetings
    value: 'Hello ${{ parameters.environment }}

```

This is a much easier way compared to the parameter with default values within the object. But it requires for the string interpolation to be possible.  
In my case the the subscriptions for development and test are equal, but there are three separate resource groups. So I would not be able to interpolate the subscription for test.

- subscription: dev/test
  - resource group: dev
  - resource group: test
- subscription: prod
  - resource group prod

By mixing and matching the two techniques I can still use the single variable file.

## Use the variables

To use the template, it should be inserted in the variable section of the stage/job definition:

```yaml
stages:
  - stage: dev
    variables:
      - template: variables.yml
        parameters:
          environment: dev
    jobs:
      - job: outputDev
        steps:
          - bash: |
              echo '${{ variables.greetings }}'
  - stage: tst
    variables:
      - template: variables.yml
        parameters:
          environment: tst
    jobs:
      - job: outputTst
        steps:
          - bash: |
              echo '${{ variables.greetings }}'
  - stage: prd
    variables:
      - template: variables.yml
        parameters:
          environment: prd
    jobs:
      - job: outputPrd
        steps:
          - bash: |
              echo '${{ variables.greetings }}'
```

Again a very simple example, but I think the idea is what counts here. Probably it would also be able to loop the various stages now, as the defintions are equal to each other.

> The variables must be accessed through the variables collection, this has to do with when the values becomes available. In this case at run time, as it is dependant on the parameter to the template file.

## Conclusion

I really like finding out how this works as it was no so clear to me when I first encountered this, this also the reason to write it down as it helps me remember.

If you want to read more about all the options with the variables, please read the [official documentation][vars].

[vars]: https://learn.microsoft.com/en-us/azure/devops/pipelines/process/variables?view=azure-devops&tabs=yaml%2Cbatch#list-variables
