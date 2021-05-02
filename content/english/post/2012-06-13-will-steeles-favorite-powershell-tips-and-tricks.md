---
title: Will Steele’s Favorite PowerShell Tips and Tricks
author: Will Steele
type: post
date: 2012-06-13T18:00:59+00:00
url: /2012/06/13/will-steeles-favorite-powershell-tips-and-tricks/
views:
  - 18264
post_views_count:
  - 1959
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Having been asked to share some PowerShell tips and tricks I first got excited at the possibility, then, felt a small spell of writer&#8217;s block. Wait, tips? I&#8217;m neither an admin nor a dev. Hmm, what&#8217;s worth sharing from my little corner of the world? A lot of common tasks are well-covered, so, I riffled through my bag of ideas and came up with a few focused squarely on data. Since I deal with data and logging a good bit, the following items have been really great to have in my private tool box. Hopefully they will help others out as well.

**<span style="text-decoration: underline;">NOTE</span>**: I wait until PowerShell is fully released to the next version before moving my servers to the new version, so, all tips listed below are for version 2.0.

### Use Pattern Matching with Get-ChildItem to Decrease Search Times

Get-ChildItem supports the use of more complex pattern matching to allow for very precise searching. For a straightforward example, I&#8217;ll start off with a directory search.

If I want to find text files in a directory, C:\test, beginning with a, e or 1 I can use this pattern to locate what I need:

<pre class="brush: powershell; title: ; notranslate" title="">Get-ChildItem -Path "C:\test\[ae1]*.txt"
</pre>

In my organization we have large numbers of folders with identical structure, file naming conventions, and, predictable characteristics. So, I can extend this basic trick into much more complex searching techniques without using any sort of pipeline to filter output. In practical terms, a search such as this:

<pre class="brush: powershell; title: ; notranslate" title="">Get-ChildItem -Path D:\data -Recurse | Where-Object { $_.Name -match '[ae1]\w+' }
</pre>

When this search executes it walks the directory tree and can be extremely slow when there are large file/folder collections to examine. Essentially this is single threading the searching and leaving the algorithm to control how quickly you get what you need.

There&#8217;s no need for that. In my case, I spell out a tree with wildcards to restrict searching to an exact depth, use patterns to eliminate unnecessary matches like this:

<pre class="brush: powershell; title: ; notranslate" title="">Get-ChildItem -Path "D:\data\*\*\200[5-8]*\[ae1]*.txt"
</pre>

This search looks at all folders 4 levels deep matching 2005, 2006, 2007 or 2008 and lists .txt files whose names begin with a, e or 1. Now that&#8217;s precision searching. In my universe I have seen well-crafted searches cut time from run to results from days to minutes.

Where this is really cool is that the same trick works for other PSProviders as well.  Here is a snapshot of the Registry using the same approach:

<pre class="brush: powershell; title: ; notranslate" title="">Get-ChildItem -Path 'HKLM:\SOFTWARE\Microsoft\Microsoft SQL*\*sql*'
</pre>

See what other providers take advantage of this capability and speed up searching with greater result accuracy.

### Capture the Exact Location of Errors in Scripts

A few of my projects require very verbose logging. If an error occurs I want to be able to know exactly where things went bad instead of having to reverse engineer the train wreck. The two main tools PowerShell offers to assist in this process are the try/catch/finally block and the InvocationInfo object. In my scripts I use the following pattern, with transcripts, to take advantage of the $Error object.

```
# Simple function to capture time stamps
function Get-TimeStamp
{
	Get-DateTime -Format 'yyyy-MM-dd HH:mm:ss'
}

# Record output
Start-Transcript "C:\logs\$(Write-TimeStamp).txt"

# Configure script
Set-StrictMode -Version 2.0

# Set preferences
$ErrorActionPreference = 'Stop'

# Clear error variable
$Error.Clear()

# try/catch/finally pattern
try
{
	Do-Work
}
catch
{
	Write "$(Get-TimeStamp):"
	Write "$(Get-TimeStamp): --------------------------------------------------"
	Write "$(Get-TimeStamp): -- SCRIPT PROCESSING CANCELLED"
	Write "$(Get-TimeStamp): --------------------------------------------------"
	Write "$(Get-TimeStamp):"
	Write "$(Get-TimeStamp): Error in $($_.InvocationInfo.ScriptName)."
	Write "$(Get-TimeStamp):"
	Write "$(Get-TimeStamp): --------------------------------------------------"
	Write "$(Get-TimeStamp): -- Error information"
	Write "$(Get-TimeStamp): --------------------------------------------------"
	Write "$(Get-TimeStamp):"
	Write "$(Get-TimeStamp): Line Number: $($_.InvocationInfo.ScriptLineNumber)"
	Write "$(Get-TimeStamp): Offset: $($_.InvocationInfo.OffsetInLine)"
	Write "$(Get-TimeStamp): Command: $($_.InvocationInfo.MyCommand)"
	Write "$(Get-TimeStamp): Line: $($_.InvocationInfo.Line)"
	Write "$(Get-TimeStamp): Error Details: $($_)"
	Write "$(Get-TimeStamp):"
	Write "$(Get-TimeStamp): --------------------------------------------------"
	Write "$(Get-TimeStamp): -- Error information"
	Write "$(Get-TimeStamp): --------------------------------------------------"
	Write "$(Get-TimeStamp):"
}
finally
{
	Stop-Transcript
}
```

If I run my script and hit a snag, this approach will generate something along these lines:

<pre class="brush: plain; title: ; notranslate" title="">2012-05-25 09:51:24: --------------------------------------------------
	2012-05-25 09:51:24: -- SCRIPT PROCESSING CANCELLED
	2012-05-25 09:51:24: --------------------------------------------------
	2012-05-25 09:51:24:
	2012-05-25 09:51:24: Error in C:\scripts\ConfigureServer.ps1.
	2012-05-25 09:51:24:
	2012-05-25 09:51:24: --------------------------------------------------
	2012-05-25 09:51:24: -- Error information
	2012-05-25 09:51:24: --------------------------------------------------
	2012-05-25 09:51:24:
	2012-05-25 09:51:24: Line Number: 1858
	2012-05-25 09:51:24: Offset: 15
	2012-05-25 09:51:24: Command: New-Item
	2012-05-25 09:51:24: Line: 						New-Item -ItemType Directory -path $log | Out-Null
	2012-05-25 09:51:24: Error Details: Cannot find drive. A drive with the name '(X)' does not exist.
	2012-05-25 09:51:24:
	2012-05-25 09:51:24: --------------------------------------------------
	2012-05-25 09:51:24: -- Error information
	2012-05-25 09:51:24: --------------------------------------------------
	2012-05-25 09:51:24:
	**********************
	Windows PowerShell Transcript End
	End time: 20120525095124
	**********************
</pre>

The catch block is passed the $Error object and processes it as the current object in the pipeline. By calling down into the InvocationInfo object my transcript tells me exactly where I need to look for the issue: line 1858 at column 15. The main gotcha with this trick is that the transcript feature is relegated to the console host off the shelf.

If I want to explicitly cancel processing in the try block I found that using throw, not Write-Error, was the key. When I used Write-Error it escaped the try scope and the InvocationInfo object referenced line 1 of its own scope.  Here is how I handle explicit, non-terminating errors:

<pre class="brush: powershell; title: ; notranslate" title="">try
{
	if(-not(Get-WmiObject -Class Win32_LogicalDisk -Filter "DeviceID='X:'"))
	{
		throw "$(Get-TimeStamp): No X drive exists."
	}
	else
	{
		Do-Work
	}
}
</pre>

### Custom Sorting with Sort-Object

A while back I needed to sort on a set of folders using a System.Version object. I wrote quite a few functions trying to do some in-flight conversions. Eventually, I stumbled upon the fact the –Property parameter allows use of expressions. In my case, I populated an expression that converted the folder name into a System.Version object for sorting. No custom function was needed.

<pre class="brush: powershell; title: ; notranslate" title="">Get-ChildItem C:\test |
Where-Object {$_.Name -as [Version]} |
Sort-Object -Property @{expression={[Version] $_.name};Ascending=$true} |
Select-Object -Last 2
</pre>

This command searches my C:\test folder and parses the following folders

<pre class="brush: plain; title: ; notranslate" title="">C:\test\1.1.5.4
C:\test\1.1.5.5
C:\test\1.2.3.1
C:\test\1.2.3.2
C:\test\1.3.1.1
</pre>

And returns the last two, or, from the perspective of a System.Version object, the two highest versioned folders:

<pre class="brush: plain; title: ; notranslate" title="">C:\test\1.2.3.2
C:\test\1.3.1.1
</pre>

Here is another example using System.Guid objects with the same folder. I create 5 folders whose names are Guids with this command:

<pre class="brush: powershell; title: ; notranslate" title="">1..5 | Foreach-Object { md "C:\test\$([Guid]::NewGuid().Guid)" }
</pre>

And, I then search using the Guid datatype within my expression to sort against:

<pre class="brush: powershell; title: ; notranslate" title="">Get-ChildItem C:\test |
Where-Object {$_.Name -as [Guid]} |
Sort-Object -Property @{e={[Guid] $_.name};Ascending=$true} |
Select-Object -Last 2
</pre>

Which returns the last two directories as Guids.

<pre class="brush: plain; title: ; notranslate" title="">Mode           LastWriteTime       Length Name
----           -------------       ------ ----
d----      6/4/2012  4:00 PM        &lt;DIR&gt; c528caa9-2439-4c92-8150-b95028b03d7c
d----      6/4/2012  4:00 PM        &lt;DIR&gt; ea568689-124d-4814-8d06-128caa7797dc
</pre>

As noted in the Get-Help documentation, you can use anything for the custom sort object so long as you can get the expression to work. Other uses of this –Property/expression approach could be sorts against IP addresses, numerically ordered files, DateTime objects, TimeSpan… Once you convert the object into the type you need let .NET take care of the sorting and move onto the next task.