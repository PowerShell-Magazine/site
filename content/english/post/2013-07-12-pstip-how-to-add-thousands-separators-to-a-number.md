---
title: '#PSTip How to add thousands separators to a number'
author: Shay Levy
type: post
date: 2013-07-12T18:00:34+00:00
url: /2013/07/12/pstip-how-to-add-thousands-separators-to-a-number/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Say you have a number like: 133903155, how quickly can you digest it? It&#8217;s much easier to read when it&#8217;s formatted as 133,903,155. One way to convert the number to a readable string is using the _-f_ Format operator:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; $number = 133903155
PS&gt; '{0:N}' -f $number
133,903,155.00
</pre>

Print the number without precision digits.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; '{0:N0}' -f $number
133,903,155
</pre>

Under the hood PowerShell is utilizing the [String.Format][1] method to format the number (see also [Composite Formatting][2]).

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; [string]::Format('{0:N0}',$number)
133,903,155
</pre>

Another way to format numbers would be by using an object&#8217;s _ToString()_ method.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; $number.ToString('N0')
133,903,155
</pre>

Note that the symbol that separates groups of integers is determined by the current culture used on your system (session):

```
PS> (Get-Culture).NumberFormat.NumberGroupSeparator
,
```


[1]: http://go.microsoft.com/fwlink/?LinkID=166450
[2]: http://go.microsoft.com/fwlink/?LinkID=166451