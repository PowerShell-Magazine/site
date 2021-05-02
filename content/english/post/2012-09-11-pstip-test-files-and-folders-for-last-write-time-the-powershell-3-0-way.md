---
title: '#PSTip Test files and folders for last write time – the PowerShell 3.0 way!'
author: Ravikanth C
type: post
date: 2012-09-11T18:00:54+00:00
url: /2012/09/11/pstip-test-files-and-folders-for-last-write-time-the-powershell-3-0-way/
views:
  - 14187
post_views_count:
  - 2563
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
In PowerShell version 2.0, when we had to check if a given file or folder is older or newer, we would do that using:

```
#Test for newer item in PowerShell v2
Get-Item C:\Scripts | Where-Object { $_.LastWriteTime -ge "August 19, 2012 2:00 PM"}

#Test for Older item in PowerShell v2
Get-Item C:\Scripts | Where-Object { $_.LastWriteTime -le "August 30, 2012 2:00 PM"}
```

In PowerShell 3.0, this is much simpler. Thanks to the -OlderThan and -NewerThan parameters of Test-Path cmdlet. When we use these parameters, based on the test condition the resulting output is either True or False.

```
#Test for newer item in PowerShell v3
Test-Path -Path C:\Scripts -NewerThan "August 30, 2012 2:00 PM"

#Test for older item in PowerShell v3
Test-Path -Path C:\Scripts -OlderThan "August 30, 2012 2:00 PM"
```

It is not always date literals. You can also use the [datetime] object to compare. For example,

<pre class="brush: powershell; title: ; notranslate" title="">Test-Path -Path C:\Scripts -OlderThan (Get-Date).AddDays(-20)
</pre>

Now, if you want to get a list of files older than a given date in a recursive way,

<pre class="brush: powershell; title: ; notranslate" title="">Get-ChildItem -Recurse | ? { Test-Path -Path $_.FullName -OlderThan "August 10, 2011 2:00 PM" }
</pre>