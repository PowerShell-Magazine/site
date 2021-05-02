---
title: '#PSTip Explore values that can be used with Start-Process’s -Verb parameter'
author: Ravikanth C
type: post
date: 2012-12-12T19:00:16+00:00
url: /2012/12/12/pstip-explore-values-that-can-be-used-with-start-processs-verb-parameter/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks
---
**Note**: This tip requires PowerShell 2.0 or above.

We know that _Start-Process_ cmdlet can be used to start a new process. This cmdlet has a parameter called &#8211;_Verb_ that specifies what action to perform while creating the new process. The value of the parameter depends on the file type of the process being created. Let us see an example first:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Start-Process notepad.exe -Verb RunAs
</pre>

The above code starts notepad.exe process as administrator. But, how do we know what all verbs are supported for a given file type?

Simple! We can use [System.Diagnostics.ProcessStartInfo][1] class for that.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; $processInfo = New-Object System.Diagnostics.ProcessStartInfo -ArgumentList "test.exe"
PS&gt; $processInfo.Verbs
open
runas
runasuser
</pre>

As you see in the above command, we can use the _New-Object_ cmdlet to initialize the _ProcessStartInfo_ class constructor with a specific file type. The value of _-ArgumentList_ parameter is just a dummy file name and it doesn&#8217;t need to exist.

[1]: http://msdn.microsoft.com/en-us/library/27xafxda.aspx