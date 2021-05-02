---
title: '#PSTip Get the contents of a file in one string'
author: Shay Levy
type: post
date: 2012-11-13T19:00:22+00:00
url: /2012/11/13/pstip-get-the-contents-of-a-file-in-one-string/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

The Get-Content cmdlet returns the contents of a file as an array of strings delimited by a newline character. In most cases this is not a problem but sometimes you&#8217;d want to get the content as one string instead of a collection of strings.

In PowerShell 2.0 and below, getting the file content as a single string required one of two methods:

1. Use a .NET method to read all lines of the file

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; $content = [System.IO.File]::ReadAllText($path)
</pre>

2. Pipe the result of Get-Content to the Out-String cmdlet

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; $content = Get-Content -Path $path | Out-String
</pre>

In PowerShell 3.0 we now have a new dynamic parameter, Raw. When specified, Get-Content ignores newline characters and returns the entire contents of a file in one string. Raw is a dynamic parameter, it is available only in file system drives.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Get-Content $path -Raw
</pre>