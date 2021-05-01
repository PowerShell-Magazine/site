---
title: '#PSTip Validating version numbers without RegEx'
author: Ravikanth C
type: post
date: 2014-01-03T19:00:22+00:00
url: /2014/01/03/pstip-validating-version-numbers-without-regex/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
<span style="line-height: 1.5em;">If you have read my earlier post on the Hyper-V </span><a style="line-height: 1.5em;" href="http://104.131.21.239/2013/12/17/pstip-copying-folders-using-copy-vmfile-cmdlet-in-windows-server-2012-r2-hyper-v/"><em>Copy-VMFile</em></a> <span style="line-height: 1.5em;">cmdlet, I was checking for the version of integration components (IC) installed in the virtual machines. I wanted to verify if the IC version is at a minimum supported level or not. If you look at any version number, it usually has four different parts &#8211; major, minor, build, and release numbers.</span>

The traditional way of checking for version numbers is to verify if all four version numbers match or not. This can sometimes be complex and error prone. A more easy and efficient way to do that is using [System.Version][1] .NET class. This is available as [Version] type accelerator in PowerShell.

```
PS> [Version]"6.3.9421.0"
Major    Minor    Build    Revision
-----    -----    -----    --------
6        3        9421     0
```

The [Version] type accelerator automatically converts the string into a System.Version object. It is then easy to perform the required comparisons using the PowerShell comparison operators. This class also has [several overloaded methods][2] to check version numbers that do not necessarily have all four components in the a.b.c.d version format.

```
PS> $PSVersionTable.BuildVersion -eq [Version]"6.3.9423.0"
False

PS> $PSVersionTable.BuildVersion -eq [Version]"6.3.9421.0"
True

PS> $PSVersionTable.BuildVersion -ge [Version]"6.3.9421.0"
True

PS> $PSVersionTable.BuildVersion -le [Version]"6.3.942.0"
False

PS> $PSVersionTable.BuildVersion -le [Version]"6.3.9942.0"
True

PS> $PSVersionTable.BuildVersion -ne [Version]"6.3.9942.0"
True
```

[1]: http://msdn.microsoft.com/en-us/library/system.version(v=vs.110).aspx
[2]: http://msdn.microsoft.com/en-us/library/system.version_methods(v=vs.110).aspx