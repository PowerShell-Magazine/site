---
title: '#PSTip Create an empty folder structure'
author: Ravikanth C
type: post
date: 2012-11-23T19:00:52+00:00
url: /2012/11/23/pstip-create-an-empty-folder-structure/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

In an earlier tip, you saw how an [empty file of a specified size][1] can be created. In today&#8217;s tip, we shall see how an empty folder structure can be created.

In the good old DOS days, we would create an empty folder structure by using _md_ or _mkdir_ commands. This is the PowerShell era. So, how do we do that in PowerShell? Simple:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; New-Item -ItemType Directory -Path ".\FolderX\FolderY\FolderZ"
</pre>

That is it!

The &#8216;.&#8217; at the beginning of _-Path_ parameter value tells [New-Item][2] cmdlet to create the folder structure in the present working directory. Without this, the folder structure gets created at the root of the current drive.

But wait, did you know that we have _md_ and _mkdir_ commands in PowerShell too? _mkdir_ is a function defined in PowerShell that uses _New-Item_ cmdlet to create folder(s) and _md_ is an alias to _mkdir_.

```
PS C:\> Get-Command mkdir
CommandType    Name    ModuleName
-----------    ----    ----------
Function       mkdir

PS C:\> Get-Alias md
CommandType    Name        ModuleName
-----------    ----        ----------
Alias          md -> mkdir
```

So, now, we can use these commands the same way we used _New-Item_ cmdlet.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; md ".\FolderX\FolderY\FolderZ"
</pre>

[1]: http://104.131.21.239/2012/11/22/pstip-create-a-file-of-the-specified-size/
[2]: http://technet.microsoft.com/library/hh849795.aspx