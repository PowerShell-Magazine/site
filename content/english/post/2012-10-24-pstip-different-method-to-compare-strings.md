---
title: '#PSTip CompareTo() method for comparing strings!'
author: Ravikanth C
type: post
date: 2012-10-24T18:00:41+00:00
url: /2012/10/24/pstip-different-method-to-compare-strings/
categories:
  - Columns
  - Tips and Tricks
tags:
  - PowerShell
  - PSTip

---
There are certainly many methods to compare strings in PowerShell.

Today, I will show you one of the methods that I recently came across &#8212; a less known method, maybe.

<pre class="brush: powershell; title: ; notranslate" title="">"PowerShell Magazine".CompareTo("PowerShell magazine")
</pre>

The output of [CompareTo()][1] method will be zero if strings are equal and -1 or 1, otherwise.

So, what do you expect as an output for the above command? 0 or 1?

The output in this case will be 1. Well, this is because the [CompareTo()][1] method does a case sensitive search by default. But, technically, the strings in our example are not different if we discount the case of these strings. So, how do we do a case-insensitive search using CompareTo() method?

Here&#8217;s how:

<pre class="brush: powershell; title: ; notranslate" title="">[string]::Compare("PowerShell Magazine", "PowerShell magazine", $True)
</pre>

[1]: http://msdn.microsoft.com/en-us/library/35f0x18w.aspx