---
title: '#PSTip Validate if a folder exists'
author: Ravikanth C
type: post
date: 2013-01-25T19:00:17+00:00
url: /2013/01/25/pstip-validate-if-a-folder-exists/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
There are multiple ways to verify if a folder exists or not in PowerShell. The below two are my favorite ways of doing it.

1. Using the [System.IO.Directory][1] .NET namespace

<pre class="brush: powershell; title: ; notranslate" title="">[System.IO.Directory]::Exists($foldername)
</pre>

The [Exists()][2] method returns True if the item specified is a directory and exists. I often use this method in the ValidateScript of PowerShell advanced functions inPowerShell 2.0 and above.

2. Using Test-Path cmdlet in PowerShell

<pre class="brush: powershell; title: ; notranslate" title="">Test-Path $foldername -PathType Container
</pre>

The Test-Path cmdlet returns True if and only if the specified path is a directory and exists.

[1]: http://msdn.microsoft.com/en-us/library/system.io.directory.aspx
[2]: http://msdn.microsoft.com/en-us/library/system.io.directory.exists.aspx