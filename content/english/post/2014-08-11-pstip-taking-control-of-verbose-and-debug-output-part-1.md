---
title: '#PSTip Taking control of verbose and debug output, part 1'
author: Bartek Bielawski
type: post
date: 2014-08-11T21:37:21+00:00
url: /2014/08/11/pstip-taking-control-of-verbose-and-debug-output-part-1/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or later.

One of the PowerShell features that makes writing scripts and functions easier is ability to produce different types of output. We can generate errors, warnings, verbose, and debug messages. For warnings and errors, we don&#8217;t have to do anything extra. By default, both are presented to users. The verbose and debug messages are different. A user has to enable these messages first. This is expected and logical behavior. A user runs a command with a goal to get results back. If we don&#8217;t provide a user with an easy way to enable verbose and debug output, it may remain &#8216;secret&#8217; and, eventually, it will be ignored.

In this series of tips, we will try to see different ways to check if commands (scripts or functions) that we&#8217;ve written, or have taken from someone else, make it easy for the user to control the verbose and debug output and explore the ways to fix it (even if we can&#8217;t modify source of the command).

The verbose messages, generated with the _Write-Verbose_ cmdlet, are used when something doesn&#8217;t work as expected, or when a user wants to understand how a command works. The debug messages, generated with _Write-Debug_, on the other hand are used (as the name suggests) during debugging of a command.

When working with cmdlets enabling verbose or debug output is easy: these commands have common switch parameters that can be used to do that. We can enable this functionality for scripts and functions too. We just need _[CmdletBinding()]_ attribute on _param()_ block, or we have to use _[Parameter()]_ attribute on one of the parameters specified. Either way, we will turn our command into &#8216;advanced&#8217;. The problem with this approach is, if we don&#8217;t enable the advanced function features, our &#8216;simple&#8217; function or script will silently accept any parameters, even if these parameters don&#8217;t exist on our command.

```
function Test-FakeParameter {
    param (
        $First
    )
    "First is: $First"
    "Second is: $Second"
}

Test-FakeParameter -Second test

First is:
Second is:
```

If user will try to use _-Verbose_ switch on &#8216;simple&#8217; command nothing will happen: there is neither error, nor verbose output:

```
function Test-SimpleWithVerbose {
    param ($First)
    Write-Verbose "You specified: $First"
    "First is: $First"
}
Test-SimpleWithVerbose -Verbose -First Test

First is: Test
```

This is confusing. To actually enable verbose output for such command, user would have to change _$VerbosePreference_ variable in current scope:

```
$VerbosePreference = 'Continue'
Test-SimpleWithVerbose -First Test

VERBOSE: You specified: Test
First is: Test
```

Does it work? Yes. Is it best approach for this problem? I don&#8217;t think so. Instead, it would be a good idea to make sure that commands we produce are &#8216;advanced&#8217; (and by extension: they will produce verbose output if user requests it with _-Verbose_ switch). For example: to check if any of functions from given module is not &#8216;advanced&#8217; we can use following command:

```
Get-Command -Module TestVerbose -CommandType Function |
    Where-Object { -not $_.CmdletBinding }

CommandType     Name                                               Source
-----------     ----                                               ------
Function        Test-NoVerbose                                  TestVerbose
Function        Test-Simple                                     TestVerbose
Function        Test-SimpleNested                               TestVerbose
Function        Test-SimpleParam                                TestVerbose
Function        Test-SimpleSubExpression                        TestVerbose
```

In the next tip, we will make our test more granular. If you look closely at the output from the last command, there is one function in _TestVerbose_ module that probably doesn&#8217;t have to be advanced to work properly (I would argue though that there is no reason to keep it &#8216;simple&#8217;). After that we will try to &#8216;fix&#8217; commands that we can&#8217;t fully control.