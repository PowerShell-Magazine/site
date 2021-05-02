---
title: '#PSTip How to get only files – the PowerShell 3.0 way!'
author: Ravikanth C
type: post
date: 2012-08-27T19:08:24+00:00
url: /2012/08/27/pstip-how-to-get-only-files-the-powershell-3-0-way/
views:
  - 14852
post_views_count:
  - 2447
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
The traditional or PowerShell 2.0 way of retrieving only files is:

<pre class="brush: powershell; title: ; notranslate" title="">Get-ChildItem $env:windir | Where-Object { ! $_.PSIsContainer }
</pre>

The new parameter -File of the Get-ChildItem cmdlet simplifies the filtering:

<pre class="brush: powershell; title: ; notranslate" title="">Get-ChildItem $env:windir -File
</pre>

Another way to achieve this in PowerShell 3.0 is another new parameter of [Get-ChildItem][1] cmdlet, –Attributes. It can be used to not only make this process easier but also speed up the whole process when executing against a large folder! This parameter takes the [file system attributes][2] as arguments and it also lets a way to combine with logical operators to achieve what we want. Let us see how:

<pre class="brush: powershell; title: ; notranslate" title="">Get-ChildItem $env:windir -Recurse -Attributes !D
</pre>

In the above command, ‘!D’ specifies that we want to retrieve only items that are not directories! Simple and fast. Isn’t it?

[1]: http://technet.microsoft.com/en-us/library/hh847897.aspx
[2]: http://msdn.microsoft.com/en-us/library/system.io.fileattributes(lightweight).aspx