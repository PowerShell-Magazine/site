---
title: '#PSTip Taking control of verbose and debug output, part 3'
author: Bartek Bielawski
type: post
date: 2014-08-13T18:00:11+00:00
url: /2014/08/13/pstip-taking-control-of-verbose-and-debug-output-part-3/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or later.

In the first two tips of this series we focused on the problem. It is time to implement a solution. The assumption here is simple. We want to fix any command that uses _Write-Verbose_ or _Write-Debug_ and make sure we can use common parameters to enable these outputs. Rather than modifying the source code, we will try to generate wrapper around original command that will preserve its parameters and at the same time make it &#8216;advanced&#8217;. First we define command that we will use for testing:

```
function Test-SimpleParam {
param (
    [string]$Text = 'test',
    [int]$Times = 1
)

Write-Verbose "Creating string with $Text times $Times"
$Text * $Times

}
Test-SimpleParam -Text [My-Test] -Times 3 -Verbose
[My-Test][My-Test][My-Test]
```

How can we fix it and see verbose output when requested? PowerShell 2.0 introduced concept of proxy commands. We could create such proxy command for our simple function, but it may be a bit too much: it would add additional level of complexity without any actual benefit. Instead, we will only &#8216;capture&#8217; parameters using helper class _[System.Management.Automation.ProxyCommand]_:

<pre class="brush: powershell; title: ; notranslate" title="">$meta = Get-Command Test-SimpleParam
$paramBlock = [System.Management.Automation.ProxyCommand]::GetParamBlock($meta)
</pre>

With _param()_ block captured we can create a script block that will be used as a body of our wrapper function:

```
$originalName = $meta.Name
$scriptBlock = [scriptBlock]::Create(@"
[CmdletBinding()]
param (
$paramBlock
)
    Write-Verbose "Calling $originalName with parameters passed."
    $originalName @PSBoundParameters
"@)
```


Finally we can create function. We will do that using _New-Item_ cmdlet with _function:_ drive:

```
$wrapperName = $meta.Name -replace '-', '-Advanced'
New-Item -Path function:\$wrapperName -Value $scriptBlock -Force

CommandType     Name                                               Source
-----------     ----                                               ------
Function        Test-AdvancedSimpleParam
```

If we try to call our wrapper function with the same parameters we have used on the original one we will get expected results:

```
Test-AdvancedSimpleParam -Text [My-Test] -Times 3 -Verbose

VERBOSE: Calling Test-SimpleParam with parameters passed.
VERBOSE: Creating string with [My-Test] times 3
[My-Test][My-Test][My-Test]
```

We got verbose output from both sources&#8211;external and internal function. It will work fine for any simple command with one exception. If a command can take pipeline input (using _$__ variable) wrapping it the way we just did will break the pipeline functionality. In the next tip, we will find a way to create wrappers also for this kind of commands.