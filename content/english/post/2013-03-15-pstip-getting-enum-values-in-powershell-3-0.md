---
title: '#PSTip Getting Enum values in PowerShell 3.0'
author: Shay Levy
type: post
date: 2013-03-15T18:00:12+00:00
url: /2013/03/15/pstip-getting-enum-values-in-powershell-3-0/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

Prior to PowerShell 3.0, to return a list of  names or values of an Enumeration object, we needed to use the static methods of the _System.Enum_ type:

```
PS> [System.Enum]::GetNames('System.ConsoleColor')
Black
DarkBlue
DarkGreen
DarkCyan
(...)

PS> [System.Enum]::GetValues('System.ConsoleColor')
Black
DarkBlue
DarkGreen
DarkCyan
(...)
```

PowerShell 3.0 runs on .NET 4.0 and in .NET 4.0 we can get the same information using new _System.Type_ methods:

```
[System.ConsoleColor].GetEnumValues()

- or -

[System.ConsoleColor].GetEnumNames()
```

