---
title: '#PSTip Getting the content of an object'
author: Shay Levy
type: post
date: 2012-08-30T18:00:46+00:00
url: /2012/08/30/pstip-getting-the-content-of-an-object/
views:
  - 6308
post_views_count:
  - 1317
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Getting the content of an object is usually done with the Get-Content cmdlet. For example, getting the content of the Windows hosts file:

<pre class="brush: powershell; title: ; notranslate" title="">Get-Content C:\Windows\System32\drivers\etc\hosts
</pre>

Did you know that Get-Content has a shortcut syntax to do the same?

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; ${C:\Windows\System32\drivers\etc\hosts}
</pre>

Here&#8217;s how you can get a definition of a function:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; ${function:prompt}
</pre>

**Note**: Keep in mind that the value in the braces **must be a literal**, no variables are allowed. This will not work:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; ${$env:WINDIR\System32\drivers\etc\hosts}
</pre>