---
title: "VS Code and PSScriptAnalyzer"
date: 2022-06-16T22:17:30-04:00
categories:
  - Blog
tags:
  - VSCode
  - PowerShell
  - ScriptAnalyzer
  - Coding
  - Static Analysis
---
People that know me, know that I love following coding rules and standards.

For instance for this blog I using GitHub pages and create markdown documents in VS Code. I've installed an [extention](https://marketplace.visualstudio.com/items?itemName=DavidAnson.vscode-markdownlint) in VSCode to lint the documents. In my professional environment, the SQL code is scanned by SonarQube and I follow all (except one) by the book. In all of my checked in T-SQL code I use square brackets around all the identifiers. For PowerShell scripts I'm a great fan of the PowerShell Script analyzer.

## PowerShell Script Analyzer

The script analyzer checks your code for best practices and other rules.\
According to the [documentation](https://docs.microsoft.com/en-us/powershell/module/psscriptanalyzer/?view=ps-modules):

> PSScriptAnalyzer is a static code checker for PowerShell modules and scripts. PSScriptAnalyzer checks the quality of PowerShell code by running a set of rules.

Most people might also know it, because it is integrated in your favorite editor when you install the PowerShell extension.

Known rules are:

- PSAvoidUsingCmdletAliases\
  Warns about the usage of aliases in your script.
- PSPossibleIncorrectComparisonWithNull\
  Warns about null comparison where null is not on the left side of the comparison.

For all the default rules check the [documentation](https://docs.microsoft.com/en-us/powershell/utility-modules/psscriptanalyzer/rules/readme?view=ps-modules) or execute the following command:

```powershell
Get-ScriptAnalyzerRule
```

The cool thing with the ScriptAnalyzer module is that you can integrate it in your Azure DevOps pipeline through a pester script. The combination of the both will give you a nice repetative checking of your files in source control.\
The following image shows the result in my playground. You can also see on the top that the number of "tests" are quickly rising (in this case over 5300). This is because every rules is applied to every file. So a new file will add over 60 tests to your pipeline.

![PSSA Pipeline](/assets/images/2022-06-16-PSSA-Pipeline.jpg)

## Custom configuration

Now that you have a quick recap on the cool things you can do with the script analyzer and some other related tools, back to the topic on hand.\
In the pipeline execution all rules are checked (this is of course configurable), but with my strange preference I would never do that. By default the ScriptAnalyzer in VS Code excludes a number of items, so these items would only be flagged during pipeline execution or manual invokation of the script analyzer.\
A better way is to have VS Code look at the same rules as your pipeline. A way to do this is to create a custom configuration, this is documented [here](https://docs.microsoft.com/en-us/powershell/utility-modules/psscriptanalyzer/using-scriptanalyzer?view=ps-modules#explicit). Within VS Code, you can point your settings to this custom configuration, either for all your VS Code sessions or just for the workspace. WIth the below snippet I've created a custom configuration in the VS Code configuration folder (that is a lot of configurations).

```JSON
"powershell.scriptAnalysis.settingsPath": ".\\.vscode\\PSScriptAnalyzerSettings.psd1"
```

The contents of the file are pretty simple, and this just to do everything.

```JSON
@{
    Severity=@('Error','Warning', 'Information')
}
```

## Examples

With the following pseudo code you'll get a single by default, and it is hard to see that there is an extra space behind the Get-Date command.

```PowerShell
Get-Date 

ls
```

There is only a warning about the alias that I've intentially used.

```json
[{
 "resource": "Untitled-1",
 "owner": "_generated_diagnostic_collection_name_#1",
 "code": "PSAvoidUsingCmdletAliases",
 "severity": 4,
 "message": "'ls' is an alias of 'Get-ChildItem'. Alias can introduce possible problems and make scripts  hard to maintain. Please consider changing alias to its full content.",
 "source": "PSScriptAnalyzer",
 "startLineNumber": 3,
 "startColumn": 1,
 "endLineNumber": 3,
 "endColumn": 3
}]
```

But when I enable the custom configuration it will light up the by default hidden information item that tries to pursuade us to remove the trailing space on line 1.

```json
[{
"resource": "Untitled-1",
"owner": "_generated_diagnostic_collection_name_#1",
"code": "PSAvoidUsingCmdletAliases",
"severity": 4,
"message": "'ls' is an alias of 'Get-ChildItem'. Alias can introduce possible problems and make scripts ard to maintain. Please consider changing alias to its full content.",
"source": "PSScriptAnalyzer",
"startLineNumber": 3,
"startColumn": 1,
"endLineNumber": 3,
"endColumn": 3
},{
"resource": "Untitled-1",
"owner": "_generated_diagnostic_collection_name_#1",
"code": "PSAvoidTrailingWhitespace",
"severity": 2,
"message": "Line has trailing whitespace",
"source": "PSScriptAnalyzer",
"startLineNumber": 1,
"startColumn": 9,
"endLineNumber": 1,
"endColumn": 10
}]
```

While I recommend resolving the actual issues, another way can be to change the custom configuration and excluded the rules.

```json
@{
    Severity=@('Error','Warning', 'Information')
    ExcludeRules=@('PSAvoidTrailingWhitespace', 'PSAvoidUsingCmdletAliases')
}
```

## Conclusion

By using a custom configuration file in your repo, and checking it into the repository the whole team can use the same configuration and together make all the code better and following the agreed upon standards. I hope you learned something, I did!

## Acknowledgements

I would like to say thanks [Arthur Baan](https://www.linkedin.com/in/arthurbaan/) and [Roger Baten](https://www.linkedin.com/in/roger-baten-2a68a04/) for their inspiration. With Arthur I have once played with a custom configuration file. And Roger pointed me to the [DBATools github](https://github.com/dataplat/dbatools) where they also use a custom configuration file. Combining these led to this little write up!
