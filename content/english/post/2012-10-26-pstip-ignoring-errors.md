---
title: '#PSTip Ignoring errors'
author: Shay Levy
type: post
date: 2012-10-26T18:00:51+00:00
url: /2012/10/26/pstip-ignoring-errors/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

PowerShell lets you control how it responds to a non-terminating error (an error that does not stop the cmdlet processing) globally via the _$ErrorActionPreference_ preference variable, or at a cmdlet level using the _-ErrorAction_ parameter. Both ways support the following values:

**Stop**: Displays the error message and stops executing.

**Inquire**: Displays the error message and asks you whether you want to continue.

**Continue**: Displays the error message and continues (Default) executing.

**SilentlyContinue**: No effect. The error message is not displayed and execution continues without interruption.

No matter which value you choose, the error is written to the host and added to the $error variable. Starting with PowerShell 3.0, at a command level only (e.g _ErrorAction)_, we have an additional value: _Ignore_. When Ignore is specified, the error is neither displayed not added to $error variable.

```
# check the error count
PS> $error.Count
0

# use SilentlyContinue to ignore the error
PS> Get-ChildItem NoSuchFile -ErrorAction SilentlyContinue

# error is ignored but is added to the $error variable
PS> $error.Count
1

PS> $error.Clear()
# Using Ignore truly discards the error and the error is not added to $error variable
PS> Get-ChildItem NoSuchFile -ErrorAction Ignore
PS> $error.Count
0
```