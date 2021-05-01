---
title: '#PSTip Get the path to x86 special folders'
author: Shay Levy
type: post
date: 2013-07-17T18:02:39+00:00
url: /2013/07/17/pstip-get-the-path-to-x86-special-folders/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

Using the [_System.Environment+SpecialFolder enumeration_][1] we can retrieve a list of all paths to system special folders. One of the values is: _System_ which holds the path to the Windows system folder. 

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; [System.Environment]::GetFolderPath('System')
C:\Windows\system32
</pre>

On 64-bit machines with .NET Framework 4 installed you will find another value: _SystemX86_ (there are more x86 items in the SpecialFolder enumeration).

```
PS> [Enum]::GetNames('System.Environment+SpecialFolder') | Sort-Object
(...)
SendTo
StartMenu
Startup
System
SystemX86
Templates
UserProfile
Windows

PS> $sysx86 = [System.Environment]::GetFolderPath('SystemX86')
PS> $sysx86
C:\Windows\SysWOW64
```

We can use it to construct a path and invoke a 32-bit instance of Windows PowerShell.

```
PS> $psx86 = Join-Path -Path $sys86 -ChildPath WindowsPowerShell\v1.0\powershell.exe
PS> $psx86
C:\Windows\SysWOW64\WindowsPowerShell\v1.0\powershell.exe

PS> Start-Process -FilePath $psx86
```

[1]: (http://msdn.microsoft.com/en-us/library/system.environment.specialfolder.aspx

