---
title: '#PSTip Taking control of verbose and debug output, part 4'
author: Bartek Bielawski
type: post
date: 2014-08-14T18:00:37+00:00
url: /2014/08/14/pstip-taking-control-of-verbose-and-debug-output-part-4/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or later.

PowerShell commands, both advanced and simple, can work with the pipeline. The difference is that in the simple commands access to pipeline is implicit with _$__ automatic variable. In advanced commands, we have to explicitly specify which parameter will access pipeline and decide what type of binding we want to use. Because of this difference wrapping any simple command in advanced function without adding explicit pipeline-aware parameter(s) won&#8217;t work.

```
function Test-AdvancedPipeline {
    [CmdletBinding()]
    param (
        [String]$Value
    )

    process {
        $Value * $_
    }
}

1, 2, 3 | Test-AdvancedPipeline -Value Test
```

```
The input object cannot be bound to any parameters for the command either because the command does not take pipeline input or the input and its properties do not match any of the parameters that take pipeline input. (raised by: Test-AdvancedPipeline). 

The input object cannot be bound to any parameters for the command either because the command does not take pipeline input or the input and its properties do not match any of the parameters that take pipeline input. (raised by: Test-AdvancedPipeline). 

The input object cannot be bound to any parameters for the command either because the command does not take pipeline input or the input and its properties do not match any of the parameters that take pipeline input. (raised by: Test-AdvancedPipeline)
```

Error message suggests that even though there is pipeline input, our command doesn&#8217;t know how to use it. This means that if we want to make our wrapper commands fully functional, we have to take this into account. We can go back to Abstract Syntax Tree (AST) to find out if the function can work with pipeline or not. This can be identified by checking if Process block contains any statements. For example: we have two functions that look like they were written with pipeline in mind. One is missing the Process block though so it&#8217;s not really using pipeline:

```
function Test-SimpleWithProcess {
    param (
        [Int]$Times = 1
    )

    process {
        Write-Verbose "Piped: $_"
        $_ * $Times
    }
}

function Test-SimpleNoProcess {
    param (
        [Int]$Times = 1
    )
    Write-Verbose "I think I can use pipeline but it's: [$_] &lt;-- empty!"
    $_ * $Times
}
```

Our test that will identify if function can access pipeline:

```
Get-Command Test-Simple*Process | Where-Object {
    [bool]$_.ScriptBlock.Ast.Body.ProcessBlock
}

CommandType     Name                                               Source
-----------     ----                                               ------
Function        Test-SimpleWithProcess
```

If we would like to create correct wrapper function, we need to do a few additional things. First of all, we have to add pipeline parameter: (_-InputObject_). We need to verify if original command had any parameters and if that was the case, we have to separate our new parameter and old parameters with comma:

```
$meta = Get-Command Test-SimpleWithProcess
$paramBlock = [System.Management.Automation.ProxyCommand]::GetParamBlock($meta)
$originalName = $meta.Name
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
```

With that done, we can define Begin block (where we will remove -InputObject parameter from _$PSBoundParameters_ collection) and define Process block where we will pipe _$InputObject_ to wrapped command. We will also create a script block from these building blocks:

```
$scriptBlock = [scriptblock]::Create(@"
[CmdletBinding()]
    param (
    $paramBlock
    )
    begin {
        Write-Verbose 'Removing InputObject from PSBoundParameters'
        `$PSBoundParameters.Remove('InputObject') | Out-Null
    }

    process {
        Write-Verbose "Running $originalName with `$InputObject as pipeline input and PSBoundParameters"
        `$InputObject | $originalName @PSBoundParameters
    }
"@)
```

With script block created we can go ahead and define new function. Again, we will use _New-Item_ cmdlet and _function:_ PowerShell drive.

```
$wrapperName = $meta.Name -replace '-', '-Advanced'
New-Item -Path function:\$wrapperName -Value $scriptBlock -Force

CommandType     Name                                               Source
-----------     ----                                               ------
Function        Test-AdvancedSimpleWithProcess
```

If we pipe anything to our wrapper function now it will just work:

```
1,2,3 | Test-AdvancedSimpleWithProcess -Times 3 -Verbose

VERBOSE: Removing InputObject from PSBoundParameters
VERBOSE: Running Test-SimpleWithProcess with 1 as pipeline input and PSBoundParameters
VERBOSE: Piped: 1
3
VERBOSE: Running Test-SimpleWithProcess with 2 as pipeline input and PSBoundParameters
VERBOSE: Piped: 2
6
VERBOSE: Running Test-SimpleWithProcess with 3 as pipeline input and PSBoundParameters
VERBOSE: Piped: 3
9
```

We have all we needed except for actual tool that will look for commands with these kind of issues and apply our &#8216;fix&#8217; automatically. This will be our goal in next and final part of this series.