---
title: '#PSTip Index into scalar in PowerShell 3.0'
author: Shay Levy
type: post
date: 2013-02-07T19:01:33+00:00
url: /2013/02/07/pstip-index-into-scalar-in-powershell-3-0/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

When you assign the output of a command to a variable, you can&#8217;t know in advance how many objects are in the variable. It may contain one object (scalar) or an array (collection) of objects. In versions prior to PowerShell 3.0, if you treat a variable as a collection of objects and try to get the first item when the variable contains a single object, you&#8217;ll get the following error:

```
PS> $foo = 1
PS> $foo[0]

Unable to index into an object of type System.Int32.
```

In PowerShell 2.0, the workaround used to avoid that error was to force the result to an array:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; [array]$foo=1
PS&gt; $foo[0]
1
</pre>

In PowerShell 3.0, we can now index into scalars without the need to tweak the object:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; $foo[0]
1
</pre>