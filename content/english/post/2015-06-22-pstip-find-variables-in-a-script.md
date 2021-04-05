---
title: '#PSTip Find variables in a script'
author: Ravikanth C
type: post
date: 2015-06-22T18:00:16+00:00
url: /2015/06/22/pstip-find-variables-in-a-script/
views:
  - 12061
post_views_count:
  - 2505
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or later.

I had recently completed writing a PowerShell module for an internal project. Within this module, I ended up using variables in the Script scope to share data between different parts of the module. Now, as the module grew into multi-hundred line script, I had a tough time finding all variables that were declared or used in the Script scope. It was important for me to ensure that these are used and disposed correctly.

[PowerShell AST][1] to the rescue! The VariableExpressionAst gives us a peek into the variables in a script.

```powershell
$AbstractSyntaxTree = [System.Management.Automation.Language.Parser]::ParseInput($psISE.CurrentFile.Editor.Text, [ref]$null, [ref]$null)
$Variables = $AbstractSyntaxTree.FindAll({$args[0] -is [System.Management.Automation.Language.VariableExpressionAst ]}, $true)
$variables | Where-object { $_.VariablePath.IsScript } | Select -Unique
```


In the last line of code, by replacing _$_.VariablePath.IsScript_ with _$_.VariablePath.IsGlobal_, we can find all variables in the Global scope.

Also, in my scenario, I was using PowerShell ISE for script editing and it was easy for me to use the ISE object model to grab the script content. If you have a file on the disk, you can use the _Get-Content_ cmdlet to read the script content instead of _$psISE.CurrentFile.Editor.Text_.

[1]: https://msdn.microsoft.com/en-us/library/system.management.automation.language.variableexpressionast_properties(v=vs.85).aspx