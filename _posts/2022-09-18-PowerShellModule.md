---
title: "PowerShell Module Build"
date: 2022-09-18T22:17:30-04:00
categories:
  - Blog
tags:
  - VSCode
  - PowerShell
  - Coding
  - Build
---
The last couple of weeks if been playing around with creating a PowerShell module with the goal of automating a build, with the lack of fantasy I called it PS-Module for now. This build should be able to be reproducable for all member off a team.

## Build

During that "build" a number of tasks should be done everytime, and succeed. Otherwise the build must stop. To my knowledge, there are two PowerShell modules that help create a build system, [PSake](https://github.com/psake) and [Invoke-Build](https://github.com/nightroman/Invoke-Build). Highly influenced by [this](https://youtu.be/PTGw4UXPPfo) video, I choose to use Invoke-Build. As the author of the video mentions, this is not a comparison and maybe I was just being lazy.

Invoke-Build uses a PowerShell script with tasks defined to complete the process. My current process has a number of tasks that are executed in order and the end result is a PowerShell script module file (.psm1), a PowerShell script manifest (.psd1) and help documents in MarkDown format generated from the help written in the PowerShell functions.

For my own sanity and helping automating the build, I use the following structure in my repository:

├───build\
│   └───PS-Module\
├───docs\
│   └───help\
├───src\
│   ├───private\
│   └───public\
└───tests\

The build\PS-Module folder is where the output is stored. I'm not totally happy with it, because I'm also used to the build folder containing definitions for pipelines. But this is highly customizable.\
Furthermore the actual source code is in the src folder, with a specific folder for the public and private functions in the PowerShell module. In these folders, every PowerShell function is created in it's own file. This helps make the maintainabity of the functions easier.\
The rest of the folders are self-explanatory.

## Process

The build process goes to the following tasks/steps:

- Clean
- Test
- Analyze
- Update Module Manifest
- Document

With Invoke-build it is easy to daisy chain task together by referencing them in the build script. This is all explained in the github repository.

The order of the tasks is also something to think about. In my case I run my tests first, as a failing tests should halt everything. And the other way around, it is a waste of time to create documents at first when there is still other work that can't complete.

### Clean

The clean task is to make sure the output directory is, surprise, clean. It removes all the files that are recreated automatically later in the build process. In my case it removes the two files in build\PS-Module folder and the psm1 file in the src directory. I think I can do without this extra file in the src folder, but for it is what it is.

### Test

The test task is very simple and clean now. It uses the Pester module to run test files.

```PowerShell
task test {
    Invoke-Pester -Path ".\tests\"
}
```

This just runs all the tests in the \tests folder. Here too I've some work to do, as it is also possible to create code coverage. Which in turn can be pushed to an Azure DevOps pipeline for visibility for everyone. Hopefully in due time I can add both these items, to make it very nice demo for others to reuse.

### Analyze

The analyze task I copied from the [PSake Github](https://github.com/psake/PowerShellBuild/blob/main/psakeFile.ps1) and modified slightly. As you might remember I ❤️ the PowerShell Script Analyzer. This task runs the script analyzer on all the PowerShell (.ps1) files and will throw an error if an error is found. Currently it will just output the warning and informationals to the output window when found. This can be tweaked to your own likings.

```PowerShell
task analyze {
    $analysis = Invoke-ScriptAnalyzer -Path .\src\*.ps1 -Recurse -Verbose:$false
    $errors   = $analysis | Where-Object {$_.Severity -eq 'Error'}
    $warnings = $analysis | Where-Object {$_.Severity -ne 'Error'}
    if (@($errors).Count -gt 0) {
        Write-Error -Message 'One or more Script Analyzer errors were found. Build cannot continue!'
        $errors | Format-Table -AutoSize
    }

    if (@($warnings).Count -gt 0) {
        Write-Warning -Message 'One or more Script Analyzer warnings or informationals were found. These should be corrected.'
        $warnings | Format-Table -AutoSize
    }
}
```

### Update Module Manifest

This is potentially the task where most of the important stuff happens. This task, which I could potentially make two task, converts all of the separate function into a single psm1 file and updates the manifest with the functions to export from the functions in the public folder.

```PowerShell
task updateModuleManifest {

    $publicFunctions = Get-ChildItem -Recurse -Path '.\src\public' | Where-Object { $_.Name -match "^[^\.]+-[^\.]+\.ps1$" } -PipelineVariable file | ForEach-Object {
        $ast = [System.Management.Automation.Language.Parser]::ParseFile($file.FullName, [ref] $null, [ref] $null)
        if ($ast.EndBlock.Statements.Name) {
            $ast.EndBlock.Statements.Name
        }
    }

    $files = Get-ChildItem -Recurse -Path '.\src\*.ps1' | Where-Object { $_.Name -match "^[^\.]+-[^\.]+\.ps1$" }
    foreach ($file in $files) {
        Get-Content $file.FullName | Out-File ".\src\$moduleName.psm1" -Append -Encoding utf8
    }
    
    $ModuleManifest = @{
        Author = $author
        Copyright = "(c) $((Get-Date).year) $Author. All right reserved."
        Path = ".\src\$moduleName.psd1"
        FunctionsToExport = $publicFunctions
        RootModule = "$moduleName.psm1"
        ModuleVersion = '0.0.3'
    }

    Update-ModuleManifest @ModuleManifest
    Copy-Item -Path ".\src\$moduleName.psd1" -Destination ".\build\$moduleName\$moduleName.psd1"
    Copy-Item -Path ".\src\$moduleName.psm1" -Destination ".\build\$moduleName\$moduleName.psm1"
    Import-Module ".\build\$moduleName\$moduleName" -Force -Verbose:$false
}
```

As you can see in the code snippet, the first part goes through all the public folder and uses the AST to get the functions name and puts it in a variable. This works very nicely and will ignore any nested functions that you migth have.

The second part just copies the contents of all the ps1 files into the psm1 file.

After that it uses an hash table to update the module manifest.

### Document

In the document task, which is also straightforward. It uses the PlatyPS module to generate markdown documentation from the help that is available in the help in the functions.

```PowerShell
task document {
    New-MarkdownHelp -Module $moduleName -OutputFolder '.\docs\help' -Force | Out-Null
}
```

## Remaining Challenges

The following things haven't been incorporated but I can see myself work on it in the coming period.

- Dev Containers
- External scripts

### Dev Containers

Dev Containers are a feature in VS Code to run your code separate from the machine where the code is being written. For all the ins and outs on Dev Containers, please follow this [link](https://code.visualstudio.com/docs/remote/remote-overview). At the time of the writing of this article I've already started playing around with it, but is not done yet.

### External scripts

As a SQL DBA it is sometimes usefull to run a sql file through a PowerShell function. While it is possible to put a complete script into a .ps1 file, it has many benefits to have the file referenced. Challenges that I currently see is how to have the end result in a nice and concise output.

## Conclusion

This has been a quickly written down summary of some of the work I've been playing around with. I might revise some of the contents over time and maybe dedicate some extra articles to specific parts.

Hope you liked this write-up and you've learned something.
