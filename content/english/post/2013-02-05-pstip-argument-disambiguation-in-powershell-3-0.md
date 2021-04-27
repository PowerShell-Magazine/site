---
title: '#PSTip Argument disambiguation in PowerShell 3.0'
author: Shay Levy
type: post
date: 2013-02-05T20:25:37+00:00
url: /2013/02/05/pstip-argument-disambiguation-in-powershell-3-0/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

When writing commands in PowerShell you can be as formal as:

<pre class="brush: powershell; title: ; notranslate" title="">Get-ChildItem -Include*.txt -Recurse
</pre>

Or you can be as terse as:

<pre class="brush: powershell; title: ; notranslate" title="">gci -i *.txt -r
</pre>

As you can see you use aliases (_gci_) instead of writing the full command name and you can also use partial names of parameters (_i,r_). Shortened parameter names are allowed as long as you have specified enough of the parameter name to make it unique.

For example, what would happen if we specify _-f_ to _Get-ChildItem:_

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; gci -f *.txt
Get-ChildItem : Parameter cannot be processed because the parameter name 'f' is ambiguous.
Possible matches include: -Filter -Force.</pre>

As you can see, specifying just _&#8216;f&#8217;_ is not enough as it yields two matches, so the parameter binder can&#8217;t decide which one to use. To disambiguate it we need to add more characters; in this case &#8216;fi&#8217; would suffice.

In PowerShell 3.0 we can also use the same technique to shorten parameter arguments. This works as long as the parameter type is an Enum object. So, instead of writing:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Write-Host hello -ForegroundColor Yellow
</pre>

We can now do:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Write-Host hello -ForegroundColor y
</pre>

If you try to use &#8216;b&#8217; for example, you&#8217;ll get an error that says it all:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Write-Host hello -ForegroundColor b
Write-Host : Cannot bind parameter 'ForegroundColor'. Cannot convert value "b" to type "System.ConsoleColor". Error:
"The identifier name b cannot be processed due to the inability to differentiate between the following enumerator
names: Black, Blue. Try a more specific identifier name."
</pre>