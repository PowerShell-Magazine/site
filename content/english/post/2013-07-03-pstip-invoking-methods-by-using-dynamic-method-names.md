---
title: '#PSTip Invoking methods by using dynamic method names'
author: Shay Levy
type: post
date: 2013-07-03T18:00:09+00:00
url: /2013/07/03/pstip-invoking-methods-by-using-dynamic-method-names/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 4.0 or above.

PowerShell enables you to refer to a property of an object in a dynamic fashion:

```
PS> $proc = Get-Process -Id $PID
PS> $property = 'Handles'
PS> $proc.$property
485

PS> $property = 'Name'
PS> $proc.$property
powershell
```

Trying the same approach with object methods failed with an error. Starting in Windows PowerShell 4.0 it is now possible to use a variable that holds the method name and dynamically invoke it:

```
$fso = New-Object -ComObject Scripting.FileSystemObject

Get-ChildItem | ForEach-Object {
    $method = if ($_.PSIsContainer) {'GetFolder'} else {'GetFile'}
    $fso.$method($_.FullName)
}
```

