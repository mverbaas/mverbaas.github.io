---
title: "Linting markdown"
date: 2023-12-20T13:15:30-04:00
categories:
  - Blog
tags:
  - markdown
  - Development
  - Azure DevOps
  - linting
---
Recently I was reviewing a pull request from a co-worker and I noticed that the markdown documents included were _violating_ the markdown rules. Me, being a fan of rules and static analysis tools, quickly modified some of them using [PR Suggestions](./2022-03-26-AzDO-Suggestion.md).

> To be honest I don't know if these rules are actual rules, or more conventions a team can agree upon. But implementing these rules at least can bring consistency in a teams markdown code.

The second thougt that popped in my head, is how can I prevent this from happening again?

## VS Code Extension

One of the methods I already use for myself when writing markdown documents is using a VS Code Extension, and I have great experience with [this one][vsCodeExt]. It surfaces the violations of the rules as problems, just as it would with _norma_ code.

Moving it from being a personal task to make sure all markdown code is following the rules, I already have VS Code [recommend the extension][workspaceRecommendations]. This would make the suggestion to all other users to install the extension when using our shared repository.

```json
{
  "recommendations": ["DavidAnson/vscode-markdownlint"]
}
```

This doesn't force the issue. It still requires an action from the other persons in the team to install the extention, and it's only for those actually using VS Code. A quick edit in the browser, using notepad or any other text editor, and your efforts go to waste.

## Pipeline

A real solution, in my opinion, is to check the rules during the build phase of your pipeline. All other code is also validated at this time, so why treat the markdown code any other way?

An Internet search led me to the [Code With Engineering Playbook site][site] and specifically the [CI Pipeline for better documentation page][codeEngineering].

I was happy to read that a Microsoft site was also recommending to lint markdown code in a (Azure DevOps) pipeline, and the big plus was that it was using the same linter that is backing the VS Code extension.

I will not be reiterating the steps to implement the linter, this can be found on the [page][codeEngineering]. I can confirm that it works in a small pipeline that I created to verify it.

## Learn

While creating the small try-out pipeline, I also learned that I still have some configuration to do. For some rules, there is a choice to be made. For instance [md029][md029], a choice for the ordered list must be made to have the linter drive the consistency.

## Next steps

One of the things that I still need to do, is to find a way to do something similar in github actions, so the blogs on this site are also enjoying the same validation.

Another thought that I have is to setup a pre-commit action, this would have the validation shift left.

## Conclusion

I happy that this code review send me down this rabbit hole and that I found the exit of the rabbit hole with solutions in areas that I like. As described in the next steps, there is still some investitgation to do.

Hope this little post helps other with the same desire for consistent coding.

[workspaceRecommendations]: https://code.visualstudio.com/docs/editor/extension-marketplace#_workspace-recommended-extensions
[vsCodeExt]: https://marketplace.visualstudio.com/items?itemName=DavidAnson.vscode-markdownlint
[site]: https://microsoft.github.io/code-with-engineering-playbook/
[codeEngineering]: https://microsoft.github.io/code-with-engineering-playbook/continuous-integration/markdown-linting/
[md029]: https://github.com/DavidAnson/markdownlint/blob/main/doc/md029.md
