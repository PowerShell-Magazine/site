---
title: '#PSTip How to expand environment variable contained in a text file'
author: Aleksandar Nikolic
type: post
date: 2012-09-07T18:00:08+00:00
url: /2012/09/07/pstip-how-to-expand-environment-variable-contained-in-a-text-file/
views:
  - 9651
post_views_count:
  - 2088
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Let&#8217;s presume you have a text file that contains the following two lines:

My current user profile is $env:USERPROFILE.

The system folder is $env:windir\System32.

If you use the Get-Content cmdlet to output the content, the environment variables will not be expanded:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Get-Content .\test.txt
My current user profile is $env:USERPROFILE.
The system folder is $env:windir\System32.
</pre>

The solution is to use the ExpandString() method as in:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Get-Content .\test.txt | ForEach-Object { $ExecutionContext.InvokeCommand.ExpandString($_) }
My current user profile is C:\Users\Aleksandar.
The system folder is C:\Windows\System32.
</pre>