---
title: '#PSTip Taking control of verbose and debug output, part 2'
author: Bartek Bielawski
type: post
date: 2014-08-12T18:00:39+00:00
url: /2014/08/12/pstip-taking-control-of-verbose-and-debug-output-part-2/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or later.

In the [first part of this series][1], we identified the problem&#8211;commands (scripts or functions) that &#8216;partially&#8217; support verbose and debug messages. And the solution we identified was to make sure any command that you write is &#8216;advanced&#8217; and supports common parameters (in a similar fashion that cmdlets support them). In this tip, we will try to narrow the scope of our test to commands that use verbose and/or debug output.

The idea is simple. If a command doesn&#8217;t use _Write-Verbose_ and _Write-Debug_, it can remain &#8216;simple&#8217; and will behave as expected anyway. There are a few ways to approach this. We can use regular expressions or PowerShell parser. Neither is optimal and should probably be used only if we are &#8216;stuck&#8217; with PowerShell 2.0. In PowerShell 3.0 we can test command structure using its Abstract Syntax Tree (AST). AST has _FindAll()_ method that will help us find any appearance of _Write-Verbose_/_Write-Debug_:

```
Get-Command -CommandType Function, ExternalScript | Where-Object {
    ($_.ScriptBlock.Ast.FindAll(
        {
            $args[0] -is [System.Management.Automation.Language.CommandAst] -and
            $args[0].CommandElements[0].Value -match '^Write-(Verbose|Debug)$'
        },
        $true
    )) -and
    -not ($_.CmdletBinding)
} 

CommandType     Name                                               Source
-----------     ----                                               ------
Function        Test-SimpleWithVerbose
Function        Test-Simple                                        TestVerbose
Function        Test-SimpleNested                                  TestVerbose
Function        Test-SimpleParam                                   TestVerbose
Function        Test-SimpleSubExpression                           TestVerbose
ExternalScript  FakeDebug.ps1                                      e:\PATH\FakeDebug.ps1
```

How does it work? The _FindAll()_ method takes two arguments. The first argument is a script block that will be used to analyze any AST element present in command syntax tree. In this script block, _$args[0]_ represents syntax element. We check if element is a command and if the command name is _Write-Verbose_ or _Write-Debug_. The second argument is used to decide if search should be recursive or not. This method returns any element of AST for which script block returned _$true_. If we find _Write-Verbose_ or _Write-Debug_ then the second test will filter out any &#8216;advanced&#8217; commands. The advantage of using AST is that it finds commands anywhere, including nested functions and sub-expressions inside double quotes:

```
function Test-SimpleSubExpression {
    "$(Write-Verbose test)"
}

function Test-SimpleNested {
    function Helper {
        Write-Verbose Helping
    }
    Helper
}
```

In my opinion though, there is no extra cost of making command &#8216;advanced&#8217;. Any function that is exported from a module is perfect candidate for &#8216;advanced&#8217; function. However, if we want to fix commands without modifying the source, (e.g. 3rd-party module) being able to identify commands that require such update can limit amount of work. We will try to do that in the next parts of this series.

[1]: /2014/08/11/pstip-taking-control-of-verbose-and-debug-output-part-1/