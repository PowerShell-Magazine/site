---
title: '#PSTip ForEach-Object Gotcha!'
author: Ravikanth C
type: post
date: 2012-11-21T19:00:40+00:00
url: /2012/11/21/pstip-foreach-object-gotcha/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

Let&#8217;s say you want to rename a bunch of files in a folder and imagine that you used the following code:

<pre class="brush: powershell; title: ; notranslate" title="">Get-ChildItem -Recurse C:\Scripts | ForEach-Object { $count = 0; Rename-Item -Path $_.FullName -NewName "TestScript${Count}$($_.Extension)"; $count++}
</pre>

What do you think will happen here? You may see numerous error messages that the new file name already exists! Let&#8217;s quickly check the syntax of ForEach-Object cmdlet:

```
PS C:\> Get-Command ForEach-Object -Syntax

ForEach-Object [-Process] <scriptblock[]> [-InputObject <psobject>] [-Begin <scriptblock>] [-End <scriptblock>]
[-RemainingScripts <scriptblock[]>] [-WhatIf] [-Confirm] [<CommonParameters>]

ForEach-Object [-MemberName] <string> [-InputObject <psobject>] [-ArgumentList <Object[]>] [-WhatIf] [-Confirm]
[<CommonParameters>]
```

As you see in the above output, the default parameter set of ForEach-Object cmdlet has -Begin, -Process, and -End parameters. When no parameter name is specified, the script block gets assigned to the -Process parameter  and gets executed every time an object is available in the pipeline. So, this behavior causes $count to get re-initialized to zero in every iteration.


So, how do we work around this? This is where the Begin script block comes handy. The script block passed to the -Begin parameter executes only once and it is a good place to initialize $count variable. Let us see how:

<pre class="brush: powershell; title: ; notranslate" title="">Get-ChildItem -Recurse C:\Scripts | ForEach-Object -Begin { $count = 0} -Process { Rename-Item -Path $_.FullName -NewName "TestScript${Count}$($_.Extension)"; $count++}
</pre>

Also, note that it is not mandatory to specify the -Begin and -Process parameters. We can simply write:

<pre class="brush: powershell; title: ; notranslate" title="">Get-ChildItem -Recurse C:\Scripts | ForEach-Object { $count = 0} { Rename-Item -Path $_.FullName -NewName "TestScript${Count}$($_.Extension)"; $count++}
</pre>