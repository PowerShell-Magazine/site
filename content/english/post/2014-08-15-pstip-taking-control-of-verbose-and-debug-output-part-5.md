---
title: '#PSTip Taking control of verbose and debug output, part 5'
author: Bartek Bielawski
type: post
date: 2014-08-15T18:00:02+00:00
url: /2014/08/15/pstip-taking-control-of-verbose-and-debug-output-part-5/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or later.

In the final part of the series, we will combine everything we&#8217;ve found out so far and create a tool that will identify any command that needs to be wrapped by advanced function in order to make access to verbose and debug messages easy and natural.

**Note**: You can find a function <a href="https://gist.github.com/PowerShellMagazine/a5db1733f34f1d2a8627" target="_blank">here</a>.

Our function will take only one parameter, (_-Name_), but will validate it extensively using knowledge we already have. First of all, it will check if command exists. Then, it will check if command is already advanced and if it is, there is no need to wrap it. Next we check command&#8217;s Abstract Syntax Tree (AST) to see if _Write-Verbose_ or _Write-Debug_ are present. If these commands are absent, we also won&#8217;t proceed. Entire _param()_ block:


    param (
        [Parameter(Mandatory = $true)]
        [ValidateScript({
            $command = Get-Command -Name $_ -ErrorAction Stop
    
            if ($command.CmdletBinding) {
                throw 'This is already an advanced command.'
            }
    
            $astVerboseOrDebug = $command.ScriptBlock.Ast.FindAll(
                {
                    $args[0] -is [System.Management.Automation.Language.CommandAst] -and
                    $args[0].CommandElements[0].Value -match '^Write-(Verbose|Debug)$'
                },
                $true
       		)
    
            if (-not $astVerboseOrDebug) {
                throw 'No need to turn into advanced: Write-Debug and Write-Verbose not used.'
            }
    
            $true
    	})]
    [String]$Name
    )
    
We filtered out commands that require additional processing and it is time to apply a fix. Depending on the command type, we will either prefix whole name (for scripts) or noun (for functions). Also, AST check for pipeline-friendliness will be different depending on the command type. We will use a _Switch_ statement for that.

```
$commandInfo = Get-Command @PSBoundParameters

switch ($commandInfo.CommandType) {
    ExternalScript {
        $isPipelineFriendly = [bool]$commandInfo.ScriptBlock.Ast.ProcessBlock
        $newName = $commandInfo.Name -replace '^', 'Invoke-'
    }

    Function {
        $isPipelineFriendly = [bool]$commandInfo.ScriptBlock.Ast.Body.ProcessBlock
        $newName = $commandInfo.Name -replace '-', '-Advanced'
    }
}
```

If a wrapped command is marked as &#8216;pipeline-friendly&#8217;, we have to modify parameter block.

```
$paramBlock = [System.Management.Automation.ProxyCommand]::GetParamBlock($commandInfo)

if ($isPipelineFriendly) {
    $inputParameter = @'
        [Parameter(ValueFromPipeline = $true)]
        [System.Object]
        ${InputObject}
    '@

    if ($paramBlock) {
        $paramBlock = $paramBlock, $inputParameter -join ','
    } else {
        $paramBlock = $inputParameter
    }
}
```

We also need to define body of the function. This will be different depending on whether the command is pipeline-friendly or not. We will use formatting operator (_-f_). Therefore, we have to be careful with curly brackets (we have double them whenever they should be taken literally):

```
if ($isPipelineFriendly) {
    $body = @'
        begin {{
            Write-Verbose 'Removing InputObject from PSBoundParameters'
            $PSBoundParameters.Remove('InputObject') | Out-Null
        }}

        process {{
            Write-Verbose 'Running {0} with $InputObject as pipeline input and PSBoundParameters'
            $InputObject | {0} @PSBoundParameters
        }}
	'@
} else {
    $body = @'
    Write-Verbose "Calling {0} with parameters passed."
    {0} @PSBoundParameters
    '@
}
```

We have all building blocks and we just have to generate our command. Because we want to use created command after function completes, we will create it in &#8216;global&#8217; scope:

```
$body = $body -f $commandInfo.Name

$scriptText = @"
[CmdletBinding()]
param (
$paramBlock
)
$body
"@

$scriptBlock = [scriptblock]::Create($scriptText)
New-Item -Path function:\Global:$newName -Value $scriptBlock -Force
```

This function can be used to wrap any function or script to enable _-Verbose_ and _-Debug_ parameters. We just need to specify the name of the target command and we will get updated, advanced function. No more secret verbose and debug outputs!